# PenT3st — 渗透测试实战工作流 Skill

> **v2.0.0**（2026-09-06）：新增"行为纪律"节（范围双检/请求预算/封禁连坐等 12 条）、注入三段差分验证、验证码辨型、客户端自述认证绕过、存量复测监控、APK 静态→动态闭环。前版归档于 `../PenT3st-V1.0.0`。

> 面向 **Claude Code** 的授权渗透测试实战 skill。带强制 checkpoint 的 5 阶段工作流，而非参考手册——每个阶段有 MUST 输出，未通过不进下一阶段。payload / playbook / 案例按需 Read。

---

## 这是什么

PenT3st 把一次授权渗透测试拆成 **5 个阶段**（Scope → Recon → Discover → Exploit → Report），每个阶段用**多角色协作**（Planner / Searcher / Pentester / Coder / Reporter / Monitor）推进，并在关键节点设 checkpoint 强制对齐授权范围与证据链。

授权渗透测试**指向性强**——有明确目标和授权，确认范围后直接进入信息收集与攻击面测绘，目标是**深度突破**而非广泛搜索，追求链路完整（权限/数据/全链）。Phase 4 明确允许在授权范围内做提权、横向移动与后渗透，展示真实影响。

### 核心特性

| 特性 | 说明 |
|------|------|
| **反幻觉硬约束** | 5 条全程规则：不准凭记忆出 payload、不准编造案例编号、无证据不下结论、出 scope 立即停、合法合规 |
| **强制 checkpoint** | 每个 Phase 有 MUST 输出，缺项不进下一阶段，缺什么问什么不假设 |
| **两步 Read 模式** | 目录型 playbook 先读 `00-index.md`（路由表）再读具体子文件，避免一次性灌入巨量 payload |
| **MCP 工具动态检测** | 不绑定任何固定工具，按用户已装的 MCP 服务动态适配，无工具自动降级为手动命令 |
| **多角色协作** | Searcher 只做被动收集禁发起攻击；Coder 仅在现有工具不足时介入；Monitor 检测低效循环并建议策略调整 |
| **合规红线内嵌** | `compliance.md` 定义绝对禁止 / 需确认 / 必做 / 中止触发条件，Phase 5 报告前强制比对 |
| **测试过程强制记录** | 每个漏洞必须四要素齐全才算确认：文字复现流程 + 差分证明 + 截图证据 + 复现稳定性 + 修复建议三档；无测试过程不得进 Phase 4 / 不得写入报告 |
| **国产组件指纹库** | 致远/通达/万户/泛微/用友/金蝶/海康/大华等指纹 + 默认凭据，覆盖国内常见老系统 |

---

## 触发条件

命中任一即进入：

- "渗透测试 / pentest / 渗透 / PenT3st / 红队测试 / 红队评估"
- 用户给出明确目标（IP / 域名 / 网段 / 应用）要求进行安全测试
- "帮我测一下 / 打一下 / 看看能不能突破 / 测试安全性"
- "内网渗透 / 横向移动 / 权限提升 / 提权"

**不应触发**：纯白盒源码审计 → `code-audit` skill；CTF → 通用对话。

---

## 目录结构

```
PenT3st/
├── SKILL.md                          # 核心入口：5 阶段工作流 + 路由引擎
├── README.md                         # 本文件
├── .claude-plugin/
│   └── marketplace.json              # Claude Code 插件清单
└── references/                       # 厚知识库，按需 Read
    ├── compliance.md                 # 合规与合法红线（Phase 5 强制读）
    ├── methodology/                  # 方法论（决策与策略）
    │   ├── 00-index.md               # 方法论入口与阶段映射
    │   ├── 01-attack-priority.md     # 攻击优先级（P0-P3 评分）
    │   ├── 02-bypass-toolkit.md      # 通用绕过决策树 + 编码字典
    │   ├── 03-evidence-discipline.md # 证据纪律（HTTP包/回显/DNSLog/复现率）
    │   ├── 04-control-gap-hunting.md # 控制缺口狩猎（9类敏感操作→探测策略）
    │   ├── 05-timebox-priority.md    # 时间盒优先级排序
    │   ├── 10-passive-recon.md       # 阶段工作流：被动侦察
    │   ├── 11-active-scanning.md     # 阶段工作流：主动扫描
    │   └── 12-exploit-postexp.md     # 阶段工作流：利用与后渗透
    ├── playbooks/                    # 19 类攻击 playbook（探测→绕过→利用）
    │   ├── 00-index.md               # 总目录 + 攻击链深度排序
    │   ├── unauth-access.md          # 默认凭据 / Redis / Actuator / Swagger
    │   ├── info-disclosure.md        # .git / 备份文件 / phpinfo / OSS bucket
    │   ├── sqli.md                   # SQL 注入（27,732 真实案例提炼）
    │   ├── arbitrary-x-authz.md      # 任意 X 子授权（任意账号/任意操作）
    │   ├── graphql.md / race-conditions.md / http-smuggling.md / dos.md / mobile.md
    │   ├── rce/                      # 框架RCE/命令注入/反序列化/SSTI/XXE/原型链/供应链
    │   ├── file-upload/              # 上传绕过/压缩包穿越/竞态下载
    │   ├── path-traversal/           # LFI/RFI/日志注入/PHP wrapper/PHAR
    │   ├── xss/                      # 按类型/绕过/利用
    │   ├── ssrf-cache-host/          # SSRF/云元数据/缓存投毒/Host头
    │   ├── logic-flaws/              # CSRF/业务逻辑/点击劫持
    │   ├── oauth-saml-jwt/           # OAuth/SAML/JWT/认证杂项
    │   ├── api-rest/                 # REST/GraphQL/JWT/WebSocket
    │   ├── llm-prompt-injection/     # Prompt注入/RAG投毒/Agent工具
    │   └── intranet-postexp/         # 内网后渗透（凭据/横移/提权/免杀/域/隧道/ADCS/Exchange/SharePoint）
    ├── dictionaries/                 # 字典与指纹
    │   ├── 00-index.md
    │   ├── default-credentials-cn.md # 国产系统默认凭据
    │   └── chinese-fingerprints.md   # 国产组件指纹与路径
    ├── industry/                     # 行业垂直 playbook
    │   ├── 00-index.md
    │   ├── banking-finance.md        # 银行/支付/金融
    │   └── telecom-isp.md            # 运营商/ISP/物联网卡
    ├── templates/
    │   └── report-pentest.md         # 渗透测试报告模板
    ├── tools/
    │   └── mcp-jshook.md             # jshook MCP 工具映射
    └── h1-reports/                   # 真实已披露漏洞案例（按 weakness 分类）
```

---

## 5 阶段工作流

| 阶段 | 角色 | MUST 输出 | 关键约束 |
|------|------|----------|---------|
| **Phase 1 · Scope** | Planner | 授权确认 + 目标范围 + 测试约束（三项缺一不进） | 无书面授权 = 违法，无例外 |
| **Phase 2a · 被动侦察** | Searcher | 不发包得到的资产清单，来源 ≥3 种 | Searcher 禁止发起任何攻击性操作 |
| **Phase 2b · 主动扫描** | Pentester | 活资产矩阵：目标→端口→服务→版本→技术栈→JS endpoint | 速率控制，避免触发 WAF/SOC |
| **Phase 2c · 攻击面地图** | Planner | 优先级排序的攻击面地图（P0-P3 + 匹配信号） | 条件触发 Read 指纹/行业 playbook |
| **Phase 3 · Discover** | Pentester + Coder | 每个候选目标走 playbook 探测，命中即产出**测试过程记录四要素**（文字复现流程 + 差分证明 + 截图 + 复现稳定性 + 修复建议初稿） | 无完整测试过程 = 未确认，不进 Phase 4 |
| **Phase 4 · Exploit** | Pentester + Coder | 利用链路径 + 权限矩阵 + 测试过程记录 + 证据包 | 深度≠破坏，横向≠无限，证明≠利用，不可复现不进报告 |
| **Phase 5 · Report** | Reporter | 执行摘要 + 漏洞详情（含完整测试过程+修复建议）+ 攻击路径图 + 修复矩阵 + 清理确认 | 报告前强制读 compliance.md；无测试过程/修复建议的漏洞不得写入 |

---

## 安装与启用

### 方式一：作为 Claude Code 插件市场安装

本项目含 `.claude-plugin/marketplace.json`，可通过 Claude Code 插件机制添加。

### 方式二：作为本地 skill 放入 skills 目录

将整个 `PenT3st` 目录放入 Claude Code 的 skills 路径（如 `~/.claude/skills/PenT3st/`），Claude Code 会自动识别 `SKILL.md` 的 frontmatter 并在触发条件命中时加载。

启用后，在对话中说出触发词（如"对 target.com 做渗透测试"）或直接给出目标，skill 即自动进入 Phase 1。

---

## 典型使用流程

```
用户：对 https://target.example.com 做渗透测试，已获书面授权
  ↓
PenT3st Phase 1：确认授权/范围/约束（checkpoint 三项）+ 检测可用 MCP 工具
  ↓
PenT3st Phase 2：被动收集 → 主动扫描 → 产出攻击面地图
  ↓
PenT3st Phase 3：按信号匹配 playbook，逐目标探测，命中存证据
  ↓
PenT3st Phase 4：深度利用 → 提权 → 横向（授权范围内）→ 清理
  ↓
PenT3st Phase 5：读合规红线 → 套报告模板 → 输出完整报告
```

**MCP 工具适配示例**：
- 装了 `kali-mcp` → 直接调 nmap/metasploit
- 装了 `burp-mcp` → 流量拦截重放
- 什么都没装 → 输出等效命令让你本地执行后贴回结果，流程不中断

---

## 数据基础

- **真实已披露漏洞案例**：2,800+ 份 High/Critical 报告，按 weakness 分类存放于 `references/h1-reports/by-weakness/`，每份含来源、标题、摘要。引用前必须 Read 实际文件，不准凭记忆编案例编号。
- **历史漏洞案例统计**：77,000+ 条，提炼为各 playbook 的参数频率表、高危占比、真实指纹。
- **结构化 payload**：305 个 + 263 个 WAF/EDR 绕过变体，均带上下文标注（HTML/属性/JS字符串/URL）。

---

## 合规声明

**本项目仅用于授权渗透测试、安全评估、CTF 训练与防御性安全研究。**

- 所有操作必须在 Phase 1 确认的**书面授权范围**内进行。授权文件（合同/授权书/邮件确认）是法律保护的唯一依据。
- **未授权渗透测试是违法行为**，无例外。
- skill 内嵌 `compliance.md` 红线：出 scope 立即停、不破坏业务数据、不安装持久化后门、不未授权社工、不超范围拖库、不未复现猜测。
- 内网后渗透 playbook（`intranet-postexp/`）所有 payload 仅在已获授权时使用（红队、合同测试、HVV 演练）。
- 报告中所有真实数据（cookie / token / PII）必须脱敏到只剩 head/tail。

使用者需自行确保具备合法测试授权，并对自身行为承担全部法律责任。

---

## License

MIT（见 `.claude-plugin/marketplace.json`）
