---
title: hexo搭建笔记
date: 2023-10-13 18:48:02
tags: "hexo"
categories: "hexo"
---

### Hexo初步搭建

#### Hexo简介

hexo是基于node.js的静态博客框架，可以方便地生成静态网页托管在GitHub和Coding上，下列是[hexo官网](https://hexo.io/zh-cn/)

#### Hexo搭建步骤

1. 安装Git
2. 安装Node.js
3. 安装Hexo
4. Github创建个人仓库
5. 生成SSH添加到GitHub
6. 将hexo部署到GitHub
7. 设置个人域名
8. 发布文章

##### 安装git

**windows：**到git官网上下载，[Download git](https://gitforwindows.org/)，下载后会有一个Git Bash的命令行工具可供使用

**linux：**

```
sudo apt-get install git
```

安装好以后可使用`git --version`查看版本

##### 安装nodejs

**windows：**[nodejs](https://nodejs.org/en/download/)官网下载

**linux：**

```
sudo apt-get install nodejs
sudo apt-get install npm
```

安装完成后可以使用`node -v`或`npm -v`查看对应的版本，检查是否安装成功

##### 安装hexo

首先创建一个blog文件夹，在该文件夹下使用git bash输入命令：

```
npm install -g hexo-cli
```

使用`hexo -v`查看版本

初始化hexo：`hexo init myblogname`

```
cd myblogname
npm install
```

###### hexo创建文件夹目录介绍

- node_modules：依赖包
- public：存放生成的页面
- scaffolds：生成文章的一些模板
- source：存放自己的文章
- themes：主题
- _config.yml：博客的配置文件

```
hexo g
hexo server
```

打开hexo服务后，在浏览器中输入localhost:4000可以看到本地生成的博客

##### github创建个人仓库

首先注册一个github账户，通过`New repository`新建仓库；创建一个和用户名相同的仓库，后面加`.github.io`便于后续部署到github page的时候识别

![](https://img-blog.csdnimg.cn/img_convert/d002150a2eb02a0358d1c88199e4e726.png)

##### 生成SSH添加到github

回到git bash中

```
git config --global user.name "yourname"
git config --global user.email "youremail"
```

可以用以下两条检验是否输入正确

```
git config user.name
git config user.email
```

然后创建ssh

```
ssh-keygen -t rsa -C "youremail"
```

在对应的路径中找到ssh存放目录，其中`id_rsa`是私人秘钥，`id_rsa.pub`是公共秘钥，将该公钥放在github上，当链接github自己的账户时，它会根据公钥匹配私钥，从而判断是否能够通过git上传文件

在GitHub的setting中找到SSH keys的设置选项，点击`New SSH key`
将`id_rsa.pub`里面的信息复制进去

![](https://img-blog.csdnimg.cn/img_convert/3194ad0a9d04d94c09485122932968f3.png)

通过`ssh -T git@github.com`查看是否成功

##### 将hexo部署到github中

在`_config.yml`的最后修改为

```yaml
deploy:
  type: git
  repo: https://github.com/YourgithubName/YourgithubName.github.io.git
  # 若后续部署失败，可以尝试更改为ssh方式,将上述链接替换成下面：
  # git@github.com:YourgithubName/YourgithubName.github.io.git（推荐）
  branch: master
```

安装deploy-git，即部署命令

```
npm install hexo-deployer-git --save
```

```
hexo clean && hexo g && hexo d
```

其中`hexo clean`表示清除之前生成的东西，可以不加

`hexo generate`表示生成静态文章，可以用`hexo g`缩写

`hexo deploy`部署文章，可以用`hexo d`缩写

出现下图表示部署成功，稍后可以在`http://yourname.github.io`看到博客

![](https://img-blog.csdnimg.cn/img_convert/a4eba29bb7c185169228b596e636422f.png)

##### 设置个人域名

在阿里云上购买一个域名，然后在域名控制台中查看购买的域名，点**解析**进去，添加解析**，解析线路选择默认**；登录GitHub，进入之前创建的仓库，点击settings，设置Custom domain，输入域名

然后在博客文件source中创建一个名为CNAME文件；写上域名

最后在gitbash中输入

```
hexo clean && hexo g && hexo d
```

过一段时间输入域名就可以看到网站了；接下来可以正式开始写文章了

```
hexo new newpapername
```

然后在source/_post中打开markdown文件开始编辑，写完后输入下列指令可以看到已经更新

```
hexo clean && hexo g && hexo d
```

### Hexo基本配置

在文件根目录下的`_config.yml`就是整个hexo框架的配置文件了。可以在里面修改大部分的配置

#### 网站

| 参数        | 描述                             |
| ----------- | -------------------------------- |
| title       | 网站标题                         |
| subtitle    | 网站副标题                       |
| description | 网站描述                         |
| author      | 作者                             |
| language    | 网站使用的语言                   |
| timezone    | 网站时区，默认使用用户电脑的时区 |

#### 网址

| 参数               | 描述                   |
| ------------------ | ---------------------- |
| url                | 网址                   |
| root               | 网站根目录             |
| permalink          | 文章的永久链接格式     |
| permalink_defaults | 永久链接各部分的默认值 |

将`url`改成自己的网站域名，permalink即生成文章时的链接格式；如新建一个文章`test.md`，此时自动生成的地址是`http://yoursite.com/年/月/日/test`

以下是官方给出的示例，更多变量可以去官网上查找 [永久链接](https://hexo.io/zh-cn/docs/permalinks) 

| 参数                          | 结果                        |
| ----------------------------- | --------------------------- |
| :year/:month/:day/:title/     | 2013/07/14/hello-world      |
| :year-:month-:day-:title.html | 2013-07-14-hello-world.html |
| :category/:title              | foo/bar/hello-world         |

`theme`即选择的主题，默认为`landscape`，需要更换主题时可以在官网上下载，把主题的文件放在`theme`文件夹下，再修改这个参数就可以了；

接下来这个`deploy`就是网站的部署的，`repo`就是仓库(`Repository`)的简写。`branch`选择仓库的哪个分支

#### git分支进行多端工作

使用场景：部署博客和想要操作的文件不一样；利用git的分支系统进行多端工作，只需简单的配置和在github上将文件同步下来即可操作

##### 机制

`hexo d`上传部署到github的是hexo编译后的文件，是用来生成网页的，不包含源文件；即上传的是本地目录中自动生成的`.deploy_git`里的，其它文件都没有上传到github；故利用git的分支管理，将源文件上传到github的另一个分支即可

##### 操作步骤

1. 创建新分支

2. 在该仓库的settings中选择默认分支为新分支（这样同步时无需指定分支）

3. 在本地的任意目录下打开git bash

   ```
   git clone git@github.com:YourgithubName/YourgithubName.github.io.git
   ```

将其克隆到本地，把除了.git 文件夹外的所有文件都删掉，把之前我们写的博客源文件全部复制过来，除了`.deploy_git`。复制过来的源文件应该有一个`.gitignore`，用来忽略一些不需要的文件，如果没有的话，自己新建一个，在里面写上如下，表示这些类型文件不需要git：

```
   .DS_Store
   Thumbs.db
   db.json
   *.log
   node_modules/
   public/
   .deploy*/
```

   注意，若之前克隆过theme中的主题文件，则应把主题文件中的`.git`删掉

```
git add .
   git commit –m "add branch"
   git push 
```

   这样就上传完了，可以去github上看一看新分支有没有上传上去，其中`node_modules`、`public`、`db.json`已经被忽略掉了，没有关系，不需要上传的，因为在别的电脑上需要重新输入命令安装。

##### 更换电脑操作

和之前的环境搭建一样

```
sudo apt-get install git	#安装git

git config --global user.name "yourgithubname"
git config --global user.email "yourgithubemail"	#设置邮箱用户名

ssh-keygen -t rsa -C "youremail"
#生成后填到github和coding上（有coding平台的话）
#验证是否成功
ssh -T git@github.com
ssh -T git@git.coding.net #(有coding平台的话)	#设置ssh key

sudo apt-get install nodejs
sudo apt-get install npm		#安装nodejs

sudo npm install hexo-cli -g	#安装hexo

#此时无需初始化，直接在任意文件夹下
git clone git@github.com:YourgithubName/YourgithubName.github.io.git

#进入克隆到的文件夹
cd xxx.github.io
npm install
npm install hexo-deployer-git --save

#生成，部署
hexo g && hexo d
```

#### coding page部署实现国内外分流

若希望博客能被百度收录且更快的访问，则可以在国内的coding page做一个托管，这样在国内访问就是coding page，国外就走github page

1. 申请coding账户，新建项目

2. 添加ssh key，该步骤和github一样

3. 修改_config.yml

   ```
   deploy:
     type: git
     repo: 
       coding: git@git.coding.net:yourgithubname/yourgithubname.git,master
       github: git@github.com:yourgithubname/yourgithubname.github.io.git,master
   ```

4. 部署

   ```
   hexo g && hexo d
   ```

5. 开启coding pages服务，绑定域名

   ![](https://img-blog.csdnimg.cn/img_convert/10a1bbb64e8a644bafb91a2cc772e5a3.png)

6. 阿里云添加解析，将github的解析改为境外，将coding的解析设为默认

### Hexo添加功能

包括搜索的SEO，阅读量统计，访问量统计和评论系统等功能；参考 [visugar.com](http://visugar.com/2017/08/01/20170801HexoPlugins/)

#### SEO（Search Engine Optimization）优化

1. 登录百度站长平台添加网站

2. 提交链接

   使用npm自动生成网站的sitemap，然后将生成的sitemap提交到百度和其他搜索引擎

   ```
   npm install hexo-generation-sitemap --save
   npm install hexo-generation-baidu-sitemap --save
   ```

   在根目录下的_config.xml`中看看url有没有改成自己的

   重新部署后，就可以在public文件夹下看到生成的sitemap.xml和baidusitemap.xml，然后就可以向百度提交你的站点地图了，建议使用自动提交，自动提交又分为三种：主动推送、自动推送、sitemap

   - 自动推送：把百度生成的自动推送代码，放在主题文件`/layout/common/head.ejs`的适当位置，然后验证一下就可以了
   - sitemap：把两个sitemap地址，提交上去，看到状态正常就OK了
   - 隔一段时间去`site:<域名>`查看是否被收录

#### 添加百度统计

在[百度统计](https://tongji.baidu.com/)中，注册一下；将代码复制到`head.ejs`文件中，再进行安装检查

#### 文章阅读量统计leanCloud

[leanCloud](https://leancloud.cn/)，进去后注册一下，进入后创建一个应用：

![](https://img-blog.csdnimg.cn/img_convert/fda4f72f190ff5910ff790b05641096c.png)

在`存储`中创建Class，命名为Counter

![](https://img-blog.csdnimg.cn/img_convert/16843063bfd1ae6f2e743001bc06067e.png)

然后在设置页面看到应用Key`，在主题的配置文件中：

```
leancloud_visitors:
  enable: true
  app_id: yourid
  app_key: yourkey
```

在`article.ejs`中适当的位置添加如下，设置文章阅读量统计显示位置

```
阅读数量:<span id="<%= url_for(post.path) %>" class="leancloud_visitors" data-flag-title="<%- post.title %>"></span>次
```

然后在`footer.ejs`的最后添加如下，重新部署即可：

```
<script src="//cdn1.lncld.net/static/js/2.5.0/av-min.js"></script>
<script>
    var APP_ID = '你的app id';
    var APP_KEY = '你的app key';
    AV.init({
        appId: APP_ID,
        appKey: APP_KEY
    });
    // 显示次数
    function showTime(Counter) {
        var query = new AV.Query("Counter");
        if($(".leancloud_visitors").length > 0){
            var url = $(".leancloud_visitors").attr('id').trim();
            // where field
            query.equalTo("words", url);
            // count
            query.count().then(function (number) {
                // There are number instances of MyClass where words equals url.
                $(document.getElementById(url)).text(number?  number : '--');
            }, function (error) {
                // error is an instance of AVError.
            });
        }
    }
    // 追加pv
    function addCount(Counter) {
        var url = $(".leancloud_visitors").length > 0 ? $(".leancloud_visitors").attr('id').trim() : 'icafebolger.com';
        var Counter = AV.Object.extend("Counter");
        var query = new Counter;
        query.save({
            words: url
        }).then(function (object) {
        })
    }
    $(function () {
        var Counter = AV.Object.extend("Counter");
        addCount(Counter);
        showTime(Counter);
    });
</script>
```