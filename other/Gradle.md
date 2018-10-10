#### 1、Gradle构建块。
每个Gradle构建都包含三个基本构建快：
- project：

  一个project代表一个正在构建的组件（比如一个JAR文件），或一个想要完成的目标，如部署应用程序。Gradle的build.gradle文件相当于Maven的pom.xml，每个Gradle构建脚本至少定义一个project。

  当构建进程启动后，Gradle基于build.gradle中的配置实例化org.gradle.api.Project类，并且能够通过priject变量使其隐式可用。

- task：

  一个project包含一个或多个task，task包含两个重要功能：任务动作(task action)、任务依赖(task dependecy)。

任务动作定义了一个任务执行时的最小工作单元。

- property：每个project和task实例都提供了可以通过getter和setter方法访问的属性。

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
