---
date: 2016-04-05 11:34
status: public
title: Gradle相关
categories: Gradle
---

1. 基础Java项目有一组有限的task用于互相处理生成一个输出。 classes是一个编译Java源代码的task。可以在build.gradle文件中通过脚本很容易使用classes。这是project.tasks.classes的缩写。
在Android项目中，相比之下这就有点复杂。因为Android项目中会有大量相同的task，并且它们的名字基于Build Types和Product Flavor生成。
为了解决这个问题，android对象有两个属性：
- applicationVariants（只适用于app plugin）
- libraryVariants（只适用于library plugin）
- testVariants（两个plugin都适用）
这三个都会分别返回一个ApplicationVariant、LibraryVariant和TestVariant对象的DomainObjectCollection（`域名对象集合`）。
DomainObjectCollection可以直接访问所有对象，或者通过过滤器进行筛选。
```
android.applicationVariants.each { variant ->
    ....
}
```
这三个variant类都共享下面的属性：

|属性名|	属性类型	|说明|
|-----|--------|----|
|name	|String	|Variant的名字，必须是唯一的。|
|description	|String	|Variant的描述说明。|
|dirName	|String	|Variant的子文件夹名，必须也是唯一的。可能也会有不止一个子文件夹，例如“debug/flavor1”|
|baseName|	String	|Variant输出的基础名字，必须唯一。|
|outputFile	|File	|Variant的输出，这是一个可读可写的属性。|
|processManifest	|ProcessManifest	|处理Manifest的task。|
|aidlCompile	|AidlCompile	|编译AIDL文件的task。|
|renderscriptCompile	|RenderscriptCompile	|编译Renderscript文件的task。|
|mergeResources	|MergeResources	|混合资源文件的task。|
|mergeAssets	|MergeAssets	|混合asset的task。|
|processResources	|ProcessAndroidResources	|处理并编译资源文件的task。|
|generateBuildConfig|	GenerateBuildConfig|	生成BuildConfig类的task。|
|javaCompile|	JavaCompile|	编译Java源代码的task。|
|processJavaResources|	Copy|	处理Java资源的task。|
|assemble|	DefaultTask|	Variant的标志性assemble task。|

ApplicationVariant类还有以下附加属性：

|属性名|	属性类型	|说明|
|----|-------|-----|
|buildType|	BuildType|	Variant的BuildType。|
|productFlavors|	List|	Variant的ProductFlavor。一般不为空但也允许空值。|
|mergedFlavor|	ProductFlavor	|android.defaultConfig和variant.productFlavors的合并。|
|signingConfig	|SigningConfig	|Variant使用的SigningConfig对象。|
|isSigningReady|	boolean	|如果是true则表明这个Variant已经具备了所有需要签名的信息。|
|testVariant	|BuildVariant	|将会测试这个Variant的TestVariant。|
|dex	|Dex	|将代码打包成dex的task。如果这个Variant是个库，这个值可以为空。|
|packageApplication|	PackageApplication|	打包最终APK的task。如果这个Variant是个库，这个值可以为空。|
|zipAlign	|ZipAlign	|zip压缩APK的task。如果这个Variant是个库或者APK不能被签名，这个值可以为空。|
|install|	DefaultTask|	负责安装的task，不能为空。|
|uninstall	|DefaultTask	|负责卸载的task。|

LibraryVariant类还有以下附加属性：

|属性名|	属性类型|	说明|
|-----|-------|-------|
|buildType	|BuildType	|Variant的BuildType.|
|mergedFlavor	|ProductFlavor	|The defaultConfig values|
|testVariant|	BuildVariant|	用于测试这个Variant。|
|packageLibrary	|Zip	|用于打包库项目的AAR文件。如果是个库项目，这个值不能为空。|

2. BuildType是gradle中的一个类型和String等类型一样
BuildType类型的属性和方法[http://google.github.io/android-gradle-dsl/1.3/com.android.build.gradle.internal.dsl.BuildType.html](http://google.github.io/android-gradle-dsl/1.3/com.android.build.gradle.internal.dsl.BuildType.html)
```def buildType = project.android.applicationVariants.buildType```

## # 修改debug版的包名达到可以relase和debug包同时存在
```
buildTypes {
        release {
            debuggable false
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
        }
        debug {
            applicationIdSuffix ".testdebug" //给debug版applicationId增加.testdebug后缀，以便可以同时存在多个版本
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'

        }
    }
```
```
final def variants = project.android.applicationVariants

//使用variant代替了it参数
variants.all{ variant ->
    def buildType = variant.buildType
    def defaultCharset = java.nio.charset.Charset.defaultCharset().toString()
    def appName = "GTapp"
    //更改debug包的包名
    if(buildType.applicationIdSuffix){//applicationIdSuffix属性有值
        variant.mergeResources.doLast{
            def f = file("${buildDir}/intermediates/res/merged/${buildType.getName()}/values/values.xml")
            String content = f.getText(defaultCharset)
            content = content.replaceAll(appName,appName+buildType.applicationIdSuffix)
            f.write(content, defaultCharset)
        }
    }
}
```

 
