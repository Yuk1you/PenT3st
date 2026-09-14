# 内网渗透 / 后渗透 — 决策索引

> ⚠️ **合规警告**：内网横向 / 后渗透必须在**明确授权范围内**进行。本目录内容默认用于**红队评估 / 授权渗透测试 / 已明确允许后渗透的 HVV**。执行前必读 [`../compliance.md`](../compliance.md) 确认授权边界。

---

## 子文件路由(Phase 4 / 内网后渗透阶段读哪一份?)

| 当前阶段 | 任务 | MUST Read |
|---|---|---|
| 已拿 shell,身份是 user | 找系统凭据 / 浏览器密码 / Kerberos 票据 | `10-credentials.md` |
| 已有一组凭据,需要打第二台机器 | SMB / WMI / PsExec / RDP / WinRM / Pass-the-Hash / Ticket | `11-lateral.md` |
| 拿到 user,需要 root/SYSTEM | UAC bypass / sudo 提权 / 内核 / SUID / Token | `12-privesc.md` |
| 防护强,被 AV/EDR 拦 | AMSI bypass / ETW patch / DLL sideload / unhook | `13-evasion.md` |
| 已进域,要打 DC | Kerberoasting / AS-REP / Golden / Silver / DCSync / Constrained | `14-domain.md` |
| 内网不出网 | reGeorg / frp / Chisel / ICMP / DNS tunnel / SOCKS / ssh -D | `15-tunneling.md` |
| 进了陌生网,先摸地形 | 主机发现 / 端口扫描 / Bloodhound 收集 / hostname / hostfile | `16-recon.md` |
| 拿到 admin,要维持长期访问 | 服务 / 计划任务 / 注册表 / WMI 订阅 / Logon / SSP / Skeleton Key | `17-persistence.md` |
| 目标含 Exchange | NTLM Relay / EWS / OAB / CVE-2021-26855 / Mailbox Export | `18-exchange.md` |
| 目标含 ADCS / 证书服务 | ESC1–ESC8 模板滥用 | `19-adcs.md` |
| 目标含 SharePoint | 信息收集 / SOAP / API / OneDrive 同步 | `20-sharepoint.md` |

---

## 数据规模(原文档)

| 类别 | 数量 |
|---|--:|
| 凭证窃取 | 20 |
| 横向移动 | 16 |
| 权限提升 | 15 |
| 免杀与规避 | 14 |
| 域渗透攻击 | 14 |
| 隧道代理 | 13 |
| 信息收集 | 12 |
| 权限维持 | 12 |
| Exchange攻击 | 5 |
| ADCS攻击 | 5 |
| SharePoint攻击 | 2 |
| **合计** | **128** |

---

## 动作尺度（提权 / 横向 / 持久化能到哪一步）

**本目录只描述攻击手法与利用链，不自行定义许可边界。** 拿到 Webshell / 凭据 / 内网入口之后能做什么、不能做什么，一律以 [`../compliance.md`](../compliance.md) 为准——该文件是动作尺度的**唯一裁决点**，其「🔀 持久化与痕迹（按作业模式分支）」与「⚠️ 需确认后执行」两节直接回答了本目录最常见的越界问题。

---

## 与红线规则的关系

- 本文件子目录所有 payload **只在已获授权时使用**（红队评估、合同渗透测试、HVV 演练）
- 未获明确授权时，这些 payload 仅用于"我能做什么"的认知，不准实操
- 授权未覆盖后渗透时，报告中把内网影响写成**受控推断**（"假设具备内网持续访问，可进一步…"），不写成"已演示横向"
