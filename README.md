# astrbot_plugins

本仓库用于集中管理和开发多个 [AstrBot](https://github.com/AstrBotDevs/AstrBot) 插件，各插件以 **Git Submodule** 的形式链接其独立的外部子仓库，本仓库本身不直接承载插件业务代码，仅作为开发工作区（workspace）和统一入口。

## 仓库定位

- 每个子目录对应一个独立的 AstrBot 插件仓库，拥有各自的 Git 历史、版本号和 Issue/PR 流程。
- 本仓库不 fork、不复制子仓库代码，只通过 submodule 引用固定的 commit，便于统一克隆、批量测试和跨插件调试。
- 新增插件、修改插件逻辑请前往对应子仓库提交，本仓库只需更新 submodule 指针。

## 目录结构

```
astrbot_plugins/
├── README.md              # 本说明文件
├── .gitmodules             # submodule 配置（子仓库地址与路径）
└── <plugin-name>/          # 各插件子仓库（submodule），互相独立
```

## 子模块操作规范

### 克隆本仓库（含所有子模块）

```bash
git clone --recurse-submodules https://github.com/Reiticia/astrbot_plugins.git
```

如果已经克隆但未拉取子模块：

```bash
git submodule update --init --recursive
```

### 新增一个插件子仓库

```bash
git submodule add <插件仓库地址> <plugin-name>
git commit -m "chore: 新增 <plugin-name> 插件子模块"
```

### 更新某个子模块到最新版本

```bash
cd <plugin-name>
git checkout main
git pull
cd ..
git add <plugin-name>
git commit -m "chore: 更新 <plugin-name> 至最新版本"
```

### 移除某个插件子模块

```bash
git submodule deinit -f <plugin-name>
git rm -f <plugin-name>
rm -rf .git/modules/<plugin-name>
git commit -m "chore: 移除 <plugin-name> 插件子模块"
```

> 子模块内的代码修改一律在子仓库内完成、提交、推送，本仓库只负责记录子模块指向的 commit，不要在本仓库直接改动子模块内容。

## 子模块 README 规范

每个插件子模块必须在自身仓库根目录提供 `README.md`，**严格按照** [`PLUGIN_README_TEMPLATE.md`](./PLUGIN_README_TEMPLATE.md) 模板编写，其中以下三个章节为**必填项**，不得删减：

- **指令列表**：插件注册的全部用户可用指令、参数、权限及示例。
- **函数调用 / 对外接口**：暴露给其他插件或开发者调用的公共函数、类、事件钩子；若不提供对外调用能力，需明确写"本插件不提供对外接口"。
- **附加功能**：指令之外的能力，如定时任务、事件监听、Web 面板、数据持久化等；若无需明确写"无"。

新增插件子模块时，先复制模板文件到子仓库并重命名为 `README.md`，再填充实际内容。

## 插件代码编写规范

以下规范适用于各插件子仓库（AstrBot 插件均为 Python 项目）：

### 基础规范

- 遵循 [PEP 8](https://peps.python.org/pep-0008/) 代码风格，函数、变量使用 `snake_case`，类名使用 `PascalCase`。
- 公开函数和类需编写类型标注（Type Hints），提高可读性和 IDE 支持。
- 关键逻辑、非直观的实现需添加注释说明"为什么"，而非复述代码本身；注释与代码注释一律使用简体中文。
- 禁止硬编码密钥、Token、密码等敏感信息，统一通过 AstrBot 提供的配置机制或环境变量读取。

### 插件结构规范

- 插件入口需符合 AstrBot 插件规范（`metadata.yaml` / `main.py` 或对应框架要求的入口文件),元信息（名称、版本、作者、描述）保持准确。
- 插件间避免直接相互引用或产生强耦合；如需共享逻辑，优先在各自插件内实现，或抽取为独立公共库。
- 对外部 API 调用、网络请求需妥善处理超时、异常和重试，不能让单个插件的异常影响 AstrBot 主进程。

### 异常处理与日志

- 使用 AstrBot 提供的日志接口输出日志，禁止使用 `print` 调试代码遗留在提交中。
- 对可预期的异常（网络错误、参数错误等）显式捕获并给出清晰提示；不要用空 `except:` 吞掉异常。

### 提交规范

- Commit message 使用简体中文，说明改动动机而非单纯罗列改动内容。
- 每次提交聚焦单一改动，避免将无关的重构和功能修改混在一起。

### 测试建议

- 插件核心逻辑（如指令解析、数据处理）建议编写单元测试，使用 `pytest`。
- 涉及与 AstrBot 框架交互的部分，建议在本地搭建 AstrBot 实例进行集成测试，验证指令注册、事件监听等行为符合预期。
