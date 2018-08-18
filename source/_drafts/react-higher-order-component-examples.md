---
title: React高阶组件研究
date: 2018-07-13 15:34:43
categories:
  - 编程语言
  - JavaScript
tags:
  - react
  - hoc
---

> 高阶组件就是一个 React 组件包裹着另外一个 React 组件

因为高阶组件一般通过函数实现，我们可以用下面的伪代码来表现，

```
Function(WrappedComponent) => EnhancedComponent
```

为了下面的内容好写一些，我们这里约定一些术语，

>**WrappedComponent**: 被包裹的 React Component
>
>**EnhancedComponent**: 包裹着 WrappedComponent 的 React Component，即函数返回的组件

好了，我自己都觉得有点绕口。接下来说下 **包裹** 这个词，它包括下面两种情况的其中一种，

1. Props Proxy
2. Inheritance Inversion

下面进入正题。

<!--more-->

## Props Proxy

简称 PP，这种情况下，HOC 对 WrappedComponent 的 props 进行操作，最经典的例子是 React-Redux。

下面我们通过代码来解释下 PP 的用途。

### 修改 WrappedComponent 的 props 

```jsx
function PPHOC(WrappedComponent) {
    return class PP extends React.Component {
        render() {
            const props = Object.assign({}, this.props, {
                user: {
                    name: 'Fran',
                    email: 'franleplant@gmail.com',
                },
            });
            return <WrappedComponent {...props} />;
        }
    };
}

class Example extends React.Component {
    render() {
        return (
            <div>
                <p>现在Exmaple的props具有date,user两个属性。</p>
                <pre>{JSON.stringify(this.props, null, 2)}</pre>
            </div>
        );
    }
}

const EnhancedExample = PPHOC(Example);
```

通过这段代码我们可以了解如何通过 PP 操作 WrappedComponent 组件的 props，需要注意的是，如果要进行的是删除或者编辑 props 的操作，要确保不会破坏原有的 WrappedComponent 依赖的 props，例如某个 props 值是必须的，但是你删除了。

### 通过 refs 访问 WrappedComponent 实例 

```jsx
function PPHOC(WrappedComponent) {
    return class PP extends React.Component {
        constructor(props) {
            super(props);

            this.state = {
                name: '',
            };

            this.updateName = this.updateName.bind(this);
        }

        updateName(instance) {
            if (instance.instanceName !== this.state.name)
                this.setState({ name: instance.instanceName });
        }

        render() {
            const props = Object.assign({}, this.props, {
                ref: this.updateName,
            });

            return (
                <div>
                    <h2>HOC Component</h2>
                    <p>这里state的name属性是WrappedComponent组件传入的</p>
                    <pre>{JSON.stringify(this.state, null, 2)}</pre>
                    <WrappedComponent {...props} />
                </div>
            );
        }
    };
}

class Example extends React.Component {
    constructor(props) {
        super(props);

        this.instanceName = 'jack';
    }

    render() {
        return (
            <div>
                <h2>Wrapped Component</h2>
                <p>Props</p>
                <pre>{JSON.stringify(this.props, null, 2)}</pre>
            </div>
        );
    }
}

const EnhancedExample = PPHOC(Example);
```

## Inheritance Inversion