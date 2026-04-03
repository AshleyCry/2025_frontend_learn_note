## React

### 手动修改props中的值会怎样？

1. 严格模式下

   React会组织你修改props并抛出警告

   ```js
   function MyComponent(props) {
     props.name = "修改后的值"; // 尝试修改props
     return <div>{props.name}</div>;
   }
   
   // 控制台警告：
   // Warning: Cannot assign to read only property 'name' of object '#<Object>'
   ```

2. 非严格模式下

   虽然JS允许修改，但：

   - 修改不会触发重新渲染（React的响应式系统不检测props的直接修改）
   - 破坏单向数据流，可能导致组件状态不可预测
   - 父组件重新渲染时，修改会覆盖（父组件传递的新props会替换修改）

### super()和super(props)有什么区别

#### ES6类

ES6中，通过extends实现类的继承，使用super调用父类，super代替的是父类的构造函数，super(name)相当于father.prototype.constructor.call(this, name)

如果不在子类中使用super会引发错误，子类没有自己的this对象，只能继承父类的this对象，然后对其进行加工。super()就是将父类的this继承给子类的，没有super()子类就得不到this对象

先调用this，再初始化super()也是禁止的行为

#### 类组件

React的类组件必须继承React.component，因此必须使用super

调用super()一般都要传入props作为参数，如果不传进去，React内部也会将其定义在组件实例中

```js
class HelloMessage extends React.Component {
   render() {
     return <div>nice to meet you! {this.props.name}</div>;
   }
}
```

不建议使用super()代替super(props)，React会在类组件构造函数生成实例之后再给this.props赋值，构造函数中在不传递props的情况下this.props为空

```js
class Button extends React.Component {
 constructor(props) {
 super(); //  props
 console.log(props);      //  {}
 console.log(this.props); //  undefined
 // ...
 }
}
```

### 函数式组件和类组件

#### 区别

##### 编写形式 略

##### 状态管理

hooks出现之前函数试组件不能管理状态，不能调用this.setState()，需要创建一个类组件或者state提升到父组件中，通过props对象传递到子组件

现在可以使用useState hook

##### 生命周期

函数式组件不存在生命周期，因为生命周期钩子要从React.Component继承

函数式组件使用UseEffect代替生命周期的作用

useEffect(callback, [])代替componentDidMount生命周期

如果callback中return一个函数，这个函数会在组建卸载的时候执行，对应componentWillUnmount

##### 调用方式

函数组建的调用是执行函数

```js
 //  你的代码
function SayHi() { 
	return <p>Hello, React</p > 
} 
// React 内部
const result = SayHi(props) // » <p>Hello, React</p >
```

类组件需要将组件实例化，然后调用对象的render方法

```js
//  
class SayHi extends React.Component { 
  render() { 
  	return <p>Hello, React</p > 
  } 
} 
// React 
const instance = new SayHi(props) // » SayHi {} 
const result = instance.render() // » <p>Hello, React</p >
```

### 受控组件和非受控组件

#### 受控组件

受我们控制的组件，组建的状态全程相应外部数据

```js
class TestComponent extends React.Component {
 constructor (props) {
   super(props);
   this.state = { username: 'lindaidai' };
 }
 render () {
 	return <input name="username" value={this.state.username} />
 }
}
```

当在输入框输入的时候，会发现输入的内容无法显示出来，这是因为value被this.state.username控制住，当用户输入新的内容时，this.state.username无法自动更新

解除被控制，可以为input标签设置onchange事件，输入的时候触发事件函数，在函数内部实现state的更新

因此受控组件一般需要初始状态和一个状态更新函数

#### 非受控组件

不受我们控制的组件

当需要时，可以使用ref查询DOM并查找其当前值

```jsx
import React, { Component } from 'react';
 export class UnControll extends Component {
   constructor (props) {
   super(props);
   this.inputRef = React.createRef();
 }
 handleSubmit = (e) => {
 	 console.log(' input ', this.inputRef.current.value);
   e.preventDefault();
 }
 render () {
  return (
   <form onSubmit={e => this.handleSubmit(e)}>
   <input defaultValue="lindaidai" ref={this.inputRef} />
   <input type="submit" value=" " />
   </form>
  )
 }
}
 
```

### React事件机制

React基于浏览器事件机制实现了一套事件机制，包括事件注册、事件合成、事件冒泡、事件派发，这套机制被称为合成事件

#### 合成事件（SyntheticEvent）

合成事件是React模拟原生DOM事件所有能力的一个事件对象，即浏览器原生事件的跨浏览器包装器

根据W3C规范来定义合成事件，兼容所有浏览器，拥有与浏览器原生事件相同的接口

```jsx
const button = <button onClick={handleClick}>按钮</button>
```

如果想要获得原生DOM事件，可以通过e.nativeEvent属性获取

```jsx
const handleClick = (e) => console.log(e.nativeEvent);;
 const button = <button onClick={handleClick}>按钮</button>
```

##### React事件与原生事件的区别

- 事件名称命名方式不同

  ```jsx
  // 原生事件绑定方式
  <button onclick="handleClick()"> </button>
   // React 合成事件绑定方式
  const button = <button onClick={handleClick}> </button>
  ```

- 事件处理函数书写不同（例子同上）

onclick看似绑定到DOM元素上，但实际并不会把事件代理函数直接绑定到真实的节点上，而是把所有的事件绑定到结构最外层，使用一个统一的事件去监听

#### 执行顺序

```jsx
import  React  from 'react';
 class App extends React.Component{
 
  constructor(props) {
    super(props);
    this.parentRef = React.createRef();
    this.childRef = React.createRef();
  }
  componentDidMount() {
    console.log("React componentDidMount ");
    this.parentRef.current?.addEventListener("click", () => {
      console.log(" 原生事件：父元素DOM事件监听 ");
    });
    this.childRef.current?.addEventListener("click", () => {
      console.log(" 原生事件：子元素DOM事件监听 ");
    });
    document.addEventListener("click", (e) => {
      console.log(" 原生事件：document DOM事件监听 ");
    });
  }
  parentClickFun = () => {
    console.log("React事件：父元素事件监听 ");
  };
  childClickFun = () => {
    console.log("React事件：子元素事件监听 ");
  };
  render() {
    return (
      <div ref={this.parentRef} onClick={this.parentClickFun}>
        <div ref={this.childRef} onClick={this.childClickFun}>
          分析事件执行顺序
        </div>
      </div>
    );
  }
 }
 export default App
```

输出顺序：

1. 原生事件：子元素DOM事件监听
2. 原生事件：父元素DOM事件监听
3. React事件：子元素事件监听
4. React事件：父元素事件监听
5. 原生事件：document DOM事件监听

可以得出结论：

- React所有事件都挂载在document对象上，v17之后变成了root
- 当真实的DOM元素触发事件，会冒泡到document对象后，再处理React事件
- 会先执行原生事件，然后处理react事件
- 最后真正执行document上挂载的事件

![图片描述](https://segmentfault.com/img/bVbd8Ro)

所以想要阻止不同时间段的冒泡行为，对应使用不同的方法，对应如下：

- 阻止合成事件间的冒泡，用e.stopPropagetion()

- 阻止合成事件与最外层document上的事件间的冒泡，用e.nativeEvent.stopImmediatePropagetion()

- 阻止合成事件与除最外层document上的原生事件的冒泡，通过判断e.target来避免

  ```jsx
  document.body.addEventListener('click', e = {
    if (e.target && e.target.matches('div.code')) {
    	return
  	}
    this.setState({ active: false })
  })
  ```

总结：

- React上注册的事件最终会绑定在document这个DOM上，而不是React组建对应的DOM（节省内存开销）
- React自身实现了一套事件冒泡机制，这也就是为什么event.stopPropagation()无效的原因
- React通过队列的形式，从出发的组件向父组件回溯，然后调用他们JSX中定义的callback
- React有一套自己的合成事件SyntheticEvent

### React中引入CSS的方式

组件式开发选择合适的css解决方案尤为重要

通常遵循以下规则：

- 可以编写局部css，不会随意污染其他组建内的原生
- 可以编写动态的css，可以获取当前组件的一些状态，根据状态的变化生成不同的css样式
- 支持所有的css特征：伪类、动画、媒体查询等
- 便携起来简洁方便、最好符合一贯的css风格特点

在这一方面，vue使用css起来更为简洁：

- 通过style标签编写样式
- scoped属性决定编写的样式是否局部有效
- lang属性设置预处理器
- 内联样式风格的方式来根据最新状态设置和改变css

常见的ReactCss引入方式：

- 组件内直接使用
- 组建中引入css文件
- 组建中引入.module.css文件
- CSS in JS

#### 组建内直接使用

直接写到style属性中，可根据state动态变化样式，写法需要使用驼峰，一些样式没有提示，伪类伪元素等无法编写

#### 组件中引入css文件

将css单独写在一个文件中，然后在组建中直接引入，样式是全局生效，样式之间会互相影响

#### 组件中引入.module.css文件

将css文件座位一个模块引入，这个模块中的所有css只作用于当前组件。不会影响当前组件的后代组建

这种方式是webpack特供的方案，只需要配置webpack配置文件中modules: true即可

缺点：

- 引用的类名，不能使用连接符，再JS中不识别
- 所有的classname必须都是用style.className的形式编写
- 不方便动态修改样式，依然需要内联的方式

#### CSS in JS

css由js生成，而不是外部定义，使用这些第三方库可以实现：

- styled-components
- emotion
- glamorous

这些组件会被自动添加上一个不重复的class，然后第三方库给该class添加相关的样式

### React生命周期

与VUE一样，React整个组件生命周期包括从创建、初始化数据、编译模板、挂载Dom -> 渲染、更新 -> 渲染、卸载等一系列过程

react16.4之后可以分成三个阶段

- 创建阶段
- 更新阶段
- 卸载阶段

#### 创建阶段

创建阶段主要分以下生命周期方法：

- constructor
- getDerivedStateFromProps
- render
- conponentDidMount

##### constructor

实例过程自动调用的方法，通过super来获取父组件的props，在该方法中，通常的操作为初始化state状态或者在this上挂载方法

##### getDerivedStateFromProps

该方法是新增的生命周期方法，是一个静态的方法，不能访问到组件的实例

执行时机：组建创建和更新阶段，不论是props变化还是state变化，都会调用

每次render方法前调用，第一个参数为即将更新的props，第二个参数为上一个状态的state，可以比较props和state来加一些限制条件，防止无用的state更新

该方法需要返回一个新的对象作为新的state或者返回null表示state状态不需要更新

##### render

类组件必须实现的方法，用于DOM结构，可以访问组件state与prop属性

不要在render里面setState，会触发死循环导致内存崩溃

##### componentDidMount

组件挂载到真实DOM节点后执行，其在render方法执行后执行

此方法多用于执行一些数据获取，事件监听等操作

#### 更新阶段

该阶段的函数主要为：

- getderivedStateFromProps
- shouldComponentUpdate
- render
- getSnapshotBeforeUpdate
- componentDidUpdate

##### shouldComponentUpdate

用于告知组建本身基于当前的props和state是否需要重新渲染组件，默认情况返回true

执行时机：有新的props或者state时都会调用，通过返回true或者false告知组件更新与否

一般情况不建议在该周期中进行深层比较，会影响效率

同时也不能调用setState，否则会导致无限循环调用更新

##### getSnapshotBeforeUpdate

该周期函数在render后执行，执行的时候DOM元素还没有被更新

该方法返回的一个snapshot值，座位componentDidUpdate第三个参数传入

```jsx
getSnapshotBeforeUpdate(prevProps, prevState) {
  console.log('#enter getSnapshotBeforeUpdate');
  return 'foo';
}
componentDidUpdate(prevProps, prevState, snapshot) {
  console.log('#enter componentDidUpdate snapshot = ', snapshot);
}
```

此方法的目的在于获取组建更新前的一些信息，比如组件的滚动位置之类的，在组建更新后可以根据这些信息恢复一些UI视觉上的状态

##### componentDidUpdate

执行时机：组件更新结束后触发

再该方法中，可以根据前后的props和state的变化做相应的操作，如获取数据，修改DOM样式等

#### 卸载阶段

##### componentWillUnmount

此方法用于组建卸载之前，清理一些注册的监听事件，或者取消订阅的网络等

一旦一个组件实例被卸载，其不会被再次挂载，而只能是被重新创建

#### 旧的生命周期：

![img](https://pic1.zhimg.com/v2-48e4dd255a7690beaef4d496ac6af7ca_1440w.jpg)

#### 新的生命周期：

![img](https://i-blog.csdnimg.cn/blog_migrate/c2ed2581a69e90453f4054553df3224d.png)

新的生命周期减少了三种方法：

- componentWillMount
- componentWillReceiveProps
- componentWillUpdate

这三种方法依然存在，只是在前者加上了UNSAFE_前缀，如UNDAFE_componentWillMount

同时新增了：

- getDerivedStateFromProps
- getSnapshotBeforeUpdate

### 高阶组件

高阶函数至少满足下列一个条件

- 接受一个或多个函数作为输入
- 输出一个函数

在React中，高阶组件接受一个或多个组件作为参数，并返回一个组件，本质也就是一个函数，并不是组件

高阶组件这种实现方式，本质上是一个装饰者设计模式

#### 如何编写

```jsx
import React, { Component } from 'react'
export default (WrappedComponent) => {
  return class EnhancedComponent extends Component {
    // do something
    render() {
      return <WrappedComponent />
    }
  }
}
```

通过对传入的原始组件WrappedComponent做一些想要的操作（比如操作Props， 提取state，给原始组件包裹其他元素等），从而加工出想要的组件EnhancedComponent

把通用的逻辑放在高阶组件中，对组件实现一致的处理，从而实现代码复用

所以高阶组件的主要功能是封装并分离组件的通用逻辑，让通用逻辑在组件间更好地被复用

在使用高阶组件的同事，一般遵循一些约定：

- props保持一致
- 不能再函数式（无状态）组建上使用ref属性，因为它没有实例
- 不要以任何方式改变原始组件WrappedComponent
- 透传不相关props属性给被包裹的组件WrappedComponent
- 不要再render（）方法中使用高阶组件
- 包装显示名字以便调试

高阶组件可以传递所有的props，但是不能传递ref

如果向一个高阶组件添加ref引用，那么ref指向的是最外层容器组件实例的，而不是被包裹的组件，如果需要传递refs的话，使用React.forwardRef

```jsx
 function withLogging(WrappedComponent) {
   class Enhance extends WrappedComponent {
     componentWillReceiveProps() {
       console.log('Current props', this.props);
       console.log('Next props', nextProps);
     }
     render() {
       const {forwardedRef, ...rest} = this.props;
       //  把forwardedRef赋值给ref
       return <WrappedComponent {...rest} ref={forwardedRef} />;
     }
 	};
    // React.forwardRef 方法会传入 props 和 ref 两个参数给其他回调函数
    // 所以这边的 ref 是由 React.forwardRef 提供的
    function forwardRef(props, ref) {
      return <Enhance {...props} forwardRef={ref} />
    }
 		return React.forwardRef(forwardRef);
 }
 const EnhancedComponent = withLogging(SomeComponent)
```

#### 应用场景

高阶组件能够提高代码的复用性和灵活性，在实际应用中，常常用于与核心业务无关但又在多个模块使用的功能，比如权限控制、日志记录、数据校验、异常处理、统计上报等

例如存在一个组件，需要从缓存中获取数据，然后渲染。

```jsx
import React, { Component } from 'react'
 class MyComponent extends Component {
   componentWillMount() {
   let data = localStorage.getItem('data');
   this.setState({data});
 }
 render() {
 	 return <div>{this.state.data}</div>
 }
}
```

如果有其他组件也有类似功能的时候，每个组件都需要重复编写componentWillMount里面的代码，使用高阶组件可以解决这个冗余问题

```jsx
 import React, { Component } from 'react'
 function withPersistentData(WrappedComponent) {
   return class extends Component {
   	 componentWillMount() {
   		 let data = localStorage.getItem('data');
   		 this.setState({data});
 	 	 }
     render() {
     // 通过{...this.props}把传递给当前组件的属性继续传递给被包装的组件WrappedComponent
     	 return <WrappedComponent data={this.state.data} {...this.props} />
     }
 	 }
 }
 class MyComponent2 extends Component {  
		render() {
 			return <div>{this.props.data}</div>
 		}
 }
 const MyComponentWithPersistentData = withPersistentData(MyComponent2)
```

### React过渡动画

当一个组件在显示与消失过程中存在过渡动画，可以很好的增加用户体验

React实现过渡动画可以有很多选择，如react-transition-group，react-motion, Animated，以及原生的CSS都能完成切换动画

react-transition-group可以很方便的实现组件的入场和离场动画

其提供了三个主要的组件：

- CSSTransition：结合CSS完成过渡动画效果
- SwitchTransition：两个组件显示和隐藏切换时，使用该组件
- TransitionGroup：将多个动画组件包裹在其中，一般用于列表中元素的动画

#### CSSTransition

当CSSTransition的in属性为true时，CSSTransition首先会给子组件加上xxx-enter、xxx-enter-active的class执行动画

动画执行后，移除两个class，并添加-enter-done的class

所以可以利用这点通过css的transition属性，让元素在两个状态之间平滑过渡，从而得到相应的动画效果

当in属性设为false时，CSSTransition给子组件加上xxx-exit, xxx-exit-active的class，然后执行动画，结束后移除两个class，并添加-enter-done的class

```jsx
 export default class App2 extends React.PureComponent {
   state = {show: true};
   onToggle = () => this.setState({show: !this.state.show});
   render() {
     const {show} = this.state;
     return (
       <div className={'container'}>
       	 <div className={'square-wrapper'}>
           <CSSTransition
             in={show}
             timeout={500}
             classNames={'fade'}
             unmountOnExit={true}
           >
             <div className={'square'} />
           </CSSTransition>
         </div>
         <Button onClick={this.onToggle}>toggle</Button>
       </div>
     );
   }
 }
```

对应的CSS为：

```css
.fade-enter {
   opacity: 0;
   transform: translateX(100%);
 }
 .fade-enter-active {
   opacity: 1;
   transform: translateX(0);
   transition: all 500ms;
 }
 .fade-exit {
   opacity: 1;
   transform: translateX(0);
 }
 .fade-exit-active {
   opacity: 0;
   transform: translateX(-100%);
   transition: all 500ms;
 }
```

#### SwitchTransition

SwitichTransition可以完成两个组件之间切换的炫酷动画

比如有一个按钮需要在on和off之间切换，我们希望看到on先从左侧退出，off再从右侧进入

SwitchTransition主要有一个属性mode，对应两个值：

- in-out：表示新组建先进入，旧组件再移除
- out-in：表示旧组件先移除，新组件再进入

SwitchTransition组件里面要有CSSTransition，不能直接包裹想要切换的组件。里面的CSSTransition组件不再像以前那样接收in属性来判断元素是何种状态，取而代之的是key属性

下面给出一个按钮入场和出场的示例，如下：

```jsx
import { SwitchTransition, CSSTransition } from "react-transition-group";
 
export default class SwitchAnimation extends PureComponent {
  constructor(props) {
    super(props);
 
    this.state = {
      isOn: true
    }
  }
 
  render() {
    const {isOn} = this.state;
 
    return (
      <SwitchTransition mode="out-in">
        <CSSTransition classNames="btn"
                       timeout={500}
                       key={isOn ? "on" : "off"}>
          {
          <button onClick={this.btnClick.bind(this)}>
            {isOn ? "on": "off"}
          </button>
        }
        </CSSTransition>
      </SwitchTransition>
    )
  }
 
  btnClick() {
    this.setState({isOn: !this.state.isOn})
  }
 }
```

css如下：

```css
.btn-enter {
  transform: translate(100%, 0);
  opacity: 0;
}
.btn-enter-active {
  transform: translate(0, 0);
  opacity: 1;
  transition: all 500ms;
}
.btn-exit {
  transform: translate(0, 0);
  opacity: 1;
}
.btn-exit-active {
  transform: translate(-100%, 0);
  opacity: 0;
  transition: all 500ms;
}
```

#### TransitionGroup

当有一组动画的时候，可以将这些CSSTransition放入一个TransitionGroup中完成动画

同样CSSTransition中没有了in属性，用到了key属性

TransitionGroup在感知children发生变化后，先保存移除的节点，当动画结束后才真正移除

- 插入的节点，先渲染DOM，然后再做动画
- 删除的节点，先做动画，然后再删除DOM

```jsx
import React, { PureComponent } from 'react'
import { CSSTransition, TransitionGroup } from 'react-transition-group';
export default class GroupAnimation extends PureComponent {
  constructor(props) {
    super(props);
    this.state = {
      friends: []
    }
  }
  render() {
    return (
      <div>
        <TransitionGroup>
          {
            this.state.friends.map((item, index) => {
              return (
                <CSSTransition classNames="friend" timeout={300} key={index}>
                  <div>{item}</div>
                </CSSTransition>
              )
            })
          }
        </TransitionGroup>
        <button onClick={e => this.addFriend()}>+friend</button>
      </div>
    )
  }
  addFriend() {
    this.setState({
      friends: [...this.state.friends, "coderwhy"]
    })
  }
}
```

css：

```css
.friend-enter {
  transform: translate(100%, 0);
  opacity: 0;
}
.friend-enter-active {
  transform: translate(0, 0);
  opacity: 1;
  transition: all 500ms;
}
.friend-exit {
  transform: translate(0, 0);
  opacity: 1;
}
.friend-exit-active {
  transform: translate(-100%, 0);
  opacity: 0;
  transition: all 500ms;
}

```

### React捕获异常

React组件内JS代码错误会导致React内部状态被破坏，导致整个应用崩溃，React自身对于错误的处理有一些解决方案

为了解决出现的错误导致整个应用崩溃，react16引入了错误边界的概念

错误边界是一种React组件，这种组建可以捕获发生在子组件树任何位置的Javascript错误，并打印这些错误，同事展示降级UI，而并不会渲染那些发生崩溃的子组件树

错误边界在渲染期间、生命周期方法和整个组件树的构造函数中捕获错误

形成错误边界的两个条件：

- 使用了static getDiveredStateFromError()
- 使用了componentDidCatch()

抛出错误后，请使用static getDerivedStateFromError()处理错误，使用componentDidCatch()打印错误信息，如下：

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  static getDerivedStateFromError(error) {
    // 更新state使下一次渲染能够显示降级后的UI
    return { hasError: true };
  }
  componentDidCatch(error, errorInfo) {
    // 可以将错误日志上报给服务器
    logErrorToMyService(error, errorInfo);
  }
  render() {
    if (this.state.hasError) {
      // 可以自定义降级后的UI并渲染
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}
```



然后就可以把自身组件作为错误边界的子组件，如下：

```jsx
<ErrorBoundary>
  <MyWidget />
</ErrorBoundary>
```



以下情况无法捕获到异常：

- 事件处理器中的错误
- 异步代码
- 服务端渲染
- Error Boundary自身抛出来的错误
- React之外的错误
  - 全局事件监听
  - Web Worker中的错误
  - 第三方库(Redux, Zustand)的全局错误

目前还没有办法将错误边界编写为函数式组件。但是你不必自己编写错误边界类。例如，你可以使用 [`react-error-boundary`](https://github.com/bvaughn/react-error-boundary) 包来代替。

react16之后，会把渲染期间发生的所有错误打印到控制台

除了错误信息和JavaScript栈外，React16还提供了组件栈追踪。现在可以准确地查看发生在组件树内的错误信息

对于错误边界无法捕获的异常，比如事件处理过程中发生问题并不会被捕获到，因为其不会在渲染期间触发，并不会导致渲染问题

这种情况可以使用js的try catch，或者监听onerror事件

### React refs

Refs在计算机中称为弹性文件系统（Resilient File System，简称ReFS）

React中的Refs提供了一种方式，允许我们访问DOM节点或在render方法中创建的React元素

本质为ReactDOM.render()返回的组件实例，如果是渲染组件则返回的是组件实例，如果渲染dom则返回具体的dom节点

创建refs的方式有三种：

- 传入字符串，使用时通过this.refs.传入的字符串的格式获取对应的元素
- 传入对象，对象通过React.createRef()的方式创建出来，使用时获取到的创建的对象中存在current属性就是对应的元素
- 传入函数，该函数会在DOM被挂载时进行回调，这个函数会传入一个元素对象，可以自己保存，使用时，直接拿到之前保存的元素对象即可
- 传入hook，通过useRef创建，使用时通过生成hook对象的current属性就是对应的数据

#### 传入字符串

```jsx
 class MyComponent extends React.Component {
   constructor(props) {
     super(props);
     this.myRef = React.createRef();
	 }
   render() {
   	 return <div ref="myref" />;
   }
 }
```

访问节点的方式如下：

```jsx
this.refs.myref.innerHTML = "hello"
```

#### 传入对象

refs通过React.creatRef()创建，然后将ref属性添加到React元素中

```jsx
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  render() {
    return <div ref={this.myRef} />;
  }
}
```

#### 传入函数

这种情况主要应对比如列表中每一项都要绑定ref的情况

**将函数传递给 `ref` 属性**。这称为 [`ref` 回调](https://zh-hans.react.dev/reference/react-dom/components/common#ref-callback)。当需要设置 ref 时，React 将传入 DOM 节点来调用你的 ref 回调，并在需要清除它时传入 `null` 。这使你可以维护自己的数组或 [Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)，并通过其索引或某种类型的 ID 访问任何 ref。

```jsx
import { useRef, useState } from "react";

export default function CatFriends() {
  const itemsRef = useRef(null);
  const [catList, setCatList] = useState(setupCatList);

  function scrollToCat(cat) {
    const map = getMap();
    const node = map.get(cat);
    node.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // 首次运行时初始化 Map。
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToCat(catList[0])}>Neo</button>
        <button onClick={() => scrollToCat(catList[5])}>Millie</button>
        <button onClick={() => scrollToCat(catList[9])}>Bella</button>
      </nav>
      <div>
        <ul>
          {catList.map((cat) => (
            <li
              key={cat}
              ref={(node) => {
                const map = getMap();
                map.set(cat, node);

                return () => {
                  map.delete(cat);
                };
              }}
            >
              <img src={cat} />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

function setupCatList() {
  const catList = [];
  for (let i = 0; i < 10; i++) {
    catList.push("https://loremflickr.com/320/240/cat?lock=" + i);
  }

  return catList;
}


```

启用严格模式后，ref 回调将在开发中运行两次。

#### 传入hook

通过useRef创建一个ref，用法与React.createRef一致

```jsx
function App(props) {
 	const myref = useRef()
 	return (
     <>
     	 <div ref={myref}></div>
     </>
 	)
 }
```

上面例子都是ref用于原生HTML上，如果ref设置的组件为一个react组件，ref对象接收到的是组件的挂载实例

### State

直接修改state的值，页面不会有任何改变，必须使用setState方法才能正常监听变化

### Virtual DOM

与普通DOM区别如下：

- 虚拟DOM不会进行排版与重绘操作，真实DOM会频繁重排与重绘
- 虚拟DOM的总损耗永远是“虚拟DOM增删改 + 真实DOM差异增删改 + 排版与重绘”，真实DOM的总损耗是“展示DOM完全增删改+排班与重绘”

传统的原生api或jQuery操作DOM时，浏览器会从构建DOM树开始从头到尾执行一遍流程

当一次操作更新10个DOM节点，浏览器收到第一个更新请求后，并不知道后续还有九次操作，因此马上执行流程，最终执行10次流程

通过VNode，同样更新10个DOM节点，虚拟DOM不会立即操作DOM，而是将这10次更新的diff内容保存到本地的一个js对象中，最终将这个js对象一次性attach到DOM树上，避免大量无谓计算

##### 优缺点

真实DOM优势：

- 易用

缺点：

- 效率低：解析速度慢，内存占用量过高
- 性能差：频繁操作真实DOM，易于导致重绘与回流

虚拟DOM优势：

- 简单方便：使用手动操作真实DOM，繁琐且容易出错，在大规模应用下维护起来比较困难
- 性能方面：使用Virtual DOM，能够有效避免真实DOM树频繁更新，减少多次引起重绘与回流，提高性能
- 跨平台：React借助虚拟DOM，带来了跨平台的能力，一套代码多端运行

缺点：

- 在一些性能要求极高的应用中虚拟DOM无法针对性能做极致优化
- 首次渲染大量DOM时，由于多了一层虚拟DOM计算，速度比正常稍慢

### JSX转换真实DOM

JSX通过babel最终转化成React.creatElement这种形式，例如：

```jsx
 <div>
   <img src="avatar.png" className="profile" />
   <Hello />
 </div>
```

会被babel转化为：

```jsx
React.createElement(
  "div",
  null,
  React.createElement("img", {
    src: "avatar.png",
    className: "profile",
  }),
  React.createElement(Hello, null)
);
```

- 当JSX组件首字母为小写时，会被认定为原生DOM标签，createElement的第一个变量被编译为字符串

- 当首字母为大写，其被认定为自定义组件，createElement的第一个变量被编译为对象

最终都会通过RenderDOM.render()进行挂载

```jsx
ReactDOM.render(<App />,  document.getElementById("root"));
```

#### 过程

react中节点大致分为四个类别

- 原声标签节点
- 文本节点
- 函数组件
- 类组件

React.createElement被调用时会传入标签类型type，标签属性props以及若干子元素children，作用是生成一个虚拟DOM对象

```jsx
function createElement(type, config, ...children) {
    if (config) {
        delete config.__self;
        delete config.__source;
    }
    // ! 源码中做了详细处理，比如过滤掉key、ref等
    const props = {
        ...config,
        children: children.map(child =>
   typeof child === "object" ? child : createTextNode(child)
  )
    };
    return {
        type,
        props
    };
 }
 function createTextNode(text) {
    return {
        type: TEXT,
        props: {
            children: [],
            nodeValue: text
        }
    };
 }
 export default {
    createElement
 };

```

- 如果是原生标签节点，type是字符串，如div, span
- 如果是文本节点，type就没有，这里是TEXT
- 如果是喊叔叔组建，type是函数名
- 如果是类组件，type是类名

虚拟DOM通过ReactDOM.render进行渲染成真实DOM，使用方法如下

```jsx
 ReactDOM.render(element, container[, callback])
```

首次调用时，容器节点中所有的DOM元素都会被替换，后续的调用则会使用React的diff算法进行高效的更新

如果提供了可选的回调函数callback，该回调将在组件被渲染或更新之后被执行

```jsx
function render(vnode, container) {
    console.log("vnode", vnode); // 虚拟DOM对象
    // vnode _> node
    const node = createNode(vnode, container);
    container.appendChild(node);
}

// 创建真实DOM节点
function createNode(vnode, parentNode) {
    let node = null;
    const {type, props} = vnode;
    if (type === TEXT) {
        node = document.createTextNode("");
    } else if (typeof type === "string") {
        node = document.createElement(type);
    } else if (typeof type === "function") {
        node = type.isReactComponent
            ? updateClassComponent(vnode, parentNode)
        : updateFunctionComponent(vnode, parentNode);
    } else {
        node = document.createDocumentFragment();
    }
    reconcileChildren(props.children, node);
    updateNode(node, props);
    return node;
}

// 遍历下子vnode，然后把子vnode->真实DOM节点，再插入父node中
function reconcileChildren(children, node) {
    for (let i = 0; i < children.length; i++) {
        let child = children[i];
        if (Array.isArray(child)) {
            for (let j = 0; j < child.length; j++) {
                render(child[j], node);
            }
        } else {
            render(child, node);
        }
    }
}
function updateNode(node, nextVal) {
    Object.keys(nextVal)
        .filter(k => k !== "children")
        .forEach(k => {
        if (k.slice(0, 2) === "on") {
            let eventName = k.slice(2).toLocaleLowerCase();
            node.addEventListener(eventName, nextVal[k]);
        } else {
            node[k] = nextVal[k];
        }
    });
}

// 返回真实dom节点
// 执行函数
function updateFunctionComponent(vnode, parentNode) {
    const {type, props} = vnode;
    let vvnode = type(props);
    const node = createNode(vvnode, parentNode);
    return node;
}

// 返回真实dom节点
// 先实例化，再执行render函数
function updateClassComponent(vnode, parentNode) {
    const {type, props} = vnode;
    let cmp = new type(props);
    const vvnode = cmp.render();
    const node = createNode(vvnode, parentNode);
    return node;
}
export default {
    render
};
```



渲染流程汇总如下：

- 使用React.creatElement将JSX代码转换为react element，Babel帮我们完成了这个转换的过程
- createElement函数对key, ref等特殊props进行处理，获取defaultProps对默认props进行赋值，并且对传入的孩子结点进行处理，最终构造成一个虚拟DOM对象
- ReactDOM.render将生成好的虚拟DOM渲染到指定容器上，起哄采用了批处理、事物等机制并且对特定浏览器进行了性能优化，最终转换为真实的DOM

![img](https://i-blog.csdnimg.cn/blog_migrate/9f56eb2fb9d38776d2459641ef61fb2f.png)

### Key

React的diff算法中，key的主要作用就是判断元素是新创建的还是被移动的，从而减少不必要的元素渲染，因此key需要是一个唯一的确定标识

- key应该是唯一的
- key不要使用随机值（随机值在下一次render时，会重新生成一个数字）
- 使用index作为key值，对性能没有优化

React判断key值的流程：

![img](https://i-blog.csdnimg.cn/blog_migrate/bbf5f9ab2308855b5fb24bfe0da3c5c7.png)

### Diff算法

diff算法通过更高效的对比新旧Virtual Dom，找出真正的DOM变化之处

传统diff算法循环递归对DOM树节点依次处理，算法复杂度达到O(n^3)，react做了优化，达到O(n)

react diff算法主要遵从三个层级的策略：

- tree层级
- component层级
- element层级

#### tree层级

DOM节点跨层级不做优化，只对相同层级的节点进行比较，只有删除、创建操作，没有移动操作

![img](https://i-blog.csdnimg.cn/blog_migrate/0d60e8ddef2ea340bc92a40dfe500021.png)

react发现新树中，R节点下没有了A，就直接删除A，在D节点下创建A以及下属节点

#### Component层级

如果是同一类的组件，则继续往下diff运算，如果不是一个类的组件，直接删除这个组建下的所有子节点，创建新的

![这里写图片描述](https://i-blog.csdnimg.cn/blog_migrate/eff8c9003e7415532d2291e82aa4537a.png)



当Component D换成 G后，即使两者结构非常相似，也会将D删除再重新创建

#### element层级

对于比较同一级的节点，每个节点在对应层级用唯一的key做标识

提供三重节点操作：INSERT_MARKUP（插入），MOVE_EXISTING（移动）和REMOVE_NODE（删除）

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4000ab69f5d343c49d71747f793debb6~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

通过key可以发现新旧集合中的节点都是相同的节点，因此无需进行节点删除和创建，只需要将旧集合中的节点位置进行移动，更新为新集合中节点的位置

如果当前节点再新集合中的位置比老集合中的位置靠前，不影响后续节点操作，这时候这个节点不用动

| index | 节点 | oldIndex | maxIndex | 操作                                                      |
| ----- | ---- | -------- | -------- | --------------------------------------------------------- |
| 0     | B    | 1        | 0        | oldIndex(1)>maxIndex(0)，maxIndex=oldIndex，maxIndex变为1 |
| 1     | A    | 0        | 1        | OldIndex(0)<maxIndex(1)，节点A移动到index（1）的位置      |
| 2     | D    | 3        | 1        | oldIndex(3)>maxIndex(1)，maxIndex变为oldIndex的值(3)      |
| 3     | C    | 2        | 3        | oldIndex(2)<maxIndex(3)，节点C移动至index(3)的位置        |

ABCD节点比较完成后，diff过程还没完，还会整体遍历老集合中的节点，如果有没用到的节点，直接删除

#### 注意事项

对于简单列表渲染而言，不使用`key`比使用`key`的性能好，例如： 将一个[1,2,3,4,5]，渲染成如下的样子：

```jsx
<div>1</div>
<div>2</div>
<div>3</div>
<div>4</div>
<div>5</div>
```

后续更改成[1,3,2,5,4]，使用`key`与不使用`key`作用如下：

```jsx
1.加key
<div key='1'>1</div>             <div key='1'>1</div>     
<div key='2'>2</div>             <div key='3'>3</div>  
<div key='3'>3</div>  ========>  <div key='2'>2</div>  
<div key='4'>4</div>             <div key='5'>5</div>  
<div key='5'>5</div>             <div key='4'>4</div>  
操作：节点2移动至下标为2的位置，节点4移动至下标为4的位置。

2.不加key
<div>1</div>             <div>1</div>     
<div>2</div>             <div>3</div>  
<div>3</div>  ========>  <div>2</div>  
<div>4</div>             <div>5</div>  
<div>5</div>             <div>4</div>  
操作：修改第1个到第5个节点的innerText
```

由于`dom`节点的移动操作开销是比较昂贵的，没有`key`的情况下要比有`key`的性能更好

### React Hooks

React 16.8新增hooks，让你在不编写class的情况下使用state以及其他的react特性

hooks解决的问题：

- 难以重用和共享组建中的与状态相关的逻辑
- 逻辑复杂的组件难以开发维护，当我们的组件需要处理多个互不相关的local state时，每个生命周期函数中可能会包含着各种互补相关的逻辑在里面
- 类组件中的this增加学习成本，类组件在基于现有工具的优化上存在些许问题
- 由于业务变动，函数组件不得不改为类组件等等

以前，函数组件也被称为无状态组件，只负责渲染的一些工作

现在函数组建也可以是有状态的组件，内部也可以维护自身的状态以及做一些逻辑方面的处理

#### useState 略

#### useEffect

相当于componentDidMount，componentDidUpdate，componentWillUnmount这三个生命周期函数的组合

#### useReducer

```jsx
import React, { useReducer } from 'react';

//初始狀態設定
const initialState = {
  count: 0,
  text: 'Initial text',
  isActive: false,
  data: [],
  color: '#3498db',
};

//所有狀態的操作handler集成，可以發現是使用switch來達成的
//當然，你也可以用if else，但我不會這麼做，做一次你就知道原因了。
//這邊入參的state跟action是必寫的內容，
//useReducer會自動把需要用到的資料丟state跟action，我們在這邊只要保留這兩個入口就好了。
const reducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'TOGGLE_ACTIVE':
      return { ...state, isActive: !state.isActive };
    case 'ADD_DATA':
      return { ...state, data: [...state.data, `New Item ${state.data.length + 1}`] };
    case 'CHANGE_TEXT':
      return { ...state, text: action.payload };
    case 'CHANGE_COLOR':
      return { ...state, color: state.color === '#3498db' ? '#e74c3c' : '#3498db' };
    default:
      return state;
  }
};

const MyComponent = () => {
  //只需要調用一次useReducer就可以了！ 
  //把上方作為操作機制的Reducer跟初始值initialState入參  
  //使用解構賦值將Hook return的兩個東西：當前值state，跟操作機制dispatch抓出。
  //有關dispatch跟state的用法見下方
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>Count: {state.count}</p>
        {/*dispatch的使用方式是以一個object入參，這邊使用的type key值是對應到reducer入參的action內容，
            reducer中的switch的參照點是action.type，當然你也可以改成其他的，但就是要對應到reducer的switch寫法*/}
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>Increment</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>Decrement</button>
        {/*state直接對應到initialState的key值，但當前值會隨著使用dispatch變化*/}
      <p>Text: {state.text}</p>
      <input
        type="text"
        value={state.text}
        onChange={(e) => dispatch({ type: 'CHANGE_TEXT', payload: e.target.value })}
      />

      <p>Active: {state.isActive ? 'Yes' : 'No'}</p>
      <button onClick={() => dispatch({ type: 'TOGGLE_ACTIVE' })}>Toggle Active</button>

      <ul>
        {state.data.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
      <button onClick={() => dispatch({ type: 'ADD_DATA' })}>Add Data</button>

      <div style={{ backgroundColor: state.color, width: '100px', height: '100px' }}>
        {/* Your content here */}
      </div>
      <button onClick={() => dispatch({ type: 'CHANGE_COLOR' })}>Change Color</button>
    </div>
  );
};

export default MyComponent;
```

### 提高组件渲染效率

#### 使用React.memo

`React.memo` 是一个高阶组件，它会记忆组件的渲染结果，仅在 props 发生变化时重新渲染：

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  /* 只在 props 改变时重新渲染 */
});
```

##### 自定义比较函数

```jsx
const MyComponent = React.memo(
  function MyComponent(props) {
    /* 渲染 */
  },
  (prevProps, nextProps) => {
    // 返回 true 表示跳过渲染，false 表示需要渲染
    return prevProps.value === nextProps.value;
  }
);
```

#### 合理使用useCallback

对于传递给子组件的回调函数，使用useCallback避免每次渲染都创建新函数

```jsx
const handleClick = useCallback(() => {
  // 处理点击
}, [dependencies]); // 依赖项变化时才会重新创建
```

#### 合理使用useMemo

对于昂贵的计算，使用useMemo缓存结果

```jsx
const computedValue = useMemo(() => {
  return expensiveCalculation(props.value);
}, [props.value]); // 仅在 props.value 变化时重新计算
```

#### 状态提升与状态分割

将状态提升到合理的位置，避免状态变化导致大范围重新渲染：

```jsx
// 不好的做法 - 状态在父组件导致所有子组件重新渲染
function Parent() {
  const [state, setState] = useState();
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildC data={state} />
    </>
  );
}

// 好的做法 - 将状态下移到真正需要它的组件
function Parent() {
  return (
    <>
      <ChildA />
      <ChildB />
      <ChildCWrapper />
    </>
  );
}
```

#### 合理使用context

使用context时，避免将频繁变化的值放入Context中，导致消费者重新渲染

```jsx
// 创建分离的 Context
const SettingsContext = React.createContext();
const UserContext = React.createContext();

function App() {
  const [settings, setSettings] = useState({});
  const [user, setUser] = useState({});
  
  return (
    <SettingsContext.Provider value={settings}>
      <UserContext.Provider value={user}>
        {/* 组件树 */}
      </UserContext.Provider>
    </SettingsContext.Provider>
  );
}
```

#### 避免在渲染函数中创建新对象/数组

```jsx
// 不好的做法 - 每次渲染都创建新对象
function Component() {
  const style = { color: 'red' }; // 每次渲染都是新对象
  return <div style={style} />;
}

// 好的做法
const staticStyle = { color: 'red' };

function Component() {
  return <div style={staticStyle} />;
}
```

#### 使用useReducer替代多个useState

当有多个相关联的状态时，useReducer可以减少状态更新次数：

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
// 而不是多个 useState
```

#### 使用React DevTools分析渲染

使用 React Developer Tools 的 "Highlight updates" 功能识别不必要的渲染：

1. 打开浏览器开发者工具
2. 切换到 React 面板
3. 勾选 "Highlight updates when components render"
4. 观察哪些组件在不必要时重新渲染

#### 避免使用内联函数

如果组件的事件绑定内联函数，每次调用render都会创建一个新的函数实例

```jsx
import React from "react";
export default class InlineFunctionComponent extends React.Component {
  render() {
    return (
      <div>
        <h1>Welcome Guest</h1>
        <input
          type="button"
          onClick={(e) => {
            this.setState({ inputValue: e.target.value });
          }}
          value="Click For Inline Function"
        />
      </div>
    );
  }
}

```

在组建内部创建一个useCallback，或者useMemouizedFn，避免每次调用创建单独的函数实例

#### 使用Fragments避免额外标记

react每个组件都要具有单个父标签，所以经常会在组建顶部添加额外标签div，如果这个标签除了充当父组件之外没有任何用途，就可以使用fragment(<></>)，这样不会向组件引入任何额外标记

#### 使用Immutable

[面试官：说说你对immutable的理解？如何应用在react项目中？ | web前端面试 - 面试官系列](https://vue3js.cn/interview/React/immutable.html#二、如何使用)

做react性能优化的时候，为了避免重复渲染，会在shouldComponentUpdate()中对比，返回true就执行render方法

Immutable通过is方法就可以完成对比，无需通过深度比较的方式做比较

#### 懒加载组建

React中使用Suspense和lazy组件实现代码拆分功能

```jsx
const johanComponent = React.lazy(
  () =>
    import(
      /* webpackChunkName: "johanC
 omponent" */ "./myAwesome.component"
    )
);
export const johanAsyncComponent = (props) => (
  <React.Suspense fallback={<Spinner />}>
    <johanComponent {...props} />
  </React.Suspense>
);
```

#### 服务端渲染

在node服务端调用react的renderToString方法，将根组件渲染成字符串，输出到响应中

```jsx
import { renderToString } from "react-dom/server";
import MyPage from "./MyPage";
app.get("/", (req, res) => {
  res.write("<!DOCTYPE html><html><head><title>My Page</title></head><body
>");
  res.write("<div id='content'>");  
  res.write(renderToString(<MyPage/>));
  res.write("</div></body></html>");
  res.end();
});
```

客户端使用render方法来生成HTML

```jsx
import ReactDOM from 'react-dom';
import MyPage from "./MyPage";
ReactDOM.render(<MyPage />, document.getElementById('app'));
```

### react-router

react-router等前端路由的原理大致相同，可以实现无刷新条件下切换显示不同的页面

路由的本质就是页面URL发生改变的时候，页面的显示结果可以根据URL的变化而变化，但是页面不会刷新，由此实现单页应用（SPA）

react-router包括这几个包：

- react-router：实现了路由的核心功能
- react-router-dom：基于react-router，加入了在浏览器运行环境下的一些功能
- react-router-native：基于react-router，加入了react-native运行环境下的一些功能
- react-router-config：用于配置静态路由的工具库

#### react-router-dom常用API

- BrowserRouter、HashRouter
- Route
- Link、NavLink
- switch
- redirect

##### BrowerRouter、HashRouter

Router中包含了对路径改变的监听，并且会将相应的路径传递给子组件

BrowserRouter是history模式，HashRouter使用hash参数

使用两者作为最顶层组件包裹其他组件

```jsx
// 1.import { BrowserRouter as Router } from "react-router-dom";
 // 2.import { HashRouter as Router } from "react-router-dom";
 
import React from 'react';
 import {
  BrowserRouter as Router,
  // HashRouter as Router  
  Switch,
  Route,
 } from "react-router-dom";
 import Home from './pages/Home';
 import Login from './pages/Login';
 import Backend from './pages/Backend';
 import Admin from './pages/Admin';
 
 
function App() {
  return (
    <Router>
        <Route path="/login" component={Login}/>
        <Route path="/backend" component={Backend}/>
        <Route path="/admin" component={Admin}/>
        <Route path="/" component={Home}/>
    </Router>
  );
 }
 
export default App;
```

###### 实现原理

路由描述了URL与UI之间的映射关系，这种映射是单向的，即URL变化引起UI更新（无需刷新页面）

对于HashRouter，改变hash值并不会导致浏览器向服务端发送请求，浏览器不发出请求，就不会刷新页面

hash值改变，触发全局window对象上的hashChange事件，所以hash模式就是利用hashchange事件监听URL的变化，从而进行DOM操作模拟页面跳转

###### HashRouter

HashRouter包裹了整个应用，通过window.addEventListener('hashChange', callback)监听hash值的变化，并传递给其嵌套的组建

然后通过context将location数据往后代组件传递，如下：

```jsx
 import React, { Component } from 'react';
 import { Provider } from './context'
 // 该组建下的Api提供给子组件使用
 class HashRouter extends Component {
  constructor() {
    super()
    this.state = {
      location: {
        pathname: window.location.hash.slice(1) || '/'
      }
    }
  }
  // url路径变化 改变location
  componentDidMount() {
    window.location.hash = window.location.hash || '/'
    window.addEventListener('hashchange', () => {
      this.setState({
        location: {
          ...this.state.location,
          pathname: window.location.hash.slice(1) || '/'
        }
      }, () => console.log(this.state.location))
    })
  }
  render() {
    let value = {
      location: this.state.location
    }
    return (
      <Provider value={value}>
        {
          this.props.children
        }
      </Provider>
    );
  }
 }
 
export default HashRouter;
```

###### BrowserRouter

通过props传进来的path与context传进来的pathname进行匹配，然后决定是否执行渲染组件

```jsx
import React, { Component } from "react";
import { Consumer } from "./context";
const { pathToRegexp } = require("path-to-regexp");
class Route extends Component {
  render() {
    return (
      <Consumer>
        {(state) => {
          console.log(state);
          let { path, component: Component } = this.props;
          let pathname = state.location.pathname;
          let reg = pathToRegexp(path, [], { end: false });
          // path pathname
          if (pathname.match(reg)) {
            return <Component></Component>;
          }
          return null;
        }}
      </Consumer>
    );
  }
}
export default Route;

```

##### Route

Route用于路径的匹配，然后进行组件的渲染，对应属性如下：

- path属性：用于设置匹配到的路径
- component属性：设置匹配到的路径后，渲染的组件
- render属性：设置匹配到路径后，渲染的内容
- exact属性：开启精确匹配，只有精确匹配到完全一致的路径，才会渲染对应的组件

```jsx
import { BrowserRouter as Router, Route } from "react-router-dom";

export default function App() {
  return (
    <Router>
      <main>
        <nav>
          <ul>
            <li>
              <a href="/">Home</a>
            </li>
            <li>
              <a href="/about">About</a>
            </li>
            <li>
              <a href="/contact">Contact</a>
            </li>
          </ul>
        </nav>
        <Route path="/" render={() => <h1>Welcome!</h1>} />
      </main>
    </Router>
  );
}

```

##### Link, NavLink

通常的路由跳转是使用Link组件，最终会被渲染成a元素，其中to代替a中的href

NavLink是在Link基础上增加了一些样式属性，例如组件被选中时，发生样式变化，则可以设置NavLink

的一些属性：

- activeStyle：活跃时（匹配时）的样式
- activeClassName：活跃时添加的class

```jsx
<NavLink to="/" exact activeStyle={{color: "red"}}>首页</NavLink>
 <NavLink to="/about" activeStyle={{color: "red"}}>关于</NavLink>
 <NavLink to="/profile" activeStyle={{color: "red"}}>我的</NavLink>
```

如果需要js实现页面的跳转，可以通过Route作为顶层组件包裹其他组件后，让页面组件接收到一些路由相关的东西，比如props.history

```jsx
 const Contact = ({ history }) => (
   <Fragment>
     <h1>Contact</h1>
     <button onClick={() => history.push("/")}>Go to home</button>
     <FakeText />
   </Fragment>
 )
```

props中的history对象还包含一些方便的方法：goBack，goForward，push等

##### redirect

用于路由重定向，这个组件出现的时候，就会执行跳转到对应的to路径中

```jsx
const About = ({
  match: {
    params: { name },
  },
}) => (
  // props.match.params.name
  <Fragment>
    {name !== "tom" ? <Redirect to="/" /> : null}
    <h1>About {name}</h1>
    <FakeText />
  </Fragment>
);

```

上述组件当接收到的路由参数name不等于tom的时候，将会自动重定向到首页

##### switch

适用于当匹配到第一个组件的时候，后面的组件就不应该继续匹配

```jsx
<Switch>
 <Route exact path="/" component={Home} />
 <Route path="/about" component={About} />
 <Route path="/profile" component={Profile} />
 <Route path="/:userid" component={User} />
 <Route component={NoMatch} />
</Switch>
```

##### react-router提供的hooks

- useHistory
- useParams
- useLocation

###### useHistory

useHistory可以让组件内部直接访问History，无需通过props获取

```jsx
import { useHistory } from "react-router-dom";
const Contact = () => {
  const history = useHistory();
  return (
    <Fragment>
      <h1>Contact</h1>
      <button onClick={() => history.push("/")}>Go to home</button>
    </Fragment>
  );
};

```

###### useParams

```jsx
const About = () => {
  const { name } = useParams();
  return (
    // props.match.params.name
    <Fragment>
      {name !== "John Doe" ? <Redirect to="/" /> : null}
      <h1>About {name}</h1>
      <Route component={Contact} />
    </Fragment>
  );
};
```

###### useLocation

useLocation返回当前URL的location对象

```jsx
import { useLocation } from "react-router-dom";
const Contact = () => {
  const { pathname } = useLocation();
  return (
    <Fragment>
      <h1>Contact</h1>
      <p>Current URL: {pathname}</p>
    </Fragment>
  );
};

```

#### 参数传递

路由传递参数主要分成了三种形式

- 动态路由
- search传递参数
- to传入对象

##### 动态路由

指路有种的路径并不会固定

例如将path在Router匹配时写成/detail/:id，那么/detail/abc，/detail/123都会匹配到该Route

```jsx
 <NavLink to="/detail/abc123"> </NavLink>
 <Switch>
   // ... 其他Route
   <Route path="/detail/:id" component={Detail}/>
   <Route component={NoMatch} />
 </Switch>
```

获取参数的方式如下：

```jsx
console.log(props.match.params.xxx)
```

##### search传递参数

获取跳转路径中的query参数：

```jsx
console.log(props.location.search)
```

##### to传入对象

```jsx
<NavLink to={{
  pathname: "/detail2", 
  query: {name: "kobe", age: 30},
  state: {height: 1.98, address: " "},
  search: "?apikey=123"
}}>
  2
</NavLink>
```

获取参数方式：

```jsx
console.log(props.location)
```

### Redux

react-redux将组件分成：

- 容器组件，存在逻辑处理
- UI组件，只负责显示和交互，内部不处理逻辑，状态由外部控制

通过将整个应用状态保存在store中，组件可以派发dispatch行为action给store，其他组件通过订阅store中的状态state来更新自身的视图

react-redux分成了两大核心：

- provider
- connection

#### Provider

redux中存在一个store用于存储state，如果将这个store存放在顶层元素中，其他组件都被包裹在顶层元素之下

那么所有的组件都能受到redux的控制，都能获取redux中的数据

```jsx
<Provider store={store}>
	<APP />
</Provider>
```

#### Connection(用于类组件)

connection方法将store上的getState和dispatch包装城组件的props

```jsx
import { connect } from "react-redux";
connect(mapStateToProps, mapDispatchToProps)(MyComponent)
```

可以传递两个参数

- mapStateToProps
- mapDispatchToProps

##### mapStateToProps

把redux中的数据映射到react中的props中去

```jsx
const mapStateToProps = (state) => {
  return {
    // prop : state.xxx  | 将state中的某个数据映射到props中
    foo: state.bar,
  };
};

```

组建内部就能够通过props获取到store中的数据

```jsx
class Foo extends Component {
    constructor(props){
        super(props);
    }
    render(){
        return(
         // 这样子渲染的其实就是state.bar的数据了
            <div>this.props.foo</div>
        )
    }
 }
 Foo = connect()(Foo)
 export default Foo
```

##### mapDispatchToProps

将redux中的dispatch映射到组件内部的props中

```jsx
const mapDispatchToProps = (dispatch) => {
  // 默认传递的函数就是dispatch
  return {
    onClick: () => {
      dispatch({
        type: "increatment",
      });
    },
  };
};
```

```jsx
class Foo extends Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <button onClick={this.props.onClick}>点击increase</button>;
  }
}
Foo = connect()(Foo);
export default Foo;

```



#### 新的Redux使用

redux基本概念：

- store：整个应用的状态容器，一个应用只有一个
- Action：描述发生了什么的对象，如{type: 'INCREMENT'}
- reducer: 纯函数，接受当前state和action，返回新的state
- Dispatch：发送action到store的方法

安装包：

```bash
npm install redux react-redux @reduxjs/toolkit
```

- redux：Redux核心库
- react-redux：链接React和Redux的官方绑定库
- @reduxjs/toolkit：Redux的官方工具集，简化Redux使用

##### 创建Redux Store

```jsx
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer // 计数器模块
  }
});
```

##### 将store提供给React应用

```jsx
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import App from './App';
import { store } from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

##### 使用Redux Toolkit创建counterSlice

```jsx
// counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

export const counterSlice = createSlice({
  name: 'counter', // slice名称
  initialState: {  // 初始状态
    value: 0
  },
  reducers: {      // reducer函数集合
    increment: state => {
      state.value += 1; // 可以直接"修改"state（Immer内部处理为不可变更新）
    },
    decrement: state => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload; // action.payload 是传入的参数
    }
  }
});

// 导出自动生成的action创建函数
export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// 导出reducer
export default counterSlice.reducer;
```

##### 在组件中使用Redux

###### 函数组件写法

```jsx
// Counter.js
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './counterSlice';

function Counter() {
  // 从store中获取counter状态
  const count = useSelector(state => state.counter.value);
  
  // 获取dispatch函数
  const dispatch = useDispatch();

  return (
    <div>
      <button onClick={() => dispatch(decrement())}>-</button>
      <span>{count}</span>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}

export default Counter;
```

##### 使用createAsyncThunk进行异步处理

```jsx
// counterSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 创建异步thunk
export const fetchCount = createAsyncThunk(
  'counter/fetchCount',
  async (amount) => {
    const response = await fetch('/api/counter?amount=' + amount);
    const data = await response.json();
    return data.value; // 会成为action.payload
  }
);

// 在slice中添加extraReducers处理异步状态
extraReducers: (builder) => {
  builder
    .addCase(fetchCount.pending, (state) => {
      state.status = 'loading';
    })
    .addCase(fetchCount.fulfilled, (state, action) => {
      state.status = 'succeeded';
      state.value += action.payload;
    })
    .addCase(fetchCount.rejected, (state) => {
      state.status = 'failed';
    });
}
```

## Fiber的详细介绍

在React15中，组建的渲染和更新是通过递归调用实现的：

- 同步递归：从根组件开始，递归遍历整个组件树，生成虚拟DOM（VDOM），然后一次性提交到真实DOM
- 不可中断：一旦开始渲染，必须一次性完成，无法中途暂停
- 阻塞主线程：如果组件树很深或者计算复杂（如大型列表、动画），会导致主线程长时间占用，用户交互（点击、输入）无法响应，出现卡顿

典型问题场景：

- 渲染一个包含1000个列表项的组件时，UI会冻结直到全部渲染完成
- 动画掉帧（如拖拽元素时卡顿）

Fiber时React16重写的协调算法，核心改进：

- 将渲染任务拆分为可中断的单元（Fiber节点）
  - 每个Fiber对应一个组件或DOM节点，包含其状态、props和渲染上下文
  - 渲染过程被分解为多个小任务，而不是一次性递归完成
- 优先级调度与时间切片（Time Slicing）
  - 高优先级任务（如用户交互）可以中断低优先级任务（如渲染）
  - 利用浏览器的requestIdleCallback在空闲时间执行任务
- 增量渲染
  - 部分渲染结果可以先提交到DOM，无需等待整个树完成，剩余部分异步加载

**Fiber 使用 链表结构 替代递归，通过 `child`、`sibling`、`return` 指针实现深度优先遍历，支持中断和恢复。**

**Fiber引入错误边界（Error Boundaries）,捕获子树错误并降级显示**

Fiber工作流程（渲染阶段和提交阶段）

阶段1：Render Phase（可中断）

- 目标：生成Fiber树并标记变更（不直接修改DOM）
- 过程：
  1. React从根节点开始，深度优先遍历组件树，为每个节点执行beginWork（计算新状态、调用render）
  2. 如果时间片用完，暂停并记录当前进度，下次从断点继续
  3. 遍历完成后，得到一颗带有effectTag(变更标记)的Fiber树

阶段2：Commit Phase（不可中断）

- 目标：将变更一次性提交到DOM
- 过程：
  1. 遍历Fiber树，根据effectTag执行DOM操作（插入、更新、删除）
  2. 调用生命周期方法（如componentDidMount、useLayoutEffect）
  3. 由于此阶段必须同步完成，因此不可中断

## React18，19与早期版本的变化

### React18

1. 并发渲染

   - 引入并发模式的底层架构，允许React中断渲染过程以优先处理高优先级任务（如用户交互），提升性能
   - 核心API：ReactDOM.createRoot替代ReactDOM.render，默认启用并发特征

   在react17以及之前的版本中，渲染是同步且不可中断的。这意味着：

   - 一旦React开始渲染一个组件树，就必须一次性完成，无法中途暂停。
   - 如果渲染过程耗时较长（如计算密集型任务或者大数据量渲染），会导致主线程阻塞，用户交互（如点击、输入）无法及时响应，造成卡顿

2. 自动批处理

   - React17及之前，批处理仅限于浏览器事件(如onclick)。react18扩展了批处理范围，包括异步操作（setTimeout, promise）和原生事件处理程序

3. 新的Hooks

   - useId: 生成唯一ID（用于SSR场景）
   - useTransition：标记非紧急的状态更新（允许中断渲染）
   - useDefferredValue：延迟渲染某些非关键部分（如输入框的联想列表）
   - useSyncExternalStore：专为状态库（Redux）优化，支持并发渲染
   - useUnsertionEffect：适用于CSS-in-JS库（如styled-components）

4. SSR改进

   - 流式SSR：服务器可以逐步发送HTML，客户端并行处理
   - 选择性Hydration：优先对用户可见部分进行hydration，减少交互延迟

5. 严格模式增强

   - 在开发模式下，组件会多次故意挂载/卸载，以暴露潜在问题（如Effect清理不彻底）

6. 过渡API（Transition）

   - 通过startTransition或useTransition区分紧急与非紧急更新，避免界面卡顿（例如搜索框输入时的联想词加载）

### React19

1. Actions：异步操作的革命性改进

   React19引入了Actions，通过支持异步函数来管理数据变更、加载状态、错误处理以及乐观更新，使复杂的逻辑的处理变得更加简单

   - 自动管理Pending状态：使用useActionState和useFormStatus等新hook处理表单的加载状态
   - 乐观更新支持：通过useOptimistic实现实时数据更新
   - 只能错误处理：集成错误边界，简化错误回退逻辑

   ```tsx
   function ChangeName({ currentName }) {
     const [error, submitAction, isPending] = useActionState(async (prev, formData) => {
       const error = await updateName(formData.get("name"));
       if (error) return error;
       return null;
     });
   
     return (
       <form action={submitAction}>
         <input type="text" name="name" defaultValue={currentName} />
         <button type="submit" disabled={isPending}>Update</button>
         {error && <p>{error}</p>}
       </form>
     );
   }
   ```

2. 原生支持Document MetaData

   原生支持<title>, <meta>, <link>等文档元数据标签，这些标签可直接在组件中声明，React会自动将他们提升至<head>，确保与服务端渲染和客户端渲染兼容，可以简化SEO和元数据管理逻辑，不需要像以前一样手动插入标签了

   ```tsx
   function BlogPost({ post }) {
     return (
       <article>
         <h1>{post.title}</h1>
         <title>{post.title}</title>
         <meta name="author" content={post.author} />
       </article>
     );
   }
   ```

3. 支持样式表优先级管理

   React19增强了样式表的加载管理，通过指定precedence属性，react可以动态调整样式表插入顺序，确保正确的样式覆盖

   ```tsx
   function Component() {
     return (
       <div>
         <link rel="stylesheet" href="styles.css" precedence="high" />
         <p>Styled Content</p>
       </div>
     );
   }
   ```

4. Server Components的稳定支持

   Server Components提供了全新的组件渲染模式，在服务器上提前渲染，减少了客户端的渲染负担，引入了相关的API和最佳实践

   - 支持在构建时或请求时生成组件
   - 无需引入额外的工具链，即可与现有React项目集成

5. 更好的错误展示系统

   改进了错误日志系统，减少了重复日志，添加了更详细的调试信息。例如，对于SSR和客户端渲染的问题，提供了差异化日志

   - 单一错误消息：减少日志冗余

     ![图片](https://s7.51cto.com/oss/202412/06/981368928c24de5963d8477250f0a52219d32a.webp)

   - 支持onCatchError和onUncatchError回调，增强了错误管理能力

6. 支持<Context>简写

   现在可以使用<Context>代替<Context.Provider>

   ```tsx
   const ThemeContext = createContext('');
   function App({children}) {
     return <ThemeContext value="dark">{children}</ThemeContext>;
   }
   ```

7. Async脚本和资源预加载支持

   为<script>标签添加了异步加载支持，同时优化了资源的预加载和预初始化功能

   - 异步脚本加载：允许在组件内声明脚本，并由React自动去重
   - 预加载API：通过preload和preinit指定浏览器提前加载的资源

   ```tsx
   import { preinit, preload } from 'react-dom';
   
   function MyComponent() {
     preinit('https://example.com/script.js', { as: 'script' });
     preload('https://example.com/font.woff', { as: 'font' });
   }
   ```

8. use API

   全新的use API，用于在渲染期间读取资源

   例如：读取Promise或context。这种模式允许条件调用，并与Suspense结合使用

   - 支持条件调用：突破了传统hook的调用限制
   - 与Suspense深度集成：自动管理异步状态，简化异步渲染逻辑

   ```tsx
   import { use } from 'react';
   
   function Comments({ commentsPromise }) {
     const comments = use(commentsPromise);
     return comments.map(comment => <p key={comment.id}>{comment}</p>);
   }
   ```

9. ref回调的清理功能

   为ref回调增加了清理函数支持，允许在组件卸载时自动执行清理逻辑：

   ```tsx
   <input ref={(ref) => {
     // ref 创建时的逻辑
     return () => {
       // ref 清理时的逻辑
     };
   }} />
   ```
