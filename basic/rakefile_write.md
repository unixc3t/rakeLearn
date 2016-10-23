[原文链接](https://ruby.github.io/rake/doc/rakefile_rdoc.html)
### Rakefile Format

首先,没有指定的格式对于rakefile文件,一个rakefile文件包含执行的ruby代码,任何合法的ruby脚本
都允许在rakefile文件中

现在.我们知道了.没有指定的语法在rakefile文件中,这有一些约定在rakefile文件中使用,
与典型的ruby程序有一点不同,因为rakefile用来指定taks和actions的,这种风格专门为rakefile
设计的.

### Tasks

tasks是rakefile工作的最小单元.task有一个名字,一组先决条件和一组actions
(名字和先决条件通常是符号或者字符串,actions通常是代码块)

### Simple Tasks

一个task声明使用task方法,task方法接受一个单独的参数,就是task的名字

    task :name

### Task with Prerequisites

任何前置条件通过一个列表指定(封闭方括号)跟着是名字和箭头给定

    task name: [:prereq1, :prereq2]

注意 :虽然语法有一点古怪,但是是合法语法. 我们构造一个hash,key是:name,值是一个先决条件列表.
如下

    hash = Hash.new
    hash[:name] = [:prereq1, :prereq2]
    task(hash)

你也可以使用字符串作为task名字和先决条件,如下

    task 'name' => %w[prereq1 prereq2]

或者这样

    task name: %w[prereq1 prereq2]

我们更喜欢这个风格用于使用先决条件的有规律的任务,在后面的文档中.使用一个字符串数组作为先决条件
意味着你将很少的改变,如果你需要移动任务到命名空间或者其他重构

### Tasks和Actions


actions通过给task方法传递一个代码块,任何ruby代码可以在代码块中,代码快通过块参数引用task对象

        task name: [:prereq1, :prereq2] do |t|
            # actions (may reference t)
        end

### Multiple Definitions

一个任务可能不止一次被指定,每种指定加入先决条件和actions到以存在的定义里.
这允许rakefile部分指定actions,不同的rakefile(单独生成)指定依赖

    task :name
    task name: :prereq1
    task name: %w[prereq2]
    task :name do |t|
        # actions
    end

### File Tasks

一些任务被用来设计创建一个文件从从一个或多个其他文件. task生成这些文件,如果存在就跳过
,文件任务用于指定文件创建任务。

文件任务声明使用file方法(替代task方法),此外文件任务命名通常使用字符串而不是符号

下面的文件任务创建了一个可执行的程序叫做prog,给予2个文件对象叫做a.o 和b.o 任务是创建a.o和b.o不显示

    file "prog" => ["a.o", "b.o"] do |t|
      sh "cc -o #{t.name} #{t.prerequisites.join(' ')}"
    end

### Director Tasks

通常根据需求需要创建目录,目录快捷方法就是创建一个文件任务来创建目录,例如下面描述

    directory "testdata/examples/doc"

等价于

    file "testdata" do |t| mkdir t.name end
    file "testdata/examples" => ["testdata"] do |t| mkdir t.name end
    file "testdata/examples/doc" => ["testdata/examples"] do |t| mkdir t.name end

这个目录方法没有接收任何前置条件和actions,但是都可以稍后添加

    directory "testdata"
    file "testdata" => ["otherdata"]
    file "testdata" do
      cp Dir["standard_data/*.data"], "testdata"
    end
    
Tasks和并行先决条件
 
rake允许并行执行先决条件使用下面语法

    multitask copy_files: %w[copy_src copy_doc copy_bin] do
      puts "All Copies Complete"
    end

在这个例子里,copyfiles是一个rake任务,actions要等先决条件执行完后执行,
最大的不同就是先决条件(copy_src,copy_bin,copy_doc)被并行执行,每个先决条件运行在自己的
线程里,

### Secondary Prerequisites

如果并行任务的主要先决条件有辅助先决条件,所有的主要/并行先决条件都要等待道辅助先决条件运行完
例如copy_xxx任务有下面的辅助先决条件
    
    task copy_src: :prep_for_copy
    task copy_bin: :prep_for_copy
    task copy_doc: :prep_for_copy
    
prep_for_copy任务在所有copy任务运行前执行,一旦prep_for_copy执行完, copy_src, copy_bin和copy_doc同时并行运行,
注意prep_for_copy仅仅运行一次,即使它被多个并行线程引用

### Thread Safety

rake内部数据结构是线程安全的对于多任务的并行执行,所以不需要用户做额外的同步对于rake来说,然而,
如果有用户数数据在不同的并行前置条件里分享,用户必须做好预防竞争条件

### Tasks with Arguments

在0.8.0之前版本的rake仅仅能够处理通过ENV hash传递的NAME=VALUE形式的命令行参数,许多人都要求一些简单形式的命令行参数,或许使用
"-"分割任务名字和参数值,问题就是没有简单的方式将命令行的参数位置与不同任务联系起来, 假设任务:a和:b期待一个命令行参数,那么第一个参数是传递给:a
还是传递给:b 

rake0.8.0版本解决了这个问题.通过直接传递需要的值给任务.例如下面,加入我们有一个release任务需要一个版本号.我们这样
    
    rake release[0.8.2]
    
0.8.2以字符串形式传递给:release任务,多个参数可以使用逗号分隔传递,如下

    rake name[john,doe]
    
这有一些忠告,rake的任务名字和他的参数需要作为一个整体的命令行参数传递给rake,这意味着没有空格,如果需要空格,以名字+参数的字符串形式
传递给rake

    rake "name[billy bob, smith]"

(引号规则在不同的系统和shell不一样,确保查看了系统文档)

### Tasks that Expect Parameters

参数仅仅用来提供给任务期望的值,为了处理命名参数,任务生命与之直接进行了扩展
例如,一个任务需要开头和最后的名字,可以这样定义

    task :name, [:first_name, :last_name]
    
第一个参数仍然是任务的名字(:name),接下来两个参数是参数的名字,在一个数组里

访问这两个参数的值,代码块定义是需要第二个参数,也就是除了t以外的第二个,例如下面

    task :name, [:first_name, :last_name] do |t, args|
      puts "First name is #{args.first_name}"
      puts "Last  name is #{args.last_name}"
    end

第一个参数是t,总是绑定当前的任务对象(task object),的哥参数是args是一个open struct结构,用来访问任务参数,额外的
命令行参数被忽略

如果你想指定参数的默认值,你可以使用with_default方法在任务代码快中,如下

    task :name, [:first_name, :last_name] do |t, args|
      args.with_defaults(:first_name => "John", :last_name => "Dough")
      puts "First name is #{args.first_name}"
      puts "Last  name is #{args.last_name}"
    end

Tasks that Expect Parameters and Have Prerequisites

对于有先决条件的的任务在使用参数上有一点不同,使用箭头指向先决条件,例如下面

    task :name, [:first_name, :last_name] => [:pre_name] do |t, args|
      args.with_defaults(:first_name => "John", :last_name => "Dough")
      puts "First name is #{args.first_name}"
      puts "Last  name is #{args.last_name}"
    end

### Tasks that take Variable-length Parameters

task需要处理一列值做外参数时,可以使用args变量的方法,这允许task可以循环变量值,这种方式也使用命名参数

    task :email, [:message] do |t, args|
      mail = Mail.new(args.message)
      recipients = args.extras
      recipients.each do |target|
        mail.send_to(target)
      end
    end

这里也有一个方便的to_a方法,返回所有参数按照给定的序列,


### Deprecated Task Parameters Format

用于声明任务参数的旧格式:忽略任务参数数组和使用:needs关键字引入依赖,这种格式仍然支持,但是不建议使用,在未来版本中将会删除

### Accessing Task Programmatically

有时以编程的方式操纵任务是很有用的, 查找一个任务使用Rake::Task.[]

### 编程任务的例子

下面的rakefile里定义了2个任务,:doit任务简单的打印出"DONE"信息, 这个:dont将查找doit并且移除他的所有先决条件和
actions

    task :doit do
      puts "DONE"
    end
    
    task :dont do
      Rake::Task[:doit].clear
    end
    
运行结果

    $ rake doit
    (in /Users/jim/working/git/rake/x)
    DONE
    $ rake dont doit   #移除后什么都没有了
    (in /Users/jim/working/git/rake/x)
    $

这种操纵task的编程能力,给予rake非常强大的元编程能力,使用时要谨慎
    
### Rules

当一个文件被作为先决条件时,但是没有一个为这个先决条件定义的任务,rake将会尝试去合成一个任务,通过查看规则列表来支持这个文件任务
假设我们现在尝试调用mycode.o任务(mycode.o也是一个先决条件),但是没有定义这个任务,但是rake有一套规则,例如下面

    rule '.o' => ['.c'] do |t|
      sh "cc #{t.source} -c -o #{t.name}"
    end

按照规则组合一个以.o结尾的任务,他必须有一个前置条件是以.c作为扩展的源文件,如果rake能够找到一个文件叫做"mycode.c"
将会自动的创建一个任务使用mycode.c文件编译构建mycode.o文件

如果一个mycode.c还不存在,就会同样循环的构建一个创造mycode.c的任务

当一个任务按照规则组合的时候,任务的source属性被设置成匹配的源文件(这里被设置成mycode.c),这允许我们使用actions编写rules
时引用源文件

### Advanced Rules

任何正则表达式可以作为规则模式,一个proc可以用来计算source文件的名字.这就允许复杂的模式和源文件

下面的例子等价于上面的例子
    
    rule( /\.o$/ => [
      proc {|task_name| task_name.sub(/\.[^.]+$/, '.c') }
    ]) do |t|
      sh "cc #{t.source} -c -o #{t.name}"
    end

注意 因为快速语法,所以第一个参数是正则表达式需要圆括号

下面的规则用于java文件
    
    rule '.class' => [
      proc { |tn| tn.sub(/\.class$/, '.java').sub(/^classes\//, 'src/') }
    ] do |t|
      java_compile(t.source, t.name)
    end

注意java_compile是一个假想方法用来调用java编译器

### importing Dependencies

任意ruby文件(包括其他rakefiles)可以使用标准ruby的require命令引入,所需文件中的规则和声明被添加到已经积累的定义中

因为这些文件在rake任务执行前被加载,读取文件必须当rake命令被调用时准备好, 这使得生成的依赖文件很难使用,到rake得到
依赖文件更新,再读取就太迟了.

这个import命令键入在主要rakefile读取后被指定加载的文件,但是在任意任务在命令行上被调用之前,此外,如果一个文件名字匹配了
一个明确任务,任务在读取文件之前被调用.这允许依赖文件被生成并且使用一个单独的命令调用

    require 'rake/loaders/makefile'
    
    file ".depends.mf" => [SRC_LIST] do |t|
      sh "makedepend -f- -- #{CFLAGS} -- #{t.prerequisites} > #{t.name}"
    end
    
    import ".depends.mf"
    
上面代码,如果.depends不存在,或者已经过时的源文件 ,一个新的depends文件使用makedepend生成在加载之前

### Comments

标准的ruby注释(以#开头)可以在任何合法的ruby源文件中使用,包括任务和rules注释,如果你希望使用-T开关查看任务描述
你需要使用desc命令描述这个任务

    desc "Create a distribution package"
    task package: %w[ ... ] do ... end

-T开关(-task也可以,如果你喜欢写出完整命令)将显示任务有描述的任务列表,如果你使用desc描述你的主要任务,
你有一个半自动方式生成你的rake文件摘要

    traken$ rake -T
    (in /home/.../rake)
    rake clean            # Remove any temporary products.
    rake clobber          # Remove any generated file.
    rake clobber_rdoc     # Remove rdoc products
    rake contrib_test     # Run tests for contrib_test
    rake default          # Default Task
    rake install          # Install the application
    rake lines            # Count lines in the main rake file
    rake rdoc             # Build the rdoc HTML Files
    rake rerdoc           # Force a rebuild of the RDOC files
    rake test             # Run tests
    rake testall          # Run all test targets

使用-T开关仅有任务和描述被显示,使用"-P"(or "-prereqs")得到一组任务列表和他的先决条件

### Namespaces

随着项目不断增长(任务数量增长),任务命名不断冲突,例如你或许有一个主程序一组样例程通过一个rakefile构建,在一个命名空间
放入关联的主程序的任务,构建样例程序的任务在不同的命名空间,任务名字互补干扰,如下

    namespace "main" do
      task :build do
        # Build the main program
      end
    end
    
    namespace "samples" do
      task :build do
        # Build the sample programs
      end
    end
    
    task build: %w[main:build samples:build]
    
引用一个命名空间分组的任务,可以在任务名字前面添加空间名称和冒号来引用任务(e.g."mian:build"引用了main空间的:build任务)
,嵌套命名空间也同样支持

注意这个task命令总是没有任何命名空间前缀修饰,task命令总是定义一个任务在当前命名空间

### FileTasks

文件任务名称不受命名空间范围影响,因为一个文件任务实际上是一个真实的系统文件,在命名空间中包括文件任务名字没有意义
,目录任务(使用director命令创建)也属于文件任务,也不受命名空间影响


### Name Resolution(名称解析)

当查找一个任务名字时,rake开始在当前命名空间开始查找.如果没有在当前空间找到,就会搜寻父命名空间,直到找到(如果没找到就
报错)

"rake"命名空间是一个隐式的空间,引用顶级名称

如果一个rake名字以^字符开头,名称解析从父空间开始,多个^也可以使用(这里类似git的^使用)

下面是例子有多个run任务,如何在不同的位置使用不同的名称

    task :run
    
    namespace "one" do
      task :run
    
      namespace "two" do
        task :run
    
        # :run            => "one:two:run"
        # "two:run"       => "one:two:run"
        # "one:two:run"   => "one:two:run"
        # "one:run"       => "one:run"
        # "^run"          => "one:run"
        # "^^run"         => "rake:run" (the top level task)
        # "rake:run"      => "rake:run" (the top level task)
      end
    
      # :run       => "one:run"
      # "two:run"  => "one:two:run"
      # "^run"     => "rake:run"
    end
    
    # :run           => "rake:run"
    # "one:run"      => "one:run"
    # "one:two:run"  => "one:two:run"
    
### FileLists
    
FileLists是rake管理文件队列方式,你可以将一个FileList作为一个字符串数组对于大多数情况,但是Filelist也支持一些
额外的操作

###### 创建FileList

创建一个File list很简单,仅仅是给定一组名字
    
    fl = FileList['file1.rb', file2.rb']

或者一个全局模式:

    fl = FileList['*.rb']
    

### Odds and Ends

##### do/end vs { }

代码块可以使用do/end或者大括号,我们强烈建议使用do/end为任务指定actions和rules, 因为rakefile惯用方法倾向与不适用大括号,
在task/file/rule方法里,使用大括号时会出现不寻常的含糊之处

例如,假设object_files方法返回一个对象列表在一个项目里,我们使用object_files作为前置条件,在大括号里指定actions

    # DON'T DO THIS!
    file "prog" => object_files {
      # Actions are expected here (but it doesn't work)!
    }

因为大括号的优先级高于do/end,所以代码块关联的是object_files方法 而不是file方法

下面的方法更合适

    # THIS IS FINE
    file "prog" => object_files do
      # Actions go here
    end

### Rakefile Path

当在命令行发出rake命令之前,rake将在当前目录查找rakefile文件,如果一个rakefile文件没有找到,将搜寻他的父目录直到
找到为止

例如,一个rakefile在project目录下,然后移动到这个Projct项目的最深出,然后执行rake任务,不会有什么影响

    $ pwd
    /home/user/project
    
    $ cd lib/foo/bar
    $ pwd
    /home/user/project/lib/foo/bar
    
    $ rake run_pwd
    /home/user/project
    
所有任务都会从rakefile所在目录运行

### Multiple Rake Files

不是所有的任务都需要被包含在一个单一的Rakefile,额外的rake文件(使用.rake作为扩展名的文件)被放在rakelib目录,位于项目的顶级目录里
,和包含主rakefile在一个目录.
不过,rails项目或许包含额外的rake文件在lib/tasks目录

### Clean and Clobber Tasks

通过 require 'rake/clean' ,rake提供了 清洁和修理的任务

#### clean

清理项目删除临时和备份文件, 添加文件到 clean列表作为被清理的目标

#### clobber

修理所有生成的没有源文件的文件,这个任务基于clean, 所有的clean文件和在blobber文件列表里一样将被删除,
这个任务目的就是还原项目本来样子,没有被打开的样子

你可以加入文件名字或者匹配模式到clean 和clobber列表里

#### Phony Task

phony任务被用来作为一个依赖,允许基于文件的任务使用没有基于文件的任务作为一个前置条件,不需要强制他们重新编译

你可以使用  require 'rake/phony' 添加phony任务


