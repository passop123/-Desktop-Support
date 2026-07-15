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
