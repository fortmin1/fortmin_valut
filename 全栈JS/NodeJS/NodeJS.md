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
## ESM 核心全局变量替代
ESM 中**没有** `require / module / exports / __dirname / __filename`，需要手动导入：
```js
import { fileURLToPath } from 'url'
import { dirname, join } from 'path'

const __filename = fileURLToPath(import.meta.url)
const __dirname = dirname(__filename)

// 拼接路径
const filePath = join(__dirname, './data.txt')
```
### import.meta.url
当前文件完整 `file://` 路径，ESM 独有元数据。
### 读取 JSON 文件
ESM 不能直接 `require('./cfg.json')`，两种方案：
1. 导入断言（Node 18+）
```js
import cfg from './config.json' assert { type: 'json' }
```
2. fs 读取 + JSON.parse
```js
import fs from 'fs/promises'
const cfg = JSON.parse(await fs.readFile('./config.json'))
```
---
## CommonJS 和 ESM 互操作规则
### 1. ESM 导入 CJS
可以直接 import，CJS 导出会被打包成 **default 导出**，无命名导出：
```js
// cjs 文件：utils.cjs
module.exports = { foo: 123 }

// esm 文件使用
import utils from './utils.cjs'
console.log(utils.foo)

// 解构会报错！不能写 import { foo }
```
### 2. CJS 导入 ESM
CJS 不能直接 `require()` ESM，只能用异步动态导入：
```js
// index.cjs
async function run() {
  const mod = await import('./esm-file.js')
}
run()
```
### 3. 混合项目区分
- `"type":"module"`：`.js`=ESM，`.cjs`=CJS
- 无 type 字段：`.js`=CJS，`.mjs`=ESM
---
## 关键注意事项（高频踩坑点）
### 1. 文件扩展名必须完整
不支持省略 `.js`、无自动 index 查找，是最常见报错。
### 2. 路径必须带 `./` / `../`
裸路径只识别第三方包，本地相对文件必须写前缀：
```js
// 错误
import utils from 'utils.js'
// 正确
import utils from './utils.js'
```
### 3. 顶层 await 合法（CJS 不支持）
ESM 模块顶层可以直接写 await，无需包裹 async 函数：

```js
// 合法 ESM
const data = await fetch('https://xxx')
```
### 4. 不能使用 require、module.exports、exports
出现直接抛错，全部替换为 import/export。
### 5. 目录、文件大小写敏感
Windows 下 CJS 大小写不敏感，ESM 统一遵循 POSIX 大小写规范，大小写不一致会找不到模块。
### 6. TypeScript 配套 ESM
```json
{
  "module": "ESNext",
  "moduleResolution": "NodeNext",
  "outDir": "./dist"
}
```

编译后输出 js 文件仍要保证导入带 `.js` 后缀。
### 7. package.json main /exports 字段
ESM 优先读取 `exports` 字段，路径规则和 CJS 有差异，建议显式配置导出入口。

### 8. 环境变量、进程对象
`process.env`、`console`、`setTimeout` 全局依然可用，不受 ESM 影响。
### 9. 运行时版本要求
- 基础 ESM 稳定：Node 14.13+
- JSON 导入断言：Node 18+
- import.meta 完整支持：Node 14.17+
    建议最低 Node 16 LTS 以上使用。
### 10. 脚本 shebang
ESM 可正常使用 `#!/usr/bin/env node` 做可执行脚本，配合 `"type":"module"` 无问题。
### 11. 第三方库兼容
大部分现代包同时支持 CJS/ESM；老旧仅 CJS 包只能用 `import xxx from 'pkg'`，无法解构命名导出。
### 12. 无法使用 require.resolve
如需解析路径，改用 `import.meta.resolve`（异步）：
```js
const path = await import.meta.resolve('./utils.js')
```
---