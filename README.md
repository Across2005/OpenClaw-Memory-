# OpenClaw Memory · 本地 Agent 自进化记忆系统

<p align="center">
  <b>让本地 Agent 拥有"记得住、分得清、用得准、越用越聪明"的长期记忆能力</b>
</p>

<p align="center">
  <a href="https://www.python.org/"><img src="https://img.shields.io/badge/python-3.10+-blue" alt="Python 3.10+"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green" alt="MIT License"></a>
  <a href="https://gitee.com/Across2005/openclaw-memory"><img src="https://img.shields.io/badge/Gitee-Across2005-red" alt="Gitee"></a>
  <img src="https://img.shields.io/badge/零依赖-纯Python_stdlib-success" alt="zero deps">
  <img src="https://img.shields.io/badge/Coze_Skill-可部署-blueviolet" alt="Coze Skill">
</p>

---

## 概况

记忆是 Agent 的灵魂。但现有 Agent 的记忆要么存在云端（泄露风险），要么只增不减（沦为噪声堆）。

OpenClaw Memory 是一个**可本地私有部署**的 Agent 记忆基座：记忆数据全部存储于本地文件系统，离线可用、数据自持；同时具备**自进化**能力——高频高价值的记忆自动前置，低频低价值的记忆随使用衰减淘汰。

项目以 Python 原型 + Coze Skill 包双形态交付，既可直接在本地运行，也可无缝部署到 Coze 平台参赛或使用。

---

## 功能

| 操作 | 说明 |
|------|------|
| **`remember`** | 写入记忆。自动分类（指令 / 偏好 / 事实 / 常识），写入前去重与极性矛盾检测，冲突时交用户拍板 |
| **`recall`** | 检索记忆。分级调动，按 `相似度 × 优先级 × 时效` 排序返回 Top-K，同时累加访问频次 |
| **`evolve`** | 自进化。重算优先级 `0.5·频次 + 0.3·时效 + 0.2·反馈`，久未访问且低价值者标记 `deprecated` |
| **`stats`** | 查看记忆存量与分类分布 |

---

## 快速开始

```bash
# 克隆
git clone https://gitee.com/Across2005/openclaw-memory.git
cd openclaw-memory

# 运行（零额外依赖，纯 Python 3.10+）
python -m openclaw_memory remember "用户偏好中文技术文档，术语统一用简体" --type preference
python -m openclaw_memory remember "翻译时必须保留原文段落编号"
python -m openclaw_memory recall "术语怎么处理"
python -m openclaw_memory evolve
python -m openclaw_memory stats

# 运行测试
python -m openclaw_memory.tests.test_system
```

记忆默认存于当前目录 `memory/`，可通过 `--root` 或环境变量 `OPENCLAW_MEMORY_ROOT` 指定。

### 可选加密

```bash
python -m openclaw_memory --passphrase "你的口令" remember "敏感偏好"
```
需安装 `cryptography`（未安装时自动回退为轻量 XOR 混淆）。

---

## 部署到 Coze

项目提供两种方式将记忆能力接入 Coze 平台：

### 方式一：纯云端代码节点（推荐参赛，零服务器）

1. 在 Bot 工作流拖入「代码节点」，语言选 Python
2. 粘贴 `coze_skill/scripts/memory_code_node.py` 全文
3. 输入参数：`action`, `content`, `store`(JSON 字符串)；输出参数：`result`, `store`, `isSuccess`
4. 建工作流变量 `memory_store`（初始 `{}`），把返回的 `store` 写回变量回传，实现跨轮持久记忆

### 方式二：API 插件（本地优先，真私有）

```bash
pip install fastapi uvicorn
python coze_skill/references/backend_server.py
```

在 Coze「插件 → 通过 OpenAPI 创建」上传 `coze_skill/references/plugin_openapi.yaml`，记忆落盘到本地引擎。

---

## 项目结构

```
openclaw-memory/
├── openclaw_memory/          # 本地 Python 原型（零依赖）
│   ├── models.py             # 记忆条目数据模型
│   ├── similarity.py         # 语义相似度（TF 余弦，零依赖）
│   ├── modules.py            # 核心算法：分类 / 矛盾 / 自进化 / 检索
│   ├── storage.py            # 三级存储：meta_index + 分类目录 + 条目文件
│   ├── system.py             # 编排门面：remember / recall / evolve / stats
│   ├── crypto.py             # 本地加密（可选）
│   ├── cli.py                # 命令行入口
│   └── tests/test_system.py  # 6 项测试（写入→检索→去重→冲突→自进化→加密）
├── coze_skill/               # Coze Skill 包
│   ├── SKILL.md              # 技能元信息（可直接导入 Coze）
│   ├── scripts/memory_code_node.py  # 代码节点脚本（纯云端）
│   └── references/           # API 插件后端 + OpenAPI 描述
└── docs/                     # 设计文档与说明报告
```

---

## 设计文档

| 文档 | 说明 |
|------|------|
| [项目方向与设计思路](docs/OpenClaw自进化记忆系统_项目方向与设计思路.md) | 核心方向、四层目的、自进化飞轮、赛题契合 |
| [设计文档](docs/OpenClaw自进化记忆系统_设计文档.md) | 系统架构、7 大模块、三级存储、数据结构 |
| [实现落地与可行性扩展](docs/OpenClaw自进化记忆系统_实现落地与可行性扩展.md) | 可运行原型、API 参考、降风险 MVP 路线 |
| [Coze 试用版说明](docs/OpenClaw自进化记忆系统_说明报告_Coze试用版.md) | Coze 平台方案 B 的取舍与部署策略 |
| [MoonBit 版说明](docs/OpenClaw自进化记忆系统_说明报告_MoonBit版.md) | 本地私有终极目标形态的路线规划 |

---

## 设计要点

- **三级存储**：系统总库（全局索引）→ 分类 Skill 库 → 记忆条目文件
- **自进化飞轮**：`使用 → 频次↑ → 反馈 → 优先级重算 → 高价值前置 / 低价值淘汰 → 下一次更准`
- **安全三件套**：本地加密（`cryptography` / XOR 回退）、指令授权（确认后才删除 / 覆盖）、防注入（记忆仅作数据召回，不当作可执行指令）

---

## License

[MIT](LICENSE) © 2026 Across2005
