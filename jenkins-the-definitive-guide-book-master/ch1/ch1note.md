
##### Jenkins的配置
安装git
git Windows下需要安装git-for-windows, 安装时勾选将可执行文件路径添加到PATH,这样安装好后Windows环境变量Path中会出现`C:\Program Files\Git\cmd`
安装完成后,重启Jenkins:
http://localhost:8080/restart(关闭Jenkins:在访问jenkins服务器的网址url地址后加上exit,http://localhost:8080/exit,重新加载配置信息:http://localhost:8080/reload)
Jenkins中配置git
 在`全局工具配置`中Git下的`Path to Git executable`中设置git路径
git.exe(需要重启Jenkins)或C:\Program Files\Git\bin\git.exe

如果用cygwin,需要将cygwin的/bin在Windows下的路径`C:\cygwin64\bin`添加到Path里


https://stackoverflow.com/questions/15135771/hudson-on-windows-error-java-io-ioexception-cannot-run-program-sh
如果选择出现提示:
test] $ sh -xe C:\Windows\TEMP\jenkins905503213843797633.sh
The system cannot find the file specified
FATAL: 命令执行失败
java.io.IOException: CreateProcess error=2, 系统找不到指定的文件。
    at java.lang.ProcessImpl.create(Native Method)
    at java.lang.ProcessImpl.<init>(Unknown Source)
    at java.lang.ProcessImpl.start(Unknown Source)
Caused: java.io.IOException: Cannot run program "sh" (in directory "C:\Program Files (x86)\Jenkins\workspace\test"): CreateProcess error=2, 系统找不到指定的文件。
https://stackoverflow.com/questions/15135771/hudson-on-windows-error-java-io-ioexception-cannot-run-program-sh
说明This happens if you have specified your Windows command as "Execute shell" rather than "Execute Windows batch command".

安装maven:
http://maven.apache.org/download.cgi下载apache-maven-3.5.4-bin.zip
1 Ensure JAVA_HOME environment variable is set and points to your JDK installation
2 将apache-maven-3.5.4解压到C:\Program Files\apache-maven-3.5.4
3 Add the bin directory of the created directory apache-maven-3.5.4 to the PATH environment variable
4 Confirm with mvn -v in a new shell.
Windows Tips:
Adding to PATH: Add the unpacked distribution’s bin directory to your **user** PATH environment variable by opening up the system properties (WinKey + Pause), selecting the “Advanced” tab, and the “Environment Variables” button, then adding or selecting the PATH variable in the user variables with the value C:\Program Files\apache-maven-3.5.4\bin. The same dialog can be used to set JAVA_HOME to the location of your JDK, e.g. C:\Program Files\Java\jdk1.7.0_51
Check environment variable value,e.g.
echo %JAVA_HOME% 
C:\Program Files\Java\jdk1.7.0_51
Jenkins中配置maven:
不需要在环境变量中新增MAVEN_HOME
在`全局工具配置`中Maven下的`MAVEN_HOME`中设置maven的安装文件夹`C:\Program Files\apache-maven-3.5.4`,也就是之前解压到的路径.在'Name'中填一个自定义的名字
之后在设置每个创建的任务时,如果在`Build`下选择`Invoke top-level Maven target`,则`Maven 版本`要设置为之前设置的自定义名称

如果出现以下错误:
[game-of-life-default] $ cmd.exe /C "mvn clean package && exit %%ERRORLEVEL%%"
'mvn' 不是内部或外部命令，也不是可运行的程序
或批处理文件。
Build step '调用顶层 Maven 目标' marked build as failure
Finished: FAILURE
说明`Maven 版本`没有设置为之前设置的自定义名称


对某些类型的项目,红色表示编译错误导致的构建错误,黄色表示其他类型的构建失败(如单元测试失败或代码覆盖不足)

推荐的关闭Jenkins的方法是选择`系统管理`中的最后一项`prepare for shutdown`



Jenkins安装目录的结构
目录 | 描述
---|---
jobs | 包含Jenkins管理的构建作业的细节,以及这些构建所输出的产物以及数据
plugin | 包含所有已经安装的插件.除了一些核心操作外,插件不存储为可执行的Jenkins文件,也不存储在扩展的Web应用程序目录.这意味着可以更新Jenkins而不需要重新安装插件
updates | Jenkins使用的一个内部目录,用来存放可用的插件更新
userContent | 可以使用这个目录存放自己为Jenkins服务器定制化的一些内容.可以在http://myserver/hudson/userContent(如果在一个应用服务器上运行Jenkins)或http://myserver/userContent(如果运行在单机模式)访问这个目录的文件
users | 如果使用的是Jenkins的本地用户数据库,用户账户信息会被存放在这个目录下
war | 包含了扩展的Web应用程序.当你以一个单机应用程序的形式运行Jenkins时,它会把web应用解压到这个目录

jobs下每个构建作业一个目录,目录内的config.xml包含了此构建作业的所有配置细节.
一个成功的构建指没有任何编译错误,一个稳定的构建就是一个成功的构建(不论为这个构建配置了怎样的质量标准,如单元测试,代码覆盖率等).

workspace目录是Jenkins为项目进行构建的地方.这个目录会被一个构建任务重复使用.
每个项目只能有一个workspace

builds目录包含为此作业所执行的构建历史
