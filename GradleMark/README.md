# Gradle mark

## Gradle基础
### 参考文档
- [极客学院gradle](http://wiki.jikexueyuan.com/project/android-studio-guide/gradle-channel-package.html)
- [Gradle for Android 问题总结](http://www.jianshu.com/p/9dcec4a14c52)
- [Android Studio中的productFlavors指定默认编译执行的任务](https://www.mobibrw.com/2016/3782)

### gradle采用驼峰命名的方式组合不同的命令
### Android studio中使用gradle的一些命令如./gradlew -v ，./gradlew clean，./gradlew build，这里注意是./gradlew, ./代表当前目录，gradlew代表 gradle wrapper，意思是gradle的一层包装，大家可以理解为在这个项目本地就封装了gradle，即gradle wrapper， 在App主目录/gradle/wrapper/gralde-wrapper.properties文件中声明了它指向的目录和版本。只要下载成功即可用grdlew wrapper的命令代替全局的gradle命令
1. ./gradlew -v 查看当前gradle版本
2. ./gradlew clean 清除app/build文件夹中的内容
3. ./gradlew build 检查依赖并编译打包，会把debug、release环境的包都打出来
4. ./gradlew assemble<Build Type Name>
./gradlew assembleDebug 编译并只打Debug包
./gradlew assembleRelease 编译并只打Release的包
5. 与productFlavors结合打包，比如./gradlew assembleFlavor1会只打Flavor1的包
6. 与buildType结合打包，比如 ./gradlew assembleDebug 会只打各个flavor的测试包
7. 与productFlavors和buildType结合打包，比如./gradlew assembleFlavor1Release会只打Flavor1渠道的Release包
8. ./gradlew installRelease， Release模式打包并安装（userCare中没有成功，有待调查）
9. ./gradlew uninstallRelease 卸载Release模式包（userCare中没有成功，有待调查）

### project中的build.gradle说明

buildscript {
     //构建过程依赖的仓库
    repositories {
        jcenter()
    }

    //构建过程需要依赖的库
    dependencies {
        //声明的是gradle插件的版本
        classpath 'com.android.tools.build:gradle:2.0.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    //这里面配置整个项目依赖的仓库,这样每个module就不用配置仓库了
    repositories {
        jcenter()

        maven {
            // LeanCloud 的包仓库
            url "http://mvn.leancloud.cn/nexus/content/repositories/releases"
        }
    }
}

//配置全局变量，可以在Module下的build.gradle中调用
ext {
    // module依赖库公共版本号
    SupportXVersion = '23.2.0'
    GsonVersion = '2.6.2'
    LeanCloudVersion = 'v3.13.4'
    JunitVersion = '4.12'

    compileSdkVersion = 22
    buildToolsVersion = "23.0.1"
    minSdkVersion = 10
    targetSdkVersion = 22
    versionCode = 34
    versionName = "v2.6.1"
}

- 大家可能很奇怪，为什么仓库repositories需要声明两次，这其实是由于它们作用不同，buildscript中的仓库是gradle脚本自身需要的资源，而allprojects下的仓库是项目所有模块需要的资源
- project下的build.gradle是基于整个project的配置，主要配置gradle 版本及 全局依赖仓库、库或者其他全部参数。
- android studio 现在重要仓库采用jcenter(),之前版本放在mavenCentral。另外有时还没有加入jcenter()仓库的第三方库，也需要在这里配置他们的库地址。需要在这里配置，才能将第三方库拉下来

### Module 中的build.gradle
//申明使用插件，表明要编译的内容和产物，
//com.android.application 表明该module 为android 应用
//com.android.library 表明为library库
//java 表名是java库
apply plugin: 'com.android.application'

//安卓构建过程需要配置的参数
android {
    //编译SDK的版本
    compileSdkVersion COMPILE_SDK_VERSION as int
    //buildtool 的版本
    buildToolsVersion BUILD_TOOLS_VERSION

    /默认配置，会同时应用到debug和release版本上
    defaultConfig {
        //应用包名
        applicationId APPLICATION_ID
        //支持最小android sdk 版本
        minSdkVersion MIN_SDK_VERSION as int
        // 目标版本
        targetSdkVersion TARGET_SDK_VERSION as int
        //应用版本号
        versionCode VERSION_CODE as int
        //应用版本名称
        versionName VERSION_NAME

        /// 配置生成的 BuildConfig 文件中的常量，代码引用直接 
        buildConfigField "String", "LOG_TAG", LOG_TAG // 日志tag
        buildConfigField "String", "LOG_HTTP_TAG", LOG_TAG_HTTP // http日志tag
        buildConfigField "String", "LOG_WEB_TAG", LOG_TAG_WEB // web日志tag

      // 默认是UMENG_CHANNEL_VALUE为umeng
        manifestPlaceholders = [UMENG_CHANNEL_VALUE: "umeng"]

      // dex突破65535的限制
        multiDexEnabled true
    }

    //java版本号
    compileOptions{
        sourceCompatibility JavaVersion.VERSION_1_7
        targetCompatibility JavaVersion.VERSION_1_7
    }

    //签名
    signingConfigs {
        release {
            //签名文件
            storeFile file(STORE_FILE)
            storePassword STORE_PASSWORD
            keyAlias KEY_ALIAS
            keyPassword KEY_PASSWORD
        }
    }
    // 为了解决部分第三方库重复打包了META-INF的问题
    packagingOptions {
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }

    //移除lint检测的error
    lintOptions {
        abortOnError false
    }

    //编译类型
    //其中debug, release是gradle默认自带的两个build type， 当然你可以定义其他类型。
    //可以针对不停编译的版本中配置不同的参数，比如混淆、签名等。preview
    buildTypes {
        debug {
            minifyEnabled false
            zipAlignEnabled false
            shrinkResources false
        }

         preview {
            debuggable false // 是否保留调试信息
            minifyEnabled true  //是否混淆
            zipAlignEnabled true // 包优化
            shrinkResources true // 移除不必要的资源

            // 签名
            signingConfig signingConfigs.release
            // 代码混淆规则文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }

        release {
            //加上后缀
            applicationIdSuffix ".release"
            minifyEnabled true //是否混淆
            zipAlignEnabled true // zip对齐优化
            shrinkResources true // 移除不必要的资源

            // 不显示Log
            buildConfigField "boolean", "LOG_DEBUG", "false"

            // 签名
            signingConfig signingConfigs.release
            //混淆文件的位置
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    // 多渠道
    productFlavors {
        //可以设置不同渠道渠道号，应用名称
        dev { // 开发
            buildConfigField "String", "CHANNEL_NUMBER", '"11111"'
        }
        '360' {
            buildConfigField "String", "CHANNEL_NUMBER", '"11112"'
        }
        GooglePlay {
            buildConfigField "String", "CHANNEL_NUMBER", "11113"'

    }
    // 多渠道批量替换
    productFlavors.all { flavor ->
         //批量修改Manifest占位符替换
        //在Manifest使用`${UMENG_CHANNEL_VALUE}`,`LEANCLOUD_CHANNEL_VALUE`,打包时将替换成渠道名，例如UMENG_CHANNEL_VALUE="dev";
        flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name, LEANCLOUD_CHANNEL_VALUE: name]
        // Project Properties->_myAPPBuildVersionName，用于程序集成下命令行修改
        if (project.hasProperty('_myAPPBuildVersionName')) {
            defaultConfig.versionName = _myAPPBuildVersionName
        }
    }

     //定义变量
     //gradle 可以用def定义一些值例如：def KeyPassword = "123123" 
     def releaseTime() {
         return new Date().format("yyyy-MM-dd", TimeZone.getTimeZone("UTC"))
     }


    // 批量打包
    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            def outputFile = output.outputFile
            if (outputFile != null && outputFile.name.endsWith('.apk')) {
                def fileName
                if (variant.buildType.name.equals('release')) {
                    // myAPP_v版本号_渠道名.apk
                    fileName = "myAPP_v${variant.versionName}_${variant.productFlavors[0].name}.apk"
                } else {
                    // myAPP_v版本号_渠道名_时间_编译类型名.apk
                    fileName = "myAPP_v${variant.versionName}_${variant.productFlavors[0].name}_${releaseTime()}_${variant.buildType.name}.apk"
                }
                output.outputFile = new File(outputFile.parent + "/${variant.buildType.name}", fileName)
            }
        }
    }
 }

//依赖第三方库
dependencies {
   //编译libs目录下所以jar包
   //compile files('libs/xxx.jar')   导入一个特定jar包
    compile fileTree(dir: 'libs', include: ['*.jar'])//导入所有的jar包
    compile project(':core')
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:recyclerview-v7:23.1.1'
    compile 'com.android.support:cardview-v7:23.1.1'
}

- buildToolsVersion这个需要你本地安装该版本才行，很多人导入新的第三方库，失败的原因之一是build version的版本不对，这个可以手动更改成你本地已有的版本或者打开 SDK Manager 去下载对应版本。
- proguardFiles这部分有两段，前一部分代表系统默认的android程序的混淆文件，该文件已经包含了基本的混淆声明，免去了我们很多事，这个文件的目录在 /tools/proguard/proguard-android.txt , 后一部分是我们项目里的自定义的混淆文件，目录就在 app/proguard-rules.pro ,在这个文件里你可以声明一些第三方依赖的一些混淆规则，最终混淆的结果是这两部分文件共同作用的。
- 一般重要的信息，例如签名信息，可以直接将信息写到gradle.properties，然后在然后在build.gradle中引用即可。
- 多渠道的关键在于定义不同的product flavor。注意:这里的flavor名如果是数字开头，必须用引号引起来。
- buildTypes是指建构的类型，一般只用两种默认类型 debug 和 release ，顾名思义 debug 用来配置开发过程中的一些内容；release 用来配置正式发布版本的内容。有时我们需要发布介于debug与release之间的preview 版本。

### Project中setting.gradle
这个文件是全局的项目配置文件，里面主要声明Project中所包括的所有module，

//一个Project中所包括的所有module
include ':app', ':model',':lib', ':core'

### Project中gradle.properties

gradle.properties为gradle的配置文件，里面可以定义一些常量供build.gradle使用，比如可以配置签名相关信息如keystore位置，密码，keyalias等,build.gradle就可以直接引用
gradle 中的一些配置参数建议写到gradle.properties

//编译版本信息
APPLICATION_ID = com.jin.myAPP
COMPILE_SDK_VERSION = 23
BUILD_TOOLS_VERSION = 23.0.1
MIN_SDK_VERSION = 15
TARGET_SDK_VERSION = 1
VERSION_CODE = 1
VERSION_NAME = 1.0.0.0

//keystore信息
STORE_FILE = ../app/mykey.keystore
STORE_PASSWORD = your password
KEY_ALIAS = your alias
KEY_PASSWORD = your password

### BuildConfig

在build.gradle中配置buildConfigField参数，编译后会在..\app\build\generated\source\buildConfig文件夹下会自动生成对应版本对应module的BuildConfig.java。BuildConfig就会包含对应版本的配置信息。程序中可以直接引用这些数据。例如BuildConfig.DEBUG。

public final class BuildConfig {
    public static final boolean DEBUG = Boolean.parseBoolean("true");
    public static final String BUILD_TYPE = "debug";
    public static final String FLAVOR = "360";
    public static final int VERSION_CODE = 45;
    public static final String VERSION_NAME = "3.2.0.0";
    // Fields from build type: debug
    public static final String HOST_IMG_SERVER = "img";
    public static final String HOST_SERVER = "test";
    // Fields from product flavor: 360
    public static final String CHANNEL_NUMBER = "232100";
}

### module 调整目录结构sourceSets

默认情况下，java文件和resource文件分别在src/main/java和src/main/res目录下，在build.gradle文件，andorid{}里面添加下面的代码，便可以将java文件和resource文件放到src/java和src/resources目录下。

sourceSets {
   main {
      java {
          srcDir 'src/java'
      }
      resources {
        srcDir 'src/resources'
     }
   }
}

更简便的写法是：

sourceSets {
   min.java.srcDirs = ['src/java']
   min.resources.srcDirs = ['src/resources']
}


### 引用本地aar：

    首先将你的库通过android studio 运行打包成aar文件。运行后在/build/output/aar文件夹中。
    首先将aar文件放到模块的libs目录下，然后在该模块的build.gradle中声明flat仓库：

    repositories{
       flatDir {
          dirs 'libs'
      }
    }

    最后在dependencies结点下依赖该aar模块：

    dependencies{
      compile (name:'xxx',ext:'aar')
    }

## Gradle进阶

### Android Studio中的productFlavors指定默认编译执行的任务
	productFlavors {
        adefault {
            applicationId "com.gpc.gradlesetting"
            manifestPlaceholders.put("TEXT_CHANNEL_VALUE", 'dev')
            buildConfigField("String", "TEST_SERVER_URL", "\"http://test.com\"")
        }
        xiaomi {
            applicationId "com.gpc.gradlesetting.xiaomi"
            manifestPlaceholders.put("TEXT_CHANNEL_VALUE", 'xiaomi')
            buildConfigField("String", "TEST_SERVER_URL", "\"http://xiaomi.com\"")
        }
        baidu {
            applicationId "com.gpc.gradlesetting.baidu"
            manifestPlaceholders.put("TEXT_CHANNEL_VALUE", 'baidu')
            buildConfigField("String", "TEST_SERVER_URL", "\"http://baidu.com\"")
        }
        tecent {
            applicationId "com.gpc.gradlesetting.tecent"
            manifestPlaceholders.put("TEXT_CHANNEL_VALUE", 'tecent')
            buildConfigField("String", "TEST_SERVER_URL", "\"http://tecent.com\"")
        }
    	}
我们点击 Android Studio的调试按钮的时候，不知道究竟是使用哪个 Flavors来编译，比如在 Android Studio 1.5的时候,是按照从上到下的顺序处理的，默认是使用排在第一个的 adefault,而到了 Android Studio 2.1 Preview 5版本，结果变成了默认编译 baidu。

选择" Build Variant",然后在出现的窗口中选择其中一个选项作为默认的编译，运行选项即可。

![](https://github.com/gupengcheng/AndroidMark/tree/master/GradleMark/screen.jpg)

