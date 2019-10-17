
##### Jenkins分布式构建架构
Jenkins使用主/从架构来挂历分布式构建.主要Jenkins服务器(直到目前我们一直使用的)是主节点.简单的说,主节点的工作是处理调度构建作业,把构建分发到从节点来实际执行,监视从节点(必要时可能让它们上线或者离线),并且
记录和发布构建产物.即使在一个分布式架构中,Jenkins主实例也可以直接执行构建作业.
从节点安装被告知的工作,这涉及执行主节点分派的构建作业.可以配置一个项目总是在特定的从节点运行,或者在某个特定类型的从节点运行,抑或让Jenkins挑选下一个可用的从节点.
从节点在远程机器上运行小的Java可执行文件,以及监听来自Jenkins主实例的请求.从节点可能(而且通常)在多种os上运行.从实例可以以许多不同的方式来启动.这取决于os和网络结构.一旦从实例在运行,它就通过tcp/ip连接与主实例进行通信.

###### Jenkins主/从策略
用Jenkins可以配置许多不同的方式来设置分布式构建,这取决于os和网络架构.事实是在所有情况下,构建作业运行在从节点,而且从节点怎么被管理对于终端用户是透明的:构建结果和构建产物最后总是会在主服务器上.

###### 主节点使用ssh启动从节点代理
如果在Unix环境工作,最方便的方式是从ssh启动Jenkins从节点.Jenkins有自己的ssh客户端.

创建后,需要向Jenkins提供关于从节点的更多细节,
其中Usage字段配置Jenkins使用该节点的程度:
"Use this node as much as possible":In this mode, Jenkins uses this node freely. Whenever there is a build that can be done by using this node, Jenkins will use it.
只要改节点可用,就告诉Jenkins自由使用这个从节点运行任何构建作业.这是目前为止最常用的一个,而且通常是你想要的.

Launch method字段:Jenkins如何启动节点
Launch slave agents via SSH:


Launch agent via Java Web Start:

Availability字段:Controls when Jenkins starts and stops this agent.
**Keep this agent online as much as possible**
In this mode, Jenkins will keep this agent online as much as possible.
If the agent goes offline, e.g. due to a temporary network failure, Jenkins will periodically attempt to restart it.
**Take this agent online and offline at specific times**
In this mode, Jenkins will bring this agent online at the scheduled time(s), remaining online for a specified amount of time.
If the agent goes offline while it is scheduled to be online, Jenkins will periodically attempt to restart it.
After this agent has been online for the number of minutes specified in the Scheduled Uptime field, it will be taken offline. 
If Keep online while builds are running is checked, and the agent is scheduled to be taken offline, Jenkins will wait for any any builds that may be in progress to complete.
**Take this agent online when in demand, and offline when idle**
如果有定期的峰值和间歇的构建活动,则这是有用的,因为未使用的从节点可以离线以节省系统资源用于其他任务,需要时该节点再重新上线.
In this mode, Jenkins will bring this agent online if there is demand, i.e. there are queued builds which meet the following criteria:
They have been in the queue for at least the specified In demand delay time period
They can be executed by this agent (e.g. have a matching label expression)
This agent will be taken offline if:
There are no active builds running on this agent
This agent has been idle for at least the specified Idle delay time period

Jenkins还需要知道在从节点哪里能找到构建作业需要的构建工具.如果已经配置了自动安装构建工具,通常不需要为从节点机器做额外的配置,Jenkins会根据需要下载并安装工具.另一方面,如果构建工具安装在本地从节点机器,则你需要告诉Jenkins在哪里可以找到它们.可以勾选Tool Location复选框.


#####　把构建作业与一个或一组从节点关联
Label Expression：
If you want to always run this project on a specific node/slave, just specify its name. This works well when you have a small number of nodes.
As the size of the cluster grows, it becomes useful not to tie projects to specific slaves, as it hurts resource utilization when slaves may come and go. For such situation, assign labels to slaves to classify their capabilities and characteristics, and specify a boolean expression over those labels to decide where to run.

Valid Operators
The following operators are supported, in the order of precedence:
(expr)
parenthesis
!expr
negation
expr&&expr
and
expr||expr
or
a -> b
隐含运算符定义逻辑约束的形式:"如果A是真的,那么B也必须是真的".例如假定有个在任何Linux发行版运行的构建,但如果它在Windows上运行则必须是win7,可以这样表达这个约束:windows -> "windows 7"
"implies" operator. Equivalent to !a|b. For example, windows->x64 could be thought of as "if run on a Windows slave, that slave must be 64bit." It still allows Jenkins to run this build on linux.
a <-> b
这个运算符定义更严格的约束形式"如果A为真,B也必须真.如果A假,B也必须假"
"if and only if" operator. Equivalent to a&&b || !a&&!b. For example, windows<->sfbay could be thought of as "if run on a Windows slave, that slave must be in the SF bay area, but if not on Windows, it must not be in the bay area."
All operators are left-associative (i.e., a->b->c <-> (a->b)->c ) An expression can contain whitespace for better readability, and it'll be ignored.
如果机器名称中含有空格,则需要用双引号括起来
Label names or slave names can be quoted if they contain unsafe characters. For example, "jenkins-solaris (Solaris)" || "Windows 2008"


###### 节点监控
如果Jenkins认为节点不能安全完成一个构建,就将此节点脱机.可以在"manage node"精确调控Jenkins监控的内容.
Jenkins以几种不同方式监控从节点:
响应时间:过于缓慢的响应时间可能是网络问题,或者从节点已经关机.
Jenkins还监视提供给Jenkins用户从节点可用的磁盘空间,临时目录空间以及swap空间,因为构建作业可能会消耗大量磁盘.它还紧密监视系统时钟,如果时钟不能正确同步,有时会发生奇怪的错误.

