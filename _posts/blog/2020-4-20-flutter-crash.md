---
layout: post
title: flutter Crash上报方案
categories: Blog
description:  上报flutter crash
keywords:  flutter,crash

---


## 一、flutter 崩溃收集的方式

## 1、通用方式

use a try/catch block

## 2、捕捉async异常

**1）try/catch**

```dart
Future main() async {
  var dir = new Directory('/tmp');
 
  try {
    var dirList = dir.list();
    await for (FileSystemEntity f in dirList) {
      if (f is File) {
        print('Found file ${f.path}');
      } else if (f is Directory) {
        print('Found dir ${f.path}');
      }
    }
  } catch (e) {
    print(e.toString());
  }
}
```



**2）使用 Future API**

```dart
myFunc()
  .then((value) {
    doSomethingWith(value);
    ...
    throw("some arbitrary error");
  })
  .catchError(handleError);
```



**3）async异常 与 Future 的更多信息**

- [Futures and Error Handling](https://link.zhihu.com/?target=https%3A//www.dartlang.org/guides/libraries/futures-error-handling)
- [Asynchronous Programming: Futures](https://link.zhihu.com/?target=https%3A//www.dartlang.org/tutorials/language/futures)
- [dart:async library](https://link.zhihu.com/?target=https%3A//api.dartlang.org/stable/2.1.0/dart-async/dart-async-library.html)



## 3、使用 runZoned

```dart
// This creates a [Zone] that contains the Flutter application and stablishes
// an error handler that captures errors and reports them.
//
// Using a zone makes sure that as many errors as possible are captured,
// including those thrown from [Timer]s, microtasks, I/O, and those forwarded
// from the `FlutterError` handler.
//
// More about zones:
//
// - https://api.dartlang.org/stable/1.24.2/dart-async/Zone-class.html
// - Zones
runZoned<Future<Null>>(() async {
  runApp(new CrashyApp());
}, onError-: (error, stackTrace) async {
  await _reportError(error, stackTrace);
});
```

## 4、使用 FlutterError.onError

FlutterError.onError is only about reporting flutter framework errors (errors that are caught by the framework typically during a frame).

```dart
// This captures errors reported by the Flutter framework.
FlutterError.onError = (FlutterErrorDetails details) async {
  if (isInDebugMode) {
    // In development mode simply print to console.
    FlutterError.dumpErrorToConsole(details);
  } else {
    // In production mode report to the application zone
    // 重定向到 3中的 runZone 中处理
    Zone.current.handleUncaughtError(details.exception, details.stack);
  }
};
```

## 5、使用 Isolate.current.addErrorListener

use Isolate.current.addErrorListener to capture uncaught errors in the root zone.

```dart
Isolate.current.addErrorListener(new RawReceivePort((dynamic pair) async {
  print('Isolate.current.addErrorListener caught an error');
  await _reportError(
    (pair as List<String>).first,
    (pair as List<String>).last,
  );
}).sendPort);
```



## 二、Flutter异常收集最佳实践

```dart
Future<Null> main() async {
  FlutterError.onError = (FlutterErrorDetails details) async {
    if (isInDebugMode) {
      FlutterError.dumpErrorToConsole(details);
    } else {
      Zone.current.handleUncaughtError(details.exception, details.stack);
    }
  };
 
  runZoned<Future<Null>>(() async {
    runApp(new HomeApp());
  }, onError-: (error, stackTrace) async {
    await _reportError(error, stackTrace);
  });
}
```



## 三、Flutter crash 收集平台

## 1、Sentry

1）商业Sentry服务器

[Sentry | Error Tracking Software — JavaScript, Python, PHP, Ruby, more](https://link.zhihu.com/?target=https%3A//sentry.io/)

Sentry官方的服务是收费的，使用天数只有13天左右，过期后不付费的话：只保存1w个事件，没有成员功能。



2）flutter官方支持

Sentry flutter package：[https://pub.dartlang.org/packages/sentry](https://link.zhihu.com/?target=https%3A//pub.dartlang.org/packages/sentry)

github: [flutter](https://link.zhihu.com/?target=https%3A//github.com/flutter)/[sentry](https://link.zhihu.com/?target=https%3A//github.com/flutter/sentry)

demo: [flutter](https://link.zhihu.com/?target=https%3A//github.com/flutter)/[crashy](https://link.zhihu.com/?target=https%3A//github.com/flutter/crashy) 、 [yjbanov](https://link.zhihu.com/?target=https%3A//github.com/yjbanov)/[crashy](https://link.zhihu.com/?target=https%3A//github.com/yjbanov/crashy)



3）总结

优点：有了 flutter package ，flutter 接入 Sentry，非常简单。

缺点：免费版限制多。



## 2、Crashlytics （--> fabric --> Firebase）

1）flutter 官方支持的计划

[Please add support for Crashlytics](https://link.zhihu.com/?target=https%3A//github.com/flutter/flutter/issues/14765)



2）非官方flutter插件

[https://github.com/kiwi-bop/flutter_crashlytics](https://link.zhihu.com/?target=https%3A//github.com/kiwi-bop/flutter_crashlytics)

[https://pub.dartlang.org/packages/flutter_crashlytics](https://link.zhihu.com/?target=https%3A//pub.dartlang.org/packages/flutter_crashlytics)



3）总结

优点：Google旗下，免费！

缺点：目前flutter官方尚未有对应package/plugin，使用 [flutter_crashlytics](https://link.zhihu.com/?target=https%3A//github.com/kiwi-bop/flutter_crashlytics) 对接比较麻烦。




