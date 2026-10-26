# 四象限待办 · Quadrant Memo

一个放在桌面上的四象限待办备忘录：按「重要 / 紧急」自动分区，可以折叠成一条小横条，也能让 AI 通过本地接口直接读取、检索和写入。

![浅色界面](docs/screenshot-light.png)

![深色界面](docs/screenshot-dark.png)

![收纳后的样子](docs/screenshot-collapsed.png)

![智能检索](docs/screenshot-search.png)

![新建待办](docs/screenshot-dialog.png)

![日期选择与「每天」](docs/screenshot-calendar.png)

- **本地优先**：数据只存在本机 SQLite，不需要登录、不联网。
- **四象限**：重要且紧急 / 重要不紧急 / 紧急不重要 / 不重要不紧急。
- **可收纳**：点右上角收纳按钮变成 470×60 的小条，双击可展开。
- **能检索**：标题、备注、标签一起搜，支持 `#标签`、`q:1`、`is:done` 等语法。
- **轻量向量检索**：本地字符 n-gram 向量，持久化在 SQLite；「智能检索」把关键词和向量相似度混合排序，不需要安装 Chroma、numpy 或模型文件。
- **AI 接口**：本机 REST API + 命令行，两种接入方式；带 OpenAPI 和接口发现文件。
- **零第三方依赖**：Python + Tkinter + SQLite，不装任何 pip 包。

## 启动

双击项目目录里的 **`启动四象限待办.vbs`**，或桌面上的 **「四象限待办」** 快捷方式。

也可以在终端运行：

```powershell
cd <项目目录>
py app.py
```

首次启动会自动创建数据库和接口配置，不会弹教程，直接可用。

## 怎么用

| 操作 | 方式 |
| --- | --- |
| 新建 | 点右上角「新建待办」或卡片右上角 `+`，`Ctrl+N` |
| 编辑 | 点待办标题，或行尾铅笔图标 |
| 完成 / 恢复 | 点左侧圆圈；右键可切换 |
| 移动象限 | 右键 → 移动到 → Q1–Q4 |
| 删除 | 行尾删除图标，或编辑弹窗左下角 |
| 快速检索 | 顶部搜索框，`Ctrl+F` |
| 智能检索 | 筛选栏「智能检索」：关键词 + 本地向量混合排序；再点一次切回精确 |
| 筛选 | 全部 / 进行中 / 今天 / 逾期 / 已完成 |
| 收纳 | 标题栏收纳按钮，或 `Ctrl+Shift+M` |
| 置顶 | 标题栏图钉按钮 |
| 切换深浅色 | 标题栏月亮 / 太阳按钮 |
| 退出 | 标题栏 `×` |

窗口可以拖动；右下角有缩放把手。

### 检索语法

搜索框和 API 的 `q` 参数共用一套规则：

| 写法 | 含义 |
| --- | --- |
| `汇报` | 标题 / 备注 / 标签包含「汇报」 |
| `汇报 数据` | 两个词都要包含（AND） |
| `#工作` | 带「工作」标签 |
| `q:1` | 只看 Q1；`q:2`…`q:4` 同理 |
| `is:active` / `is:done` | 只看进行中 / 已完成 |
| `due:today` / `due:overdue` / `due:week` | 今天 / 逾期 / 未来 7 天 |
| `source:ai` | AI 写入的待办 |
| `-草稿` | 排除含「草稿」的条目 |

输入搜索词时会自动切到「全部」范围（包含已完成），清空后恢复原来的筛选。

### 向量检索

「智能检索」开启时（默认），搜索会同时计算：

- 关键词匹配分：标题 > 标签 > 备注；
- 本地向量余弦相似度：字符 1/2/3-gram + 英文词 + 标签 + 备注，384 维，特征哈希；
- 混合排序：`0.58 × 关键词归一化分 + 0.42 × 向量相似度`。

向量保存在同一个 SQLite 的 `task_vectors` 表中，任务新增/修改会自动更新。可通过 API 的 `/api/embeddings/status` 查看，`/api/embeddings/rebuild` 重建。默认实现不依赖 Chroma / numpy / 外部模型；如果以后想换成 Ollama 或云端 embedding，只需要替换 `qm/vector.py` 的 `embed_text` / `embed_task`。

精确关键词检索仍可通过「智能检索」按钮关闭，或 API 传 `mode=keyword`。

### 截止日期

点击日期输入框会弹出月历（默认显示电脑当前月份，今天有高亮），可以直接点选日期；也可以输入：

`今天`、`明天`、`后天`、`周五`、`下周一`、`3天后`、`2周后`、
`2026-10-30`、`10-30`、`10月30日`。

最下面的 **「每天」** 用于循环待办：任务会标记为「每天」，不会计入逾期，会出现在「今天」和本周范围里；当天勾选完成后，第二天会自动恢复为待办。

**「常驻」** 用于长期任务：没有截止日期，不参与今天/逾期筛选，并会排在该象限的最前面，适合长期目标或随时要看的备忘。

### 快捷键

`Ctrl+N` 新建 · `Ctrl+F` 搜索 · `Ctrl+Shift+M` 收纳 / 展开 · `F5` 刷新 · `Esc` 清空搜索

## AI 接口

应用启动后会同时在本机开启接口（默认 `http://127.0.0.1:8765`），窗口底部和标题栏会显示状态。

- 接口文档：`http://127.0.0.1:8765/docs`
- OpenAPI：`http://127.0.0.1:8765/openapi.json`
- 接口发现文件：`%LOCALAPPDATA%\QuadrantMemo\endpoint.json`
- 数据文件：`%LOCALAPPDATA%\QuadrantMemo\tasks.db`

AI 只要读取 `endpoint.json` 里的 `base_url`，就能直接访问；也可以直接让 AI 看 [AI_API.md](AI_API.md)。

```powershell
# 看看接口是否在线
curl.exe http://127.0.0.1:8765/api/health

# 检索
curl.exe "http://127.0.0.1:8765/api/search?q=汇报"

# 向量 / 混合检索（适合自然语言）
curl.exe "http://127.0.0.1:8765/api/semantic_search?q=工作&limit=10"

# 新建（source 会记为 ai）
curl.exe -X POST http://127.0.0.1:8765/api/tasks `
  -H "Content-Type: application/json" `
  -d "{\"title\":\"整理季度汇报\",\"quadrant\":1,\"due_date\":\"明天\",\"tags\":[\"工作\"]}"
```

如果只想让接口运行、不开窗口：

```powershell
py app.py --headless
```

不启动接口：`py app.py --no-api`。

### 命令行（无 GUI 也能用）

```powershell
py qm_cli.py list
py qm_cli.py search 汇报 --json
py qm_cli.py search 工作 --mode hybrid
py qm_cli.py add "整理季度汇报" -q 1 --due 明天 --tags 工作,汇报
py qm_cli.py add "每日复盘" -q 2 --repeat daily --tags 习惯
py qm_cli.py done 12
py qm_cli.py reindex
py qm_cli.py stats
```

`qm_cli.py` 直接读写同一个 SQLite，不依赖 GUI 和 HTTP 服务。

## 数据与安全

- 数据库：`%LOCALAPPDATA%\QuadrantMemo\tasks.db`
- 配置：`%LOCALAPPDATA%\QuadrantMemo\config.json`
- 接口默认只监听 `127.0.0.1`，不对局域网开放。
- 默认不需要 Token；如有需要，启动前设置环境变量 `QM_TOKEN`，之后请求带上 `X-API-Token`。
- 备份：复制 `tasks.db` 即可；也可以用 `py qm_cli.py export --out backup.json`。
- 卸载：退出应用，删除程序目录；如需彻底清除数据，再删除 `%LOCALAPPDATA%\QuadrantMemo`。

## 文件结构

```
quadrant_memo/
├─ app.py                  # 启动入口（GUI / --headless）
├─ qm_cli.py               # 命令行接口
├─ 启动四象限待办.vbs       # 双击静默启动
├─ 启动（调试）.cmd
├─ qm/
│  ├─ repository.py        # SQLite 数据层、检索、统计
│  ├─ vector.py            # 本地轻量向量索引（SQLite 持久化）
│  ├─ api.py               # 本地 REST API + OpenAPI + 文档页
│  ├─ ui.py                # 桌面界面
│  ├─ dialogs.py           # 新建 / 编辑 / 确认弹窗
│  ├─ datepicker.py        # 月历、今天/明天/每天/清除
│  ├─ widgets.py           # 圆角卡片、搜索框等自绘控件
│  ├─ theme.py             # 明暗配色
│  └─ ...
├─ assets/icon.ico
├─ tests/                  # 数据层与 API 测试
├─ AI_API.md               # 给 AI 的接口说明
└─ README.md
```

## 常见问题

**接口地址不是 8765？**  
端口被占用时会自动顺延到 8766、8767…… 以 `endpoint.json` 或窗口底部显示的地址为准。

**双击 VBS 没反应？**  
运行 `启动（调试）.cmd` 看报错；确认 Python 3.10+ 已安装且带 Tkinter（官方安装包默认包含）。

**同时开两个窗口？**  
程序做了单实例限制，第二次启动会把已有窗口带到前台。

**想换电脑？**  
复制整个程序目录，再把 `%LOCALAPPDATA%\QuadrantMemo\tasks.db` 放到新电脑同一位置即可。
