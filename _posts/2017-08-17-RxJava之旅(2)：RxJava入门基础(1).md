---
layout: post
title: 'RxJava之旅(2)：RxJava入门基础(1)'
date: 2017-08-17
author: Chuckiefan
tags: RxJava Android
---



# 0.前言

这可能是我拖延最严重的系列文章了，从第一篇[《 RxJava之旅(1)：RxJava思想基础》](https://chuckiefan.github.io/2017/08/14/RxJava之旅(1)-RxJava思想基础.html)到这一篇的间隔差不多有一年，导致每想到这个系列都惭愧不已，甚至好几次冲动差点删了。事情是这样的，去年写完第一篇以后立刻投入到了公司的新项目中，等项目最繁忙的阶段过去已经是2017年初，彼时RxJava2已经发布，导致我觉得这个系列文章失去了继续下去的价值，另一方面由于~~太懒~~一些原因，始终没有去了解RxJava2。最近抽出空来看了一下第一篇的内容，突发想起来事实上RxJava1和RxJava2本质上都是响应式编程，所以继续这个系列是有意义的。接下来我会从RxJava1入手，然后通过RxJava1和RxJava2的不同对RxJava2进行介绍。因此，在RxJava2已经出现的时候，介绍RxJava1是有意义的。

这一篇文章我们会从响应式编程入手，从响应式编程和面向对象编程进行对比，加深响应式编程的理解，从而降低RxJava的学习曲线。

# 1.开始

作为入门，首先让我们来快速地浏览一下RxJava的使用，在你的模块(module)的`build.gradle`中添加依赖：

`compile 'io.reactivex:rxjava:x.y.z'`

其中`x.y.z`替换为最新的版本，关于版本号的查询，请点击[这里](https://maven-badges.herokuapp.com/maven-central/io.reactivex/rxjava)。截止本文时，最新版本为1.3.0，因此配置如下：

    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])        
        androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
        })
        compile 'com.android.support:appcompat-v7:26.0.0-alpha1'
        compile 'com.android.support.constraint:constraint-layout:1.0.2'
    
        testCompile 'junit:junit:4.12'

        //RxJava2
        compile "io.reactivex.rxjava2:rxjava:2.1.2"
        //RxJava1
        compile 'io.reactivex:rxjava:1.3.0'
    }

> 我们可以看到，这里同时配置了RxJava1和RxJava2。为了RxJava1到RxJava2的平稳过渡，在编写RxJava2时使用了不同的包名，这样即使将二者同时放入依赖中也不会造成依赖冲突。因此RxJava1和RxJava2是可以同时使用的，只是要小心查看一下所引入的包名是否正确，以免在应该使用RxJava1的地方引入了RxJava2的类。

# 2.RP是一种怎样的体验

在正式开始之前让我们先耐心地讨论一个问题。上一篇文章我们认真地讨论了什么是RP(Reactive Programming, 响应式编程，下同)，以及响应式编程和面向过程与面向对象编程的特点。

我们这里用一小节的内容来看看同样一个需求用RP和OOP(Object Oriented Programing,面向对象编程，下同)是如何实现的，这样的对比可以帮助更好地理解响应式编程以及RxJava。

## 2.1一个简单的问题

我们的问题非常简单：在Android中有两个输入框用于数字的输入，分别输入数字a和数字b，并计算得到结果c，即a+b=c。

## 2.2界面与初始化

![界面](http://upload-images.jianshu.io/upload_images/1226129-cbcbc086e4ed1eee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个界面非常简单，所以布局文件的代码就不贴出来了。

关于Activity的部分也没有难度，但是有必要贴出来说明每个控件的对应关系：

    //输入框a
    editTextA = (EditText) findViewById(R.id.et_a);
    //输入框b
    editTextB = (EditText) findViewById(R.id.et_b);
    //结果c
    textViewC = (TextView) findViewById(R.id.tv_c);
    //计算按钮c
    button = (Button) findViewById(R.id.btn_cal);

接下来我们分别用OOP和RP的方式来解决这个简单的问题。
## 2.3OOP的解决方案

首先让我们使用面向对象的方式来解决这个问题，很简单这里涉及到三个成员变量，分别是a，b和c，以及用于计算和返回结果的两个方法。我们定义了一个叫做`Plus`的类，代码如下：

```java
    public class Plus {
        private int a;
        private int b;
        private int c;//c没有get或set方法

        public int getA() {
            return a;
        }

        public void setA(int a) {
            this.a = a;
        }

        public int getB() {
            return b;
        }

        public void setB(int b) {
            this.b = b;
        }
        //相加
        public void sum(){
            c = a + b;
        }
        //获取结果
        public int getResult(){
            return c;
        } 
    
    }
```
    


![](http://upload-images.jianshu.io/upload_images/1226129-6056a647bc95bf31.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



我们用OOP的方式将这个问题抽象成一个类，通过set方法为变量赋值，通过sum方法进行计算，最后通过getResult方法获取计算结果。

接下来我们考虑RP的方式是如何解决的。

## 2.4RP的解决方案

在第一篇中我们介绍了响应式编程的核心概念，即*一切皆数据流*的编程思维。那么使用数据流的方式考虑我们这个简单的问题，在a+b=c的公式中，a和b都是数据流，而c是对数据流a和数据流b的响应。就是说通过监听数据流a和数据流b的变化，从而对c进行变化。

问题已经变得清晰了，我们需要两个数据流来表示a和b，然后将二者最新值相加并显示在c的位置。

> 以下部分的代码涉及到了很多还未介绍的RxJava内容，在这里我们不会对此进行细致的介绍，在后面将会逐一讲解

## 2.4.1数据源的创建 

首先来看数据流A和数据流B的创建：

    final Observable<TextViewAfterTextChangeEvent> observableA = RxTextView.afterTextChangeEvents(editTextA).filter(new Predicate<TextViewAfterTextChangeEvent>() {
            @Override
            public boolean test(@NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent) throws Exception {
                return null != textViewAfterTextChangeEvent.editable() && !TextUtils.isEmpty(textViewAfterTextChangeEvent.editable().toString());
            }
        }); 
        
    final Observable<TextViewAfterTextChangeEvent> observableB = RxTextView.afterTextChangeEvents(editTextB).filter(new Predicate<TextViewAfterTextChangeEvent>() {
            @Override
            public boolean test(@NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent) throws Exception {
                return null != textViewAfterTextChangeEvent.editable() && !TextUtils.isEmpty(textViewAfterTextChangeEvent.editable().toString());
            }
        });

这里用到了`RxTextView`，这是RxBinding库中的一个类，用于TextView相关事件的监听。而RxBinding是用于Android UI控件的响应式库，具体的内容我们会在后面的文章具体介绍。在这里我们所用到的`RxTextView.afterTextChangeEvents()`实际上对应着`TextWatcher::afterTextChanged（）`方法，这样相当于使用`editTextA.addTextChangedListener()`方法添加了一个`TextWatcher`，然后监听其`afterTextChanged()`方法，将每次变化后的数据作为新产生的数据项，以供我们后续的处理。

对于RxBinding的引用，在module的`build.gradle`里`dependencies`代码块中配置如下依赖：

`compile 'com.jakewharton.rxbinding2:rxbinding:2.0.0'`


### 2.4.2数据流的订阅（监听）

我们已经定义了数据流，接下来可以去监听数据源的变化了。但是这里还有一个问题：由谁来监听数据流？对两个数据流分别进行监听并不是一个好的方案，这样会破坏RP的响应式设计，所以我们需要一个新的数据流用来将数据流a和数据流b组合在一起，形成一个新的数据流c，然后对数据流c进行订阅，将每次变化的值展示在界面上。

让我们看一下数据流c是如何创建的：

    Observable.combineLatest(observableA, observableB, new BiFunction<TextViewAfterTextChangeEvent, TextViewAfterTextChangeEvent, String>() {
            @Override
            public String apply(@NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent, @NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent2) throws Exception {
                int a = TextUtils.isEmpty(textViewAfterTextChangeEvent.editable().toString())? 0 : Integer.valueOf(textViewAfterTextChangeEvent.editable().toString());
                int b = TextUtils.isEmpty(textViewAfterTextChangeEvent2.editable().toString())? 0 : Integer.valueOf(textViewAfterTextChangeEvent2.editable().toString());
                return String.valueOf(a + b);

            }
        });

这里我们第一次用到了RxJava中的操作符(operator)，操作符是RP中一个很重要的概念，往往用于数据流的转变。如果我们将数据流比作生产的流水线的话，操作符就是流水线上的某些仪器，可以对流水线上的物品进行转换，组合等操作。

这里我们简单介绍一个所用到的`combineLatest`，具体的在后面介绍。`combineLatest`可以将多个数据流（我们这里是两个）的最后一项数据组合在一起，然后生成一个新的数据流继续向下流动。图示如下：


![](http://upload-images.jianshu.io/upload_images/1226129-6d1b45370ee0434e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我们这里取到a和b的最后一个`int`数值，然后将它们加在一起。

# 2.4.3订阅

在得到新生成的数据流c后，我们对数据流c进行订阅，将c的每次变化都展示在界面，代码如下：

    Observable.combineLatest(observableA, observableB, new BiFunction<TextViewAfterTextChangeEvent, TextViewAfterTextChangeEvent, String>() {
            @Override
            public String apply(@NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent, @NonNull TextViewAfterTextChangeEvent textViewAfterTextChangeEvent2) throws Exception {
                int a = TextUtils.isEmpty(textViewAfterTextChangeEvent.editable().toString())? 0 : Integer.valueOf(textViewAfterTextChangeEvent.editable().toString());
                int b = TextUtils.isEmpty(textViewAfterTextChangeEvent2.editable().toString())? 0 : Integer.valueOf(textViewAfterTextChangeEvent2.editable().toString());
                return String.valueOf(a + b);

            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.d(TAG, "onSubscribe");

            }

            @Override
            public void onNext(@NonNull String s) {
                Log.d(TAG, s);
                textViewC.setText(s);//展示结果c
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, e.getMessage());
            }

            @Override
            public void onComplete() {
                Log.d(TAG, "completed");
            }
        });

# 2.4.4 效果

可以注意到我们并没有使用按钮进行处理，而是根据数据的变化自动进行响应。这符合RP的概念，也是RxJava的优点之一。最后展现的结果如图：

![](http://upload-images.jianshu.io/upload_images/1226129-7425a4c5d9aef178.gif?imageMogr2/auto-orient/strip)


# 3.Hello RxJava1!

我们已经看到了使用RxJava进行响应式编程是如何便捷，那么这一节我们终于可以开始学习RxJava1了，让我们从一段最简单的代码开始。

##3.1 一段简单的代码

    rx.Observable.just("Hello world")
                .subscribe(new rx.Observer<String>() {
                    @Override
                    public void onCompleted() {
                        Log.d(TAG,"completed.");
                    }

                    @Override
                    public void onError(Throwable e) {
                        Log.e(TAG,e.getMessage());
                    }

                    @Override
                    public void onNext(String s) {
                        Log.d(TAG,"next msg: " + s);
                    }
                });

这是一段最简单的RxJava代码，我们的入门就从这里开始。

##3.2 创建

`just`方法的完整定义如下：

`public static <T> Observable<T> just(final T value)`

`just`方法用于创建一个被观察者，该方法可以将任何对象转化为可观察对象，其返回类型为`Observable<T>`。

关于创建的细节我们将在下一篇进行讨论。

##3.3订阅

回顾一下第一篇的内容，我们说过RxJava本质上是观察者模式。在观察者模式中存在观察者和被观察者，观察者订阅被观察者，当观察者发生变化时，被观察者进行响应。


RxJava和我们常见的观察者模式有一点不同：由被观察者去*订阅*观察者，而不是传统的观察者去订阅被观察者。当`Observable`调用`subscribe`方法时，其入参为`Observer`对象，即观察者。在`Observer`中有三个方法，分别为`onNext`，`onError`，`onComplete`。 

观察者调用`onNext`对每一项流过的数据进行处理，如果处理失败则调用`onError`，当所有数据都处理完成且没有错误时，应当调用`onComplete`方法。具体过程我们在下一篇进行介绍。

# 4.总结

事实上这一篇我们依然没有对RxJava有什么实质上的介绍，更像是对第一篇的补充。但是这一切我认为是值得的，因为很多人包括我在学习RxJava的初期都是直接扑向写法和API，由于对RP的理解不够充分因此人为造成了较为陡峭的学习曲线。

在下一篇我们会详细介绍创建和订阅以及其源码细节。

