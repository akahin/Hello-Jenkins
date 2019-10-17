
##### 参数化构建作业
parameterized build插件可以为构建作业配置参数,可以当构建作业启动时由用户输入,或者从其他构建作业中传入.
正确的分析和处理参数值是构建脚本的工作.Jenkins只是为用户输入参数值和将这些参数写入构建脚本中提供了一个用户界面.

###### 为构建适配参数化构建脚本
可以在构建脚本里使用这些环节变量.例如在一个Ant或maven构建中,可以使用特定的env属性来获取当前环境变量:
```
<target name="printversion">
  <property environment="env" />
  <echo message="${env.VERSION}" />
<target />
```
另一个方法是将参数作为属性值传递到构建脚本中.下面是一个有关maven pom文件比较复杂的例子
```
...
<dependencies>
 <depengency>
  <groupId>com.wakaleo.gameoflife</groupId>
  <artifactId>gameoflife-web</artifactId>
  <type>war</type>
  <version>${target.version}</version>
 </depengency>
</dependencies>
<properties>
 <target.version>RELEASE</target/version>
 ...
</properties>
```

Boolean参数以复选框形式显示.

运行时参数:
Defines a run parameter, where users can pick a single run of a certain project. The absolute url of this run will be exposed as an environment variable, or through variable substitution in some other parts of the configuration. In the build this can be used to query Jenkins for further information.
运行时参数让你为给定的构建作业选择一个特定运行(或构建).用户从列表中选择构建运行数值.相应运行的构建url存储于特定的参数中.
URL(例如http://jenkins.myorg.com/job/game-of-life/187/)可以用来获取信息或者是获取构建运行的构建产物.例如,可以从之前的构建中获取jar或war归档文件,并在一个单独的构建作业中进一步测试特定的二进制文件.
例如访问一个多模块maven项目中以前构建的war文件,url会类似于这样:
http://buildserver/job/game-of-life/197/artifact/gameoflife-web/target/gameoflife.war
所以可以用以下表达式访问WAR文件:
${RELEASE_BUILD}gameoflife-web/target/gameoflife.war

文件参数:允许上传一个文件到构建作业中

######　远程启动参数化构建作业
参数化构建作业url地址形式说明
http://jenkins.acme.org/job/myjob/buildWithParameters?PARAMETER=true
所以可以这样触发构建:
http://jenkins.acme.org/job/myjob/buildWithParameters?VERSION=1.2.3
当使用一个URL来启动构建作业时,需要注意参数名字是大小写敏感的,并且值需要转义(就像任何的http参数一样).如果正使用运行时参数,则需要提供构建作业的名字和执行序列号,而不只是执行序列号.


###### 参数化触发
将当前构建作业的参数传递到新构建作业中是非常有用的.
在"post-build Actions"部分使用常规的"build other projects"不能触发参数化构建,可以用"Jenkins parameterized trigger"插件来完成
安装好这个插件后,在作业配置界面将会看到"triggering parameterized builds on other projects".这里有多种启动其他构建作业的方法.
触发的构建作业必须也是参数化构建作业,并且也必须在配置中添加这个参数.


###### 多重结构的构建作业
多重结构的构建作业是Jenkins中非常强大的功能.一个多重结构的构建作业可以被理解为可以自动地运行所有可能的参数组合的一个参数化构建作业.这对于测试是非常有用的.
要创建多重构建作业,选择"Mulit-configuration project"
相比其他构建作业,多重构建作业多了一个矩阵配置(configuration matrix).在这里可以定义不同的配置选项轴,包括运行构建作业的各个从节点,各jdk版本,或用于构建的自定义属性.
例如对一个构架,可能想在不同的数据库和os上测试.因此可以定义一个轴,轴定义了运行构建所需的各类os的各从节点机器.其他轴定义了所有数据库.Jenkins将在每个可能的数据库和os上执行构建作业.

###### 配置从节点


