# AGENTS.md

## What this is

Minecraft mod/resource-pack update client ("McPatch"). Downloads incremental file updates from a server and applies them to a `.minecraft` directory. Runs as standalone JAR, Java agent (`-javaagent`), or called from a mod loader.

## Build

```bash
./gradlew shadowJar          # fat JAR in build/libs/Mcpatch-<version>.jar
```

- **Java 8 target** (Gradle toolchain). The wrapper is Gradle 9.5.1.
- If only a newer JDK is available, temporarily change `toolchain` to `sourceCompatibility`/`targetCompatibility` in `build.gradle.kts` — the code is compatible with `--release 8`.
- Version comes from `GITHUB_REF` tag (CI) or `DBG_VERSION` env var (defaults to `0.0.0`).
- No test source set exists (`src/test/` is absent). `./gradlew build` works but runs no tests.
- No lint, checkstyle, or formatter config.

## Entry points

- `Main.main(args)` — standalone execution. Pass `"windowless"` as first arg to disable GUI.
- `Main.premain(agentArgs)` — Java agent mode. Pass `"windowless"` as agent arg to disable GUI.
- `BalloonUpdateMain.modloader(enableLogFile, disableTheme)` — called by mod loaders.

All three funnel into `Main.AppMain(...)`.

## Dev mode

`Env.isDevelopment()` returns true when the class isn't inside a JAR. In dev mode:

- Config file `mcpatch.yml` **must** exist at `<project>/test/mcpatch.yml` (or set env vars below).
- Set `MCPATCH_DEV_WORK_DIR` and `MCPATCH_DEV_PROG_DIR` to override the default `test/` directory for config and working directory.
- Without these env vars, both program dir and working dir default to `<project>/test/`.

## Config

Runtime config is `mcpatch.yml` (YAML). Key fields: `urls` (server list), `version-file-path`, `silent-mode`, `base-path`, `http-timeout`, `retries`, `ignore-ssl-cert`. See `AppConfig.java` for all fields and defaults.

## Architecture (single module, no subprojects)

```
com.github.balloonupdate.mcpatch.client
├── Main.java              — entry points + directory resolution
├── Work.java              — core update logic (download, diff, apply)
├── BalloonUpdateMain.java — modloader facade
├── config/AppConfig.java  — YAML config parsing
├── network/
│   ├── Servers.java       — retry + failover wrapper
│   ├── UpdatingServer.java— protocol interface
│   └── impl/              — HttpProtocol, McpatchProtocol, WebdavProtocol, AlistProtocol
├── data/                  — domain objects (VersionMeta, IndexFile, FileChange, etc.)
├── ui/                    — Swing UI (McPatchWindow, ChangeLogs)
├── logging/               — custom logging (Log, FileHandler, ConsoleHandler)
├── utils/                 — Env, HashUtility, PathUtility, etc.
└── exceptions/            — McpatchBusinessException
```

- `Servers` wraps all protocols with automatic retry (configurable `retries`) and server failover.
- UI uses FlatLaf theme (`com.formdev:flatlaf`). Theme can be disabled via config or `disableTheme` param.
- Logging goes to console + optional file (`mcpatch.log` / `mcpatch.log.txt`).

## CI

GitHub Actions on tag push `v*`: setup JDK 8 (Zulu), run `shadowJar`, publish `build/libs/*` as GitHub release. See `.github/workflows/release.yml`.

## Conventions

- Chinese comments and UI strings throughout. Error messages, log output, and dialog text are all in Chinese.
- No `src/test` directory — the project has no automated tests.
- Package namespace: `com.github.balloonupdate.mcpatch.client` (main), `com.github.kasuminova.GUI` (Swing setup).
