# dsh/stable —— 这条分支怎么用

上游是 [Dwinovo/minecraft-numen](https://github.com/Dwinovo/minecraft-numen)，这里放的是给「大肥鱼 + 整合包」场景打的小补丁，
目标是**能跟上上游、又随时能回滚**，不是另起炉灶。

## 远端

| 名字 | 地址 | 用途 |
|---|---|---|
| `upstream` | `https://github.com/Dwinovo/minecraft-numen.git` | 只读，永远不往上推 |
| `origin` | `https://github.com/XINGO0123/minecraft-numen.git` | 自己的 fork，`dsh/stable` 推这里 |

## 分支

- `1.21.1`：上游的镜像，只 `git pull`，不改。
- `dsh/stable`：改动都在这里。**一个改动一个 commit**，commit message 写明「改了什么、为什么、对应哪个 issue」。

## 跟上游

```powershell
git fetch upstream 1.21.1
git checkout dsh/stable
git rebase upstream/1.21.1      # 冲突基本只会在我们改过的那几个文件里
```

上游发版后重新构建、部署、跑一遍 `bench`（见下），没问题再打 tag：`git tag v0.1.5-dsh.1`。

## 构建

需要 JDK 21（仓库 `.tool-versions` 写的就是 21）：

```powershell
$env:JAVA_HOME='D:\game\MC\runtime\jdk-21.0.13+9'
.\gradlew.bat :numen:neoforge:build
# 产物：numen\neoforge\build\libs\numen-neoforge-1.21.1-<version>.jar
```

首次构建要下 Gradle 9.7.1 与 NeoForge/Minecraft 依赖，几分钟到十几分钟；之后增量构建很快。

## 部署到本机实例

```powershell
# 先备份，再换 jar（游戏必须先关掉）
$inst='D:\game\MC\instances\numen\mods'
Move-Item "$inst\numen-neoforge-*.jar" 'D:\game\MC\dl\backup\' -Force
Copy-Item 'numen\neoforge\build\libs\numen-neoforge-1.21.1-*.jar' $inst
```

回滚就反过来：把 `D:\game\MC\dl\backup\` 里的旧 jar 拷回去。

## 这条分支上有什么（保持最小）

| 补丁 | 文件 | 解决的现象 |
|---|---|---|
| 易碎格不再默认排除 | `numen/pathing/.../spec/RouteSpec.java` | 整片麦田被当成墙，进不了田、收不了麦（issue #168 / #171）。落地那一步仍由 `Contact` 按 `Kind.FRAGILE` 拒掉 |
| 走不通的步连周围一圈一起记 | `numen/pathing/.../search/TripMemory.java` | 水流/活塞把身体挤偏一格就当成新情况，原地来回重搜（issue #157） |

**还没做的**：搭柱按高度增罚（#170）。要把「已经爬了多高」放进搜索状态，改动面大，等 `bench` 跑到基线再动。

具体 commit：`git log --oneline upstream/1.21.1..dsh/stable`。
