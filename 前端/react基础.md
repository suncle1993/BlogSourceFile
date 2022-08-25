---
categories:
  - 前端
date: '2017-04-27T21:53:00'
description: ''
tags:
  - react
title: react基础
---



# Hello World

Hello World作为开篇示例

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Root extends React.Component {
  render() {
    return <h1>Hello World!</h1>;
  }
}

ReactDOM.render(<Root />, document.getElementById('root'));
```

# JSX简介

react使用的JSX是JavaScript的一个语法扩展，上述Hello World示例中的return语句即为JSX写法。

<!--more-->

以下代码1和代码2的效果是一模一样的：

**代码1：使用JavaScript代码构建DOM**

```javascript
class Root extends React.Component {
  render() {
    const child1 = React.createElement('li', null, 'First Text Content');
    const child2 = React.createElement('li', null, 'Second Text Content');
    const root = React.createElement('ul', { className: 'my-list' }, child1, child2);
    return root
  }
}

ReactDOM.render(<Root />, document.getElementById('root'));
```

**代码2：使用JSX构建DOM**

```javascript
class Root extends React.Component {
  render() {
    const root =(
      <ul className="my-list">
      <li>First Text Content</li>
      <li>Second Text Content</li>
      </ul>
    );
    return root
  }
}

ReactDOM.render(<Root />, document.getElementById('root'));
```

对比两种代码可以发现JSX构建 的DOM比原生的JavaScript代码构建的DOM更简洁更易读。

JSX是将XML语法直接加入到JavaScript代码中，所以可以直接用代码构建界面。之后JSX通过翻译器转换到纯JavaScript再由浏览器执行。在实际开发中，JSX在产品打包阶段都已经编译成纯JavaScript，JSX的语法不会带来任何性能影响。

因此，可以将JSX理解为为提升开发效率而发明的一个比较高级但很直观的语法糖。它非常有用，却不是一个必需品，没有JSX的React也可以正常工作

**代码3：JSX解析是通过首字母大小写区分组件类和HTML标签** 

```javascript
class root extends React.Component {
  render() {
    const element =(
      <ul className="my-list">
      <li>First Text Content</li>
      <li>Second Text Content</li>
      </ul>
    );
    return element
  }
}

ReactDOM.render(<root />, document.getElementById('root'));
```

将以上组件类Root改名为root之后render在做JSX解析的时候会判定成HTML标签，因此渲染没有结果。

**代码4：更新JSX元素**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Root extends React.Component {
  render() {
    return <h1>It is {(new Date()).toLocaleTimeString()}</h1>
  }
}

function render() {
  ReactDOM.render(<Root />, document.getElementById('root'))
}

setInterval(render, 1000)
```

每个一秒钟刷新一次Root组件，即更新一次Root组件的JSX。
# Component组合

React是基于组件的，整个项目就是各个组件拼接而成，这也是目前最主流前端架构。以下代码会演示组件之间的组合方式

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class ChildB extends React.Component {
  render() {
    return (
      <ul className="ChildA-list">
        <li>first</li>
        <li>second</li>
      </ul>
    )
  }
}

class ChildA extends React.Component {
  render() {
    return <h3>ChildA</h3>
  }
}

class Root extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello World</h1>
        <ChildA />
        <ChildB />
      </div>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('root'))
```

ChildA，ChildB和Root三个组件之间可以随意组合成为最后的项目。

# 内部状态state

**代码1：使用setState方法修改state**

要想使界面上显示的Hello abc在三秒后变为Hello suncle，就可以通过修改组件内部状态state来实现。

如果直接修改state的属性值并不会产生效果，代码如下：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Root extends React.Component {
  state = {name: 'abc'}
  render() {
    setTimeout(() => this.state.name = 'suncle', 3000)
    return (
      <div>
        <h1>{`Hello ${this.state.name}`}</h1>
      </div>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('root'))
```

所以，我们需要通过setState方法来修改state，setState方法是组件内部的方法，使用方法如下：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Root extends React.Component {
  state = {name: 'abc'} // 组件的内部状态，只能在组件内部共享
  render() {
    setTimeout(() => this.setState({name: 'suncle'}), 3000)
    return (
      <div>
        <h1>{`Hello ${this.state.name}`}</h1>
      </div>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('root'))
```

**代码2：根据input组件更新state**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Root extends React.Component {
  state = {name: 'abc'}	

  handleChange(event) {
    this.setState({name: event.target.value})
  }
  render() {
    return (
      <div>
        <input value={this.state.name} onChange={this.handleChange.bind(this)}/>
        <h1>{`Hello ${this.state.name}`}</h1>
      </div>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('root'))
```

# 组件参数传递props

state是组件的内部状态，组件和组件之间是不能共享的。如果父组件需要给子组件传递参数，那么就需要通过xml attribute的方式给组件传递props。

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class Child extends React.Component {
  render() {
    // this.props即通过xml attribute传递进来的参数
    return <h1>{`Hello ${this.props.name}`}</h1>
  }
}

class Root extends React.Component {
  state = {name: 'abc'}

  handleChange(event) {
    this.setState({name: event.target.value})
  }
  render() {
    return (
      <div>
        <input value={this.state.name} onChange={this.handleChange.bind(this)}/>
        <Child name={this.state.name}/>
      </div>
    )
  }
}

ReactDOM.render(<Root />, document.getElementById('root'))
```

---

**参考**

1. [react官方文档](https://facebook.github.io/react/)
2. [深入理解React中es6创建组件this的方法](http://www.jb51.net/article/91447.htm)
3. [reactjs-state-vs-prop](http://stackoverflow.com/questions/23481061/reactjs-state-vs-prop)

**附录**

由前端开发的配置越来越复杂，依赖项也越来越多，因此构建好一个基础开发环境就显得尤为重要，[react-mobx-starter](https://github.com/Flowsnow/react-mobx-starter)这个项目构建的基础环境就非常适用于react-mobx开发调试。