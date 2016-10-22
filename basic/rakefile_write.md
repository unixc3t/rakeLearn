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

    




