#### INTRODUCTION

创建和发布你的gem十分简单,让我们制作一个"hello world"gem

#### YOUR FIRST GEM

我们新建一个ruby文件和一个gemspec文件为了我们的hola gem,你需要一个新的名字用于发布
你的gem,可以查看[basic remommendations](http://guides.rubygems.org/patterns/#consistent-naming)

    % tree
    .
    ├── hola.gemspec
    └── lib
        └── hola.rb

代码放到lib目录里,约定有一个ruby文件和你的gem有一样的名字,当使用require 'hola'的时候会被读取,这个文件负责设置你的gem
的api和代码

lib/hora.rb的代码十分简单, 仅仅用来让你看到可以从gem里输出一些东西

    % cat lib/hola.rb
    class Hola
      def self.hi
        puts "Hello world!"
      end
    end

gemspec定义了一些关于这个gem信息,你在rubyGems.org看到的所有信息都来自于这个文件

    % cat hola.gemspec
    Gem::Specification.new do |s|
      s.name        = 'hola'
      s.version     = '0.0.0'
      s.date        = '2010-04-28'
      s.summary     = "Hola!"
      s.description = "A simple hello world gem"
      s.authors     = ["Nick Quaranto"]
      s.email       = 'nick@quaran.to'
      s.files       = ["lib/hola.rb"]
      s.homepage    =
        'http://rubygems.org/gems/hola'
      s.license       = 'MIT'
    end
    
描述的成员比你在这个例子里看到的多,如果匹配/^==[A-Z]/ 描述可以通过RDoc's markup format显示在
rubyGems网站页面.但是注意数据的其他用户可能不知道标记的意思

看起来眼熟,这个gemspec也是ruby, 所以你可以封装脚本用来生成文件名字和增加版本号,gemspec里有许多fields可以包含
详细看[点击查看](http://guides.rubygems.org/specification-reference)

创建完gemspec之后,你可以使用它构建一个gem,然后本地安装它

    % gem build hola.gemspec
    Successfully built RubyGem
    Name: hola
    Version: 0.0.0
    File: hola-0.0.0.gem
    
    % gem install ./hola-0.0.0.gem
    Successfully installed hola-0.0.0
    1 gem installed
    
当然冒烟测试还没进行,最后一步云require这个gem并且使用它

    % irb
    >> require 'hola'
    => true
    >> Hola.hi
    Hello world!

如果你使用的是早于ruby1.9,2版本,你需要会话开始时使用irb -rubygems 或者require 这个rubygems
在你启动irb之后

现在你能够分享hola给你社区的伙伴,发布你的gem到rubygems.org近需要一个命令,提供一个访问网站的账户

使用你的rubygems账号设置你的电脑
    
    $ curl -u qrush https://rubygems.org/api/v1/api_key.yaml >
    ~/.gem/credentials; chmod 0600 ~/.gem/credentials
    
    Enter host password for user 'qrush'
    
如果你使用curl ,openSSL或者证书有问题,你或许简单的尝试输入上面的url到你的浏览器地址栏里,你的浏览器
将会询问你,rubyGems.org的账号和地址,你的浏览器尝试下载api_key.yaml文件,保存到~/.gem目录下,并调用他的
证书

一旦设置后,你就可以将这个gem推送到rubyges.org

    % gem push hola-0.0.0.gem
    Pushing gem to RubyGems.org...
    Successfully registered gem: hola (0.0.0)
   
只需要一会(少于1分钟)你的gem将会可以被任何人使用,你可以在rubyGems.org网站看到,或者从其他安装了rubyGems的
电脑上

使用ruby和rubyGems分享代码如此简单

#### REQUIRING MORE FILES

所有东西都在一个文件不太好,,让我们再加入一些代码到gem里

    % cat lib/hola.rb
    class Hola
      def self.hi(language = "english")
        translator = Translator.new(language)
        translator.hi
      end
    end
    
    class Hola::Translator
      def initialize(language)
        @language = language
      end
    
      def hi
        case @language
        when "spanish"
          "hola mundo"
        else
          "hello world"
        end
      end
    end

这个文件现在变得十分臃肿,让我们将tranlator放到单独一个文件里,就如前面说的,gem的跟文件负责加载代码,gem的其他文件
通常放到和gem一样名字的目录里在lib目录下,我们可以额分割gem像下面这样
    
    % tree
    .
    ├── hola.gemspec
    └── lib
        ├── hola
        │   └── translator.rb
        └── hola.rb

Translator位于lib/hola里面,使用require语句从Lib/hola.rb中得到,Translator代码没有改变太多

    % cat lib/hola/translator.rb
    class Hola::Translator
      def initialize(language)
        @language = language
      end
    
      def hi
        case @language
        when "spanish"
          "hola mundo"
        else
          "hello world"
        end
      end
    end

现在hola.rb文件有一行代码加载Translator

    % cat lib/hola.rb
    class Hola
      def self.hi(language = "english")
        translator = Translator.new(language)
        translator.hi
      end
    end
    
    require 'hola/translator'

注意:对于新创建文件,不要忘记添加到hola.gemspec文件里,例如下面
    
    % cat hola.gemspec
    Gem::Specification.new do |s|
    ...
    s.files       = ["lib/hola.rb", "lib/hola/translator.rb"]
    ...
    end

如果不像上面添加,新文件不会添加到安装后的gem里

让我们首先尝试一下,启动irb

    % irb -Ilib -rhola
    irb(main):001:0> Hola.hi("english")
    => "hello world"
    irb(main):002:0> Hola.hi("spanish")
    => "hola mundo"

我们需要使用一个陌生的命令标记 -Ilib .通常rubyGems会为你引入Lib目录,所以用户不需要担心配置你的
加载路径,然而,如果你在rubyGems之外运行代码,你必须配置,也可以使用代码来修改$load_path,但是大
多数情况这么做被认为是一种反模式,这有许多反模式(和好的模式)对于gem,详细请看[这里](http://guides.rubygems.org/patterns)

如果你添加更多的文件到gem里,确保不要忘记添加他们到你的gemspec文件数组里,在你推送一个新的gem之前,
许多开发者使用[Hoe](https://github.com/seattlerb/hoe),[Jeweler](https://github.com/technicalpickles/jeweler)
[rake](https://github.com/jimweirich/rake),[Bundler](http://railscasts.com/episodes/245-new-gem-with-bundler)
[动态gemspec](https://github.com/wycats/newgem-template/blob/master/newgem.gemspec)
来解决这个问题

