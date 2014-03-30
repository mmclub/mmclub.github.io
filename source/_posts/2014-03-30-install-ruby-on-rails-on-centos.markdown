---
layout: post
title: "How To Install Ruby on Rails On CentOS 6.5 Using Rbenv"
date: 2014-03-30 11:11:37 +0800
comments: true
categories: 林翔宇, CentOS, Rails, VPS
---


## Introduction

### Ruby on Rails

Ruby on Rails is one of the most popular web development framework, it's build upon  Ruby Programming Language, and it's the hottest web development stack currently.

### Rbenv

[Rbenv](https://github.com/sstephenson/rbenv) is a shell script tools created by [Sam Stephenson](http://sstephenson.us/). It's used for groom your app’s Ruby environment.Use rbenv can pick a Ruby version for your application and guarantee that your development environment matches production.

rbenv works by inserting a directory of shims at the front of your PATH:

     ~/.rbenv/shims:/usr/local/bin:/usr/bin:/bin
    
Through a process called rehashing, rbenv maintains shims in that directory to match every Ruby command across every installed version of Ruby—irb, gem, rake, rails, ruby, and so on.

### CentOS

CentOS is derived from  Red Hat Enterprise Linux. The target users of these distributions are usually businesses, which require their systems to be running the most stable way for a long time.So we are going to use  CentOS 6.5 running our applications.

## Step One - Install dependencies

Before, installing any package, it's always recommended to update package repository cache use yum.

     sudo yum update
    
Now,in order to get necessary development tools and dependencies, run the following:

     sudo yum groupinstall -y 'development tools'
     sudo yum install -y gcc-c++ glibc-headers openssl-devel readline libyaml-devel readline-devel zlib zlib-devel  sqlite-devel  


## Step Two - Install Rbenv and ruby-build


Then we are ready to get Rbenv downloaded installed, run the following to check out rbenv into ~/.rbenv:

     git clone git://github.com/sstephenson/rbenv.git ~/.rbenv
    
Add ~/.rbenv/bin to your $PATH for access to the rbenv command-line utility:

     echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    
Add rbenv init to your shell to enable rbenv shims and autocompletion.
    
     echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    
Ruby-build is a Rbenv plugin which provides the rbenv install command that simplifies the process of installing new Ruby versions. Install rbenv-build:

     git clone https://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    
reloaded your bash_profile to enable rbenv command:

     source ~/.bash_profile

## Step Three - Install Ruby

Install Ruby 2.1.0 and make it the default

     rbenv install 2.1.0
     rbenv rehash
     rbenv global 2.1.0
    
Now you can run:

     ruby -v
    
to verify your ruby environment has been installed successful。 It will output something like this:

     ruby 2.1.0p0 (2013-12-25 revision 44422) [x86_64-linux]
    
## Step Four - Install Nodejs

Ruby on Rails need a JavaScript runtime support. It use [execjs](https://github.com/sstephenson/execjs) gem which can automatically picks the best runtime available to evaluate your JavaScript program, then returns the result to you as a Ruby object.

ExecJS supports these runtimes:

- therubyracer - Google V8 embedded within Ruby
- therubyrhino - Mozilla Rhino embedded within JRuby
- Node.js
- Apple JavaScriptCore - Included with Mac OS X
- Microsoft Windows Script Host (JScript)
    
So we can use therubyracer gem or  Node.js in with our CentOS and MRI Ruby. In this guide we use [Nodejs](http://nodejs.org/) as JavaScript runtime.

Node.js is available from the [Fedora Extra Packages for Enterprise Linux (EPEL)](https://fedoraproject.org/wiki/EPEL) repository.

Extra Packages for Enterprise Linux (or EPEL) is a Fedora Special Interest Group that creates, maintains, and manages a high quality set of additional packages for Enterprise Linux, including, but not limited to, Red Hat Enterprise Linux (RHEL), CentOS and Scientific Linux (SL), Oracle Enterprise Linux(OEL).

To check if you have EPEL, run

     yum repolist

if you don't see epel, install it via RPM

     rpm -Uvh http://download-i2.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
    
And then run the following command to install node:
    
     sudo yum install nodejs --enablerepo=epel
    

    
## Step Four - Install Rails Gem and test it.


 Rails 4.0 needs RubyGems 2.0.3, so you have to update your system by using following command

     gem update --system 2.0.3
    
Now, you can install the rails gem

     gem install rails
     rbenv rehash
    
Test your rails:

     rails new projectname
     cd projectname
     rails server
    
now open your browser and open http://your-server-ip:3000,you can find the rails project default page.
    



