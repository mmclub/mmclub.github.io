---
layout: post
title: "Puma: 一个为并发构建的Ruby服务器"
date: 2014-03-03 19:14:42 +0800
comments: true
categories: [Ruby, Web, Puma, 林翔宇]
---

本文翻译自Puma官方文档，同时发布于[Github](https://github.com/oa414/puma/blob/master/README.zh.md)

译者：林翔宇


## 概述

Puma是一个简单，快速，基于线程，并且高并发的面向Ruby/Rack程序的 HTTP 1.1 服务器。Puma同时面向开发和产品环境。为了达到最高的效率，推荐使用一个实现了真正线程的Ruby实现，比如Rubinius或者JRuby。



## 为速度和并发构建。

Puma是一个简单，快速，基于线程，并且高并发的面向Ruby/Rack程序的 HTTP 1.1 服务器。它可以用于任何支持Rack的程序，可以取代Webrick或者Mongrel。它是为了[Rubinius](http://rubini.us)而设计的服务器，但是同样可以工作在JRuby和MRI下。Puma同时面向开发和产品环境。



Puam使用了一个继承自Mongrel的C优化的Ragel扩展，来快速并且精确地解析HTTP1.1请求（并且便于移植）。之后，Puma将这些请求放到一个你可以控制的内部线程池的线程之中，所以Puma为你的web应用提供了一个真实的并发环境。


在 Rubinius2.0 中，Puma会用真实的线程利用你所有的CPU核心，这意味着你不用使用多个进程来增加吞吐量。你可以看到在JRuby上类似的优点。


在 MRI中, 因为有全局解释器锁的存在（GIL），同时只能有一个真实的线程在运行。但是当你在做许多IO阻塞的事情的时候（比如HTTP调用像 Twitter的外部API的时候），Puma仍然会通过运行并发运行阻塞的IO调用增加MRI的吞吐量。（ 基于 EventMachine的服务器，如Thin会关掉这个功能，要求你只能使用特别的库）。每个人的选择因实际的应用而异。为了达到最大的吞吐量，强烈推荐使用一个实现了真实线程的Ruby实现，比如[Rubinius](http://rubini.us) 或者 [JRuby](http://jruby.org)。


## 快速入门


最简单的使用Puma的方式是使用 RubyGems:

    $ gem install puma

现在`puma`命令已经加入到你的环境变量里了，在你的应用程序根目录下运行下面命令启动你的Rack应用:

    $ puma app.ru

## 高级步骤

### Sinatra

你可以用下面的命令用Puma运行你的Sinatra应用


    $ ruby app.rb -s Puma

或者在你的应用中设置总是使用Puma

    require 'sinatra'
    configure { set :server, :puma }

如果你使用 Bundler,确保你把Puma加入了你的Gemfile中(步骤见下面)。


### Rails

首先，确保你把Puma加入了你的Gemfile中。


    gem 'puma'

然后用Rails命令启动Puma

    $ rails s Puma

### Rackup

你可以把Puma作为`rackup`的选项。


    $ rackup -s Puma

你可以改变你的 `config.ru` 来默认选择Puma服务器，加入下面的这行:

    #\ -s puma

## 配置

Puma提供了很多控制服务器器运行的选项，完整列表请查阅`puma -h` (或者 `puma --help`) 

### 线程池

Puma利用了一个你可以改变的动态线程池，你可以通过`-t` (或者 `--threads`) 选项:设置最小和最大的线程池里的可以使用的线程数量


    $ puma -t 8:32

Puma会动态的根据当前的负载改变线程数量，默认的数量是`0:16`, 大胆的尝试吧，但是不要把最大线程数量设置的太大，这会耗尽系统资源。


### 集群模式


Puma 2 提供了集群模式 , 允许你使用 fork出来的进程处理并发处理多个请求。 你可以用-w` (or `--workers`) 参数调整运行的work进程数量。


    $ puma -t 8:32 -w 3

在提供真实线程的Ruby实现中，你应该把这个数字调整到和CPU核心数量相同。

注意集群模式下仍然是使用线程的， `-t` 选项设置每个 worker的进程数量，所以 `-w 2 -t 16:16` 可能开启了32个线程。

如果你在并发模式你可以选择在启动worker进程前预加载你的应用。如果想利用[MRI Ruby 2.0](https://blog.heroku.com/archives/2013/3/6/matz_highlights_ruby_2_0_at_waza)推出的[Copy on Write](http://en.wikipedia.org/wiki/Copy-on-write) 的优点的话，这是必要的。只需要调用的时候指定一个 `--preload` 参数:


    # CLI invocation
    $ puma -t 8:32 -w 3 --preload

如果你在使用配置文件，使用`preload_app!`方法，并且使用`-C`参数指定配置文件:


    $ puma -C config/puma.rb

    # config/puma.rb
    threads 8,32
    workers 3
    preload_app!

此外，你可以在配置文件里面定义一个代码块，来在每个工作线程启动的时候运行：


    # config/puma.rb
    on_worker_boot do
      # configuration here
    end

这些代码可以用来在启动应用的时候初始化进程, 允许你做一些和Puma服务器有关，但是不和应用结合的事情。比如，你可以发送一个日志通知Puma启动了。


你可以多次增加钩子调用。

如果你使用ActiveRecord预加载你的应用， 推荐你在这里这样初始化你的连接池：

    # config/puma.rb
    on_worker_boot do
      ActiveSupport.on_load(:active_record) do
        ActiveRecord::Base.establish_connection
      end
    end

### 绑定 TCP / Sockets

相比较其他需要很多配置参数的服务器，Puma仅仅通过`-b` (或者 `--bind`)选项使用一个URI参数：

    $ puma -b tcp://127.0.0.1:9292

想通过使用UNIX Sockets来取代TCP么（这能提高5-10%的性能），没问题！


    $ puma -b unix:///var/run/puma.sock

如果你需要改变Unix socket的权限，只需要加入一个umask参数：

    $ puma -b 'unix:///var/run/puma.sock?umask=0777'

希望更加安全？使用 SSL sockets!

    $ puma -b 'ssl://127.0.0.1:9292?key=path_to_key&cert=path_to_cert'

### 控制/状态服务

Puma提供了一个内置的控制/状态服务app来提供对Puam自身的控制，这是一个打开Puma控制服务器的样例：


    $ puma --control tcp://127.0.0.1:9293 --control-token foo

它会在localhost的9293端口打开Puma配置服务器。此外，所有的到控制服务器的请求需要包括一个`token=foo` 作为查询参数，作为简单的认证。更多app用法参见[status.rb](https://github.com/puma/puma/blob/master/lib/puma/app/status.rb)。



### 配置文件

你可以同时使用一个`-C` (或 `--config`) 参数提供一个哦诶之文件:

    $ puma -C /path/to/config

请参考 [示例配置](https://github.com/puma/puma/blob/master/examples/config.rb) 或者查询 [configuration.rb](https://github.com/puma/puma/blob/master/lib/puma/configuration.rb) 查看所有的配置项目

## 重启

Puma有能力在更新版本的时候重启自己，当平台允许的时候(MRI, Rubinius, JRuby),Puma 进行热重启。这是和
 *unicorn* 和 *nginx* 在重启的时候保持服务器sockets连接相同的。它可以保证重启的服务器替代旧的服务器的时候请求已经被处理。

Puma有2个内建的重启机制：

  * 发送`SIGUSR2` 信号到`puma` 进程
  * 使用 status server 和 issue `/restart`

当前和重启的进程不会共享代码，所以手动停止Puma并且关闭它也是安全的。

如果新的进程不能加载，它会简单的退出。你应该在生产环境下用一个监控程序运行Puma。


### Cleanup Code

Puma不能理解所有你的App使用的资源，所以它在你用`-C`提供的配置文件里面提供了一个叫`on_restart`的钩子。传递给`on_restart` 会被Puma在重启自己的时候调用。

你应该在这个代码块里面关闭全局日志文件，redis连接等，这样它们的文件描述符不会在重启的进程里面泄漏。否则会导致文件描述符打开太多，当服务器重启次数很多的时候会导致应用崩溃 。

### 平台限制

不同的平台有差异，下列是Puma在不同平台上有差异的地方


  * **JRuby**, **Windows**: 服务器的Socket不能被无缝重启， 这些平台不能通过Ruby传递信息到新的进程
  * **JRuby**, **Windows**: 不支持集群模式 ，因为没有fork  fork(2)
  * **Windows**: 不支持daemon mode ，因为没有fork(2)

## pumactl

`pumactl` 是一个简单的CLI前端来控制或者查看上述的app状态。 请参考  `pumactl --help` 

## 维护多个Puma  / init.d / upstart 脚本

如果你想马上得到一个维护多个程序的脚本，请查看用于init.d 和 upstart脚本的[tools/jungle](https://github.com/puma/puma/tree/master/tools/jungle)
## Capistrano 部署

Puma 已经被包括进 Capistrano [deploy script](https://github.com/puma/puma/blob/master/lib/puma/capistrano.rb), 你只要 require :

config/deploy.rb

```ruby
require 'puma/capistrano'
```

然后

```bash
$ bundle exec cap puma:start
$ bundle exec cap puma:restart
$ bundle exec cap puma:stop
$ bundle exec cap puma:phased_restart
```


## 协议

Puma 的版权属于Evan Phoenix 和 贡献者. 它基于BSD 协议. 详见LICENSE文件。



