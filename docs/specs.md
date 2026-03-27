# DOSBoxLauncher MVP Implementation Plan

## Context

DOSBoxLauncher 项目目前已完成 PRD (`docs/prd.md`) 和概念文档 (`docs/concept.md`)，资源文件（80+ 图标、DOSBox-X 可执行文件、便携数据目录）已就绪，但 **零 Python 源码**。本计划从零构建完整的 MVP 应用。

---

## Architecture

三层架构：Data -> Business Logic -> UI

```
UI Layer (PyQt5)
  main_window.py, game_table.py, detail_panel.py, dialogs/
Business Logic Layer
  config.py, profiles.py, launcher.py
Foundation
  constants.py, exceptions.py
  JSON files (config.json, games.json)
```

数据层模块无 UI 依赖，可独立单元测试。

---

## File Tree

```
DOSBoxLauncher/
├── startbox.py                          # 根入口脚本
├── pyproject.toml                       # 项目元数据、依赖、工具配置
├── dosboxlauncher/
│   ├── __init__.py                      # __version__, APP_NAME
│   ├── __main__.py                      # python -m dosboxlauncher
│   ├── constants.py                     # 常量、DOSBox 配置规格、图标映射
│   ├── exceptions.py                    # 自定义异常层级
│   ├── config.py                        # AppConfig 数据类 + 加载/保存
│   ├── profiles.py                      # GameProfile 数据类 + GameStore CRUD
│   ├── launcher.py                      # DOSBox-X 进程管理
│   └── ui/
│       ├── __init__.py
│       ├── app.py                       # QApplication 引导
│       ├── resources.py                 # 图标/封面路径解析、格式转换
│       ├── main_window.py              # 主窗口：菜单、工具栏、状态栏、布局
│       ├── game_table.py               # GameTableModel + GameTableView
│       ├── detail_panel.py             # 游戏详情面板
│       └── dialogs/
│           ├── __init__.py
│           ├── game_dialog.py           # 添加/编辑游戏对话框（2 tab）
│           ├── category_dialog.py       # 分类管理对话框
│           ├── settings_dialog.py       # 全局设置对话框
│           ├── dosbox_config_widget.py  # 可复用的 DOSBox 配置编辑器
│           └── about_dialog.py          # 关于对话框
├── tests/
│   ├── conftest.py                      # 共享 fixtures
│   ├── test_config.py
│   ├── test_profiles.py
│   ├── test_launcher.py
│   └── test_constants.py
└── assets/
    ├── icons/                           # (已存在)
    ├── app.ico                          # 需生成
    ├── app.png                          # 需生成
    └── default_cover.png               # 需生成
```

---

## Implementation Steps (按依赖顺序)

### Phase 0: Documentation

**Step 0.1 — 保存 spec 到项目**
- 将本计划文件保存到 `docs/specs.md`，作为项目技术规格文档。

### Phase 1: Foundation (无 UI)

**Step 1.1 — pyproject.toml**
- name=`dosboxlauncher`, version=`0.1.0`, requires-python=`>=3.10`
- dependencies: `PyQt5>=5.15`
- dev extras: `pytest`, `pytest-cov`, `pytest-qt`, `ruff`, `mypy`
- Ruff/mypy/pytest 工具配置

**Step 1.2 — Package Init**
- `dosboxlauncher/__init__.py`: `__version__`, `APP_NAME`
- `dosboxlauncher/__main__.py`: `main()` 函数（初期 stub）
- `startbox.py`: 简单调用 main()

**Step 1.3 — constants.py + exceptions.py**
- 常量：数据目录名、文件名、默认值
- `DOSBOX_CONFIG_SPEC`: 字典定义所有 DOSBox-X 配置项的元数据（section, key, label, widget_type, default, choices, tooltip）——UI 和数据层的单一事实来源
- `ICON_MAP`: 逻辑名 → 图标文件名映射
- 异常层级：`DOSBoxLauncherError` → `ConfigError`, `ProfileError`, `LaunchError`

**Step 1.4 — config.py**
- `Category` 数据类 (id, name, order)
- `AppConfig` 数据类 (dosbox_path, game_library_path, screenshot_path, categories, dosbox_defaults)
- `resolve_data_dir()`: portable/ 存在则用它，否则 ~/.config/dosboxlauncher/
- `load_config()` / `save_config()`: JSON 读写，不存在则创建默认
- `ensure_data_dir()`: 创建数据目录和 cover/ 子目录
- "All" 分类为虚拟分类，不存储在 JSON 中，UI 层注入

**Step 1.5 — profiles.py**
- `GameProfile` 数据类（id, name, executable, game_dir, category_id, description, play_count, last_played, dosbox_overrides, created_at, updated_at）
- `GameStore` 类：CRUD 操作 (load, save, add, update, delete, get, next_id)
- delete 时同时删除 cover/{id}.jpg
- JSON 写入使用临时文件 + os.replace() 确保原子性

**Step 1.6 — launcher.py**
- `DOSBoxLauncher` 类：
  - `validate_dosbox()`: 验证 DOSBox-X 路径
  - `validate_game()`: 验证游戏可执行文件
  - `merge_config()`: 合并全局配置与游戏覆盖
  - `generate_conf()`: 生成临时 .conf 文件（INI 格式 + [autoexec] 段）
  - `launch()`: 完整启动流程，返回 subprocess.Popen
  - `cleanup_conf()`: 清理临时文件

**Step 1.7 — tests/**
- `conftest.py`: tmp_data_dir fixture，sample_config/sample_game 工厂
- `test_config.py`: 测试数据目录解析、配置加载/保存、默认值创建
- `test_profiles.py`: 测试 CRUD、ID 生成、时间戳、封面清理
- `test_launcher.py`: 测试配置合并、conf 生成、验证错误、subprocess mock
- `test_constants.py`: 测试配置规格完整性

### Phase 2: UI Foundation

**Step 2.1 — ui/resources.py**
- `get_icon(name)` → QIcon（缓存机制）
- `get_app_icon()` → 应用图标
- `get_default_cover()` → 默认封面 QPixmap
- `get_cover_pixmap(game_id, data_dir)` → 封面或默认封面
- `convert_and_save_cover(source, dest)` → QPixmap 转 JPEG
- `assets_dir()` → assets/ 路径解析

**Step 2.2 — ui/game_table.py**
- `GameTableModel(QAbstractTableModel)`:
  - 5 列：Cover, Name, Category, Last Played, Play Count
  - set_games() / set_filter(text, category_id) / sort()
  - 封面缩略图通过 DecorationRole 显示
- `GameTableView(QTableView)`:
  - 单行选择、交替行色、列宽策略
  - 自定义 delegate 渲染封面缩略图
  - 右键上下文菜单（Edit/Delete/Launch）
  - game_selected / game_launched 信号

**Step 2.3 — ui/detail_panel.py**
- `GameDetailPanel(QFrame)`:
  - 水平布局：左侧封面大图（160x200），右侧信息
  - set_game() / clear()

**Step 2.4 — ui/dialogs/dosbox_config_widget.py**
- 根据 DOSBOX_CONFIG_SPEC 动态生成表单
- 5 个分组（QGroupBox）：DOSBox General, Display, CPU, Sound, DOS
- 每组使用 QFormLayout，根据 widget_type 生成对应控件
- 两种模式：global（直接编辑）/ override（有"覆盖"复选框，未覆盖项灰色显示）
- get_values() / set_values() / set_defaults()

**Step 2.5 — 其他对话框**
- `game_dialog.py`: 2 tab（Game Info + DOSBox Config），Add/Edit 模式
- `category_dialog.py`: 列表 + 增/改/删按钮
- `settings_dialog.py`: 2 tab（General 路径 + DOSBox Defaults）
- `about_dialog.py`: 应用图标 + 版本 + 版权

### Phase 3: Main Window & Integration

**Step 3.1 — ui/main_window.py**
- 菜单栏（File/Edit/Help，含所有菜单项和图标）
- 工具栏（Add/Edit/Delete/Launch + Search + Category Filter + Sort + Settings）
- 中央：GameTableView + GameDetailPanel
- 状态栏：游戏总数 + 最近游玩
- 所有 action slot 实现（_on_add_game, _on_edit_game, _on_delete_game, _on_launch_game, _on_settings, _on_categories, _on_about）
- 按钮状态管理（Edit/Delete/Launch 仅选中时可用）

**Step 3.2 — ui/app.py**
- `run()`: 创建 QApplication，解析数据目录，加载配置和游戏数据，创建 MainWindow，启动事件循环
- 配置 logging

**Step 3.3 — 连接 __main__.py**
- `main()` 调用 `ui.app.run()`

### Phase 4: Assets & Polish

**Step 4.1 — 生成缺失资源**
- `assets/app.ico` / `assets/app.png`: 应用图标（ImageGen 生成）
- `assets/default_cover.png`: 160x200 默认封面占位图（ImageGen 生成）

**Step 4.2 — Lint & Type Check**
- `ruff check . && ruff format --check .`
- `mypy dosboxlauncher/`
- 修复所有问题

**Step 4.3 — 运行测试**
- `pytest` 全套测试通过

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| 封面格式转换 | QPixmap（PyQt5）| 不增加 Pillow 依赖 |
| DOSBox 配置规格 | constants.py 中单一字典 | UI 和数据验证的单一事实来源 |
| "All" 分类 | 虚拟，不存 JSON | PRD 要求不可删除，UI 层注入更简洁 |
| GameStore 内存模式 | 全量加载/保存 | 游戏列表小，无需增量 I/O |
| DosboxConfigWidget | 独立文件复用 | SettingsDialog 和 GameDialog 共用 |
| 进程启动 | Popen 非阻塞 | 不冻结 UI |
| JSON 写入 | 临时文件 + os.replace() | 原子写入防崩溃损坏 |
| 分类筛选 | 工具栏下拉选择器 | MVP 最简实现 |

---

## Verification

1. `pip install -e ".[dev]"` 安装成功
2. `pytest` 所有测试通过
3. `ruff check . && ruff format --check .` 无错误
4. `mypy dosboxlauncher/` 无错误
5. `python -m dosboxlauncher` 或 `python startbox.py` 启动应用窗口
6. 手动验证：添加/编辑/删除游戏、分类管理、设置修改、配置合并、启动游戏（需要 DOSBox-X 和 DOS 游戏文件）
