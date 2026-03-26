# DOSBox-X Launcher — 产品需求文档（PRD）

> **版本**：MVP  
> **日期**：2026-03-26  
> **状态**：草稿

---

## 1. 产品概述

### 1.1 产品定位

DOSBox-X Launcher 是一个基于 Python 3.10 + PyQt5 的跨平台桌面应用，作为 DOSBox-X 的图形化前端。它让用户方便地管理 DOS 游戏库、快速配置并启动游戏，无需手动编写 `.conf` 文件。

### 1.2 目标用户

- DOS 游戏爱好者 / 怀旧玩家
- 需要管理多个 DOS 游戏的用户
- 不熟悉 DOSBox-X 命令行配置的用户

### 1.3 MVP 目标

提供一个**可用的**基础版本，具备完整的游戏库增删改查、可视化列表浏览、DOSBox-X 配置管理以及一键启动能力。

### 1.4 技术栈

| 项目     | 选型            |
|----------|-----------------|
| 语言     | Python 3.10+    |
| UI 框架  | PyQt5 Widgets   |
| 数据存储 | JSON 文件       |
| 后端引擎 | DOSBox-X（外部进程，用户自行安装） |
| 支持平台 | Windows / Linux / macOS |

---

## 2. 用户故事

### 2.1 游戏库管理

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-01 | 作为用户，我想要添加一个 DOS 游戏到游戏库，以便统一管理我的游戏。 | P0 |
| US-02 | 作为用户，我想要编辑已有游戏的信息（名称、分类、可执行文件路径等），以便修正或更新游戏资料。 | P0 |
| US-03 | 作为用户，我想要从游戏库中删除一个游戏条目，且不会删除磁盘上的游戏文件，以免误删数据。 | P0 |
| US-04 | 作为用户，我想要为游戏指定封面图（本地图片），导入后自动拷贝到数据目录统一管理，以便在列表中快速识别游戏。 | P1 |
| US-05 | 作为用户，我想要在没有指定封面图时看到一个默认占位图，以保持界面整洁。 | P1 |

### 2.2 分类管理

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-06 | 作为用户，我想要创建自定义游戏分类（如"角色扮演""即时战略"），以便组织我的游戏库。 | P0 |
| US-07 | 作为用户，我想要编辑和删除自定义分类，以便灵活调整分类体系。 | P0 |
| US-08 | 作为用户，我想要有一个始终存在且不可删除的"All"分类，以便随时查看全部游戏。 | P0 |
| US-09 | 作为用户，我想要通过拖拽调整分类的显示顺序，以便按我的习惯排列。 | P1 |

### 2.3 游戏浏览

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-10 | 作为用户，我想要在列表视图中看到每个游戏的封面缩略图、名称、分类、最近游玩时间和游玩次数。 | P0 |
| US-11 | 作为用户，我想要点击某个游戏后在详情面板中查看其详细信息（大封面图、名称、分类、游玩数据）。 | P0 |
| US-12 | 作为用户，我想要在状态栏中看到游戏总数和最近游玩的游戏名称。 | P1 |

### 2.4 DOSBox-X 配置

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-13 | 作为用户，我想要设定全局默认 DOSBox-X 配置参数，以便对所有游戏生效。 | P0 |
| US-14 | 作为用户，我想要为单个游戏设定差异化的 DOSBox-X 配置，使其覆盖全局默认值，以满足特定游戏的需要。 | P0 |
| US-15 | 作为用户，我想要通过一个配置对话框来修改这些 DOSBox-X 参数，而不是手动编辑 `.conf` 文件。 | P0 |

### 2.5 一键启动

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-16 | 作为用户，我想要选中一个游戏后点击"启动"按钮，即可自动启动 DOSBox-X 运行该游戏。 | P0 |
| US-17 | 作为用户，我想要启动器自动合并全局配置与游戏配置并生成临时 `.conf` 文件传递给 DOSBox-X，无需我手动处理。 | P0 |
| US-18 | 作为用户，我想要每次游玩后自动记录游玩次数和最近游玩时间，以便追踪游戏历程。 | P0 |

### 2.6 全局设置

| 编号 | 用户故事 | 优先级 |
|------|---------|--------|
| US-19 | 作为用户，我想要在设置中指定 DOSBox-X 可执行文件的路径，以便启动器能正确调用它。 | P0 |
| US-20 | 作为用户，我想要在设置中指定游戏库的根目录路径。 | P1 |
| US-21 | 作为用户，我想要在设置中指定截图输出目录路径。 | P1 |

---

## 3. 功能需求

### 3.1 游戏库管理（F-01）

#### 3.1.1 添加游戏

- 用户通过工具栏"Add"按钮或菜单 `File > Add Game` 打开添加游戏对话框。
- 对话框包含以下字段：
  - **Game Name**（游戏名称）：必填，文本输入。
  - **Executable Path**（可执行文件路径）：必填，支持文件浏览器选择 `.exe`、`.com`、`.bat` 文件。
  - **Game Directory**（游戏目录）：必填，自动由可执行文件路径推导，也可手动修改。
  - **Category**（分类）：必填，下拉选择已有分类。
  - **Cover Image**（封面图）：可选，支持文件浏览器选择本地图片（`.png`、`.jpg`、`.bmp`）。
  - **Description**（描述）：可选，多行文本输入。
- 点击"OK"后：
  1. 为新游戏分配一个自增数字 ID。
  2. 如果用户指定了封面图，将原图拷贝到数据目录的 `cover/{id}.jpg`（自动转换格式为 JPEG）。
  3. 新游戏条目写入 `games.json`。
- 点击"Cancel"放弃操作。

#### 3.1.2 编辑游戏

- 用户选中游戏后通过工具栏"Edit"按钮、右键菜单或菜单 `Edit > Edit Game` 打开编辑对话框。
- 对话框与添加对话框相同，但预填当前游戏信息。
- 如果用户更换了封面图，将新图拷贝覆盖到 `cover/{id}.jpg`。
- 修改确认后更新 `games.json` 中对应条目。

#### 3.1.3 删除游戏

- 用户选中游戏后通过工具栏"Delete"按钮、右键菜单或菜单 `Edit > Delete Game` 触发删除。
- 弹出确认对话框：`"Are you sure you want to remove '<game name>' from the library? This will NOT delete any files on disk."`
- 确认后从 `games.json` 中移除条目，同时删除数据目录中对应的 `cover/{id}.jpg` 封面文件（如存在）。磁盘上的游戏文件不做任何处理。

### 3.2 分类管理（F-02）

- 分类数据保存在 `config.json` 的 `categories` 字段中。
- 内置一个不可删除的 `"All"` 分类，在列表最顶部固定显示。
- 用户可通过设置或菜单打开分类管理界面：
  - 新增分类：输入分类名称。
  - 编辑分类：修改分类名称，已关联的游戏自动更新。
  - 删除分类：删除后，原属该分类的游戏归入 `"Uncategorized"`（或用户指定的分类）。
- 支持拖拽调整分类的显示顺序（P1）。

### 3.3 游戏列表视图（F-03）

- 主窗口中部以表格形式展示游戏列表。
- 列定义：

| 列       | 内容                     | 宽度策略     |
|----------|--------------------------|-------------|
| Cover    | 封面缩略图（固定尺寸）     | 固定宽度     |
| Name     | 游戏名称                  | 拉伸填充     |
| Category | 所属分类                  | 固定宽度     |
| Last Played | 最近游玩日期（`YYYY-MM-DD` 或 `—`） | 固定宽度 |
| Play Count  | 游玩次数                 | 固定宽度     |

- 点击列头可按该列排序（升序/降序切换）。
- 选中一行时，详情面板同步更新。
- 双击游戏行等同于点击"Launch"按钮，直接启动游戏。

### 3.4 游戏详情面板（F-04）

- 位于游戏列表下方。
- 展示选中游戏的详细信息：
  - 封面大图（左侧）。
  - 游戏名称、分类、游玩次数、最近游玩时间（右侧上方）。
  - 游戏描述文本（右侧下方）。
- 未选中任何游戏时显示空白或提示文字。

### 3.5 DOSBox-X 配置管理（F-05）

> **配置文件规格参考**：DOSBox-X 配置文件（`.conf`）的完整规格见项目目录下的：
> - `docs/dosbox-x.reference.conf` — 常用配置项及说明。
> - `docs/dosbox-x.reference.full.conf` — 包含所有高级选项的完整参考。
>
> 配置文件采用 INI 格式，分为多个 `[section]`，每个 section 下有若干 `key = value` 形式的配置项。

#### 3.5.1 全局默认配置

- 保存在 `config.json` 的 `dosbox_defaults` 字段中。
- 提供一个配置对话框，以分组方式展示常用 DOSBox-X 配置项的可视化编辑。
- 修改后保存到 `config.json`。
- MVP 阶段需要在配置对话框中暴露的关键配置项如下：

| Section     | 配置项       | 类型       | 默认值       | 说明                                                 |
|-------------|-------------|-----------|-------------|------------------------------------------------------|
| `[dosbox]`  | `machine`   | 下拉选择   | `svga_s3`   | 模拟的机器类型（影响显卡/显示模式）。常用值：`svga_s3`、`vgaonly`、`tandy`、`cga`、`hercules` 等 |
| `[dosbox]`  | `memsize`   | 整数输入   | `16`        | 模拟内存大小（MB）。部分游戏需要特定内存大小             |
| `[cpu]`     | `core`      | 下拉选择   | `auto`      | CPU 核心类型。可选 `auto`、`dynamic`、`normal`、`simple` 等 |
| `[cpu]`     | `cputype`   | 下拉选择   | `auto`      | CPU 类型。可选 `auto`、`386`、`486`、`pentium` 等       |
| `[cpu]`     | `cycles`    | 文本输入   | `auto`      | 模拟 CPU 速度。`auto` 自动检测；`fixed <数字>` 固定速度（如 `fixed 5000`）；`max` 最大速度 |
| `[sdl]`     | `fullscreen`| 布尔开关   | `false`     | 是否以全屏模式启动 DOSBox-X                            |
| `[sdl]`     | `output`    | 下拉选择   | `default`   | 视频输出模式。可选 `surface`、`opengl`、`direct3d` 等    |
| `[render]`  | `aspect`    | 布尔开关   | `false`     | 是否启用宽高比矫正（推荐对 4:3 游戏启用）               |
| `[render]`  | `scaler`    | 下拉选择   | `normal2x`  | 画面缩放算法。可选 `none`、`normal2x`、`normal3x`、`hq2x`、`hq3x` 等 |
| `[sblaster]`| `sbtype`    | 下拉选择   | `sb16`      | Sound Blaster 声卡类型。可选 `sb1`、`sb2`、`sbpro1`、`sbpro2`、`sb16`、`none` 等 |
| `[sblaster]`| `sbbase`    | 下拉选择   | `220`       | Sound Blaster I/O 基地址                               |
| `[sblaster]`| `irq`       | 下拉选择   | `7`         | Sound Blaster IRQ 中断号                               |
| `[sblaster]`| `dma`       | 下拉选择   | `1`         | Sound Blaster DMA 通道号                               |
| `[midi]`    | `mpu401`    | 下拉选择   | `intelligent` | MPU-401 类型。可选 `intelligent`、`uart`、`none`      |
| `[midi]`    | `mididevice`| 下拉选择   | `default`   | MIDI 输出设备                                          |
| `[gus]`     | `gus`       | 布尔开关   | `false`     | 是否启用 Gravis Ultrasound 声卡模拟                    |
| `[dos]`     | `xms`       | 布尔开关   | `true`      | 是否启用 XMS 扩展内存                                  |
| `[dos]`     | `ems`       | 布尔开关   | `true`      | 是否启用 EMS 扩展内存                                  |
| `[dos]`     | `umb`       | 布尔开关   | `true`      | 是否启用 UMB 上位内存                                  |
| `[dos]`     | `keyboardlayout` | 文本输入 | `auto`   | 键盘布局语言代码                                       |

- 对话框中可按 section 分组（如"Display"、"CPU"、"Sound"、"DOS"），便于用户查找。
- 每个配置项旁可显示简短提示说明。

#### 3.5.2 游戏独立配置

- 保存在 `games.json` 中每个游戏条目的 `dosbox_overrides` 字段。
- 使用继承机制：游戏配置仅存储与全局配置不同的部分。
- 在游戏编辑对话框中提供"DOSBox Config"选项卡，列出与全局配置对话框相同的可覆盖配置项。
- 未覆盖的项显示全局默认值（灰色/斜体标识），覆盖的项以正常样式显示。

#### 3.5.3 配置合并与生成

- 启动游戏时，系统将全局配置与游戏独立配置合并。
- 游戏配置优先级高于全局配置（覆盖策略）。
- 合并结果按 DOSBox-X `.conf` 文件的 INI 格式（`[section]` + `key = value`）写入系统临时目录。
- 生成的 `.conf` 文件末尾自动附加 `[autoexec]` 段，用于挂载游戏目录并启动游戏可执行文件，示例：
  ```ini
  [autoexec]
  mount c "/path/to/game_dir"
  c:
  PAL.EXE
  exit
  ```
- 临时文件在 DOSBox-X 进程结束后清理。

### 3.6 一键启动（F-06）

- 用户选中游戏后，通过以下方式触发启动：
  - 工具栏"Launch"按钮。
  - 菜单 `File > Launch Game`。
  - 游戏列表双击。
  - 右键菜单"Launch"。
- 启动流程：
  1. 校验 DOSBox-X 可执行文件路径是否有效。
  2. 校验游戏可执行文件路径是否存在。
  3. 合并全局配置与游戏配置，生成临时 `.conf` 文件。
  4. 通过 `subprocess` 调用 DOSBox-X，传入临时 `.conf` 文件。
  5. 更新游戏的游玩次数（+1）和最近游玩时间（当前时间戳）。
  6. 将更新后的数据保存到 `games.json`。
- 校验失败时弹出错误对话框，提示具体原因（如 "DOSBox-X executable not found at: ..."）。

### 3.7 全局设置（F-07）

- 通过工具栏"Settings"按钮或菜单 `Edit > Settings` 打开全局设置对话框。
- 设置项：

| 设置项            | 说明                                 | 类型         |
|-------------------|--------------------------------------|-------------|
| DOSBox-X Path     | DOSBox-X 可执行文件路径。默认值为项目根目录下的 `dosbox/dosbox.exe`（该文件虽名为 `dosbox.exe`，实际为 DOSBox-X）。用户可修改为其他路径。 | 文件路径选择 |
| Game Library Path | 游戏库根目录路径                      | 目录路径选择 |
| Screenshot Path   | DOSBox-X 截图输出目录路径（P1）        | 目录路径选择 |

- 设置保存在 `config.json` 中。
- 修改保存后立即生效，无需重启应用。

### 3.8 菜单栏（F-08）

```
File
  ├── Add Game          添加游戏
  ├── Launch Game       启动游戏（选中游戏时可用）
  ├── ─────────
  └── Exit              退出应用

Edit
  ├── Edit Game         编辑游戏（选中游戏时可用）
  ├── Delete Game       删除游戏（选中游戏时可用）
  ├── ─────────
  ├── Categories        管理分类
  └── Settings          全局设置

Help
  └── About             关于对话框
```

### 3.9 工具栏（F-09）

从左到右依次排列以下按钮：

| 按钮     | 动作            | 启用条件       |
|----------|----------------|---------------|
| Add      | 打开添加游戏对话框 | 始终可用       |
| Edit     | 打开编辑游戏对话框 | 选中游戏时可用 |
| Delete   | 删除选中游戏     | 选中游戏时可用 |
| Launch   | 启动选中游戏     | 选中游戏时可用 |

工具栏右侧区域：

| 控件     | 说明                              |
|----------|----------------------------------|
| Search   | 实时搜索过滤游戏列表（按名称匹配） |
| Sort     | 下拉菜单选择排序方式               |
| Settings | 打开全局设置对话框                 |

### 3.10 状态栏（F-10）

- 左侧显示：游戏总数，格式为 `"X game(s)"`。
- 右侧显示：最近游玩的游戏名称，格式为 `"Last played: <name>"`；如果从未游玩过任何游戏则不显示。

---

## 4. 数据模型

### 4.1 数据存储位置

- **默认路径**：`~/.config/dosboxlauncher/`
- **便携模式**：如果项目根目录下存在 `portable/` 目录，则以该目录作为数据目录。
- 便携模式优先级高于默认路径。
- 数据目录下包含以下内容：
  - `config.json` — 全局配置文件。
  - `games.json` — 游戏列表数据文件。
  - `cover/` — 封面图存储目录，文件名为 `{id}.jpg`（`id` 为游戏的数字 ID）。

### 4.2 config.json

```json
{
  "dosbox_path": "dosbox/dosbox.exe",
  "game_library_path": "/path/to/games",
  "screenshot_path": "/path/to/screenshots",
  "categories": [
    {"id": "cat-001", "name": "RPG", "order": 0},
    {"id": "cat-002", "name": "Strategy", "order": 1}
  ],
  "dosbox_defaults": {
    "dosbox.machine": "svga_s3",
    "dosbox.memsize": 16,
    "cpu.cycles": "auto",
    "cpu.core": "auto",
    "sdl.fullscreen": false,
    "render.aspect": false,
    "sblaster.sbtype": "sb16",
    "dos.xms": true,
    "dos.ems": true,
    "dos.umb": true
  }
}
```

### 4.3 games.json

```json
[
  {
    "id": 1,
    "name": "Legend of Sword and Fairy",
    "executable": "/path/to/PAL.EXE",
    "game_dir": "/path/to/pal/",
    "category_id": "cat-001",
    "description": "Classic Chinese RPG.",
    "play_count": 12,
    "last_played": "2026-03-01T14:30:00",
    "dosbox_overrides": {
      "cpu.cycles": "fixed 5000"
    },
    "created_at": "2026-01-15T10:00:00",
    "updated_at": "2026-03-01T14:30:00"
  }
]
```

> 封面图不存储路径字段，而是根据游戏 ID 约定存放在数据目录的 `cover/{id}.jpg`。程序加载时按此路径查找封面文件，不存在则显示默认占位图。

### 4.4 字段说明

#### Game 对象

| 字段             | 类型       | 必填 | 说明                                       |
|------------------|-----------|------|--------------------------------------------|
| id               | int       | 是   | 自增数字 ID，创建时自动分配（取当前最大 ID + 1） |
| name             | string    | 是   | 游戏名称                                    |
| executable       | string    | 是   | 游戏可执行文件绝对路径                       |
| game_dir         | string    | 是   | 游戏目录绝对路径                             |
| category_id      | string    | 是   | 所属分类 ID                                 |
| description      | string    | 否   | 游戏描述文本                                 |
| play_count       | int       | 是   | 游玩次数，默认 0                             |
| last_played      | string    | 否   | 最近游玩时间 ISO 8601 格式，未游玩时为 null   |
| dosbox_overrides  | object   | 否   | 游戏独立的 DOSBox-X 配置覆盖项               |
| created_at       | string    | 是   | 创建时间 ISO 8601 格式                       |
| updated_at       | string    | 是   | 最后更新时间 ISO 8601 格式                   |

> 注意：`cover_image` 字段已移除。封面图统一按 `cover/{id}.jpg` 约定路径存储，无需在 JSON 中记录。

#### Category 对象

| 字段   | 类型   | 必填 | 说明                                 |
|--------|-------|------|--------------------------------------|
| id     | string | 是   | 分类 ID，唯一标识                     |
| name   | string | 是   | 分类名称                             |
| order  | int    | 是   | 显示顺序，用于排序                    |

---

## 5. 界面布局

主窗口从上到下分为五个区域：

```
┌──────────────────────────────────────────────────────────────────┐
│ File  Edit  Help                                                │  ← Menu Bar
├──────────────────────────────────────────────────────────────────┤
│ [+Add]  [Edit]  [Delete]  [Launch]  |  [Search...]  | [Sort▼] [Settings] │  ← Toolbar
├──────────────────────────────────────────────────────────────────┤
│ Cover  Name                Category    Last Played    Play Count │
│ ──────────────────────────────────────────────────────────────── │
│ [img] Legend of Sword...   RPG         2026-03-01     12         │
│ [img] Command & Conquer    Strategy    2026-01-20     28         │  ← Game List
│ [img] KKND                 Strategy    —              2          │
│  ...                                                             │
├──────────────────────────────────────────────────────────────────┤
│                        Game Details Panel                         │
│  ┌──────────┐                                                    │
│  │          │  Legend of Sword and Fairy                          │
│  │  Cover   │  Category: RPG                                     │
│  │  Image   │  Play Count: 12  |  Last Played: 2026-03-01        │
│  │          │                                                    │
│  │          │  Classic Chinese RPG about Li Xiaoyao and Zhao      │
│  └──────────┘  Ling'er's adventure...                            │
├──────────────────────────────────────────────────────────────────┤
│ 42 game(s)  |  Last played: Legend of Sword and Fairy            │  ← Status Bar
└──────────────────────────────────────────────────────────────────┘
```

- 界面语言为**英文**。
- 封面缩略图在列表中统一尺寸显示。
- 详情面板高度可适应内容，但有最小高度保证布局稳定。

---

## 6. 交互细节

### 6.1 游戏列表交互

- **单击**：选中游戏，更新详情面板。
- **双击**：启动选中的游戏。
- **右键菜单**：显示上下文菜单，包含 Edit / Delete / Launch 选项。
- **列头点击**：按该列排序，再次点击切换升降序。

### 6.2 工具栏按钮状态

- Add 和 Settings 始终可用。
- Edit、Delete、Launch 仅在选中游戏时可用，未选中时禁用（灰色）。
- Search 为实时过滤输入框，随输入内容即时过滤游戏列表。
- Sort 下拉菜单提供以下排序选项：Name (A-Z) / Name (Z-A) / Last Played / Play Count。

### 6.3 分类筛选

- 分类列表可在侧边栏或工具栏下拉中展示（MVP 中可选择较简单的实现方式，如工具栏中的下拉选择器）。
- 选择某个分类后，游戏列表仅显示该分类下的游戏。
- 选择"All"则显示全部游戏。

### 6.4 错误提示

- 所有用户操作的错误均通过 `QMessageBox` 弹出对话框提示。
- 错误消息应明确指出问题原因和建议的解决方式。

---

## 7. 约束与假设

### 7.1 约束

- **仅支持 DOSBox-X**，不支持原版 DOSBox。启动器在启动游戏时只调用 DOSBox-X，不对原版 DOSBox 做任何兼容处理。
- DOSBox-X 由用户自行安装，启动器不负责 DOSBox-X 的下载或安装。
- 数据存储使用 JSON 文件，不引入数据库。
- 界面语言为英文。

### 7.2 假设

- 用户的 DOS 游戏文件已存放在本地磁盘上。
- 单用户场景，不考虑多用户并发访问同一数据文件。

---

## 8. 术语表

| 术语 | 定义 |
|------|------|
| DOSBox-X | 一个开源的 x86 模拟器，用于运行 DOS 程序和游戏。本项目仅支持 DOSBox-X，不支持原版 DOSBox。项目自带的可执行文件位于 `dosbox/dosbox.exe`，虽文件名为 `dosbox.exe` 但实际为 DOSBox-X |
| 游戏库（Game Library） | 用户在启动器中管理的所有 DOS 游戏条目的集合 |
| 全局配置（Global Config） | 适用于所有游戏的 DOSBox-X 默认配置参数 |
| 游戏配置覆盖（Game Overrides） | 单个游戏独立设定的 DOSBox-X 配置，优先级高于全局配置 |
| 便携模式（Portable Mode） | 项目根目录下存在 `portable/` 目录时，数据文件保存在该目录中而非用户主目录 |
| 临时 conf 文件 | 启动游戏时，由全局配置与游戏配置合并生成的一次性 DOSBox-X 配置文件 |
