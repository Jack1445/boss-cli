# Boss URL 与功能对照

本文件记录 boss-cli 已知页面 URL、对应命令和当前位置要求。修改页面导航、命令入口或报错文案前，先对照这里，避免把某个命令改成隐式跳转或多路径兜底。

## 页面 URL

| URL / Path | 页面功能 | 相关命令 | 当前行为 |
| --- | --- | --- | --- |
| `https://www.zhipin.com/web/user/?ka=header-login` | 登录入口 | `boss login` | 直接打开登录页，交还给人工扫码/验证。 |
| `/web/chat/index` | 沟通列表、候选人会话页 | `boss list`、`boss list --unread`、`boss chat <姓名>`、`boss chat [姓名] --index <序号> [--unread]`、`boss send`、`boss action` | `list` / `chat` 会先检查当前位置，不在该页时直接进入；`chat --index` 使用 `boss list` 输出的 1-based 序号，带 `--unread` 时使用 `boss list --unread` 输出的序号；同时提供姓名时会校验序号对应候选人姓名，适合处理同名候选人；`send` / `action` 要求当前会话已打开。 |
| `/web/chat/recommend` | 推荐候选人列表 | `boss recommend`、`boss preview <姓名>`、`boss greet <姓名>` | `recommend` 会先检查当前位置，不在该页时直接进入；`preview` / `greet` 要求当前已在该页且列表已加载。 |
| `/web/chat/aiform` | 深度搜索 / Agent 搜索 | `boss deep-search [--core <要求>] [--bonus <加分项>] [--clear-core] [--clear-bonus] [--match]`、`boss preview <姓名>`、`boss greet <姓名>` | `deep-search` 会先检查当前位置，不在该页时直接进入；默认只输出招聘要求表单、核心要求、加分项、今日剩余匹配次数和按钮状态，不输出候选列表；`--core` / `--bonus` 可重复，并按传入列表同步对应分组（多余行会删除，不足会新增），`--clear-core` / `--clear-bonus` 清空对应分组；只有 `--match` 会消耗今日匹配次数并输出列表顶部最新 20 条；`preview` / `greet` 要求当前已在该页且列表已加载。 |
| `/web/chat/search` | 常规搜索 | `boss search [关键词]`、`boss preview <姓名>` | `search` 会先检查当前位置，不在该页时直接进入；带关键词时填入搜索框并回车搜索；`preview` 要求当前已在该页且列表已加载，不支持 `--job`。 |
| `/web/chat/job/list` | 职位管理列表（内容位于 `/web/frame/job_v2/list` iframe） | `boss positions`、`boss jd <name>` | 命令自动点击侧栏“职位管理”，等待 job_v2 iframe 加载后读取。 |

## 约定

- 当前 URL 校验优先于 DOM 猜测。命令依赖特定页面时，先检查 path，再执行页面读取或点击。
- 不为高风险操作添加隐式兜底跳转。需要用户处于特定页面的命令，应直接报出当前 URL 与目标 URL。
- 页面入口发生变化时，同步更新本文件和相关命令 help 文案。

## 本地耗时诊断

设置 `BOSS_CLI_TIMING=true` 后，CLI 在 stderr 输出以 `[boss-cli][timing]` 开头的 JSON 事件，包含步骤名、开始/完成/失败状态、时间、PID、span_id 和耗时，不包含候选人正文。记录覆盖命令前的 npm 版本检查、会话锁、浏览器初始化、岗位与沟通页准备、列表提取及命令清理。计时不会改变上述命令的导航位置要求或等待策略；stdout 数据格式保持原样。

命令清理时除断开 CDP 外，还释放本次自行启动 Chrome 的 stdout/stderr 管道和子进程引用；不会关闭 Chrome。连接模式下 `browser.process()` 为 null，不能依靠它清理自行 spawn 的进程。回归验证：构建后运行 `node scripts/test-browser-detach.mjs`，使用独立无登录配置验证 CLI 退出而 Chrome 继续运行。
