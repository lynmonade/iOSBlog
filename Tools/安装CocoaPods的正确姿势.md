曾几何时，在Xcode中引入第三方库时，都会产生各种报错，各种冲突，各种奇葩问题。某个大神在受不了上述问题后，码出了CocoaPods，一个专门用来安装和管理第三方框架的神器。有了它，引入第三方库变得如此丝滑。

由于CocoaPods是需要Ruby的，因此需要先安装Ruby的环境。由于神奇的防火墙，因此不能用Ruby官方的下载链接，必须要用淘宝的Ruby镜像。需要注意的是，Mac OS升级到10.11.1后，淘宝镜像必须使用https协议。

#安装Ruby
打开Terminal，并依次输入: 

```
gem sources --remove https://rubygems.org/
gem sources --remove http://ruby.taobao.org/
gem sources --a https://ruby.taobao.org/
```
为了验证你的ruby镜像是且仅是淘宝的，可以用以下命令查看: 

```
gem sources -l
```

如果出现下面的文字就表明你安装正确啦！

```
*** CURRENT SOURCES ***

https://ruby.taobao.org/
192:~ mac$ 
```

#安装CocoaPods
接着用下面的命令安装CocoaPods

```
sudo gem install cocoapods
```
等个30秒左右，CocoaPods就安装完了(开始Terminal会卡一下，没事的)。


##坑爹问题
最后我们还需要从CocoapPods的官网下载配置数据。我们可以用`pod setup`命令来完成。但这一步有一个非常坑点的地方:需要连到cocoapods.org...由于神奇的墙，因此你的Terminal会卡在Setting up CocoaPods master repo很久很久。

cocoapods会把它的配置数据下载并安装到~/.cocoapods文件夹中。你可以用下面的命令每隔几分钟查看下载目前下载了多少，据说最后是100+M。

```
cd ~/.cococapods
du -sh *
```

下载慢问题并不是每次安装都会出现，有一次我在凌晨5点起来安装，就非常快，难道防火墙也睡觉？

##解决办法
如果实在无法忍受缓慢的下载，我们可以更改CocoaPods的索引地址，使之从国内的镜像下载数据。

```
pod repo remove master
pod repo add master https://gitcafe.com/akuandev/Specs.git
pod repo update
```

#使用CocoaPods
##把第三方框架加载到项目中
1. 使用sublime新建一个名为Podfile的文件(没有后缀名), Podfile内容如下
2. 写完后，使用Terminal, cd到项目所在文件夹, 如cd /Users/mac/Documents/workspace/LearnPodInXcode64
3. 在Terminal中输入pod install, 稍等片刻便安装成功了。

Podfile文件内容样例: 

```
platform :ios, '7.0'
pod 'JSONModel'
pod 'MJRefresh'
```

##删除第三方库、引入特定版本的第三方库

```
//如要删除第三方库，只需把对应的pod语句删除，然后重新执行pod install即可。

//引入特定版本的库
pod 'AFNetworking', '~> 2.6.3'
```

##特别注意
做完上面的步骤后，重新打开Xcode项目，就可以看到两个库已经成功引入了。注意特别注意的是，当你使用CocoaPods引入库后，千万不要通过.xcodeproj文件来打开项目，而是应该打开.xcworkspace文件来打开整个项目。这样你才会看到引入的库。



#References
* [CocoaPods安装使用心得，分享给墙内的朋友们](http://www.cocoachina.com/bbs/read.php?tid=277900)
* [CocoaPods安装和使用及问题：Setting up CocoaPods master repo](http://my.oschina.net/w11h22j33/blog/206129)
* [OS x10.11.1 cocoa pods 适配方法](http://www.jianshu.com/p/e8b2d560e808)
