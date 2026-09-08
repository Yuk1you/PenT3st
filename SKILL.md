---
name: PenT3st
description: "渗透测试实战工作流 skill。融合多 Agent 协作、动态工具检测、智能任务规划。5 阶段方法论（scope → recon → discover → exploit → report）、19 类攻击 playbook、305 个结构化 payload、263 个 WAF/EDR 绕过变体、国产组件指纹库。当用户提到 \"渗透测试 / pentest / 渗透 / PenT3st / 红队\" 或给出明确目标让你测试时触发。"
argument-hint: "<target-or-scope-or-phase>"
level: 2
---

# PenT3st — 渗透测试实战工作流

这是一个**带强制 checkpoint 的渗透测试工作流**，不是参考手册。每个阶段有 MUST 输出，未通过不进下一阶段。详细 payload / playbook / 案例**按需 Read**，不准凭记忆生成。

授权渗透测试**指向性强**——有明确的目标和授权，确认范围后直接进入信息收集与攻击面测绘，目标是深度突破而非广泛搜索，追求链路完整（权限/数据/全链）而非发现即止。

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
2. **不准编造案例/编号**。引用任何外部案例或编号前，必须 Read 实际出处文件（说不出文件路径就别引）；无法给出处的观察一律标"待验证"。
3. **无证据不下结论**。无 HTTP 包/截图/视频时只能写"待验证 / 假设"，不写"已确认 / 发现漏洞"。
4. **出 scope 立即停**。任何时候发现要测的资产不在 Phase 1 已确认的授权范围 → 立即停手，回到 Phase 1 重核。
5. **合法合规**。必须确认已获得合法授权再开始测试，未授权渗透测试是违法行为。

---

## 行为纪律（全程适用）

**A. 范围 / 上报纪律（硬性）**
1. **范围双检硬性规定**：每个候选上报前必须自行完成双检——①是否在授权范围内 ②是否命中得分条款（有实质影响：权限/数据/完整链；oracle/配置泄露/静态证据不给分）。**双检不过 → 不上报仅留档**，不等用户问、不等提醒。范围判定**前置到侦察阶段**，不是发现漏洞后才判。
2. **测绘数据有时效**：外部测绘/EASM 快照（子域、存活、指纹）可能过期，测前先自行验活（如 DNS/HTTP 探测），别信过期的资产清单。
3. **负结果诚实记录**：负结论（排除/闭合/不可达）也要落盘成档，如实登记——避免浪费申报额度、避免后续重复打已闭合资产。

**B. 会话 / 节流纪律**
4. **会话内清单化机制**：用户口头规则立即写入会话持续检查清单，每次探测/上报前强制过一遍，避免"用户说完跟没听见一样"。
5. **同主域封禁连坐**：同一主域下多子系统常共享 WAF 封禁域，并发打点会连坐全封——需全局节流与错峰，多系统打点前先评估连坐风险。
6. **请求预算**：单脚本默认请求预算 ≤3，超预算必须人工确认；脚本内置异常检测 + 指数退避。新目标先探每会话/每 IP 请求配额再定打法。
7. **跨会话笔记机制**：接替会话先读会话笔记（SESSION_NOTES），结束前更新——避免重复打已闭合资产、避让活跃会话。

**C. 技术判别纪律**
8. **证据自动化落盘**：探测脚本自动落盘 `summary.txt` + 关键响应**原始包**（+脚本+日志三件套）；只留原始数据，不加自造字段。原始包是评审认可的唯一证据形态。
9. **响应异常三分法**：连接/响应异常先三分判别——本地准入劫持 / 目标 WAF / 服务端限流（用响应体特征指纹），再对症，不误判目标防护。
10. **准确接口名先提取**：不盲猜接口方法名，先 grep 已固化页面提取准确 `.do`/端点名再发请求（盲猜浪费配额且触发服务端限流）。
11. **横向扩展 + 边界确认**：发现单点越权/IDOR 后用字典/相邻号扩展，并做 3 跨度 + 边界外对照确认数据边界，量化影响面再上报。
12. **短信轰炸向量合规线**：send-code 无频控可作骚扰向量但**非权限/数据**——识别后只留档不深测，与"写操作合规停手"并列。

**扩大利用原则**：有漏洞就继续利用扩大战果，不要停在"发现"。发现 → 验证 → 扩大利用（拉数据/提权/横向）→ 停在拿不到更多为止。

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
5. **网络搜索默认用 Tavily**：查 CVE / 最新漏洞 / 组件情报 / 漏洞利用细节等网络搜索，默认调用 Tavily API；WebSearch 不可用/返回空时直接用 Tavily，不反复重试。跨平台调用模板（PowerShell / bash）：
   ```powershell
   $k=$env:TAVILY_API_KEY
   $body=@{api_key=$k; query="<组件名 版本 CVE 漏洞>"; search_depth="advanced"; max_results=5}|ConvertTo-Json
   Invoke-RestMethod -Uri "https://api.tavily.com/search" -Method Post -ContentType "application/json" -Body $body
   ```
   ```bash
   curl -s -X POST https://api.tavily.com/search -H "Content-Type: application/json" \
     -d "{\"api_key\":\"$TAVILY_API_KEY\",\"query\":\"<组件名 版本 CVE 漏洞>\",\"search_depth\":\"advanced\",\"max_results\":5}"
   ```
   （未配置 `TAVILY_API_KEY` 时，用环境可用的任一网络搜索工具替代）

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
| 指纹命中任何组件（国产/开源组件，有版本更好） | **MUST** 先网络搜索（Tavily，模板见上文"工具调用原则"第 5 条）查该组件最新 CVE，近 12 个月优先 → 针对性验证 |

**指纹 → CVE 规则**：指纹识别命中组件后，**不直接打记忆中的历史 payload**——先查该组件最新 CVE 再针对性验证（新披露 CVE 的验证脚本与利用面往往更完整，命中率高）。

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

### 场景专项检查（强制，命中场景才执行）

- **登录/注册验证码**：先**辨型**——vcode 字段绑的是图形码还是短信码（看输入框 minlength/maxlength、按钮文案"换一张"vs"获取验证码"、JS sendCode 逻辑与渲染顺序）。图形码通常只生成 imageCodeId 不参与服务端校验。**同一方向连续 3 次同错 → 触发"换假设"**，禁止继续同方向重试。再验证码复用/消费时机：同 token 重复提交错误码，响应恒定=仅成功时消费（可复用）；含递增计数器=可当枚举 oracle。短信双因素：send-code 无账号枚举、确认真发短信后**立即停手**（扰民线），定性"双因素锁死"即闭合。
- **注入类测试（任何参数疑似进 DB/SQL）**：强制**三段差分验证**——
  1. 单引号 → 异常体差分（状态码+长度+响应体三重对比）
  2. 注释符（`--`/`#`）→ 空结果/异常差分（区分 WAF 拦截与语法变化）
  3. `%27` 等 URL 编码重放 → 确认 WAF 是否只做明文匹配
  **附加规则**：单包异常 ≠ 注入成立，WAF 403 是拦截行为不是注入证据；**500 语法错 + 200 注释闭合的成对差分才是注入实锤**；WAF 拦截页带唯一编号/体积递增 = 动态计数升级（穿透词清单需实测：编码后的 `%'`/`--`/`||` 常放行，`and`/`or`/`union` 多形态全拦）；布尔/时间盲注遇 WAF 动态策略不稳 → 按抽样原则停手保已确认证据；ORM 参数化类负结论 → 明确写"已排除"并关闭该线。

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
| GraphQL endpoint / introspection | `references/playbooks/graphql.md` — introspection 关闭时从前端 bundle 静态提取 schema（nodes/mutations/权限模型）再批量探测；多租户 HTTP 自定义头（`tenant:`）做常见值枚举看鉴权差分 |
| 加密锁 / U盾 / 扫码 / 桌面控件（ActiveX/WebSocket 本地端口）/ 客户端自述标识 | 客户端自述身份认证绕过：校验若全在客户端（明文硬编码 PIN）+ 服务端只收自述标识（单位代码/序列号）→ 认证绕过点；规整编号猜高权限账号（全零/顺位/全 F） |
| 登录/注册验证码提交前 | 验证码辨型检查点（见上方"场景专项检查"）— 先辨型再打 |
| 并发 / TOCTOU | `references/playbooks/race-conditions.md` |
| ReDoS / 资源不限速 / 算法爆炸 | `references/playbooks/dos.md` |
| APK / IPA / 移动端 | `references/playbooks/mobile.md` — 静态逆向输出四件套（后端域名/UAT 路径/API 全集/硬编码密钥）；多市场源先 MD5 diff 确认同一包；**硬编码凭证 ≠ 权限**（先验用户态/内网隔离再定性）；静态清单中"协议未知/有签名/有频控"链路转动态（模拟器 + 系统级 CA 信任 + UI 走链）闭环 |
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

这是**利用阶段**——不止于发现漏洞，要刻意**扩大战果**（见"行为纪律"的扩大利用原则），尝试**深度利用、权限提升和横向移动**，展示真实影响。

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

### 高价值通道优先（认证绕过类）

**客户端自述身份认证绕过**（加密锁/U盾/控件通道，发现即优先，常达严重级）：
1. 登录页找非传统通道（加密锁/U盾/扫码/桌面控件 ActiveX/WebSocket 本地端口）
2. 逆向前端 JS，确认 PIN/密码/签名校验是否**全在客户端**完成（明文硬编码 PIN 常量 = 强信号，无服务端往返）
3. 服务端建会话接口若只收自述标识（单位代码/序列号/用户名）→ 绕过点成立
4. 差分验证：无效标识（拒绝/空响应）vs 规整值（服务端真实处理）
5. 规整编号猜高权限账号：全零/顺位/全 F 常指向运营方超管；相邻编号差分（登录计数累加证服务端宽松匹配）
6. **全程只读**，写接口仅识别不执行（合规红线）

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
8. **存量漏洞复测（长期/多轮任务）**：对已报漏洞周期复测——已修复的存量证据（截图+原始响应）仍计分；未修复复测可作催办升级材料；同域修复一个点后快速扫同 pattern 是否全量修复。同服务多协议暴露面可绕单端口封禁（续测前确认不违反授权范围）。

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
