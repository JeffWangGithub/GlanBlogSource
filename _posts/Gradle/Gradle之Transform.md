---
title: Gradle之Transform
date: 2017-02-15 15:04:34
tags: transform API, gradle
categories: Gradle
---
### Transfrom 
Trasnform API是gradle 1.5.0之后版本提供的，该API的作用是:允许第三方在打包Dex文件之前的编译过程中操作.class文件， 目前Android gradle插件中的jarMerge, proguard, multi-dex，instantRun等一些列操作都已经转换为Transform的实现了。
#### 如何使用Transform API
使用Transform基本分两个步骤:
1. 先继承Transform类实现子类 
2. 将实现的Transform进行注册进android编译流程中  android.registerTransform(transform) 或android.registerTransform(theTransform, dependencies)
#### Transform类中必须的实现
1. getName()
 Transform中的抽象方法,实现之后返回当前Transform子类的名字.该方法return出去的名字并不是最终编译时的名字。
gradle core的代码中可以找到Transform的管理类， com.android.build.gradle.internal.pipeline.TransformManager在此类中会根据transform返回的name按照transform${InputType1}And${InputType2}And${InputTypeN}And${name}For${flavor}${BuildType}的形式进行拼装成完成的名字

    ```
    private static String getTaskNamePrefix(@NonNull Transform transform) {
        StringBuilder sb = new StringBuilder(100);
        sb.append("transform");
        Iterator<ContentType> iterator = transform.getInputTypes().iterator();
        // there's always at least one
        sb.append(capitalize(iterator.next().name().toLowerCase(Locale.getDefault())));
        while (iterator.hasNext()) {
            sb.append("And").append(capitalize(
                    iterator.next().name().toLowerCase(Locale.getDefault())));
        }
        sb.append("With").append(capitalize(transform.getName())).append("For");
        return sb.toString();
    }
    ```
2. getInputTypes()
指名当前的Transform要处理的数据类型。 具体类型包含两种:
-  QualifiedContent.DefaultContentType.CLASSES 代表要处理的数据是编译之后字节码， 这些数据的容器可以是jar包也可以是文件夹 
-  QualifiedContent.DefaultContentType.RESOURCES标识要处理的是标准的java资源
    ```
        /**
         * 指明当前Trasfrom要处理的数据类型
         */
        @Override
        public Set<QualifiedContent.ContentType> getInputTypes() {
            //ImmutableSet 表示不可变的set
            //此处的inputType包含两个大类
            //1. CLASSES标识要处理的数据是编译过后的java字节码，这些数据的容器可以是jar包也可以是文件夹
            //2. RESOURCES标识要处理的是java资源
            return ImmutableSet.<QualifiedContent.ContentType>of(QualifiedContent.DefaultContentType.CLASSES);
        }
    ```
3. getScopes()
配置当前Transform的影响范围,相当于作用域.其中Scope的属性解释如下图所示.
|Type	| Des|
|--------|-------|-------|
|PROJECT|	只处理当前项目|
|SUB_PROJECTS|  只处理子项目|
|PROJECT_LOCAL_DEPS	|只处理当前项目的本地依赖,例如jar, aar|
|SUB_PROJECTS_LOCAL_DEPS	|只处理子项目的本地依赖,例如jar, aar|
|EXTERNAL_LIBRARIES|	只处理外部的依赖库|
|PROVIDED_ONLY|	只处理本地或远程以provided形式引入的依赖库|
|TESTED_CODE|	测试代码|
跟getInputTypes方法一样, getScopes也是返回一个集合,那么就可以根据自己的需求配置多种Scope
4. isIncremental()
当前Transform是否支持增量编译










