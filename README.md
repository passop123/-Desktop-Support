# 常见办公电脑故障排查手册
作者：pangxiegong  
时间：2025-7-13
## 简介
目标：快速定位并解决常见办公电脑/网络问题，若无法解决按“上报流程”提交工单并附上必要信息。适用于技术支持一线人员或办公自查。

---

## 常用检查项（先做这几步，能解决 >70% 问题）
1. 确认电源/线缆（电源、网线、显示器线）已插牢并有指示灯。  
2. 重启设备（电脑、路由器、打印机）— 先重启再做深层排查。  
3. 记录错误提示（截屏或拍照）并记录发生时间。  
4. 若网络问题，先做 ping 测试并记录结果（见“快速命令”）。

---

## 快速命令（Windows）
- 打开命令提示符（Win + R → 输入 cmd）：
  - ipconfig /all    （查看本机 IP、网关、DNS）
  - ping 8.8.8.8 -n 4    （测试公网连通性）
  - ping baidu.com -n 4  （测试域名解析）
  - nslookup baidu.com   （DNS 解析）
  - ipconfig /flushdns   （清 DNS 缓存）
  - netsh winsock reset  （重置 Winsock）
- 打印机服务重启（管理员 cmd）：
  - net stop spooler
  - net start spooler

---

## 常见问题与处理步骤（按顺序执行）

### 1) 无法上网（网页打不开 / 无网络）
- 现象：网页打不开、Office 无法登录、邮件收发失败。  
- 先做（2 分钟）：
  1. 检查网线/Wi‑Fi 是否连上，路由器指示灯是否正常；重启路由器与电脑。  
  2. cmd → ping 8.8.8.8 -n 4（记录是否能通）；cmd → ipconfig /all（检查是否有有效 IP，非 169.254.x.x）。  
  3. 若能 ping IP 但域名解析失败，执行 nslookup 或 ipconfig /flushdns。  
- 上报条件：网线断、路由器无响应、IP 为 169.254.x.x、重置无效。上报时附 ipconfig /all 输出与 ping 截图。

### 2) 打印机无法打印
- 现象：打印任务卡住、打印机离线或报错。  
- 先做（5 分钟）：
  1. 检查打印机电源、网络/USB 连接、纸张与墨粉。  
  2. 取消打印队列任务，重启打印机与电脑。  
  3. 若为网络打印机，ping 打印机 IP。必要时重启 Print Spooler 服务。  
- 上报条件：物理故障、驱动需要管理员安装或网络不通。上报时附打印机型号、IP、错误截图。

### 3) 登录失败 / 密码问题
- 现象：无法登录电脑/系统/邮箱、提示账户被锁。  
- 先做（3 分钟）：
  1. 检查 Caps Lock/Num Lock；确认用户名格式（domain\user 或 邮箱）。  
  2. 尝试重启后再登录；在另一台设备确认账号是否可用。  
- 上报条件：多次尝试无效或提示账户锁定。上报时附错误提示与最后尝试时间。

### 4) 电脑很慢 / 程序卡顿
- 现象：开机慢、应用无响应。  
- 先做（5–10 分钟）：
  1. 打开任务管理器（Ctrl+Shift+Esc），截图 CPU/内存/磁盘占用情况。  
  2. 关闭占用高的非必要进程，清理临时文件，检查磁盘空间。  
- 上报条件：重启/清理无效、磁盘有异常或持续高占用。上报时附任务管理器截图与最近重启时间。

### 5) VPN / 远程桌面无法连接
- 现象：VPN 连接失败或 RDP 超时。  
- 先做（5 分钟）：
  1. 确保网络可达（ping 8.8.8.8），重启 VPN 客户端与电脑。  
  2. 检查证书/双因子认证步骤是否正确。  
- 上报条件：服务端故障或证书账号问题。上报时附客户端错误日志和截图。

### 6） 电脑蓝屏 / 内核崩溃（BSOD）处理流程

- 现象：Windows 出现蓝屏（带 STOP code 或 BUGCHECK），或系统重启后提示“系统发生问题需重启”等内核崩溃表现；macOS 出现 kernel panic（屏幕黑/重启并显示 panic 信息）。
- 先做（尽量在 5 分钟内完成现场记录）：
  1. 记录时间点与重现步骤：记录发生时间、正在运行的程序、是否插有外接设备、是否近期安装了新驱动或补丁。拍照/截图蓝屏上的 STOP code/错误信息（手机拍照最稳妥）。
  2. 不要立即清空或删除任何 Dump/日志文件，保留 C:\Windows\Minidump 和 %SystemRoot%\MEMORY.DMP 文件。
  3. 尝试重启并进入安全模式（按 F8 或在恢复环境选择“安全模式”）。观察是否能稳定启动并复现蓝屏。
  4. 若系统无法启动或每次启动均蓝屏，做以下收集（见“信息收集”）。

- 现场快速检查命令（在管理员 PowerShell / CMD 中）：
  - 查看 minidump：dir C:\Windows\Minidump
  - 导出系统信息：systeminfo > C:\Temp\systeminfo.txt
  - 列出驱动：driverquery /v > C:\Temp\drivers.txt
  - 检查 SFC：sfc /scannow
  - DISM 修复（Windows 10/11）：DISM /Online /Cleanup-Image /RestoreHealth
  - 检查磁盘：chkdsk C: /f /r（注意需要重启）
  - 内存检测：运行 Windows 内存诊断（mdsched.exe）或安排 memtest86
  - 查看事件日志（按时间区间）：事件查看器 → Windows 日志 → 系统，过滤 Critical/Error，与蓝屏时间点匹配

- Dump / 日志收集（上报与分析必备）：
  - 收集路径与文件：
    - 小型内存转储：C:\Windows\Minidump\*.dmp
    - 完整/内核转储（如配置）：%SystemRoot%\MEMORY.DMP
    - 事件查看器 System log 的相关条目（时间窗口 ±5 分钟）
    - systeminfo、driverquery 输出、最近安装的驱动/软件清单（过去 7 天）
    - BIOS/UEFI 版本、固件更新历史
    - 如果有外接硬件（扩展坞、USB 设备、外接显卡等），列出并尽量复现/断开测试结果
  - 工具建议：
    - WinDbg（Microsoft Debugging Tools）+ 符号：使用 !analyze -v 分析 dump（符号路径 SRV*c:\symbols*https://msdl.microsoft.com/download/symbols）
    - NirSoft BlueScreenView / WhoCrashed（快速定位可疑驱动）
    - memtest86（内存稳定性测试）

- 常见 STOP CODE 指引（示例）：
  - IRQL_NOT_LESS_OR_EQUAL（通常与驱动或硬件中断相关）
  - MEMORY_MANAGEMENT / PFN_LIST_CORRUPT（可能为内存/驱动/磁盘问题）
  - PAGE_FAULT_IN_NONPAGED_AREA（内存/驱动/反病毒冲突）
  - DRIVER_IRQL_NOT_LESS_OR_EQUAL（特定驱动）
  - KMODE_EXCEPTION_NOT_HANDLED（驱动或内核模块异常）
  - SYSTEM_SERVICE_EXCEPTION（图形驱动/系统服务异常）
  - （若 STOP code 指向具体驱动 *.sys，优先考虑回滚/更新该驱动）

- 排查建议顺序（从低成本到高侵入）：
  1. 记录并保留 Dump 与日志；拍照蓝屏信息。
  2. 断开近期新增外设（USB/扩展坞/打印机/外置显卡）并重启验证。
  3. 进入安全模式，若安全模式稳定，排查第三方驱动/启动项（msconfig/任务管理器 → 启动）。
  4. 回滚或更新显卡/网卡/存储控制器等关键驱动；若近期有 Windows 更新或驱动更新，尝试回滚。
  5. 运行 SFC / DISM / chkdsk / 内存检测（memtest）；根据检测结果决定是否更换内存条或硬盘进一步检测。
  6. 若怀疑驱动，使用 Driver Verifier（谨慎，可能触发蓝屏以捕获问题），并收集新的 dump 交 L2 分析。
  7. 如怀疑硬件（内存错误、硬盘故障、主板异常），联系硬件支持并记录保修信息，按资产流程更换或 RMA。

- 上报条件（需升级到 L2 / 硬件 / 安全团队）：
  - 连续多次蓝屏或每次启动均蓝屏且无法进入系统。
  - Dump 分析显示内存/硬件错误（memtest/SMART 报告有错误）。
  - Dump 指向第三方驱动且现场无法回滚或更新（需厂商支持）。
  - 发生在多个用户/同一硬件型号或批次（可能为批量硬件故障）。
  - 可疑安全事件（dump 中出现异常模块/未知驱动或与已知恶意模块相关）。

- 上报时必须提供的内容（模板字段）：
  - 报告标题：BSOD - {主诉简短描述} - {主机名} - {日期时间}
  - ITSM 工单号：
  - 发生时间：
  - 主机信息：主机名、资产编号、序列号、OS 版本与补丁号（从 systeminfo）
  - 停机表现：STOP code（如 0x0000001E / MEMORY_MANAGEMENT）、错误文字描述、是否重现
  - Dump 路径与文件（附下载/压缩）：C:\Windows\Minidump\xxx.dmp、%SystemRoot%\MEMORY.DMP
  - 近期变更：最近 7 天内安装的驱动/软件/Windows 更新清单
  - 已做检查项与结果：SFC/DISM/chkdsk、memtest、磁盘 SMART、是否能进入安全模式
  - 是否为现场可修复（断开外设/回滚驱动可恢复）或需更换硬件
  - 联系人与联系电话、优先级与影响范围（单机/部门/多人）
  - 附件：蓝屏照片、事件查看器截图、driverquery/systeminfo 输出、memtest/SMART 报告

- macOS kernel panic 简要处理（如需支持 mac）：
  - 记录 panic 信息（屏幕上会出现 panic 内容），重启并在 Console.app 中导出 panic logs（/Library/Logs/DiagnosticReports/）。
  - 断开外设并重启；如持续 panic，启动至安全模式（按 Shift）或恢复模式排查。
  - 常见原因：第三方内核扩展（kext）、外设驱动、硬件故障（内存/SSD）。
  - 收集系统信息并上报至 L2/mac 支持。

- 备注与建议：
  - 尽量保留原始 dump 与日志以便厂商或 L2 使用 WinDbg 深入分析。

### 6） 域控制器（Active Directory / 域控）管理与 Runbook

概述
- 适用范围：公司 AD 域架构、域控制器（DC）、域服务（DNS、Kerberos、SYSVOL/DFS-R）、组策略（GPO）及与终端相关的目录服务支持。
- 目标：保证域控可用性、认证与目录服务稳定、复制可靠、数据完整与安全合规；快速响应域控相关事件并提供恢复能力。

角色与职责（建议）
- AD 管理员（L2/L3）：域控日常维护、域/林级别变更、FSMO 管理、系统态备份与恢复、复杂复制故障排查。
- 桌面运维（L1/L2）：账号/密码/解锁、计算机加入域、GPO 应用问题初步排查、收集日志并在必要时上报 AD 管理员。
- 安全团队：域安全策略（账户策略、Kerberos、LDAP/LDAPS、LAPS）与审计、应对可疑认证行为。
- 资产/变更管理：域控补丁、重启与变更需按变更流程审批。

SLA（参考）
- 认证中断（影响多人/关键业务）：响应 15 分钟，协同修复/回滚 2 小时内（视影响扩大级别升级）。
- 单用户认证问题：响应 30 分钟，解决或升级 8 小时内。

常用工具
- Windows Server 内建工具：Event Viewer、Windows Server Backup、wbadmin、ntdsutil、dcdiag、repadmin、nltest
- PowerShell AD 模块：ActiveDirectory（Get-ADUser / Get-ADComputer 等）
- 其他：Sysinternals、DNS 管理工具、备份系统（VSS 支持）

日常维护检查（例）
- 每日：检查 DC 状态（dcdiag 简要），查看重要复制错误/高严重性事件、确保 PDC 时间同步正常。
- 每周：检查复制拓扑（repadmin /replsummary）、检查 SYSVOL/DFS-R 健康、补丁合规与重启计划。
- 每月：FSMO 角色持有检查、系统态备份验证、GPO/权限审查、关键服务恢复演练。

常见问题 Runbook（按场景）

A. 用户账号相关（解锁 / 重置密码 / 创建 / 禁用）
- 场景：用户无法登录域账户 / 账号被锁定 / 需创建新用户。
- L1 检查（3-10 分钟）：
  1. 验证用户身份（按公司流程）。确认最近更改/重置记录。
  2. 检查是否为密码过期或锁定（在 DC 上或通过 AD 管理工具）。
  3. 若需解锁或重置：使用 AD 用户管理或 PowerShell（示例见“常用命令”）。
  4. 指导用户清除本地/手机缓存密码（Outlook/Mobile），并在必要时强制用户在下次登录更改密码。
- 上报条件：无法在多台 DC 上查询到账号、AD 有复制错误、疑似安全事件（暴力破解/异常锁定）。
- 上报时附：用户名、错误提示、最后尝试时间、相关事件日志截图（Security/Directory 服务）。

B. 计算机无法加入域 / 域信任问题
- 先做（5-15 分钟）：
  1. 检查网络与 DNS（客户端能否解析 DC 名称与 SRV 记录）。
  2. 确认时间同步（差异不应超过 5 分钟），检查 NTP 配置。
  3. 确认用来加入域的账号拥有权限（域加入权限/临时管理员）。
  4. 若出现计算机已存在于 AD，先删除旧计算机对象或重命名后再加入。
- 上报条件：DNS 配置错误、多个 DC 无法解析、复制问题或证书/LDAP 问题。

C. GPO 无法下发 / 策略不生效
- 检查（5-30 分钟）：
  1. 客户端运行 gpupdate /force 并检查结果。
  2. 在客户端运行 gpresult /h gp.html 或 Get-GPResultantSetOfPolicy 检查策略应用情况。
  3. 检查 SYSVOL 是否被正确复制（SYSVOL/\\domain\SYSVOL 是否包含 GPO 文件）。
  4. 在 DC 上检查事件查看器（File Replication Service / DFS-R）与 GPO 相关事件。
- 上报条件：SYSVOL 丢失/复制失败、GPO 文件损坏、权限问题。

D. DC 复制失败 / SYSVOL 不同步
- 初步排查（10-30 分钟）：
  1. 在 DC 上运行：repadmin /replsummary、repadmin /showrepl <DCName>，查看失败项与错误代码。
  2. 检查 DNS、网络连通性、时间同步与端口（TCP/UDP 389/636/3268/3269/88/RPC）。
  3. 检查事件日志：Directory Service、DNS Server、DFS Replication。记录错误 ID 与时间。
  4. 若为 DFS-R 问题，使用 dfsrdiag 或检查 DFSR 事件；若为 FRS（旧）则检查 FRS 日志。
  5. 尝试强制复制：repadmin /syncall /APed（谨慎）。
- 上报条件：repadmin 无法修复、错误指向数据库损坏、跨站点大量失败、SYSVOL 丢失。
- 上报需附：repadmin 输出、事件日志截屏、网络/防火墙变更记录、近期补丁/变更信息。

E. 单个 DC 无响应 / 整个域认证中断（紧急）
- 立即行动（按优先顺序）：
  1. 确认影响范围（单 DC 还是全部 DC）。检查 DNS 记录与 SRV 解析（nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>）。
  2. 检查 DC 主机的硬件/虚拟机健康（监控、主机日志）。
  3. 尝试远程服务重启（NTDS、DNS、DFS-R）；若不能远程，安排机房/宿主机操作。
  4. 若 PDC emulator 故障且需要时间同步/密码更改功能，切换或临时安排时间源。
  5. 若需临时移除故障 DC（例如严重磁盘损坏），按变更/资产流程执行并通知依赖团队。
- 上报条件：无法恢复、需转移 FSMO 角色、数据损坏或怀疑安全事件。
- 上报需附：dcdiag 输出、事件日志、主机层面状态、已尝试的恢复步骤。

F. 恢复 DC / 系统态恢复（有备份时）
- 注意：恢复 DC 风险高，按审批流程执行并在隔离环境先做演练。
- 非权威恢复（常用）：从系统态备份恢复 DC 系统态（wbadmin、第三方备份），在恢复后让复制自然修复。
- 权威还原（仅在特定场景）：使用 ntdsutil 设置为 authoritative restore（谨慎并仅在专家指导下）。
- 元数据清理（移除已损坏/离线 DC 的剩余元数据）：使用 ntdsutil 或 AD 用户和计算机中的删除后触发元数据清理。

常用命令与 PowerShell 示例
- 基本 DC 健康检查：
  - dcdiag /v > C:\Temp\dcdiag.txt
  - repadmin /replsummary > C:\Temp\replsummary.txt
  - repadmin /showrepl * /csv > C:\Temp\showrepl.csv
- DNS 与 SRV 检查：
  - nslookup
  - nslookup -type=SRV _ldap._tcp.dc._msdcs.<domain>
- PowerShell（需 AD 模块）：
  - Import-Module ActiveDirectory
  - Get-ADDomainController -Filter * | Select-Object Name,OperatingSystem,IPv4Address
  - Get-ADUser -Identity "domain\\user" -Properties LockedOut,PasswordLastSet
  - Unlock-ADAccount -Identity "user"
  - Set-ADAccountPassword -Identity "user" -Reset -NewPassword (ConvertTo-SecureString "NewPass" -AsPlainText -Force)
  - New-ADUser -Name "CN" -SamAccountName "user" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true
  - Get-ADReplicationFailure -Scope Site | Format-Table
  - Get-EventLog -LogName System -After (Get-Date).AddHours(-6) | Where-Object {$_.EntryType -eq "Error"}
- FSMO 相关：
  - netdom query fsmo
  - Get-ADForest | Select-Object SchemaMaster,DomainNamingMaster
  - Move-ADDirectoryServerOperationMasterRole -Identity <TargetDC> -OperationMasterRole PDCEmulator,SchemaMaster,...
- 元数据清理 / ntdsutil（谨慎使用）：
  - ntdsutil -> metadata cleanup -> select operation target -> remove selected server

日志与关键事件 ID（常查）
- Directory Service、DNS Server、DFS Replication、System、Security 中与复制、NETLOGON、DNS、时间同步、服务崩溃相关的错误/警告。
- 示例关注点（按具体事件再检索 ID）：
  - 复制错误（repadmin 错误输出对应的事件）
  - DFSR/FRS 事件（SYSVOL 复制失败）
  - Kerberos / 认证失败高频（可能为密码/时间/密钥问题）
  - 证书/LDAPS 错误（证书到期或链错误）

备份与恢复策略（建议）
- 必须有定期的 System State 备份（至少每日或视变更频率），并定期做恢复演练。
- 备份保留策略与加密存储、备份验证（每月至少一次恢复演练）。
- 在多站点环境，确保远程站点也有本地备份与恢复路径。

安全建议（域控专用）
- 最小化直接登录到 DC 的账号（仅管理员），使用受保护的管理工作站（PAW）。
- 使用 LAPS 管理本地管理员密码，启用 LDAPS 并保护证书生命周期。
- 将域管理员分层（Tiered admin model），限制特权账号暴露面。
- 审计关键事件（登录、权限提升、组成员变更），并将日志集中化（SIEM）。
- 定期扫描并修补域控（补丁窗口、重启计划与变更审批）。

上报模板（域控事件）
- 标题：AD Incident - {简短描述} - {域/主机名} - {日期时间}
- 工单号：
- 影响范围（单 DC/多 DC/全域）：
- 主要症状（认证失败/复制错误/DFS-R/SYSVOL/服务宕机）：
- 发生时间与持续性：
- 已采集证据：dcdiag/repadmin 输出、事件日志截图、备份可用性说明
- 已尝试的步骤：列出已执行的命令与结果
- 建议/下一步动作（如 FSMO 转移、系统态恢复、硬件更换）
- 联系人/联系电话/优先级

附：何时要紧急升级到 L3 或厂商支持
- 大面积认证中断、FSMO 角色持有 DC 丢失且无法恢复、系统态数据库（NTDS）损坏、备份恢复失败、怀疑针对 AD 的攻击（如 DCShadow、Kerberos 离线攻击迹象）等场景需立即升级。

备注
- 域控相关操作风险高（尤其是元数据清理、FSMO seize、权威还原），执行前务必审批并在维护窗口内操作，且先在测试环境或备份环境演练。
- 若需要，我可以把上述章节插入你的 TXT 文档、生成合并后的文件并提供下载链接，或将某些 Runbook（例如“授权还原 / ntdsutil 操作步骤”或“SYSVOL DFS-R 恢复流程”）扩写为逐步操作指南与命令脚本。


### 7）企业安全策略（Endpoint & AD 相关）

目的与适用范围
- 目的：建立统一、可执行的安全策略，保护公司信息资产、终端与目录服务，降低安全事件风险并保证业务连续性。
- 适用范围：公司所有企业终端（桌面/笔记本/移动设备）、域控/目录服务、远程访问、BYOD、第三方接入与相关运维流程。

安全原则
- 最小权限（Least Privilege）：用户与进程仅授予完成任务所需的最低权限。
- 防御深度（Defense in Depth）：多层防护：网络、端点、身份与数据保护相互补足。
- 可审计与可追溯：关键操作与变更须留痕并集中化日志管理。
- 主动检测与快速响应：建立监控、告警与事件响应机制，缩短响应时间。
- 持续改进：定期评估、验收并修正策略与控制项。

角色与职责（高层）
- 信息安全负责人：策略制定、审批、合规评估与威胁情报协调。
- 桌面运维/AD 管理员：策略执行、补丁管理、终端加固与目录安全维护。
- 服务台（L1/L2）：按策略执行日常操作（密码重置、初步排查）并按流程上报异常。
- 业务单位主管：审批特殊授权/例外需求并配合合规检查。

关键控制项
1) 身份与访问管理（IAM）
  - 强制使用企业 SSO 与多因素认证（MFA）访问关键系统（邮箱、VPN、管理控制台）。
  - 禁止常驻本地管理员权限；采用临时特权审批与 Just-In-Time 管理（JIT）。
  - 本地管理员密码使用 LAPS 管理并周期性审计。

2) 端点安全
  - 强制全盘加密（Windows BitLocker / macOS FileVault）。
  - 部署 EDR/防病毒并开启策略强制（实时监控、自动隔离可疑进程）。
  - 启用开机固件安全（UEFI Secure Boot）并保持 BIOS/固件更新策略。

3) 补丁与配置管理
  - 所有终端与服务器遵循补丁窗口与分阶段发布（测试 -> Pilot -> 全量）。
  - 紧急补丁（高危漏洞）按应急流程在 72 小时内评估并尽快部署。
  - 使用配置基线（CIS/公司自定义）并周期性扫描合规性。

4) 网络与远程访问
  - 采用 Zero Trust 思想：对内网不默认信任，细分网络（VLAN/微分段）。
  - VPN/远程桌面必须走公司受控客户端并启用 MFA 与客户端健康检查（补丁、防护在线）。
  - 对关键端口与服务实施最小暴露与严格 ACL。

5) 数据保护与加密
  - 机密数据分类分级（公开/内部/机密/受限），不同级别实施不同保护措施。
  - 静态数据与传输数据均需加密（TLS 1.2+、磁盘加密）。
  - 采用 DLP 策略阻断敏感数据泄露到外部邮箱/外设/云存储（视业务需求分级启用）。

6) 备份与恢复
  - 关键系统（域控、关键文件共享、关键数据库）必须具备定期备份与恢复演练记录。
  - 备份采用异地或隔离存储，备份数据加密并做定期恢复演练。

7) 日志、监控与 SIEM
  - 终端、DC、VPN、关键应用与安全设备日志集中到 SIEM/日志平台（至少保留 1 年或合规要求）。
  - 制定关键告警规则（多次失败登录、异常提权、可疑持久化行为）并联动响应流程。

8) 漏洞与配置扫描
  - 定期进行漏洞扫描与第三方组件检测，发现高危漏洞需制定修复 SLA。
  - 对外服务/远程访问入口做渗透测试与配置审计（每年至少一次）。

9) 应急响应与事件管理
  - 建立 IR 流程与联络矩阵（含安全团队、运维、法务、通讯），定义分级（P1/P2/P3）与响应时限。
  - 关键事件（域控受侵、横向渗透、数据泄露）触发紧急 RACI 与保全取证流程（封存日志、快照）。

10) 第三方与供应商安全
  - 第三方接入必须签署安全条款并通过安全评估（供应商分级管理）。
  - 对外托管/云服务需契合公司加密、备份与访问控制要求。

11) BYOD 与移动设备
  - BYOD 需先注册并安装 MDM/管理代理，应用受限与数据容器化（企业应用与个人数据隔离）。
  - 严格的入网与数据访问策略，敏感数据不得下载到未受管设备。

12) 培训与安全意识
  - 定期对全员开展安全意识培训（钓鱼测试、密码与社交工程防范）。
  - 对运维与管理员开展专项安全培训（Least Privilege、应急恢复步骤、日志分析）。

政策执行、例外与审计
- 所有策略应形成书面文档并由信息安全负责人审批；任何例外需提交例外请求（含风险评估、期限与缓解措施）并定期复审。
- 定期（季度/年度）开展合规检查与第三方审计，保留审计结果与整改记录。

关键指标（KPI/OKR）
- 端点补丁合规率（目标 >= 95%）
- 高危漏洞修复平均时长（MTTR）与 SLA 达成率
- MFA 覆盖率与受保护账户比例
- 恶意软件检测率与平均响应时长
- 关键系统恢复时间（RTO）与恢复点（RPO）达成情况

例：安全例外请求模板（简短）
- 标题：
- 申请人/部门：
- 受影响系统/资产：
- 例外原因与业务影响：
- 风险缓解措施：
- 有效期（起止）：
- 批准人/审批时间：

例：安全事件上报（简短）
- 标题：Security Incident - {简短描述} - {主机/系统/时间}
- 影响范围：用户/系统/数据范围
- 发现时间与报告人：
- 初步影响评估：
- 已采取隔离/缓解措施：
- 下一步建议与联系人

附注
- 本策略为公司安全框架核心，需与桌面运维、AD 管理、网络、合规与业务团队联合执行；建议结合行业标准（ISO27001、NIST CSF、CIS）细化并形成可执行 SOP/Runbook。


 ### 7）Office 故障 Runbook（Word / Excel / PowerPoint / Outlook / Office 激活等）

概述
- 适用范围：Microsoft Office 桌面套件（Office 365 / Microsoft 365 / Office 2016/2019/2021）、Outlook 邮件客户端与相关组件（PST/OST/Exchange/Cached Mode）。
- 目标：快速恢复办公应用与邮件收发能力，防止数据丢失，并在必要时将问题升级到 L2/L3 或厂商支持。

现场快速处置（尽量在 5–15 分钟内完成）
1. 记录现象：哪个应用、具体错误提示（错误代码/字样）、发生时间、是否为单用户或多用户、是否突然发生或刚更新后开始。
2. 复现与最小影响测试：能否在安全模式下启动应用（见下）？是否仅在特定文档/邮件触发？
3. 立即备份：若存在本地数据（PST、重要文档），先拷贝到安全位置（OneDrive/网络盘/USB）以避免进一步损坏。

常见场景与排查步骤

A. Office 程序无法启动 / 崩溃 / 打不开文档
- 先做（3–10 分钟）：
  1. 启动安全模式：按住 Ctrl 双击应用快捷方式，或在运行中执行：winword /safe、excel /safe、outlook /safe。若安全模式可用，怀疑加载项或插件。
  2. 修复 Office：控制面板 -> 程序和功能 -> 选择 Office -> 更改 -> 先尝试 Quick Repair，必要时执行 Online Repair（会短暂影响）。
  3. 禁用加载项：应用 -> 文件 -> 选项 -> 加载项 -> 管理 COM 加载项 -> 转到 -> 取消可疑加载项。
  4. 检查更新：Office 更新/Windows 更新是否刚刚安装导致问题；尝试回滚最近更新或等待修复补丁。
- 上报条件：Online Repair 无效、多个用户受影响（同一版本/补丁）、涉及第三方付费插件或数据损坏。

B. Outlook 无法发送/接收邮件 / 同步问题
- 先做（5–15 分钟）：
  1. 检查网络、Exchange/VPN 可达性（ping/mail server、Outlook Web Access 是否可用）。
  2. 检查账户设置：文件 -> 帐户设置 -> 检查账户配置、是否处于 Cached Exchange Mode（可尝试临时关闭）。
  3. 测试自动配置：在系统托盘右键 Outlook 图标（按 Ctrl 并右击图标）选择 “Test E-mail AutoConfiguration...” 验证 Autodiscover/SMTP/IMAP/Exchange。
  4. 清理或重建 OST：退出 Outlook，定位 OST（%localappdata%\Microsoft\Outlook\），将 .ost 重命名为 .ost.old，重启 Outlook 让其重建（注意：Cached 模式下会重新同步）。
  5. 启用日志：File → Options → Advanced → Enable troubleshooting logging（或注册表/命令行启用），收集 Outlook 发送/接收日志。
  6. 使用 Microsoft Support and Recovery Assistant (SaRA) 进行自动诊断。
- 上报条件：邮箱无法登录/大量队列堆积、PST/OST 损坏导致数据丢失、服务器端问题或多用户受影响。

C. 激活 / 许可证问题（Office 显示未激活或功能受限）
- 排查（5–20 分钟）：
  1. 检查 Office 账户：File -> Account，查看激活状态与登录账户是否正确。
  2. 检查许可证命令：以管理员运行命令行并执行（路径视版本）：
     - cscript "C:\Program Files\Microsoft Office\Office16\ospp.vbs" /dstatus
     - 或在 Click-to-Run 环境使用 OfficeC2RClient 或查看 "账号" 页面。
  3. 确认网络与公司许可服务器（若使用 KMS）或 Microsoft 365 服务可达。
  4. 如果为公司批量许可（KMS/MAK/仅读取租户），联系许可管理员核对订阅/许可证分配。
- 上报条件：无法激活导致业务中断、批量用户受影响或许可证系统异常。

D. Outlook 配置损坏 / “无法打开 Outlook 窗口”
- 处理（5–20 分钟）：
  1. 创建新 Outlook 配置文件：控制面板 -> Mail -> Show Profiles -> Add，按向导创建并测试。
  2. 若新配置正常，迁移旧配置中的数据或导出关键设置；若含有本地 PST，先导出备份。
  3. 若问题与 Exchange Server 相关，检查服务器端 mailbox 状态并协调 Exchange 团队。

E. PST/OST 损坏与数据恢复
- 步骤：
  1. 尝试使用 Inbox Repair Tool (scanpst.exe) 修复本地 PST（路径随 Office 版本）。
  2. 对于 OST，建议重建（见上）；对于重要损坏的 PST，先拷贝原始文件再修复。
  3. 若修复失败且数据重要，升级到 L2 并考虑专业数据恢复服务。
- 上报条件：scanpst 无法修复、大量邮件丢失或涉及合规/取证需求（保全原始文件）。

F. 协同 / 联机文档问题（OneDrive / SharePoint / Office 同步）
- 检查（5–20 分钟）：
  1. 在 OneDrive 客户端检查同步状态与冲突文件。
  2. 若 Office 文档无法保存或提示权限问题，检查 SharePoint 权限及网络路径。
  3. 尝试在 Office Web（Office Online）打开以判断是否为客户端问题。
- 上报条件：多人协作数据丢失、SharePoint 后端故障或同步服务中断。

必收集的日志与信息（上报/分析必备）
- Office 版本与 Build（File → Account → About）。
- 应用错误截图/错误代码与时间戳。
- Outlook: Enable logging 后的日志（%temp%\Outlook Logging\ 或 %localappdata%\Microsoft\Outlook\）
- Windows 事件查看器 Application 日志（与 Office/Outlook 时间点匹配的 Error/Warning）。
- scanpst 输出、PST/OST 文件路径与大小、是否已备份的副本。
- ospp.vbs /dstatus 输出（激活问题）。
- SaRA 诊断结果（若运行过）。
- 对于网络/Exchange 问题：Autodiscover 测试输出、Exchange 服务器端错误、SMTP 路由/队列截图。

常用命令与工具
- 启动安全模式：winword /safe、excel /safe、outlook /safe
- Office 修复：通过“程序和功能”→ 更改 → Quick/Online Repair
- 激活状态检查：cscript ospp.vbs /dstatus （Office 32/64 路径需对应）
- Outlook 自动配置测试：Ctrl + 右击系统托盘 Outlook 图标 → Test E-mail AutoConfiguration
- 重建 OST：关闭 Outlook -> 重命名 OST -> 重启 Outlook
- 修复 PST：scanpst.exe <path to .pst>
- Microsoft Support and Recovery Assistant（SaRA）工具
- Office 卸载清理工具（Office Uninstall Support Tool）用于彻底卸载后重装（谨慎使用）

常见错误与快速建议（示例）
- “Cannot start Microsoft Outlook. Cannot open the Outlook window.” → 重建配置文件或启动 Outlook /safe
- “We couldn't sign you in” / 激活对话框反复弹出 → 检查账户/许可证与 cscript ospp.vbs 输出
- 文档保存失败提示权限 → 检查 OneDrive/SharePoint 权限、文件占用情况
- Outlook 持续提示输入密码 → 检查凭据管理器中存储的凭据、MFA 策略、Autodiscover 返回值

升级与上报条件（升级到 L2 / Exchange / 安全团队 / 微软）
- 关键数据丢失或无法恢复（PST 丢失、多人邮件不可达）
- 大量用户或整个部门受影响（疑为服务器/服务端故障）
- 激活/许可证系统异常影响批量用户
- 出现与恶意软件或凭证泄露相关的可疑行为
- 在本地重装/修复无效且需厂商深入分析时

上报时必须提供的信息（模板字段）
- 标题：Office Issue - {应用/问题简述} - {主机名/用户} - {日期时间}
- ITSM 工单号：
- 发生时间与持续性：
- 影响范围（单机/单用户/多人/全局）：
- Office 版本与 Build：
- 主要错误信息与截图（含错误代码）：
- 已尝试步骤（Quick Repair/Online Repair/安全模式/重建 OST/scanpst）与结果：
- 必附文件/日志：事件查看器截图、Outlook 日志、scanpst 输出、PST/OST（若允许）或其哈希、ospp.vbs 输出、SaRA 报告
- 建议优先级与下一步（如需重建邮箱、恢复 PST、联系 Exchange 管理员、联系微软支持）

备注与建议
- 在执行 Online Repair、完全卸载或重建配置文件前，务必先备份本地 PST/重要文档与配置文件。
- 对于涉及合规或取证的邮件丢失，保全原始 PST 与日志并及时上报法务/合规团队。
- 若 Office 问题与补丁更新相关，记录补丁/更新编号并在内网发布回退或修复建议；必要时采用 Pilot 测试策略避免大规模影响。

---

## 何时上报（Escalate）
- 操作后仍未解决、涉及硬件故障、需管理员权限、账户锁定、多用户受影响或涉及敏感数据时上报。  
- 上报需提供的必填信息见下。

---

## 上报时必须提供的信息（工单模板）
1. 报修人姓名 / 联系方式 / 上报时间  
2. 设备：电脑名 / 资产编号 / 操作系统 / IP（ipconfig /all）  
3. 故障现象与发生时间、复现步骤  
4. 已尝试步骤与结果（附命令输出或截图）  
5. 附件：错误截图 / ping 输出 / 任务管理器截图等  
6. 是否影响多人（是/否）  

示例工单标题：  
【工单】无法上网 — 电脑名：PC-123 — 时间：2026-07-13 09:12

---

## 故障记录模板（复制到工单系统）
- 报修人：XXX（手机/邮箱）  
- 设备：PC-XXX / 笔记本 / 打印机 型号  
- 故障描述：无法上网（详细表现）  
- 发生时间：_____  
- 已尝试步骤：重启路由/电脑；ping 8.8.8.8（失败）；ipconfig /all（见附件）  
- 附件：ipconfig.txt / ping 截图 / 错误截图  
- 建议等级：高 / 中 / 低

---
