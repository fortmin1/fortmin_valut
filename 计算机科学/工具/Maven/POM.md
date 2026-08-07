> 核心结论：**不是只有 parent 才影响当前项目**。pom 是多层合并模型：`super‑pom(内置) → parent父pom → 当前项目pom → settings.xml`，多层配置会合并，不是简单覆盖。
> 
> 每个 jar 包本身都自带 pom 文件，依赖解析时也会读取它的 pom，传递依赖就是这么来的。
## 一、各个 POM 是什么
### 1. Super‑POM（隐式内置，看不见）
Maven 内核自带的默认 pom，所有项目的根。
里面定义了默认中央仓库`repo.maven.apache.org`、默认插件版本、默认目录结构。
> 所有 pom 都继承它，你写的 pom 只写差异部分。
### 2. Parent POM 父 pom
项目 pom 中`<parent>`指定的 pom，可以是：
1. 本地磁盘的多模块父工程（relativePath）
2. **远程仓库下载下来的 pom（就是你刚才报错的`th‑pdiot:1.0.4`）**
父 pom 本身也可以有自己的 parent，形成**pom 继承链**：
`super‑pom ← th‑pdiot(1.0.4) ← dataSync‑jdbc(你的项目)`
✅继承会传递：`properties、dependencyManagement、pluginManagement、repositories、build配置`都会向下继承。
> ⚠️注意：`<dependencies>`会继承；但`<repositories>`会继承；**mirror 镜像不在 pom 里，mirror 只存在 settings.xml**。
### 3. 当前项目 pom（本项目 pom.xml）
你自己项目的 pom。会**合并父 pom 的配置**，本项目写的同 id 配置会覆盖父 pom。
### 4. 依赖 jar 包自带的 pom（很容易被忽略）
你引入的每一个依赖`spring‑core、mybatis`，nexus 仓库里除了`.jar`，还附带一个`.pom`文件。
Maven 下载这个 jar 时，**同时读取这个 jar 的 pom**，解析它里面声明的`<dependencies>`，生成传递依赖。
> 👉这就是为什么你只引入 spring‑context，自动带出 spring‑core、spring‑beans。
> 这个依赖包的 pom**不会参与继承**，只用来算传递依赖，不会修改你项目的 build、仓库配置。
> 区分：
> - `<parent>`：**继承关系**，影响本项目构建行为；
> - `<dependencies>`：**依赖关系**，只拿 jar，读取它的 pom 做传递依赖，**不继承它的构建配置**。
---
## 二、配置合并规则：父 pom ↔ 子 pom
1. `properties`：子 pom 同名 key 覆盖父 pom。
2. `dependencyManagement`：子会合并父，子如果 groupId+artifactId+version 完全相同，子覆盖父版本。
3. `dependencies`：父的全部继承到子项目，子额外追加自己的依赖。
4. `<repositories>`（仓库列表）：**合并，不是覆盖**。父的仓库 + 子 pom 写的仓库全部生效，maven 会按顺序挨个尝试拉取。
5. `<build>、<plugins>`：pluginManagement 用来锁定版本；实际 plugins 会继承合并。
6. relativePath：只是**本地文件查找提示**；一旦找不到，就去远程仓库下载 parent 的 pom。
> ❗不是只有 parent 影响项目，继承链上每一层 parent 都会叠加。父的父也会生效。
## 三、pom.xml 和 settings.xml 的关系（重点，仓库地址冲突时如何生效）
> pom.xml：**项目级构建描述文件**，存代码库，提交 git。
> settings.xml：**本机用户环境配置**，不提交 git，只作用当前机器。
> 两类配置不能互相完全覆盖，分工不一样：

| 配置项                               | 在哪里写                                             |
| --------------------------------- | ------------------------------------------------ |
| repositories / pluginRepositories | **pom.xml（含父 pom）**                              |
| mirror（镜像拦截）                      | **只能 settings.xml**，pom 里不能写 mirror              |
| localRepository 本地仓库路径            | settings.xml                                     |
| profile（里面可以套 repositories）       | settings.xml/pom.xml 两边都可以写 profile              |
| http 仓库 block 拦截                  | settings.xml mirror `maven‑default‑http‑blocker` |

### 核心逻辑：
1. **pom（包括所有父 pom）提供候选仓库列表**：比如 pom 写了内网 nexus、阿里云。maven 拿到一张候选仓库清单。
2. **settings.xml 的 mirror 做拦截替换**：
> mirrorOf 匹配到哪些仓库 id，就把对该仓库的网络请求，重定向到 mirror 的 url。
> mirror 不会修改 pom 里面的仓库列表，只是**网络请求时偷换地址**。

> 举个例子：
> pom 里面配置仓库 A：`http://192.168.14.27:8081/xxx`
> settings 配置一个 mirror，`mirrorOf=A`，url 指向阿里云。
> 👉pom 逻辑上还是访问仓库 A，但是**实际网络请求打到阿里云**。

> 大坑：`<mirrorOf>*</mirrorOf>`，匹配所有仓库 id。pom 里面所有`<repositories>`全部失效，所有请求走 mirror 的 url。这是无数人踩坑点。
### profile 两边都可以写：
- pom 里面的 profile：跟随代码，提交 git；里面可以写 repositories。
- settings.xml 里面的 profile：本机环境，不进 git；适合放内网 nexus 这种本机环境配置。
- 两边 profile 如果 id 相同，**settings 的 profile 优先级高于 pom 的 profile**。
- `<activeProfiles>`在 settings 中激活 profile。
### 仓库完整执行流程（一次依赖拉取完整链路）
1. 先看本地仓库有没有这个 jar。有直接用。
2. 没有，读取完整继承链，合并 super‑pom + 所有父 pom + 当前 pom，得到全部`repositories`候选列表。
3. 读取 settings.xml 所有 mirror，对每一个候选仓库，判断是否被 mirror 匹配拦截。
4. 遍历处理后的远程仓库，依次尝试下载构件。
5. Maven3.8+：如果原始 pom 的仓库是 http 协议，会被内置`maven‑default‑http‑blocker`镜像拦截，直接报错。
6. 下载失败，本地生成`.lastUpdated`失败缓存，短时间不再重试。

> 注意：**settings 不会新增仓库列表，它只能通过 profile 追加仓库，或者 mirror 重定向已有仓库**。
> 
> 如果你的 pom 完全没有内网 nexus 仓库，仅仅只在 settings 写 mirror，mirrorOf=*，也能拉；但是一旦 pom 里面写了其他仓库，就会被全部接管。

## 四、回答你之前场景：为什么父 pom 找不到
`th‑pdiot:1.0.4`是 parent 父 pom。
1. 子 pom 声明 parent，maven 需要先下载这个 parent 的 pom。
2. 合并 pom 得到仓库列表，**如果仓库列表里没有内网 nexus，只有阿里云**，阿里云没有这个私有构件，下载失败。
3. 失败写入`.lastUpdated`缓存。
4. 报错：`Non‑resolvable parent POM`。
> 关键点：下载 parent 父 pom，同样会走上面整套仓库逻辑；**父 pom 本身也是一个要从远程仓库下载的构件**。
## 五、effective‑pom /effective‑settings 调试命令（理解合并结果神器）
```
# 把super‑pom + 所有父pom + 当前pom合并输出，看到最终生效的pom
mvn help:effective-pom

# 输出settings最终合并结果
mvn help:effective-settings
```
> 看 effective‑pom 输出的`<repositories>`，就能看到最终 maven 会去哪些仓库拉包。
## 六、最佳实践总结（结合你内网 nexus 场景）
1. **内网私有仓库不要写在 pom.xml**，写到 settings.xml 的 profile 中，本机激活。不同开发机器 nexus 地址可以不一样，不要硬编码进代码提交 git。
2. 不要使用`mirrorOf=*`，会把 pom 中所有 repositories 全部接管。
3. pom 只负责业务、版本管理、dependencyManagement；settings 负责本机环境：镜像、内网仓库、本地仓库路径、http 放行。
4. parent 父 pom 是远程构件，一定要设置`<relativePath/>`，不要让 maven 去本地找。
5. 遇到`.lastUpdated`缓存，构建带上 `-U` 参数强制刷新。
### 简单比喻方便记忆
- super‑pom：maven 出厂默认配置
- parent 继承：类的继承，子类复用父类的字段方法
- dependencies：导入第三方库，只拿库，不继承库的配置
- pom：项目源代码一部分；settings：本机电脑的环境变量。
- mirror：网络代理，不改配置文件，只是访问的时候转发请求。
# 附录：POM配置项
```

```