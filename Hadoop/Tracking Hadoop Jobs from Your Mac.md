英文原文: [Cloudera](http://blog.cloudera.com/blog/2013/05/tracking-hadoop-jobs-from-your-mac-theres-an-app-for-that/)，编译：[ImportNew](http://www.importnew.com) - [Royce Wong](http://www.importnew.com/author/roycewong)

## 专为Mac本跟踪Hadoop 任务的应用

先把感谢送给Etsy开发者Brad Greenlee ([@bgreenlee](https://twitter.com/bgreenlee))。他开发的针对JobTracer的Mac OS应用非常棒！

[JobTracker.app](http://bgreenlee.github.io/JobTracker)是一个针对Hadoop JobTracker的Mac菜单栏应用。它提供了对Mapreduce 任务启动、完成、失败的通知，可以通过它轻松访问这些任务的详细页面。

当我在[Etsy](https://www.etsy.com/)进行Apache Hadoop 的开发工作时，我发现为了查看任务的执行进度，我不得不不断的查看Jobtracker页面，这占据了非常多的时间(话说译者坚决同意)。当时我们尝试着去解决这个问题，写了一个[Scalding](https://github.com/twitter/scalding) 工作流监听器，将已经完成和失败的任务发布给IRC，但是这个有点烦。所以我写了这个JobTracker.app。

### 安装和使用

你可以从[GitHub项目页面](https://github.com/bgreenlee/JobTracker)下载二进制文件。解压之后将它放到你的应用文件夹里。运行时，菜单栏会多出一个小帽子的图标(![pith_helmet](images/pith_helmet_sm.png))。点击它就会看到下面的菜单。

![jobtracker_menu.png](images/jobtracker_menu.png)

首先你必须在Preferences里输入你的JobTracker URL地址:

![perferences](images/perferences.png)

默认情况下，它会跟踪所有任务。很可能你不希望这样，所以将你的用户名和其他你想查看的用户名输入到"Usernames to track"文本框中，多个使用英文逗号隔开即可。

注意应用仅在Etsy内部使用的Hadoop版本测试过。这个APP使用了有点可怕的方式获取JobTracker数据（通过解析JobTracker页面，因为当前除了通过Java程序还没有API去访问JobTracker），在不同的版本上应用可能会失败。如果你想使用它但没有成功，在GitHub上[提交一个issue](https://github.com/bgreenlee/JobTracker/issues)，我会为你解决。

### 未来发展

待开发的特性列表中，下步将会是[同时跟踪多个集群](https://github.com/bgreenlee/JobTracker/issues/2)。如果你有任何请求，请[告诉我](brad@etsy.com)。






