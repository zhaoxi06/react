# 一、 React.Component
ES6：使用`import React from 'react`加载React<br>
ES5：使用`var React = require('react')`加载React<br>
也可以使用`<script>`标签加载
## 1.1常用的生命周期方法
1. `render()`方法是必须的。返回以下类型中的一个：
    - React元素：通常由JSX创建。可写法能是个原生DOM，也可能是个组件
    - 字符串和数字：这些将被渲染为DOM中的text node
    - Protals：由ReactDOM.createPortal创建
    - null：什么都不渲染
    - 布尔值：什么都不渲染。通常存在于`return test && <Child />`写法

    当返回null 或 false时，ReactDOM.findDOMNode(this) 将返回 null。<br>
render()函数应该纯净，意味着其不应该改变组件的状态，其每次调用都应返回相同的结果，同时不直接和浏览器交互。若需要和浏览器交互，将任务放在componentDidMount()阶段或其他的生命周期方法。保持render() 方法纯净使得组件更容易思考。<br>
**注意：** 若`shouldComponentUpdate()`返回false，render()函数将不会被调用。

2. `constructor()`：若不初始化状态且不绑定方法，那也不需要为React组件定义一个构造函数。<br>
**避免以下这种方式**<br>
`this.state = {color: props.initialColor};`<br>

3. `componentDidMount()`在组件挂载（插入树中）后立即执行。这个方法一般用于实现以下内容：

    - 对DOM节点初始化
    - 从远端加载数据，实例化网络请求
    - 设置订阅。但不应忘记在`componentWillUnmount()`取消订阅。

    可以在`componentDidMount()`里面调用`setState()`，但这样将会触发额外的渲染，但它是在浏览器更新屏幕之前发生。这保证了即使`render()`被调用两次，用户也不会看到中间状态。但需 **谨慎** 使用此模式，因为它通常会导致性能问题。但是，当需要在渲染依赖于其大小或位置的东西之前测量DOM节点时，可能需要对模态和工具提示灯情况进行处理。

4. `componentDidUpdate()`：在更新发生后立即调用。初始渲染不会调用此方法。<br>
    `componentDidUpdate(prevProps, prevState, snapshot)`<br>
    这是在更新组件时对DOM进行操作的好时机。同时，这也是一个适合发送请求的地方，当对比了当前属性和之前属性有变化时；如果没有变化，也就不要请求。
    ```
        componentDidUpdate(prevProps){
            if(this.props.userID !== prevProps.userID){
                this.fetchDate(this.props.userID);
            }
        }
    ```
    可以在`componentDidUpdate()`里面调用`setState()`，但要注意，**必须在一个条件下被包裹** ，比如像上面的例子，否则会导致死循环。<br>
    **注意：** 若`shouldComponentUpdate()`返回false，componentDidUpdate()函数将不会被调用。

5. `componentWillUnmount()`：在卸载和销毁组件之前立即调用。在此方法中执行任何必应的清理，例如使计时器无效，取消网络请求或清除在其中创建的任何订阅。

## 1.2很少用的生命周期方法
1. `shouldComponentUpdate(nextProps, nextState)`：当接收到新属性或状态时，在渲染前被调用。不会在初始化渲染或当使用`forceUpdate()`时被调用。<br>
    此方法仅作为 **性能优化** 存在，不要依赖它来阻止渲染，因为这可能导致错误。可以考虑使用`PureComponent`。

2. `static getDerivedStateFromProps()`：组件实例化后和接受新属性时，在`render`方法之前将会调用。返回一个对象来更新状态，或者返回`null`来表明新属性不需要更新状态。<br>
    **注意：** 如果父组件导致了组件的重新渲染，即使属性没有更新，这一方法也会被调用。调用<br>
    调用`this.setState()`通常不会触发`getDerivedStateFromProps()`。

3. `getSnapshotBeforeUpdate(prevProps, prevState)`：在最近呈现的输出被提交到DOM之前调用。它使组件可以在可能更高之前从DOM捕获一些信息（例如滚动位置）。此生命周期的返回值将作为参数传递给`componentDidUpdate()`。

## 1.3错误边界

&emsp;&emsp;错误边界是React组件。它捕获发生在子组件树中任意地方（渲染期间、生命周期方法中、整棵树的构造函数中）的JS错误，打印错误日志，并且显示回退的用户界面，而不是损坏的组件树。<br>
如果定义了这个生命周期方法，这个组件将成为一个错误边界。<br>
仅适用错误边界来从意外中恢复，**不要试图将它们用户控制流程**。

1. `static getDerivedStateFromError(error)`：在后代组件抛出错误后调用此生命周期。它接收作为参数抛出的错误，并返回值以更新状态。
2. `componentDidCatch(error, info)`：接收两个参数，第一个参数是抛出的错误；第二个参数是包含哪个组件引发错误的信息。

## 1.4其他API
1. `setState(updater, [callback])`将需要处理的变化塞入（将需要改变的变化存放到组件的state对象中，采用队列处理）组件的state对象中，并告诉该组件及其子组件需要用更新的状态来重新渲染。这是用于响应事件处理和服务端响应的更新用户界面的主要方式。<br>
&emsp;&emsp;将`setState()`认为是一次请求，而不是一次立即执行更新组件的命令。出于性能考虑，React可能会推迟它，稍后会一次性更新这些组件，也就是说，它可能是批处理或推迟更新。这使得在调用`setState()`后立即读取`this.state`的一个潜在陷阱。代替地，使用`componentDidUpdate`或一个`setState`回调，可以保证在更新被应用之后触发。<br>
&emsp;&emsp;除非`shouldComponentUpdate()`返回false，否则`setState()`永远都会导致重新渲染。<br>
&emsp;&emsp;`setState()`接收两个参数，第一个参数是带签名的updater函数。<br>
&emsp;&emsp;`(prvState, props) => stateChange`<br>
&emsp;&emsp;第二个参数是可选的回调函数。它会在setState执行完成以及组件被重渲之后执行。通常对于这类逻辑，推荐使用`componentDidUpdate`。<br>
&emsp;&emsp;也可以选择性地传递一个对象作为`setState()`的第一个参数而不是一个函数。<br>
&emsp;&emsp;下面这种形式的setState()也是异步的，并在相同的周期中多次调用可能会被合并到一起。换句话说，下面例子中，之后的调用在同一周期中将会重新之前调用的值，因此数量仅会被加一。
```
    Object.assign(
        previousState,
        {quantity: state.quantity + 1},
        {quantity: state.quantity + 1},
        ...
    )
```

2. `forceUpdate()`默认情况下，组件或状态发生改变，组件将会重渲。若render()方法依赖其他数据，可以通过调用`forceUpate()`来告诉React组件需要重渲。<br>
&emsp;&emsp;调用`forceUpdate()`将会导致组件的`render()`方法被调用，并忽略`shouldComponentUpdate()`。这将会触发每一个子组件的生命周期方法，涵盖每个子组件的shouldComponentUpdate()方法。若当标签改变，React仅会更新DOM。<br>
&emsp;&emsp;通常应该尝试避免所有forceUpdate() 的用法并仅在render()函数里从this.props和this.state读取数据。

## 1.5类属性

1. `defaultProps`可以被定义为组件类的一个属性，用来给类设置默认的属性。这对于未定义(undefined)的属性来说有用，而对于设置为空(null)的属性并没有用。
```
    class CustonButton extends React.Component{
        // ...
        render(){
            //props.color将会被设置为blue
            return <CustomButton />;
            //props.color仍然是null
            return <CustomButton color={null} />;
        }
    }
    CustomButton.defaultProps = {
        color: 'blue'
    };
```
2. `displayName`被用在调试信息中。JSX会自动设置该值。

## 1.6 实例属性
1. `props`
2. `state`

# 二、 ReactDOM
ES6：使用`import ReactDOM from 'react-dom`加载ReactDOM<br>
ES5：使用`var ReactDOM = require('react-dom')`加载ReactDOM<br>
也可以使用`<script>`标签加载<br>
react-dom这个软件包提供了针对DOM的方法，可以在应用的顶级域中调用，也可以在有需要的情况下用作跳出React模型的出口。大部分组件都不应该需要使用这个包。

1. `render()`渲染一个React元素，添加到位于提供的container里的DOM元素中，并返回这个组件的一个引用 (或者对于无状态组件返回null).<br>
&emsp;&emsp;如果这个React元素之前已经被渲染到container里去了，这段代码就会进行一次更新，并且只会改变那些反映元素最新状态所必须的DOM元素。<br>
&emsp;&emsp;回调函数是可选的。如果你提供了，程序会在渲染或更新之后执行这个函数。
```
    ReactDOM.render(
        element,
        container,
        [callback]
    )
```

2. `unmountComponentAtNode(container)`从DOM元素中移除已挂载的React组件，清除它的事件处理器和state。如果容器内没有挂载任何组件，这个函数什么都不会干。 有组件被卸载的时候返回true，没有组件可供卸载时返回 false。
3. `findDOMNode(component)`
4. `createPortal()`创建一个portal。

# 三、 ReactDOMServer
ES6：使用`import ReactDOMServer from 'react-dom/server`加载ReactDOM<br>
ES5：使用`var ReactDOMServer = require('react-dom/server')`加载ReactDOM<br>
ReactDOMServer 类可以让你在服务端渲染你的组件。

1. `renderToString(element)`把一个React元素渲染为原始的HTML。可以用这个方法在服务端生成HTML，并根据初始请求发送标记来加快页面的加载速度，同时让搜索引擎可以抓取你的页面来达到优化SEO的目的。可以在浏览器和服务器中使用。
2. `renderToStaticMarkup()`类似 renderToString，但是不会创建额外的DOM属性，例如 data-reactid 这些仅在React内部使用的属性。如果希望把React当作一个简单的静态页面生成器来使用，这很有用，因为去掉额外的属性可以节省很多字节。可以在浏览器和服务器中使用。

# 四、 DOM Elements
&emsp;&emsp;在React中，所有的DOM特性和属性（包括事件处理函数）都是小驼峰命名法命名。比如说，与HTML中的tabindex属性对应的React实现命名则是tabIndex。aria-*和data-*属性是例外的，一律使用小写字母命名。
## 4.1 React和HTML DOM属性的区别
1. checked属性<br>
`<input>`标签type属性值为checkbox或radio时，支持checked属性。这对于构建受控组件很有用。与之相对defaultChecked这是非受控组件的属性，用来设定对应组件首次加载时是否选中状态。
2. 类名属性<br>
在React中，使用className属性指定一个CSS类。
3. dangerouslySetInnerHTML函数.是React提供的替换浏览器DOM中的innerHTML接口的一个函数。
4. htmlFor
5. onChange函数
6. selected
7. style属性<br>
**通常不建议使用style属性作为样式元素的主要方法**。在大多数情况下，className应该用于引用外部CSS样式表中定义的类。style最常用于React应用程序，以在渲染时添加动态计算样式。<br>
&emsp;&emsp;style属性接受一个键为小驼峰命名法命名的javascript对象作为值，而不是像css字符串。这和DOM中style属性接受javascript对象对象key的命名方式保持一致性，更高效而且能够防止跨站脚本（XSS）的安全漏洞.
```
    const divStyle = {
        color: 'blue',
        backgroundImage: 'url(' + imgUrl + ')',
        WebkitTransition: 'all',
        msTransition: 'all'
    };
    function HelloWorldComponent(){
        return <div style={divStyle}>Hello World!</div>;
    }
```
样式属性不会自动补齐前缀的。为了支持旧的浏览器，需要手动支持相关的样式特性.样式key使用小驼峰命名法是为了和JS访问DOM特性对对象的处理保持一致性（例如 node.style.backgroundImage）。浏览器后缀除了ms以外，都应该以大写字母开头。这就是为什么WebkitTransition有一个大写字母W。