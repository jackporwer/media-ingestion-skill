# Media Ingestion Skill

通用 Agent Skill：把电影、电视剧和动漫从夸克分享或影视资源页面，安全地转存到夸克，再通过 OpenList 复制到自己的媒体库，并在复制完成后完成文件校验和规范命名。

这个项目**不绑定 Hermes**。只要使用中的 Agent 具备所需的浏览器、终端和 HTTP 能力，就可以安装并使用。Hermes、DSH、Codex、Claude Code、OpenCode 或其他 Agent 都可以作为调用方；实际的文件复制和状态校验应由 OpenList 与夸克适配器完成。

## 功能

- 通过片名、年份、季数和集数查找影视资源；
- 直接检查用户提供的夸克分享链接；
- 读取夸克分享中的真实文件和文件大小；
- 选择正确的视频、字幕和集数范围；
- 引导安装夸克 CLI；
- 支持浏览器 OAuth 授权和授权码授权；
- 将选中的文件转存到夸克暂存目录；
- 通过 OpenList 创建异步复制任务；
- 等待任务完成并刷新目标目录；
- 校验源文件与目标文件的字节数；
- 按电影、电视剧和动漫规则生成目录及文件名；
- 保存任务状态，支持中断后恢复；
- 保护已有文件，默认不覆盖、不删除源文件。

## 工作流程

```text
用户输入片名或夸克分享链接
        ↓
解析作品、年份、季数和集数范围
        ↓
搜索资源或读取夸克分享
        ↓
检查真实文件、大小、集数和字幕
        ↓
用户确认候选资源（有歧义时）
        ↓
安装并授权夸克 CLI
        ↓
选中文件转存到夸克
        ↓
OpenList 创建复制任务
        ↓
轮询任务并刷新目标目录
        ↓
校验文件大小
        ↓
安全重命名
        ↓
报告已验证结果
```

任务只有在以下条件全部满足后才算完成：

```text
任务达到成功状态
任务 error 为空
目标目录刷新后能看到目标文件
目标文件不是目录
目标字节数等于源文件字节数
目标名称符合命名规则
```

## 安装 Skill

### 方式一：安装压缩包

下载仓库中的 `media-ingestion-skill.zip`，解压后将 `media-ingestion/` 目录安装到 Agent 的全局 Skills 目录。

解压后的目录必须包含：

```text
media-ingestion/
├─ SKILL.md
├─ agents/openai.yaml
└─ references/
```

### 方式二：直接使用仓库目录

把仓库中的 `media-ingestion/` 目录复制到 Agent 的全局 Skills 目录，不要只复制 `SKILL.md`。`references/` 中包含夸克授权、OpenList 校验、媒体选择和任务恢复规则。

安装后可以使用：

```text
$media-ingestion
```

或者直接用自然语言提出影视入库请求，让 Agent 根据 Skill 自动匹配。

## 首次配置

首次使用至少需要配置 OpenList、夸克账号和媒体库路径。

### OpenList

```dotenv
OPENLIST_BASE_URL=http://192.168.0.139:50001
OPENLIST_USERNAME=your-openlist-user
OPENLIST_PASSWORD=your-openlist-password
```

如果 Agent 与 OpenList 在同一个容器网络中，可以使用容器服务名；如果 Agent 在其他电脑上，应使用 NAS 的局域网地址或 HTTPS 地址。不要直接照抄 `127.0.0.1`，因为它指向运行 Agent 的机器。

### 夸克

```dotenv
QUARK_BASE_URL=https://pan.quark.cn
QUARK_ACCOUNT_REF=your-quark-account-reference
```

实际的 Cookie、Token、密码和授权码不能写入 Skill、仓库、任务状态或聊天记录。

### 片名搜索站点（可选）

只有当用户输入片名、而不是直接提供夸克分享链接时，才需要配置影视资源搜索站点：

```dotenv
MEDIA_SITE_BASE_URL=https://your-private-media-site.example
MEDIA_SITE_USERNAME=your-site-user
MEDIA_SITE_PASSWORD=your-site-password
```

如果浏览器已经登录，可以使用浏览器会话，不必把账号密码持久化保存。

### 媒体库路径

路径是 OpenList 看到的路径，不一定是 NAS 宿主机路径：

```yaml
media_roots:
  movies: /影视/电影
  tv: /影视/电视剧
  anime: /影视/动漫
  staging: /夸克暂存
```

首次安装时必须确认：

- 电影、电视剧、动漫目录属于正确的媒体分类；
- 夸克暂存目录已经在 OpenList 中可见；
- Agent 使用的 OpenList 账号具备读取、创建目录和复制权限；
- 任务状态目录不在媒体库内部；
- 源路径和目标路径不是同一路径。

## 夸克 CLI 安装和授权

首次进行夸克操作前，Skill 会先检查 CLI。当前常见的夸克 CLI 安装形式是：

```bash
bash scripts/install.sh
```

Windows 没有 Bash 时，应使用该 CLI 提供的 PowerShell/Node 等效安装方式，不要自行替换成未知来源的脚本。

安装后先验证：

```bash
node scripts/quark-drive.cjs --help
```

### 浏览器授权

```bash
node scripts/quark-drive.cjs login
```

命令会打开浏览器或输出授权地址。用户完成夸克授权后，Agent 等待登录结果，再执行账号状态检查。

### 授权码授权

浏览器 OAuth 无法自动完成时，Agent 会：

1. 展示授权链接；
2. 引导用户在浏览器中完成授权；
3. 引导用户从跳转 URL 复制 `code` 参数；
4. 要求用户把授权码粘贴回当前对话；
5. 使用授权码执行：

```bash
node scripts/quark-drive.cjs login --token <authorization-code>
```

授权码只在当前登录过程中使用，不能写入文件、日志、任务状态或公开回复。不要要求用户发送密码、Cookie、access token 或 refresh token。

如果授权失败，Agent 应先展示失败原因，然后停止等待用户处理，不得无限循环重试登录。

## 使用示例

### 直接使用夸克分享

```text
把这个夸克分享里的《白蛇：浮生》入库：
https://pan.quark.cn/s/xxxxx
```

Agent 应检查分享实际内容，确认视频、字幕和大小后再转存。

### 通过片名搜索

```text
入库：白蛇：浮生（2024）
```

如果同名作品、年份或版本不明确，Agent 应先列出候选并等待选择。

### 电视剧整季

```text
入库：某电视剧第三季，要求完整季并保留中文字幕
```

Agent 应验证集数连续性，不能只凭分享标题中的“全集”判断完整。

### 恢复中断任务

```text
继续上次未完成的影视入库任务
```

恢复时先读取任务状态、刷新源目录和目标目录，优先验证已有任务和目标文件，禁止重复提交同一批复制任务。

## 默认命名规则

```text
电影目录：<中文片名> (<年份>)
电影文件：<中文片名> (<年份>).mkv

季目录：  <剧名> S01
剧集文件：<剧名> S01E01.mkv
字幕文件：<剧名> S01E01.zh-CN.srt
```

默认质量偏好：

```yaml
selection:
  preferred_resolution: 2160p
  allow_fallback: true
  prefer_hdr: true
  prefer_dolby_vision: true
  require_chinese_subtitle: false
  mode: first-qualified
```

默认冲突策略：

```yaml
conflicts:
  same_name_same_size: skip
  same_name_different_size: ask
  overwrite: false
```

## 任务状态

建议将任务状态保存在独立目录，例如：

```dotenv
MEDIA_INGESTION_STATE_DIR=/data/media-ingestion/jobs
```

状态中可以保存：

- 用户原始请求；
- 作品和年份；
- 分享链接；
- 精确文件名和字节数；
- OpenList 任务 ID；
- 目标目录和目标名称；
- 当前阶段；
- 失败原因。

状态中禁止保存：

- OpenList 密码；
- 夸克密码、Cookie、Token；
- 授权码；
- Authorization Header；
- 浏览器会话数据。

## 安全边界

- 默认只复制，不删除夸克源文件；
- 默认禁止覆盖已有文件；
- 同名但大小不同必须询问用户；
- 不移动正在下载或仍在增长的源文件；
- 不重命名影视库根目录或已有集合目录；
- 没有真实文件列表时不判断画质和完整性；
- 没有 OpenList 验证结果时不声称入库完成。

## 故障排查

### 提示未授权

先展示 provider 返回的未授权原因，执行登录流程，成功后只重试原操作一次。

### 找不到夸克 CLI

检查 Node.js、Skill 安装目录和 `scripts/install.sh`。没有安装器或当前 Agent 无终端能力时，停在配置阶段，不要模拟转存结果。

### OpenList 登录失败

确认 `OPENLIST_BASE_URL` 从当前 Agent 所在环境可访问，检查用户名、密码和是否使用了错误的 `127.0.0.1`。

### 复制任务一直等待

保留任务 ID，读取任务状态并刷新目标目录。不要因为进度达到 100% 就直接报告完成。

### 目标存在同名文件

- 同名同大小：可以跳过并做一次验证；
- 同名不同大小：暂停并等待用户决定；
- 默认不覆盖。

## 兼容性说明

这个仓库只定义通用 Skill 和执行契约，不绑定：

- Hermes；
- DSH；
- 某个 Agent 的目录结构；
- 某个 NAS 的宿主机路径；
- 某个 OpenList 容器名；
- 某个影视搜索网站。

Agent 需要具备的能力由当前环境决定：浏览器用于搜索、终端用于 CLI、HTTP 用于 OpenList API、持久化目录用于恢复任务。

## 项目结构

```text
media-ingestion/
├─ SKILL.md
├─ agents/openai.yaml
└─ references/
   ├─ configuration.md
   ├─ discovery.md
   ├─ job-state.md
   ├─ media-selection-policy.md
   ├─ openlist-verification.md
   └─ quark-cli.md
```

## 许可证

本项目建议使用 MIT License。发布前请确认其中的 CLI、站点适配器和示例文件没有携带第三方私有代码、凭据或受限制的资源。
