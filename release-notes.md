===============
 Release Notes
===============

V10.1.0版本--JEWEL
此次主版本的更新将是下一个长期稳定发行版的基础版本。在Infernalis（9.2.X）版本和Hammer（0.94.X）版本后，我们做了许多重大修改，而且更新过程并不简单哦，请仔细阅读版本说明。

这个候选版本有几个已知的issues，参见下文。

V10.1.0版本已知issues

- 当使用jewel版本和infernalis（或者hammer）版本混合运行monitor集群时，任意MDSMap更新都会导致pre-jewel版本的monitor宕机。解决办法是仅升级所有的monitor。此问题已经有解决办法但仍在测试中。一些用来在主镜像和副本之间进行切换的rbd-mirror功能还没有合并进来。

## 在INFERNALIS版本上进行的重要修改 ##

- #####*CephFS*
-  这是第一个CephFS宣称稳定并且生产就绪的版本！一些功能默认被禁用了，包含快照和多活MDS server
-  修复及灾备工具现已功能完善
-  包含了一个新模块cephfs-volume-manager，它提供了一个高级接口用来为Openstack Manila和相似的工程创建共享（shares）
-  现在实验性的在单一集群中支持多CephFS文件系统。
- #####*RWG*
-  Multisite功能已几乎完全被重构和重写，以便支持任意数量的集群节点、双向故障切换和双活设置
-  现在可以通过NFS访问radosgw buckets（实验性的）
-  现已支持AWS4认证协议
-  支持S3请求payer buckets
-  新的多租户架构改善了与Swift的兼容性，为每个用户租户提供了独立容器名称空间
-  现已支持Openstack Keystone v3 API。同时也有一些Switf API的其他小功能和兼容性上的改变，例如删除bulk和SLO（static large objects）
- #####*RBD*
-  新引入了跨集群异步备份（asynchronous replication）RBD镜像的支持。这是通过per-RBD镜像日志来实现的，该日志可经流式处理后通过WAN传输到另一节点，同时一个新的rbd-mirror守护进程来执行跨集群副本操作。
-  Exlusive-lock、object-map、fast-diff和journaling功能可以动态的开启和禁用。Deep-flatten可动态关闭但不能动态重新启用。
     	-  重写了RBD CLI用来提供命令行帮助说明和full bash的完全支持
 		-  现在可以重命名RBD快照
- #####*RADOS*
-  包含了一个实验性的功能—BlueStore，它是一个新的OSD后端。计划在K或L版本中让它成文默认的后端。
-  OSD现在持久化清洗后的结果（scrub results）并提供了librados进行精细查询，包含异常发现特性、获取与某一特定对象相同的替代版本（如果存在）的功能和细粒度控制的修复。

## 在HAMMER版本上进行的重要修改 ##

- #####*General*
-  Ceph的守护进程现已通过systemd来管理(Ubuntu Trusty是个例外，仍沿用upstart来管理)。
		-  Ceph的守护进程以ceph用户而非root运行。
		-  在Red Hat的发行版本中，也存在SeLinux Policy。
- #####*RADOS*
		-  RADOS的cache层现在可通过代理将写操作发送到持久层(base tier)，允许直接处理写操作无需先强制将对象移至cache层。
		-  对SHEC 纠删码的支持已不再标记为实验性的。SHEC通过消耗一些额外的存储空间来换取更快的修复速度。
		-  为（优化）客户端IO、数据恢复、数据清理、快照裁剪提供统一的队列。
		-  对低层级的修复工具（ceph-objectstore-tool）做了许多改进。
		-  为方便使用新的存储后端（如NewStore），内部ObjectStore API已做了重大的修改。
- #####*RGW*
		-  Swift API现已支持对象过期机制
		-  Swift API的兼容性得到了很大的改善
- #####*RBD*
		- 通过rbd du命令显示实际的使用（容量）信息（当object-map处于enabled状态时该操作很快）
		- 对object-map特性的稳定性做了许多改善。
		- object-map和exclusive-lock特性可动态的开启或者关闭。
		- 现在你可以依据个体镜像(individual image)来存储用户的元数据并设置持久的librbd选项。
		- 新的深度扁平（deep-flatten）特性允许对一个克隆及其所有的快照进行扁平化处理（在此之前快照不能被扁平化。）
		- 更快的export-diff命令（使用了aio）。另外新增了fast-diff特性。
		- 可通过单位后缀指定-size参数大小（如--size 64G）。
		- 有一个新的rbd status命令，现在可通过该命令来显示谁打开映射了某个镜像。
- #####*CephFS*
- 现已可对快照进行重命名操作。
 - 在管理、诊断、检查和修复工具上已作出了持续的改善。
 - 由未使用inode引发的client端cache状态的缓存和撤回问题已经得到极大的改善。
 - 32位主机上，ceph-fuse客户端表现得更好。

## 发行版的兼容性 ##

我们已决定放弃对多个旧的Linux发行版的支持，这样一来我们就可以转用更新的编译器工具链（如C++11）。 尽管通过安装向后兼容的开发工具来在旧版本中生成Ceph还是可能的，但我们不会在ceph.com上为这些发行版生成并发布release版本的安装包。

我们现在为以下的版本构建安装包：

    - CentOS 7或后续版本。我们已放弃对CentOS 6的支持（以及RHEL 6其它的衍生系列，如Scientific Linux 6）。
    - Debian Jessie 8.x或后续版本。 Debian Wheezy 7.x的g++对C++11的支持并不完整（而且没有systemd）。
- Ubuntu Trusty 14.04或后续版本。 Ubuntu Precise 12.04已不再被支持。
- Fedora 22或后续版本。

## 从Firefly更新至Jewel ##

我们不推荐从Firefly v0.80.z直接更新。虽然直接更新是可能的，但会存在停机时间。我们推荐先将集群更新至Hammer v0.94.6或之后的v0.94.z发行版，只有这样之后才能做到线上更新至Jewel 10.2.z(见下文)。
若需线下直接从Firefly更新至Jewel，那么在任一Jewel OSD允许启动之前，必须停止所有的Firefly OSD并将其标记为down状态。此操作受Jewel监控器（Jewel monitor）的限制，因此，需使用如下的升级步骤：
    1.在monitor主机上更新Ceph
    2.重启所有的ceph-mon守护进程
    3.在所有的OSD主机上更新Ceph
    4.停止所有的ceph-osd守护进程
    5.将所有的OSD标记为down，所用的执行命令类似于：ceph osd down seq 0 1000
    6.启动所有的ceph-osd守护进程
    7.升级并重启余下的守护进程(ceph-mds, radosgw)

## 从Hammer更新至Jewel ##

集群中所有节点必须先升级到Hammer v0.94.4或之后的v0.94.z发行版；只有这样才能够升级到Jewel 10.2.z。对于支持systemd(CentOS 7, Fedora, Debian Jessie 8.x, OpenSUSE)的所有Linux发行版，Ceph守护进程现在使用原生的systemd文件而不是遗留的sysvinit脚本来进行管理。比如：

	systemctl start ceph.target   				# start all daemons

	systemctl status ceph-osd@12   			# check status of osd 12

- 主流的发行版中，Ubuntu trusty 14.04还未使用systemd。(下一个Ubuntu LTS，16.04，将会使用systemd替换upstart)。

现在，默认情况下，Ceph守护进程将以ceph用户和组的身份运行。 在Fedora和Debian（以及其他诸如RHELCentOS和Ubuntu等衍生发行版）中ceph用户被赋予了一个静态UID。在SUSE中ceph用户会在其创建的时候被动态的赋予一个UID。
如果你的系统中已有ceph用户，那么升级安装包的过程中将会出现问题。我们建议在升级前，先移除或重命名已有的ceph用户或ceph组。

升级过程中，管理员有两个选择：

	1. 将如下行添加至所有主机的ceph.conf中：

		setuser match path = varlibceph$type$cluster-$id

		这将使得Ceph的守护进程以root身份运行（即：未放弃特权及切换至ceph用户），如果Ceph守护进程的数据目录的所有者仍是root。 新部署的守护进程将以ceph用户来创建数据目录，并以非特权运行，但升级的守护进程仍以root身份运行。

	2. 升级过程中修复数据的所有权。这是我们所偏好的选择，但这需要做更多工作而且耗时。
对每个主机，需做如下步骤的操作：

	1. 升级ceph安装包。这就创建了ceph用户和组。如：

		ceph-deploy install --stable Jewel HOST

	2. 停止守护进程

		service ceph stop           # fedora, centos, rhel, debian

		stop ceph-all               # ubuntu

	3. 修复所有权

		chown -R cephceph varlibceph

	4. 重启守护进程

		start ceph-all                # ubuntu

		systemctl start ceph.target   # debian, centos, fedora, rhel

	可选的，相同的过程可以用单一的守护进程类型来完成，比如：只停止监控器并改变varlibcephmon的所有者。

- 处于实验阶段的KeyValueStore OSD后端所采用的磁盘格式(on-disk format)已发生了变化。如果你所要升级的测试集群中用到了它，那么你需要移除所有用到该后端的OSD。

- 与集群满一样，当达到存储池的配额时，librados操作将会无限期阻塞。（而之前的版本中会返回-ENOSPC）。默认状态下，当一个集群或pool满了，就会发生阻塞。如果你的librados应用能优雅地处理ENOSPC或EDQUOT等错误，那么你可以通过使用lirados中新的OPERATION_FULL_TRY标记来获取错误返回码。

- librbd的rbd_aio_read和Imageaio_read API方法在成功时将不再返回已读的字节数，而是在成功时返回0，失败时返回一个负值。

- ‘ceph scrub’, ‘ceph compact’ 和 ‘ceph sync force’ 已弃用，取而代之的是 ‘ceph mon scrub’, ‘ceph mon compact’ 和 ‘ceph mon sync force’。

- ‘ceph mon_metadata’ 现在应写成 ‘ceph mon metadata’。没有必要弃用这个命令（自引入以来一直存在）。

-  osdmaptool命令的–dump-json已由–dump json替换。

-  pg ls-by-{pool,primary,osd} 和 pg ls 这两命令中的recovery参数改为了recovering, 用来包含所列出的pg中正处于recovering状态的pg。

## 从INFERNALIS更新至Jewel ##

-  在Infernalis版本上没有重要的兼容性改变。仅升级每个主机上的守护进程并重启所以守护进程就搞定了。
-  RBD CLI在执行创建、导入、克隆操作时不再接受已弃用的“-image-features”参数。用“-image-feature”来代替。
-  RBD传统镜像格式（version 1）在Jewel版本中被弃用了。尝试新建一个version 1 RBD将会得到警告。Ceph的后续版本中将会移除对version 1 RBD的支持。
-  “send_pg_creates”和”map_pg_creates” CLI 命令已经完全不在提供支持。
-  添加了一个新的配置选项 ”mon_election_timeout” ，用来指定monitor选举过程的最大等待时长。
-  使用Firefly（0.80）或者更老版本创建的CephFS文件系统在更新到Jewel版本后，必须使用新的 “cephfs-data-scan tmap_upgrade” 命令。更多信息请参见CephFS文档中的 “Upgrading“。
-  移除了”ceph mds setmap”命令。
-  默认为新镜像提供的RBD镜像（RBD image）功能已被升级来提供以下功能：exclusive lock, object map, fast-diff和 deep-flatten。RBD内核驱动或者更老版本的RBD客户端现在还不支持这些功能。这些功能可以通过RBD CLI在per-image上禁用掉或者可以通过在Ceph配置文件中添加如下客户端内容，将这些默认功能升级到pre-jewel版本：
rbd default features = 1。
-  RBD 传统镜像格式（version 1）在Jewel版本中已被弃用。
-  在升级后，用户要设置”sortbitwise“标志来启用新的内部对象排序规则。
- *ceph osd set sortbitwise* 这个标志对于新的对象枚举API和新的像BlueStore一样的后端是非常重要的。
