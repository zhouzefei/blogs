Redux

> redux我们会用它做react的数据管理，在开始源码阅读之前，先了解下redux内部大致的内容

![Alt text](./images/redux/1.png)

<pre>
从图中我们大致了解：redux核心方法createStore 接受三个参数 reducer,  preloadedState,  enhancer，经过一些处理生成一些常用的api，如 dispatch,  subscribe,  getState,  replaceReducer。
</pre>
<br/>

### 来个简单的demo🌰：


```javascript

//一个基础的redux使用方式
const order = (state,action) => {
    switch (action.type) {
        case xxx: return Object.assign( {}, state, { otherInfo: { name: 'abc' } });
    }
};

const reducers = combineReducers({
  order,
});

const store = (history) => { 
    let middlewares = [ thunk ];
    // 结合上图看createStore的入参
    return createStore(reducers, applyMiddleware(...middlewares)); 
};

```

### 源码解析
> createStore 着手
```javascript
export default function createStore(reducer, preloadedState, enhancer) {
    ......
    return {
        dispatch,
        subscribe,
        getState,
        replaceReducer,
        [$$observable]: observable
    }
}
```

- 参数一：reducer
<pre>
从实例中， reducer = combineReducers({ sign, order, })。
combineReducers 方法定义在 combineReducers.js
</pre>
核心代码如下：
```javascript
export default function combineReducers(reducers) {
    // 第一部分 整合多个reducer，因为createStore只能接受一个reducer
    // 检查 reducer的正确性 必须是 function
    const reducerKeys = Object.keys(reducers)
    const finalReducers = {}
    for (let i = 0; i < reducerKeys.length; i++) {
        const key = reducerKeys[i]
        ......//测试环境输出reducer不合理日志
        if (typeof reducers[key] === 'function') {
            finalReducers[key] = reducers[key]
        }
    }
    const finalReducerKeys = Object.keys(finalReducers)
    ......
    // 第二部分 比较state是否发生变化
    return function combination(state = {}, action) {
        ......  
        // 标志位 state是否发生变化
        let hasChanged = false
        // 变化后新的值
        const nextState = {}
        // 遍历每一个reducer
        for (let i = 0; i < finalReducerKeys.length; i++) {
            const key = finalReducerKeys[i]
            const reducer = finalReducers[key]
            const previousStateForKey = state[key] // 旧的值
            const nextStateForKey = reducer(previousStateForKey, action) // 新的值
            // 如果没有返回则报错
            if (typeof nextStateForKey === 'undefined') {
                const errorMessage = getUndefinedStateErrorMessage(key, action)
                throw new Error(errorMessage)
            }
            nextState[key] = nextStateForKey
            hasChanged = hasChanged || nextStateForKey !== previousStateForKey 
        }
        return hasChanged ? nextState : state
    }
}
```
#### 重点！！！「为什么不能修改state本身？」：
```javascript
//实例：
reducer.order = (state,action) => { 
    switch (action.type) {  
        // 这里为什么要拷贝一份 而不能是 
        // case xxx: return Object.assign( state, { otherInfo: { name: 'abc' } } );
        case xxx: return Object.assign( {}, state, { otherInfo: { name: 'abc' } });
    }
}

// 源码中写道：

// 旧的值
const previousStateForKey = state[key] 

// 新的值  previousStateForKey 即 reducer.order 的实参
const nextStateForKey = reducer(previousStateForKey, action) 

nextStateForKey !== previousStateForKey

如果直接`return Object.assign( state, { otherInfo: { name: 'abc' } } );` 即修改了previousStateForKey，虽然有一层Object.assign浅拷贝，但是对于多层的则会出现问题。 后续的 `hasChanged` 拿不到正确的值。。
```
总结：

    1. 将所有`reducer`是`function`的整合到`finalReducers`。
    2. 在dispatch一个action的时候遍历所有reducer，拿到每一个新的state，然后进行新旧比较

<br/>

- preloadedState(可选参数初始化状态)


- enhancer
```javascript
if (
    (typeof preloadedState === 'function' && typeof enhancer === 'function') ||
    (typeof enhancer === 'function' && typeof arguments[3] === 'function')
) {
    throw new Error(
    'It looks like you are passing several store enhancers to ' +
    'createStore(). This is not supported. Instead, compose them ' +
    'together to a single function.')
}
if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
}
if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
        throw new Error('Expected the enhancer to be a function.')
    }
    return enhancer(createStore)(reducer, preloadedState)
}
```