---
layout: post
title: "在新机器上使用octopress写文章"
date: 2014-08-05 18:57
comments: true
categories: WebBuilding
---
[主要参考来源][]

1. 安装git

2. 从github上面克隆代码到本机特定目录。

    >git clone -b source git@github.com:pliaohuimin/pliaohuimin.github.com.git



3. 安装[RUBY][] 1.9.3。记得勾选“Add Ruby executables to your PATH”，将Ruby的执行路径加入到环境变量中，如果忘记勾选，也可以手动设置

4. 安装[DevKit][]，将其解压到D:/DevKit，有两点需要注意：

     解压目录中没有有中文和空格；

     必须先安装Ruby，而且Ruby需要是RubyInstallser安装。

     解压DevKit后，在命令行输入以下命令来进行安装：

    >d:    

    >cd DevKit

    >ruby dk.rb init 

    >ruby dk.rb install


5. 安装Python

6. 安装[easy_install][],

    For Windows 7 and earlier, download ez_setup.py using your favorite web browser or other technique and "run" that file.

7. 安装pygments

    easy_install会安装在Python安装目录的Scripts目录中，例如我的Python目录是C:\Python27，所以需要将C:\Python27\Scripts目录加入到环境变量中才能在命令提示符中使用easy_install命令。

    在命令提示符中输入如下命令就可以安装Pygments了。

    >easy_install pygments


8. 然后需要安装Octopress的依赖项，安装依赖项需要用到Ruby的gem，使用下面的命令可以更换gem的更新源，使用国内的淘宝镜像速度相对快点。

    >gem sources -a http://ruby.taobao.org/

    >gem sources -r http://rubygems.org/

    >gem sources -l



    修改Octopress目录下的Gemfile文件，将第一行的http://rubygems.org/ 修改为http://ruby.taobao.org/

    在命令提示符中进入到Octopress目录，输入下面命令进行依赖项的安装

    >gem install bundler

    >bundle install


9. 在Octopress中添加文章
    使用下面命令可以在Octopress中添加文章

    >rake new_post['post name']



    注意，rake new_post['my first octopress blog']中的my first octopress blog 并不是博客标题，而是和生成的文件名以及url地址有关，该名称不支持中文。博客标题可以在生成的markdown文件中修改。生成的markdown文件在octopress/source/_posts目录中。

    编辑markdown文件，将标题可以修改为中文标题，还可以设置分类等信息以及编写正文部分

    每次执行了添加博客的命令，或是修改了现有博客的内容后，都要执行下面命令进行重新生成静态网页

    >rake generate


10. 发布到外网
    创建_deploy目录,将上一层的.git目录拷贝到_deploy下。将主干checkout出来。只需要操作一下，以后可以直接部署。
    >git checkout master

    下面指令帮你把生成的静态网页更新到主干master上去。
    >rake deploy

done
输入rake preview，可以直接localhost:4000来进行预览



##问题：
1.执行rake generate时，报错
YAML Exception reading 2012-07-21-8.textile: invalid byte sequence in GBK
YAML Exception reading 2012-07-22-13.textile: invalid byte sequence in GBK
YAML Exception reading 2012-07-23-20.textile: invalid byte sequence in GBK
YAML Exception reading 2012-07-28-35.textile: invalid byte sequence in GBK

[1解决方法][]：
找到你的Ruby安装目录，如我的是：D:\Ruby193, 在里面找到文件D:\Ruby193\lib\ruby\gems\1.9.1\gems\jekyll-0.12.0\lib\jekyll\convertible.rb
在该文件中找到下面句子：
>    def read_yaml(base, name)
>      self.content = File.read(File.join(base, name))

将它修改为：
>    def read_yaml(base, name)
>      self.content = File.read(File.join(base, name),:encoding=>"utf-8")

然后确保所有带中文字符的markdown文件是无BOM的UTF-8格式即可。

 
2.解决问题1后，执行rake generate时，继续报错
Liquid Exception: incompatible character encodings: UTF-8 and GBK in index.html

将
    >export LC_ALL=zh_CN.UTF-8 
    >export LANG=zh_CN.UTF-8

放到~/.bash_profile中，并执行：
    >source ~/.bash_profile

在cmd窗口执行，chcp 65001
然后继续在cmd窗口执行(不可以在bash窗口执行，估计只对当前窗口有效,而bash窗口又无法执行chcp命令）：
    >rake generate

[1解决方法]: http://changfengmingzhi.blog.163.com/blog/static/16710528820131013103511364/
[主要参考来源]: http://www.cnblogs.com/oec2003/archive/2013/05/27/3100896.html

[RUBY]: http://rubyinstaller.org/downloads/

[DevKit]: http://cloud.github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20111229-1559-sfx.exe

[easy_install]: https://pypi.python.org/pypi/setuptools

 