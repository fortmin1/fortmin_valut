# NodeJS中使用ESM
## 开启ESM
### 方式 1：package.json 添加 `"type": "module"`（推荐全局生效）
```json
{ "type": "module" }
```
生效规则：
- `.js`、`.mjs` 文件默认按 ESM 解析
- `.cjs` 强制 CommonJS
### 方式 2：文件后缀 `.mjs`（单文件启用）
无需改 package.json，文件命名 `xxx.mjs`，直接 ESM。
## 使用ESM
### 导入导出
```js
// ESM
import fs from 'fs'
import { readFile } from 'fs/promises'
import * as allPath from 'path'
export const name = 'demo'
export default function test() {}

// 动态导入（异步）
const mod = await import('./util.js')
```
### 内置模块、第三方包导入
直接 `import xxx from '包名'`，和 CJS `require` 一致。
### 本地文件导入必须写完整后缀
**ESM 不允许省略 .js/.json/.mjs**
```js
// 正确
import utils from './utils.js'
import config from './config.json' assert { type: 'json' }

// 错误（CJS 习惯，ESM 报错）
import utils from './utils'
```
### 目录导入不能省略 index.js
```js
// 正确
import api from './api/index.js'
// 错误
import api from './api'
```