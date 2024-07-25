---
title: Github+HEXO实现网站DIY
tag: Github
---

# Work No.6

### 一、创建github.page网页

- 点击Repositories，选择新建Repositories仓库
- 在仓库名字Repository name栏填写网页的网址，后缀需要添加 .github.io
  - 例如 webname.github.io

![image-20240724153741084](/images/image-20240724153741084.png)

- 之后点击 Create repository ，创建该仓库，其他无需调整
- 之后需要创建一个 index.html 文件，用于后续能够选取分支

![image-20240724154237430](/images/image-20240724154237430.png)

- 文件名字输入 index.html，内容为 `<hl> hello </hl>` 这个文件会初始化成一个网页，网页的内容只有 hello，这个随便填就行，后续会用模板，不用考虑这个

![image-20240724154427037](/images/image-20240724154427037.png)

- 进入仓库，选择 Setting ，点击 pages ，在 branch 上选择 main 分支（刚刚创建文件也是为了创建这个分支），点击 save 保存，之后等待几分钟，就会出现创建的网址，点击 visit site 即可进入建立的网页

![image-20240724155135615](/images/image-20240724155135615.png)

- 完成这一步之后，就可以得到一个”hello“的网页，该网页没有任何排版和格式

### 二、配置HEXO并进行部署

- 打开git bash，执行如下命令安装hexo

```
	$ npm install -g hexo-cli # 此命令完成对 hexo 的安装
```

- 如果没有安装npm、git、node.js的话
  - node.js参考https://blog.csdn.net/yaorongke/article/details/119084295
  - git安装参考https://blog.csdn.net/mukes/article/details/115693833
- 之后随便在想要的地方建一个文件夹，这个文件夹的作用是用于博客文章存放和仓库交互，例如<blog>
- 进入该文件，将该文件初始化

```
	$ cd blog # 进入该文件夹
	$ hexo init # 该命令完成 hexo 在本地博客目录的初始化
```

- 之后生成网页的静态文件，如果需要更新网页的配置，需要先清理，在重新生成

```
	$ hexo g # 生成静态文件，生成的文件在public文件夹里面
	
	$ hexo clean # 清除生成的静态文件
```

- 可以进行本地预览，本地预览的ctrl+c会退出运行，所以用右键复制网址到网页上预览

```
	$ hexo s # 开启本地预览
```

- 安装部署插件

```
	$ npm install hexo-deployer-git --save # 安装部署插件
```

- `hexo` 有 2 种 `_config.yml` 文件，一个是根目录下的全局的 `_config.yml`，一个是各个主体 `theme` 下的 `_config.yml`。将前者称为站点配置文件， 后者称为主题配置文件。打开根目录下站点配置文件 `_config.yml`，配置有关 `deploy`和`url `的部分：

```
	# URL
	## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
	url: GitHub pages的网址，例如https://GitHub用户名/仓库名               //修改这个地方，需要把网址改对，否则网页没有格式
	permalink: :year/:month/:day/:title/
	permalink_defaults:
	pretty_urls:
	  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
	  trailing_html: true # Set to false to remove trailing '.html' from permalinks
	
	# Deployment
	## Docs: https://hexo.io/docs/one-command-deployment
	deploy:
	  type: git
	  repo: git@github.com:GitHub账户名/仓库名.github.io.git
	  branch: master #如果分支是main记得修改，需要推送到哪个分支，就选择哪个分支
```

![image-20240724161404111](/images/image-20240724161404111.png)

- 部署到 GitHub

```
	$ hexo d # 将public里面的文件发送到云端GitHub上面
```

- 之后就可以刷新网页看到上传上去的网页，就拥有了自己的网页

### 三、主题更换和选择

- 我们上面用的是系统默认的主题，比较丑，所以我们可以使用hexo去挑选我们想要的主题类型
  - [开始使用 | Hexo Fluid 用户手册 (fluid-dev.github.io)](https://fluid-dev.github.io/hexo-fluid-docs/start/#更新主题)
- [Themes | Hexo](https://hexo.io/themes/)可以在这个网址里面挑选自己喜欢的
- 本文挑选了fluid进行下载，选择直接下载zip包或者`git clone https://github.com/fluid-dev/hexo-theme-fluid.git`
- 将下载的包解压放到 themes 文件夹里面，文件夹的名字改成fluid，最好不要文件夹套娃，如果出现，则将里面的文件全部移出来
- 继续修改博客根目录下的 _config.yml 文件，注意，不是fluid里面的_config.yml文件

```
	language: zh-CN  # 指定语言，会影响主题显示的语言，按需修改
	
	theme: fluid  # 指定主题
```

- 首次使用主题的「关于页」需要手动创建，打开git，输入代码

```
	$ hexo new page about
```

- 创建成功后修改 `/source/about/index.md`，添加 `layout` 属性，`layout: about` 必须存在，并且不能修改成其他值，否则不会显示头像等样式

```
	---
	title: 标题
	layout: about
	---
```

- 之后重复上面的步骤先进行静态页面清除，再生成，之后上传到GitHub仓库中进行页面更新，等待几分钟后，如下图所示

<img src="/images/image-20240724162639684.png" alt="image-20240724162639684" style="zoom:50%;" />

### 四、文档上传

- 使用Typora进行.md文档的编写，非常好用的一个编辑器，绿色版可以网上搜索
- 之后将md文件放到博客文件夹下面` source/_posts/ `文件夹下面
- 之后使用vim编辑器对文档进行title和tag的编辑
  - [Linux vi/vim | 菜鸟教程 (runoob.com)](https://www.runoob.com/linux/linux-vim.html)

- 进入md文件，首先点击 i 进入编辑模式，在开头位置输入，其中title为文章的名字，tag为文章的标签，方便进行检索

```
	---
	title： hello word
	tag： write
	---
```

- 编辑完成点击 Esc 退出报错，直接输入 :wq 保存编辑并且退出，完成文件的标记修改
- 之后回到git bash中输入

	$ hexo g # 生成静态文件，生成的文件在public文件夹里面
	
	$ hexo d # 将生成的文件导入GitHub站点

- 到此，等待几分钟后，你的文章就在你的网页上面出现了
