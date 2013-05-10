# Quartz快速入门

[http://quartz-scheduler.org/overview/quick-start](http://quartz-scheduler.org/overview/quick-start)

欢迎来到这个Quartz快速入门的教程。我们今天会介绍的有以下内容：

- <a href="#downloadAndInstall">下载和安装Quartz</a>
- <a href="#configure">配置Quartz适用于你自己的需要</a>
- <a href="#start">启动一个示例程序</a>

在你更熟悉Quartz任务调度器了之后，你可以看看更多更高级的功能，譬如企业级应用的功能，让job和trigger运行在Terracotta客户端中，而不是随意选择的客户端。

## <a name="downloadAndInstall">下载和安装</a>

首先，下载最近的最稳定的[版本](http://quartz-scheduler.org/downloads/)，不一定要注册。解压缩安装，这样你的应用程序可以使用Quartz。

### Quartz Jar文件

Quartz包中有很多jar，在根目录中。最主要的一个库叫作quartz-all-xxx.jar(xxx是版本号)。为了使用Quartz，你需要把这个jar放在你应用程序的classpath里。

你下载完之后，解压缩包，将quartz-all-xxx.jar这个文件取出来，放到任何你喜欢的地方。

因为我主要将Quartz用在应用服务器的环境中，所以我希望将Quartz jar放在我的程序中(也就是.ear或者.war中)。然而，如果你希望多个应用都能够使用Quartz，那么就放在你的应用服务器的classpath好了。如果你只有一个独立的应用，那么就放在应用的classpath中，和其他的jar放在一起。

Quartz需要依赖很多第三方的库(以jar的形式)，这些jar都放在了`lib`目录下，要使用所有的功能的话，需要将所有的jar都放在classpath中。如果是一个独立的应用，建议你将所有的jar都放在classpath中。而如果在应用服务器的环境下，你可能已经有其中的某些jar了，这时你需要进行选择，只将某些jar放到classpath下。

> 在应用服务器的环境下，你需要注意可能会有一些奇怪的结果，是由于你包含了同一个jar的不同版本。例如，Weblogic包含了J2EE的实现，但是不同于servlet.jar。所以，你最好将servlet.jar从classpath中移出来，以便知道真正用到了哪些类。

### Properties文件

Quartz使用quartz.properties作为配置文件。最开始的时候可能不是必须的，但是要进行一些基本的配置，你还是需要将其放入classpath里。

再说一次，还是看你是什么环境。我个人而言用WebLogic，所以我将所有的配置文件(包括quartz.properties)放在应用程序根文件夹下。当我将所有的程序都打包到.ear时，这些配置文件都打包到.jar文件里，并包含在最后的.ear文件中。这将自动将quartz.properties文件包含在classpath下。

如果你的应用是.war，那么你可能要将quartz.properties放在WEB-INF/classes文件架下。

## <a name="configure">配置Quartz</a>

配置是最重要的部分。Quartz是一个配置性很强的应用，最好的配置方法就是修改quartz.properties。

在解压缩包里已经含有了一些配置文件的范例，你可以查看examples/ 文件夹。不过我建议你创建你自己的quartz.properties，而不是直接拷贝范例文件然后删掉你不要的配置。创建你自己的配置文件能让你学到更多的东西.

查看所有有关配置的文档，请查看[Quartz配置参考文档](http://quartz-scheduler.org/documentation/quartz-2.1.x/configuration)。

最基本的配置文件quartz.properties如下：

    org.quartz.scheduler.instanceName = MyScheduler
    org.quartz.threadPool.threadCount = 3
    org.quartz.jobStore.class = org.quartz.simpl.RAMJobStore

这个配置有如下含义：　　

- org.quartz.scheduler.instanceName - 调度器的名字是"MyScheduler"。
- org.quartz.threadPool.threadCount - 在线程池中有三个线程。这就意味着最多有3个job可以同时运行。
- org.quartz.jobStore.class - 所有的Quartz的数据保存在内存中（而不是数据库中）。尽管可能你有数据库，并且希望配合Quartz一起使用。但我仍旧建议你首先使用`RAMJobStore`，再去接触数据库。

##　<a name="start">启动一个示例程序</a>
现在是时候启动程序了。下面的代码包含了一个调度器，首先启动它，然后关闭它。

QuartzTest.java：

    import org.quartz.Scheduler;
    import org.quartz.SchedulerException;
    import org.quartz.impl.StdSchedulerFactory;
    import static org.quartz.JobBuilder.*;
    import static org.quartz.TriggerBuilder.*;
    import static org.quartz.SimpleScheduleBuilder.*;


    public class QuartzTest {

        public static void main(String[] args) {

            try {
                // Grab the Scheduler instance from the Factory 
                Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

                // and start it off
                scheduler.start();

                scheduler.shutdown();

            } catch (SchedulerException se) {
                se.printStackTrace();
            }
        }
    }

一旦你通过StdSchedulerFactory.getDefaultScheduler()获得了一个调度器,你的程序会一直运行，直到调用scheduler.shutdown()方法。

注意还有一些`import static ...`，下面的代码你会见到它们的作用。

如果你没有设置好logging输出日志，所有的日志都回输出到控制台，可能如下：

    [INFO] 21 Jan 08:46:27.857 AM main [org.quartz.core.QuartzScheduler]
    Quartz Scheduler v.2.0.0-SNAPSHOT created.

    [INFO] 21 Jan 08:46:27.859 AM main [org.quartz.simpl.RAMJobStore]
    RAMJobStore initialized.

    [INFO] 21 Jan 08:46:27.865 AM main [org.quartz.core.QuartzScheduler]
    Scheduler meta-data: Quartz Scheduler (v2.0.0) 'Scheduler' with instanceId 'NON_CLUSTERED'
      Scheduler class: 'org.quartz.core.QuartzScheduler' - running locally.
      NOT STARTED.
      Currently in standby mode.
      Number of jobs executed: 0
      Using thread pool 'org.quartz.simpl.SimpleThreadPool' - with 50 threads.
      Using job-store 'org.quartz.simpl.RAMJobStore' - which does not support persistence. and is not clustered.


    [INFO] 21 Jan 08:46:27.865 AM main [org.quartz.impl.StdSchedulerFactory]
    Quartz scheduler 'Scheduler' initialized from default resource file in Quartz package: 'quartz.properties'

    [INFO] 21 Jan 08:46:27.866 AM main [org.quartz.impl.StdSchedulerFactory]
    Quartz scheduler version: 2.0.0

    [INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
    Scheduler Scheduler_$_NON_CLUSTERED started.

    [INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
    Scheduler Scheduler_$_NON_CLUSTERED shutting down.

    [INFO] 21 Jan 08:46:27.866 AM main [org.quartz.core.QuartzScheduler]
    Scheduler Scheduler_$_NON_CLUSTERED paused.

    [INFO] 21 Jan 08:46:27.867 AM main [org.quartz.core.QuartzScheduler]
    Scheduler Scheduler_$_NON_CLUSTERED shutdown complete.

你可以在start()和shutdown()之间添加你想要完成的工作。

    // define the job and tie it to our HelloJob class
    JobDetail job = newJob(HelloJob.class)
        .withIdentity("job1", "group1")
        .build();

    // Trigger the job to run now, and then repeat every 40 seconds
    Trigger trigger = newTrigger()
        .withIdentity("trigger1", "group1")
        .startNow()
        .withSchedule(simpleSchedule()
                .withIntervalInSeconds(40)
                .repeatForever())            
        .build();

    // Tell quartz to schedule the job using our trigger
    scheduler.scheduleJob(job, trigger);

（也许你需要在调用shutdown()之前一段时间，以便job可以被顺利触发和执行。例如，你可以增加一行代码`Thread.sleep(60000) `）。

现在执行它吧。