#### FINDING GEMS
 
search命令让你可以通过gem的名字远程查找gem,你也可以使用正则表达式
    
    $ gem search ^rails
    
    *** REMOTE GEMS ***
    
    rails (4.0.0)
    rails-3-settings (0.1.1)
    rails-action-args (0.1.1)
    rails-admin (0.0.0)
    rails-ajax (0.2.0.20130731)
    [...]

如果你看到一个gem，你想要更多的信息，你可以添加细节选项。
    
    $ gem search ^rails$ -d
    
    *** REMOTE GEMS ***
    
    rails (4.0.0)
        Author: David Heinemeier Hansson
        Homepage: http://www.rubyonrails.org
    
        Full-stack web application framework.

#### INSTALLING GEMS

install命令下载安装gem和必要的依赖,然后构建安装Gem的文档

    $ gem install drip
    Fetching: rbtree-0.4.1.gem (100%)
    Building native extensions.  This could take a while...
    Successfully installed rbtree-0.4.1
    Fetching: drip-0.0.2.gem (100%)
    Successfully installed drip-0.0.2
    Parsing documentation for rbtree-0.4.1
    Installing ri documentation for rbtree-0.4.1
    Parsing documentation for drip-0.0.2
    Installing ri documentation for drip-0.0.2
    Done installing documentation for rbtree, drip after 0 seconds
    2 gems installe

命令中的drip依赖于rbtree,在rbtree基础上增加了扩展, rubygem安装了rbteee依赖并构建了扩展,
然后安装drip,然后构建安装文档.

你可以关闭生成文档使用 --no-doc参数,当安装gem时.

#### REQUIRING CODE

在require语句出现的地方rubygem修改你的ruby加载路径控制你的RUBY代码,当你require一个gem,
你将gem的lib目录天际到你的$LAOD_PATHl,让我你们使用irb和pretty_print库来测试一下

tip:传递-r给irb 将会自动引入一个lib 当rib加载的时候

    % irb -rpp
    >> pp $LOAD_PATH
    [".../lib/ruby/site_ruby/1.9.1",
     ".../lib/ruby/site_ruby",
     ".../lib/ruby/vendor_ruby/1.9.1",
     ".../lib/ruby/vendor_ruby",
     ".../lib/ruby/1.9.1",
     "."]

默认情况,仅仅是一些系统目录和ruby标准库被加载到$LOAD_PATH, 加入awosome_print目录到load path里
你可以引入他的一个文件

    % irb -rpp
    >> require 'ap'
    => true
    >> pp $LOAD_PATH.first
    ".../gems/awesome_print-1.0.2/lib"
    
注意: ruby1.8你必须在引入任何gem之前执行,require 'rubygems'

一旦你引入了ap,rubyGems会自动加载它的lib目录

一个gem应该是什么样的,将ruby代码放到lib目录,其中命名一个ruby文件和你的gem名字一样(例如一个
freewill的gem,应该有一个叫freewill.rb的文件)能被rubygem加载

lib目录本身通常仅包含一个.rb文件和一个与gem名字一样的目录包含其余的文件

    % tree freewill/
    freewill/
    └── lib/
        ├── freewill/
        │   ├── user.rb
        │   ├── widget.rb
        │   └── ...
        └── freewill.rb

#### LISTING INSTALLED GEMS

使用list命令显示本地安装的gem

    $ gem list
    
    *** LOCAL GEMS ***
    
    bigdecimal (1.2.0)
    drip (0.0.2)
    io-console (0.4.2)
    json (1.7.7)
    minitest (4.3.2)
    psych (2.0.0)
    rake (0.9.6)
    rbtree (0.4.1)
    rdoc (4.0.0)
    test-unit (2.0.0.0)

#### UNINSTALLING GEMS

使用 uninstall 命令移除你已经安装的gems

    $ gem uninstall drip
    Successfully uninstalled drip-0.0.2
    
如果你卸载一个gem的依赖,rubyGems会要求你确认

    $ gem uninstall rbtree
    
    You have requested to uninstall the gem:
      rbtree-0.4.1
    
    drip-0.0.2 depends on rbtree (>= 0)
    If you remove this gem, these dependencies will not be met.
    Continue with Uninstall? [yN]  n
    ERROR:  While executing gem ... (Gem::DependencyRemovalException)
        Uninstallation aborted due to dependent gem(s)

#### VIEWING DOCUMENTATION

使用ri命令查看安装的gem的文档

    $ ri RBTree
    RBTree < MultiRBTree
    
    (from gem rbtree-0.4.0)
    -------------------------------------------
    A sorted associative collection that cannot
    contain duplicate keys. RBTree is a
    subclass of MultiRBTree.
    -------------------------------------------

你可以使用server 命令能够在浏览器里查看安装的gem的文档

    $ gem server
    Server started at http://0.0.0.0:8808
    Server started at http://[::]:8808

#### FETCHING AND UNPACKING GEMS

如果你想检查一个gem的内容而不想安装它,你可以使用fetch 命令下载这个.gem文件,然后提取他的内容使用
unpack命令

    $ gem fetch malice
    Fetching: malice-13.gem (100%)
    Downloaded malice-13
    $ gem unpack malice-13.gem
    Fetching: malice-13.gem (100%)
    Unpacked gem: '.../malice-13'
    $ more malice-13/README
    
    Malice v. 13
    
    DESCRIPTION
    
    A small, malicious library.
    
    [...]
    $ rm -r malice-13*

你也可以解压你已经安装的gem,修改一些文件,然后使用修改过得gem代替已经安装的gem

    $ gem unpack rake
    Unpacked gem: '.../rake-10.1.0'
    $ vim rake-10.1.0/lib/rake/...
    $ ruby -I rake-10.1.0/lib -S rake some_rake_task
    [...]

-I参数添加到你解压的rake到ruby的$LOAD_PATH防止rubyGems读取默认的gem,-S参数表示,查找rake
在shell的path里面使你不需要输入完整的rake路径

