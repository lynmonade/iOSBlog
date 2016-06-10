目前，很多第三方库都支持使用AppleDoc创建对应的API文档。你可以到第三方库的github页面去了解其是否支持appledoc以及相关的doc生成命令。
#安装appledoc
```objc
//在terminal中输入
git clone git://github.com/tomaz/appledoc.git
cd ./appledoc
sudo sh install-appledoc.sh

//安装完成后，验证下是否成功
appledoc --version
```

#生成文档
```objc
1. 先去github下载第三方库的源码。(git clone XXX.git)
2. cd到第三方库所在目录。
3. 在terminal中执行生成doc的命令。下面以AFNetworking为例(命令都可以对应的github页面上找到)

appledoc --project-name="AFNetworking" --project-company="AFNetworking" --company-id="com.afnetworking.afnetworking" AFNetworking/*.h

4. 然后就可以用dash查看文档了。
```

#参考
[用 appledoc 生成文档](http://blog.ibireme.com/2013/08/26/appledoc-guide/)