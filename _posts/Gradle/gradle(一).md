---
date: 2016-05-30 23:56
status: public
title: Gradle(一)
categories: Gradle
---

###Task
```
###gradle基本语法
基本语法认识文章(http://www.cnblogs.com/davenkin/p/gradle-learning-3.html)[http://www.cnblogs.com/davenkin/p/gradle-learning-3.html]
task showDescription1 << {
   description = 'this is task showDescription'
   println description
}
task showDescription2 << {
   println description
}
showDescription2.description = 'this is task showDescription'
task showDescription3 << {
   println description
}
showDescription3 {
   description = 'this is task showDescription'
}
```
以上3个Task完成的功能均相同，即先设置Task的description属性，在将其输出到命令行。
1. 对于每一个Task，Gradle都会在Project中创建一个同名的Property，所以我们可以将该Task当作Property来访问，showDescription2便是这种情况。另外，Gradle还会创建一个同名的方法，该方法接受一个闭包，我们可以使用该方法来配置Task，showDescription3便是这种情况。`
2. Groovy中的Bean和Java中的Bean有一个很大的不同，即Groovy为每一个字段都会自动生成getter和setter，并且我们可以通过像访问字段本身一样调用getter和setter
3. 闭包的delegate代理机制。Gradle大量地使用了Groovy闭包的delegate机制。简单来说，delegate机制可以使我们将一个闭包中的执行代码的作用对象设置成任意其他对象

###增量构建
1.如果我们将Gradle的Task看作一个黑盒子，那么我们便可以抽象出输入和输出的概念，一个Task对输入进行操作，然后产生输出。比如，在使用java插件编译源代码时，输入即为Java源文件，输出则为class文件。如果多次执行一个Task时的输入和输出是一样的，那么我们便可以认为这样的Task是没有必要重复执行的。此时，反复执行相同的Task是冗余的，并且是耗时的。
2.为了解决这样的问题，Gradle引入了`增量式构建的概念`。在增量式构建中，我们为`每个Task定义输入（inputs）和输入（outputs），如果在执行一个Task时，如果它的输入和输出与前一次执行时没有发生变化，那么Gradle便会认为该Task是最新的（UP-TO-DATE）`，因此Gradle将不予执行。
3.一个Task的inputs和outputs可以是一个或多个文件，可以是文件夹，还可以是Project的某个Property，甚至可以是某个闭包所定义的条件。
```
task combineFileContentIncremental {
   def sources = fileTree('sourceDir')
   def destination = file('destination.txt')
   inputs.dir sources
   outputs.file destination
   doLast {
      destination.withPrintWriter { writer ->
         sources.each {source ->
            writer.println source.text
         }
      }
   }
}
```
第一次执行此task，gradle会完整的执行该task。但是输入输出没有改变的情况下再执行改task，combineFileContentIncremental`被标记为UP-TO-DATE`，表示该Task是最新的，Gradle将不予执行。在实际应用中，你将遇到很多这样的情况，因为Gradle的很多插件都引入了增量式构建机制。
如果我们修改了inputs（即sourceDir文件夹）中的任何一个文件或删除掉了destination.txt，当调用“gradle combineFileContentIncremental”时，Gradle又会重新执行，因为此时的Task已经不再是最新的了。
```
:combineFileContentIncremental UP-TO-DATE
BUILD SUCCESSFUL
Total time: 2.104 secs
```