## Gradle基础学习

### 1. Gradle文件setting.gradle和build.gradle

##### 1.1 settings文件

- setting文件是用于初始化以及工程树的配置，一般放在项目根目录下。一般setting文件是用来配置子工程，即模块（module）。**子工程只有在setting.gradle中配置了gradle才会识别，才会在构建的时候包含进去。**

- 一般的settings.gradle文件是这样的：

  ```
  include ':module1','module2'
  ```

  上面的module1和module2是模块的名字，默认是放在settings文件的平级目录。

  而如果module1和module2不在settings的同级目录，那么需要这样配置：

  ```
  include ':module1'
  project(':module1').projectDir=new File(rootDir,'test/module1')
  include ':module2'
  project(':module2').projectDir=new File(rootDir,'test/module2')
  ```

  上面的代码指定了module1和module2的目录。实际中通过这个特性，可以灵活的将各个模块分级分块，只要在setting.gradle中指定目录就行了。

##### 1.2 build文件

- 每个工程都有一个build.gradle文件，该文件是project构建的入口，在这里可以针对Project进行配置，比如配置版本，需要哪些操作，依赖哪些库等。同时RootProject也会有一个build.gradle文件，可以对他的子工程进行统一公共配置。

### 2. Project和Task概念

一个项目中有多个project，一个project又包含多个task,每个task都是一次原子性操作，比如打一个jar包，上传到maven等，他们组合形成了整个gradle的构建。

##### 2.1  创建任务

- 方式一：(通过task关键字)

  ```
  task customTask1{
  	doFirst{
  		println 'hello'
  	}
  }
  ```

  上面的task实际上是project的一个函数，原型为create(String name,Closure closure)

- 方式二：（通过TaskContainer）

```
tasks.create("customTask2"){
	doFirst{
		println 'hello2'
	}
}
```

Project中已经为我们定义好了一个TaskContainer对象，他就是tasks

##### 2.2 任务依赖

- 任务之间是有依赖关系的，可以控制哪些任务先执行，哪些后执行。比如install安装命令，需要依赖package打包命令，因为在安装之前必须要先打包。

- 可以使用dependsOn 关键字来设置依赖

  ```
  task helloFirst{
  	doLast(){
  		println 'I am helloFirst'
  	}
  }
  // 第一种
  task helloTask(dependsOn: helloFirst){
  	doLast(){
  		println 'I am helloTask'
  	}
  }
  // 第二种
  task helloTask2(){
  	dependsOn helloFirst
  	doLast(){
  		println 'I am helloTask2'
  	}
  }
  ```

  执行 gradle helloTask ，输出：


  ```
  I am helloFirst
  I am helloTask
  ```

  ##### 2.3 任务配置

创建task时可以指定配置，如上面提到的dependsOn,配置有一下几种

| 配置项 | 描述 |默认值|
| ---- | ---- | ---- |
| type | 基于一个存在的task创建，和继承类差不多 |DefaultTask|
| overwrite | 是否替换存在的task,和type配合使用 |false|
| dependsOn | 用于配置任务的依赖 |[]|
| action | 添加到任务中的一个Action或一个闭包 |null|
| description | 用于配置任务的描述 |null|
| group | 用于配置任务的分组 |null|
|  |  ||

```
task helloTask{
	description '测试task'
	group BasePlugin.BUILD_GROUP
	doLast{
		println 'I am helloTask'
	}
}
```

##### 2.4 任务顺序

- TaskB.shouldRunAfter(TaskA): 表示TaskB应该在TaskA之后执行，非必须，可能顺序不一定会按照这个顺序执行。

- TaskB.mustRunAfter(TaskA): 表示TaskB必须在TaskA之后执行。

注意： mustRunAfter与dependsOn区别： 前者没有强依赖关系，假设TaskA不存在也能顺利编译。后者具有强依赖关系，假设TaskA不存在，则会报错。

##### 2.5 任务启用和禁用

- Task有个enabled属性，用于启用和禁用任务。默认是true,表示启用。设置为false,则禁止该任务执行，输出会提示该任务跳过。

  ```
  task taskA{
  	doLast{
  		println 'this is taskA'
  	}
  }
  taskA.enabled=false
  ```

  执行 gradle taskA, 则taskA不会执行。 假若有其他任务依赖了taskA,则会跳过taskA.

  