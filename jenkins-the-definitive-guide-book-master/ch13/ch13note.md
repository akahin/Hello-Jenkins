##### Jenkins的维护

###### 监控磁盘空间
使用"清除历史构建"的问题在于可能使构建信息变的不精确.
一般情况下,占用空间最多的是构建产物如jar文件,war文件等.构建历史主要存储在xml格式的日志文件里,并不会占用太多空间.单击advanced按钮,Jenkins允许仅清除构建版本文件,保留构建数据.

######　使用Disk Usage插件
Disk Usage插件是Jenkins管理工具中最重要的插件之一.
如果想进一步了解磁盘空间的使用速度,可以在系统配置界面勾选"show disk usage trend graph on the project page"

###### 磁盘使用及Jenkins Maven项目类型
如果使用Jenkins maven构建作业,则需要了解更多的细节信息.在Jenkins中,maven构建作业会默认自动备份构建产物,这些构建产物快照会大量占用磁盘空间.建立多模块项目会进一步恶化此问题
事实上,如果需要备份maven产物快照,最好将它们直接部署到本地的maven仓库管理器.可以配置nexus pro来实现上述目的,并配置artifactory来删除旧的构建产物快照.

######　监控服务器负载
Jenkins提供内置的服务器活动监控功能.在manage Jenkins界面,单击"load statistics",将看到主节点服务器负载随时间的变化图.该图将跟踪3个指标:
总执行数量(Number of online executors,蓝线), 繁忙的执行数量(Number of busy executors,红线), 队列长度(Queue length, 灰线)
Load statistics keep track of four key metrics of resource utilization:
**Number of online executors**
指主从节点上所有执行器的数量.当从节点接入或者离开集群时,这个值将发生变化.这个指标可以有效地反映从节点的动态配置情况.
For a computer: if the computer is online then this is the number of executors that the computer has; if the computer is offline then this is zero. 
For a label: this is the sum of all executors across all online computers in this label. 
For the entire Jenkins: this is the sum of all executors across all online computers in this Jenkins installation. 
Other than configuration changes, this value can also change when agents go offline.
**Number of busy executors**
被占用执行构建的执行器数量.需要确保有足够的执行器来应对峰值时刻的构建作业需求.如果所有的执行器都被构建作业永久性占用了,则需要添加更多的执行器或者从节点进入集群.
This line tracks the number of executors (among the executors counted above) that are carrying out builds. The ratio of this to the number of online executors gives you the resource utilization. If all your executors are busy for a prolonged period of time, consider adding more computers to your Jenkins cluster.
**Number of available executors**
等待执行的构建作业数量.如果所有执行器都被占用了,构建作业将被加入队列中进行等待.这个指标并未计入等待上游构建执行完成的项目,所以它是衡量何时能从额外容量获益的有效指标.
This line tracks the number of executors (among the online executors counted above) that are available to carry out builds. The ratio of this to the total number of executors gives you the resource availability. If none of your executors are available for a prolonged period of time, consider adding more computers to your Jenkins cluster.
**Queue length**
This is the number of jobs that are in the build queue, waiting for an available executor (of this computer, of this label, or in this Jenkins, respectively.) This doesn't include jobs that are in the quiet period, nor does it include jobs that are in the queue because earlier builds are still in progress. If this line ever goes above 0, that means your Jenkins will run more builds by adding more computers.
Note: The number of busy executors and the number of available executors need not necessarily be equal to the number of online executors as executors can be suspended from accepting builds and thus be neither busy nor available.

The graph is an exponential moving average of periodically collected data values. 3 timespans are updated every 10 seconds, 1 minute and 1 hour respectively.

在从节点界面,使用load statistics按钮可以得到类似的从节点图表.
另一个选择是安装monitoring插件,该插件使用JavaMelody生成构建服务器状态相关的完整HTML报告.其中包括CPU和系统负载,平均相应时间和内存使用.一旦安装了这个插件,就能在manage Jenkins界面使用
"monitoring of Jenkins/Jenkins master"或"Jenkins/Jenkins nodes"菜单访问JavaMelody生成的图表.

#####备份配置
###### Jenkins备份基础
最简单的备份手段是周期性备份JENKINS_HOME目录,在Jenkins运行过程中仍然可以进行备份操作--并不需要关闭服务器.这个方法缺点是JENKINS_HOME目录的文件太多,可以不备份以下数据,这些数据可以有Jenkins轻松在线重建:
$JENKINS_HOME/war
$JENKINS_HOME/cache
$JENKINS_HOME/tools
$JENKINS_HOME下的job/构建名/workspace(项目恢复过程中如果Jenkins发现workspace不见了会自动生成该目录).

如何测试备份出的文件:把备份提取到一个临时目录,然后启动一个Jenkins实例并指向这个目录,然后启动一个Jenkins实例并指向这个目录.例如假设我们已经把备份提取到一个叫做/tmp/jenkins-backup的临时目录下,为了测试这个备份,首先要设置JENKINS_HOME目录为这个临时目录:
export JENKINS_HOME=/tmp/jenkins-backup
然后在另一个不同端口上启动jenkins,并测试执行情况:
java -jar jenkins.war --httpPort=8888
现在Jenkins将在8888端口启动,可以检测它是否正常运行.

###### 使用"Backup"插件
该插件允许你配置及运行构建作业配置或构建历史的备份.在起始界面,可以设定备份内容,可以指定只备份xml配置文件,或者同时备份配置文件和构建历史.你也可以选择备份(或不备份)自动生成的maven构建产物(在许多构建流程中,这些构建产物会保存在本地企业仓库管理器中).也可以指定备份作业的工作空间(一般情况下,就像之前所讨论的,无须备份它)以及生成的所有指纹.

###### 更为轻量的自动备份
如果只想备份构建的配置信息,则更适合使用"Thin Backup"插件,它允许你对配置信息进行全面或增量式的备份.此插件不会保存构建历史和构建产物,因此备份更快,也无须关闭Jenkins服务器.
和backup插件类似,可在Jenkins系统配置界面找到该插件的按钮.

######　构建作业归档
另一种解决硬盘空间问题的方法是删除或归档那些不再使用的项目.在你需要访问归档项目数据或构建产物时可以轻易地解压缩相应归档的项目.
归档项目很简单:只需将构建项目目录移出作业项目job/.通常可将其压缩为tar或zip
如果想归档的项目没有运行,就可以安全删除项目目录并移动归档文件.
最后可在Jenkins界面硬盘加载配置,归档的项目将从仪表盘消失.

###### 构建迁移
可通过在Jenkins实例间复制或移动构建作业目录来拷贝构建作业.项目作业目录是自给自足的--它包含全项目的配置信息和所有构建历史.将构建作业目录直接复制到运行中的Jenkins实例也是安全的.如要删除原有服务器上的构建作业目录,需要先关闭服务器上的Jenkins服务.甚至无需重启新的jenkins实例就能查看导入的结果--可以在manage Jenkins界面单击reload configuration from disk.

如果想把作业迁移到一个全新的Jenkins配置,请记得安装或迁移原有服务器的插件.在plugins目录下,可以简单地复制所有文件到新实例对应目录下.








