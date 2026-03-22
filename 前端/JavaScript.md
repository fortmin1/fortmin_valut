# 模块化
| 特性   | CommonJS (CJS)         | ES Modules (ESM)     | TypeScript `namespace` |
| ---- | ---------------------- | -------------------- | ---------------------- |
| 归属   | JS 社区模块化方案（Node.js 原生） | JS 官方模块化标准（ES6+）     | TS 特有作用域方案（非标准 JS）     |
| 运行环境 | Node.js 原生支持，浏览器需打包    | 现代浏览器 / Node.js 原生支持 | 仅 TS 编译阶段有效，编译后消失      |
| 核心粒度 | 文件级模块                  | 文件级模块                | 全局作用域下的逻辑分组            |
| 加载方式 | 同步加载                   | 异步加载（静态分析）           | 无 “加载” 概念，靠文件加载顺序      |
|      |                        |                      |                        |
## CommonJs
CJS 是 Node.js 的默认模块化方案，核心是 `module.exports` 和 `require`，**所有导出都是 “值的引用”，且导出的是一个对象（或原始值）**。
##### （1）核心语法

|操作|语法示例|说明|
|---|---|---|
|导出单个值|`module.exports = 123;`|直接导出原始值（覆盖默认导出对象）|
|导出多个值|`module.exports = { add, User };`|导出一个对象，包含多个成员|
|便捷导出|`exports.add = (a,b) => a+b;`|`exports` 是 `module.exports` 的引用|
|导入|`const { add } = require('./utils');`|同步导入，解构导出对象的成员|
|导入整个模块|`const utils = require('./utils');`|导入导出的完整对象|

##### （2）导出 / 导入的本质（关键）
- **导出的是什么**：
    - `module.exports` 是一个**普通 JS 对象**（默认是空对象 `{}`），你可以给它赋值任意类型（原始值、函数、对象等）；
    - `exports` 只是 `module.exports` 的引用，**不能直接给 `exports` 赋值**（比如 `exports = 123` 无效，因为会断开引用）。
   
- **导入的是什么**：
    - `require()` 函数执行时，会读取目标文件的代码，**执行**后返回 `module.exports` 的值；
    - 导入的是**值的引用**（运行时确定），如果导出的值后续被修改，导入方也会看到变化。    
##### （3）完整示例
```
// utils.js (CJS 模块)
// 定义内部变量
const version = '1.0';

// 导出函数（便捷方式）
exports.add = function(a, b) {
  return a + b;
};

// 导出对象（覆盖 exports 引用，不推荐混合使用）
module.exports = {
  version,
  multiply: (a, b) => a * b
};

// index.js (导入)
// 导入整个导出对象
const utils = require('./utils');
console.log(utils.version); // '1.0'
console.log(utils.multiply(2, 3)); // 6

// 解构导入
const { multiply } = require('./utils');
console.log(multiply(3, 4)); // 12
```
## ESM
ESM 是 ES6 推出的官方标准，核心是 `export` 和 `import`，**导出的是 “绑定关系”（而非值），支持多种导出形式**。