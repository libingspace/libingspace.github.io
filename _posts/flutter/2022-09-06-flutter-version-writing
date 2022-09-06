---
layout:       post
title:        "MAC 升级ruby和cocoapods"
author:       "Hux"
header-style: text
catalog:      true
tags:
    - Flutter
    
---

> 在做flutter开发时，需要ruby环境，而ruby版本太低。我开始了升级，然后开始掉坑

公司APP发布到testflight的版本已经过期，原本只是想搞个ruby升级搞了一天，但是执行pod命令直接就报错了。错误信息和ruby有关，
我感觉是ruby版本太低了，所以先进行升级。

### CocoaPods
CocoaPods依赖的语言环境，cocoapods就是用ruby写的，所以安装cocoapods前 需要准备好ruby环境。
CocoaPods是OSx和iOS下的一个类库管理工具，通过CocoaPods工具可以为项目添加依赖库和管理其版本。

### CocoaPods常用命令
```
pod init: 创建Podfile文件，我们可以通过将需要的pod依赖库添加到podfile文件中，实现在项目中添加依赖
```
```
pod install: 执行第三方库的安装，如果podfile.lock存在则会根据该文件指定的版本进行安装。如果不存在podfile.lock则会
生成一个。
```
```
pod update: 执行各种库的安装，并且更新到最高版本。如果podfile中指定的依赖库不是写死的。
```
```
pod search: 搜索可以用的pod依赖库，并且结果中会告诉我们如何在pod中使用依赖库
```
```
pod outdated: 查询可更新版本的pod依赖库
```
### 各种软件包管理工具
    
在升级ruby的过程中，我尝试了reben、rvm、gem、brew。都可以安装ruby。实话说，我却是不知道搞这么多出来是干啥，

网上的资料也都很多。没办法，就挨个尝试，先尝试了rbenv，到后面SSL错误，搞不定。换rvm，也是SSL错误。再换brew，可以正常安装。
下面描述一下brew的使用
```
安装软件：brew install xxx
卸载软件：brew uninstall xxx
搜索软件：brew search xxx
更新软件：brew upgrade xxx
查看列表：brew list
更新brew：brew update
清理所有包的旧版本：brew cleanup
清理指定包的旧版本：brew cleanup $FORMULA
查看可清理的旧版本包，不执行实际操作：brew cleanup -n
```
安装ruby流程
```
brew install ruby

echo 'export PATH="/usr/local/opt/ruby/bin:$PATH"' >> ~/.zshrc
export LDFLAGS="-L/usr/local/opt/ruby/lib"
export CPPFLAGS="-I/usr/local/opt/ruby/include"
source ~/.zshrc
```
更换ruby源
```shell
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```
安装CocoaPods
```shell
sudo gem install -n /usr/local/bin cocoapods
# 一定要加上/usr/local/bin
```

安装成功后pod命令就可以正常使用了。
我在安装成功后，flutter遇到了版本问题。继续折腾了好几个小时。