> babel是一个JS编译器，主要用于将ECMAScript2015+版本的代码转换为向后兼容的js语法，以便能够运行在当前和旧版本的浏览器或其他环境中

那么Babel能够做什么：
- 语法转换
- 通过`Polyfill`方式在目标环境中添加缺失的特性)(@babel/polyfill 模块)
- 源码转换


#### 插件
##### 语法插件(解析)
只允许Babel解析(parse)特定类型的语法(不是转换)，可以在AST转换时使用，以支持解析新语法例如
```javascript
import * as babel from "@babel/core";
const code = babel.transformFromAstSync(ast, {    //支持可选链    
    plugins: ["@babel/plugin-proposal-optional-chaining"],babelrc: false
}).code;
```

##### 转换插件(转换)
转换插件会启用相应的语法插件(因此不需要同时制定这两种插件)，解析->转换。

</br>

#### 插件的使用
如果插件发布在npm上，则可以直接填写插件的名称，Babel会自动检查它是否已经被安装在node_modules目录下，在项目目录下创建.babelrc文件，如下
```javascript
//.babelrc
{
    "plugins":["@babel/plugin-transform-arrow-functions"]
}
```

#### 预设
通过使用或创建一个`preset` 即可轻松使用一组插件
> 官方Preset
- @babel/preset-env
- @babel/preset-flow
- @babel/preset-react
- @babel/preset-typescript

##### @babel/preset-env
`@babel/preset-env` 主要作用是对目标浏览器中缺失的功能进行代码转换和加载`polyfill`，在不进行任何配置的情况下，@babel/preset-env 所包含的插件将支持所有最新的JS特性(ES2015,ES2016等，但不包含stage阶段)，将其转换成ES5代码。所以代码中使用了目前仍在stage阶段，只配置@babel/preset-env，转换时会抛出错误，需要另外安装相应的插件。
官方推荐使用`.browserslistrc`文件来指定目标环境

语法转换只是将高版本的语法转换成低版本的，但是新的内置函数、实例方法无法转换。`polyfill`的中文意思就是垫片。即垫平不同浏览器或者不同环境下的差异，让新的内置函数、实例方法等在低版本浏览器中使用