---
title: React 滚动事件监听
date: 2018-11-19
categories:
- Front-end
tags:
- React
toc: true
---

对于多页面的网站来说，一个导航栏是必不可少的。现在常用的一些 64 px 高度的导航栏占用的空间对于一些展示型网站来说就有点过于大了。为了能够节省下这部分空间益达到完美的图像展示空间，我们需要在适当的时候隐藏导航栏。

<!--more-->

## 自动隐藏Header

### 初始化

首先还是以`create-react-app`作为框架，在终端下输入`create-react-app react-scroll`创建项目文件夹，创建完成后进入文件夹输入`npm start`，如果顺利的话此时已经可以在`localhost:3000`看见初始的应用界面了。

首先先改一下源文件，修改header并且加上一个足够长的内容(2000px)，让页面可以滚动足够的距离。这里为了节约篇幅，只放上需要改动的部分。

```javascript
//......
class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          Header1
        </header>
        <div className="App-container" />
      </div>
    );
  }
}
//......
```

对应的css文件也需要改一下，同上，这里的代码只是需要改动的部分，未在这里贴出的代码不需要调整。

```css
.App-header {
  position: fixed; /*固定header*/
  top: 0px;
  width: 100%;
  min-height: 64px;
}

.App-container {
  margin: 70px;
  width: auto;
  height: 2000px;
  border-radius: 10px;
  background-color: rgba(126, 248, 238, 0.856);
}
```

现在的页面应该是这个样子，现在我们可以进行下一步的修改，让header能随着我们的滚动而消失或浮现。![初始化](https://i.loli.net/2018/11/19/5bf2b0528ee57.gif)

### 添加滚动事件监听及判断

#### 滚动事件监听

现在我们需要加入对滚动事件的监控来获取滚动数据。在这个例子里使用的是`window.addEventLister('scroll', function() {})`方法，这个方法接受两个参数，`event`和`function`，针对我们这个例子的话分别是`scroll`事件并且绑定为`handleScroll`函数。

```javascript
class App extends Component {
    componentDidMount() {
        window.addEventLister('scroll', handleScroll);
    }
    
    handleScroll() {
        console.log('scroll');
    }
    
    //...
}
```

![滚动初体验](https://i.loli.net/2018/11/22/5bf622abc976d.gif)

#### 滚动事件判断

现在知道了我们的页面什么时候滚动，那怎么判断我们滚动的方向呢？当添加`window.addEventListener`并把`scroll`事件绑定到`handleScroll`之后，就可以通过`window.scrollY`获取现在滚动的位置距离**页面顶部**有多远[^1]。

```javascript
class App extends Component {
    //...
    handleScroll() {
        console.log(window.scrollY)
    }
    //...
}
```

![Y轴座标](https://i.loli.net/2018/11/22/5bf625cf80f86.gif)

那如何通过这个参数判断滚动方向呢？假设说我们有一个`prevScroll`的变量，保存着我们之前页面所在的Y轴座标，那么我们通过`window.scrollY - prevScroll`就可以判断滚动方向了。如果这个数值为正，则为向下滚动；如果这个数值为负，则为向上滚动。

![滚动判断](https://i.loli.net/2018/11/22/5bf6295d240f0.png)

如果对React足够熟悉的话应该立马就想到了`this.state`。现在我们创建一个`this.state.prevScroll`用来储存上一次的页面滚动位置，然后在每一次`handleScroll`进行完判断之后更新`this.state.prevScroll`。

```javascript
class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            prevScroll: 0 //初始化prevScroll
        }
    }
    //...
    //滚动判断
    handleScroll() {
        const { prevScroll } = this.state;
        const changeY = window.scrollY;
        
        if (changeY >= 0) {
            console.log('scroll downward')
        } else {
            console.log('scroll upward')
        }
        
        this.setState({ prevScroll: window.scrollY })
    }
    //...
}
```

![滚动判断](https://i.loli.net/2018/11/22/5bf63ee2a7bfa.gif)

现在通过同样的原理，把header的style属性指定为`this.state.headerStyle`，在滚动判断中添加适当的内容，就可以实现根据页面滚动显示或隐藏元素了。

![滚动显示/隐藏header](https://i.loli.net/2018/11/22/5bf68cd4f4147.gif)

#### 一些优化

细心的人可能发现了，我们这个组件反复渲染了多次，如果在`render`函数下加入一个`console.log('render')`的话，就可以看到对于每一个scroll事件的变化，我们的组件都渲染了两次，而且同样是scroll downward，依然进行了渲染，这样子的话有可能对性能造成不必要的负担。所以这里加上`shouldComponentUpdate`函数来判断什么时候进行渲染。

```javascript
class App extends Component {
    constructor(props) {
        super(props);
        this.state = {
            prevScroll: 0,
            currentDirection: 0,
            headerStyle: {}
        };
    }
    //...
    shouldComponentUpdate(nextProps, nextState) {
        return (nextState.currentDirection !== this.state.currentDirection)
    }
    
    componentWillUnmount() {
        window.removeEventListener('scroll', this.handleScroll.bind(this));
    }
    //...
}
```

![优化滚动显示](https://i.loli.net/2018/11/22/5bf6ab0d794ca.gif)

### 最后

当然，这里的demo只是一个最简单的版本，实际上在这个例子的基础上还可以添加更多的条件判断，并且`scroll`事件的函数也不止`scrollY`，善用这些事件监听可以让你的页面更加地灵活。

[^1]: 当然与之相对还有`window.scrollX`，监控现在滚动位置距离页面左侧有多远。

