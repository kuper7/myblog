
## Git连接GitHub仓库，同步上传图片及CSDN外链图片转存失败解决方案

​		最近在使用markdown编辑器写博客时，发现了一个问题，就是本地图片的上传问题，查阅发现，可以上传本地图片，使用外链进行调用，下面介绍Git连接GitHub仓库，同步上传文件及图片的步骤，以及之后解决CSDN外链图片失败问题的步骤。

### 一、Git连接GitHub仓库，同步上传图片

#### 1.创建仓库

​		登陆GitHub后，点击头像隔壁的“+”按钮，会下拉显示几个选项，选择New Repository创建仓库。详细步骤可见：https://www.cnblogs.com/ghm-777/p/11433425.html。

#### 2.下载git并同步和上传文件到github

​		使用git的原因是，这样我们就可以将图片等内容放在一个项目文件夹里，通过git一次性上传到github仓库，实现同步。

同步和上传文件的详细步骤可见：https://www.cnblogs.com/cxk1995/p/5800196.html。

### 二、解决csdn外链图片转存失败

#### 1.问题描述：

​		已经完成了图片的上传，但在CSDN的**markdown编辑器**写博客时，依旧出现了“**外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传**”

#### 2.原因分析：

​		墙内无法顺畅的访问github远程仓库中的图片，加载超时出错

#### 3.解决方案：

​		将图片设置为共有之后开启github pages功能，获得一个**域名**如**https://pangcong1117.github.io/**，使用**域名加上图片路径即可成功的外链图片**，以我的仓库中的”语音评估模式图.png“为例，存储仓库名为myblog，图片路径为“**/myblog/images/语音评估模式图.png**"。markdown外链方法如下：

```
<img src="https://pangcong1117.github.io/myblog/images/语音评估模式图.png"/>
```

​		如果想使博客中图片居中显示，只需在前面加上center，方法如下：

```
<center><img src="https://pangcong1117.github.io/myblog/images/语音评估模式图.png"/>
```

