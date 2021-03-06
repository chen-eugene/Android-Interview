#### 1、Gradle构建块。
每个Gradle构建都包含三个基本构建快：
- project：

  一个project代表一个正在构建的组件（比如一个JAR文件），或一个想要完成的目标，如部署应用程序。Gradle的build.gradle文件相当于Maven的pom.xml，每个Gradle构建脚本至少定义一个project。

  当构建进程启动后，Gradle基于build.gradle中的配置实例化org.gradle.api.Project类，并且能够通过priject变量使其隐式可用。

- task：

  一个project包含一个或多个task，task包含两个重要功能：任务动作(task action)、任务依赖(task dependecy)。

  任务动作定义了一个任务执行时的最小工作单元。

- property：每个project和task实例都提供了可以通过getter和setter方法访问的属性。

#### 2、Gradle申明属性的方式

 - 扩展属性：使用ext命名空间
 
 为了添加属性，需要使用ext命名空间
 ```
扩展属性申明：
project.ext.myProp = 'myValue'     只有在初始申明扩展属性时需要使用ext命名空间
ext {
	someOtherProp = 123
}

assert myProp == 'myValue'         使用ext命名空间访问属性是可选的
println project.someOtherProp
ext.someOtherProp = 567
 ```
 - 在gradle.property文件中申明

   Gradle属性可以通过gradle.properties文件中申明直接添加到项目中，这个文件位于<USER_HOME>/.gradle目录或项目的根目录下。这些这些属性可通过项目实例访问。记住，即使你有多个项目，每个用户也只能有一个Gradle属性文件<USER_HOME>/.gradle目录下。这是Gradle对它的限制。
   
```
 在gradle.properties文件中申明：
 exampleProp = myValue
 someOtherProp = 455
 
 在项目中访问这两个变量：
 assert proect.exampleProp == 'myValue'
 
 task printGradleProperty << {
 	println "Second property:$someOtherProp"
 }
```
   
#### 3、Gradle包装器：能够让机器在没有安装Gradle运行时的情况下运行Gradle构建，能够解决运行时版本不兼容问题。
   
#### 3、gradle脚本的执行时序。
Gradle脚本执行分为三个过程：

- 初始化：分析有哪些module将要被构建，为每个module创建对应的 project实例。这个时候settings.gradle文件会被解析。 

- 配置：处理所有的模块的 build 脚本，处理依赖，属性等。这个时候每个模块的build.gradle文件会被解析并配置，这个时候会构建整个task的链表。

- 执行：根据task链表来执行某一个特定的task，这个task所依赖的其他task都将会被提前执行。

  project.afterEvaluate，它表示所有的模块都已经配置完了，可以准备执行task了； 如果注册了多个project.afterEvaluate回调，那么执行顺序等同于注册顺序。
