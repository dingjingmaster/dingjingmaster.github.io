# 使用qemu进行完整系统仿真


## 介绍

### 虚拟化加速器(Virtualisation Accelerators)

QEMU的系统仿真提供了运行客户操作系统的机器(CPU、内存和仿真设备)的虚拟模型。它支持许多管理程序(称为加速器)以及能够模拟许多cpu的称为Tiny Code Generator (TCG)的JIT。

支持的加速器(即：管理程序)
|加速器(Accelerator)|宿主机(Host OS)|宿主机架构(Host Archiectures)|
|----|----|----|
|KVM|Linux|Arm (64 bit only), MIPS, PPC, RISC-V, s390x, x86|
|Xen|Linux (as dom0)|Arm, x86|
|Intel HAXM (hax)|Linux, Windows|x86|
|Hypervisor Framework (hvf)|MacOS|x86 (64 bit only), Arm (64 bit only)|
|Windows Hypervisor Platform (wphx)|Windows|x86|
|NetBSD Virtual Machine Monitor (nvmm)|NetBSD|x86|
|Tiny Code Generator (tcg)|Linux, other POSIX, Windows, MacOS|Arm, x86, Loongarch64, MIPS, PPC, s390x, Sparc64|

### 功能概述

系统仿真提供了广泛的设备模型来模拟您可能想要添加到计算机中的各种硬件组件。这包括大量的VirtIO设备，这些设备专门针对虚拟化下的高效操作进行了调优。可以使用vhost-user(用于VirtIO)或Multi-process QEMU从主QEMU进程中卸载一些设备仿真。如果平台支持，QEMU还支持直接将设备传递给来宾虚拟机，以消除设备模拟开销。有关详细信息，请参阅设备仿真。

有一个功能齐全的块层，允许构建复杂的存储拓扑结构，可以堆叠在多个层上，支持重定向、网络、快照和迁移支持。

chardev系统使用非常灵活，允许使用stdio、file、unix socket和TCP网络处理来自字符设备的IO。

QEMU提供了许多管理接口，包括一个基于行的Human Monitor Protocol (HMP)，它允许您动态地添加和删除设备以及自省系统状态。QEMU监控协议(QMP)是一个定义良好的、版本化的、机器可用的API，它为其他工具提供了一个丰富的接口，用于创建、控制和管理虚拟机。这是高级工具接口所使用的接口，例如使用libvirt框架的Virt管理器。

对于常见的加速器QEMU，支持使用gdbstub进行调试，它允许用户连接GDB并调试系统软件映像。

### 运行

QEMU提供了丰富而复杂的API，很难理解。虽然有些体系结构可以仅使用磁盘映像启动某些内容，但这些示例使用默认值省略了许多细节，这些默认值对于现代系统来说可能不是最优的。

对于非x86系统，我们模拟了大量的机器类型，命令行更明确地定义了机器和引导行为。您将经常在手册的QEMU系统仿真器目标部分中找到示例命令行。

虽然该项目并不想阻止用户使用命令行启动vm，但我们确实想强调，有许多项目致力于提供更友好的用户体验。围绕libvirt框架构建的虚拟机可以使用特性探测来构建适合在现有硬件上运行的现代虚拟机映像。

QEMU命令行的一般形式可以表示为

大多数选项都会生成一些帮助信息。例如:
```shell
$ qemu-system-x86_64 [machine opts] \
                [cpu opts] \
                [accelerator opts] \
                [device opts] \
                [backend opts] \
                [interface opts] \
                [boot opts]
```

Help也可以作为参数传递给另一个选项，下面命令将列出QEMU二进制文件支持的机器类型。例如:
```shell
$ qemu-system-x86_64 -M help
```

将列出可控制scsi-hd设备行为的附加选项的参数及其默认值。
```shell
$ qemu-system-x86_64 -device scsi-hd,help
```

配置预览：
|配置|说明|
|---|---|
|Machine|定义机器类型，内存数量等|
|CPU|vcpu的类型、数量/拓扑。大多数加速器都提供一个主机cpu选项，它只是通过主机cpu配置，而不会过滤掉任何特性。|
|Accelerator|这取决于您运行的管理程序。请注意，默认值是TCG，它是纯模拟的，因此必须指定加速器类型以利用硬件虚拟化。|
|Devices|默认情况下未使用机器类型定义的其他设备。|
|Backends|Backends 是 QEMU 处理客户数据的方式，例如块设备如何存储、网络设备如何查看网络或串行设备如何指向外部世界。|
|Interfaces|系统如何显示，如何管理和控制或调试。|
|Boot|系统如何引导，通过固件还是直接引导内核。|

在下面的例子中，我们首先定义一个virt machine，它主要是用于运行 aarch64 客户机的通用平台。我们启用虚拟化，这样就可以在模拟的客户机中使用KVM。由于virt machine带有一些内置的pflash设备，我们给它们命名，以便稍后覆盖默认值。

```shell
$ qemu-system-aarch64 \
   -machine type=virt,virtualization=on,pflash0=rom,pflash1=efivars \
   -m 4096 \
```

然后我们使用max选项定义了 4 个 vcpu，这使 QEMU 能够模拟的所有 arm 特性。我们为 arm 的指针认证算法提供了一个更加仿真友好的实现。我们显式地指定了 TCG 加速。

```shell
-cpu max,pauth-impdef=on \
-smp 4 \
-accel tcg \
```

由于virt平台没有任何默认的网络或存储设备，我们需要定义它们。我们给它们id，以便稍后将它们链接到后端。

```shell
-device virtio-net-pci,netdev=unet \
-device virtio-scsi-pci \
-device scsi-hd,drive=hd \
```

我们将用户模式网络连接到网络设备。由于用户模式网络不能从外部直接访问，我们将localhost端口2222转发给客户机上的ssh端口。

```shell
-netdev user,id=unet,hostfwd=tcp::2222-:22 \
```

我们将客户机可见块设备连接到为客户机预留的LVM分区。

```shell
-blockdev driver=raw,node-name=hd,file.driver=host_device,file.filename=/dev/lvm-disk/debian-bullseye-arm64 \
```

然后告诉QEMU将QEMU监视器与串口输出复用(我们可以使用字符后端多路复用器中的key在两者之间切换)。由于没有默认的图形设备，我们禁用了显示，因此我们只能在终端中工作。

```shell
-serial mon:stdio \
-display none \
```

最后，我们覆盖默认固件，以确保有一些存储空间供EFI保存其配置。该固件负责查找磁盘、引导grub并最终运行我们的系统。
```shell
-blockdev node-name=rom,driver=file,filename=(pwd)/pc-bios/edk2-aarch64-code.fd,read-only=true \
-blockdev node-name=efivars,driver=file,filename=$HOME/images/qemu-arm64-efivars
```

## 调用

```shell
qemu-system-x86_64 [options] [disk_image]
```

disk_image是IDE硬盘0的原始硬盘映像。有些目标不需要磁盘映像。

### 标准配置选项(standard options)

#### `-h`

#### `-version`

#### `-machine [type=]name[,prop=value[,...]]`

按名称选择模拟机器。使用-machine列出可用的机器。<br/>

对于旨在支持不同版本之间的动态迁移兼容性的体系结构，每个版本都将引入一种新的版本机器类型。例如，2.8.0版本为x86_64/i686架构引入了“pc-i440fx-2.8”和“pc-q35-2.8”机器类型。<br/>

为了允许来宾从QEMU 2.8.0版本实时迁移到QEMU 2.9.0版本，2.9.0版本也必须支持“pc-i440fx-2.8”和“pc-q35-2.8”机器。为了允许用户热迁移虚拟机在升级时跳过多个中间版本，QEMU的新版本将支持许多以前版本的机器类型。<br/>

支持的机器属性为:
- `accel=accels1[:accels2[:...]]`: 这用于启用加速器。根据目标架构，可以使用kvm、xen、hax、hvf、nvmm、whpx或tcg。缺省情况下，使用tcg。如果指定了多个加速器，则如果前一个加速器初始化失败，则使用下一个加速器。
- `vmport=on\|off\|auto`: 允许模拟VMWare IO端口，对于vmmouse等auto表示根据accel选择值。对于accel=xen，默认是关闭的，否则默认是打开的。
- `dump-guest-core=on\|off`: 在核心转储中包含客户内存。默认是打开的。
- `mem-merge=on\|off`: 启用或禁用内存合并支持。该特性在主机支持的情况下，支持在虚拟机实例之间对相同的内存页进行重复数据删除(默认开启)。
- `aes-key-wrap=on\|off`: 在s390-ccw主机上启用或禁用AES密钥包装支持。此特性控制是否创建AES封装密钥以允许执行AES加密函数。默认是打开的。
- `dea-key-wrap=on\|off`: 在s390-ccw主机上启用或禁用DEA密钥包装支持。该特性控制是否创建DEA包装密钥以允许执行DEA加密函数。默认是打开的。
- `nvdimm=on\|off`: 启用/禁用NVDIMM支持。默认为关闭。
- `memory-encryption=`: 使用的内存加密对象。默认为none。
- `hmat=on\|off`: 启用或禁用ACPI异构内存属性表(HMAT)支持。默认为关闭。
- `memory-backend='id'`: 替代传统的-mem-path和mem-prealloc选项。允许使用内存后端作为主RAM。
- `cxl-fmw.0.targets.0=firsttarget,cxl-fmw.0.targets.1=secondtarget,cxl-fmw.0.size=size[,cxl-fmw.0.interleave-granularity=granularity]`：
定义CXL固定内存窗口(CFMW)。
CXL 2.0 ECN: CEDT CFMWS & QTG _DSM中描述。<br/>它们是系统上的主机物理地址(HPA)区域，可以在一个或多个CXL主机桥上交错。系统软件将把特定的设备分配到这些窗口中，并在根端口、交换机端口和设备中适当地配置下游主机管理的设备内存(HDM)解码器，以满足交织要求，然后再启用内存设备。<br/>目标。X=target提供到CXL主机桥的映射关系，CXL主机桥可以通过-device表项中的id来标识。当固定内存窗口表示交错内存时，需要多个条目来指定所有目标。X是从0开始的目标索引。<br/>size=size设置CFMW的大小。必须是256MiB的倍数。该区域将与256MiB对齐，但位置取决于平台和配置。<br/>交错-粒度=粒度设置交织的粒度。默认256简约。仅支持256KiB、512KiB、1024KiB、2048KiB、4096KiB、8192KiB和16384KiB粒度。
    ```shell
    -machine cxl-fmw.0.targets.0=cxl.0,cxl-fmw.0.targets.1=cxl.1,cxl-fmw.0.size=128G,cxl-fmw.0.interleave-granularity=512k
    ```

#### `sgx-epc.0.memdev=@var{memid},sgx-epc.0.node=@var{numaid}`

定义一个SGX EPC section

- `-cpu model`: 选择CPU型号(-cpu帮助列表和其他功能选择)

#### `-accel name[,prop=value[,...]]`

这用于启用加速器。根据目标架构，可以使用kvm、xen、hax、hvf、nvmm、whpx或tcg。缺省情况下，使用tcg。如果指定了多个加速器，则如果前一个加速器初始化失败，则使用下一个加速器。

- `igd-passthru=on|off`: 当Xen在使用时，该选项控制Intel集成图形设备是否可以传递到客户机(默认=关闭)
- `kernel-irqchip=on|off|split`: 控制KVM内核内的irqchip支持。默认是中断控制器的完全加速。在x86上，分割irqchip减少了内核攻击面，代价是非msi中断的性能损失。除了用于调试之外，不建议完全禁用内核内的irqchip。
- `kvm-shadow-mem=size`: KVM MMU 大小设置
- `split-wx=on|off`：控制对TCG代码生成缓冲区使用分割w^x映射。一些操作系统要求启用此功能，在这种情况下，此功能将默认开启。在其他操作系统上，这将默认关闭，但可以在测试或调试时启用此功能。
- `tb-size=n`: 控制TCG转换块缓存的大小(以MiB为单位)。
- `thread=single|multi`: 控制TCG线程数。当TCG是多线程时，每个vCPU将有一个线程，因此可以利用额外的主机内核。默认情况下，后端和前端都支持多线程，并且没有启用不兼容的TCG特性(例如icount/replay)。
- `dirty-ring-size=n`: 使用KVM加速器时，它控制每个vCPU脏页环缓冲区的大小(每个vCPU的条目数)。它的值应该是2的幂，并且应该是1024或更大(但仍然小于内核支持的最大值)。4096可能是一个很好的初始值，如果您不知道哪个是最好的。将该值设置为0以禁用该特性。默认情况下，该特性是禁用的(dirty-ring-size=0)。启用后，KVM将在位图中记录脏页面。
- `notify-vmexit=run|internal-error|disable,notify-window=n`: 在x86主机上启用或禁用通知虚拟机退出功能，并指定相应的通知窗口来触发虚拟机退出。Run选项启用该特性。它什么都不做，如果发生退出则继续。内部错误选项启用该特性。它会引发一个内部错误。“禁用”选项不会启用该功能。此功能可以缓解由于事件窗口在指定时间内未打开(即通知窗口)而导致的CPU卡死问题。默认值:notify-vmexit =运行,notify-window = 0。

#### `-smp [[cpus=]n][,maxcpus=maxcpus][,sockets=sockets][,dies=dies][,clusters=clusters][,cores=cores][,threads=threads]`

模拟一个SMP系统，初始有n个cpu在机器类型板上。在支持CPU热插拔的单板上，可以设置可选的“maxcpus”参数，以便在运行时添加更多的CPU。当省略这两个参数时，将从提供的拓扑成员计算最大CPU数量，并且初始CPU计数将与最大数量匹配。当只给出其中一个时，省略的那个将被设置为对应的值。这两个参数都可以指定，但最大CPU数必须大于或等于初始CPU数。CPU拓扑结构的乘积必须等于CPU的最大个数。这两个参数都有一个上限，该上限由所选的特定机器类型决定。

如果需要控制CPU拓扑信息的上报，可以指定拓扑参数的值。机器可能只支持参数的一个子集，不同的机器可能支持不同的子集，这取决于相应CPU目标的容量。因此，对于特定的机器类型板，可以通过受支持的子选项定义预期的拓扑层次结构。除了子选项外，还可以提供不支持的参数，但为了正确解析，必须将它们的值设置为1。

必须指定初始CPU数或至少一个拓扑参数。指定的参数必须大于零，不允许显式配置如“cpus=0”。任何省略参数的值将从给定的参数中计算出来。

例如，下面的子选项为只支持套接字/内核/线程的机器定义了CPU拓扑层次结构(机器上总共有2个插槽，每个插槽有2个内核，每个内核有2个线程)。选项中的一些成员可以省略，但它们的值将自动计算:

```shell
-smp 8,sockets=2,cores=2,threads=2,maxcpus=8
```

下面的子选项定义了一个CPU拓扑层次结构(机器上总共有2个插座，每个插座有2个模具，每个模具有2个内核，每个内核有2个线程)，用于支持插座/模具/内核/线程的PC机器。选项中的一些成员可以省略，但它们的值将自动计算:

```shell
-smp 16,sockets=2,dies=2,cores=2,threads=2,maxcpus=16
```

下面的子选项为支持套接字/集群/内核/线程的ARM virt机器定义了一个CPU拓扑层次结构(机器上总共有2个插座，每个插座有2个集群，每个集群有2个内核，每个内核有2个线程)。选项中的一些成员可以省略，但它们的值将自动计算:

```shell
-smp 16,sockets=2,clusters=2,cores=2,threads=2,maxcpus=16
```

历史版本，当计算缺失值时，优先考虑最粗糙的拓扑参数(即套接字优先于核心，而核心优先于线程)，然而，这种行为被认为是容易改变的。在6.2之前，优先级是套接字而不是核心，而不是线程。从6.2开始，优先级是内核而不是套接字，而不是线程。

例如，下面的选项定义了一个机器板，在6.2之前有2个1核插座，在6.2之后有1个2核插座:

```shell
-smp 2
```

> 注意:集群拓扑只会在ACPI中生成，如果在-smp中显式地指定了集群拓扑，集群拓扑就会暴露给来宾。

#### `-numa node[,mem=size][,cpus=firstcpu[-lastcpu]][,nodeid=node][,initiator=initiator]`

#### `-numa node[,memdev=id][,cpus=firstcpu[-lastcpu]][,nodeid=node][,initiator=initiator]`

#### `-numa dist,src=source,dst=destination,val=distance`

#### `-numa cpu,node-id=node[,socket-id=x][,core-id=y][,thread-id=z]`

#### `-numa hmat-lb,initiator=node,target=node,hierarchy=hierarchy,data-type=type[,latency=lat][,bandwidth=bw]`

#### `-numa hmat-cache,node-id=node,size=size,level=level[,associativity=str][,policy=str][,line=size]`

定义一个NUMA节点并为其分配RAM和vcpu。设置源节点到目标节点的NUMA距离。为给定的节点设置ACPI异构内存属性。

传统的VCPU分配使用“CPU”选项，其中第一个CPU和最后一个CPU是CPU索引。每个“cpus”选项表示一个连续的CPU索引范围(如果省略lastcpu，则表示单个VCPU)。一个不连续的vcpu集合可以通过提供多个“cpu”选项来表示。如果在所有节点上省略了“cpu”，vcpu将自动在它们之间分配。

例如，将vcpu 0、1、2、5分配给NUMA节点:
```shell
-numa node,cpus=0-2,cpus=5
```

'cpu'选项是'cpus'选项的新选项，它使用 'socket-id|core-id|thread-id' 属性来使用cpu的拓扑布局属性将cpu对象分配给节点。属性集是特定于机器的，取决于使用的机器类型/ 'smp'选项。可以使用'hotpluggable-cpus' monitor命令查询。'node-id'属性指定将分配CPU对象的节点，在使用'CPU'选项之前，需要使用'node'选项声明node。

例如：
```shell
-M pc \
-smp 1,sockets=2,maxcpus=2 \
-numa node,nodeid=0 -numa node,nodeid=1 \
-numa cpu,node-id=0,socket-id=0 -numa cpu,node-id=1,socket-id=1
```

遗留 'mem' 将给定的RAM数量分配给节点(5.1和更新的机器类型不支持)。'memdev' 将内存从给定的内存后端设备分配给一个节点。如果在所有节点中都省略了'mem'和'memdev'， RAM将在它们之间平均分配。

“mem”和“memdev”是互斥的。此外，如果一个节点使用“memdev”，那么所有节点都必须使用它。

'initiator'是一个附加选项，它指向一个启动器NUMA节点，该节点对该NUMA节点具有最佳性能(最低延迟或最大带宽)。注意，只有当机器属性“hmat”设置为“on”时，才能设置该选项。

下面的示例创建一个具有2个NUMA节点的机器，节点0具有CPU。节点1只有内存，启动器为节点0。注意，因为节点0有CPU，所以默认情况下节点0的启动器是它自己，并且必须是它自己。

```shell
-machine hmat=on \
-m 2G,slots=2,maxmem=4G \
-object memory-backend-ram,size=1G,id=m0 \
-object memory-backend-ram,size=1G,id=m1 \
-numa node,nodeid=0,memdev=m0 \
-numa node,nodeid=1,memdev=m1,initiator=0 \
-smp 2,sockets=2,maxcpus=2  \
-numa cpu,node-id=0,socket-id=0 \
-numa cpu,node-id=0,socket-id=1
```

source和destination是NUMA节点id。distance是从源到目标的NUMA距离。节点到自身的距离总是10。如果任意一对节点都给定了距离，那么所有的节点都必须给定距离。虽然，当每对节点只在一个方向上给出距离时，则假设相反方向上的距离是相同的。然而，如果对一个节点对给出了不对称的距离对，那么所有节点对都必须提供两个方向的距离值，即使它们是对称的。当一个节点与另一个节点不可达时，将pair的距离设置为255。

注意-numa选项不会分配任何指定的资源，它只是将现有资源分配给NUMA节点。这意味着仍然必须使用-m、-smp选项来分别分配RAM和vcpu。

使用' HMAT -lb '在ACPI异构属性内存表(HMAT)中设置启动器和目标NUMA节点之间的System Locality Latency和Bandwidth Information。启动器NUMA节点可以创建内存请求，通常它有一个或多个处理器。目标NUMA节点包含可寻址内存。

在“hmat-lb”选项中，node为NUMA节点id。hierarchy是目标NUMA节点的内存层次结构:如果hierarchy是' memory '，该结构表示内存性能;如果层次结构是'一级|二级|三级'，这个结构表示每个域的内存侧缓存的聚合性能。' data- Type '的类型是这个结构实例表示的数据类型:如果' hierarchy '是' memory '， ' data- Type '是' access|read|write '延迟或' access|read|write '目标内存带宽;如果“hierarchy”为“一级|二级|三级”，“data-type”为“access|read|write”命中时延或“access|read|write”命中目标内存侧缓存带宽。

Lat是延迟值，以纳秒为单位。bw为带宽值，可能的值和单位为NUM[M|G|T]，表示带宽值为每秒NUM个字节(根据所使用后缀的不同可以分为MB/s、GB/s或TB/s)。注意，如果latency或bandwidth值为0，则表示不提供相应的latency或bandwidth信息。

在“hmat-cache”选项中，node-id是内存所属的NUMA-id。Size是内存侧缓存的大小，以字节为单位。Level是这个结构中描述的缓存级别，注意缓存级别0不应该与' hmat-cache '选项一起使用。Associativity是缓存的结合性，可能的值是' none/direct(直接映射)/complex(复杂缓存索引)'。Policy是写策略。line是缓存线的大小，以字节为单位。

例如，以下选项描述2个NUMA节点。节点0有2个cpu和一个ram，节点1只有一个ram。节点0的处理器以5纳秒的访问时延访问节点0的内存，访问带宽为200 MB/s;NUMA节点0中的处理器以10纳秒的访问延迟访问NUMA节点1中的内存，访问带宽为100 MB/s。对于内存侧缓存信息，NUMA节点0和1都有1级内存缓存，大小为10KB，策略为write-back，缓存Line size为8字节:

```shell
-machine hmat=on \
-m 2G \
-object memory-backend-ram,size=1G,id=m0 \
-object memory-backend-ram,size=1G,id=m1 \
-smp 2,sockets=2,maxcpus=2 \
-numa node,nodeid=0,memdev=m0 \
-numa node,nodeid=1,memdev=m1,initiator=0 \
-numa cpu,node-id=0,socket-id=0 \
-numa cpu,node-id=0,socket-id=1 \
-numa hmat-lb,initiator=0,target=0,hierarchy=memory,data-type=access-latency,latency=5 \
-numa hmat-lb,initiator=0,target=0,hierarchy=memory,data-type=access-bandwidth,bandwidth=200M \
-numa hmat-lb,initiator=0,target=1,hierarchy=memory,data-type=access-latency,latency=10 \
-numa hmat-lb,initiator=0,target=1,hierarchy=memory,data-type=access-bandwidth,bandwidth=100M \
-numa hmat-cache,node-id=0,size=10K,level=1,associativity=direct,policy=write-back,line=8 \
-numa hmat-cache,node-id=1,size=10K,level=1,associativity=direct,policy=write-back,line=8
```

#### `-add-fd fd=fd,set=set[,opaque=opaque]`

向fd集添加文件描述符。有效的选项是:

- `fd=fd`：此选项定义将其副本添加到fd集的文件描述符。文件描述符不能是stdin、stdout或stderr。
- `set=set`：该选项定义了要向其中添加文件描述符的fd集的ID。
- `opaque=opaque`: 此选项定义了一个可用于描述fd的自由格式字符串。
    
你可以从fd集合中使用预打开的文件描述符打开一个image:

```shell
qemu-system-x86_64 \
 -add-fd fd=3,set=2,opaque="rdwr:/path/to/file" \
 -add-fd fd=4,set=2,opaque="rdonly:/path/to/file" \
 -drive file=/dev/fdset/2,index=0,media=disk
```

#### `-set group.id.arg=value`

为group类型的项目id设置参数arg

#### `-global driver.prop=value`

#### `-global driver=driver,property=property,value=value`

将驱动属性prop的默认值设置为value，例如:

```shell
qemu-system-x86_64 -global ide-hd.physical_block_size=4096 disk-image.img
```

特别是，您可以使用它来设置由机器模型自动创建的设备的驱动程序属性。要创建一个非自动创建的设备并设置其属性，使用-device。

-global driver。Prop=value 是 -global driver=driver,property=Prop,value=value的简写。即使驱动程序包含一个点。

#### `-boot [order=drives][,once=drives][,menu=on|off][,splash=sp_name][,splash-time=sp_time][,reboot-timeout=rb_timeout][,strict=on|off]`

将引导顺序驱动器指定为驱动器号字符串。有效的驱动器号取决于目标体系结构。x86 PC使用:a, b(软盘1和软盘2)，c(第一个硬盘)，d(第一个CD-ROM)， n-p (Etherboot from network adapter 1-4)，硬盘启动为默认。若要只在第一次启动时应用特定的引导顺序，请指定via一次。注意，order或once参数不应该与设备的bootindex属性一起使用，因为固件实现通常不同时支持这两者。

交互式引导菜单/提示可以通过menu=on启用，只要固件/BIOS支持它们。默认为非交互式引导。

如果固件/ bios支持，当选项splash=sp_name和menu=on时，可以将splash图片传递给bios，使用户能够将其显示为logo。目前Seabios for X86系统支持它。限制:splash文件可以是jpeg文件或24bpp格式的BMP文件(真彩色)。SVGA模式应该支持分辨率，所以推荐320x240、640x480、800x640。

超时可以传递给bios，当引导失败时，guest将暂停rb_timeout ms，然后重新启动。如果rb_timeout为' -1 '，guest将不会重新启动，qemu默认将' -1 '传递给bios。目前Seabios for X86系统支持它。

在固件/BIOS支持的情况下，通过strict=on进行严格引导。这仅在引导优先级由bootindex选项更改时才生效。默认为非严格引导。

```shell
# try to boot from network first, then from hard disk
qemu-system-x86_64 -boot order=nc
# boot from CD-ROM first, switch back to default order after reboot
qemu-system-x86_64 -boot once=d
# boot with a splash picture for 5 seconds.
qemu-system-x86_64 -boot menu=on,splash=/root/boot.bmp,splash-time=5000
```

> 注意:传统格式' -boot drives '仍然被支持，但不鼓励使用，因为它可能会从未来的版本中删除。

#### `-m [size=]megs[,slots=n,maxmem=size]`

设置客户启动RAM大小为兆字节。缺省值是128 MiB。可以选择使用后缀“M”或“G”分别表示以兆字节或千兆字节为单位的值。可选插槽对，maxmem可用于设置热插拔内存插槽数量和最大内存数量。注意，maxmem必须与页面大小对齐。

例如，下面的命令行将客户机启动RAM大小设置为1GB，创建3个插槽来热插拔额外的内存，并将客户机可以达到的最大内存设置为4GB:

```shell
qemu-system-x86_64 -m 1G,slots=3,maxmem=4G
```

如果slots和maxmem未指定，内存热插拔将不会启用，并且客户启动RAM将永远不会增加。

#### `-mem-path path`

从路径中临时创建的文件分配客户RAM。

#### `-mem-prealloc`

使用-mem-path时预分配内存。

#### `-k language`

使用键盘布局语言(例如fr代表法语)。这个选项只在不容易获得原始PC键码的情况下才需要(例如在mac上，使用一些X11服务器或VNC或curses显示)。您通常不需要在PC/Linux或PC/Windows主机上使用它。

可用值是：
```
ar  de-ch  es  fo     fr-ca  hu  ja  mk     no  pt-br  sv
da  en-gb  et  fr     fr-ch  is  lt  nl     pl  ru     th
de  en-us  fi  fr-be  hr     it  lv  nl-be  pt  sl     tr
```

> 默认值是：en-us

#### `-audio-help`

将显示当前指定(已弃用)环境变量的-audiodev等效项。

#### `-audio [driver=]driver,model=value[,prop[=value][,...]]`

此选项是一次性配置客户音频硬件和主机音频后端的快捷方式。驱动程序选项与下面对应的-audiodev选项相同。客户硬件模型可以用model=modelname来设置。

使用“driver=help”列出可用的驱动程序，使用“model=help”列出可用的设备类型。

下面的两个例子完全相同，展示了如何使用-audio来缩短命令行长度:

```
qemu-system-x86_64 -audiodev pa,id=pa -device sb16,audiodev=pa
qemu-system-x86_64 -audio pa,model=sb16
```

#### `-audiodev [driver=]driver,id=id[,prop[=value][,...]]`

添加一个由id标识的新的音频后端驱动程序。有全局和驱动程序特定的属性。有些值可以为输入和输出设置不同的值，它们被标记为in|out..你可以用in来设置输入的属性。Prop和输出的out.prop属性。例如:

```shell
-audiodev alsa,id=example,in.frequency=44110,out.frequency=8000
-audiodev alsa,id=example,out.channels=1 # leaves in.channels unspecified
```

> 注意:参数验证已知是不完整的，在许多情况下，指定无效选项会导致QEMU打印错误消息并在没有声音的情况下继续模拟。

可用的全局配置是：
- `id=identifier`：标识音频后端
- `timer-period=period`：以微秒为单位设置音频子系统使用的计时器周期。默认值是10000(10毫秒)。
- `in|out.mixing-engine=on|off`：使用QEMU的混合引擎在QEMU内混合所有流，并转换后端不支持的音频格式。当关闭时，固定设置也必须关闭。注意，禁用此选项意味着所选后端必须支持多个流和虚拟卡使用的音频格式，否则将得不到声音。不建议禁用此选项，除非你想使用5.1或7.1音频，因为混合引擎只支持单声道和立体声音频。默认开启。
- `in|out.fixed-settings=on|off`: 使用固定的主机音频设置。当关闭时，它将根据客人打开声卡的方式而改变。在这种情况下，您不能指定频率、渠道或格式。默认开启。
- `in|out.frequency=frequency`: 指定使用固定设置时使用的频率。默认为44100Hz。
- `in|out.channels=channels`: 指定使用固定设置时要使用的通道数量。默认为2(立体声)。
- `in|out.format=format`: 指定使用固定设置时要使用的示例格式。取值范围:s8, s16, s32, u8, u16, u32, f32。默认为s16。
- `in|out.voices=voices`：指定要使用的声音数量。默认值是1。
- `in|out.buffer-length=usecs`: 以微秒为单位设置缓冲区的大小。

#### `-audiodev none,id=id[,prop[=value][,...]]`

创建一个丢弃所有输出的虚拟后端。此后端没有后端特定的属性。

#### `-audiodev alsa,id=id[,prop[=value][,...]]`

使用ALSA创建后端。此后端仅在Linux上可用。

ALSA的特定选项有:

- `in|out.dev=device`: 指定用于输入和/或输出的ALSA设备。默认就是默认。
- `in|out.period-length=usecs`: 以微秒为单位设置周期长度。
- `in|out.try-poll=on|off`: 尝试对设备使用轮询模式。默认开启。
- `threshold=threshold`: 回放开始时的阈值(以微秒为单位)。默认值为0。

#### `-audiodev coreaudio,id=id[,prop[=value][,...]]`

创建一个后端使用苹果的核心音频。此后端仅在Mac OS上可用，且仅支持回放。

核心音频的具体选项是:

- `in|out.buffer-count=count`: 设置缓冲区的计数。

#### `-audiodev dsound,id=id[,prop[=value][,...]]`

使用微软的DirectSound创建一个后端。此后端仅在Windows上可用，仅支持回放。

DirectSound的具体选项有:

- `latency=usecs`: 增加额外的usecs微秒延迟播放。默认值是10000(10毫秒)。

#### `-audiodev oss,id=id[,prop[=value][,...]]`

使用OSS创建后端。这个后端可以在大多数类unix系统上使用。

OSS的具体选项有:
- `in|out.dev=device`: 指定要使用的网管设备的文件名。默认为/dev/dsp。
- `in|out.buffer-count=count`: 设置缓冲区的计数。
- `in|out.try-poll=on|of`: 尝试对设备使用轮询模式。默认开启。
- `try-mmap=on|off`: 尝试使用内存映射设备访问。默认是关闭的。
- `exclusive=on|off`: 以独占模式打开设备(在这种情况下vmix将不起作用)。默认是关闭的。
- `dsp-policy=policy`: 设置定时策略(0到10之间，数值越小，延迟越小，CPU占用率越高)。使用-1使用由buffer和buffer-count指定的缓冲区大小。如果没有OSS 4，则忽略此选项。缺省值是5。

#### `-audiodev pa,id=id[,prop[=value][,...]]`

使用PulseAudio创建后端。这个后端在大多数系统上都可用。

PulseAudio的特定选项是:
- `server=server`: 设置要连接的PulseAudio服务器。
- `in|out.name=sink`: 使用指定的源/接收器进行记录/回放。
- `in|out.latency=usecs`: 期望的延迟(微秒)。PulseAudio服务器将尝试执行此值，但实际延迟可能更低或更高。

#### `-audiodev sdl,id=id[,prop[=value][,...]]`

使用SDL创建后端。这个后端在大多数系统上都可用，但是如果可能的话，您应该使用您平台的本机后端。

SDL特定的选项有:
- `in|out.buffer-count=count`: 设置缓冲区的计数。

#### `-audiodev sndio,id=id[,prop[=value][,...]]`

使用SNDIO创建后端。这个后端可以在OpenBSD和大多数其他类unix系统上使用。

Sndio特定的选项有:
- `in|out.dev=device`: 指定用于输入和/或输出的sndio设备。默认就是默认。
- `in|out.latency=usecs`: 设置所需的周期长度，以微秒为单位。

#### `-audiodev spice,id=id[,prop[=value][,...]]`

创建通过SPICE发送音频的后端。这个后端需要-spice，在这种情况下会自动选择，所以通常可以忽略这个选项。此后端没有后端特定的属性。

#### `-audiodev wav,id=id[,prop[=value][,...]]`

创建一个后端，将音频写入WAV文件。

后端特定的选项是:
- `path=path`: 将录制的音频写入指定的文件。默认为qemu.wav。

#### `-device driver[,prop[=value][,...]]`
添加设备驱动程序。Prop =value设置驱动程序属性。有效的属性取决于驱动程序。要获得有关可能的驱动程序和属性的帮助，请使用-device help和-device driver,help。

#### `-device ipmi-bmc-sim,id=id[,prop[=value][,...]]`

添加IPMI BMC。这是对通常位于系统上的硬件管理接口处理器的模拟。它提供了一个看门狗和重置和电源控制系统的能力。您需要将其连接到IPMI接口以使其有用

BMC使用的IPMI从地址。默认值是0x20。该地址为BMC在管理节点I2C网络上的地址。如果你不知道这意味着什么，忽略它是安全的。

- `id=id`: 接口使用该设备的BMC id。
- `slave_addr=val`: 定义BMC使用的从地址。默认值是0x20。
- `sdrfile=file`: 包含原始传感器数据记录(SDR)数据的文件。默认为none。
- `fruareasize=val`: FRU (Field Replaceable Unit)区域大小。默认值是1024。
- `frudatafile=file`: 包含原始现场可更换单元(FRU)库存数据的文件。默认为none。
- `guid=uuid`: 值为BMC的GUID，标准UUID格式。如果设置了，get“get GUID”命令到BMC将返回它。否则“Get GUID”将返回一个错误。

#### `-device ipmi-bmc-extern,id=id,chardev=id[,slave_addr=val]`

添加到外部IPMI BMC模拟器的连接。不要像上面那样在本地模拟BMC，而是连接到提供IPMI服务的外部实体。


连接到外部BMC模拟器。如果您这样做，强烈建议您在连接丢失时使用" reconnect= " chardev选项重新连接到模拟器。注意，如果不小心使用，可能会造成安全问题，因为该接口具有发送重置、nmi和关闭虚拟机的能力。最好QEMU连接到在本地主机上的安全端口上运行的外部模拟器，这样模拟器和QEMU都不会暴露给任何外部网络。


参见“lanserv/README”。OpenIPMI库中的vm”文件，以获取外部接口的更多详细信息。

#### `-device isa-ipmi-kcs,bmc=id[,ioport=val][,irq=val]`

在ISA总线上添加KCS IPMI接口。如果合适的话，还会添加相应的ACPI和SMBIOS条目。

- `bmc=id`: 要连接的BMC，上面的ipmi-bmc-sim或ipmi-bmc-extern中的一个。
- `ioport=val`: 定义接口I/O地址。KCS的默认值是0xca0。
- `irq=val`: 定义要使用的中断。缺省值是5。若要禁用中断，请将此值设置为0。

#### `-device isa-ipmi-bt,bmc=id[,ioport=val][,irq=val]`

类似于KCS接口，但定义了BT接口。默认端口为“0xe4”，默认中断为“5”。

#### `-device pci-ipmi-kcs,bmc=id`

在PCI总线上添加KCS IPMI接口。
- `bmc=id`: 要连接的BMC，上面的ipmi-bmc-sim或ipmi-bmc-extern中的一个。

#### `-device pci-ipmi-bt,bmc=id`

类似于KCS接口，但在PCI总线上定义了一个BT接口。

#### `-device intel-iommu[,option=...]`

只有-machine q35支持这一点，它将在客户机中启用Intel VT-d仿真。它支持以下选项:

- `intremap=on|off (default: auto)`: 这将启用中断重映射功能。必须启用完整的x2apic。目前它只支持kvm内核-irqchip模式关闭或分裂，而完整的内核-irqchip还不支持。默认值为auto，由kernel-irqchip的模式决定。
- `caching-mode=on|off (default: off)`: 这将为VT-d模拟设备启用缓存模式。当启用缓存模式时，每个客户机DMA缓冲区映射将以同步方式从客户机IOMMU驱动程序到vIOMMU设备生成IOTLB失效。-device vfio-pci需要使用VT-d设备，因为主机分配的设备需要在客户DMA启动之前在主机上设置DMA映射。
- `device-iotlb=on|off (default: off)`: 这为仿真VT-d设备启用了device-iotlb功能。到目前为止，virtio/vhost应该是这个参数的唯一真实用户，并为设备配置了ats=on。
- `aw-bits=39|48 (default: 39)`: 这决定了IOVA地址空间的地址宽度。对于3级IOMMU页表，地址空间宽度为39位;对于4级IOMMU页表，地址空间宽度为48位。

#### `-name name`

设置来宾的名称。该名称将显示在SDL窗口标题中。该名称也将用于VNC服务器。还可以选择在Linux中设置顶部可见进程名。在Linux上还可以启用单个线程的命名，以帮助调试。

#### `-uuid uuid`

设置系统UUID。

### 块设备配置项(Block device options)

QEMU块设备处理选项有很长的历史，随着块层的特性集和复杂性的增长，它已经经历了几次迭代。许多QEMU的在线指南经常引用旧的和已弃用的选项，这可能导致混淆。

推荐的描述磁盘的现代方法是组合使用-device来指定硬件设备和-blockdev来描述后端。设备定义客户机看到的内容，后端描述QEMU如何处理数据。

#### `-fda file`

#### `-fdb file`

使用文件作为软盘0/1映像(请参阅系统仿真用户指南中的磁盘映像章节)。

#### `-hda file`

#### `-hdb file`

#### `-hdc file`

#### `-hdd file`

使用文件作为硬盘0、1、2或3映像(请参阅系统仿真用户指南中的磁盘映像章节)。

#### `-cdrom file`

使用文件作为光盘镜像(-hdc和-cdrom不能同时使用)。可以使用/dev/cdrom作为文件名来使用主机光盘。

#### `-blockdev option[,option[,option[,...]]]`

定义一个新的块驱动程序节点。有些选项适用于所有块驱动程序，其他选项只适用于特定的块驱动程序。请参阅下面的通用选项和最常见块驱动程序的选项列表。

期望引用另一个节点(例如文件)的选项可以通过两种方式给出。要么指定已经存在的节点的节点名(file=node-name)，要么内联定义一个新节点，在点(file.filename=path,file.aio=native)后面为引用的节点添加选项。

使用-blockdev创建的块驱动程序节点可以用于客户设备，方法是在定义块设备的-device参数中为drive属性指定其节点名。

任何块驱动程序节点的有效选项:
- `driver`: 指定给定节点要使用的块驱动程序。
- `node-name`: 这定义了块驱动程序节点的名称，稍后将通过该名称引用它。名称必须是唯一的，即它不能与不同块驱动程序节点的名称匹配，或者(如果您也使用-drive)驱动器的ID。如果不指定节点名称，则自动生成。生成的节点名称不是可预测的，并且在QEMU调用之间会发生变化。对于顶层，必须指定显式的节点名称。
- `read-only`: 只读打开节点。客户写入尝试将失败。请注意，有些块驱动程序只支持只读访问，无论是在一般情况下还是在某些配置中。在这种情况下，默认值read-only=off不起作用，必须显式指定该选项。
- `auto-read-only`: 如果设置了auto-read-only=on，即使请求read-only=off, QEMU也可能会返回到只读使用，甚至根据需要在模式之间切换，例如，取决于映像文件是否可写，或者是否有写入用户连接到节点。
- `force-share`: 覆盖QEMU的映像锁定系统，强制节点对通常请求独占访问的权限使用较弱的共享访问。当多个实例可能打开同一个文件时(无论QEMU的调用是第一个实例还是第二个实例)，两个实例都必须允许第二个实例成功打开文件的共享访问。
    > 启用force-share=on需要read-only=on。
- `cache.direct`: 使用cache.direct=on可以避免主机页缓存。这将尝试直接对客户内存执行磁盘IO。QEMU仍然可以执行数据的内部复制。
- `cache.no-flush`: 如果不关心主机故障带来的数据完整性，可以使用cache.no-flush=on。这个选项告诉QEMU它永远不需要向磁盘写入任何数据，而是可以将数据保存在缓存中。如果出现任何问题，比如主机断电、磁盘存储意外断开等，那么您的映像很可能无法使用。
- `discard=discard`: discard的值是“ignore”(或“off”)或“unmap”(或“on”)之一，并控制是否忽略丢弃(也称为修剪或unmap)请求或将其传递给文件系统。某些机器类型可能不支持丢弃请求。
- `detect-zeroes=detect-zeroes`：detect-zero是“off”，“on”或“unmap”，并允许操作系统自动将普通的零写转换为驱动程序特定的优化的零写命令。你甚至可以选择“unmap”，如果discard被设置为“unmap”，允许零写转换为unmap操作。

#### `Driver-specific options for file`

这是用于访问常规文件的协议级块驱动程序。

- `filename`: 映像文件在本地文件系统中的路径
- `aio`: 指定AIO后端(threads/native/io_uring，默认值:threads)
- `locking`: 指定映像文件是否受Linux OFD / POSIX锁保护。如果可用，默认使用Linux Open File Descriptor API，否则不应用锁。(auto/on/off，默认:auto)

例子：
```shell
-blockdev driver=file,node-name=disk,filename=disk.img
```

#### `Driver-specific options for raw`

这是原始图像的图像格式块驱动程序。它通常堆叠在协议级块驱动程序(如文件)之上。

- `file`: 数据源块驱动程序节点(例如文件驱动程序节点)的引用或定义

例1:
```shell
-blockdev driver=file,node-name=disk_file,filename=disk.img
-blockdev driver=raw,node-name=disk,file=disk_file
```

例2:
```shell
-blockdev driver=raw,node-name=disk,file.driver=file,file.filename=disk.img
```

#### `Driver-specific options for qcow2`

这是qcow2图像的图像格式块驱动程序。它通常堆叠在协议级块驱动程序(如文件)之上。

- `file`: 数据源块驱动程序节点(例如文件驱动程序节点)的引用或定义
- `backing`: 对备份文件块设备的引用或定义(默认情况下取自映像文件)。为了禁用默认的备份文件，这里允许传递null。
- `lazy-refcounts`: 是否启用惰性引用计数功能(on/off;默认是从图像文件中获取的)
- `cache-size`: L2表和refcount块缓存的最大总大小(默认:L2 -cache-size和refcount-cache-size的总和)
- `l2-cache-size`: L2表缓存的最大大小(以字节为单位)(默认:如果没有指定cache-size，则Linux平台上为32M，非Linux平台上为8M;否则，在cache-size范围内尽可能大，同时允许请求的或最小的refcount缓存大小)
- `refcount-cache-size`: refcount块缓存的最大大小(以字节为单位)(默认为集群大小的4倍;或者如果指定了cache-size，则指定不用于L2缓存的部分)
- `cache-clean-interval`: 清理L2和refcount缓存中未使用的条目。时间间隔以秒为单位。支持平台默认值为600，其他平台默认值为0。将其设置为0将禁用此特性。
- `pass-discard-request`: qcow2设备的丢弃请求是否应该转发到数据源(开/关;默认值:如果指定discard=unmap则打开，否则关闭)
- `pass-discard-snapshot`: 当快照操作(例如删除快照)释放qcow2文件中的集群时，是否应该发出对数据源的丢弃请求(on/off;默认值:)
- `pass-discard-other`: 对于数据源的丢弃请求是否应该在集群被释放的其他情况下发出(on/off;默认值:)
- `overlap-check`: 对映像的写入执行哪些重叠检查(none/constant/cached/all;默认值:缓存)。有关详细信息或更细粒度的控制，请参阅blockdev-add的QAPI文档。

    例1:
    ```shell
    -blockdev driver=file,node-name=my_file,filename=/tmp/disk.qcow2
    -blockdev driver=qcow2,node-name=hda,file=my_file,overlap-check=none,cache-size=16777216
    ```

    例2:
    ```shell
    -blockdev driver=qcow2,node-name=disk,file.driver=http,file.filename=http://example.com/image.qcow2
    ```
- `Driver-specific options for other drivers`: 请参考blockdev-add QMP命令的QAPI文档。

#### `-drive option[,option[,option[,...]]]`

定义一个新的驱动器。这包括创建一个块驱动程序节点(后端)以及一个客户设备，并且主要是定义相应的-blockdev和-device选项的快捷方式。

-drive接受-blockdev接受的所有选项。此外，它还知道以下选项:

- `file=file`: 此选项定义与此驱动器一起使用的磁盘映像(请参阅系统仿真用户指南中的磁盘映像章节)。如果文件名包含逗号，则必须将其加倍(例如，" file=my，，file "来使用文件" my,file ")。可以使用特定于协议的url指定iSCSI设备等特殊文件。有关更多信息，请参阅“设备URL语法”部分。

- `if=interface`: 此选项定义驱动器连接到哪种类型的接口上。可选类型有:ide、scsi、sd、mtd、floppy、pflash、virtio、none。
- `bus=bus,unit=unit`: 这些选项通过定义总线号和单元id来定义驱动器的连接位置。
- `index=index`: 此选项通过使用给定接口类型的可用连接器列表中的索引来定义驱动器的连接位置。
- `media=media`: 此选项定义媒体类型:磁盘或光盘。
- `snapshot=snapshot`: 快照是“开启”或“关闭”，并控制给定驱动器的快照模式(参见-snapshot)。
- `cache=cache`: Cache为“none”、“writeback”、“insecure”、“directsync”或“writethrough”，并控制主机缓存如何访问块数据。这是一个设置缓存的快捷方式。直接和缓存。No-flush选项(如在-blockdev中)，以及缓存。Writeback，它为块客户设备提供了一个默认的写缓存选项(如-device)。模式对应如下设置:
    ||cache.writeback|cache.direct|cache.no-flush|
    |----|----|----|----|
    |writeback|on|off|off|
    |none|on|on|off|
    |writethrough|off|off|off|
    |directsync|off|on|off|
    |unsafe|on|off|on|
    > 默认为 cache=writeback。
- `aio=aio`: aio是“threads”、“native”或“io_uring”，可以在基于pthread的磁盘I/O、本机Linux aio或Linux io_uring API之间进行选择。
- `format=format`: 指定将使用哪种磁盘格式，而不是检测格式。可用于指定format=raw以避免解释不受信任的格式标头。
- `werror=action,rerror=action`: 指定对写错误和读错误采取的操作。有效的操作是:“ignore”(忽略错误并尝试继续)，“stop”(暂停QEMU)，“report”(向客户报告错误)，“enospc”(仅在主机磁盘已满时暂停QEMU;否则向客人报告错误)。默认设置为werror=enospc和rerror=report。
- `copy-on-read=copy-on-read`: “读时复制”选项为“开”或“关”，允许是否将读备份文件扇区复制到映像文件中。
- `bps=b,bps_rd=r,bps_wr=w`: 为所有请求类型或仅为读或写指定以字节/秒为单位的带宽限制。较小的值可能导致超时或挂起。磁盘的安全最小值为2mb /s。
- `bps_max=bm,bps_rd_max=rm,bps_wr_max=wm`: 指定每秒字节数的突发，可以用于所有请求类型，也可以仅用于读或写。突发允许来宾I/O暂时超过限制。
- `iops=i,iops_rd=r,iops_wr=w`: 以每秒请求数为单位指定请求速率限制，可以针对所有请求类型，也可以仅针对读或写。
- `iops_max=bm,iops_rd_max=rm,iops_wr_max=wm`: 为所有请求类型指定每秒请求的突发数，或者仅为读或写。突发允许来宾I/O暂时超过限制。
- `iops_size=is`: 为了达到iops节流的目的，让请求的每一个字节都算作一个新请求。使用此选项可防止客户通过发送更少但更大的请求来规避iops限制。
- `group=g`: 加入一个给定名称为g的节流配额组。同一组中的所有驱动器一起入账。使用此选项可防止客户使用许多小磁盘而不是单个较大磁盘来规避节流限制。

    默认情况下，缓存。使用Writeback =on模式。一旦数据出现在主机页缓存中，它就会报告数据写入已完成。这是安全的，只要您的客户操作系统确保在需要的地方正确地刷新磁盘缓存。如果您的客户操作系统不能正确处理易失性磁盘写缓存，并且您的主机崩溃或断电，那么客户可能会遇到数据损坏。

    对于这样的客户机，您应该考虑使用cache.writeback=off。这意味着将使用主机页缓存读取和写入数据，但是只有在QEMU确保对磁盘的每次写入都刷新之后，才会向客户机发送写入通知。请注意，这会对性能产生重大影响。

    当使用-snapshot选项时，总是使用不安全的缓存。

    读时复制避免重复访问相同的备份文件扇区，当备份文件通过较慢的网络时非常有用。默认情况下，读时复制是关闭的。

    除了-cdrom，你可以使用:
    ```shell
    qemu-system-x86_64 -drive file=file,index=2,media=cdrom
    ```

    除了-hda， -hdb， -hdc， -hdd，你可以使用:
    ```shell
    qemu-system-x86_64 -drive file=file,index=0,media=disk
    qemu-system-x86_64 -drive file=file,index=1,media=disk
    qemu-system-x86_64 -drive file=file,index=2,media=disk
    qemu-system-x86_64 -drive file=file,index=3,media=disk
    ```

    你可以从fd集合中使用预打开的文件描述符打开一个镜像:
    ```shell
    qemu-system-x86_64 \
        -add-fd fd=3,set=2,opaque="rdwr:/path/to/file" \
        -add-fd fd=4,set=2,opaque="rdonly:/path/to/file" \
        -drive file=/dev/fdset/2,index=0,media=disk
    ```

    您可以将一个CDROM连接到ide0的从机:
    ```shell
    qemu-system-x86_64 -drive file=file,if=ide,index=1,media=cdrom
    ```

    如果你不指定" file= "参数，你定义一个空驱动器:
    ```shell
    qemu-system-x86_64 -drive if=ide,index=1,media=cdrom
    ```

    除了-fda， -fdb，你可以使用:
    ```shell
    qemu-system-x86_64 -drive file=file,index=0,if=floppy
    qemu-system-x86_64 -drive file=file,index=1,if=floppy
    ```

    默认情况下，interface为“ide”，index自动递增:
    ```shell
    qemu-system-x86_64 -drive file=a -drive file=b
    ```

#### `-mtdblock file`

使用文件作为板上闪存镜像。

#### `-sd file`

使用文件作为SecureDigital卡映像。

#### `-snapshot`

写入临时文件而不是磁盘映像文件。在这种情况下，您使用的原始磁盘映像不会被写回。但是，您可以通过按C-a s强制回写(请参阅系统仿真用户指南中的磁盘映像章节)。

#### `-fsdev local,id=id,path=path,security_model=security_model [,writeout=writeout][,readonly=on][,fmode=fmode][,dmode=dmode] [,throttling.option=value[,throttling.option=value[,...]]]`

#### `-fsdev proxy,id=id,socket=socket[,writeout=writeout][,readonly=on]`

#### `-fsdev proxy,id=id,sock_fd=sock_fd[,writeout=writeout][,readonly=on]`

#### `-fsdev synth,id=id[,readonly=on]`

定义一个新的文件系统设备。有效的选项是:

- `local`: 对文件系统的访问由QEMU完成。
- `proxy`: 对文件系统的访问由virtfs-proxy-helper(1)完成。
- `synth`: 合成文件系统，仅供QTests使用。
- `id=id`: 此设备的标识符。
- `path=path`: 文件系统设备的导出路径。该路径下的文件将可用于客户端上的9p客户端。
- `security_model=security_model`: 指定用于此导出路径的安全模型。支持的安全模型是“passthrough”，“mapped-xattr”，“mapped-file”和“none”。在“直通”安全模型中，文件存储时使用的凭据与在客户机上创建的凭据相同。这要求QEMU以根用户身份运行。在“mapped-xattr”安全模型中，一些文件属性如uid、gid、模式位和链接目标被存储为文件属性。对于“mapped-file”，这些属性存储在隐藏的.virtfs_metadata目录中。此安全模型导出的目录不能与其他unix工具交互。“none”安全模型与直通相同，只是如果服务器未能设置文件属性(如所有权)，则服务器不会报告失败。仅本地fsdriver必须使用安全模型。其他fsdrivers(如proxy)不将安全模型作为参数。
- `writeout=writeout`: 这是一个可选参数。唯一支持的值是immediate。这意味着主机页缓存将用于读写数据，但只有当存储子系统报告数据已被写入时，才会向客户机发送写入通知。
- `readonly=on`: 允许将9p共享导出为只读挂载。默认情况下给予读写权限。
- `socket=socket`: 允许代理文件系统驱动程序使用传入的套接字文件与virtfs-proxy-helper(1)通信。
- `sock_fd=sock_fd`: 允许代理文件系统驱动程序使用传入的套接字描述符与virtfs-proxy-helper(1)通信。通常，像libvirt这样的助手会创建socketpair并将其中一个fds作为sock_fd传递。
- `fmode=fmode`: 指定主机上新创建文件的默认模式。仅适用于“mapped-xattr”和“mapped-file”安全模型。
- `dmode=dmode`: 指定主机上新创建目录的默认模式。仅适用于“mapped-xattr”和“mapped-file”安全模型。
- `throttling.bps-total=b,throttling.bps-read=r,throttling.bps-write=w`: 为所有请求类型或仅为读或写指定以字节/秒为单位的带宽限制。
- `throttling.bps-total-max=bm,bps-read-max=rm,bps-write-max=wm`: 指定每秒字节数的突发，可以用于所有请求类型，也可以仅用于读或写。突发允许来宾I/O暂时超过限制。
- `throttling.iops-total=i,throttling.iops-read=r, throttling.iops-write=w`: 以每秒请求数为单位指定请求速率限制，可以针对所有请求类型，也可以仅针对读或写。
- `throttling.iops-total-max=im,throttling.iops-read-max=irm, throttling.iops-write-max=iwm`: 为所有请求类型指定每秒请求的突发数，或者仅为读或写。突发允许来宾I/O暂时超过限制。
- `throttling.iops-size=is`: 为了达到iops节流的目的，让请求的每一个字节都算作一个新请求。

#### `-device virtio-9p-type,fsdev=id,mount_tag=mount_tag`

- `type`: 指定要使用的变量。根据机器类型，支持的值为“pci”、“ccw”或“device”。
- `fsdev=id`: 指定与-fsdev选项一起指定的id值。
- `mount_tag=mount_tag`: 指定来宾用来挂载此导出点的标记名称。

#### `-virtfs local,path=path,mount_tag=mount_tag ,security_model=security_model[,writeout=writeout][,readonly=on] [,fmode=fmode][,dmode=dmode][,multidevs=multidevs]`

#### `-virtfs proxy,socket=socket,mount_tag=mount_tag [,writeout=writeout][,readonly=on]`

#### `-virtfs proxy,sock_fd=sock_fd,mount_tag=mount_tag [,writeout=writeout][,readonly=on]`

#### `-virtfs synth,mount_tag=mount_tag`

定义一个新的虚拟文件系统设备，并使用virtio-9p-device(又名9pfs)将其公开给客户机，这本质上意味着主机上的某个目录通过使用主机和客户机之间通信的9P网络协议作为传递文件系统，可以被客户机直接访问，如果需要，甚至可以被多个客户机同时共享。

> 注意-virtfs实际上只是其通用形式-fsdev -device virtio-9p-pci的方便快捷方式。

直通文件系统选项的一般形式是:

- `local`: 对文件系统的访问由QEMU完成。
- `proxy`: 对文件系统的访问由virtfs-proxy-helper(1)完成。
- `synth`: 合成文件系统，仅供QTests使用。
- `id=id`: 文件系统设备的标识符
- `path=path`: 文件系统设备的导出路径。该路径下的文件将可用于客户端上的9p客户端。
- `security_model = security_model`: 指定用于此导出路径的安全模型。支持的安全模型是“passthrough”，“mapped-xattr”，“mapped-file”和“none”。在“直通”安全模型中，文件存储时使用的凭据与在客户机上创建的凭据相同。这要求QEMU以根用户身份运行。在“mapped-xattr”安全模型中，一些文件属性如uid、gid、模式位和链接目标被存储为文件属性。对于“mapped-file”，这些属性存储在隐藏的.virtfs_metadata目录中。此安全模型导出的目录不能与其他unix工具交互。“none”安全模型与直通相同，只是如果服务器未能设置文件属性(如所有权)，则服务器不会报告失败。仅本地fsdriver必须使用安全模型。其他fsdrivers(如proxy)不将安全模型作为参数。
- `writeout=writeout`: 这是一个可选参数。唯一支持的值是immediate。这意味着主机页缓存将用于读写数据，但只有当存储子系统报告数据已被写入时，才会向客户机发送写入通知。
- `readonly=on`: 允许将9p共享导出为只读挂载。默认情况下给予读写权限。
- `socket=socket`: 允许代理文件系统驱动程序使用传入的套接字文件与virtfs-proxy-helper(1)通信。通常，像libvirt这样的助手会创建socketpair并将其中一个fds作为sock_fd传递。
- `sock_fd`: 允许代理文件系统驱动程序使用传入的' sock_fd '作为与virtfs-proxy-helper(1)接口的套接字描述符。
- `fmode=fmode`: 指定主机上新创建文件的默认模式。仅适用于“mapped-xattr”和“mapped-file”安全模型。
- `dmode=dmode`: 指定主机上新创建目录的默认模式。仅适用于“mapped-xattr”和“mapped-file”安全模型。
- `mount_tag=mount_tag`: 指定来宾用来挂载此导出点的标记名称。
- `multidevs=multidevs`: 指定如何处理与9p导出共享的多个设备。支持的行为是“重映射”、“禁止”或“警告”。后者是默认行为，virtfs 9p希望只有一个设备共享同一个导出，如果多个设备通过同一个9p导出共享和访问，那么主机端qemu只会记录(一次)警告消息。为了避免guest上的文件ID冲突，您应该为每个设备创建一个单独的virtfs导出，以便与guest共享(推荐的方式)，或者您可以使用“remap”，它允许您仅使用一个导出共享多个设备，这是通过将原始inode号从主机重新映射到guest来实现的，以防止此类冲突。在这种用例中需要重新映射inode，因为来自主机的原始设备id永远不会传递给客户，也不会在客户上公开。相反，与virtfs共享的导出的所有文件总是在guest上共享相同的设备id。因此，两个具有相同inode号但实际上来自主机上不同设备的文件将导致文件ID冲突，从而在guest上产生潜在的错误行为。另一方面，“forbid”假设像“warn”一样，同一导出只共享一个设备，但是它不仅会记录警告消息，还会拒绝访问guest上的其他设备。请注意，“禁止”目前不会阻止所有可能的文件访问操作(例如，readdir()仍然会从其他设备返回条目)。

#### `-iscsi`

配置iSCSI会话参数。

### USB选项(USB convenience options)

#### `-usb`

使用板载USB主机控制器在机器类型上启用USB仿真(如果默认未启用)。请注意，板载USB主机控制器可能不支持USB 3.0。在这种情况下，-device qemu-xhci可以在具有PCI的机器上使用。

#### `-usbdevice devname`

添加USB设备devname，并在可能和必要的情况下启用板载USB控制器(就像通过-machine USB =on可以做到的那样)。注意，这个选项主要是为了方便用户。更细粒度的控制可以通过选择USB主机控制器(如果需要)和通过-device选项来实现所需的USB设备。例如，不使用-usbdevice mouse，可以使用-device qemu-xhci -device USB -mouse将USB鼠标连接到USB 3.0控制器上(至少在支持PCI且默认没有启用USB控制器的机器上是这样)。详细操作请参见《系统仿真用户指南》中“连接USB设备”章节。devname可以使用的设备有:

- `braille`: 盲文设备。这将使用BrlAPI在真实或假设备上显示盲文输出(即它还自动在USB -braille USB设备旁边创建相应的盲文chardev)。
- `keyboard`: 标准USB键盘。将覆盖PS/2键盘(如果存在)。
- `mouse`: 虚拟鼠标。这将在激活时覆盖PS/2鼠标模拟。
- `tablet`: 使用绝对坐标的指针设备(如触摸屏)。这意味着QEMU无需抓取鼠标就可以报告鼠标位置。还在激活时覆盖PS/2鼠标模拟。
- `wacom-tablet`: 

### 显示配置(Display options)

### i386 target only

### 网络配置(Network options)

### TPM设备选项(TPM device options)

### 引导映像或特定于内核的配置(Boot Image or Kernel specific)

### 调试/高级配置(Debug/Expert options)

### 创建通用对象(Generic object creation)

### 设备URL语法(Device URL Syntax)

## 设备模拟(Device Emulation)

### 通用术语(Common Terms)

### 模拟设备(Emulated Devices)

## 图形界面的特殊按键(Keys in the graphical frontends)

在图形模拟过程中，可以使用特殊的组合键来更改模式。默认的键映射如下所示，但如果你使用-alt-grab，那么修饰符是Ctrl-alt-shift(而不是Ctrl-alt)，如果你使用-ctrl-grab，那么修饰符是右Ctrl键(而不是Ctrl-alt):

## 字符终端的特殊按键(Keys in the character backend multiplexer)

## QEMU 监视器(QEMU Monitor)

QEMU监视器用于向QEMU模拟器提供复杂的命令。你可以用它来:
- 删除或插入可移动媒体图像(如CD-ROM或软盘)。
- 冻结/解冻虚拟机，并从磁盘文件保存或恢复其状态。
- 在没有外部调试器的情况下检查VM状态。

### 命令(commands)

- `help or ? [cmd]`
- `quit or q`
- `exit_preconfig`: 该命令使QEMU退出预配置状态，并使用在预配置状态期间通过QMP监控器在命令行上提供的配置数据继续VM初始化。该命令仅在preconfig状态下可用(即使用-preconfig命令行选项时)。
- `block_resize`: 
...

### 整数表达式(Integer expressions)

监视器理解每个整数参数的整数表达式。您可以使用寄存器名来获取特定CPU寄存器的值，只需在它们前面加上$。

## 磁盘镜像(Disk Images)

QEMU支持许多磁盘映像格式，包括可增长的磁盘映像(它们的大小会随着写入非空扇区而增加)、压缩和加密的磁盘映像。

### 快速开始创建磁盘映像(Quick start for disk image creation)

### 快照模式(Snapshot mode)

### 虚拟机快照(VM snapshots)

### 磁盘映像文件格式(Disk image file formats)

### 只读格式(Read-only formats)

### 使用宿主机驱动器(Using host drives)

### 虚拟FAT磁盘映像(Virtual FAT disk images)

### NBD 访问(NBD access)

### iSCSI LUNs

### GlusterFS磁盘映像(GlusterFS disk images)

### ssh (Secure Shell)磁盘映像

### NVMe磁盘映像

### 磁盘镜像文件锁定

### 过滤驱动

## QEMU virtio-net standby (net_failover)

本文档解释了virtio-net备用特性的设置和使用，该特性用于创建设备的net_failover对。

总的思想是我们有一对设备，一个(vfio-)pci和一个virtio-net设备。在迁移之前，拔出vfio设备，数据通过virtio-net设备，在目标端插入另一个vfio-pci设备来接管数据路径。在客户机中，net_failover内核模块将对具有相同MAC地址的网络设备进行配对。

这两台设备分别称为主设备和备用设备。基于快速硬件的网络设备称为主设备，virtio-net设备称为备用设备。

### 限制

目前只有PCIe设备被允许作为主设备，这一限制在未来可以通过增强的QEMU支持来解除。此外，只允许网络设备作为主设备。用户需要确保主备设备没有插在同一个PCIe插槽。

### Usecase

### 使用

### Hotplug

### 迁移(Migration)

## 直接引导Linux(Direct Linux Boot)

本节解释如何在QEMU中启动Linux内核，而不需要创建一个完全可引导的映像。它对于快速的Linux内核测试非常有用。

语法为:
```shell
qemu-system-x86_64 -kernel bzImage -hda rootdisk.img -append "root=/dev/hda"
```

使用-kernel提供Linux内核映像，使用-append提供内核命令行参数。-initrd选项可用于提供INITRD镜像。

如果不需要图形输出，可以禁用它，并使用-nographic选项将虚拟串口和QEMU监视器重定向到控制台。典型的命令行是:

```shell
qemu-system-x86_64 -kernel bzImage -hda rootdisk.img -append "root=/dev/hda console=ttyS0" -nographic
```

使用Ctrl-a c在串行控制台和监视器之间切换(参见图形化前端中的键)。

## 通用的装载器(Generic Loader)

“加载器”设备允许用户在启动时将多个图像或值加载到QEMU中。

### 将数据加载到内存(Loading Data into Memory Values)

加载器设备允许从命令行设置内存值。这可以通过下面的语法来实现:

```shell
-device loader,addr=<addr>,data=<data>,data-len=<data-len> \
                [,data-be=<data-be>][,cpu-num=<cpu-num>]
````

- `<addr>`: 存储数据的地址。
- `<data>`: 要写入地址的值。数据的最大大小为8字节。
- `<data-len>`: 以字节为单位的数据长度。如果data参数为，则必须包含此参数。
- `<date-be>`: 如果要存储在客户机上的数据应该写入大端数据，则设置为true。默认是写小的端序数据。
- `<cpu-num>`: 应该加载数据的CPU地址空间的编号。如果未指定，则使用第一个CPU的地址空间。

所有值都使用标准的QemuOps解析。这允许用户以所支持的任何格式指定任何值。默认情况下，这些值将被解析为十进制。要使用十六进制值，用户应该在数字前面加上“0x”。

加载值0x8000000e来访问0xfd1a0104的示例如下:
```shell
-device loader,addr=0xfd1a0104,data=0x8000000e,data-len=4
```

### 设置CPU的程序计数器(Setting a CPU’s Program Counter)

加载器设备允许从命令行设置CPU的PC。这可以通过下面的语法来实现:

```shell
-device loader,addr=<addr>,cpu-num=<cpu-num>
```

- `<addr>`: 该值用作CPU的PC。
- `<cpu-num>`: 需要设置为该值的PC所在的CPU号。

所有值都使用标准QemuOpts解析。这允许用户以所支持的任何格式指定任何值。默认情况下，这些值将被解析为十进制。要使用十六进制值，用户应该在数字前面加上“0x”。

将cpu0的PC设置为0x8000的示例如下:

```shell
-device loader,addr=0x8000,cpu-num=0
```

### 加载文件(Loading Files)

加载器设备还允许将文件加载到内存中。它可以加载ELF, U-Boot和Intel HEX可执行格式以及原始图像。语法如下所示:

```shell
-device loader,file=<file>[,addr=<addr>][,cpu-num=<cpu-num>][,force-raw=<raw>]
```
- `<file>`: 要加载到内存中的文件
- `<addr>`: 载入文件的内存地址。这对于原始图像是必需的，对于非原始文件则被忽略。
- `<cpu-num>`: 这指定了应该使用的CPU。这是一个可选参数，将导致CPU的PC被设置为加载原始文件的内存地址或可执行格式头中指定的入口点。此选项应仅用于引导映像。这也将导致映像被写入指定CPU的地址空间。如果不指定，默认为CPU 0。
- `<force-raw>`: 设置' force-raw=on '强制将文件视为原始图像。这可以用来加载支持的可执行格式，就像它们是原始的一样。

所有值都使用标准QemuOpts解析。这允许用户以所支持的任何格式指定任何值。默认情况下，这些值将被解析为十进制。要使用十六进制值，用户应该在数字前面加上“0x”。

例子：
```shell
-device loader,file=./images/boot.elf,cpu-num=0
```

## Guest Loader

guest loader 类似于通用加载器，尽管它针对的是加载管理程序来宾的特定用例。这对于调试管理程序非常有用，而不必跳过固件和引导加载程序的束缚。

guest loader 做两件事:
- 将blob(内核和初始ram磁盘)加载到内存中
- 设置平台FDT数据，以便管理程序可以找到并引导它们

这通常是由像grub这样的引导加载程序使用它的多引导功能完成的。一个典型的例子如下:

```shell
qemu-system-x86_64 -kernel ~/xen.git/xen/xen \
    -append "dom0_mem=1G,max:1G loglvl=all guest_loglvl=all" \
    -device guest-loader,addr=0x42000000,kernel=Image,bootargs="root=/dev/sda2 ro console=hvc0 earlyprintk=xen" \
    -device guest-loader,addr=0x47000000,initrd=rootfs.cpio
```

在上面的例子中，Xen管理程序是通过-kernel参数加载的，并通过-append传递它的引导参数。Dom0客户机被加载到内存区域中。每个blob将在FDT中获得/chosen/module@<addr>条目，以指示它的位置和大小。可以通过使用附加参数来传递附加信息。

目前仅支持使用FDT数据引导的机器是ARM和RiscV virt机器。

### 参数

guest-loader的完整语法是:

```shell
-device guest-loader,addr=<addr>[,kernel=<file>,[bootargs=<args>]][,initrd=<file>]
```

- `addr=<addr>`: 这是必选项，表示blob的起始地址。
- `kernel|initrd=<file>`: 内核或initrd blob的文件名。这两个blob都有“multiboot,module”兼容性字符串，以及“multiboot,kernel”或“multiboot,ramdisk”。
- `bootargs=<args>`: 这是内核blob的一个可选字段，它将通过/chosen/module@<addr>/bootargs节点传递命令。

## `QEMU Barrier Client`

## `VNC security`

## 网络服务的TLS设置(TLS setup for network services)

QEMU中的几乎所有网络服务都能够使用TLS进行会话数据加密，并使用x509证书进行简单的客户端身份验证。下面介绍如何生成适合QEMU使用的证书，适用于VNC服务器、TCP后端字符设备、NBD服务器和客户端以及迁移服务器和客户端。

在高层次上，QEMU要求以PEM格式提供证书和私钥。除了核心字段之外，证书还应该包括各种扩展数据集，包括v3基本约束数据、密钥用途、密钥使用和主题alt名称。

GnuTLS包包含一个名为certtool的命令，可以使用该命令轻松生成所需格式的证书和密钥，并提供预期的数据。或者也可以使用证书管理服务。

至少需要设置一个证书颁发机构，并向每个服务器颁发证书。如果使用x509证书进行身份验证，那么每个客户端也需要颁发一个证书。

假设QEMU网络服务只公开给私有内部网上的客户端，就不需要使用商业证书颁发机构来创建证书。自签名CA就足够了，而且实际上可能更安全，因为它消除了恶意第三方欺骗CA错误颁发证书以模拟您的服务的能力。唯一可能需要商业CA的例外情况是启用VNC websockets服务器并将其直接暴露给远程浏览器客户机。在这种情况下，使用商业CA来避免在web浏览器中安装自定义CA证书可能是有用的。

建议服务器将其证书保存在/etc/pki/qemu中，或者对于无特权用户，将其证书保存在$HOME/.pki/qemu中。
