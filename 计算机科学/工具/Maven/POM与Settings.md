> 重要前提

1. **pom.xml**：项目对象模型，属于**项目代码一部分**，提交 Git；包含项目构建、依赖、仓库、插件。继承链：`super‑pom → 父pom → 当前pom`。
2. **settings.xml**：本机环境配置，**不提交 Git**；分两份：用户级 `~/.m2/settings.xml`、全局 `maven/conf/settings.xml`。
3. `mirror` **只能写在 settings，pom 没有 mirror**。
4. `repositories` 两边都可以通过 profile 定义；pom 顶层直接写 repositories；settings 只能写在 profile 内部。

## 一、pom.xml 全部核心配置项（项目 POM）
```xml
<project>
  <!-- 项目坐标，唯一标识构件 -->
  <groupId></groupId>
  <artifactId></artifactId>
  <version></version>
  <packaging></packaging>
  <name></name>
  <description></description>

  <!-- 父POM继承，继承父的全部配置 -->
  <parent>
    <groupId></groupId>
    <artifactId></artifactId>
    <version></version>
    <relativePath/>
  </parent>

  <!-- 属性，变量占位 -->
  <properties></properties>

  <!-- 直接依赖，本项目实际引入的jar -->
  <dependencies>
    <dependency>
      <groupId/><artifactId/><version/>
      <scope/><exclusions/>
    </dependency>
  </dependencies>

  <!-- 依赖版本管理，只声明版本，不引入依赖；子项目继承 -->
  <dependencyManagement>
    <dependencies></dependencies>
  </dependencyManagement>

  <!-- 插件版本管理 -->
  <pluginManagement>
    <plugins></plugins>
  </pluginManagement>

  <!-- 构建插件，实际执行的插件 -->
  <build>
    <plugins></plugins>
    <resources></resources>
    <testResources></testResources>
    <directory></directory>
  </build>

  <!-- 项目直接定义远程仓库（供依赖下载） -->
  <repositories>
    <repository>
      <id/><url/><releases/><snapshots/>
    </repository>
  </repositories>

  <!-- 插件仓库，专门下载maven插件 -->
  <pluginRepositories>
    <pluginRepository></pluginRepository>
  </pluginRepositories>

  <!-- pom内的profile，跟随代码提交git -->
  <profiles>
    <profile>
      <id></id>
      <properties></properties>
      <dependencies></dependencies>
      <repositories></repositories>
      <pluginRepositories></pluginRepositories>
      <build></build>
    </profile>
  </profiles>

  <!-- 分发配置，mvn deploy上传到nexus -->
  <distributionManagement>
    <repository></repository>
    <snapshotRepository></snapshotRepository>
  </distributionManagement>
</project>
```

> ✅pom 独有：`groupId/artifactId/version/packaging`、`parent`、`dependencies`、`dependencyManagement`、`pluginManagement`、`distributionManagement`。

## 二、settings.xml 全部核心配置项（本机环境）
```xml
<settings>
  <!-- 本地仓库jar缓存目录 -->
  <localRepository></localRepository>

  <!-- 是否离线构建 -->
  <offline></offline>

  <!-- maven交互模式 -->
  <interactiveMode></interactiveMode>

  <!-- 服务账号，nexus仓库用户名密码 -->
  <servers>
    <server>
      <id></id>
      <username></username>
      <password></password>
    </server>
  </servers>

  <!-- 镜像拦截，只能在settings，pom不支持mirror -->
  <mirrors>
    <mirror>
      <id></id>
      <mirrorOf></mirrorOf>
      <url></url>
      <blocked></blocked>
    </mirror>
  </mirrors>

  <!-- 本机的profile，不提交git -->
  <profiles>
    <profile>
      <id></id>
      <properties></properties>
      <repositories></repositories>
      <pluginRepositories></pluginRepositories>
    </profile>
  </profiles>

  <!-- 激活profile列表 -->
  <activeProfiles>
    <activeProfile>xxx</activeProfile>
  </activeProfiles>
</settings>
```

> ✅settings 独有：`localRepository`、`servers`、`mirrors`、`offline`、`interactiveMode`。

> 补充：`settings‑security.xml`：存放安全配置 `<settingsSecurity><relaxedBaseUrls>true</relaxedBaseUrls></settingsSecurity>`，独立文件。

## 三、两边**都可以出现**的共同配置（重点）

只有放在 `<profile>` 内部的配置，**pom 的 profile 和 settings 的 profile 都支持**：

表格

|共同配置项|说明|优先级规则|
|---|---|---|
|`<properties>`|自定义变量|profile id 相同时：**settings.xml > pom.xml**|
|`<repositories>`|依赖仓库列表|profile id 相同时：**settings.xml > pom.xml**；全部合并生效，不是覆盖|
|`<pluginRepositories>`|插件仓库列表|profile id 相同时：**settings.xml > pom.xml**|

> ⚠️注意：

1. pom 可以在顶层直接写 `<repositories>`（不在 profile 里面）；**settings.xml 不允许顶层直接写 repositories，只能写在 profile 内部**。
2. settings 没有 dependencies、build、parent、distributionManagement。

### 关键：profile ID 冲突规则

> 如果 settings.xml 和 pom.xml 里面都定义了同一个 id 的 profile，**settings 的 profile 配置优先级更高，会覆盖同 id 下的同名 property；repositories 会合并**。

## 四、仓库完整逻辑复盘（pom vs settings）

1. pom（含父 pom）：顶层`<repositories>` + pom 内部 profile 的`<repositories>` → 生成原始候选仓库列表。
2. settings.xml profile 的`<repositories>`追加到候选列表。
3. settings.xml 的`<mirrors>`**不新增仓库，只做网络请求拦截重定向**。mirrorOf 匹配仓库 id，把请求转发到 mirror 的 url。
4. 访问 parent 父 pom，同样走这套仓库解析逻辑。

## 五、一张表区分：哪些在哪，哪些不能在哪

表格

|配置|pom.xml|settings.xml|备注|
|---|---|---|---|
|groupId/artifactId/version|✅|❌|项目坐标，只有 pom 有|
|parent 继承|✅|❌|父 pom，settings 不能写|
|dependencies|✅|❌|项目依赖，settings 不能写|
|dependencyManagement|✅|❌|版本锁定|
|distributionManagement|✅|❌|deploy 上传 nexus 地址|
|localRepository|❌|✅|本地 jar 缓存目录|
|servers（账号密码）|❌|✅|nexus 登录凭证|
|mirrors|❌|✅|镜像拦截，pom 不支持 mirror 标签|
|offline 离线|❌|✅|离线模式开关|
|profiles 标签|✅|✅|两边都可以定义 profile|
|properties（profile 内）|✅|✅|profile 内变量，settings 优先级高|
|repositories|✅顶层直接写<br><br>✅profile 内|❌不能顶层写<br><br>✅只能 profile 内|所有仓库合并，依次尝试访问|
|pluginRepositories|✅顶层直接写<br><br>✅profile 内|❌不能顶层写<br><br>✅只能 profile 内|插件下载仓库|
|activeProfiles|❌|✅|只在 settings 激活 profile|

## 六、最佳实践总结

1. pom.xml：放项目业务、版本、依赖、打包、分发地址；**不要写内网 nexus 仓库地址**，避免提交 git。
2. settings.xml：放本机环境：本地仓库、nexus 账号 servers、mirror 镜像、内网仓库写到 profile 中。
3. 内网私有仓库统一写在 settings 的 profile，本机激活；不同机器可以不同，不和代码绑定。
4. 调试：`mvn help:effective‑pom`看合并后的 pom；`mvn help:effective‑settings`看 settings 生效结果。

> 常见踩坑：
> 
> 1. 想在 settings 写 dependencies，**做不到，settings 没有这个配置**。
> 2. 想在 pom 写 mirror，**做不到，mirror 只能写 settings**。
> 3. settings 无法直接新增仓库，必须包在 profile 标签里面，并且 activeProfiles 激活。

如果你需要，我可以给一份极简示例：同一个 profileId，分别在 pom 和 settings 写配置，演示合并效果。