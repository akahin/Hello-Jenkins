
`Manage Jenkins`下的:
`Reload Configuration from Disk`
Jenkins用xml文件存储所有它自己的以及构建作业的配置细节,并放在Jenkins的主目录,同样在这个目录下存储了所有的构建历史数据.
如果将将构建作业从一个Jenkins实例迁移到另一个Jenkins实例,则需要添加或者删除对应的构建作业目录到Jenkins的builds目录.这时可以使用这个选项.
`Script Console`
Executes arbitrary script for administration/trouble-shooting/diagnostics.
在服务器上执行groovy脚本,它对于高级故障跟踪是非常有用的:高级故障跟踪技术取决于Jenkins内部架构背景知识,主要对插件开发者有用


Discard all the loaded data in memory and reload everything from file system. Useful when you modified config files directly on disk.
