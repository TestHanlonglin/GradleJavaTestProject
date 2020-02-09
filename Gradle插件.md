### Gradle插件

#### 1. Java插件

##### 1.1 应用java插件

由于应用插件使用的是Project.apply()方法，它的定义有：

```groovy
apply(Map<String,?> options)
apply(Closure closure)
apply(Action<? super ObjectConfigurationAction> action)
```

应用java插件的方式有：

```groovy
// 方式一：
apply plugin:'java'

// 方式二：
apply plugin:org.gradle.api.plugins.JavaPlugin
// 因为org.gradle.api.plugins是默认导入的，所以此方式也可以简写为
apply plugin:JavaPlugin

// 方式三：
apply {
	plugin 'java'
}
```

##### 1.2 Java插件约定的目录结构

- java插件为我们约定的项目结构为：

- > example1
  >
  > > build.gradle
  > >
  > > src
  > >
  > > > main
  > > >
  > > > > java
  > > > >
  > > > > resources
  > > > 
  > > > test
  > > >  
  > > > > java
  > > > > 
  > > > > resources
  
  
  
	- 注意： src/main/java 默认为源码目录， 其中main, test 成为源集，即souceSet,我们还可以定义其他的源集，如vip
	
	```groovy
	apply plugin: 'java'
	sourceSets{
		vip{
			
		}
	}
	```
	
	接着在src下新建 vip/java,  vip/resources 目录即可。
	
	我们还可以修改源集的java目录，资源目录等,例如当从eclipse中迁移过来的时候:
	
	```groovy
	sourceSets{
		main{
			java{
				srcDir 'src/java'
			}
			resources{
				srcDir 'src/resources'
			}
		}
	}
	```
	
	
	
##### 1.3 依赖第三方库

-  常见的依赖第三方库的方式:

  ```	groovy
  repositories{
  	mavenCentral()
  }
  
  dependencies{
  	complime 'com.squareup.okhttp3:okhttp3:3.0.0'
  }
  ```

  - gradle 提供的依赖方式：

    | 名称        | 继承自              | 被哪个任务调用  | 意义                          |
    | ----------- | ------------------- | --------------- | ----------------------------- |
    | compile     | -                   | compileJava     | 编译时依赖                    |
    | runtime     | compile             | -               | 运行时依赖                    |
    | testCompile | complie             | compileTestJava | 编译测试用例时依赖            |
    | testRuntime | runtime,testCompile | test            | 仅仅在测试用例运行时依赖      |
    | archives    | -                   | uploadArchives  | 该项目发布构建（JAR包等）依赖 |
    | default     | runtime             | -               | 默认依赖配置                  |

    除此之外，Java插件可以为不同的源集在编译时和运行时指定不同的依赖，比如mainCompile, mainRuntime

    | 名称             | 继承自           | 被哪个任务调用       | 意义                         |
    | ---------------- | ---------------- | -------------------- | ---------------------------- |
    | souceSetCompile  | -                | compileSourceSetJava | 为指定的源集提供的编译时依赖 |
    | sourceSetRuntime | sourceSetCompile | -                    | 为指定的源集提供运行时依赖   |

  - 依赖其他模块
  
    ```groovy
    dependencies{
    	compile project(':moulde1')
    }
    ```
  
    其中module1是配置在setting.gradle中的模块名
  
  - 依赖本地jar包
  
    ```groovy
    // 依赖指定的jar包
    dependencies{
    	compile files('libs/okhttp.jar','libs/glide.jar')
    }
    
    // 依赖libs下所有jar包
    dependencies{
    	compile fileTree(dir:'libs',include: '*.jar')
    }
    ```
    
##### 1.4 源集souceSet概念

源集是java插件用来描述和管理源代码及其资源的一个抽象概念，是一个java源代码和资源文件的集合。比如前面的main,test,他们都包含java和resource目录。通过源集，可以方便的设置源集属性。

```groovy
apply plugin: 'java'
sourceSets{
	main{
		// 这里对main sourceSet进行设置
	}
}
```

- 源集的属性：

  | 属性名              | 类型               | 描述                              |
  | ------------------- | ------------------ | --------------------------------- |
  | name                | String             | 他是只读的，比如main              |
  | output.classesDir   | File               | 该源集编译后的class文件目录       |
  | output.resoursesDir | File               | 该源集编译生成的资源目录          |
  | compileClassPath    | FileCollection     | 编译该源集所需的classPath         |
  | java                | SourceDirectorySet | 该源集的java源文件                |
  | jav.srcDir          | Set                | 该源集的java源文件所在目录        |
  | resources           | SourceDirectorySet | 该源集的resources资源文件         |
  | resources.srcDir    | Set                | 该源集的resources资源文件所在目录 |

##### 1.5 Java插件添加的任务

| 任务名称                  | 类型        | 描述                                                  |
| ------------------------- | ----------- | ----------------------------------------------------- |
| compileJava               | JavaCompile | 使用javac编译java源文件                               |
| processResources          | Copy        | 把资源文件拷贝到生产的资源文件目录里                  |
| classes                   | Task        | 组装产生的类和资源文件目录                            |
| compileTestjava           | JavaCompile | 使用javac编译测试java源文件                           |
| processTestResources      | Copy        | 把测试资源文件拷贝到生产的资源文件目录里              |
| testClasses               | Task        | 组装产生的测试类和测试资源文件目录                    |
| jar                       | Jar         | 组装jar文件                                           |
| javadoc                   | JavaDoc     | 使用javadoc生成javaAPI文档                            |
| test                      | Test        | 使用JUnit或TestNG运行单元测试                         |
| uploadArchives            | Upload      | 上传包含jar的构建，用archives{}闭包配置               |
| clean                     | Delete      | 清理构建生成的目录文件                                |
| cleanTaskName             | Delete      | 删除指定任务生成的文件，比如cleanJar删除jar任务生成的 |
| compileSourceSetJava      | JavaCompile | 使用javac编译指定源集的java源码                       |
| processSourceSetResources | Copy        | 把指定源集的资源文件复制到生产文件下的资源文件目录    |
| sourceSetClasses          | Task        | 组装指定源集的类和资源文件目录                        |

此外还有一些用于描述整个构建生命周期的任务，比如build, assemble, check等。其中build和assemble都会构建项目，区别是build会执行单元测试，而assemble不会，而只执行编译和生成jar或zip。

##### 1.6 Java插件添加的属性

| 属性名              | 类型              | 描述                                 |
| ------------------- | ----------------- | ------------------------------------ |
| sourceSets          | SouceSetContainer | 该java项目的源集，可以访问和配置源集 |
| sourceCompatibility | JavaVersion       | 编译java源文件所使用的的java版本     |
| targetCompatibility | JavaVersion       | 编译成的类的java版本                 |
| archivesBaseName    | String            | 我们打包成jar或zip文件的名字         |
| manifest            | Mainfest          | 用于访问或配置我们的manifest文件     |
| libsDir             | File              | 存放生成的类库目录                   |
| distDir             | File              | 存放生成的发布的文件目录             |

##### 1.7 使用命令行构建Java程序

1. 首先我们先使用命令创建一个项目，创建目录example1, 进入并执行命令  <kbd>gradle init</kbd>,会弹出选项，依次选择  application-->Java-->Groovy-->JUnit4-->输入projectName :  example1-->输入souceSet package: com.mypackage,1.2中的项目的所有目录均生成好了，其中还包括一个helloWorld的测试程序。
2. 在example1目录下 执行<kbd>gradle build</kbd>, 则会生成build文件夹，包含class，jar包等。
3. 还可以使用 assemble, clean 等来测试。

##### 1.8 发布构件

- 发布构建大体分两步
  - 1. 定义发布构件的类型，比如一个jar，或者zip.这是在archives{}闭包中配置的。
    2. 配置上传到指定位置，这是在uploadArchives{}闭包中配置的。

```groovy
apply plugin:'java'
apply plugin:'maven'

task publishJarTask(type:jar)
def publishFile=file('build/buildFile')

group 'com.example1'
version '1.0.0'

// 定义发布构建的类型 
// 发布构建是通过artifacts{}闭包配置的，可以有两种方式，一种是task， 一种是文件
artifacts{
    // 方式一： 通过task
	archives publishJarTask
    // 方式二： 通过文件
	// archives publishFile
}

// 上传构件
uploadArchives{
	repositories{
        // 发布到本地目录
		flatDir{
			name 'libs'
			dirs "$projectDir/libs"
		}
        // 发布到本地maven仓库
		mavenLocal()
        // 发布到私服上
        mavenDeveloper{
            repository(url: "http://repo.mycompany.com/content/repositories/release"){
                authentication(userName:"uname",password:"pwd")
            }
            snapshotRespository(url: 'http://repo.mycompany.com/content/repositories/release'){
                authentication(userName:"uname",password:"pwd")
            }
        }
	}
}
```

#### 2. Android插件

##### 2.1 应用android插件

- android插件对于gradle来说是第三方插件，所以需要配置仓库和引用脚本。

  ```
  buildScript{
  	repositories{
  		jcenter()
  	}
  	dependencies{
  		classpath 'com.android.tools.build:gradle:1.5.0'
  	}
  }
  ```

  buildScript脚本一般配置在根工程的build.gradle脚本中，这样子工程就不用重复配置了。我们在子工程中就来使用了：

  ```
  apply plugin: 'com.android.application'
  android{
  	compileSdkVersion 23
  	buildToolsVersion "23.0.1"
  }
  ```

##### 2.2 Android插件分类

| 工程类型     | 插件id                  | 功能                        |
| ------------ | ----------------------- | --------------------------- |
| App工程      | com.android.application | 打包生成apk，一般为主工程   |
| Librrary工程 | com.android.library     | 打包生成aar，一般为模块工程 |
| Test工程     | com.android.test        | 用来测试的工程              |

##### 2.3 Android插件约定的目录结构

> example1
>
> > build.gradle
> >
> > example1.iml
> >
> > libs
> >
> > proguard-rules.pro
> >
> > src
> >
> > > androidTest
> > >
> > > > java
> > >
> > > main
> > >
> > > > AndroidManifest.xml
> > > >
> > > > java
> > > >
> > > > res
> > >
> > > test
> > >
> > > > java

##### 2.4Android插件的配置

通常的andoid插件的配置脚本如下：

```
apply plugin:'com.android.application'

android{
	compileSdkVersion 23
	buildToolsVersion "23.3.0"
	defaultConfig{
		applicationId 'com.example'
		minSdkVersion 15
		targetSdkVersion 23
		versionCode 1
		versionName '1.0.0'
	}
	buildTypes{
		release{
			minifyEanabled true
             proguardFiles getDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
		}
		debug{
		
		}
	}
}
```

android Gradle工程的配置，都是在android{}闭包中配置的，这是唯一的入口。通过它可以对 Android Gradle 工程进行篇日志。具体实现是com.android.build.gradle.AppExtension,创建原型如下：

```groovy
extension = project.extension.create('android',
                                     getExtensionClass(),
                                    (ProjectInternal)project,
                                    instantiator,
                                    androidBuilder,
                                    sdkHandler,
                                    buildTypeContainer,
                                    productFlavorContainer,
                                    signingConfigContainer,
                                    extraModeInfo,
                                    isLibrary());
```

- 插件的配置项

  | 配置名            | 描述                                                         | 原型                                                         |
  | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
  | compileSdkVersion | 配置编译andorid工程的SDK                                     | public void compileSdkVerison(int apiLevel){}<br /><br />public void compileSdkVerison(String versionl){} |
  | buildToolsVersion | 使用的android构建工具的版本，可以在android SDK目录里看到，它是一个工具包，包括appt,dex等工具。 | public void buildToolsVersion(String version){}              |
  | defaultConfig     | 它是android的默认配置，它是一个ProductFlavor. ProductFlavor允许我们根据不同的情况生成不同的APK包。比如渠道包。如果我们没有为自定义ProductFlavor单独配置的话，就会使用默认的配置。<br />它其中会包含属性，比如：<br />- applicationId : 应用id<br />- minSdkVersion: 最低支持的Android系统的API Level<br />- targetSdkVersion: 表明是基于哪个android版本开发的。<br />- versionCode: app内部版本号，用于升级<br />- versionName : app外部版本号，用户可以看到，就是我们发布的版本。 | -                                                            |
  | buildTypes        | 是一个NamedDominObjectContainer类型。是一个域对象。buildTypes里有release，debug等，我们可以在buildTypes增加任意多个BuildType类型，比如debug. buildTypes概念类似于sourceSets | -                                                            |

##### 2.5Android插件添加的任务

  - Android插件是基于Java插件，基本包含了java的所有任务，包括继承的任务，比如assemble,check,build等，可以使用 <kbd>gradle tasks</kbd>来查看android插件的所有任务。这里只介绍 几个

    | 任务名称             | 描述                                                   | 类型 |
    | -------------------- | ------------------------------------------------------ | ---- |
    | connectedCheck       | 在所有连接的设备或模拟器上运行check检查                | -    |
    | deviceCheck          | 通过API连接远程设备运行checks,它被用于CI集成的服务器上 | -    |
    | lint                 | 在所有的ProductFlavor上运行lint检查                    | -    |
    | install 和 uninstall | 在连接的设备上安装或卸载app                            | -    |
    | signingReport        | 打印app的签名                                          | -    |
    | androidDependencies  | 打印android的依赖                                      | -    |

    

    

