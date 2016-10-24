#### STRUCTURE OF A GEM

每个gem有一个名字,版本,和平台,例如rake这个gem,有一个版本号叫做0.8.7,
rake的平台是ruby,意味着可以运行在任何ruby平台上

平台基于cpu的机构,操作系统类型和某些系统的版本,例如"x86-mingw32"或者"java"
这个平台表示这个gem仅仅工作在使用ruby构建的一样的平台,rubygem将会自动的下载适应当前
平台的版本 具体使用gem help查看platform详细信息

一个gem里面有下面这些组件

* code 代码(包括测试代码和相应的工具)
* documenttation 文档
* gemspec

每个gem都有和下面一样的代码组织结构

    % tree freewill
    freewill/
    ├── bin/
    │   └── freewill
    ├── lib/
    │   └── freewill.rb
    ├── test/
    │   └── test_freewill.rb
    ├── README
    ├── Rakefile
    └── freewill.gemspec

你可以看到这些主要的gem组件

* 这个lib目录存放gem的代码
* test或者spec目录包含测试,基于开发人员使用的测试框架
* 一个gem通常有一个rakefile,这个rake使用这个文件自动运行测试,生成代码,和执行任务
* gem也可以包含一个可执行的文件在bin目录里,当gem被安装的时候被加载到用户的PATH里
* 文档通常包含在README文件里,或者在代码注释里,当你安装一个gem的时候,文档会自动为你
生成,大多数gem使用rdoc文档,或者yard文档
* 最后就是gemspec文件,包含了关于gem的信息,gem的文件,测试信息,平台,版本号,作者的
邮件,姓名

#### THE GEMSPEC

你的程序,你的gem的使用者,想知道谁写了这个gem, 这个gemspec包含了这些信息

下面是一个gemspec文件例子,下一章学习如何编写自己的gem

    % cat freewill.gemspec
    Gem::Specification.new do |s|
      s.name        = 'freewill'
      s.version     = '1.0.0'
      s.summary     = "Freewill!"
      s.description = "I will choose Freewill!"
      s.authors     = ["Nick Quaranto"]
      s.email       = 'nick@quaran.to'
      s.homepage    = 'http://example.com/freewill'
      s.files       = ["lib/freewill.rb", ...]
    end

