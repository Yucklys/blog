---
title: 在 React 中引入 Electron
date: 2018-12-16
categories:
- Front-end
tags:
- React
- Electron
---

最近体验了一下 Electron，迫不及待的换上了我心爱的 React，仿佛看见了新世界，但是当我试图`import { ipcRenderer } from 'electron'`会报错 `TypeError: fs.existsSync is not a function`。

<!--more-->

## Webpack target属性
首先这个问题早已有人想到了，webpack的`target: electron-renderer`属性就是为此而来的。这里展示的是改造`create-react-app`的方法，其他脚手架诸如此类。

项目新建好后，首先一发`yarn eject`展开所有配置，在多出来的config文件夹下就有我们需要更改的`webpack.config.js`文件。`create-react-app`在基础的webpack配置上添加了很多内容，比如说环境检测以及sass，不过我们这里直接在return里添加target属性，如果想要根据develop或product环境切换target的话可以自行添加。

```javascript
// other configs...
module.exports = function(webpackEnv) {
// some configs
    return {
        target: 'electron-renderer', // 添加target
        // other configs
    }
}
```

OK，现在我们再运行`yarn start`的话，会发现通过浏览器无法打开`localhost:3000`了。接下来在根目录新建Electron主程序入口`main.js`。

```javascript
const { app, BrowserWindow } = require('electron');

// 保持对window对象的全局引用，如果不这么做的话，当JavaScript对象被
// 垃圾回收的时候，window对象将会自动的关闭
let win;

function createWindow() {
	// 创建浏览器窗口。
	win = new BrowserWindow({ width: 800, height: 600 });

	// 然后加载应用的 index.html。
	// win.loadFile('index.html');
	// 这里没有选择官方的从文件加载的方法，而是选择从localhost加载
	win.loadURL('http://localhost:3000');

	// 打开开发者工具
	win.webContents.openDevTools();

	// 当 window 被关闭，这个事件会被触发。
	win.on('closed', () => {
		// 取消引用 window 对象，如果你的应用支持多窗口的话，
		// 通常会把多个 window 对象存放在一个数组里面，
		// 与此同时，你应该删除相应的元素。
		win = null;
	});
}

// Electron 会在初始化后并准备
// 创建浏览器窗口时，调用这个函数。
// 部分 API 在 ready 事件触发后才能使用。
app.on('ready', createWindow);

// 当全部窗口关闭时退出。
app.on('window-all-closed', () => {
	// 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
	// 否则绝大部分应用及其菜单栏会保持激活。
	if (process.platform !== 'darwin') {
		app.quit();
	}
});

app.on('activate', () => {
	// 在macOS上，当单击dock图标并且没有其他窗口打开时，
	// 通常在应用程序中重新创建一个窗口。
	if (win === null) {
		createWindow();
	}
});

// 在这个文件中，你可以续写应用剩下主进程代码。
// 也可以拆分成几个文件，然后用 require 导入。
```

同时更改`package.json`，加入`main: "main.js"`，`homepage: "./"`，同时在scripts中添加`"electron": "electron ."`，不要忘记`yarn add electron`。

准备工作完成之后，可以写一个小例子来测试一下渲染进程以及主进程的通信。

```javascript
// 在这个文件中，你可以续写应用剩下主进程代码。
// 也可以拆分成几个文件，然后用 require 导入。
const { ipcMain } = require('electron');

ipcMain.on('react-test', (event, arg) => {
	console.log(`get ${arg} from Renderer process.`);
	event.sender.send('main-process-reply', 'world');
}); // 接收到react-test信号后返回main-process-reply并打印'get ${arg} from Renderer process'
```

```javascript
const { ipcRenderer } = require('electron');

class App extends {
    componentDidMount() {
        ipcRenderer.send('react-test', 'Hello'); // 发送react-test信号，同时arg为'Hello'

        ipcRenderer.on('main-process-reply', (_event, arg) => {
            console.log(`Hello ${arg}`);
        }); // 接收main-process-reply并打印arg
    }

    render() {
        // ...
    }
}
```

接下来依次运行`yarn start`以及`yarn electron`，在终端以及electron应用的控制台应该都可以看到打印出的信息，通信成功！

## Preload提前引入electron
利用webpack的target属性确实是最稳妥，最官方的解决方案了，但是对于一些高度封装webpack并且没有eject选项的脚手架来说，就无法通过target属性来简单解决了。这里使用的是`dva new --demo`新建的一个项目，我们会在主进程中提前引入我们需要的包以达到在渲染进程中使用的目的。

首先开始的步骤同上，安装electron并且新建main.js，在主进程文件main.js中写入我们之前用过的测试例子，同时做一些修改。

```javascript
function createWindow() {
  win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: __dirname + '/preload.js'
    }
  });

  win.loadURL('http://localhost:8000'); // Roadhog默认的localhost端口为8000
  // ...
}
```

这里我们在preload选项中引用了根目录下在preload.js文件，在该文件中写入：

```javascript
global.electron = require('electron');
```

OK，现在我们修改`src/index.js`文件，就可以实现进程间通讯了。

```javascript
// ...
// 3. Router
class HomePage extends React.Component {
  componentDidMount() {
    const { ipcRenderer } = window.electron;
    ipcRenderer.send('react-test', 'Hello');
    ipcRenderer.on('main-process-reply', (_event, arg) => {
      console.log(`Hello, ${arg}`);
    });
  }

  render() {
    return <div>Hello, world!</div>;
  }
}
// ...
```

通过这种方式，在浏览器中打开的话会报错，无法引用ipcRenderer，但是如果通过electron打开的话就可以正常地进行通讯了。而且通过`preload: 'file'`这种方式，我们还可以很方便地把一些设置全局共享，比如：

```javascript
const os = require('os');
const path = require('path');

global.path = path.join(os.homedir(), '.config');
```

## 总结
除开这两种自己动手的方法以外，在 GitHub 上还有各种xxx-electron-boilerplate可供使用，不过有些年代比较久远，并且年久失修，issue 没人处理，很多依赖都落后几个版本了，反而不是很好用。当然，[electron-react-boilerplate](https://github.com/electron-react-boilerplate/electron-react-boilerplate)这个1w star项目现在还是很活跃的，比起自己这样凑活着搭起来的框架更成体系。具体是怎么样的结合就看自己项目的需求了。
