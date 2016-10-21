### Rake Command Line Usage

rake命令行使用方式

    % rake [options ...]  [VAR=VALUE ...]  [targets ...]

选项:

  *name=value*

设置环境变量name对应的值,在rake命令执行期间使用,可以通过ENV访问

  *--all (-A)*

结合-T和-D选项一起使用 强制显示所有任务,即使没有注释的任务

*--backtrace{=output} (-n)*

激活完整的堆栈跟踪检查(类似–trace,但是没有任务的跟踪信息),输出参数是可选的, 但是如果指定就
控制了bracktrace的输出位置, 如果output是stdout,bracktrace直接输出到标准输出,如果是stderr,
或者没指定 直接输出到标准错误

*--comments*

与-W组合使用,强制输出包含注释的选项 与--all相反

*--describe pattern (-D)*

描述任务(匹配选项PATTERN)

*--dry-run (-n)*

运行演习,打印执行和被调用的任务,但是不是真正的执行任何一个actions


*--execute code(-e)*

执行一些ruby代码

_--execute-print code (-p)_

执行一些ruby代码,然后打印结果

_--execute-continue code (-E)_

执行一些ruby代码然后继续执行常规任务

_--help (-H)_

打印帮助文档

_--jobs number (-j)_

设置允许的最大并发线程数,rake按照需要分配线程,如果忽略,rake会判断cpu的数量,然后加上4
并发线程被用来执行多任务并行处理的先决条件,-m选项表示将所有任务都进行多任务并行执行
例子:

    (no -j)   : Allow up to (# of CPUs + 4) number of threads
    --jobs    : Allow unlimited number of threads
    --jobs=1  : Allow only one thread (the main thread)
    --jobs=16 : Allow up to 16 concurrent threads

_--job-stats level_

显示运行完成的Job统计,默认情况,将会显示申请的当前活动线程数量(来自 -j 选项)和在任意时刻
运行的线程最大数量

如果选项level是history, 稍后一组完整的task运行历史信息将会被打印

_--libdir directory (-I)_

添加目录到用于require的目录列表

_--multitask (-m)_

所有任务作为并行任务

_--nosearch (-N)_

不在父目录里搜寻rakefile文件

_--prereqs (-P)_

显示任务列表和他们的先决条件

_--quiet (-q)_

不显示来自FileUtils的命令

_--rakefile filename (-f)_

提供文件名字作为rakefile文件,默认rakefile名字是*rakefile*和Rakefile(railefile优先),如果
raikefile在当前目录没有被找到,rake将会查找查找父目录,
rakefile所在的目录被当做actions运行的当前目录

--rakelibdir rakelibdir (-R)

默认导入RAKELIBDIR目录中的任意以.rake后缀的文件 (默认是'rakelib'目录)

--require name (-r)

在执行rakefile之前,引入需要文件

--rules

追踪rules方案

--silent(-s)

类似 --quiet 但是也压制了目录中的通告

--suppress-backtrace pattern

正则表达式模式将会移除堆栈输出,注意这个-bracktrace选项是完整的堆栈信息,没有Lines压制

--system (-g)

使用系统范围(全局)rakefiles. 项目自身rakefile被忽略,默认情况,项目中没有rakefile文件,系统的
rakefile被使用, 在unix like 系统中,系统的rakefile文件在$HOME/.rake目录中,window系统中在
$APPDATA/Rake目录里

_--no-system (-G)_

使用项目级别rakefile,忽略系统级的rakefile文件

_--tasks pattern (-T)_

显示主要任务和他们的注释. 注释使用desc命令定义,如果给了一个pattern,只有匹配pattern的task被显示

_--trace{=output} (-t)_

打开(调用/执行)追踪,同时激活错误堆栈信息.output参数是可选的,如果指定了堆栈输出设置,如果输出是stdout
堆栈信息直接通过标准输出接口输出.,如果是stderr或没设置,直接通过标准错误输出.

_--verbose (-v)_

打印系统命令到标准输出

_--where pattern (-W)_

打印匹配pattern的任务的文件和所在行号,默认情况打印所有任务,不仅仅是有描述的任务.

_--no-deprecation-warnings (-X)_

不显示描述警告

此外,任何命令行选项以var=value形式添加到环境变量ENV中,
