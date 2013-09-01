---
layout: post
title: "从wordpress迁移到octopress"
date: 2013-09-01 03:54
comments: true
categories: 
---
##为什么要迁移：##
>并不是觉得wordpress不好用，而是由于之前一直在使用的noads.biz服务器开始在我的个人网站上放广告，而我找不到其它更好的劫持php,mysql的免费服务器，所以决定使用octopress，可以直接把内容放在github上。

##迁移步骤：##

###准备工作###
* step 1:在github上注册一个用户。并用该用户创建一个“username.github.com“的项目库，其中username为你在github注册的用户
* step 2:在LINUX上安装好GIT
* step 3:在LINUX上安装好ruby
curl -L https://get.rvm.io | bash -s stable --ruby
source /home/liao/.rvm/scripts/rvm
如果执行下列命令报错，请重新打开终端，并再次执行source /home/liao/.rvm/scripts/rvm

###安装好OCTOPRESS###
* step 4:克隆octopress项目
git clone git://github.com/imathis/octopress.git
* step 5:安装OCTOPRESS
gem install bundler
bundle install
* step 6:安装主题
rake install

###部署到GITHUB###
* step 7:rake setup_github_pages
期间会要求你输入项目地址，请输入:git@github.com:username.github.com
期中username为你在github的用户名。    
* step 8:生成代码
rake generate
* step 9:部署到github上
rake deploy
期间可能会报错，可以尝试跳转到_deploy目录下，运行git pull origin master。再回到上一层再次执行rake deploy
* step 10:备份相关文件到source分支
git add 
git commit -m 'your message
git push origin source
如果报错，可以尝试先执行下列命令，再一次执行上面的命令。
git branch source
git checkout source

###关于域名的绑定###
如果是使用一级域名，在source目录下添加CNAME文件，并在里面添加你的域名，如www.minlh.tk
然后在你的域名服务商那里添加一行A RECORD，IP地址为207.97.227.245

###关于文章的迁移###
使用wordpress后台管理工具，将数据导出成xml格式
安装必要工具：
gem install hpricot
gem install ya2yaml
使用XDITE提供的工具
https://gist.github.com/xdite/1273518

执行直接執行 ruby xxx.rb xxx.xml 名
st目录，将该目录下的所有文件拷贝到source/_post目录下即可。

###关于文章中图片的迁移###
将wp-content/uploads目录下载，上传到source目录即可。
图片链接出现时border=0 hspace=0 alt="" align=baseline，会导致生成的blogs双引号自动转换成&#8220;字样，需要将这些去掉，图片才能正常显示。


