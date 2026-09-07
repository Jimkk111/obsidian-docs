---
title: React.js
source: https://www.yuque.com/mook-mvqcp/wc9wsu/wl5da0qg5pzhwogf
created: 2026-05-23
updated: 2026-05-23
tags:
  - React
---

**注意：文中的组件指函数式组件。**

### 一、组件

React中的函数式组件是纯函数，其内部只关注于如何计算JSX，输入相同的参数输出相同的模板。

因为state更新页面重新渲染，组件会重新被执行。

### 二、React中使用JSX

### 三、props

### 四、条件渲染与列表渲染

### 五、事件

### 六、组件的状态

state是 React 组件内部用于描述 **“当前状态”** 的普通 JavaScript 对象。

state是组件自己定义和管理的数据，反映了组件的状态。

通过setState方法修改state可以被React识别到组件的状态发生了改变从而重新渲染组件，如果通过this.state这种方式修改state，并不能实现这个功能。

#### 1、定义state

#### 2、使用state

#### 3、修改state

#### 4、深入理解useState和setState

函数式组件需要通过useState注册state和获取setState函数。

##### （1）state是组件状态的快照

**setSate内部执行的更新操作使用的state基于页面尚未真正更新前的状态。**

##### （2）useState与setter简化源码

```javascript

// 全局 Fiber 节点（当前正在渲染的组件）
let currentlyRenderingFiber = null;
// 当前 Fiber 的 hooks 链表
let currentHook = null;

// 简化的 useState
function useState(initialState) {
  // 获取当前 Fiber 节点对应的 hook 对象
  const hook = getOrCreateHook(currentlyRenderingFiber);
  
  // 如果是首次渲染，初始化 state
  if (hook.isMounted === false) {
    hook.state = typeof initialState === 'function' ? initialState() : initialState;
    hook.isMounted = true;
    hook.queue = []; // 更新队列
  }
  
  // 更新函数（即你调用的 setCount）
  const setState = (action) => {
    // 1. 将更新动作包装成更新对象，加入队列
    const update = { action };
    hook.queue.push(update);
    
    // 2. 触发一次更新调度
    scheduleUpdate(currentlyRenderingFiber);
  };
  
  return [hook.state, setState];
}

// 获取或创建当前 Fiber 上的 hook（链表）
function getOrCreateHook(fiber) {
  if (!fiber.hooks) fiber.hooks = [];
  if (!currentHook) currentHook = fiber.hooks[0];
  // 简化：只处理第一个 hook
  if (!fiber.hooks[0]) fiber.hooks[0] = { queue: [], isMounted: false };
  return fiber.hooks[0];
}

// 调度更新（简化版：微任务批量处理）
let isBatching = false;
let dirtyFibers = new Set();

function scheduleUpdate(fiber) {
  dirtyFibers.add(fiber);
  if (!isBatching) {
    isBatching = true;
    // 微任务中批量处理所有脏 Fiber
    Promise.resolve().then(() => {
      for (let f of dirtyFibers) {
        performWorkOnFiber(f);
      }
      dirtyFibers.clear();
      isBatching = false;
    });
  }
}

// 执行 Fiber 的更新（重新渲染组件）
function performWorkOnFiber(fiber) {
  // 重新执行函数组件，得到新的虚拟 DOM
  const component = fiber.type;      // 函数组件本身
  const prevHook = currentHook;
  currentHook = fiber.hooks[0];      // 设置当前 hook
  
  // 关键：处理更新队列，计算出最新 state
  let newState = fiber.hooks[0].state;
  while (fiber.hooks[0].queue.length) {
    const update = fiber.hooks[0].queue.shift();
    const action = update.action;
    if (typeof action === 'function') {
      newState = action(newState);   // 函数式更新
    } else {
      newState = action;              // 直接赋值（替换）
    }
  }
  fiber.hooks[0].state = newState;
  
  // 重新执行组件函数，得到新的 React 元素（虚拟 DOM）
  const newVNode = component();      // 这里会再次调用 useState，拿到最新 state
  
  // 然后进行 diff 并更新真实 DOM（省略）
  updateRealDOM(newVNode);
  
  currentHook = prevHook;
}

```

##### （3）调用setState的过程

#### 5、修改state中的对象或者数组

在 React 中，状态必须被视为不可变数据（immutable）。当需要更新 state 中的对象或数组时，不能直接修改原对象/数组，而是必须创建一个新的副本，然后修改这个副本，最后用 setState（类组件）或 setXXX（函数组件）去更新。

类组件的 setState 会自动浅合并顶层属性，但嵌套对象和数组仍然需要手动创建副本

##### （1）修改对象

```jsx

const [user,setUser] = useState({name:"Mok",age:21})

setUser({
  name:"Mok",
  age:"18"
})
setUser(prev => {
  ...prev, // 展开旧state
    age:17   // 覆盖旧的prev.age
})

// 对于嵌套的对象，每一层都需要展开
const [state, setState] = useState({
  user: { name: 'Alice', address: { city: 'Beijing', zip: '100000' } }
});

// 更新 city
setState(prev => ({
  ...prev,
  user: {
    ...prev.user,
    address: {
      ...prev.user.address,
      city: 'Shanghai'
    }
  }
}));

```

##### （2）修改数组

规则与修改对象类似，不过要注意哪些方法会返回新数组，哪些是在原数组上操作，在setState的action中用对方法。

##### （3）Immer简化深层更新

Immer库可以简化深层更新操作。

```jsx

import { produce } from 'immer';

setState(produce(draft => {
  draft.user.address.city = 'Shanghai';
}));

```

### 七、状态管理

#### （一）控制UI的状态变更

有些UI因交互的进行发生变更，为了方便实现对变更的控制，可以看成是状态的变更，利用React声明式地利用state响应UI的变更。

实现声明式地控制UI，可以按照以下步骤进行：

#### （二）设计好的state结构

有以下原则用于设计好的state结构

#### （三）控制状态的保留或重置

React中React元素构成React元素树（亦称渲染树、组件树，虚拟DOM树），组件实例、状态、副作用、更新队列、优先级等用fiber树管理，fiber树是独立于虚拟DOM树的数据结构，一个组件实例对应一个fiber结点。

重新渲染组件时，组件的state是否被保留取决于它在父元素中的位置或者自身的key，原因是React会对比新React元素树和fiber树，在同一个父节点下，如果两次渲染中同一个位置上的元素类型相同（并且 `key` 也相同），React 就会复用该位置原有的 Fiber 节点，而不是新建一个。

所以state被保留还是重置取决于重新渲染时父元素的这个位置上的元素是否是同一类以及Key是否改变了，无关从什么分支返回的JSX。

#### （四）利用Reducer整合复杂的状态逻辑

#### （五）“状态提升”实现组件共享状态

#### （六）使用Context深层传递参数

#### （七）混合使用Reducer、Contenxt以管理应用不断增长的状态

### 八、脱围机制

React提供了一些“后门”，用于处理组件外部世界或执行非标准操作。

#### （一）ref

ref是一种可以保存组件状态但发生改变时不会导致页面重新渲染的数据。

##### 1.如何定义和使用ref

##### 2.ref访问DOM

定义好ref对象后，把它传给需要目标DOM的JSX标签的ref属性，这样当 React  创建一个 DOM 节点时，React 会把对该节点的引用放入这个ref对象。

###### （1）如何使用 ref 回调管理 ref 列表

###### （2）使用命令句柄useImperativeHandle暴露一部分API

##### 3.使用ref的注意事项

###### （1）**将 ref 视为脱围机制**

###### （2）**不要在渲染过程中读取或写入**`ref.current`

#### （二）Effect

useEffect(callback,Dependencies)是一个Hooks，用于组件和外部系统同步。

传递给useEffect的callback在页面渲染完成后执行。

dependencies是一个数组，如果是空数组，表示callback只在页面首次渲染完成后执行；如果声明了具体的依赖项，则callback会在页面首次渲染完成后以及由于这些依赖发生改变后导致页面重新渲染后执行。

如果不传入dependencies，那么callback会在每次页面渲染后都会执行。

**用途：**useEffect经常用于处理请求数据、操作DOM的代码。

### 九、API

#### 1、Hooks

（1）useCallBack

缓存函数引用，避免因为回调函数的引用发生改变导致子组件做无必要的重新渲染。

（2）useEffectEvent

**2、内置组件**

(1)<Suspence> </ Suspence>

这个组件用于给其包裹的组件在尚未加载完成前提高后备方案（fallback）

其属性fallback可以指定后备内容。

#### 3、DOM元素

#### 4、React顶层API

### 十、拓展与生态

#### 1、状态管理（Redux）

#### 2、路由（React Router）

#### 3、移动端（React Native）
