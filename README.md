# 二次元图片插件 (maibot-moepix)

> 基于 Lolicon API 的二次元图片获取插件。支持 AI 智能调用和命令直接调用，内容级别采用白名单控制，发送后自动撤回。

## ✨ 功能特性

| 功能 | 说明 | 触发方式 |
|------|------|----------|
| **AI 智能调用** | AI 根据用户自然语言（如"来张二次元图"、"白丝萝莉"），自主调用工具获取图片 | 任意对话，AI 自主决策 |
| **命令直接调用** | 用户通过 `/tu` 命令直接获取 | `/tu [标签]` |
| **扩展内容权限** | 白名单控制，AI 可动态开启/关闭，管理员可在 WebUI 手动配置 | AI 工具 + 配置文件 |
| **自动撤回** | 图片发送后指定时间自动撤回 | 配置 `recall_after_seconds` |
| **频率限制** | 防止滥用，可配置每分钟最大请求次数 | 自动执行 |
| **热重载配置** | 修改配置无需重启插件 | 文件监听自动生效 |
| **配置保护** | 插件更新时自动合并新增配置项，不覆盖用户已有配置 | 自动执行 |

## 📦 安装

将 `maibot-setu` 文件夹放入 MaiBot 的 `plugins/` 目录下：


```
your-maibot/
├── plugins/
│   └── maibot-moepix/
│       ├── _manifest.json
│       ├── plugin.py
│       ├── config.example.toml
│       ├── .gitignore
│       └── _locales/
│           └── zh-CN.json
└── ...
```

确保安装了 Python 依赖：

```bash
pip install aiohttp
```

首次安装时，插件会自动从 `config.example.toml` 复制创建 `config.toml`。

## ⚙️ 配置说明

配置文件：`config.toml`，也可在 WebUI 配置编辑器中修改。

### 扩展内容权限设置

```toml
[kz]
# 扩展内容白名单
allowed_chats = []
```

**白名单格式：**
- 群聊：`"group_群号"`，例如 `"group_123456"`
- 私聊：`"user_QQ号"`，例如 `"user_123456"`

**扩展内容权限有两种启用路径：**

1. **AI 动态开启**：用户向 AI 表达意愿，AI 觉得用户真诚则调用工具开启（自动持久化）。用户也可要求 AI 关闭，AI 调用关闭。
2. **管理员手动配置**：在 WebUI 配置编辑器中直接编辑 `allowed_chats` 列表。

**重要：未开启扩展权限的会话，无论命令还是 AI 调用，均不可请求扩展内容。**

### API 设置

```toml
[api]
base_url = "https://api.lolicon.app/setu/v2"  # Lolicon API 地址
default_num = 1               # 默认请求图片数量
default_size = ["regular"]    # 图片规格
proxy = ""                    # 图片反代地址（留空=直连 Pixiv）
exclude_ai = true             # 排除 AI 作品
max_num = 3                   # 单次最大请求数量
send_mode = "image"           # 发送模式: image/link/both
```

**`proxy` 说明：**
- 留空（`""`）：使用 Pixiv 原始地址 `i.pximg.net`（需要服务器能直接访问 Pixiv）
- `"i.yuki.sh"`：使用自定义反代
- `"i.pixiv.re"`：Cloudflare 反代（可能间歇性不可用）

**`send_mode` 说明：**
- `"image"`：直接发送图片（默认，体验最好但可能被审查）
- `"link"`：只发送图片链接文本（安全，不会被审查，用户需自行打开）
- `"both"`：同时发送图片和链接
- `"kz_link"`：全年龄内容发图片，扩展内容发链接（推荐用于防审查场景）

### 频率限制与撤回

```toml
[limits]
rate_per_minute = 3           # 每分钟最多请求次数
recall_after_seconds = 90     # 发送后自动撤回延迟秒数（0=不撤回）
```

**自动撤回**依赖 NapCat 适配器的 `delete_msg` API。建议群聊设置 60-180 秒，私聊可设为 0。

## 🎮 使用方法

### 命令调用

```
/tu                    # 随机 1 张
/tu 白丝               # 标签"白丝"，1 张
```

### AI 智能调用

直接用自然语言与机器人对话，AI 会自动判断是否调用色图工具：

- "来张白丝萝莉图"
- "多来几张二次元图"
- "有没有某画师的图"

### 扩展内容使用流程

**AI 动态开启：**

1. 用户向 AI 表达意愿
2. AI 评估用户态度和场景，认为合适则调用
3. 插件将该会话加入白名单（自动持久化）
4. 用户可随时要求 AI 关闭 → AI 调用

**管理员手动配置：**

1. 在 WebUI 配置编辑器中编辑
2. 添加群号或用户号
3. 配置自动热重载，立即生效

## 🔒 安全设计

1. **默认安全**：安装后白名单为空，任何会话均不可请求扩展内容。
2. **统一控制**：无论 AI 还是命令，权限检查一致，不可绕过。
3. **持久化**：AI 开启的权限会写入配置文件，重启后依然有效。
4. **频率限制**：基于滑动窗口的频率限制，防止滥用。
5. **自动撤回**：图片发送后自动撤回，避免内容长期留存。
6. **日志审计**：所有关键操作均通过框架日志系统记录。

## 📁 文件结构

```
maibot-moepix/
├── _manifest.json          # 插件元数据清单（manifest v2）
├── plugin.py               # 插件主入口（配置模型 + 所有组件）
├── config.example.toml     # 配置模板文件（随 git 更新）
├── config.toml             # 用户配置文件（不纳入 git，受保护）
├── .gitignore              # 排除 config.toml
├── README.md               # 本文件
└── _locales/
    └── zh-CN.json          # 中文
```

## 📄 许可证

WTFPL
