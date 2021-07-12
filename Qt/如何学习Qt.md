# 如何学习Qt

## 理解Qt的主要机制

### Events and the main event loop

* Qt的main函数
    ```c++
  int main(int argc, char* argv[]) {
   QCoreApplication app(argc, argv);
   // ... instansiate resources using app ...
   return app.exec();
  }
  ```

* Qt's **main event loop** is entered by running the `exec()` member function on an instance of a [QCoreApplication](http://doc.qt.io/qt-5/qcoreapplication.html), a [QGuiApplication](http://doc.qt.io/qt-5/qguiapplication.html) or a [QApplication](http://doc.qt.io/qt-5/qapplication.html) : Qt 的主事件循环是通过在 QCoreApplication、QGuiApplication 或 QApplication 的实例上运行 exec() 成员函数来进入的。

    ```c++
    while (true) {
      dispatchEventsFromQueue();
      waitForEvents();
    }
    ```

    *  The termination of the loop will be performed when `exit()` or `quit()` is called on the `app` object. 当在 app 对象上调用 exit() 或 quit() 时，将执行循环的终止。
    * 避免在UI消息中做耗时的工作



### Meta-object system

* The macro `Q_OBJECT` is what the MOC is looking for in order to generate the moc_-file.![image-20210707205203428](C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210707205203428.png)

```cpp
#include <QObject>

class MyClass : public QObject {
  Q_OBJECT
  Q_DISABLE_COPY(MyClass)
  // ... properties ...

public:
  MyClass();
  // ... functions and member variables ...

};
```

* The generated moc_-files implement functions which are used for several different features. The perhaps most important ones are **the *signals and slots mechanism***, **the *run-time type information (RTTI)*, and the *dynamic property system*.** 最重要的是信号和槽机制、运行时类型信息 (RTTI) 和动态属性系统。

* RTTI 举例 1:

  ```c++
  MyClass myclass;
  qDebug() << myclass.metaObject()->className();
  ```

*  RTTI 举例 2: The [qobject_cast](https://doc.qt.io/qt-5/qobject.html#qobject_cast)() function behaves similarly to the standard C++ `dynamic_cast()`, with the advantages that it doesn't require RTTI support and it works across dynamic library boundaries qobject_cast() 函数的行为类似于标准的 C++ dynamic_cast()，优点是它不需要 RTTI 支持并且可以跨动态库边界工作.

  ```c++
  QObject *obj = new QTimer;          // QTimer inherits QObject
  
  QTimer *timer = qobject_cast<QTimer *>(obj);
  // timer == (QObject *)obj
  
  QAbstractButton *button = qobject_cast<QAbstractButton *>(obj);
  // button == nullptr
  ```

* QObject enables even more features

  * `QObjects` enable translations of strings for internationalisation using [tr()](http://doc.qt.io/qt-5/qobject.html#tr).
  * All `QObjects` can receive and filter `QEvents`.

### Signals and slots

* Signals 
  In order to enable signals for a class, it has to inherit from `QObject` and use the `Q_OBJECT` macro

```c++
class MySender : public QObject {
  Q_OBJECT
  ...

signals:
  void signalName(const QString& use, int different, float types);
};
```

*  Slots
   Similarly to the signal object, the receiver class needs to inherit from `QObject` and also use the `Q_OBJECT` macro

```c++
class MyReceiver : public QObject {
  Q_OBJECT
  ...

public slots:
  void slotName(const QString& use, int different, float types) {
    qDebug() << use << " " << different << " " << types;
  }
};
```



* Connect

  You may connect many different signals to the same slot, or use the same signal for many different slots. It's your choice - the system is very flexible.
  We'll use the static function [QObject::connect()](http://doc.qt.io/qt-5/qobject.html#connect):

  ```c++
  MySender sender;
  MyReciever receiver;
  
  connect(&sender, &MySender::signalName, &receiver, &MyReciever::slotName);
  ```

* Emit

  ```c++
  void MySender::functionToEmitSignal() {
    emit signalName("Important data that will be sent to all listeners", 42, 1.618033);
  }
  ```

  

* lambdas 
  You may connect many different signals to the same slot, or use the same signal for many different slots. It's your choice - the system is very flexible. As mentioned, the system even allows to connect functions and lambdas directly, without using a listener, e.g:

  ```c++
  connect(&sender, &MySender::signalName, [](const QString& use, int different, float types) {
    ...
  }));
  ```



* disconnect
  The connection is automatically disconnected if either the sender or the receiver becomes deleted. However, as you might have guessed it's also possible to manually disconnect it by using [disconnect](http://doc.qt.io/qt-5/qobject.html#disconnect-5):

### Hierarchy and memory management

* QObject 定义

![image-20210711234731413](C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210711234731413.png)

* the hierarchy tree might become very big and it will be difficult to have a good understanding of its full length. By calling [QObject::dumpObjectTree()](http://doc.qt.io/qt-5/qobject.html#dumpObjectTree) it's possible to output the current hierarchy .
* Other effective functions when navigating the tree are the following: [QObject::children()](http://doc.qt.io/qt-5/qobject.html#children), [QObject::findChild()](http://doc.qt.io/qt-5/qobject.html#findChild), [QObject::findChildren()](http://doc.qt.io/qt-5/qobject.html#findChildren) and [QObject::parent()](http://doc.qt.io/qt-5/qobject.html#parent).



### 布局需要考虑的几个因素

* layout和QWidget的关系。请看如下两个函数

![image-20210712131432915](C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210712131432915.png)

![image-20210712131342950](C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210712131342950.png)

 * eg：<img src="C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210712125124946.png" alt="image-20210712125124946" style="zoom:67%;" />

* QSizePolicy
  * 

### 如何掌握技术

* 不要追求体系，不必追求过度深入。技术是达成目标的工具。
  * Where are we?（我们现在在哪？）
  * Where are we going?（我们要到哪儿去？）
  * How can we get there?（我们如何到达那里？）
  * eg：提问的时候，留意第一，二点。
* 时刻对学习过的知识进行总结，复习，可以通过回忆，对比，联想，自问自答的方式来检查自己是否掌握。
  * eg：python c++的函数 和verilog的模块对比记忆
* 平时积累基础知识，如英语，数学，c++（对于QT）
  * 



### 如何掌握QT

* 随时查阅官方手册。
* 查阅官方提供的examples
* 掌握**如何学习**的方法



https://m.toutiaocdn.com/i6639872529924096519/?app=news_article_lite&timestamp=1625621400&use_new_style=1&req_id=202107070929590102120810685F1349DA&group_id=6639872529924096519&share_token=208654ea-4bf9-4287-823f-ab6a6b2743c2

<img src="C:\Users\yh\AppData\Roaming\Typora\typora-user-images\image-20210712140202506.png" alt="image-20210712140202506" style="zoom:50%;" />
