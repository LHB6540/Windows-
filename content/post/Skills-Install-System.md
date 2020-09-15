---
title: "Windows系统安装"
date: 2020-09-14T17:46:56+08:00
lastmod: 2020-09-14T17:46:56+08:00
draft: false
tags: ["系统安装", "Legacy/UEFI", "MBR/GPT"]
categories: ["实用技能"]
author: "LHB6540"
contentCopyright: '<a rel="license noopener" href="https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License" target="_blank">Creative Commons Attribution-ShareAlike License</a>'
---

<!--上述为文章属性，请修改内容为“***”的部分、时间及版权声明
title 文章标题
tags标签，可填写一个或多个标签
categories分类根据实际内容请填写“计算机基础知识”、“实用技能”、“故障解决”中的一个
author作者仅填写第一作者，贡献者请在文末以Logo及链接的形式按照示例代码注明
推荐使用typora编辑器
-->

<!--文章内容最大标题等级为二级标题-->



## 前提知识  
### Legacy/UEFI	

​	在[初识计算机一文](https://www.ipbase.top/post/basicalknowladge-meet-the-computer/)中，我们已经了解了计算机的基本软硬件组成。操作系统作为最底层的软件，对于计算机的正常使用至关重要。在开始安装计算机系统前，我们先大致描述一下正常安装了操作系统的计算机的启动过程。

1. 通电，按下电源开关
2. BIOS开始自检系统硬件
3. 自检通过后，根据BIOS的存储的设置，选择用于启动系统的磁盘或其他存储设备

在上面的描述中，较新的计算机一般使用UEFI代替BIOS引导启动操作系统，这种引导方式可以加快系统启动的速度。通常，将基于BIOS的引导过程称之为Legacy，将基于UEFI的引导方式称之为UEFI(或EFI启动，下文统称为UEFI)。

值得注意的是，UEFI和BIOS处于相同级别，而非从属关系。由于我们常常在BIOS设置界面中配置引导方式，可能会导致误以为UEFI是BIOS的一种功能的错误认识。还有一种情况是，所谓的BIOS设置界面，其实是UEFI的配置页面。如何区分是处于BIOS配置界面还是UEFI配置界面，其实从外观就可以作出快速区分。由于两者采用了不同的语言编写，所带来的限制导致BIOS界面简陋，而UEFI界面则可以十分精美华丽，并且支持鼠标拖动点击等操作。下图左侧为常见的BIOS界面，右侧为UEFI配置界面

![](https://qumgog.bn.files.1drv.com/y4m76HEKzUlKVVcKuNLTdvblDNtelkdx_mFfkrlqFljaR-5dtkgBarq-bHk9Atxh25gIYBS6Fvr-FlFgMy13SBbl4OdeXYuJ3E77MSOvkps0u7qIrxN88LS7Pi-MbqZhJqxK39NJxXzrFCwzT4OEER_urB01mSzZ81XCH1M2MqPhB0Qkax21dfKC7e3RujT5qFmcTs1Ew1vUA1j_m2A-A47pw?width=640&height=225&cropmode=none)

### 磁盘分区表

​	讨论完引导方式，下一步我们需要考虑的是磁盘分区格式。所谓磁盘分区格式，其区分在于磁盘的起始扇区的不同。两种磁盘分区格式分别为MBR和GPT。

MBR分区格式将磁盘的第一个扇区(传统MBR一个扇区的大小定义为为512字节)的前446个字节用于存放主引导记录，后面64个字节用于存放分区表。所谓分区表，即记录磁盘的分区状态，由于64个字节的大小限制，因此最多只能有4个分区即将磁盘分为C、D、E、F四个物理分区，注意此处称为物理分区，因为实际应用中MBR磁盘可能需要更多分区，因此产生了逻辑分区以解决问题，本文不再详述逻辑分区的，更多资料可以参考文末的附录1。

![](https://qumfog.bn.files.1drv.com/y4mk6gsPmE1GoW3IYUqkDBg8w8I6MbvHC-AAvgTXtjxNWMii6Z8RK7vLnObvrOz7m7EyuQSGVM41mKIWRQyWhrxNzuuWrq7IvhugoiazwySSpY-qaPcD1G951sHq1h5VzlWcAvMa1OLulC-l5veCrQA7j5_AY7xhs71APPomHyjR5eG9zhju_jkn8CH8EshVrYo3SaXBS0vSjj_0xt73QC71A?width=724&height=280&cropmode=none)

由于每个物理分区的仅有16个字节，因此MBR分区格式存在以下问题

1. 无法使用2.2TB以上的磁盘容量
2. MBR分区仅有一个区块，损坏后，经常难以恢复
3. MBR主引导区仅有446字节，无法存储较多的启动选项

为了解决上述问题，一种新的分区方案即GPT出现了，GPT磁盘采用GUID分区表，因此有时又称为GUID分区格式磁盘。
由于如今的扇区设计已经不再只有一个只是512字节了，最高的一个扇区可以有4K个字节，为了在某些功能上和MBR兼容，因此我们引入LBA(逻辑区块地址)的概念，每个LBA仍旧是512字节。LBA从0开始编号，LBA0作为MBR的兼容模块，LBA1记录GPT表头信息，LBA2-33记录实际分区信息，同时GPT分区表提供了分区备份，如果从磁盘末尾开始用-1开始编号，那么LBA-1至LBA-33即对LBA1-LBA33的备份。
GPT分区实现了：

1. 对MBR的兼容
2. 对分区信息的备份
3. 支持更大的硬盘容量，理论最大支持8ZB(1ZB=2^30TB)

### 引导方式和磁盘分区表的搭配

​	前面讲述到根据BIOS或UEFI(后文统一简写为BIOS/UEFI)的设置来选择用于启动系统的磁盘，至此BIOS/UEFI的工作已经接近结束。根据BIOS/UEFI的配置的Legacy或UEFI引导方式。如果我们曾经有过安装系统的经验，常常会听到这样的说法：UEFI搭配GPT，Legacy搭配MBR。这样的描述并不准确，听起来似乎Legacy只认识MBR分区格式的磁盘而UEFI只认识GPT分区格式的磁盘。如果这样，GUID分区中的LBA0兼容模块就没有存在的必要了。

​	计算机是否支持某种磁盘自然重要，而这是由BIOS/UEFI包含的驱动所决定的。选择UEFI或者Legacy不能决定是否支持某种磁盘。实际的启动过程如下，先假设现有的BIOS/UEFI均支持两种格式的磁盘：

1. 如果选择了Legacy的引导方式，那么选择了目标磁盘后，将会读取MBR的第一个扇区或GUID的LBA0中的启动引导程序，BIOS/UEFI的工作至此结束，剩下的工作交由启动引导程序启动系统
2. 如果选择了UEFI的引导方式，那么选择目标磁盘后，将会读取磁盘的EFI分区(或者ESP分区)，这是一个FAT或FAT32格式的物理分区，但其有特殊的分区标志位，因此在Windows系统中一般不可见。如果EFI分区中有多个引导项，在配置UEFI启动方式时可能需要进一步指定EFI分区中具体的引导项。

如此看来，似乎有4种组合方式，但实际上，有Windows操作系统本身的限制，我们最常使用的Windows 7和Windows 10不能在GPT磁盘上以Legacy的方式启动。

| **Windows种类**                   | **能否读写GPT磁盘**                  | **能否从GPT磁盘启动**                  |
| --------------------------------- | ------------------------------------ | -------------------------------------- |
| 32位 Windows XP                   | 不能。只能看到一个Protective MBR分区 | 不支持                                 |
| Windows 2000/NT/9x                | 不能。只能看到一个Protective MBR分区 | 不支持                                 |
| Windows Server 2003 SP1及以上版本 | 能                                   | 只有基于Itanium的系统才能从GPT磁盘启动 |
| Windows Vista                     | 能                                   | 只有基于Itanium的系统才能从GPT磁盘启动 |
| Windows Server 2008               | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows 7                         | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows 8/8.1                     | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows 10/Server 2016            | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |

因此只能有UEFI+GPT、UEFI+MBR、Legacy+MBR三种组合搭配。值得注意的时，第二张方式并不推荐，因为我们在使用磁盘工具将磁盘转换为MBR分区格式后，需要手动创建EFI分区，这将会占据一个物理分区，而且手动创建的EFI分区如果不设置标志位，将在Windows中可见，对于用于启动的关键分区，暴露分区增加系统启动失败的额外风险。同时，部分BIOS/UEFI可能不支持这种配置

​	因此推荐的搭配仍旧为Legacy+MBR，UEFI+GPT，下方演示的实际操作也仅演示这两种操作

## 配置BIOS/UEFI

​	由于BIOS/UEFI嵌入在主板中，不同厂商提供的主板进入BIOS/UEFI界面的方式也不尽相同。您可以在搜索引擎搜索自己的笔记本型号或主板型号进入BIOS的方式。进入BIOS/UEFI的大致方法为按下电源开关后，快速点击或者长按键盘上的某个按键。下表仅供参考

![](https://qkmgog.bn.files.1drv.com/y4msKZGOJFU5r9YueM6u3uQ0pBp6mw3qHq8Q9fH-9eI_azIYJli85uczf2qD6cC9TGmB8pPPz7C3NrvNu_fV4D6KG5uHfKvgLxkXwXXdar9aD9oV82ICfCocaRwXp-vPJWZkfcwEOejjR8p_v8ZY7vkwvBNsjAFzLgdk1HnIOJj7ehS4tmPzOLwTfCy9BKssElemDNjMtFosw5OZiwLMKKIvw?width=499&height=328&cropmode=none)

​	进入BIOS/UEFI后，查看BIOS/UEFI支持的引导方式，由于各种版本的BIOS/UEFI配置不尽相同，请自行使用搜索引擎查看操作方法，然后根据自己的实际需要选择方案，并选中目标磁盘作为第一启动项

## 分区规划

​	在进行系统安装前，如果是全新的磁盘，需要进行分区规划以更好地管理磁盘。如果在原先已有系统的磁盘重新安装系统，只需要做好原先系统盘内资料的备份工作；如果新安装的系统决定换一种启动方式，那么还需要转换磁盘分区表格式。请根据实际需要选择下方的步骤，使用到的分区工具为[DiskGenius](https://www.diskgenius.cn/)

### 新磁盘分区规划

新购置的磁盘可能已经有预先规划的分区，可能没有任何分区。但由于我们要进行新磁盘规划，直接在左侧的列表选中目标磁盘然后右键，在弹出的菜单项中选择快速分区即可。弹出的界面如下图所示

![](https://qkmdog.bn.files.1drv.com/y4mXIEwKGevyPAVgfir3pcHzRBgTwgVzz4Qd6QaHJEmcgIyXVs7OzKz597x7Hf4jwbMupXt1ELxYwXGFRI-1AtB4Kk4iVci1VxNKJQzWD5RevOUhstD7w9YmrGrTz_9blWU6lR0Hxyj2U6J_IQOzsp-nvs6BveXl9OcK85hMKxYnvj1ay0RFG0LXQnrjDfyPOyG-UdIxScNdZt5j2FwQWVR1A?width=683&height=460&cropmode=none)

1、根据在BIOS中选择的启动方式选择分区表格式，UEFI搭配GUID，Legacy搭配MBR。

2、根据自己的需要划分分区数量，在右侧具体分区大小建议Windows 7 给予系统分区30GB以上空间，Windows 10 给予50GB以上空间

3、如果选择MBR分区表，请勾选重建MBR

4、如果选择GUID,请勾选创建新ESP分区，默认300MB的ESP分区。建议勾选创建MSR分区，将占用128MB，此为微软保留分区，一般情况下为空

5、右侧扇区对齐倍数，固态硬盘建议选择4096扇区，机械硬盘默认即可

点击确定创建新分区

注意，如果选择了GUID分区表，如果划分超4四个分区，不能直接使用工具将其转换为MBR分区表，需要先合并分区或者删除分局，本文暂不讨论，将在以后的文章讲述。而超过4个分区的MBR分区表，则可以直接转换为GUID分区表，但转换并不会创建EFI分区，需要调整分区大小，使用空出来的空间来手动创建EFI分区，本文暂不讨论，将在以后的文章讲述

### 系统盘个人资料备份

略

### 磁盘分区表转换

请参考[磁盘管理](#)

## 系统安装

常见的系统安装有两种方式，镜像安装和PE安装。镜像安装即使用刻录在光盘、U盘等设备的系统镜像启动计算机，然后直接进入系统安装界面。PE安装，即预安装环境安装，预安装环境可以理解为一个迷你的系统，在各个环境中，我们可以使用工具安装系统镜像，而可以省略上一种方法中刻录镜像的步骤，同时PE盘中可以使用各种实用工具，例如硬盘分区工具、引导修复等等。本文演示以PE安装为例，镜像安装将在以后的文章中介绍

### 镜像下载

PE盘本身只是个微型系统，不包含系统我们要安装的系统的镜像。因此我们需要预先下载要安装的系统镜像。推荐的系统镜像下载网站为

https://msdn.itellyou.cn/

进入网站，在左侧菜单栏选择操作系统，展开后，选择要安装的系统

- Windows 7 选择Windows 7后建议选择此版本
  Windows 7 Ultimate with Service Pack 1 (x64) - DVD (Chinese-Simplified) Ultimate 即包含各种版本的混合版，可以在安装时根据需要选择家庭版、专业版等等，同时它包含服务补丁，由于Windows 7 已经不再被官方维护提供安全更新，因此推荐下载此版本

- Windows 10 建议Windows 10，version 最新的版本号 update(最新日期)，省去安装后花费时间下载安装更新，根据需要选择consumer editions个人零售版本或者business editions商业版本。在安装时，均可选择具体的安装版本是家庭版、专业版等等

### PE盘制作

一般的PE安装媒介为U盘，因此也常常成为PE盘安装

- 准备一个U盘，2GB以上即可，如果希望将系统镜像放在其中，请根据镜像大小准备更大容量的U盘

- 备份U盘资料，制作启动盘将清空U盘所有资料，建立新的分区

- PE盘制作软件推荐下载[微PE](http://www.wepe.com.cn/download.html)，纯净，其他的品牌可能会在安装系统中添加推广软件

  1、在上述链接中下载微PE软件，根据电脑CPU选择32位或者64位版本下载

  2、打开软件，以管理员身份运行

  3、在主界面，点击右下角其他安装方式右侧的安装PE到U盘

  4、如图配置

  5、点击立即安装进U盘

- (可选)将系统镜像放进U盘的文件目录

### 进入PE环境

- 插入PE盘到要安装系统的计算机
- 设置BIOS将PE盘作为第一启动项；如果有此选项，推荐U盘以UEFI方式启动加快启动速度
- PE环境如图所示

  ![](https://9ukyfw.bn.files.1drv.com/y4mju86bXNvujXS1cFTvcwvHpeSk7DPWg4LS9IaCASLeOXTg0f1N9oIoecnIP08fRxcmyhC9TvvHIL-X3_iNgqtKiToyHCYUx7bBOQ8VqzVITcq2O0KttZ8wRyWrE632SnqzWhzaUAK9SxH7yqFZwN_EBMSkfk48u8EVwo1GQ2JC-JowoGam5b5A_DOE0fF-VgpTrCAo6ObeBFQH38Sj3LOcg?width=1363&height=765&cropmode=none)

### 安装系统

- 点击桌面的Windows安装器

![](https://9uk0fw.bn.files.1drv.com/y4mI2qqlY-PvxFqdJJfXqs7ZACwrRvzwBOpTnlAxHC2U9eabxEfqrOVmxJ0r8gmCzrFUHqKripOxZCuSZGXpNvVbQYNLNBHD_sSdQ28fv1c3QzpwvLnIIDNE1sUKL_BYPrGJlsy69cFdgSkMEWEYw-8iD-GOEPSDB25fmO6NbuvkXtjlNmxoMvABpWG27jLMW8VmMLiMwMiJfMFuo-WsqFiGw?width=641&height=510&cropmode=none)

- 最顶端选择Windows Vista/7/8/10/2008/2012项
- 选择Windows 安装文件的位置，点击搜索，选择镜像文件的位置
- 选择引导驱动器的位置，点击搜索，由于某些特殊原因，可能分区显示的名称不是EFI或者ESP，还有可能与启动盘的EFI分区混淆，因此我们可以进入PE中的DiskGenius工具，查看该磁盘ESP分区对应的盘符

![](https://9uk1fw.bn.files.1drv.com/y4mYLZfeMPh85wQ_oGUuLqG6iqZe9jeJya8X7Y3bkrydBjOQQpWaJHk_ga-y3wNO4uj4RDtDTmmi7R1Z0uVOs_fdtYiUHzgXC4GMXWlFzp2e3BmKhA-bwhCLIloeOmW9J8GP39XVFNh6pvuHaOmXBoqq551kR84k5sg0gv7ycp_2M_0brKXDbHQm1ujjApDd3Lrmgu4dlPoi-9pF9f1-Y8dtQ?width=266&height=101&cropmode=none)

​	如图ESP分区对应的盘符微Z，因此点击搜索后，选择Z盘即可

​	如果选择Legacy+MBR，则选择和下方系统安装位置相同的分区即可，如下图

​	![](https://9uk5fw.bn.files.1drv.com/y4mfYw1jaRdpbRm64KCk6vTGqPv9rr0T3wTEir1BUrEH1EWKwhQZu7s0AHdWZYAHNfCtpPNpthHsk3gFXO76Ji9rc11c68c4diIGKRaDvoOJzkyTFu1J95EyQ3zqZPyFPRNbC85Qf3Uiw6FKZElOAvPirZHteSRcR1MFQBLlWLkowh8wY0QSJON-nU7gT4sbza7SWD6dLeqXDrqjNfKiXHx_Q?width=635&height=506&cropmode=none)

- 选择安装系统的位置，此处选择C盘
- 选项中，选择Windows的具体版本
- 确认无误后，点击开始安装，此操作将情况C盘

## 参与本项目

[GitHub](https://github.com/LHB6540/windows-computer-maintenance-guide) 

## 交流学习

广中医电脑义修咨询群：465636714   


## 贡献者

<center class="half">
  <a href="https://github.com/lhb6540">
    <img src="https://9uk2fw.bn.files.1drv.com/y4mOeqWCaEnlNq9gSFz1GshYzEVfJ1pMuySa7rbkCDECRXOGy0-_64bgbkMdk75c-e1qBgXWB3-i9-xr2MvdEqEDmN4d9BvdtgyhGMrGOXQLCqdbkDjCf9h5j8XrZApz-jUkBOJoxiT42Prwh0UB70UPou4nmsuuTVMi2jKgHqE94h6yeEJcZRgO34k9gs0S3nZcgjbI6N1Pm3s8jsnA9fdkQ?width=139&height=139&cropmode=none" width="30"/>
  </a>
</center>


