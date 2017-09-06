---
author: Justinha
ms.assetid: bfafe378-7ac3-453e-99ca-e5fde83dee2f
MSHAttr: PreferredLib:/library/windows/hardware
title: "UEFI 的 BCD 系统存储设置"
ms.openlocfilehash: 36f32e696b52e56a2d112af7b8cc2aa2368e6fa1
ms.sourcegitcommit: d33e870dc4850bf0ea47fdae0d163b04c1c90f15
translationtype: MT
---
对于典型的部署方案中，您不需要修改 BCD 存储。 本主题讨论中可以修改 BCD 存储的各种 BCD 设置。 在 UEFI 系统，这包括以下引导应用程序设置︰

1.  [Windows 启动管理器](#windowsbootmanager)

2.  [Windows 启动加载程序](#windowsbootloader)

3.  [Windows 内存测试器](#windowsmemorytester)

<!-- more -->

以下各节描述了这些引导应用程序详细信息以及如何修改 UEFI 系统为每个应用程序中的每个可用的设置。

为简单起见，本部分中的 BCDEdit 示例修改 BCD 系统存储区。 若要修改另一个存储，如 BCD 模板的副本包括在命令行中存储区名称。

## <a name="span-idwindowsbootmanagerspanspan-idwindowsbootmanagerspanspan-idwindowsbootmanagerspanwindows-boot-manager-settings-for-uefi"></a><span id="WindowsBootManager"></span><span id="windowsbootmanager"></span><span id="WINDOWSBOOTMANAGER"></span>（uefi） 的 Windows 启动管理器设置


Windows 引导管理器 (`{bootmgr}`) 管理引导过程。 基于 UEFI 的系统包含固件启动管理器，Bootmgfw.efi，用于加载的 EFI 应用程序基于存储在 NVRAM 中的变量。

BCD 设置为`device`，`path`元素在 Windows 启动管理器指示固件启动管理器。 为 Windows 命名 BCD 模板的模板包含以下设置 Windows 启动管理器。

``` syntax
## Windows Boot Manager

identifier              {bootmgr}
device                  partition=\Device\HarddiskVolume1
path                    \EFI\Microsoft\Boot\bootmgfw.efi
description             Windows Boot Manager
```

### <a name="span-iddevicesettingspanspan-iddevicesettingspanspan-iddevicesettingspandevice-setting"></a><span id="Device_Setting"></span><span id="device_setting"></span><span id="DEVICE_SETTING"></span>设备设置

`device`元素指定包含 Windows 启动管理器的卷。 对于 UEFI 系统，`device`元素的 Windows 启动管理器设置为系统分区的卷号。 要确定正确的卷号，请使用 Diskpart 工具以查看磁盘分区。 下面的示例假定系统有一个具有多个分区，其中包括已分配驱动器盘符的 S 系统分区的硬盘

以下的 Diskpart 命令选择磁盘 0，然后列出在该磁盘上，包括其驱动器号的卷的详细信息。 它显示为系统分区的卷 2。

``` syntax
DISKPART> select disk 0
DISKPART> list volume

  Volume ###  Ltr  Label   Fs     Type        Size     Status     Info
  ----------  ---  ------  -----  ----------  -------  ---------  ------
  Volume 0     D           NTFS   Partition    103 GB  Healthy
  Volume 1     C           NTFS   Partition     49 GB  Healthy    Boot
  Volume 2     S           FAT32  Partition    200 MB  Healthy    System
```

如果系统分区没有指派的驱动器号，指定一个使用**Diskpart assign**命令。 下面的示例假定系统分区是卷 2，并将其分配驱动器号为 S。

``` syntax
Diskpart
select disk 0
list volume
select volume 2   // assuming volume 2 is the system partition
assign letter=s
```

在确定系统分区卷之后，可以设置`device`元素的 Windows 启动管理器中为相应的驱动器号。 下面的示例设置`device`驱动器 s。

``` syntax
Bcdedit /set {bootmgr} device partition=s:// system partition
```

### <a name="span-idpathsettingspanspan-idpathsettingspanspan-idpathsettingspanpath-setting"></a><span id="Path_Setting"></span><span id="path_setting"></span><span id="PATH_SETTING"></span>设置路径

`path`元素指定 Windows 启动管理器应用程序的位置，在该卷上。 对于 UEFI 系统，`path`表示固件启动管理器，其路径是\\EFI\\Microsoft\\启动\\Bootmgfw.efi。

您可以确认 BCD 模板具有正确的路径通过枚举存储区中的值，如下所示︰

``` syntax
bcdedit /store bcd-template /enum all
```

若要显式设置`path`到\\EFI\\Microsoft\\启动\\Bootmgfw.efi，使用下面的命令。

``` syntax
Bcdedit /set {bootmgr} path \efi\microsoft\boot\bootmgfw.efi
```

### <a name="span-idothersettingsspanspan-idothersettingsspanspan-idothersettingsspanother-settings"></a><span id="Other_Settings"></span><span id="other_settings"></span><span id="OTHER_SETTINGS"></span>其他设置

下面的示例中所示，您应该设置 Windows 启动管理器是在 UEFI 固件的显示顺序中的第一个项目。

``` syntax
Bcdedit /set {fwbootmgr} displayorder {bootmgr} /addfirst
```

在 Windows 启动管理器显示的顺序，则还应指定顶层的 Windows 启动加载程序应用程序。 下面的示例演示如何将放在顶部的显示顺序指定的 Windows 引导加载程序。

``` syntax
Bcdedit /set {bootmgr} displayorder {<GUID>} /addfirst
```

在前面的示例中， &lt;GUID&gt;是指定 Windows 启动加载程序对象的标识符。 下一节讨论中更详细地介绍此标识符。

**请注意**  
具有多个已安装的操作系统的多重引导系统具有 Windows 引导加载程序的多个实例。 Windows 引导加载程序的每个实例都有其自己的标识符。 您可以设置默认 Windows 引导加载程序 (`{default}`) 对任何这些标识符。

 

## <a name="span-idwindowsbootloaderspanspan-idwindowsbootloaderspanspan-idwindowsbootloaderspanwindows-boot-loader-settings"></a><span id="WindowsBootLoader"></span><span id="windowsbootloader"></span><span id="WINDOWSBOOTLOADER"></span>Windows 启动加载程序设置


BCD 存储有至少一个实例，并可以选择的多个实例，Windows 引导加载程序。 一个单独的 BCD 对象表示每个实例。 每个实例加载具有指定对象的元素的配置的 Windows 的已安装版本之一。 每个 Windows 启动加载程序对象都有其自己的标识符，并且该对象的`device`，`path`设置指示正确的分区和启动应用程序。

`BCD-template`对于 Windows 包含具有以下设置的单个 Windows 启动加载程序对象。

``` syntax
## Windows Boot Loader

identifier              {9f25ee7a-e7b7-11db-94b5-f7e662935912}
device                  partition=C:
path                    \Windows\system32\winload.efi
description             Microsoft Windows Server
locale                  en-US
inherit                 {bootloadersettings}
osdevice                partition=C:
systemroot              \Windows
```

此 Windows 引导装载器的标识符为 {9f25ee7a-e7b7-11db-94b5-f7e662935912}。 您可以在您的系统上使用此 GUID 或让 BCDEdit 工具为您生成新的 GUID。

为了简化 BCDEdit 命令，您可以指定一个 BCD 系统中的加载程序将存储为默认值加载程序在 Windows 启动。 然后，可以使用标准标识符 (`{default}`) 代替完整的 GUID。下面的示例指定为默认引导加载程序，假设它使用的标识符 GUID 从 BCD 模板 EFI Windows 引导加载程序。

``` syntax
Bcdedit /default {9f25ee7a-e7b7-11db-94b5-f7e662935912}
```

### <a name="span-iddeviceandosdevicesettingsspanspan-iddeviceandosdevicesettingsspanspan-iddeviceandosdevicesettingsspandevice-and-osdevice-settings"></a><span id="Device_and_OSDevice_Settings"></span><span id="device_and_osdevice_settings"></span><span id="DEVICE_AND_OSDEVICE_SETTINGS"></span>设备和 OSDevice 设置

下列元素指定密钥的位置︰

`device`元素指定包含启动应用程序的分区。

`osdevice`元素指定包含系统根分区。

对于 Windows 的 EFI 引导装载器，通常，这两个元素被设置为 Windows 系统分区的驱动器号。 但是，如果 BitLocker 启用或计算机有多个安装版本的 Windows 中， `osdevice` ，`device`可能设置为不同的分区。BCD 模板将这两个元素设置为驱动器 c，C 是典型值。 您可以显式设置`osdevice`和`device`的值，如下面的示例中所示。 该示例还假定为 EFI 作为默认的引导加载程序对象指定了 Windows 引导加载程序。

``` syntax
Bcdedit /set {default} device partition=c:
Bcdedit /set {default} osdevice partition=c:
```

### <a name="span-idpathsettingspanspan-idpathsettingspanspan-idpathsettingspanpath-setting"></a><span id="Path_Setting"></span><span id="path_setting"></span><span id="PATH_SETTING"></span>设置路径

`path` Windows 启动加载程序中的元素指定该卷上的引导加载程序的位置。 对于 UEFI 系统， `path` efi，其路径是指示 Windows 引导加载程序\\Windows\\System32\\Winload.efi。

您可以确认 BCD 模板具有正确`path`通过枚举存储区中的值的值。 您可以显式设置`path`值，如下面的示例中所示。

``` syntax
Bcdedit /set {default} path \windows\system32\winload.efi
```

## <a name="span-idwindowsmemorytesterspanspan-idwindowsmemorytesterspanspan-idwindowsmemorytesterspanwindows-memory-tester-settings"></a><span id="WindowsMemoryTester"></span><span id="windowsmemorytester"></span><span id="WINDOWSMEMORYTESTER"></span>Windows 内存测试设置


Windows 内存测试器 (`{memdiag}`) 在启动时运行内存诊断。 BCD 设置为应用程序的`device`，`path`元素指示正确的应用程序。

**请注意**  
注意︰ 英特尔 Itanium 计算机不包含 Windows 内存测试器，不需要`{memdiag}`设置。

 

Windows 的 BCD 模板具有以下设置。

``` syntax
## Windows Memory Tester

identifier              {memdiag}
device                  partition=\Device\HarddiskVolume1
path                    \boot\memtest.exe
description             Windows Memory Diagnostic
```

### <a name="span-iddevicesettingspanspan-iddevicesettingspanspan-iddevicesettingspandevice-setting"></a><span id="Device_Setting"></span><span id="device_setting"></span><span id="DEVICE_SETTING"></span>设备设置

对于 UEFI 系统， `device` Windows 内存测试的元素设置为系统分区的驱动器号。 下面的示例假定系统分区驱动器 S，在前面的示例中使用。

``` syntax
Bcdedit /set {bootmgr} device partition=s:  // system partition
```

### <a name="span-idpathsettingspanspan-idpathsettingspanspan-idpathsettingspanpath-setting"></a><span id="Path_Setting"></span><span id="path_setting"></span><span id="PATH_SETTING"></span>设置路径

`path`元素指定卷上的位置的 Windows 测试管理器中的`device`指定元素。 对于 UEFI 系统，`path`指示 EFI 版本的应用程序 (\\EFI\\Microsoft\\启动\\Memtest.efi)。

您可以确认 BCD 模板具有正确`path`通过枚举存储区中的值的值。 您还可以使用 BCDEdit 工具显式设置`path`值，如下面的示例中所示。

``` syntax
Bcdedit /set {memdiag} path \efi\microsoft\boot\memtest.efi
```

 

 





