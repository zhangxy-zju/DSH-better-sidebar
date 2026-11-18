# dsh-better-sidebar：当前文件定位补丁

**定位按钮位于 Markdown 文件工具栏的靶心图标 `◎`，不是独立文件页或文件树按钮。**

本目录是 `dsh-better-sidebar` 的插件源码 checkout（保留自己的 `.git`），不是 specs 临时任务材料。
当前 Web profile 通过本地目录链接安装这里提供的补丁版；`node_modules` 与 `lib` 是本机开发/构建产物，**不作为远程恢复输入**。

## 当前实际版本（2026-11-18）

- DSH：`0.2.2-rc.4`，profile：`web`。
- 官方插件基底：`dsh-better-sidebar 0.32.1`。
- 本地补丁包：`0.32.1-reveal.1`。
- 稳定源码位置：
  `/Users/mumu/Documents/Work/ADA/work-summary/ADA_WorkSpace/operations/dsh-better-sidebar`
- 补丁的能力：Markdown 工具栏定位当前文件，打开右侧文件树，逐级展开父目录，精确定位并选中文件，滚动到目录视口内；活动搜索会清空；不会切换会话或修改文件内容/Git。
- 与官方 `0.32.1` 的边界：官方新版没有完整“打开当前 Markdown → 一键定位到目录选中文件”的入口；这里新增完整路径导航和延迟精确选择。保留其他插件的回调接口及 Peer 插件接入方式。

## 换机恢复（主机安装 DeepSeek Harness 的场景）

**适用对象主要是 Mac mini；MacBook 只是浏览器通过 SSH 访问 Mac mini 时无需再次安装 DSH 或插件。**
先恢复 [DSH 主交接](../deepseek-harness-dsh/README.md) 的服务、profile 和工作区，再恢复这里的本地插件。

### A. 获取本插件真实源码

如果已经随 `work-summary` 登记子模块并取得上层 `master`：

```sh
WORK=/Users/mumu/Documents/Work/ADA
PLUGIN="$WORK/work-summary/ADA_WorkSpace/operations/dsh-better-sidebar"
git -C "$WORK/work-summary" submodule update --init --recursive -- ADA_WorkSpace/operations/dsh-better-sidebar
# 已初始化的 machine 在拉取后也应执行上面的一行，保持记录的插件 revision。
```

若只有普通 clone，没有执行 `submodule update`，该目录可能只是空目录，DSH 会识别不到本地插件包。此时应补齐上层子模块记录指向的源码，不要把这里误当成取消“无 specs 目录”约定的理由。
源码恢复后，`package.json` 应存在，版本应为当时的补丁版，例如 `0.32.1-reveal.1`。
本插件自己的远端以 `git remote -v` 和恢复时记录为准；未经用户确认不要手工强覆盖到第三方仓最新 `dev` / `main` —— 那样会把本地 patch 改掉。

### B. 安装兼容 DSH 和完整开发依赖

**普通约定是先停在已验证版本、再根据已知迁移说明有计划升级，不要自动取 npm latest。**
官方插件 `0.32.1` 要求 DSH `0.2.2-rc.2+`，当前组合使用的源码插件构建锁文件基于 rc.2 系列类型声明，实际运行宿主为 rc.4。
适合的修复件版本可随 patch 明确提升；不要同时将“官方升级”和“给定位新增功能”混为一次无可回退的操作。

```sh
cd "$PLUGIN"
pnpm install --frozen-lockfile
pnpm typecheck
pnpm lint
pnpm test
pnpm build
pnpm check:consumer-types
```

需要 Node.js 满足插件和 DSH 的要求；常见 npm 的商店位置/ pnpm 版本报错时沿用本机 `work-summary` 已登记的 pnpm11 路径，不要在 profile 目录手改 `package.json`、`pnpm-lock.yaml`、`cordis.patch.yml` 或直接跑 pnpm。

注意本机 Git hook（如强制提交格式 `[to #需求id]`）可能使此上游测试的临时 Git commit 失败；这不是定位交互测试结果。应在**不绕过/改写生产仓库 hooks**的前提下报告或隔离测试的定义问题。当前验证了类型检查、构建、consumer 检查、lint 以及定位相关的 31 项测试，没有声称基础插件的全量测试都是绿色。

### C. 安装本目录的补丁插件（不要重装整个 profile）

用以下三行覆盖同一 bundle 的**引用版本或路径**；插件已启用时先停用它，移除的是 bundle 包引用，不是到源码目录删除文件：

1. 插件管理：`set_bundle` → `target=dsh-better-sidebar` → `enabled=false`；
2. 插件管理：`remove_bundle` → `target=dsh-better-sidebar`；
3. 插件管理：`install_bundle` → `target=$PLUGIN`；当管理器提示已启用但未移动或不是 `applied` 时，先按 `nextStep` 处理，**不要放弃到 specs 恢复旧链接**。

也可以打开 DSH Web UI 的 **插件 → 添加插件**，选择本目录；已安装同 id 时需要先停用/移除其官方远程安装的引用。不要叠加安装无语义差别的重复行。检查 profile 中实际链接：

```sh
readlink "$HOME/.dsh/profiles/web/node_modules/dsh-better-sidebar"
```

应指向这个 operations 路径。**本机 profile 本身不保证随 work-summary 同步**，迁移机器时必须执行插件安装行，而不能只复制 `work-summary` 然后就声称插件已使用补丁。

### D. 恢复服务与验收

沿用已经安装的 `com.mumu.dsh-web` / `dsh-web-service` 原服务，不另起抢占 `:3080` 的 Web。确认本机启动器监听 `127.0.0.1:3080`。插件从 npm `node_modules` 目录切换为本地目录时通常要求 DSH 进程重启；使用本机 launchd 重载，或安全停止原进程再由 launchd 拉回，不要为了让改动生效额外开第二大宿主。

```sh
launchctl kickstart -k "gui/$(id -u)/com.mumu.dsh-web"
lsof -nP -iTCP:3080 -sTCP:LISTEN
```

readlink 中的链接与进程读取的适配进程如上，确认新版适用于恢复机再让普通用户提供当前会话内容 unattended demo。验收步骤：

1. 新建无任务会话，不填写任何模型提示；从右上角文件 → 文件树打开多层嵌套的 `.md`。
2. 点击文件路径旁的靶心按钮 `◎ 在目录树中定位当前文件`。
3. 确认目录自动展开，选中的为当前 **精确路径** 的文件而非祖先目录，目标进入视口中央。
4. 树搜索先填入不匹配词，点击后应清空搜索且仍能定位。
5. 再用键盘 Tab / Enter 激活，焦点保留在可访问按钮上，键盘 Enter 不能有点击丢失焦点或重复动作。
6. 用 `readlink`（旧源码目录不可恢复为非记录路径）再确认看到的是新路径补丁。

验收旧截图：[current Markdown reveal](./evidence-current-file-reveal-final.png)。意图是说选中文件位于视口中央，而不是只根据截图判断祖先目录已经展开。

## 升级与维护注意事项

- Unreleased patch 放在其自己的插件 checkout 中有版本化状态，不继续放 specs，也不手写 profile state 取悦恢复脚本。
- 原上游定位链请从文件工具栏开始：`EditorHost` 根据 `scope.cwd` 和文件绝对路径发起定位；原生右侧编辑器标签通过 `native/tab-adapter` 定位当前那个窗口的树；`TreePanel` 对这次定位清空搜索；`FileTree` 等真正行出来后选择当前文件并在树容器居中。没有后台 agent 指令或新工具。
- **换上游官方版本可能主动移除此补丁。** 想升级官方插件先记录正在使用的补丁提交/revision，读取新版本进度/migration，取新版本源码后重新应用 `reveal` 小 diff，而不是安装 latest 就声称补丁还在。需要区分宿主 DSH 版本、插件 peer 要求和具体 `.d.ts` 构建锁目录；不要为旧宿主强开版本豁免。
- 补丁随当前个人 fork 源码分支和父仓的 revision 迁移。不得直接向插件作者上游推送本机功能及换机文档。currentUser 当前是 GitHub `zhangxy-zju`。
- 本 README 不承诺线上业务 CR、生成资产或 app workflow 这些处于另域的发布流程，它是只为本机操作的本地 DeepSeek 插件补丁。

## 恢复验证记录

于 2026-11-18 DSH rc.4、插件 0.32.1-reveal.1、macOS 本机的 existing `http://127.0.0.1:3080` 上实测：深目录定位至文件 `summary.md`；`data-dsh-selected=true`，文件仅在此目标而不是祖先目录上选中；目标行 top 461px（视口 830px）且视口 scrollTop 6327px；第二次定位和带搜索清空定位验证；键盘定位保留焦点 。源码目录移动到本 operations 目录后重新安装新链接的验证以当前报告为准，只引用最终产物构建通过，不编造多机测试。
