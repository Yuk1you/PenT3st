---
name: PenT3st
description: "渗透测试实战工作流 skill。融合多 Agent 协作、动态工具检测、智能任务规划。5 阶段方法论（scope → recon → discover → exploit → report）、19 类攻击 playbook、305 个结构化 payload、263 个 WAF/EDR 绕过变体、2887 份 HackerOne 真实案例、国产组件指纹库。当用户提到 \"渗透测试 / pentest / 渗透 / PenT3st / 红队\" 或给出明确目标让你测试时触发。"
argument-hint: "<target-or-scope-or-phase>"
level: 2
---

# PenT3st — 渗透测试实战工作流

这是一个**带强制 checkpoint 的渗透测试工作流**，不是参考手册。每个阶段有 MUST 输出，未通过不进下一阶段。详细 payload / playbook / 案例**按需 Read**，不准凭记忆生成。

与 SRC/Bug Bounty 不同，渗透测试**指向性强**——有明确的目标和授权，确认范围后直接进入信息收集与攻击面测绘，目标是深度突破而非广泛搜索。

---

## 触发条件

命中任一即进入：
- "渗透测试 / pentest / 渗透 / PenT3st / 红队测试 / 红队评估"
- 用户给出明确目标（IP / 域名 / 网段 / 应用）要求进行安全测试
- "帮我测一下 / 打一下 / 看看能不能突破 / 测试安全性"
- "内网渗透 / 横向移动 / 权限提升 / 提权"

**不应触发**：纯白盒源码审计 → `code-audit` skill；CTF → 通用对话。

---

## 反幻觉硬约束（全程适用）

1. **不准凭记忆出 payload**。要给 SQLi/RCE/SSRF/XSS 任何 payload 前，先 Read 对应 `references/playbooks/<type>.md`（或 `<type>/00-index.md` + 具体子文件，见下表）。payload 必须能在文件里查到出处。
2. **不准编造案例编号**。引用 H1/WooYun 案例前必须 Read `references/h1-reports/by-weakness/` 下的实际文件。说不出文件路径就别引。
3. **无证据不下结论**。无 HTTP 包/截图/视频时只能写"待验证 / 假设"，不写"已确认 / 发现漏洞"。
4. **出 scope 立即停**。任何时候发现要测的资产不在 Phase 1 已确认的授权范围 → 立即停手，回到 Phase 1 重核。
5. **合法合规**。必须确认已获得合法授权再开始测试，未授权渗透测试是违法行为。

---

## 多角色协作模型

PenT3st 采用**角色分工**模式，在每个 Phase 中自动切换角色视角：

| 角色 | 职责 | 活跃阶段 |
|------|------|---------|
| **Planner** | 任务分解，生成 3-7 步执行计划 | 每个 Phase 开始时 |
| **Searcher** | 被动信息收集，OSINT 情报，不发起攻击 | Phase 2 |
| **Pentester** | 主动探测、漏洞验证、工具执行 | Phase 2b, 3, 4 |
| **Coder** | 编写自定义 exploit / PoC 脚本 | Phase 3, 4（工具不足时） |
| **Reporter** | 生成渗透测试报告 | Phase 5 |
| **Monitor** | 检测执行循环和低效行为，建议策略调整 | 全程后台 |

**角色规则**：
- 每个角色只关注其职责范围内的工具和操作
- Searcher **禁止**发起任何攻击性操作
- Coder 仅在现有工具无法完成任务时介入
- Monitor 检测到以下情况时触发干预：
  - 连续 3 次相同操作无进展 → 建议换策略
  - 单个目标超过时间盒 50% → 建议跳过或降级
  - 接近阶段结束仍无产出 → 触发 Reflector 自省

---

## MCP 工具动态检测

PenT3st **不绑定任何固定工具**，根据用户实际安装的 MCP 服务动态适配。

### 工具检测流程（Phase 1 完成后自动执行）

1. 检测当前会话可用的 MCP server（通过观察 system prompt 中已注册的工具列表）
2. 按能力分类，构建**可用工具矩阵**
3. 基于可用工具调整后续阶段的执行策略

### 工具能力映射表

| 能力域 | 可能的 MCP 工具 | 无工具时的降级策略 |
|--------|---------------|-------------------|
| **网络扫描** | kali-mcp (nmap), shodan-mcp | 建议用户手动执行 nmap 并贴回结果 |
| **Web 漏洞扫描** | burp-mcp, zap-mcp, nuclei-mcp | 使用内置 payload 手动构造请求 |
| **流量拦截** | burp-mcp, mitmproxy-mcp | 指导用户使用 curl / httpie 手动测试 |
| **移动端测试** | frida-mcp, adb-mcp | 指导用户手动操作 |
| **浏览器自动化** | chrome-devtools (playwright) | 指导用户手动浏览器操作 |
| **漏洞利用** | metasploit-mcp, kali-mcp | Coder 角色编写自定义 PoC |
| **密码破解** | kali-mcp (hashcat/john) | 建议用户本地执行 |
| **DNS/子域枚举** | kali-mcp, subfinder-mcp | 使用 Web API（crt.sh 等） |
| **代码分析** | semgrep-mcp, codeql-mcp | 使用内置 Grep/Read 分析 |

### 工具调用原则

1. **有工具就用工具**：检测到对应 MCP 工具已安装，直接调用
2. **无工具就降级**：给出手动执行命令，让用户执行后贴回结果
3. **不强制依赖**：任何 Phase 都不因缺少某个工具而中断
4. **用户做主**：工具选择建议由用户最终确认

---

## Phase 1 · Scope（授权与范围确认）

**进入条件**：用户首次给出目标。

渗透测试指向性强，此阶段快速确认授权和范围即可。

**MUST 输出 checkpoint**（三项缺一不进 Phase 2，缺什么向用户问什么，不要假设）：

- [ ] **授权确认**：已获得书面授权（渗透测试合同/授权书/邮件确认）—— 向用户确认
- [ ] **目标范围**：可测目标清单（域名 / IP / 网段 / 应用 / API）+ 禁测项（逐条列）
- [ ] **测试约束**：测试类型（黑盒/灰盒/白盒）、时间窗口、是否允许社工/DoS/数据破坏

**Phase 1 完成后自动执行**：MCP 工具动态检测，输出可用工具矩阵。

---

## Phase 2 · Recon（信息收集与攻击面测绘）

**进入条件**：Phase 1 checkpoint 三项全过 + 工具矩阵已生成。

### Planner 预规划

进入本阶段前，先以 Planner 角色生成 **3-7 步执行计划**：

```xml
<task_assignment>
  <goal>对目标 {target} 进行全面信息收集与攻击面测绘</goal>
  <available_tools>基于 Phase 1 检测到的可用工具列表</available_tools>
  <execution_plan>
    1. [具体步骤，基于可用工具调整]
    2. ...
  </execution_plan>
  <constraints>时间窗口、禁测项等约束</constraints>
</task_assignment>
```

### 2a. 被动信息收集（Searcher 角色）

**MUST Read** `references/methodology/10-passive-recon.md`——完整被动侦察工作流，包含子域名枚举（crt.sh/subfinder/amass pipeline）、DNS 全记录类型分析（SPF/DMARC/DKIM 情报提取 + Zone Transfer）、搜索引擎 OSINT（Google/GitHub/Shodan/FOFA/Censys dorks 模板）、历史 URL（gau/Wayback pipeline）、ASN/IP 归属、泄露数据库查询。

**MUST 输出**：不发包给目标得到的资产清单 + 历史信息，来源 ≥3 种：
- CT 日志（crt.sh / Censys）
- Wayback / CommonCrawl 历史快照
- GitHub dorks（`org:target` + `password|api_key|SECRET|.env`）
- FOFA / Shodan favicon hash
- SecurityTrails / DNS 历史
- ASN / IP 段（bgp.he.net）

### 2b. 主动探测与指纹识别（Pentester 角色）

**MUST Read** `references/methodology/11-active-scanning.md`——完整主动探测工作流，包含 Nmap 扫描策略（6 种主机发现 + 5 种端口扫描类型 + 速度控制 + NSE 脚本引擎 + 7 套命令模板）、Web 应用指纹识别（whatweb/httpx/响应头分析/Favicon Hash/WAF 检测）、目录路径枚举（feroxbuster/gobuster/ffuf + 敏感路径必查清单）、服务特定探测（数据库/远程管理/文件共享/邮件）。

**执行顺序**（参照 `11-active-scanning.md` 第六节）：
1. 主机发现 → 存活主机列表
2. TCP 端口扫描 → Top 1000（全体）→ 全端口（重点目标）
3. UDP 端口扫描 → Top 200（重点目标）
4. 服务识别 → 对开放端口做 `-sV -sC`
5. Web 指纹 → whatweb / httpx 对 Web 端口
6. WAF 检测 → wafw00f
7. 目录枚举 → feroxbuster + 敏感路径清单
8. 服务特定枚举 → SMB/FTP/数据库/邮件/远程管理

**MUST 输出**：活资产矩阵——`目标 → 端口 → 服务 → 版本 → 技术栈 → JS endpoint`。

### 2c. 攻击面地图

**MUST 输出**：汇总 2a + 2b，产出优先级排序的攻击面地图：

```
目标资产 | 服务/技术栈 | 入口点 | 认证状态 | 攻击优先级(P0-P3) | 匹配信号
---------|------------|--------|---------|-------------------|--------
```

**条件触发 Read**（命中就必读，不命中不读）：

| 命中信号 | MUST Read |
|---|---|
| 指纹含 `weaver/seeyon/tongda/landray/yongyou/kingdee/hikvision/dahua` | `references/dictionaries/chinese-fingerprints.md` + `references/dictionaries/default-credentials-cn.md` |
| 资产含 银行 / 支付 / 网银 / 第三方支付聚合 | `references/industry/banking-finance.md` |
| 资产含 运营商 / BOSS / 网管 / 物联网卡 | `references/industry/telecom-isp.md` |
| 不确定优先级怎么排 | `references/methodology/01-attack-priority.md` |

---

## Phase 3 · Discover（漏洞发现）

**进入条件**：Phase 2 攻击面地图 ≥1 个候选目标。

### Planner 预规划

```xml
<task_assignment>
  <goal>对攻击面地图中 P0-P1 目标进行系统化漏洞发现</goal>
  <targets>从攻击面地图中按优先级选取</targets>
  <execution_plan>
    1. [按信号匹配 playbook]
    2. [逐目标逐入口测试]
    ...
  </execution_plan>
</task_assignment>
```

### Pentester 角色执行

**强制流程（按优先级对每个候选目标走一遍）**：
1. 看目标信号，从下表选 playbook
2. **Read 该 playbook 文件**（不准跳过、不准凭记忆替代）
3. 按 playbook 的"参数频率表"挑入口
4. 按 playbook 的"payload 库"探测——payload 来自文件，不来自训练记忆
5. 被 WAF 拦 → Read `references/methodology/02-bypass-toolkit.md` 决策树
6. 命中后立即按下方"测试过程记录"MUST 输出完整记录 → 标记为 Phase 4 候选

### MUST 输出：测试过程记录（每个确认漏洞一份）

> 漏洞是否真实有效，靠这份记录证明。没有完整测试过程的漏洞 = 未确认，不进 Phase 4，不写进报告。

每个命中的漏洞，**当场**产出以下四要素（引用 `references/templates/report-pentest.md` 第 5 章单漏洞模板）：

- [ ] **文字复现流程**：逐步编号的操作描述（Step 1 做什么 → Step 2 做什么），每步配完整 HTTP 请求/响应包。文字要说清"我做了什么操作、为什么这样做、观察到什么"——不是只甩 HTTP 包。参照 `references/methodology/03-evidence-discipline.md` §3 原则 1。
- [ ] **差分证明**：按漏洞类型提供对照组（盲注真/假/baseline、IDOR 自己/他人/不存在、越权 拒绝/通过/绕过）。参照 `03-evidence-discipline.md` §3 原则 2。无差分 = 单包幻觉。
- [ ] **截图证据**：至少 1 张触发截图（含完整 URL bar + 时间戳 + 响应关键字段高亮）+ 1 张流量截图（Burp/curl）。敏感数据马赛克但保留格式。参照 `03-evidence-discipline.md` §6 截图规范。
- [ ] **复现稳定性**：按等级复现 N 次（P0 ≥3 次/P1 ≥3 次/逻辑类 ≥5 次），记录每次结果与关键指标。复现率不达 100% 主动注明。
- [ ] **修复建议初稿**：立即（24h）/ 短期（1 周）/ 长期三档，精确到代码或配置层，不说"加强安全意识"空话。

**反模式（这些测试过程会被打回）**：
- ❌ 只贴一个 HTTP 包，无文字说明操作过程
- ❌ 单包定论，无差分对照（盲注只发 sleep(5) 不发 sleep(0)）
- ❌ 无截图或截图无 URL bar
- ❌ "可能存在""理论上可以"——要么复现确认，要么标"待验证"
- ❌ 修复建议只有"加强过滤""提高安全意识"

### Playbook 路由表

| 入口信号 | MUST Read |
|---|---|
| Actuator / Swagger / 默认端口 / 弱密码 | `references/playbooks/unauth-access.md` |
| .git / .svn / .env / heapdump / 路径列举 | `references/playbooks/info-disclosure.md` |
| 用户态 ID 可遍历 / 任意 X 越权 | `references/playbooks/arbitrary-x-authz.md` |
| 密码重置 / 支付 / 验证码 / 订单 / 提现 | `references/playbooks/logic-flaws/00-index.md` |
| OAuth / SAML / JWT / redirect_uri | `references/playbooks/oauth-saml-jwt/00-index.md` |
| REST API / BOLA / Mass Assignment / 速率 | `references/playbooks/api-rest/00-index.md` |
| 任何用户输入进 DB | `references/playbooks/sqli.md` |
| 反序列化 / SSTI / XXE / 原型链 / 框架 RCE | `references/playbooks/rce/00-index.md` |
| URL 入参 / 缓存 / Host 注入 | `references/playbooks/ssrf-cache-host/00-index.md` |
| 文件路径入参 / LFI / RFI | `references/playbooks/path-traversal/00-index.md` |
| 上传点 + 解析漏洞 | `references/playbooks/file-upload/00-index.md` |
| 用户输入回显到 HTML / JS | `references/playbooks/xss/00-index.md` |
| 反代 + Content-Length / TE | `references/playbooks/http-smuggling.md` |
| GraphQL endpoint / introspection | `references/playbooks/graphql.md` |
| 并发 / TOCTOU | `references/playbooks/race-conditions.md` |
| ReDoS / 资源不限速 / 算法爆炸 | `references/playbooks/dos.md` |
| APK / IPA / 移动端 | `references/playbooks/mobile.md` |
| LLM agent / prompt 入口 / 工具调用 | `references/playbooks/llm-prompt-injection/00-index.md` |

**两步 Read 模式（已拆分的 playbook）**：目录形式的 playbook（`rce/` / `oauth-saml-jwt/` / `ssrf-cache-host/` / `api-rest/` / `logic-flaws/` / `file-upload/` / `path-traversal/` / `xss/` / `llm-prompt-injection/` / `intranet-postexp/`）第一步只 Read `00-index.md`——它含**子文件路由表**和通用方法论。**不要把 00-index 当 payload 库用**，据子文件路由定位到具体场景后**再 Read 对应子文件**（如 `rce/14-ssti.md` / `oauth-saml-jwt/12-jwt.md`）。单文件形式的 playbook（`sqli.md` 等）直接 Read 即可。

### Coder 角色介入条件

当以下情况发生时，切换 Coder 角色：
- 现有 playbook payload 均被拦截，需要定制绕过 payload
- 发现非标准协议/接口，需要编写专用测试脚本
- 需要将多步手动操作自动化为 PoC 脚本

### Monitor 检查点

每 3 个目标测试完成后，Monitor 角色自检：
- 是否存在重复无效操作？→ 换策略或跳过
- 时间盒是否已过半？→ 优先处理 P0 目标
- 是否有 finding 可以进 Phase 4？→ 及时推进

**通用方法论**（仅在卡壳时 Read，不要预加载）：
- 不知道下一步打什么 → `references/methodology/01-attack-priority.md`
- 被 WAF / EDR 拦 → `references/methodology/02-bypass-toolkit.md`
- 怀疑自己幻觉 / 想检查证据链 → `references/methodology/03-evidence-discipline.md`
- 找不到漏洞点 → `references/methodology/04-control-gap-hunting.md`
- 时间盒优先级排序 → `references/methodology/05-timebox-priority.md`
- 被动侦察具体手法 → `references/methodology/10-passive-recon.md`
- 主动扫描具体手法 → `references/methodology/11-active-scanning.md`
- 利用 / 提权 / 横移具体手法 → `references/methodology/12-exploit-postexp.md`

---

## Phase 4 · Exploit（漏洞利用与后渗透）

**进入条件**：Phase 3 至少一个漏洞已确认可触发。

这是渗透测试区别于 SRC 的关键阶段——不止于发现漏洞，要尝试**深度利用、权限提升和横向移动**，展示真实影响。

### Planner 预规划

```xml
<task_assignment>
  <goal>对已确认漏洞进行深度利用，尝试权限提升和横向移动</goal>
  <confirmed_vulns>Phase 3 确认的漏洞清单</confirmed_vulns>
  <execution_plan>
    1. [漏洞 A：利用路径 → 预期权限提升]
    2. [漏洞 B：利用链构建]
    ...
  </execution_plan>
  <safety_constraints>不破坏数据、不安装后门、关键系统先暂停确认</safety_constraints>
</task_assignment>
```

### Pentester + Coder 协作执行

**MUST Read** `references/methodology/12-exploit-postexp.md`——完整漏洞利用与后渗透工作流，包含利用前影响评估检查表、利用链构建决策、PoC 编写规范、Linux 权限提升（SUID/Capabilities/Sudo/Cron/内核/容器逃逸）、Windows 权限提升（服务/令牌模拟/UAC 绕过/AlwaysInstallElevated）、横向移动（凭据收集 + Pass-the-Hash + Kerberoasting + 域攻击 DCSync/Golden Ticket/BloodHound）、清理流程与验证。

**强制流程**：
1. 对每个已确认漏洞，**先做利用前影响评估**（`12-exploit-postexp.md` 第一节 checklist）
2. 评估**利用链**可行性（单漏洞直接利用 vs 漏洞组合链）
3. 尝试从当前权限**提权**（低权限 → 管理员 / root / SYSTEM）
   - Linux → 按 `12-exploit-postexp.md` 第二节枚举：SUID → Capabilities → Sudo → Cron → 内核
   - Windows → 按 `12-exploit-postexp.md` 第三节枚举：服务权限 → 令牌模拟 → UAC 绕过
4. 如拿到 shell / 凭据 / 内网入口 → **MUST Read** `references/playbooks/intranet-postexp/00-index.md`
5. 横向移动 → 按 `12-exploit-postexp.md` 第四节：凭据收集 → PtH/PtT → 内网探测 → 域攻击
6. **每一步都记录测试过程**：文字描述做了什么操作（不是只甩命令）+ 命令输出截图 / 文本回显 / 网络包。命令型证据同样遵循 Phase 3 的"文字复现流程 + 截图 + 差分 + 复现稳定性"四要素。
7. 测试完成后 → 按 `12-exploit-postexp.md` 第五节**清理流程 checklist**，逐项确认

### Coder 角色任务

- 将手动利用步骤封装为**可复现的 PoC 脚本**
- 编写自动化利用链脚本（如需）
- PoC 脚本必须包含：用途说明、使用方法、清理步骤

**MUST 输出**：

- [ ] **利用链路径**：入口漏洞 → 利用方式 → 达到的权限/影响（带文字逐步说明）
- [ ] **权限矩阵**：当前拿到的最高权限、可访问的资产范围
- [ ] **测试过程记录**：每个利用步骤的文字复现流程 + 命令输出截图 + 复现稳定性（参照 Phase 3 四要素，命令型漏洞同样适用）
- [ ] **证据包**：每步操作的 HTTP 包 / 命令输出 / 截图，文件命名 `VULN-{NNN}-step{N}-*.png`

### 安全约束（不可逾越）

- 不破坏业务数据和服务可用性
- 不安装持久化后门（测试完成后清理所有痕迹）
- 不访问与授权范围无关的敏感数据
- 发现关键漏洞（如域控/核心数据库）时先暂停，与甲方确认后再继续

### Monitor 检查点

- 利用是否在授权范围内？→ 越界立即停
- 是否存在对业务的潜在影响？→ 暂停确认
- 利用链是否可复现？→ 不可复现的不进报告

---

## Phase 5 · Report（报告交付）

**进入条件**：Phase 4 利用链路径 + 证据包就绪。

### Reporter 角色执行

**MUST 流程**（顺序执行）：
1. Read `references/compliance.md` 核对合规红线（不准跳）
2. Read `references/templates/report-pentest.md` 取渗透测试报告模板
3. 按模板结构输出完整报告：

### 报告结构

**1. 执行摘要**（给管理层看）
- 测试范围与时间
- 整体安全评级（Critical / High / Medium / Low / Info）
- 关键发现概述（≤5 条）
- 风险趋势判断

**2. 漏洞详情**（给技术团队看，每个漏洞一节）
- 漏洞名称 + CVSS 4.0 评分 + 风险等级
- 影响范围（受影响的资产/系统）
- **完整测试过程**（以下四要素缺一不可，直接取自 Phase 3/4 的测试过程记录）：
  - 文字复现流程：逐 Step 编号的操作描述 + 每步完整 HTTP 包/命令
  - 差分证明：对照组（真/假/baseline 等）
  - 截图证据：触发截图 + 流量截图（含 URL bar + 时间戳）
  - 复现稳定性：N 次复现记录表
- **修复建议**（三档，具体到代码/配置层）：
  - 立即（24h）：临时缓解措施
  - 短期（1 周）：根因修复
  - 长期：架构/流程改进

> ⚠️ 报告中每个漏洞必须有完整测试过程，否则无法证明漏洞真实有效。无文字流程/无截图/无修复建议的漏洞条目，视为未确认，不得写入报告。

**3. 攻击路径图**
- 从入口到最深渗透点的完整路径可视化
- 每个节点标注：漏洞类型 + 权限变化

**4. 修复优先级矩阵**
```
漏洞 | CVSS | 利用难度 | 业务影响 | 修复优先级 | 建议修复方案
-----|------|---------|---------|-----------|------------
```

**5. 清理确认**
- [ ] 所有测试账号已清理
- [ ] 所有上传文件已删除
- [ ] 所有临时后门/shell 已关闭
- [ ] 无持久化修改残留

---

## 优雅终止机制

借鉴 Barrier Tools 模式，在以下情况触发优雅终止：

### done 信号
- 所有 P0-P1 目标已测试完毕
- 时间盒已到
- 用户主动要求停止

### ask 信号（暂停等待用户输入）
- 发现关键系统漏洞，需确认是否继续深入
- 即将执行可能影响业务的操作
- 遇到授权边界模糊的资产
- 需要用户提供额外凭据/信息

### Reflector 自省（自动触发）
- 连续 3 次操作无新发现 → 反思策略，建议用户：
  - 切换攻击面
  - 降级为灰盒（请求更多信息）
  - 结束当前 Phase 进入下一阶段
- 接近 Phase 时间盒上限 → 输出当前阶段已有成果，建议推进

---

## MCP 工具集成

PenT3st **不预设任何工具依赖**，根据用户环境动态适配。

### 已知 MCP 工具生态

| 工具 | 能力 | 典型用途 |
|------|------|---------|
| **kali-mcp** | nmap/metasploit/sqlmap/nikto/gobuster/hydra 等 | 全栈渗透 |
| **burp-mcp** | 流量拦截/扫描/重放 | Web 漏洞测试 |
| **frida-mcp** | 动态 Hook/SSL pinning bypass | 移动端/客户端测试 |
| **adb-mcp** | Android 设备交互 | 移动端测试 |
| **chrome-devtools** | 浏览器自动化/截图/网络监控 | Web 前端测试 |
| **nuclei-mcp** | 模板化漏洞扫描 | 批量漏洞验证 |
| **subfinder-mcp** | 子域名枚举 | 信息收集 |
| **shodan-mcp** | 互联网资产搜索 | 被动侦察 |
| **semgrep-mcp** | 静态代码分析 | 白盒/灰盒测试 |

### 无工具降级策略

当某能力域无对应 MCP 工具时，PenT3st 自动降级：
1. **给出等效命令**：输出用户可在本地终端执行的命令
2. **等待结果回传**：用户执行后粘贴结果继续分析
3. **纯手动指导**：提供详细步骤，用户自行操作
4. **不中断流程**：缺工具不卡 Phase，用可用方式继续推进
