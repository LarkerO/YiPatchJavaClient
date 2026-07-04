# McPatch Java Client

Minecraft 整合包文件增量更新工具。通过对比文件差异，仅下载变化的部分，实现快速更新模组、配置、资源包等文件。

本仓库是 jar 格式的客户端程序，使用 Java 编写，支持在 Windows（含 Win7）、Linux、Linux Arm、macOS 等平台运行。

> 如果需要更好的性能和稳定性，优先考虑 [exe 客户端](https://github.com/BalloonUpdate/Mcpatch2RustClient/releases)（Rust 编写，仅支持 x86 平台）。

## 生态

| 组件 | 仓库 | 说明 |
|---|---|---|
| 管理端 | [McPatch2](https://github.com/BalloonUpdate/McPatch2) | 打包更新包、管理版本、内置服务端 |
| jar 客户端 | [Mcpatch2JavaClient](https://github.com/BalloonUpdate/Mcpatch2JavaClient)（本仓库） | 运行在玩家端，下载并应用更新 |
| exe 客户端 | [Mcpatch2RustClient](https://github.com/BalloonUpdate/Mcpatch2RustClient) | Rust 版客户端，性能更优 |
| 加载器 | [McPatch2Loader](https://github.com/BalloonUpdate/McPatch2Loader) | 配合 exe 客户端使用的一键启动加载器 |

完整文档：[McPatchDocs](https://balloonupdate.github.io/McPatchDocs/docs/v2/start)

## 下载

- [GitHub Releases](https://github.com/BalloonUpdate/Mcpatch2JavaClient/releases)
- [hoshiroko.com](https://mcpatch.hoshiroko.com)（镜像）

## 安装

### 1. 放置客户端

将下载的 `Mcpatch-xxx.jar` 放到 Minecraft 客户端的 `.minecraft/mcpatch` 目录下（目录不存在需手动创建）。

```
Minecraft客户端/
├── .minecraft/
│   ├── mcpatch/
│   │   ├── Mcpatch-xxx.jar     # 客户端程序
│   │   ├── mcpatch.yml         # 配置文件（首次运行自动生成）
│   │   ├── version-label.txt   # 版本号记录文件（自动生成）
│   │   └── mcpatch.log         # 日志文件（自动生成）
│   └── versions/
└── PCL启动器.exe
```

### 2. 配置客户端

首次运行客户端会自动生成配置文件 `mcpatch.yml`。打开编辑，将 `urls` 改为你的更新服务器地址：

```yaml
urls:
  - mcpatch://你的服务器IP:6700    # 使用管理端内置服务端（私有协议，免备案）
```

如果管理端和客户端在同一台电脑上，IP 填 `127.0.0.1` 即可。

### 3. 设置一键启动

在启动器的 JVM 参数**最前面**添加：

```
-javaagent:mcpatch/Mcpatch-xxx.jar
```

注意 `.jar` 后面要保留一个空格。

> **版本隔离**：如果开启了版本隔离，工作目录会变成 `.minecraft/versions/xxxx/`，需要改为：
> ```
> -javaagent:../../mcpatch/Mcpatch-xxx.jar
> ```

启动游戏时，客户端会自动在 Minecraft 窗口弹出前完成更新。

## 支持的协议

| 协议 | URL 格式 | 说明 |
|---|---|---|
| 私有协议 | `mcpatch://IP:端口` | McPatch 自带协议，免备案，适合小服或内网 |
| HTTP/HTTPS | `http(s)://IP:端口/路径` | 标准 HTTP 文件服务 |
| WebDAV | `webdav(s)://用户:密码@IP:端口` | WebDAV 协议 |
| AList | `alist://...` | AList 网盘服务 |

支持填写多个地址作为备用，故障时自动切换。

## 配置参考

配置文件 `mcpatch.yml` 完整字段：

```yaml
# 更新服务器地址（支持多个备用地址）
urls:
  - mcpatch://127.0.0.1:6700

# 版本号记录文件路径
version-file-path: version-label.txt

# 出错时是否继续启动游戏（仅非图形模式有效）
allow-error: false

# 无更新时是否弹框提示
show-finish-message: true

# 安静模式：仅在下载时才显示窗口
silent-mode: false

# 窗口标题
window-title: Mcpatch

# 更新目标目录（空字符串=自动搜索 .minecraft 父目录）
base-path: ''

# 私有协议超时（毫秒）
private-timeout: 7000

# HTTP/WebDAV 自定义请求头
http-headers: {}

# HTTP/WebDAV 超时（毫秒）
http-timeout: 5000

# HTTP/WebDAV 重试次数
http-retries: 3

# 是否忽略 SSL 证书验证
http-ignore-certificate: false
```

完整配置说明见 [配置文件参考](https://balloonupdate.github.io/McPatchDocs/docs/v2/config)。

## 从源码构建

需要 JDK 17+（Gradle 9.5.1 运行要求），通过 toolchain 自动编译为 Java 8 兼容产物：

```bash
./gradlew shadowJar
```

产物：`build/libs/Mcpatch-<version>.jar`

如果只有 JDK 8 可用，需将 `build.gradle.kts` 中的 `toolchain` 改为：

```kotlin
java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}
```

## License

[MIT](LICENSE)
