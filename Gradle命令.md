## Gradle命令学习

### 1. Gradle准备

#### 1.1 Gradle安装目录介绍(windows版)

- docs:  API,DSL,指南等文件
- getting-started.html:  入门链接
- init.d:  gradle的初始化脚本目录
- lib:  相关库
- LICENSE:  
- NOTICE:
- media:  一些icon资源
- samples:  示例
- src:  源文件
- bin:  可执行脚本文件

#### 1.2 Gradle安装配置（windows版）

- windows版本只需要将解压的gradle目录  ....\bin 添加到环境变量path中即可

### 2.Gradle命令操作

#### 2.1 Gradle系统任务

- 查看gradle系统自带的任务： <kbd>gradle tasks</kbd>, 该命令可以查看gradle自带的命令以及说明

```powershell

> Task :tasks

------------------------------------------------------------
Tasks runnable from root project
------------------------------------------------------------

Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.

Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradleTest'.
components - Displays the components produced by root project 'gradleTest'. [incubating]
dependencies - Displays all dependencies declared in root project 'gradleTest'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradleTest'.
dependentComponents - Displays the dependent components of components in root project 'gradleTest'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'gradleTest'. [incubating]
projects - Displays the sub-projects of root project 'gradleTest'.
properties - Displays the properties of root project 'gradleTest'.
tasks - Displays the tasks runnable from root project 'gradleTest'.

To see all tasks and more detail, run gradle tasks --all

To see more detail about a task, run gradle help --task <task>

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

#### 2.2 Gradle日志和堆栈信息

- 日志分类（级别从上到下）：

  - ERROR: 错误消息

  - QUIET: 重要消息

  - WARNING: 警告消息

  - LIFECYCLE: 进度消息

  - INFO: 信息消息

  - DEBUG: 调试消息

- 日志的使用: 任务前加 -首字母或-小写 ，例如：
  
  ```
  gradle -q tasks
  ```
  
- 输出错误堆栈信息: 任务前加 -s 或 -stacktrace

  ```
  gradle -s tasks
  ```

#### 2.3 Gradle 查看帮助信息

有三种方式都可以查看帮助信息

- gradle -？
- gradle -h
- gradle -help

####  2.4 Gradle help任务

gradle 内置了help任务，可以从上面的gradle -tasks的结果中可以看到。这个help任务可以让我们了解每一个task的使用帮助。注意和2.3的区别，2.3是针对gradle整体的命令来说的帮助信息，而此help任务可以让我们详细了解每一个任务的帮助信息。

- 用法： gradle help --task 【任务名】，例如：gradle help --task tasks:

  ```
  
  > Task :help
  Detailed task information for tasks
  
  Path
       :tasks
  
  Type
       TaskReportTask (org.gradle.api.tasks.diagnostics.TaskReportTask)
  
  Options
       --all     Show additional tasks and detail.
  
       --group     Show tasks for a specific group.
  
  Description
       Displays the tasks runnable from root project 'gradleTest'.
  
  Group
       help
  
  BUILD SUCCESSFUL in 1s
  1 actionable task: 1 executed
  
  ```

  该命令会将tasks任务的介绍，路径，类型，可使用的参数，所属分类都展示出来。

  

#### 2.5 强制刷新依赖

当我们项目中引入第三方库的时候，项目中有缓存存在，如果我们想更新到最新的代码，需要强制刷新依赖，这时就会用到这个命令。一般在项目构建的时候会用。

- gradle --refresh-dependencies assemble

#### 2.6 多任务调用

有时我们需要同时执行多个任务，比如构建的时候，需要先清理，再进行构建。多任务调用只需要按顺序以空格分开即可。

比如 先清理再生成jar包：

​	gradle clean jar

比如 先清理再构建

​	gradle clean assembleRelease

#### 2.7 Gradle自定义任务调用

##### 1、编写自定义任务

1. 在任意目录下创建build.gradle文件，例如这里是C:\Users\My\Desktop\DragonForest\gradleTest\build.gradle 

2. 编辑，输入如下

   ```
   
   task helloTask{
   	doLast(){
   		print 'hello,I am first gradle test'
   	}
   }
   ```

##### 2、执行自定义任务

两种方式：

- 1. 切换到上面的build.gradle目录下，执行命令： gradle helloTask即可
- 2. 在外部目录，指定build.gradle 文件后执行，执行命令：gradle -b C:\Users\My\Desktop\DragonForest\gradleTest\build.gradle helloTask

