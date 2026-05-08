# 面向对象编程
## 继承与Mixin
JS只支持单继承，查找属性/方法，走的是原型链。
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
##### （1）核心语法

|操作|语法示例|说明|
|---|---|---|
|命名导出（多个）|`export const add = (a,b) => a+b;`|导出单个成员（可多个）|
|命名导出（批量）|`export { add, User };`|批量导出已定义的成员|
|默认导出（单个）|`export default function() {};`|每个模块只能有一个默认导出|
|导入命名成员|`import { add } from './utils.js';`|解构导入命名导出的成员|
|导入默认成员|`import add from './utils.js';`|导入默认导出的成员|
|导入整个模块|`import * as utils from './utils.js';`|导入为一个模块对象（所有成员挂在上面）|
|动态导入|`import('./utils.js').then(utils => {})`|异步导入（运行时执行）|

##### （2）导出 / 导入的本质（关键）
- **导出的是什么**：
    - 命名导出：导出的是**变量 / 函数的绑定关系**（不是值的拷贝），如果导出方修改了变量值，导入方会同步更新；
    - 默认导出：本质是命名为 `default` 的特殊命名导出，`export default xxx` 等价于 `export { xxx as default }`；
    - ESM 是**静态的**（编译期分析），`export/import` 必须放在顶层（不能在 if 里），这也是它能支持树摇的原因。
- **导入的是什么**：
    - 导入的是**绑定关系**（而非值的拷贝），且导入的成员是只读的（不能修改导入的变量）；
    - 模块对象（`import * as utils`）是只读的，不能修改其属性。    
##### （3）完整示例
```
// utils.js (ESM 模块)
// 命名导出
export const version = '1.0';
export function add(a, b) {
  return a + b;
}

// 默认导出
export default function multiply(a, b) {
  return a * b;
}

// index.js (导入)
// 导入命名成员 + 默认成员
import multiply, { version, add } from './utils.js';
console.log(version); // '1.0'
console.log(add(1, 2)); // 3
console.log(multiply(2, 3)); // 6

// 导入整个模块为对象
import * as utils from './utils.js';
console.log(utils.version); // '1.0'
console.log(utils.default(3, 4)); // 12（默认导出在 default 属性里）
```