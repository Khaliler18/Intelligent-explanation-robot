# SLAM A 虚拟机环境

本文件记录**电脑 A 上实际部署的虚拟机环境**（实测值、落地参数、校验值与偏差），与之配套的是 [SLAM 虚拟机环境基线](虚拟机环境基线.md)（两台电脑共用的要求值）。

两者的分工：

- [虚拟机环境基线](虚拟机环境基线.md)：**应该是什么**——统一要求、资源口径、验收标准。
- 本文件：**实际是什么**——虚拟机目录、逐项键值、版本号、文件校验值，以及相对基线的偏差与理由。

服务于 [本地仿真验证 TODO](本地仿真验证TODO.md) 的 A0（环境前置检查）。**本文件同时是电脑 B 的对照依据**——B 机由本环境的镜像部署，实测值应与本文件一致；将来若出现差异，可据此判断是部署偏差还是环境变化。

## 一、部署位置与实例标识

| 项目 | 值 |
| --- | --- |
| 虚拟机目录 | `VMs\Ubuntu22.04-GLIM\` |
| 配置文件 | `VMs\Ubuntu22.04-GLIM\Ubuntu22.04-GLIM.vmx` |
| 显示名称 | `Ubuntu22.04-GLIM` |
| 配置变更前备份 | 原为 `.vmx.bak-before-baseline`；2026-10-13 虚拟机整体删除时一并消失。当前 `.vmx` 是按基线手写重建的版本，无更早备份 |
| 虚拟机平台 | VMware Workstation Pro **17.5.0 build-22583795** |
| 虚拟硬件兼容级别 | Workstation 17.x（`virtualHW.version = "21"`） |
| 系统镜像 | `VMs\ubuntu-22.04.5-desktop-amd64.iso` |
| 系统镜像 SHA-256 | `bfd1cee02bc4f35db939e69b934ba49a39a378797ce9aee20f6e3e3e728fefbf` |
| 系统镜像大小 | 4,762,707,968 字节 |
| 镜像校验状态 | **已实测校验通过**：2026-10-13 对 `VMs\ubuntu-22.04.5-desktop-amd64.iso` 计算 SHA-256，与基线记录的哈希完全一致 |
| 虚拟机 UUID | `uuid.bios` = `56 4d 4d 81 9f 83 91 a5-f8 6f d8 e8 de c0 a8 29`（虚拟机重建后由 VMware 生成） |
| 当前状态 | **Ubuntu 22.04.5 LTS 已安装并成功启动**（2026-10-13 19:18 写入完成）；虚拟磁盘占用 **13.06 GB**；ROS 2、GLIM 等软件尚未安装 |

虚拟机平台版本由 `vmware-vdiskmanager.exe` 文件版本核实，与基线写明的 VMware Workstation Pro 17.5.0 一致。

## 二、实测硬件配置

逐项对应 `.vmx` 中的实际键值，可直接核对。

| 项目 | 基线要求 | 实际值（电脑 A） | `.vmx` 键 |
| --- | --- | --- | --- |
| 固件 | UEFI | UEFI，安全启动关闭 | `firmware = "efi"`、`uefi.secureBoot.enabled = "FALSE"` |
| 客户机类型 | Ubuntu 64 位 | ubuntu-64 | `guestOS = "ubuntu-64"` |
| vCPU | 8 个（1 处理器 × 8 核心） | 8 个（1 × 8） | `numvcpus = "8"`、`cpuid.coresPerSocket = "8"` |
| 内存 | 16 GB | 16384 MB | `memsize = "16384"` |
| 虚拟磁盘 | 100 GB、单一虚拟磁盘、按需增长 | **100 GiB、单一文件、按需增长** | `scsi0:0.fileName = "Ubuntu22.04-GLIM.vmdk"` |
| 磁盘控制器 | 两台一致 | LSI Logic SCSI | `scsi0.virtualDev = "lsilogic"` |
| 显示 3D 加速 | 开启 | 开启 | `mks.enable3d = "TRUE"` |
| 显存 | 按 VMware 可用上限 | **2 GB** | `svga.graphicsMemoryKB = "2097152"` |
| 网卡 1 | NAT，用于下载 | NAT | `ethernet0.connectionType = "nat"` |
| 网卡 2 | 桥接，用于雷达网络 | 桥接 | `ethernet1.connectionType = "bridged"` |
| 网卡型号 | 两台一致 | Intel E1000 | `ethernet0/1.virtualDev = "e1000"` |
| USB 控制器 | 未规定 | EHCI（2.0）+ **XHCI（3.0）** | `usb_xhci.present = "TRUE"` |
| 光驱 | 挂载系统镜像 | 指向 ISO，开机即连接 | `sata0:1.fileName = "..\ubuntu-22.04.5-desktop-amd64.iso"` |
| 时间同步 | 未规定 | 开启 | `tools.syncTime = "TRUE"` |
| vCPU / 内存热插拔 | 未规定 | 关闭 | `vcpu.hotadd = "FALSE"`、`mem.hotadd = "FALSE"` |
| 内存自动缩容 | 未规定 | 关闭 | `MemAllowAutoScaleDown = "FALSE"` |
| 声卡 / 蓝牙 | 未规定 | 关闭 | `sound.present = "FALSE"`、`usb.vbluetooth.startConnected = "FALSE"` |
| 软驱 | 未规定 | 关闭 | `floppy0.present = "FALSE"` |
| 电源操作 | 未规定 | 全部 soft | `powerType.* = "soft"` |

### 虚拟磁盘实测

磁盘类型由 VMDK 稀疏头与内嵌描述符直接解析得到，非图形界面显示值。

| 项目 | 实测值 |
| --- | --- |
| 磁盘类型 | `monolithicSparse`——单一文件、按需增长、**未预分配** |
| 逻辑容量 | 209,715,200 扇区 = **100.00 GiB**（107.37 GB） |
| 实际占用 | 安装系统后 13,694,992,384 字节 = 13.06 GB（精简置备；装系统前为 12.56 MB） |
| 适配器类型 | `lsilogic`，与 `scsi0.virtualDev` 一致 |
| 虚拟硬件版本 | 21 |
| 磁盘 UUID | 待补录（重建磁盘后变化；读取时虚拟机在运行、文件被占用，需关机后重取） |

### 快照链

虚拟机共 **2 层快照**，磁盘为链式结构。统计磁盘占用时必须把各层相加。

```
Ubuntu22.04-GLIM.vmdk            12.76 GB   基础盘（不含任何软件）
  └─ 快照 01-ubuntu-installed
Ubuntu22.04-GLIM-000001.vmdk      8.67 GB   增量盘 1（ROS 2 + GLIM + Livox 驱动）
  └─ 快照 02-ros2-glim-livox
Ubuntu22.04-GLIM-000002.vmdk      0.02 GB   增量盘 2（快照 2 之后的写入）
                        合计     21.45 GB
```

| 快照 | 名称 | 内容 |
| --- | --- | --- |
| 1 | `01-ubuntu-installed` | 纯净 Ubuntu 22.04.5，未安装任何软件 |
| 2 | `02-ros2-glim-livox` | 完整开发环境：ROS 2 Humble + GLIM + Livox SDK2 + livox_ros_driver2 |

两次快照均**未勾选「拍摄虚拟机内存」**。后续实验若需要干净回退点，回滚到快照 2 即可保留完整开发环境。
| 校验方法 | 读取 VMDK 头部（magic `KDMV`、version 1、grainSize 128 扇区、flags `0x3`），按 `descriptorOffset` 定位并读取内嵌描述符 |

对比原磁盘：原盘为 80 GiB 且 `createType` 为 `twoGbMaxExtentSparse`（拆分为 21 个分卷），两项均不符合基线，现已替换。

### 已安装系统

| 项目 | 值 |
| --- | --- |
| 发行版 | Ubuntu 22.04.5 LTS (Jammy Jellyfish)，64 位 |
| 安装完成时间 | 2026-10-13 19:18（磁盘写入至 13,040 MB 后停止增长） |
| 启动模式 | UEFI（安装器分区确认对话框显示 ESP + ext4） |
| 分区布局 | 分区 #1 = ESP，分区 #2 = ext4（占其余全部空间）；无 LVM、无 ZFS、无加密 |
| 虚拟磁盘占用 | 13,694,992,384 字节 = 13.06 GB |
| 客户机上报内核 | `6.8.0-138-generic` |
| 安装介质内核 | `6.8.0-40-generic`（live 会话） |
| 光驱状态 | `sata0:1.startConnected = "FALSE"`，装完后已断开，避免再次从 ISO 引导 |
| 安装写入速率 | 约 20-45 MB/s，全程约 11 分钟 |
| 用户名 | `wang` |
| 主机名 | `Slam-a` |
| 根分区 | `/dev/sda2`，98 GB，已用 12 GB（13%） |
| 交换空间 | `/swapfile`，2 GB，已启用（优先级 -2） |
| 网卡 1（NAT） | `ens33`，DHCP 获得 `192.168.229.132/24`，链路 `UP` |
| 网卡 2（桥接） | `ens34`，仅 IPv6 链路本地地址，**无 IPv4**（当前未接 MID-360，属预期状态） |
| MAC 地址映射 | `ens33` = `00:0c:29:c0:a8:29`（= `ethernet0.generatedAddress`）；`ens34` = `00:0c:29:c0:a8:33`（= `ethernet1.generatedAddress`）。**验证通过**：光驱/网卡在客户机内的命名与 `.vmx` 中两块虚拟网卡一一对应 |

**内核版本差异需注意**：安装介质自带 `6.8.0-40-generic`，安装完成后的系统上报 `6.8.0-138-generic`，说明系统在安装后自动应用了更新。基线要求两台"更新状态"一致——**B 机由本镜像部署，版本天然一致**；但若将来需要重建环境，须对照电脑 A 记录的软件包版本清单固定版本，否则两台会分叉。

## 三、相对基线的偏差与本机增补

以下各项是基线未逐字规定、或本机做了明确取舍的设置。**这些设置随模板镜像下发，电脑 B 天然一致**；列出是为了记录取舍理由——将来若需重建环境，须照此设置。

| 项目 | 基线写法 | 本机取值 | 理由 |
| --- | --- | --- | --- |
| 显存 | 按 VMware 可用上限 | 2 GB | 宿主机 32 GB，客户机 16 GB 内存再叠加 8 GB 显存会触发 VMware 保留内存校验告警；GLIM 查看器与 RViz2 的 OpenGL 渲染用不到 8 GB。两台统一取 2 GB。 |
| USB 控制器 | 未规定 | 增加 XHCI | 两台之间传递 rosbag 与示例数据集体积可达数十 GB，USB 2.0 约 40 MB/s 会成为瓶颈。 |
| 网卡型号 | 相同类型即可 | `e1000` | 装完系统后第一件事就是联网安装 ROS 2 与 GLIM 依赖，`e1000` 在 Ubuntu 22.04 免驱必通。基线验收通过、确认 `vmxnet3` 驱动可用后再考虑两台一起更换。 |
| 内存自动缩容 | 未规定 | 关闭 | A2 要求记录"峰值资源占用"。若允许 VMware 在宿主机内存紧张时静默缩减客户机内存，两次跑同一份数据的资源曲线不可比。 |
| vCPU / 内存热插拔 | 未规定 | 关闭 | 让两台模板的资源配置完全确定，避免快照后热插拔引入差异。 |
| 安全启动 | 未规定 | 显式关闭 | 显式写入以保证模板确定性；本阶段 MID-360 驱动为纯用户态，不需要签名内核模块，但保留后续加装驱动的余地。 |
| 时间同步 | 未规定 | 开启 | 客户机时钟漂移会影响 rosbag 时间戳与日志比对。与 A2 的 `use_sim_time` 不冲突：前者管系统时钟，后者管 ROS 时间源。 |
| 声卡 / 蓝牙 | 未规定 | 关闭 | SLAM 虚拟机不使用，减少无用设备。 |

按基线"允许不同的部署参数有记录"的口径，以下项目属于**允许两台不同但必须登记**的内容：

- 网卡 MAC 地址（`addressType = "generated"`，由 VMware 各自生成）。电脑 A 实测：`ethernet0` = `00:0c:29:c0:a8:29`，`ethernet1` = `00:0c:29:c0:a8:33`
- 虚拟机 UUID：电脑 A 为 `56 4d 4d 81 9f 83 91 a5-f8 6f d8 e8 de c0 a8 29`
- 网卡 2 桥接所映射的宿主机物理网卡
- 宿主机 IP、实例标识
- NAT 网段内由 DHCP 分配的地址（电脑 A 为 `192.168.229.132`）

## 四、待完成事项

- [x] **创建 100 GB 单一虚拟磁盘**。已完成，实测见第二节"虚拟磁盘实测"。
- [x] 在图形界面逐项核对第二节表格，确认 `.vmx` 中的设置被正确加载。已完成，见第七节。
- [x] 安装 Ubuntu 22.04.5 LTS（UEFI 启动）。已完成，语言 English、时区 Asia/Shanghai，见第二节"已安装系统"。
- [x] **在系统内核对三件事**。已完成：`ip a` 获得 NAT 地址 `192.168.229.132/24`；`swapon --show` 显示 `/swapfile` 2 GB 已启用；`df -h /` 显示 `/dev/sda2` 98 GB、已用 13%。详见第二节"已安装系统"。
- [x] **拍摄基线快照 `01-ubuntu-installed`**。已完成，未勾选「拍摄虚拟机内存」，生成 `Ubuntu22.04-GLIM-Snapshot1.vmsn`（0.28 MB）与增量盘 `Ubuntu22.04-GLIM-000001.vmdk`。**注意：快照建立后所有写入都进入增量盘，基础盘不再变化，统计磁盘占用时两者需相加。**
- [x] 安装 ROS 2 Humble、GLIM 与相关依赖。已完成，版本与来源见第八节。
- [x] 验证 GLIM 查看器与 RViz2 可正常启动（3D 加速生效）。已完成，证据见第八节。
- [x] 安装 `livox_ros_driver2` 与 Livox SDK2。**已完成**：Livox SDK2 编译安装至 `/usr/local`，`livox_ros_driver2` 编译于 `~/ws_livox`；动态库解析与 launch 文件解析均已验证。版本、路径与验证记录见第八节。
- [x] 拍摄第二个基线快照 `02-ros2-glim-livox`。**已完成**，快照链见下。
- [x] 生成交付包并记录基线编号。**已完成**：基线编号 **`SLAM-TPL-v1`**，交付包位于 `B环境\`，共 21.48 GB。详见第九节。
- [ ] 电脑 B 完成部署后，核对两台的资源、系统、软件版本和配置文件，登记 B 机的唯一实例标识与网卡映射。
- [ ] 记录完整软件包清单文件（`dpkg -l` 输出已存于虚拟机 `~/pkg-01-after-upgrade.txt`、`~/pkg-02-after-ros2.txt`）与内核版本，作为电脑 B 的固定依据。
- [ ] 关闭虚拟机后导出标准模板镜像，记录基线编号与镜像 SHA-256。

完成上述项后，A0 的验收条件（电脑 A 符合共用基线，可启动建图节点和查看器）方可勾选。

## 五、变更记录

| 日期 | 变更内容 | 说明 |
| --- | --- | --- |
| 2026-10-13 | 建立本文件 | 记录电脑 A 虚拟机环境的实测配置 |
| 2026-10-13 | 改写 `.vmx` 硬件参数 | 原配置（2 vCPU / 4 GB 内存 / BIOS 固件 / 仅 1 张网卡 / 未挂载镜像）改为符合基线；原文件备份为 `.vmx.bak-before-baseline` |
| 2026-10-13 | 旧虚拟磁盘移出 | 80 GiB、21 分卷的旧盘整体移动到 `_old-disk-80G-split\`，**未删除**，作为重建失败时的回退。旧盘为空盘，无数据损失。 |
| 2026-10-13 | 新建 100 GiB 单一虚拟磁盘 | 在图形界面创建 `Ubuntu22.04-GLIM.vmdk`，类型 `monolithicSparse`，未预分配。已解析稀疏头验证容量与类型，见第二节。 |
| 2026-10-13 | 图形界面保存后复核 | VMware 重写 `.vmx` 时仅补写 `scsi0:0.present = "TRUE"`，其余键值全部保留，无被回退项。复核明细见第七节。 |
| 2026-10-13 | 首次 UEFI 引导成功 | 从 ISO 进入 GRUB 2.06 菜单，菜单含 `UEFI Firmware Settings`，确认 UEFI 固件生效；`.nvram` 已生成（260 KB）。 |
| 2026-10-13 | 光驱被 VMware Tools 顶掉 | 触发「安装 VMware Tools」后光驱被换为 `linux.iso` 且「启动时连接」被关闭，EFI 报 `No Media`。详见第六节。 |
| 2026-10-13 | 首次安装失败：未落盘 | 安装器显示 `Please remove the installation medium` 后，虚拟磁盘仍为 13172736 字节、一字节未写入。详见第六节。 |
| 2026-10-13 | 虚拟机被删除后重建 | 虚拟机整体删除（磁盘、`.vmx`、旧盘备份全部未保留）。按基线手写重建 `.vmx`，用 `vmware-vdiskmanager -q` 重建 100 GiB `monolithicSparse` 磁盘。当时盘上无系统，无数据损失。 |
| 2026-10-13 | 宿主机清理 | 关闭 VBS（`EnableVirtualizationBasedSecurity` 1→0）、`SystemGuard` 1→0、卸载「虚拟机平台」（`vmcompute` 服务随之消失）、`bcdedit /set hypervisorlaunchtype off`、关闭侧通道缓解（`ulm.disableMitigations = "TRUE"`）。 |
| 2026-10-13 | **安装成功** | 磁盘从 12.6 MB 增长至 13.06 GB，速率约 20-45 MB/s，11 分钟完成，系统可正常启动。详见第二节"已安装系统"。 |

### 旧磁盘回退说明（已失效）

原先保留的 `_old-disk-80G-split\`（80 GiB、21 分卷的旧盘）已在 2026-10-13 虚拟机整体删除时一并移除，回退路径不再存在。当前虚拟磁盘为 `vmware-vdiskmanager` 重建的 100 GiB 单一文件稀疏盘，系统已装入其中。

## 六、已知问题

- `Ubuntu22.04-GLIM\Ubuntu22.04-GLIM.vmx.lck\` 锁目录在配置完成后仍然存在，其内部 `M09642.lck` 记录持有者为 `vmware.exe`。VMware 图形界面在库中打开虚拟机时会保持此锁，属正常现象；图形界面关闭后应自行清理。**在锁存在期间不要用文本编辑器直接改 `.vmx`**，否则改动会被图形界面回写覆盖。早期还残留过一个 `.vmrest.lck`，现已消失。
- **「安装 VMware Tools」会顶掉光驱，导致无法引导。** 2026-10-13 触发该菜单项后，`.vmx` 中 `sata0:1.fileName` 被改为 `C:\Program Files (x86)\VMware\VMware Workstation\linux.iso`，`sata0:1.startConnected` 被置为 `FALSE`，原镜像路径被记入 `toolsInstall.origFileName`。EFI 随即报 `EFI VMware Virtual SATA CDROM Drive (1.0) ... No Media`，虚拟机无法从 ISO 引导。**本项目的 Ubuntu 22.04 不使用该菜单项**，VMware Tools 应在系统内以 `sudo apt install open-vm-tools open-vm-tools-desktop` 安装。恢复方法：虚拟机设置 → CD/DVD (SATA) → 选「使用 ISO 映像文件」指向 `VMs\ubuntu-22.04.5-desktop-amd64.iso` → **勾选「启动时连接」** → 确定 → 虚拟机 → 电源 → **重置**（此时盘上无系统，不能用「重新启动客户机」）。
- **宿主机虚拟化安全特性使 VMware 降级为 ULM 模式，导致首次安装未落盘。** 2026-10-13 排查发现 `Monitor Mode: ULM`，当时的宿主机状态为：`vmcompute`、`HvHost`、`WslService` 服务运行中，`DeviceGuard\EnableVirtualizationBasedSecurity = 1`（内核隔离 `HypervisorEnforcedCodeIntegrity\Enabled` 已为 `0`，不是它）、侧通道缓解默认开启。后果：Ubuntu 安装器长时间无响应（描述为"卡的要死"），并且一次显示 `Please remove the installation medium` 之后，虚拟磁盘仍为创建时的 13172736 字节，**连分区表都未写入**。注意该提示并非安装成功的专属提示，live 会话自身重启时同样会显示。处理办法见 [虚拟机环境基线](虚拟机环境基线.md) 的「宿主机前提」。
- **处理之后 `Monitor Mode: ULM` 仍然存在，但已不影响使用。** 关闭 VBS、卸载「虚拟机平台」、设置 `hypervisorlaunchtype Off` 之后，`\Hyper-V Hypervisor Logical Processor(*)\Total Run Time` 计数器仍能取到 `hv lp` 实例，说明 hypervisor 依旧加载；`Monitor Mode` 也仍为 `ULM`。但此时安装写入速率已达 20-45 MB/s、11 分钟装完，**性能实际已可接受**。因此判断：造成首次失败的并非 ULM 本身，而是 VBS + Hyper-V 特性 + 侧通道缓解三者叠加的开销。未继续追查的残留嫌疑项为 BCD 中的 `isolatedcontext Yes` 与注册表中的 `RunAsPPL = 2`（LSA 保护），如需进一步提升性能可从此处入手。
- 命令行创建虚拟磁盘（`vmware-vdiskmanager.exe`）在当前环境下会报 `Failed to get a safe directory name for the log`，加 `-q` 参数可绕过，但在中文路径下不够稳妥。**磁盘创建统一在图形界面完成**，命令行方式不作为本项目的标准步骤。
- 当前 PowerShell 会话受沙箱限制，`Get-Process` 只能枚举约 35 个进程，无法据此判断 VMware 是否运行；需要确认时应查看 `.vmx.lck` 目录内的锁文件内容，或使用 `tasklist`（可能返回 Access denied）。

## 七、配置复核记录

2026-10-13，图形界面保存设置后，重新读取 `.vmx` 逐项比对基线，结果如下。全部为**通过**，无待修正项。

| 核对项 | 基线要求 | `.vmx` 实际 | 结果 |
| --- | --- | --- | --- |
| 虚拟硬件兼容级别 | Workstation 17.x | `virtualHW.version = "21"` | 通过 |
| 固件 | UEFI | `firmware = "efi"` | 通过 |
| 安全引导 | 未规定 | `uefi.secureBoot.enabled = "FALSE"` | 通过 |
| 客户机类型 | Ubuntu 64 位 | `guestOS = "ubuntu-64"` | 通过 |
| vCPU | 8（1 × 8） | `numvcpus = "8"`、`cpuid.coresPerSocket = "8"` | 通过 |
| 内存 | 16 GB | `memsize = "16384"` | 通过 |
| 虚拟磁盘 | 100 GB 单一虚拟磁盘 | `monolithicSparse`、100.00 GiB | 通过 |
| 磁盘控制器 | 两台一致 | `scsi0.virtualDev = "lsilogic"` | 通过 |
| 3D 加速 | 开启 | `mks.enable3d = "TRUE"` | 通过 |
| 显存 | 两台统一 | `svga.graphicsMemoryKB = "2097152"`（2 GB） | 通过 |
| 网卡 1 | NAT | `ethernet0.connectionType = "nat"` | 通过 |
| 网卡 2 | 桥接 | `ethernet1.connectionType = "bridged"` | 通过 |
| 光驱 | 挂载系统镜像 | `deviceType = "cdrom-image"`，指向 ISO，`startConnected = "TRUE"` | 通过 |
| 时间同步 | 未规定 | `tools.syncTime = "TRUE"` | 通过 |
| 热插拔 | 未规定 | `vcpu.hotadd`、`mem.hotadd` 均为 `FALSE` | 通过 |
| 内存自动缩容 | 未规定 | `MemAllowAutoScaleDown = "FALSE"` | 通过 |
| USB | 未规定 | `ehci.present`、`usb_xhci.present` 均为 `TRUE` | 通过 |
| 声卡 / 蓝牙 / 软驱 | 未规定 | 均为 `FALSE` | 通过 |
| 电源操作 | 未规定 | `powerType.*` 全部 `soft` | 通过 |
| 系统镜像完整性 | 与基线哈希一致 | SHA-256 实测 `bfd1cee0…fefbf`，与基线记录一致 | 通过 |

图形界面在保存时重写 `.vmx`，只追加了 `scsi0:0.present = "TRUE"` 一行，未回退或改写任何已设键值，说明写入的键名均被 VMware Workstation 17.5.0 正确接受。

## 八、软件环境

记录时间：2026-10-13。电脑 B 部署时以本节为对照，逐项对齐。

### 软件源配置

| 用途 | 地址 | GPG Key 获取方式 |
| --- | --- | --- |
| Ubuntu 主源 | `http://cn.archive.ubuntu.com/ubuntu` | 系统自带 |
| Ubuntu 安全源 | `http://security.ubuntu.com/ubuntu` | 系统自带 |
| ROS 2 | `https://mirrors.tuna.tsinghua.edu.cn/ros2/ubuntu jammy main` | `https://mirrors.tuna.tsinghua.edu.cn/rosdistro/ros.key` |
| GLIM | `https://koide3.github.io/ppa/ubuntu2204 ./` | `https://koide3.github.io/ppa/ubuntu2204/KEY.gpg` |

**两个 key 的处理方式不同，不能混用**：

- ROS 2 的 `ros.key` 是**二进制 keyring**，必须直接 `curl -o /usr/share/keyrings/ros-archive-keyring.gpg`，**不可**经过 `gpg --dearmor`。
- GLIM 的 `KEY.gpg` 是**文本（armored）格式**，**必须** `gpg --dearmor`。

（清华 help 页给出的 ROS 2 写法带 `| gpg --dearmor`，与本项目实际验证不符，以本节为准。）

### 网络环境限制

该机器所在网络**无法直连部分国际源**，实测结果：

| 地址 | 结果 |
| --- | --- |
| `mirrors.tuna.tsinghua.edu.cn` | 可达 |
| `cn.archive.ubuntu.com`、`security.ubuntu.com` | 可达 |
| `koide3.github.io` | 可达（HTTP/2 200） |
| `raw.githubusercontent.com` | **连接被重置**，不可用 |
| `packages.ros.org` | **连接被拒**，不可用 |

因此 ROS 2 **必须走国内镜像**；GLIM 可直接用官方 PPA。电脑 B 若网络环境不同，仍应使用相同地址以保证版本一致。

### 版本清单

| 组件 | 版本 | 来源 |
| --- | --- | --- |
| ROS 2 | Humble，共 **282** 个 `ros-humble-*` 包 | 清华 ros2 镜像 |
| GLIM 本体 | `ros-humble-glim` **1.2.2-0jammy** | koide3 PPA |
| GLIM ROS 接口 | `ros-humble-glim-ros` **1.2.2-0jammy** | koide3 PPA |
| gtsam_points | `libgtsam-points-dev` **1.2.2** | koide3 PPA |
| GTSAM | `libgtsam-notbb-dev` **4.3.0**（传递依赖，实际参与链接） | koide3 PPA |
| iridescence | `libiridescence-dev` **1.0.2** | koide3 PPA |
| GLFW | `libglfw3-dev` **3.3.6-1** | Ubuntu |
| METIS | `libmetis-dev` / `libmetis5` **5.1.0.dfsg-7build2** | Ubuntu |
| Boost | `libboost-all-dev` **1.74.0.3ubuntu7** | Ubuntu |
| 内核 | `6.8.0-138-generic` | 系统更新 |
| Livox SDK2 | 源码编译，commit **`c0796f04c143143899c87a773d9f6b7136453c0b`** | GitHub；安装至 `/usr/local` |
| livox_ros_driver2 | 源码编译，commit **`21445540f0d100dc86a7e6df312dd70bbdb4afdf`** | GitHub；工作空间 `~/ws_livox` |

**两类组件的版本固定方式不同，不能混用**：

- **GLIM 及其依赖**（`ros-humble-glim`、`libgtsam-points-dev` 等）：有 apt 包，**按 apt 版本号精确安装**。
- **Livox SDK2 与 livox_ros_driver2**：无 apt 包，必须源码编译，**按 git commit 号固定**。电脑 B 需 `git clone` 后 `git checkout <同一 commit>` 再编译。

### GitHub 访问的稳定性

`github.com` 可达（`git ls-remote` 成功），但**连接不稳定**：实测同一仓库连续两次执行 `git ls-remote`，第一次成功、第二次报 `gnutls_handshake() failed: Error in the pull function.`。`git clone` 可能需要重试；必要时用 `git clone --depth 1` 减少传输量以提高成功率。

注意这与 `raw.githubusercontent.com`（**完全不可用**，连接被重置）是两回事，GitHub Pages（`koide3.github.io`）也可达。

**如需重建环境，应按版本号精确安装**（PPA 保留历史版本，例如 `sudo apt install -y libgtsam-points-dev=1.2.2`），不要装"当时的最新版"，否则两台会分叉。**B 机由镜像部署，正常情况下无需重新安装。**

### 关键路径

| 项目 | 路径 |
| --- | --- |
| ROS 2 安装目录 | `/opt/ros/humble` |
| **GLIM 配置目录** | **`/opt/ros/humble/share/glim/config`** |
| GLIM ROS 包名 | `glim_ros`（**注意：不是文档中常见的 `glim_ros2`**） |
| 可执行文件 | `glim_rosnode`（SLAM 主节点）、`glim_rosbag`（回放建图）、`offline_viewer`（离线查看器）、`map_editor`、`validator_node` |
| Livox SDK2 安装位置 | 库 `/usr/local/lib/liblivox_lidar_sdk_shared.so`（**注意库名含 `lidar`，不是 `liblivox_sdk_shared.so`**）；头文件 `/usr/local/include/livox_lidar_{api,def,cfg}.h` |
| livox_ros_driver2 工作空间 | `~/ws_livox`，环境 `source ~/ws_livox/install/setup.bash` |
| 驱动可执行文件 | `livox_ros_driver2_node` |
| 驱动 launch 文件 | `~/ws_livox/install/livox_ros_driver2/share/livox_ros_driver2/launch_ROS2/`（共 11 个。**目录名是 `launch_ROS2` 而不是标准的 `launch`，但 `ros2 launch` 仍能直接找到**） |
| MID-360 启动命令 | `ros2 launch livox_ros_driver2 rviz_MID360_launch.py`（官方 README 第 112 行确认格式为 `ros2 launch livox_ros_driver2 [launch file]`，不带路径） |

ROS 2 的可执行文件安装在 `/opt/ros/humble/lib/<包名>/` 下，**不在 `bin/`**，用 `dpkg -L | grep bin/` 查不到。

### 环境验证记录

| 验证项 | 命令 | 结果 |
| --- | --- | --- |
| DDS 通信 | `ros2 topic list` | 正常 |
| RViz2 | `rviz2` | 窗口正常，网格渲染正常，**31 fps** |
| GLIM 查看器 | `ros2 run glim_ros offline_viewer` | 正常弹出交互式查看器；日志 `[viewer] [info] Starting interactive viewer` |
| 3D 加速 | `glxinfo -B` | `OpenGL renderer string: SVGA3D; build: RELEASE; LLVM;`，**非 llvmpipe**，判定硬件加速路径生效；`Max core profile version: 4.3` |
| livox_ros_driver2 构建 | `./build.sh humble` | `Summary: 1 package finished`；仅 1 条 `unused variable 'index'` 警告（**上游代码自身的警告，非错误**） |
| 驱动 launch 文件解析 | `ros2 launch livox_ros_driver2 rviz_MID360_launch.py --show-args` | 返回 `No arguments.`，说明 launch 文件被正确找到并解析 |
| 动态库解析 | `ldd .../\|grep -c "not found"` | `0`；`liblivox_lidar_sdk_shared.so => /usr/local/lib/liblivox_lidar_sdk_shared.so` 正常解析 |

**注意**：`livox_ros_driver2_node` **本身不直接链接** livox 库，链接关系在 `liblivox_ros_driver2.so` 上。因此查节点二进制的 `ldd` 看不到 livox 相关条目，需查驱动库本身。

## 九、交付包与基线编号

| 项目 | 值 |
| --- | --- |
| 基线编号 | **`SLAM-TPL-v1`** |
| 交付包位置 | `B环境\`（工作区根目录下） |
| 交付包总大小 | 21.48 GB |
| 生成时间 | 2026-10-13 |
| 校验方式 | 对源目录与交付目录逐文件计算 SHA-256 并比对，**9 个文件全部一致** |
| 校验值文件 | `B环境\校验值.txt` |
| 部署文档 | `B环境\部署文档.md` |

### 交付包内容

```
B环境\
├── 部署文档.md              面向 B 线的完整部署步骤
├── 校验值.txt               9 个文件的 SHA-256
└── Slam-B\                  虚拟机本体
    ├── Ubuntu22.04-GLIM.vmx / .nvram / .vmxf / .vmsd
    ├── Ubuntu22.04-GLIM.vmdk              12.76 GB  基础盘
    ├── Ubuntu22.04-GLIM-000001.vmdk        8.67 GB  增量盘 1
    ├── Ubuntu22.04-GLIM-000002.vmdk          46 MB  增量盘 2
    └── Ubuntu22.04-GLIM-Snapshot1/2.vmsn            两层快照
```

### 为什么用文件级打包而不是导出 OVA

原计划用 VMware 的 OVF/OVA 导出（VMware → 文件 → 导出为 OVF），这是生成"单一镜像文件 + 单一 SHA-256"的标准做法。实际执行时改用**文件级打包**，原因与新方案的优点如下：

- **`vmrun clone` 在本环境不可用**：`vmrun -T ws list` 报 `vmrun was unable to start`，与 `vmware-vdiskmanager` 早先的「无法确定日志目录」属同一类问题（写 `%APPDATA%\VMware` 受限）。`vmrun` 没有类似 `vdiskmanager -q` 的绕过参数。OVF 导出则必须在图形界面手工操作。
- **文件级打包完整保留两层快照**：`01-ubuntu-installed` 与 `02-ros2-glim-livox` 随包交付，B 机开箱即有退路。OVA 导出会压平快照，只剩一个状态。
- **代价**：交付物是文件夹而非单一文件，没有统一的镜像哈希；改用**逐文件 SHA-256**（`校验值.txt`）替代，传输后逐项核对即可。

### 有意排除的文件

| 文件 | 大小 | 排除原因 |
| --- | --- | --- |
| `Ubuntu22.04-GLIM-Snapshot2.vmem` | 16 GB | 运行态内存文件。它的修改时间与关机时刻一致，属运行期产生的内存映像，对新建的 B 机无意义 |
| `vmware*.log`、`mksSandbox*.log` | 约 1.2 MB | 运行日志 |
| `*.scoreboard` | 约 40 KB | 运行时统计 |
| `ubuntu-22.04.5-desktop-amd64.iso` | 4.5 GB | 系统已装好，不再需要。B 机若需重装，按基线记录的 SHA-256 自备 |

### B 机部署后的必改项

见 `B环境\部署文档.md` 第五、六节。要点：**打开时选「我已复制该虚拟机」**（否则 MAC/UUID 与 A 冲突）、改主机名 `Slam-b`、重新生成 `/etc/machine-id`、配置桥接网卡的物理网卡映射。**用户名 `wang` 保持不变**——`~/ws_livox/install/` 中的 colcon 脚本把绝对路径写死为 `/home/wang/ws_livox/...`。

**3D 加速的上报异常项**：`glxinfo -B` 同时报 `Accelerated: no` 与 `Video memory: 1MB`，但 `.vmx` 中显存配置为 2 GB（`svga.graphicsMemoryKB = "2097152"`）。这是 VMware SVGA 驱动的已知上报怪癖，与实际渲染能力无关——判定依据取 renderer 名称（`SVGA3D` 而非 `llvmpipe`）与实际帧率。

### 待补事项

- `~/.bashrc` 中 `source /opt/ros/humble/setup.bash` 重复出现两次（第 118、119 行）。功能无害（重复 source 幂等），但导出标准模板前应清理为一行。
