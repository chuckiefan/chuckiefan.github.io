---
layout: post
title: 'Android Gradle 使用教程'
date: 2016-07-01
author: Chuckiefan
tags: Android Gradle
---

![gradle](http://upload-images.jianshu.io/upload_images/1226129-31f15c4c81dba08e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 1.介绍
>  如果你正在查阅build.gradle文件的所有可选项，请点击这里进行查阅：[DSL参考](http://google.github.io/android-gradle-dsl/current/)

## 1.1新构建系统的特性
gradle构建系统具有如下的特点：
* 易于代码和资源复用
* 易于创建应用的版本，例如发布多apk以及应用的不同渠道版本
* 构建过程易于配置，扩展和优化
* 良好的IDE整合

## 1.2为什么使用Gradle？
Gradle既是一个先进的构建系统，也是一个允许通过插件创建自定义构建逻辑的构建工具集。以下是一些我们为什么选择Gradle的原因：
* 用于描述和操作构建逻辑的，基于Groovy的特定领域语言(DSL)
* 基于Groovy的构建文件，允许混合通过使用DSL的声明式元素以及使用代码去操作DSL元素来提供自定义逻辑。
* 通过Maven或者Ivy的内置依赖管理
* 非常灵活。
* 插件能够导出自己的DSL以及自己的API，用于使用构建文件
* 支持IDE整合的良好工具API

## 1.3要求
* Gradle 2.2
* SDK版本19.0.0及以上，一些特性可能会需要更新的版本

# 2.基础项目搭建
一个Gradle项目在一个文件中描述了该项目的构建情况，该文件被称为build.gradle，位于项目的根目录。([点击这里查看构建系统概述](https://docs.gradle.org/current/userguide/userguide_single.html "点击这里查看构建系统的概述"))
## 2.1简单的构建文件
大多数简单的Android项目拥有如下的build.gradle文件:

```groovy
buildscript {
    repositories { 
        jcenter()
    }`
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.1'
    }
}
apply plugin: 'com.android.application'
android {
    compileSdkVersion 23
    buildToolsVersion "23.1.0"`
}
```
在Android的构建文件中，有以下三个主要区域：
`buildscript{…} `用于配置用于构建的代码。在上面的例子中，声明使用jCenter库，并且有一个依赖于Maven类路径的Maven Artifact。该Artifact是包含包含Android构建插件的1.3.1版本Gradle。
> 上例中的Artifact仅仅影响到构建代码的运行，并不作用于代码项目。至于项目本身需要声明自己的库和依赖关系，该部分将会在下文介绍。

之后，该构建文件应用了com.android.application插件。该插件用于构建Android应用。
最后`android{…}`中配置了所有用于Android应用构建的参数。这里是Android特定领域语言(DSL)的入口点。默认情况下，仅仅编译目标(compilation target)和构建工具版本(version of the build-tool)是必备的。也就是上例中的compileSdkVersion属性和buildtoolsVersion属性。该编译目标和旧版本的project.properties文件中的target属性是相同的。新版本中既可以指定一个整型(即API level)，也可以使用表示相同值的字符串对compileSdkVersion进行指定。
> 重点：你应当使用com.android.application插件，使用java插件会导致错误。
> 注意1：另外，你也需要一个local.properties文件，设置sdk.dir去配置本地的SDK路径位置。
> 注意2：另外，你也可以设置名为`ANDROID_HOME`的环境变量。 这两者并没有什么区别，你可以选择任意一种。例如：`sdk.dir=/path/to/Android/Sdk`

## 2.2项目结构
上面例子的基本构建文件预期了一个默认的文件夹结构。Gradle遵循“约定优于配置”的概念，尽可能提供良好的默认项值。基本项目由两个被称为代码集(source sets)的部分构成，一个是主要的源代码，另一个是测试代码。它们各自位于：
* `src/main`
* `src/androidTest`

在这些目录里，每一个资源组件都有各自的子目录。对于Java和Android插件，java源代码和java资源的路径位置为：
* `java/`
* `resources/`

对于Android插件而言，有如下特定的文件和目录：
* `AndroidManifest.xml`
* `res/`
* `assets`
* `aidl/`
* `rs/`
* `jni/`
* `jniLibs/`

这意味着主资源集中的所有`*.java`文件位于`src/main.java`中，主清单文件(main manifest)位于`src/main/AndroidManifest.xml`。
> 注意：
> `src/androidTest/AndroidManifest.xml`由于会自动创建，因此并不是必须手动编写的。

### 2.2.1配置结构
当默认的项目结构并不完善时，可能需要进行配置。本部分只介绍Android项目结构的配置，关于纯java项目的项目结构配置，请参阅：[gradle documentation](https://docs.gradle.org/current/userguide/java_plugin.html#N12394 "gradle documentation")。
Android插件使用和纯java项目相同的语法，但是由于其使用自己的资源集，项目的配置由`android{...}`代码块完成。例如，旧的项目结构(Eclipse)中对主代码和测试代码进行映射：

```groovy
android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res']
            assets.srcDirs = ['assets']
        }

 androidTest.setRoot('tests')`
      }
}
```

> 注1： 因为旧的结构中在同一个文件夹中放入所有的资源文件，我们需要对这些资源集重新进行映射,将java代码，资源文件等等放入src文件夹。
> 注2： `setRoot()`移动整个资源集和其子文件夹到一个新的文件夹中，上例将`src/androidTest/*`移动到`tests/*` 中，当然此处的语句`androidTest.setRoot('tests') `只是Android的特性，并不适用于Java的资源集。

## 2.3构建的任务
### 2.3.1通用任务
在构建文件中应用插件会自动创建一系列构建任务的集合。Java插件和Android插件都是如此。关于任务的约定如下：
* assemble 该任务用于组合项目的输出 
	* check 该任务用于运行所有的的检测
	* build 该任务执行assemble和check任务
	* clean该任务清除项目的输出

任务assemble，check以及build并不真正做任何事。这些插件中的“祖先”任务用于添加在任务中真正执行的任务。
这样，不论当前的任务类型是什么以及何种插件被应用，都将会允许你使用执行相同的任务。例如，findbugs插件将会创建一个一个新的任务，并通过check任务进行依赖，无论什么时候执行调用该任务，都可以直接使用check任务。
在命令行中，你可以通过下面的命令得到高级别任务：
`gradle tasks `
查看所有依赖任务列表可以使用下面的命令：
`gradle tasks —all`
> 注意：gradle自动显示任务已经声明任务的输入和输出情况。

当你在没有变更项目内容的情况下再次运行构建任务时，Gradle将会报告所有的任务已经UP-TO-DATE，意味着没有需要运行的任务。这样会使得任务彼此正确的依赖，而不需要没必要的构建操作。

### 2.3.2Java项目任务
以下是由Java插件所创建的，依赖于祖先任务的最重要的两个任务：
* `assemble` jar：创建输出的任务
* `check` test:运行测试的任务

jar任务本身直接或间接依赖于其他任务：例如classes任务将会编译Java代码。测试代码被任务testClasses所编译，但是就像classes任务一样，几乎很少有人会去调用，因为执行test任务即可。
总的来说，你可能应当仅仅调用assemble任务或者check任务并忽略其他的任务。你可以点击这里查看Java插件所有的任务集合及其描述信息：[Java插件的任务集合](https://docs.gradle.org/current/userguide/java_plugin.html "Java插件的任务集")

### 2.3.3Android的任务
Android插件使用相同的概念来保持和其他插件之间的兼容性，并且，Android插件还添加了额外的祖先任务：
* assemble 
* check
* connectedCheck 运行check任务需要一个连接的设备或者模拟器，在所有的连接设备中，该任务将并行执行。
* deviceCheck 使用API去连接远程设备，用于CI服务器(持续集成服务器)。
* build
* clean

新的祖先任务是必备的，这是为了能够在不需要连接设备的情况下进行规则检查。请注意，build任务并不依赖deviceCheck任务或者connectedCheck任务。
一个Android的项目有至少两个输出文件：一个debug apk文件以及一个release apk文件。它们中的每一个都有其自己的祖先任务来优化构建：
* assemble
	* assembleDebug
	* assembleRelease

assembleDebug和assembleRelease两者都依赖于其他的多步任务执行来构建app文件。assemble任务依赖于assembleDebug和assembleRelease，可调用assemble任务构建上述两种apk。
> 提示： gradle支持骆驼命名法缩写的形式在命令行中为任务命名。例如：
> `gradle aR`
> 和下面的命令相同：
> `gradle assembleRelease`
> 除非有其他任务和’aR’重复。

check任务有自己的依赖项：
* `check`
	* `lint`
* `connectedCheck`
	* `connectedAndroidTest`
* `deviceCheck`
	* 这取决于当任务创建时，其他插件什么时候实现测试拓展点。

最后，插件创建了安装了卸载所有构建类型的任务（包括debug，release和test），只要能够被安装（需要签名）。例如：
* `installDebug`
* `installRelease`
* `uninstallAll`
	* `uninstallDebug`
	* `uninstallRelease`
	* `uninstallDebugAndroidTest`

## 2.4构建自定义基础
Android插件提供了一个宽泛的领域定制语言（DSL）在构建系统中对大多数内容进行自定义。
### 2.4.1Manifest条目
通过DSL，能够配置最重要的manifest条目，例如：

* `minSdkVersion`
* `targetSdkVersion`
* `versionCode`
* `versionName`
* `applicationId` 关于包名，详情请点击：[应用ID VS 包名](http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename) 
* `testApplicationId` (用于测试apk)
* `testInstrumentationRunner`

例如：

```groovy
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"


    defaultConfig { 
        versionCode 12
        versionName "2.0"
        minSdkVersion 16
        targetSdkVersion 23
    }
}
```

关于完整的构建属性清单以及其默认值，请查看[Android插件特定领域语言参考](http://google.github.io/android-gradle-dsl/current/)。
把这些清单属性放入构建文件中的好处是，这些值可以动态获取。例如，别人可以从文件中阅读版本名称或者使用自定义逻辑：

```groovy
def computeVersionName() {
    //...
}
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"
    defaultConfig {
        versionCode 12 
        versionName computeVersionName()
        minSdkVersion 16
        targetSdkVersion 23
    }
}
```

> 注意：不要使用当前域中可能和getters冲突的文件名。例如`defaultConfig{...}`调用`getVersionName()`会自动使用`defaultConfig.getVersion()`而不是自定义的方法。

### 2.4.2构建类型
默认情况下，Android插件自动建立了debug版和release版应用。二者最大的不同是在安全设备（非开发设备）上的调试能力，以及APK文件被签名的详情信息。debug版本是由已知用户名/密码（防止在构建过程中的提示）所自动创建的key/证书所签名。release版本在构建过程中并不被签名，这将会在后面发生。
这项配置通过一个叫做`BuildType`的对象完成。默认情况下,有两个实例被创建，分别是debug版和release版。Android插件运行自定义这两个实例，就像其他构建类型一样。这是由`buildTypes`特定领域语言容器所完成的：

```groovy
android {
    buildTypes {
        debug {
            applicationIdSuffix ".debug"
        }


        jnidebug {
            initWith(buildTypes.debug)
            applicationIdSuffix ".jnidebug"
            jniDebuggable true
        }
    }`}
```

上述片断实现了如下内容：
* 配置默认的debug构建类型
	* 设置包为”应用ID.debug”，使得应用的debug版和release版都能在同一个设备上安装。
* 创建了一个新的叫作`jnidebug`的构建类型，并对其使用`debug`构建类型进行复制。
* 继续配置`jnidebug`，开启JNI组件调试功能，并添加一个不同的包名后缀。

创建一个新的构建类型就和使用`buildTypes`容器下的一个新元素一样简单，要么调用`initWith()`要么将其完全配置结束。关于构建类型的完整属性清单，请查阅：[Android构建类型参考](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.BuildType.html)
另外，关于修改构建属性，构建类型可以用于添加指定的代码和资源文件。对每一种构建类型，新的相匹配资源集被创建，其默认位置为”src/构建类型名称”，例如`src/debug/java`目录能够用于添加仅仅在debug APK文件中所编译的代码或资源文件。这意味着构建类型的命名不能和`main`以及`androidTest`重复（，并且必须独一无二（这是插件所限制的）。
就像其他任何的资源集一样，构建类型资源的位置能够重新指定：
`android {`
`    sourceSets.jnidebug.setRoot('foo/jnidebug')`
`}`
此外，对于每一种构建类型，一个新的`assemble构建类型名称`任务被创建，例如`assembleDebug`。`assembleDebug`任务和`assembleRelease`任务在上文中已经被提及，这也就是他们为什么会存在的原因。当`debug`构建类型和`release`构建类型预创建时，这些任务(`assembleDebug`和`assembleRelease`)也会被自动创建。根据这个规则，上述的build.gradle片段也将会生成一个叫做`assembleJnidebug`的任务，该任务的依赖关系也和`assembleDebug`以及`assembleRelease`一样。
> 提示：请记得你能够输入`aJ`来运行`assembleJnidebug `任务。

可能的使用情况：
* 一些权限只在debug模式下开启，在release模式下禁用
* 调试的自定义实现
* debug模式下的使用资源不同（例如当一个资源值与资源证书相挂钩时）
不同构建类型的代码/资源被用于以下情况：
* Manifest清单被合并到app清单文件中
* 作为其他资源文件夹的代码
* 资源文件被主资源文件覆盖并替换现有的值

### 2.4.3签名配置
对一个应用的签名需要以下内容(关于APK文件签名的详细信息，请查阅[签名你的应用](https://developer.android.com/studio/publish/app-signing.html))：

* keystore
* keystore密码
* keystore别名
* key密码
* 商店类型

位置，key名称以及密码和商店类型共同构成签名配置。默认情况下，`debug`配置使用debug keystore，其带有已知的密码和默认的key和key的密码。debug的keystore位于`$HOME/.android/debug.keystore`，如果不存在会被创建。`debug`构建类型被自动设成使用`debug`签名配置。
创建其他配置信息或自定义默认的内置配置信息是可行的。这是通过`signingConfigs`特定领域语言容器完成的：

```groovy
android {
    signingConfigs {
        debug {
                    storeFile file("debug.keystore")
        }

        myConfig {
            storeFile file("other.keystore")
            storePassword "android"`            keyAlias "androiddebugkey"
            keyPassword "android"
        }
    }


    buildTypes {
        foo {
            signingConfig signingConfigs.myConfig
        }`
    }
}
```

上述片段改变了debug版本keystore文件的位置为项目的根目录。这将会自动影响到任何构建类型，任何构建类型都能够使用它。在上例中即为`debug`构建类型。这也将会创建一个新的签名配置，新的构建类型(`foot`)便可以使用这个新的签名配置。
> 注意1：只有debug keystore文件自动创建并位于默认位置。改变debug keystore文件位置并不会按需创建。只有使用不同命名创建签名配置并使用默认的debug keystore位置的情况下才会自动创建。从另一方面来说，这是和keystore文件的位置挂钩的，而不是和配置信息的命名相对应的。
> 注意2：keystore文件的位置通常情况下和项目的根目录相关，但是也能够是绝对目录，虽然这并不被推荐（除非是debug的，因为它会自动被创建）。
> 注意3：如果你把这些文件纳入到版本控制系统中，你可能并不想要把密码放在其中。[这里](http://stackoverflow.com/questions/18328730/how-to-create-a-release-signed-apk-file-using-gradle)介绍了从控制台或环境变量中读取值的方法。

# 3.依赖，Android库以及多项目建立

## 3.1依赖二进制包

### 3.1.1本地包

要去配置一个依赖库或者额外的jar库，你需要在`compile`配置中添加一个依赖关系。下面的片段在libs文件夹中添加了所有jar文件的依赖关系：

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}


android {
    //...
}
```

> 注意：`dependencies`特定领域语言元素是标准Gradle API中的一部分。在这里面的每一项都被加入到编译类路径中，并都在最终的APK文件中被打包进去。以下是可能的配置信息：

* `compile` 主应用
* `androidTestCompile` 测试应用
* `debugCompile` debug构建类型
* `releaseCompile` release构建类型

由于在构建APK时并不可能没有关联的构建类型，因此APK总是被配置至少两个编译配置信息：`compile`以及`构建类型Compile`。创建一个新的构建类型将会自动创建一个基于该构建类型名称的编译配置。如果debug版本需要添加一个自定义的库（如报告程序崩溃情况）时，或者不同构建类型依赖于相同库的不同版本时这将是非常有用的（[点击这里](https://docs.gradle.org/current/userguide/dependency_management.html#sub:version_conflicts)详见不同版本的冲突是如何处理的）。
### 3.1.2远程依赖
Gradle支持从Maven和Ivy库中拉取依赖(artifact)。首先，该仓库必须被添加到列表中，其次依赖必须被声明。

```groovy
repositories {
     jcenter()
}


dependencies {
    compile 'com.google.guava:guava:18.0'
}


android {
    //...
}
```

> 注意1：`jcenter()`是指定仓库的URL缩写。Gradle同时支持远程和本地的仓库。
> 注意2：Gradle遵循依赖的传递性。这意味着，如果如果一个依赖，依赖于其本身，这也会被拉取。

关于建立依赖的更多信息，请阅读[Gradle使用指南](https://docs.gradle.org/current/userguide/artifact_dependencies_tutorial.html)和[DSL文档](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.dsl.DependencyHandler.html)
## 3.2多项目的搭建
Gradle项目也能够通过多项目搭建依赖于其他Gradle项目。一个多项目的搭建通常为将所有项目作为已给项目根目录的子目录。例如，给出如下的结构：
MyProject/
 + app/
 + libraries/
    + lib1/
    + lib2/
我们能够识别出三个项目。Gradle会用下述的命名进行参考：
:app
:libraries:lib1
:libraries:lib2
每一个项目拥有其自己的build.gradle文件，声明了其如何得到其构建。额外的，将会有一个被命名为setting.gradle的文件位于根目录，用于声明所有项目。一下给出了结构：
MyProject/
 | settings.gradle
 + app/
    | build.gradle
 + libraries/
    + lib1/
       | build.gradle
    + lib2/
       | build.gradle

setting.gradle的内容非常简单。它定义了哪个文件夹是一个Gradle项目：

```groovy
include ':app', ':libraries:lib1', ':libraries:lib2'
```


`:app`项目很可能依赖于其他的项目作为库，这是通过以下声明实现的：

```groovy
dependencies {
     compile project(':libraries:lib1')
}
```

关于多项目搭建的更多信息，请[点击这里](https://docs.gradle.org/current/userguide/multi_project_builds.html)。

## 3.3库项目
在上述的多项目搭建中`:libraries:lib1 `和` :libraries:lib2`作为Java项目，`:app`Android项目将会使用这两个项目的jar输出。但是，如果你想要通过Android API或使用Android风格的资源来共享代码，这些库就不能是常规的Java项目，它们必须是Android库项目。
### 3.3.1创建一个库项目
一个库项目和一个常规的Android项目非常相似。因为构建库和构建应用是不同的，因此使用的插件也不同。本质上来说，两个插件大多数代码都是相似的，并且都由相同的`com.android.tools.build.gradle`jar文件提供。

```groovy
buildscript {
    repositories {
        jcenter()
    }


    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.1'
    }
}


apply plugin: 'com.android.library'


android {
    compileSdkVersion 23
    buildToolsVersion "23.0.1"
}
```

这将会创建一个使用API 23进行编译的库项目。资源集，构建类型以及依赖关系的使用都和应用项目相同，并可以通过相同的方式自定义。
### 3.3.2项目和项目库的区别
库项目的主要输出是一个.arr包（表示Android archive），是编译代码（就像jar文件或者原生的.so文件一样）和资源文件（清单，res文件以及assets文件）的组合。一个库项目也可以生成一个测试apk文件用于测试库的独立性。对此使用的祖先任务也是相同的(`assembleDebug`和`assembleReleas`)。因此用命令去构建这样一个项目并没有什么区别。对于其他方面，库项目表现得和应用项目一样。库项目也拥有构建类型和产品渠道（product flavors，见下文），并且能够潜在地生成多个版本的aar文件。请注意，*构建类型的大多数配置并不能够用于库项目*。但是你可以使用自定义资源集来改变库的内容，这取决于当前库是否被用于一个项目，或者用于被测试。
### 3.3.4引用一个库
引用一个库和二进制包被引用是一样的：

```groovy
dependencies {
    compile project(':libraries:lib1')
    compile project(':libraries:lib2')
}
```

> 注意：如果你有多个库，那么引用的顺序就变得很重要。这和旧的构建系统中在project.properties文件中的依赖顺序一样重要。

### 3.3.5库发布
默认情况下，一个库仅仅发布其release版本。该版本被用于所有项目对库的引用，不论这些项目本身使用怎样的的构建类型，而这是一个我们将要取消的临时限制。你可以控制要发布何种版本：

```groovy
android {
    defaultPublishConfig "debug"
}
```

注意，发布的配置名参考于版本的命名。Release和Debug版仅仅适合没有渠道的情况下。如果你想要使用渠道改变默认的发布版本，你可以这样写：

```groovy
android {
    defaultPublishConfig "flavor1Debug"`
}
```

发布一个库的所有版本是可能的。我们计划允许使用一个标准的项目到项目的依赖，但是目前这是不可能的，这是因为Gradle中的一些限制（我们正在努力解决这些）。
发布所有版本在默认情况下是不可以的。下面的片段展示了如何开启这项功能：

```groovy
android {
    publishNonDefault true
}
```

很重要的一点是，你需要意识到发布多版本的arr文件而不是包含多个版本的arr文件。每一个arr包包含单个版本。发布一个版本意味着使得这个arr文件和Gradle项目输出的依赖一样可用。这能够被用于当发布到maven库时，或者当另一个项目依赖于该库时。
Gradle有一个默认依赖的概念。这是当我们这样写时：

```groovy
dependencies {
    compile project(':libraries:lib2')
}
```

创建一个依赖于另一个发布类库的类库时，你需要指定使用哪一个：

```groovy
dependencies {
    flavor1Compile project(path: ':lib1', configuration: 'flavor1Release')
    flavor2Compile project(path: ':lib1', configuration: 'flavor2Release')
}
```

> 重点1：请注意到已发布的配置是全版本的，包括所有的构建类型，并且需要被引用。
> 重点2：当开启非默认发布时，maven发布插件将会发布这些额外的版本作为额外的包（带有classifier）。这意味着发布到maven库并不能真正地兼容。你应该发布一个单版本，或者开启所有的配置用于发布内部项目依赖。

# 4.测试
应用项目中整合了测试应用的构建。因此不需要再有一个独立的测试项目。
## 4.1单元测试
在1.1中所提到的单元测试支持，请[点击这里](http://tools.android.com/tech-docs/unit-testing-support)。本章节的剩下部分介绍了“工具测试(instrumentation tests)”，能够运行在真机或者模拟机上，其要求是要构建一个测试APK文件。
## 4.2基础配置
如之前所提到的，`main`资源集后面就是`androidTest`资源集，默认情况下位于`src/androidTest`，使用这个资源集使得测试APK被构建并能够安装到设备中，从而可以使用Android测试框架来测试应用，包括Android单元测试，instrumentation tests以及uiautomator tests。清单中的`<instrumentation>`节点是被生成的，但是你可以创建一个`src/androidTest/AndroidManifest.xml`文件来添加测试清单(manifest)的其他组件。
还有一些值能够在instrumentation测试应用中配置。（详情请查阅[DSL参考](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.ProductFlavor.html)）

* testApplicationId
* testInstrumentationRunner
* testHandleProfiling
* testFunctionalTest

如前所述，上述信息被配置在`defaultConfig`对象：

```groovy
android {
    defaultConfig {
        testApplicationId "com.test.foo"        testInstrumentationRunner "android.test.InstrumentationTestRunner"
        testHandleProfiling true 
        testFunctionalTest true
    }
}
```

测试应用清单文件的instrumentation节点中`targetPackage`属性的值自动化填充为测试应用的包名。即使在`defaultConfig`中或者在构建类型对象中自定义,该值并不会发生改变。这也就是清单文件被自动生成的部分原因。
另外，`androidTest`资源集能够配置自己的依赖关系。默认情况下，应用和其自身的依赖被添加到测试app的类路径中，但是可以使用下面的片段进行扩展：

```groovy
dependencies {
    androidTestCompile 'com.google.guava:guava:11.0.2'
}
```

测试app是由`assembleAndroidTest`任务构建的。这并不是对主`assemble`任务的依赖，而是当测试准备运行时自动调用的。
当前只有一个构建类型能够被测试。默认情况下是`debug`构建类型。但是可以进行如下配置：

```groovy
android {
    //...
    testBuildType "staging"
}
```

## 4.3解决主apk和测试apk之间的冲突
当instrumentation测试运行时，主APK和测试APK共享相同的类路径。如果主APK和测试APK使用相同的库（如Guava）的不同版本时，Gradle构建会失败。如果gradle并不捕获这一点，你的应用会在测试版和正常版表现地不同（包括任何会崩溃的情况）。
为了让应用构建成功，请确保两个APK都使用相同版本的库。如果错误来自于间接依赖（在你自己的build.gradle中没有声明的库），仅仅在需要的地方(`compile`或者`androidTestCompile`)添加最新的依赖即可。你也可以使用[Gradle解决冲突机制](https://docs.gradle.org/current/dsl/org.gradle.api.artifacts.ResolutionStrategy.html)。你可以通过运行以下代码检查依赖树：`./gradlew :app:dependencies`和` ./gradlew :app:androidDependencies`。
## 4.4运行测试

如前所述，`check`任务需要一个连接的设备，并通过祖先任务`connnectedCheck`启动。这个任务会依赖`connectedDebugAndroidTest`。这个任务做了以下几件事：

* 确保app和测试app被构建(依赖于`assembleDebug`和`assembleDebugAndroidTest`)。
* 安装两款应用
* 运行测试
* 卸载两款应用

如果有多个设备连接，所有的测试都会平行运行在所有连接的设备上。如果其中一个测试失败，在任何设备中，构建都将会失败。
## 4.5测试Android库

测试Android库项目和测试Android应用项目是完全相同的。唯一的区别在于，整个库以及依赖作为测试app的一个库被自动添加。结果是，测试APK不仅仅包括了自身的代码，也包含了库本身及其依赖。`androidTest`任务变成仅仅安装(卸载)测试APK（因为没有其他APK可供安装）。
## 4.6测试报告

当运行单元测试时，Gradle输出一个HTML格式的报告可供轻松地查看结果。Android插件构建并拓展HTML报告用于从所有连接设备中汇集。所有的测试结果以xml文件的形式存储在`build/reports/androidTests/`中（和常规的jNnit存储在`build/reports/tests`相似）。该路径可配置如下：

```groovy
android {
    //...
    testOptions {
        resultsDir = "${project.buildDir}/foo/results"`
    }
}
```

`android.testOptions.resultsDir`的值请参考：[Project.file(String)](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html#file(java.lang.Object))。

### 4.6.1多项目报告

在带有多应用和多库的多项目的搭建中，当同时运行所有测试时，可能生成一个所有测试的测试报告是很有用的。
为了做到这一点，可用一个不同的gradle插件：

```groovy
buildscript {
    repositories {
        jcenter()
    }


    dependencies {
        classpath 'com.android.tools.build:gradle:0.5.6'
    }
}


apply plugin: 'android-reporting'
```

这应该被用在根项目中，也就是setting.gradle旁边的build.gradle。
之后，从根文件夹开，以下的命令将会运行所有的测试并汇集成报告：
`gradle deviceCheck mergeAndroidReports --continue`
> 注意：`--continue`选项确保了所有子目录在内的所有测试都能够被运行，即使其中有失败的情况。

## 4.7Lint支持

你可以在一个指定版本运行lint检查（如下），例如：`./gradlew lintRelease`或者全版本的lint检查(`./gradlew lint`)，在这种情况下会产生一个描述指定版本的报告。你可以通过添加lintOption部分来配置lint（如下）。你应该添加一些典型的部分：[详见](http://google.github.io/android-gradle-dsl/current/com.android.build.gradle.internal.dsl.LintOptions.html#com.android.build.gradle.internal.dsl.LintOptions)。

```groovy
android {
    lintOptions {
        // turn off checking the given issue id's
        disable 'TypographyFractions','TypographyQuotes'

        // turn on the given issue id's
        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'

        // check *only* the given issue id's
        check 'NewApi', 'InlinedApi'
    }
}
```

# 5.构建版本

新构建系统的一个目标就是能够对同一个应用创建不同的版本。
主要使用情况有两个：

1. 一个应用的不同版本。例如，一个免费/demo版本和一个专业版本
2. 同一个应用在Google Play商店打包分发多个详见：[http://developer.android.com/google/play/publishing/multiple-apks.html](http://developer.android.com/google/play/publishing/multiple-apks.html)
3. 前两者的组合

目标是能够在同一个项目中生成不同的应用，而不是使用同一个库的不同应用项目。

## 5.1产品渠道

产品渠道(product flavor)定义了一个项目应用构建的自定义版本。单个的项目可以有不同的渠道，从而生成的应用不同。
这个新的概念被设计用于帮助版本差异非常小的情况下。如果当问到“这是否是同一个应用？”时，如果是，那么通过库项目去实现可能更适合一些。
产品渠道通过一个叫做`productFlavors`的特定领域容器声明：

```groovy
android {
    //....


    productFlavors {
        flavor1 {
            //...         
        }


        flavor2 {
            ...
        }
    }
}
```

上例中将会创建两个渠道，分别是`flavor1`和`flavor2`

> 注意：渠道的名称不能喝已经存在的构建类型名称冲突，也不能使用`androidTest`和`test`资源集的名称。

## 5.2构建类型+产品渠道=构建版本

正如我们前面所看到的，每一个构建类型生成一个新的APK文件。产品渠道也是相同的：项目的输出变成所有可能构建类型和产品渠道的组合。每一个（构建类型，产品渠道）组合被称为构建版本。例如，在默认的`debug`和`release`构建类型中，上面的例子可以生成四个构建版本：

* Flavor1 - debug
* Flavor1 - release
* Flavor2 - debug
* Flavor2 - release

没有渠道的项目依然拥有构建版本，使用的是单个默认的`default`渠道/配置.

## 5.3产品渠道配置

每一个渠道的完整配置如下：

```groovy
android {
    //...


    defaultConfig {
        minSdkVersion 8
        versionCode 10
    }


    productFlavors {
        flavor1 {
            applicationId "com.example.flavor1"
            versionCode 20
         }

         flavor2 {
             applicationId "com.example.flavor2"
             minSdkVersion 14
         }
    }
}
```

注意到`android.productFlavors.* `对象是`ProductFlavor`，其类型和`android.defaultConfig`对象类型相同。这意味着它们共享相同的属性。
`defaultConfig`为所有的渠道提供了基础的配置，每一个种渠道能够覆盖它的任何值。在上述的例子中，配置信息以如下结尾：


* flavor1
	* applicationId: com.example.flavor1
	* minSdkVersion: 8
	* versionCode: 20
* flavor2
	* applicationId: com.example.flavor2
	* minSdkVersion: 14
	* versionCode: 10

通常情况下，构建类型配置会覆盖其他配置。例如，构建类型的`applicationIdSuffix`追加在产品渠道的`applicationId`后面。有一些在构建类型和产品渠道都能够设置的情况。在这种情况下，例如`signingConfig`就是这种属性。这使得所有发布包共享这些签名配置。通过设置`android.buildTypes.release.signingConfig`或者对每一个包通过设置自己的`android.productFlavors.*.signingConfig`来分别使用签名配置。

## 5.4资源集和依赖

和构建类型相似，产品渠道也是通过自己的资源集来贡献代码。上述的例子创建了四个资源集：

* `android.sourceSets.flavor1` 位置为src/flavor1
*  `android.sourceSets.flavor2`位置为src/flavor2/
* `android.sourceSets.androidTestFlavor1`位置为 src/androidTestFlavor1/
* `android.sourceSets.androidTestFlavor2`位置为 src/androidTestFlavor2/
这些资源集通过构建类型和`android.sourceSets.main`被用于构建APK。下面的规则用于处理所有被用于构建单个APK的情况：

* 所有的代码（`src/*/java`)共同用于多文件夹来生成单个输出。
* 清单被共同合并到单个的清单中。这允许产品渠道拥有不同的组件、权限，和构建类型相似。
* 所有的资源遵循覆盖的优先级。构建类型覆盖产品渠道，产品渠道覆盖`main`资源集。
* 每个构建版本生成自己的R类。各个版本直接不存在共享。

最后，和构建类型一样，产品渠道可以拥有自己的依赖。例如，如果渠道用于生成基于广告的app或者付费的app，每一个渠道拥有自己的广告sdk依赖。

```groovy
dependencies {
    flavor1Compile "..."
}
```

在这个例子中，文件`src/flavor1/AndroidManifest.xml`可能需要包含网络权限
每个版本同样也创建了额外的资源集：

* `android.sourceSets.flavor1Debug`位于`src/flavor1Debug`
* `android.sourceSets.flavor1Release`位于`src/flavor1Release`
* `android.sourceSets.flavor2Debug`位于`src/flavor2Debug`
* `android.sourceSets.flavor2Release`位于`src/flavor2Release`

他们比构建类型拥有更高的优先级，并且允许自定义版本等级。
## 5.5构建和任务
我们之前看到的，每一种构建类型创建自己的`assmble名字`任务，但是构建版本是构建类型和产品渠道的组合。
当使用产品类型时，更多的任务会被创建，如：

1.  `assemble构建版本名`
2. `assemble构建类型名`
3. `assemble产品渠道名`

第1项允许你直接构建单个的版本，例如`aseembleFlavor1Debug`。

第2项允许你构建所有已给构建类型的apk文件。例如`assembleDebug`将会构建`flavor1Debug`和`flavor2Debug`。

第3项允许你构建给定渠道的所有APK文件。例如，`assembleFlavor1`将会构建`assembleFlavor1Debug`和`assembleFlavor1Release`。

任务`assemble`会构建有可能的版本。

## 5.6多渠道版本

在一些情况下，可能会想要基于多种条件下创建相同应用的多个版本。
考虑一个游戏作为例子，该游戏有一个demo版本和一个付费版本并且想要使用多apk支持中的ABI过滤条件。3个ABI和2个版本需要生成6个APK文件（不考虑构建类型）。
但是，支付版本的代码对于相对应的3个ABI版本是相同的，因此创建6个渠道并不是一个好方法。
取而代之的是，配置两个渠道版本，所有的构建版本应该自动构建可能的组合。
这个功能是通过渠道规格(flavor dimension)来实现的。渠道被设置到指定的规格：

```groovy
android {
    //...


    flavorDimensions "abi", "version"


    productFlavors {
        freeapp {
            dimension "version"
            //...
        }

        paidapp {
            dimension "version"
            ...
        }


        arm {
            dimension "abi"
            ...
        }
        
        mips {
            dimension "abi"
            ...
        }

       x86 {
           dimension "abi"
            ...
        }
    }
}
```

`android.flavorDimensions`数组定义了可能的规格。每一个产品渠道指定一个规格。
从指定了规格的产品渠道(free app,paid app)以及构建类型（debug，release），能够创建如下的构建版本：

* x86-freeapp-debug
* x86-freeapp-release
* arm-freeapp-debug
* arm-freeapp-release
* mips-freeapp-debug
* mips-freeapp-release
* x86-paidapp-debug
* x86-paidapp-release
* arm-paidapp-debug
* arm-paidapp-release
* mips-paidapp-debug
* mips-paidapp-release

规格的顺序是通过` android.flavorDimensions`定义的，这一点非常重要。
每一个版本通过不同的产品渠道进行配置：

* `android.defaultConfig`
* abi规格
* 版本规格

规格的顺序驱动了是哪个渠道去覆盖哪个渠道，这对于渠道的哪个资源值去替代另一个渠道的资源值而言非常重要。
渠道规格通过高优先级进行定义。在这种情况下：

`abi`\>`version`\>`defaultConfig`
多渠道项目也拥有额外的资源集，和构建版本资源集相似但是不包括构建类型：

* `android.sourceSets.x86Freeapp`位于`src/x86Freeapp`
* `android.sourceSets.armPaidapp`位于`src/armPaidapp`

这样允许渠道组合级别的自定义。他们比基本的渠道资源集拥有更高的优先级，但是优先级低于构建类型资源集。

## 5.7测试

多渠道项目测试和简单的项目测试非常相似。
`androidTest`资源集用于所有渠道的通用测试，而每个渠道可以有自己的测试。
如上所提及的，每个渠道的资源集按照如下形式被创建：

* `android.sourceSets.androidTestFlavor1`位于`src/androidTestFlavor1`
* `android.sourceSets.androidTestFlavor2`位于` src/androidTestFlavor2/`
相似地，它们可以有各自的依赖：

```groovy
dependencies {
    androidTestFlavor1Compile "..."
}
```

运行测试可以通过祖先任务`deviceCheck`完成，或者通过主`androidTest`任务。
每一个渠道都有自己的测试任务：`androidTest版本名`，例如：

* `androidTestFlavor1Debug`
* `androidTestFlavor2Debug`

相似地，测试APK的构建和卸载安装按照每个版本进行：

* assembleFlavor1Test
* installFlavor1Debug
* installFlavor1Test
* uninstallFlavor1Debug

最后，HTML报告生成支持通过渠道的汇总。
测试结果和报告的位置如下所示，首先是渠道版，其次是汇总版：

* build/androidTest-results/flavors/渠道名
* build/androidTest-results/all/
* build/reports/androidTests/flavors/渠道名
* build/reports/androidTests/all/

或者可以自定义路径，但这仅仅改变了文件夹的根目录，但是仍然会对每个渠道创建子目录并汇集报告结果。

## 5.8构建配置

Android Studio生成一个叫作`BuildConfig`的类，包含了用于构建一个特定版本的常量值。你可以指定这些常量值来改变不同版本，例如：

```java
private void javaCode() {
    if (BuildConfig.FLAVOR.equals("paidapp")) {
        doIt();
    else {
        showOnlyInPaidAppDialog();
    }
}

```
以下是构建配置中含有的值：

* `boolean DEBUG` – if the build is debuggable.
* `int VERSION_CODE`
* `String VERSION_NAME`
* `String APPLICATION_ID`
* `String BUILD_TYPE` - 构建类型的名字，例如”release”
* `String FLAVOR` – 渠道名称，例如”paid app”

如果项目使用渠道规格，将会生成额外的规格。使用上面的例子，会有如下的构建配置示例：

* `String FLAVOR = "armFreeapp"`
* `String FLAVOR_abi = "arm"`
* `String FLAVOR_version = "free app"`

## 5.9过滤版本

当你添加规格和渠道时，你应该停止使用没有意义的版本。例如你可能为了更快的测试，会定义一个使用你的Web API的渠道，以及一个使用硬编码的假数据。第二个渠道仅仅在开发的时候有用，但是在发布版本的构建时却没有用处。你可以删除这个版本，改成使用`variantFilter`，例如：

```groovy
android {
    productFlavors {
        realData
        fakeData
    }

    variantFilter { variant ->
        def names = variant.flavors*.name

        if (names.contains("fakeData") && variant.buildType.name == "release") {
            variant.ignore = true
        }
    }
}
```

在上面的配置中，你的项目只有三个版本：

* `realDataDebug`
* `realDataRelease`
* `fakeDataDebug`

# 6.高级构建自定义

## 6.1运行混淆

混淆插件在Android插件中被自动使用，如果构建类型通过`minifyEnabled`属性开启了混淆，那么任务会被自动创建。

```groovy
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFile getDefaultProguardFile('proguard-android.txt')
        }
    }

   productFlavors {
        flavor1 {
        }
        flavor2 {            proguardFile 'some-other-rules.txt'        
        }

    }
}
```

版本使用所有在这个构建类型以及产品渠道中声明的规则文件。
有两个默认规则文件：

* proguard-android.txt
* proguard-android-potimize.txt

这些文件位于SDK中。使用`getDefaultProguardFile()`会返回文件的全路径。这些文件是相同的，除非开启了优化。

## 6.2缩减资源

你可以在构建时自动移除所有未使用的资源。关于更多信息，请查阅[资源缩减](http://tools.android.com/tech-docs/new-build-system/resource-shrinking)文档。

## 6.3操作任务

基本的Java项目拥有有限的任务集共同创建输出。
`classes`是编译java源代码的任务之一，缩写为：`project.tasks.classes`。
在Android项目中有一些复杂。这是因为可能有大量相同的任务，并且它们的名字基于产品渠道和构建类型命名。
为了去解决这一点，`android`对象有以下两个属性：

* `applicationVariants`(只用于app插件)
* `libraryVariants`(只用于库插件)
* `testVariants`(同时适用于app和库插件)

三者分别返回`ApplicationVariant`，`LibraryVariant`，`TestVariant`的[领域对象集合](https://docs.gradle.org/current/javadoc/org/gradle/api/DomainObjectCollection.html)。

注意到，访问以上三种集合的任意一种都会引发所有任务的创建。这意味着在访问集合后不会发生任何重新配置的情况。
领域对象集合直接访问所有对象，或者通过更方便的过滤器。

```groovy
android.applicationVariants.all { variant ->
   ....
}
```

所有版本类都含有以下属性：

1. name 类型为String，表示版本名称，要求不重复。
2. description 类型为String，人类可读的版本描述。
3. dirName 类型为String，版本的子文件夹名称，要求不重复。可能会有多个文件夹，例如”debug/flavor”。
4. baseName 类型为String，输出版本的基本名称，要求不重复。
5. outputFile 类型为File，版本的输出。拥有读/写属性。
6. processManifest 类型为ProcessManifest，用于处理清单的任务。
7. aidlCompile 类型为AidlCompile，编译AIDL文件的任务。
8. renderscriptCompile 类型为RenderscriptCompile，编译Renderscript的任务。
9. mergeResources 类型为MergeResources，合并资源文件的任务。
10. mergeAssets 类型为MergeAssets，合并assets的任务。
11. processResources 类型为ProcessAndroidResources，用于处理和编译资源的任务。
12. generateBuildConfig 类型为GenerateBuildConfig,生成BuildConfig类的任务
13. javaCompile 类型为JavaCompile，编译Java代码的任务。
14. processJavaResources 类型为Copy，处理Java资源的任务。
15. assemble 类型为DefaultTask，对版本输出的祖先任务。
ApplicationVariant类添加了如下内容：
1. buildType 类型为BuildType，版本的构建类型。
2. productFlavors 类型为List\<ProductFlavor\>,版本的产品渠道可以不设置但是永远不为空。
3. mergedFlavor 类型为ProductFlavor,android.defaultConfig和variant.productFlavors的合并。
4. signingConfig 类型为SigningConfig,该版本的签名配置。
5. isSigningReady 类型为boolean。为真时该版本拥有签名的所有必需信息。
6. testVariant 类型为BuildVariant。TestVariant会测试该版本。
7. dex 类型为Dex，编译代码的任务。如果是个库项目可以为空。
8. packageApplication 类型为PackageApplication，构建最终APK的任务。如果是个库项目可以为空。
9. zipAlign 类型为ZipAlign。打包apk的任务，如果是个库项目或者APK不能被签名时可以为空。
10. install 类型为DefaultTask，安装任务，可以为空。
11. unstall 类型为DefaultTask，卸载任务。
LibraryVariant类添加了一下内容：
1. buildType 类型为BuildType，版本的构建类型。
2. mergedFlavor 类型为ProductFlavor，默认配置值。
3. testVariant 类型为BuildVariant，要测试的构建版本。
4. packageLibrary 类型为Zip，打包库AAR压缩包的任务，如果不是库项目则为空。
TestVariant类添加了如下内容：
1. buildType 类型为BuildType，版本的构建类型。
2. productFlavors 类型为List\<ProductFlavor\>,版本的产品渠道可以不设置但是永远不为空。
3. mergedFlavor 类型为ProductFlavor,android.defaultConfig和variant.productFlavors的合并。
4. signingConfig 类型为SigningConfig,该版本的签名配置。
5. isSigningReady 类型为boolean。为真时该版本拥有签名的所有必需信息。
6. testVariant 类型为BuildVariant。TestVariant会测试该版本。
7. dex 类型为Dex，编译代码的任务。如果是个库项目可以为空。
8. packageApplication 类型为PackageApplication，构建最终APK的任务。如果是个库项目可以为空。
9. zipAlign 类型为ZipAlign。打包apk的任务，如果是个库项目或者APK不能被签名时可以为空。
10. install 类型为DefaultTask，安装任务，可以为空。
11. unstall 类型为DefaultTask，卸载任务。
12. connectedAndroidTest 类型为DefaultTask，在连接设备上运行Android测试的任务。
13. providerAndroidTest 类型为DefaultTask，使用扩展API运行Android测试的任务。
Android特定任务类型API：
* ProcessManifest
	* 文件 manifestOutputFile
* AidlCompile
	* 文件 sourceOutputDir
* RenderscriptCompile
	* 文件 sourceOutputDir
	* 文件 resOutputDir
* MergeResources
	* 文件 outputDir
* MergeAssets
	* 文件 outputDir
* ProcessAndroidResources
	* 文件 manifestFile
	* 文件 resDir
	* 文件 assetsDir
	* 文件 sourceOutputDir
	* 文件 textSymbolOutputDir
	* 文件 packageOutputFile
	* 文件 proguardOutputFile
* GenerateBuildConfig
	* 文件 sourceOutputDir
* Dex
	* 文件 outputFolder
* PackageApplication
	* 文件 resourceFile
	* 文件 dexFile
* File javaResourceDir
	* 文件 jniDir
	* 文件 outputFile
		* 在版本对象直接使用”outputFile”改变最终输出文件
* ZipAlign
	* 文件 inputFile
	* 文件 outputFile
		* 在版本对象直接使用”outputFile”改变最终输出文件

每一个任务类型的APK是有限的，这是由于Gradle的工作方式以及Android插件是如何建立的。
首先，Gradle的任务仅仅配置了输入输出的位置以及可能的选项。因此，任务仅仅定义了输入输出。
其次，大多数任务的输入并不是无关紧要的，通常来自于资源集和构建类型以及产品渠道的混合。为了保持构建文件易于阅读和理解，开发者通过领域特定语言对这些对象进行微调就能够进行构建，而不是深入改变项目的输入和选项。
另外也需要注意到的是，除了ZipAlign任务类型，所有的任务需要设置私有的数据才能工作。这意味着不可能手动创建这些类型的新任务。
主观上API是能够修改的。通常情况下当前的API是围绕着给定的输入输出（当任务能够添加额外必须的处理时）。反馈是很重要的，特别是一些需求不可见时。
关于Gradle的其他任务（DefaultTask, JavaCompile, Copy, Zip），请参考Gradle文档。

## 6.3设置Java语言等级

你可以使用`compileOptions`代码块选择编译器的语言等级，默认情况下是基于`compileSdkVersion`的值。

```groovy
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_6
        targetCompatibility JavaVersion.VERSION_1_6
    }
}
```

