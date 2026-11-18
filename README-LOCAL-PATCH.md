# Better Sidebar 当前文件定位补丁

## 现在怎么使用

打开 Markdown 后，点击文件路径旁的 **◎ 在目录树中定位当前文件**。
插件会打开编辑器的文件侧栏、清除活动搜索、逐级展开父目录、选择当前文件，并将目标滚动到目录视口中央。不会修改文件或会话、运行 model 或写 Git。

- 宿主：DeepSeek Harness Web `0.2.2-rc.4`。

- 官方基底：[`dsh-better-sidebar 0.32.1`](https://github.com/omdsh-dev/DSH-better-sidebar/releases/tag/v0.32.1)。
- 本地补丁：`0.32.1-reveal.1`。
- 安装源码：`/Users/mumu/Documents/Work/ADA/work-summary/ADA_WorkSpace/operations/dsh-better-sidebar`。
- 取恢复源码：[个人 fork](https://github.com/zhangxy-zju/DSH-better-sidebar)，分支 `feat/reveal-current-file-0321`，当前 revision `35f0c38dde4ca354e8fbcf02fbada3961dc52b7f`。
- 本目录是保留 `.git` 的插件 checkout；`node_modules/` 是开发依赖，`lib/` 为构建输出。它们不作为同步输入。

官方 0.32.1 不具备完整的一键文件定位入口。这里的补丁在既有目录懒加载基础上实现精确定位；它不是官方新版本的承诺。

## 换机恢复

仅 MacBook 浏览器通过 SSH 看本服务时，不需要在 MacBook 安装 DeepSeek Harness 或插件。下面针对**提供 Web 服务的 Mac mini 或其它主机**。

### 1. 先恢复服务，恢复兼容 DSH

先遵循 [DeepSeek Harness 换机交接](../deepseek-harness-dsh/README.md) 恢复 `web` profile、工作区和现有 `com.mumu.dsh-web`，不要用默认 profile 覆盖现有模型/LaunchAgent 配置。

若没有安装 DSH，可先使用这一已验证版本；其它版本需另核对插件 peer 要求和迁移文档（本插件要求 DSH `0.2.2-rc.2+`）：

```sh
cd /Users/mumu/Documents/Work/ADA
node --version # 本版本 DSH 要求 Node >= 24.9
npm install --global @deepseek-ai/dsh@0.2.2-rc.4
```

已有兼容 DSH 以及其它插件、模型设置时，不必重复此升级，也不要只装本插件；需按现有不涉及凭据的配置文档恢复它们。

### 2. 通过上层 work-summary 恢复本子模块

本目录是**独立 Git submodule**；不是 specs 临时产物，也不是嵌在父仓中的普通源码目录：

```sh
WORK=/Users/mumu/Documents/Work/ADA
PLUGIN="$WORK/work-summary/ADA_WorkSpace/operations/dsh-better-sidebar"
git -C "$WORK/work-summary" submodule update --init --recursive -- ADA_WorkSpace/operations/dsh-better-sidebar
```

这一步会以父仓记录的补丁 revision 恢复真实源码。普通 `git pull` 不会自动下载未初始化的 submodule，不能只拉父仓便声称插件已可安装。检查：

```sh RGCTX
test -f "$PLUGIN/package.json"
node -p "require('$PLUGIN/package.json').version"
git -C "$PLUGIN" rev-parse --short HEAD
```

应看到插件版本 `0.32.1-reveal.1`、父仓登记的对应 commit。若此 revision 已不存在或 fork 无法访问，应先修复仓库/凭据，不能用旧的 specs 路径替代。源码和 README 随 `work-summary-master` + fork 分发；依赖输出不应复制旧设备的绝对路径。

### 3. 构建和测试

```sh
cd "$PLUGIN"
pnpm install --frozen-lockfile
pnpm typecheck
pnpm lint
pnpm test
pnpm build
pnpm check:consumer-types
```

不要手改 profile 的 `package.json` 或 `cordis.patch.yml`，也不在 profile 目录直接执行 pnpm；这些由插件管理器负责。
依赖安装超时是在外网下载节点依赖，本地缓存完整时优先 `--prefer-offline`，不要删除配置以绕过失败。

本公司在其它工作仓注入的 Git hook 影响测试创建临时提交；因此曾出现套餐全量 Git 用例 16 个与本功能无关的失败。没有把所有基础插件测试都宣称通过。**此项功能相关的类型检查、lint、构建接口检查与 157 项回归检查通过**；未随机跳过测试以显示全绿。当前真实测试请读执行输出，不把上次报告当当前验收。

### 4. 将本地目录作为插件安装

**管理器场景**，使用 Agent 可使用插件管理器执行：

1. `set_bundle`：`target=dsh-better-sidebar`，`enabled=false`；
2. `remove_bundle`：`target=dsh-better-sidebar`（移除包引用，不是删除源码目录）；
3. `install_bundle`：`target=$PLUGIN`。

**网页场景**，在 **插件 → 添加插件** 选择 `PLUGIN`；若同 id 已存在，先停用并移除原官方 bundle 再安装此本地目录。不能把 `package.json` 提供插件名的远程安装与本补丁目录链接机械地重复挂载。
读取插件管理器返回的 `application` / `nextStep`；已停用与已完整安装并不是一件事。确认版本 `0.32.1-reveal.1` 已启用且 profile 链接指向此目录，不得再指向已移除的 specs 副本：

```sh
readlink "$HOME/.dsh/profiles/web/node_modules/dsh-better-sidebar"
```

工作区和用户 `web` profile 是独立配置层，父仓恢复不能取代这里的安装动作。

### 5. 重启旧服务，验收 GUI

`com.mumu.dsh-web` 是本机用户 LaunchAgent，保持原有 `3080` 页面，不另外启动第二个 DSH 服务。换本地目录后，看到管理器需要重启或切换包位置时执行：

```sh
launchctl kickstart -k "gui/$(id -u)/com.mumu.dsh-web"
lsof -nP -iTCP:3080 -sTCP:LISTEN
```

沿用 [双机接入说明](../deepseek-harness-dsh/macbook.md)，不把最新含认证 token 的 URL 写入本文或父仓。

验收只需 Web 控制台：

1. 新建空会话或直接编辑无模型提示，在右上「文件」的目录树打开一个多层嵌套 `.md`。
2. 点击路径栏的 **◎**，确认展开的是精确父目录、选中的是目标文件（`data-dsh-selected=true`、`aria-pressed=true`）而不是祖先目录。
3. 目标行应在树视口内居中，不能使右侧整体滚动或编辑文件。
4. 先在树搜索中输入不匹配词再定位，应清搜索且仍能选中该文件。
5. 重复点击仍能定位；键盘激活保留按钮焦点。

本地旧验收图：[定位验收截图](./evidence-current-file-reveal-final.png)。在上层主交接以本机再次复验，不宣称已经在第二台机器通过。

## 升级与维护

- 如需上游更新，先读取新版本的发布说明/兼容要求，以新版本源码为基础重新应用本功能的小 patch。不要用 `npm latest` 覆盖后仍声称定位补丁存在。
- 本补丁先按匹配 DSH 的 release tag 直接建立功能分支，不是上游的 `dev` 提交。不推插件原作者仓库，也不创建 PR，除非用户另提贡献许可。上游合并工作流用在该时才合规执行。
- 当前执行的是个人 fork 上的补丁分支，父仓以 Git revision 精确恢复。进一步改动时先在插件本地检查差异、不夹带任务数据，按需再推同名 fork 分支，并显式更新父仓 submodule revision。
- 原生编辑器调速器通过 `native/tab-adapter` 维护每-tab 展开的目录和 reveal，在 `TreePanel` 对本次点击清搜索，`FileTree` 只锁定实际出现的目标行；循环、选中与滚动一个版本。修改时不要退回到“先选祖先路径以蒙混 reveal”。
- 此文档仅针对插件和恢复，不涉及业务 CR 合入、线上部署或换 model 凭据。模型/API 密钥按原来各自安全约定恢复，绝不写进这里。

## 本次实现与验证范围

2026-11-18 已基于 DSH `0.2.2-rc.4` 和插件 `0.32.1-reveal.1`：

- 子模块式的 task 目录迁移到本正式位置，后台正在运行的 profile 插件已重新安装此处。
- 在已派生 release 中构建和针对性回归（目录/状态测试、旧原生文件树回归、i18n、类型及 lint）已通过。
- 实际 GUI 中读取到目标文件 `summary.md` 的 `data-dsh-selected=true`；目标行 `top=461px`，位于 830px 视口内，实际目录滚动量 `6327px`；重复定位、搜索清空，以及键盘焦点均验证通过。
- 全量上游基础测试中的公司 hook 兼容问题不等同于本功能失败，上面明确披露，坐标测量也不是 GUI 截图单独猜测。
