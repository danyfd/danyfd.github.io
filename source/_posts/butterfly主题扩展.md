---
title: butterfly主题扩展
date: 2023-10-13 18:55:38
tags: "hexo"
categories: "hexo"
---

#### Butterfly主题配置

##### 安装步骤

- 移动到博客所在根目录

- 下载主题代码到themes中

  ```
  # 方式1：git方式安装
  git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
  
  # 方式2：npm方式安装
  npm i hexo-themebutterfly
  ```

- 修改hexo的_config.yml文件，将主题改为butterfly

  ```
  theme: butterfly
  ```

  若重启、打开页面报如下错误，因没有**pug**以及 **stylus** 的渲染器，需要安装

  ```
  extends includes/layout.pug block content include ./includes/mixins/post-ui.pug #recent-posts.recent-posts +postUI include includes/pagination.pug
  ```

- 安装pub & stylus

  ```
  npm install hexo-renderer-pug hexo-renderer-stylus --save
  ```

- 最后重启hexo即可

##### 修改主题配置

主题的配置在 **/www/wwwroot/blog/themes/butterfly/_config.yml** 文件中

##### 鼠标样式更改

首先准备想要生成鼠标样式的图片，调整对应的像素大小，将图片类型转换为.cur（或直接在[鼠标指针 - 光标 - 电脑鼠标指针下载 - 致美化 - 漫锋网 (zhutix.com)](https://zhutix.com/tag/cursors/)中找想要的样式），在主题路径source的css文件夹中新建一个mouse文件夹用于存放这些图片

###### 代码引入

在butterfly主题的`_config.yml`文件中修改如下：

![](https://s1.ax1x.com/2023/01/28/pSUjNCV.png)

在主题路径source中的css文件夹新建一个`mouse.css`文件，加入类似下列内容：

```css
body,
html {
    cursor: url('./mouse/alternate.cur'), auto !important;
}

/* 悬停图片时的鼠标指针 */
/* 选择链接标签时的鼠标指针 */
/* 选中输入框时的鼠标指针 */
/* 悬停按钮时的鼠标指针 */
/* 悬停列表标签时的鼠标指针 */
/* 悬停页脚链接标签（例如页脚徽标）时的鼠标指针 */
/* 悬停页码时的鼠标指针 */
/* 悬停菜单栏时的鼠标指针 */
img,
a:hover,
input:hover,
button:hover,
i:hover,
#footer-wrap a:hover,
#pagination .page-number:hover,
#nav .site-page:hover {
    cursor: url('./mouse/link.cur'), auto !important;
}
```

重新刷新即可看到鼠标样式已经生效

##### 搜索功能

###### 安装插件

```
npm install hexo-generator-search --save
```

###### 更改配置

在`主配置文件`中插入类似下列代码：

```yaml
search:
  path: search.xml
  field: all # post:文章范围、page:页面范围、all:覆盖所有
  content: true # 内容是否包含每一篇文章的全部内容
  template:  # ./search.xml 指定定制的XML模板
```

将butterfly配置文件中`local_search`的`enable`参数设为true表示开启本地搜索

##### 主题图片的更改

<font color=red>图片的大小最好不要超过3M，否则会影响文章的打开速度</font>

###### 网站logo

将图片的名称更改为`favicon.png`替换掉source下的img中的原logo即可

###### 头像修改

将图片的名称更改为`avatar.png`加入source下的img中，并将对应参数修改：

```yaml
# Avatar (头像)
avatar:
  img: /img/avatar.png
  #是否一直旋转
  effect: false
```

##### 主图修改

将图片加入source下的img中，在配置文件中修改`index_img`参数的值为该路径

- `default_top_img:`后面是当图片显示错误时的代替图片
- `index_img:`后是进入网站时显示的图像
- `archive_img:`是点击头像下面的文章进入的页面的顶部图
- `tag_img:`是点击头像下面的标签进入的页面的顶部图
- `category_img:`是点击头像下面的分类进入的页面的顶部图

###### 随机图片的匹配

此时设置了每一个分类，每一个标签页的图片，但是tags主页与categories主页都是原图，需要调整cover属性：

```yaml
cover:
  # display the cover or not
  #首页是否显示文章封面
  index_enable: true
  #侧边栏是否显示文章封面
  aside_enable: true
  #归档页是否显示文章封面
  archives_enable: true
  # the position of cover in home page
  # left/right/both
  position: both
  #当没有设置封面的时候，使用以下的照片
  default_cover:
     - 图片地址1
     - 图片地址2
     .....

```

###### 错误页面的匹配

```yaml
# Replace Broken Images
#在这里可以自定义所有图片不能显示的情况下显示的图片
error_img:
  flink: /img/404.jpg
  post_page: /img/404.jpg

# A simple 404 page
error_404:
  enable: true
  subtitle: '页面丢失，联系博主进行解决吧'
  #图片的背景
  background: 链接地址

```

##### 主页文章节选

主页文章节选只支持自动节选和文章页description，修改主题配置文件：

```yaml
index_post_content:
  method: 3	#1.只显示description，2.优先description，3.只显示自动节选，4.不显示文章内容
  length: 500 # if you set method to 2 or 3, the length need to config

```

##### 文章封面

文章封面的获取顺序`Front-matter的cover`>`配置文件的default_cover`

##### 字数统计

```
npm install hexo-wordcount --save

```

在butterfly的配置文件中将wordcount的enable参数设置为true

##### 标题栏标签完善

hexo自带的标签包括分类，首页，标签等，且其默认配置了对应的目录：

![](https://s1.ax1x.com/2023/01/30/pSdTrqI.png)

在source目录执行下面命令，建立标签页，其他的categories等也是相同的操作

```
hexo new page "tags"

```

在生成路径的文件中增加type属性，内容与title相同并用双引号括住

```
title: tags
date: 2023-01-30 11:27:08
type: "tags"

```

打开主题路径下的配置文件配置对应的标签及相应图标：

![](https://s1.ax1x.com/2023/01/30/pSdTzLR.png)

图标可以在阿里图标官网选择：[iconfont-阿里巴巴矢量图标库](https://www.iconfont.cn/)

找到想要的图标加入到购物车，添加至项目，将项目下载至本地，在主题的source/css文件下将红色标注的移动过去

![](https://moonshuo.cn//images/202211011545280.png)

原css目录下的mouse.css（先前添加的鼠标样式）修改为style.css（将hexo配置文件中的inject参数也做对应修改）；将该iconfont.css中的内容添加进去：

![](https://s1.ax1x.com/2023/01/30/pSwYKbV.png)

调整标题栏格式，重新部署即可：

![](https://s1.ax1x.com/2023/01/30/pSwYlUU.png)

##### 设置透明度

在`style.css`中插入类似下列代码，背景图片最好与主图是同一张图片，否则会显得突兀，关于渐变色的设置可以在[Gradients](https://uigradients.com/#DigitalWater)中选择：

```css
/* 背景样式 */
#web_bg {
	background-image: url("../img/index.jpg"),
		linear-gradient(60deg, rgba(255, 165, 150, 0.5) 5%, rgba(0, 228, 255, 0.35))
}

/* 侧边栏个人信息卡片动态渐变色 */
#aside-content>.card-widget {
	background: linear-gradient(-45deg,
			#e8d8b9,
			#eccec5,
			#a3e9eb,
			#bdbdf0,
			#eec1ea);
	box-shadow: 0 0 5px rgb(66, 68, 68);
	position: relative;
	background-size: 400% 400%;
	-webkit-animation: Gradient 10s ease infinite;
	-moz-animation: Gradient 10s ease infinite;
	animation: Gradient 10s ease infinite !important;
}

@-webkit-keyframes Gradient {
	0% {
		background-position: 0% 50%;
	}

	50% {
		background-position: 100% 50%;
	}

	100% {
		background-position: 0% 50%;
	}
}

@-moz-keyframes Gradient {
	0% {
		background-position: 0% 50%;
	}

	50% {
		background-position: 100% 50%;
	}

	100% {
		background-position: 0% 50%;
	}
}

@keyframes Gradient {
	0% {
		background-position: 0% 50%;
	}

	50% {
		background-position: 100% 50%;
	}

	100% {
		background-position: 0% 50%;
	}
}

/* 个人信息Follow me按钮 */
#aside-content>.card-widget.card-info>#card-info-btn {
	background-color: #3eb8be;
	border-radius: 8px;
}

#aside-content>.sticky_layout>.card-widget,
/*分类页面*/
.layout>#page,
/*时间轴页面*/
.layout>#archive {
	background: rgba(255, 255, 255, .9);
}

```

##### 页脚设置

进入`….themes\butterfly\source\css\_layout\footer.styl`，把blue那行删掉

###### 更改底层文字

在js文件夹下新建foot.js文件，复制下列代码并添加[jquery.js文件](https://code.jquery.com/jquery-3.6.1.min.js)：

```js
//动态心跳
$(document).ready(function(e){
    $('.copyright').html('©2023 <i class="fa-fw fas fa-heartbeat card-announcement-animation cc_pointer"></i> By dany\'s blog');
})

$(document).ready(function(e){
    show_date_time();
})

//本站运行时间
function show_date_time(){
$('.framework-info').html('本站已运行<span id="span_dt_dt" style="color: #fff;"></span>');
window.setTimeout("show_date_time()", 1000);
BirthDay=new Date("1/28/2023 0:0:0");
today=new Date();
timeold=(today.getTime()-BirthDay.getTime());
sectimeold=timeold/1000
secondsold=Math.floor(sectimeold);
msPerDay=24*60*60*1000
e_daysold=timeold/msPerDay
daysold=Math.floor(e_daysold);
e_hrsold=(e_daysold-daysold)*24;
hrsold=Math.floor(e_hrsold);
e_minsold=(e_hrsold-hrsold)*60;
minsold=Math.floor((e_hrsold-hrsold)*60);
seconds=Math.floor((e_minsold-minsold)*60);
span_dt_dt.innerHTML='<font style=color:#3eb8be>'+daysold+'</font> 天 <font style=color:#f391a9>'+hrsold+'</font> 时 <font style=color:#fdb933>'+minsold+'</font> 分 <font style=color:#a3cf62>'+seconds+'</font> 秒';
}

```

在主题配置文件中配置foot和jquery文件：

```yaml
inject:
  head:
    - <link rel="stylesheet" href="/css/style.css">
  bottom:
    - <script src="/js/jquery.js"></script>
    - <script src="/js/foot.js"></script>

```

##### 加载动画（butterfly4.5以上方案）

修改 `themes/butterfly/layout/includes/loading/fullpage-loading.pug`

```
#loading-box(onclick='document.getElementById("loading-box").classList.add("loaded")')
  .loading-bg
    div.loading-img
    .loading-image-dot

script.
  const preloader = {
    endLoading: () => {
      document.body.style.overflow = 'auto';
      document.getElementById('loading-box').classList.add("loaded")
    },
    initLoading: () => {
      document.body.style.overflow = '';
      document.getElementById('loading-box').classList.remove("loaded")

    }
  }
  window.addEventListener('load',()=> { preloader.endLoading() })

  if (!{theme.pjax && theme.pjax.enable}) {
    document.addEventListener('pjax:send', () => { preloader.initLoading() })
    document.addEventListener('pjax:complete', () => { preloader.endLoading() })
  }

```

修改`themes/butterfly/layout/includes/loading/index.pug`

```
if theme.preloader.source === 1
  include ./fullpage-loading.pug
else if theme.preloader.source === 2
  include ./pace.pug
else
  include ./fullpage-loading.pug
  include ./pace.pug

```

修改`themes/butterfly/source/css/_layout/loading.styl`, 注意其中颜色代码`--anzhiyu-card-bg`等需自行替换为自己的色值

```stylus
if hexo-config('preloader')
  .loading-bg
    display: flex;
    width: 100%;
    height: 100%;
    position: fixed;
    background: var(#fff);
    z-index: 1001;
    opacity: 1;
    transition: .3s;

  #loading-box
    .loading-img
      width: 100px;
      height: 100px;
      border-radius: 50%;
      margin: auto;
      border: 4px solid #f0f0f2;
      animation-duration: .3s;
      animation-name: loadingAction;
      animation-iteration-count: infinite;
      animation-direction: alternate;
    .loading-image-dot
      width: 30px;
      height: 30px;
      background: #6bdf8f;
      position: absolute;
      border-radius: 50%;
      border: 6px solid #fff;
      top: 50%;
      left: 50%;
      transform: translate(18px, 24px);
    &.loaded
      .loading-bg
        opacity: 0;
        z-index: -1000;

  @keyframes loadingAction
    0% {
        opacity: 1;
    }

    100% {
        opacity: .4;
    }

```

在自定义css文件中加入如下代码，图片最好转换为base64编码：

```css
.loading-img {
  background: url(../img/avatar.png) no-repeat center center;
  background-size: cover;
}

```

最后将主题配置文件中的`preloader`修改为`true`

##### 导航栏修改

在自定义css文件中加入下列代码：

```css
/* 导航栏修改 */
/* 圆角隐藏 */
ul.menus_item_child {
	border-radius: 5px;
}
  
/* 一级菜单居中 */
#nav .menus_items {
	position: absolute;
	width: fit-content;
	left: 50%;
	transform: translateX(-50%);
	display: flex;
	flex-direction: row;
	justify-content: center;
	align-items: center;
	height: 60px;
}

#nav a:hover {
	background: var(--anzhiyu-main);
	transition: 0.3s;
}
  
#nav-totop:hover .totopbtn i {
	opacity: 1;
}
#nav-totop #percent {
	font-size: 12px;
	background: var(--anzhiyu-white);
	color: var(--anzhiyu-main);
	width: 25px;
	height: 25px;
	border-radius: 35px;
	display: flex;
	justify-content: center;
	align-items: center;
	transition: 0.3s;
}
.nav-fixed #nav-totop #percent,
.page #nav-totop #percent {
	background: var(--font-color);
	color: var(--card-bg);
	font-size: 13px;
}
  
#nav-totop {
	width: 35px;
}
#page-header:not(.is-top-bar) #percent {
	transition: 0.3s;
}
#page-header:not(.is-top-bar) #nav-totop {
	width: 0;
	opacity: 0;
	transition: width 0.3s, opacity 0.2s;
	margin-left: 0 !important;
}
#nav-totop #percent {
	font-weight: 700;
}
#nav-totop:hover #percent {
	opacity: 0;
	transform: scale(1.5);
	font-weight: 700;
}
#page-header #nav #nav-right div {
	margin-left: 0.5rem;
	padding: 0;
}
  
#nav-totop {
	display: flex;
	align-items: center;
	justify-content: center;
	transition: 0.3s;
}
.nav-button {
	cursor: pointer;
}
div#menus {
	display: flex;
	align-items: center;
}
#page-header #nav .nav-button a {
	height: 35px;
	width: 35px;
	display: flex;
	align-items: center;
	justify-content: center;
}
  
#nav .site-page {
	padding-bottom: 0px;
}
#nav *::after {
	background-color: transparent !important;
}

```

##### Aplayer音乐插件

运行以下指令安装`hexo-tag-aplayer`插件：

```
npm install hexo-tag-aplayer --save

```

在hexo配置文件中新增配置项：

```yaml
aplayer:
	meting: true
	asset_inject: false

```

修改主题配置文件中关于Aplayer的配置内容：

```yaml
aplayerInject:
	enable: true
  	per_page: true
  	
inject:
  head:
  bottom:
    - <div class="aplayer no-destroy" data-id="5183531430" data-server="netease" data-type="playlist" data-fixed="true" data-mini="true" data-listFolded="false" data-order="random" data-preload="none" data-autoplay="false" muted></div>

```

在`\themes\butterfly\source\css`中新建一个`custom.css`添加css样式使Aplayer自动缩进隐藏，且在主题配置文件`inject`中加入该文件

```css
.aplayer.aplayer-fixed.aplayer-narrow .aplayer-body {
  left: -66px !important;
  /* 默认情况下缩进左侧66px，只留一点箭头部分 */
}

.aplayer.aplayer-fixed.aplayer-narrow .aplayer-body:hover {
  left: 0 !important;
  /* 鼠标悬停是左侧缩进归零，完全显示按钮 */
}

```

```html
<link rel="stylesheet" href="/css/custom.css"  media="defer" onload="this.media='all'">

```

关于Aplayer的部分参数配置如下：

| option        | description                                             |
| ------------- | ------------------------------------------------------- |
| data-id       | 必填，歌单id                                            |
| data-server   | 必填，服务商：netease, tencent, kugou, xiami, baidu等   |
| data-type     | 必填，歌单类型：song, playlist, album, search, artist等 |
| data-fixed    | enable fixed mode                                       |
| data-mini     | enable mini mode                                        |
| data-autoplay | audio autoplay                                          |
| data-theme    | `#2980b9`，main color                                   |
| data-preload  | values: 'none', 'metadata', 'auto'                      |
| data-order    | player play order, values: 'list', 'random'             |