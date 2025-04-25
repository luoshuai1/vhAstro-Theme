---
title: flutter开发核心
date: 2021-09-13 14:04:32
categories: Code
tags:
  - flutter
  - dart
  - 移动开发

id: "js-flutter"
recommend: true
---

### 摘要

:::note
  flutter开发核心 Future/stream/bloc
:::

### Future(异步操作)

:::note
  Future有三种状态**未完成、完成带有值、完成带有异常**，使用Future可以简化事件任务。Dart中，可以使用Future对象来表示异步操作的结果，Future返回类型是Future<T>

  有三种方法处理Future的结果：

  then: 处理操作执行结果或者错误并返回一个新的Future
  catchError: 注册一个处理错误的回调
  whenComplete:类似final，无论错误还是正确，Future执行结束后总是被调用
  使用then来回调，场景使用：UI需要接口的数据，一些异步执行函数
:::

```dart
# demo1
main() {
  Future f1 = new Future(() {
    print("我是第一个");
  });
  f1.then((_) => print("f1 then"));
  print("我是main");
}
# print:
# 我是main
# 我是第一个
# f3 then
```

### Future和async/await结合使用
:::note
  使用链式调用的方式把多个future连接在一起，会严重降低代码的可读性。使用async和await关键字实现异步的功能。
  **async和await可以帮助我们像写同步代码一样编写异步代码**
:::

```dart
Future<String> getStr()async{
  var str = HttpRequest.getString('www.fgyong.cn');
  return str;
}
```
:::md
使用http请求地址www.fgyong.cn获取数据，然后返回。如何接收文本呢？

其实很简单，只需要使用await关键字即可，用来注册then回调。
:::


```dart
main(List<String> args) async {
  String string = await getStr();
  print(string);
}
```

:::md
等同于
:::

```dart
main(List<String> args) async {
  getStr().then((value) {
    print(value);
  });
}
```

### stream—连续异步操作

:::note
如果Future表示单个计算的结果，则流是一系列结果。

侦听流以获取有关结果（数据和错误）以及流关闭的通知。还可以在收听流时暂停播放或在流完成之前停止收听。

可以说Future 用于处理单个异步操作，Stream 用来处理连续的异步操作。

Stream 分单订阅流和广播流。

单订阅流在发送完成事件之前只允许设置一个监听器，并且只有在流上设置监听器后才开始产生事件，取消监听器后将停止发送事件。即使取消了第一个监听器，也不允许在单订阅流上设置其他的监听器。广播流则允许设置多个监听器，也可以在取消上一个监听器后再次添加新的监听器。

Stream 有同步流和异步流之分。

它们的区别在于同步流会在执行 add，addError 或 close 方法时立即向流的监听器 StreamSubscription 发送事件，而异步流总是在事件队列中的代码执行完成后在发送事件。

在 Dart 有几种方式创建 Stream

1. 使用 async* 函数，函数标记为 async *，我们可以使用 yield 作为关键字并返回 Stream 数据
2. 从现有的生成一个新的流 Stream，使用 map，where，takeWhile 等方法。
```dart
Stream<int> countStream(int to) async* {
  for (int i = 1; i <= to; i++) {
    yield i;
  }
}
 
Stream stream = countStream(10);
stream.listen(print);

```
3. 使用 StreamController。

```dart
StreamController<Map> _streamController = StreamController(
  onCancel: () {},
  onListen: () {},
  onPause: () {},
  onResume: () {},
  sync: false,
);
 
Stream _stream = _streamController.stream;

```
4. 使用 Future 对象生成

```dart
Future<int> _delay(int seconds) async {
  await Future.delayed(Duration(seconds: seconds));
  return seconds;
}
 
List<Future> futures = [];
for (int i = 0; i < 10; i++) {
  futures.add(_delay(3));
}
 
Stream _futuresStream = Stream.fromFutures(futures);

```
:::

:::md
我们在实际的开发过程中，基本都是使用的StreamContoller来创建流。监听使用StreamBuilder，当流发生变化时执行，例子：
:::

```dart
import 'dart:async';
import 'package:flutter/material.dart';
 
class StreamCounter extends StatefulWidget {
  @override
  _StreamCounterState createState() => _StreamCounterState();
}
 
class _StreamCounterState extends State<StreamCounter> {
  // 创建一个 StreamController
  StreamController<int> _counterStreamController = StreamController<int>(
    onCancel: () {
      print('cancel');
    },
    onListen: () {
      print('listen');
    },
  );
 
  int _counter = 0;
  Stream _counterStream;
  StreamSink _counterSink;
 
  // 使用 StreamSink 向 Stream 发送事件，当 _counter 大于 9 时调用 close 方法关闭流。
  void _incrementCounter() {
    if (_counter > 9) {
      _counterSink.close();
      return;
    }
    _counter++;
    _counterSink.add(_counter);
  }
 
  // 主动关闭流
  void _closeStream() {
    _counterStreamController.close();
  }
 
  @override
  void initState() {
    super.initState();
    _counterSink = _counterStreamController.sink;
    _counterStream = _counterStreamController.stream;
  }
 
  @override
  void dispose() {
    super.dispose();
    _counterSink.close();
    _counterStreamController.close();
  }
 
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Stream Counter'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text('You have pushed the button this many times:'),
            // 使用 StreamBuilder 显示和更新 UI
            StreamBuilder<int>(
              stream: _counterStream,
              initialData: _counter,
              builder: (context, snapshot) {
                if (snapshot.connectionState == ConnectionState.done) {
                  return Text(
                    'Done',
                    style: Theme.of(context).textTheme.bodyText2,
                  );
                }
 
                int number = snapshot.data;
                return Text(
                  '$number',
                  style: Theme.of(context).textTheme.bodyText2,
                );
              },
            ),
          ],
        ),
      ),
      floatingActionButton: Row(
        mainAxisAlignment: MainAxisAlignment.center,
        children: <Widget>[
          FloatingActionButton(
            onPressed: _incrementCounter,
            tooltip: 'Increment',
            child: Icon(Icons.add),
          ),
          SizedBox(width: 24.0),
          FloatingActionButton(
            onPressed: _closeStream,
            tooltip: 'Close',
            child: Icon(Icons.close),
          ),
        ],
      ),
    );
  }
}
```

### bloc—状态管理

:::note
上文了解到stream，是使用bloc的必要知识。bloc的一个基础使用是Cubit，Cubit类似是bloc的简化版，在一些小的项目上可以使用Cubit。项目维护的数据多使用Bloc最好，下面写的是bloc核心知识和使用案例
:::

#### bloc处理关系图：

![50497ffb3f12283ec301d5f10feba77c.png](https://wp-cdn.4ce.cn/v2/wroGSHS.png)

Bloc模式
  > bloc：逻辑层
  > state：数据层
  > event：所有的交互事件
  > view：页面

#### Bloc模板

view：默认添加了一个初始化事件

```dart
class CounterPage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (BuildContext context) => CounterBloc()..add(InitEvent()),
      child: Builder(builder: (context) => _buildPage(context)),
    );
  }
 
  Widget _buildPage(BuildContext context) {
    final bloc = BlocProvider.of<CounterBloc>(context);
 
    return Container();
  }
}
```
bloc

```dart
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState().init());
 
  @override
  Stream<CounterState> mapEventToState(CounterEvent event) async* {
    if (event is InitEvent) {
      yield await init();
    }
  }
 
  Future<CounterState> init() async {
    return state.clone();
  }
}
```
event

```dart
abstract class CounterEvent {}
 
class InitEvent extends CounterEvent {}
```
state

```dart
class CounterState {
  CounterState init() {
    return CounterState();
  }
 
  CounterState clone() {
    return CounterState();
  }
}
```