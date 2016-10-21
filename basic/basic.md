### Installation

 #### Gem 安装

    gem install rake

### Usage

#### Simple Example

首先你必须写一个"RakeFile"文件,包含构建的rules,下面是一个例子

    task default: %w[test]

    task :test do
      ruby "test/unittest.rb"
    end
这个RakeFile文件有2个任务

* 一个任务叫做"test",调用后运行一个单元测试文件
* 一个任务叫做"default",这个任务什么都不做,但是它有一个依赖,名字叫做"test"的任务,调用这个
"default"任务,也会调用运行这个"test"任务.

运行"rake"命令不需要任何参数,将会直接运行这个default任务.如下

    % ls
    Rakefile     test/
    % rake
    (in /home/some_user/Projects/rake)
    ruby test/unittest.rb
    ....unit test output here...

使用rake -help得到所有的可用选项
