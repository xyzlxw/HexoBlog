---
title: 使用GitHub和Hexo写博客
---

本文使用GitHub Pages和Hexo创建博客。
<!-- more -->

### 安装[Node.js](http://nodejs.org)
安装64-bit版本，保持默认设置即可，使用命令`node -v`或`npm -v`检查安装是否正确。
  
### 安装[Git](http://git-scm.com)
安装64-bit版本，保持默认设置即可，使用命令`git --version`检查安装是否正确。

### 配置[GitHub](https://github.com/)
* 注册账号"yourname"
* 创建仓库"yourname.github.io"

### 配置SSH keys
* 打开Git Bash, 生成SSH KEY
``` bash
$ cd ~/.ssh
$ ssh-keygen -t rsa -C "yourname@hotmail.com"
```
* 打开文件id_rsa.pub，复制文件内容
* 登陆GitHub，点击右上角的 Account Settings--->SSH Public keys ---> add another public keys
* 把你本地生成的密钥复制到里面(key文本框中)， 点击`add key`
* 测试配置是否成功 
``` bash
$ ssh -T git@GitHub.com
```

### 域名[GoDaddy](https://sg.godaddy.com/zh/)
* 购买域名"yoursite.com"
* 点击你的账户，管理我的域名
* 将GoDaddy的Nameservers更改成 f1g1ns1.dnspod.net 和 f1g1ns2.dnspod.net

### 配置[DNSpod](https://www.dnspod.cn/)
* 注册
* 添加配置
```
  *   A     192.30.252.153
  @   A     192.30.252.154
  www CNAME yourname.github.io
```

### 生成和发布 [Hexo](https://hexo.io/)

#### 安装
``` bash
$ cd d:/hexo
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ npm install hexo-deployer-git --save
$ hexo g # 或者hexo generate
$ hexo s # 或者hexo server，可以在http://localhost:4000/ 查看
```

#### 常用命令
```
hexo generate (hexo g) #生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
hexo server (hexo s) #启动本地web服务，用于博客的预览
hexo deploy (hexo d) #部署播客到远端(比如github, heroku等平台)
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
```

#### 常用简写
``` bash
$ hexo n == hexo new
$ hexo g == hexo generate
$ hexo s == hexo server
$ hexo d == hexo deploy
```

#### 部署
部署到github，需要在配置文件_config.yml中作如下修改：
```
url: http://yoursite.com
...
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```
然后在命令行中执行
``` bash
$ hexo d
```
测试yml可以使用[YAML Lint](http://www.yamllint.com/)

#### 使用git命令行部署
将我们之前创建的repo克隆到本地，新建一个目录叫做.deploy用于存放克隆的代码
``` bash
$ cd d:/hexo/blog
$ git clone https://github.com/yourname/yourname.github.io.git .deploy/yourname.github.io
```
创建一个deploy脚本文件
```
hexo g
cp -R public/* .deploy/yourname.github.io
cd .deploy/yourname.github.io
git add .
git commit -m “update”
git push origin master
```

### 添加域名的CNAME文件
在source目录下创建CNAME文件，文件内容是
```
yoursite.com
```

### 更换Hexo主题
首先下载主题freemind：
```
$ hexo clean
$ git clone https://github.com/wzpan/hexo-theme-freemind.git themes/freemind
```
然后修改Hexo目录下的_config.yml配置文件中的theme属性，将其设置为freemind
```
theme: freemind
```
最后更新主题
```
$ cd themes/freemind
$ git pull
```

### 支持LaTex
* 安装[Pandoc](http://www.pandoc.org/installing.html)
* 卸载hexo默认的marked, 然后安装pandoc
```
$ npm install hexo-renderer-mathjax --save
$ npm uninstall hexo-renderer-marked --save
$ npm install hexo-renderer-pandoc --save
```

