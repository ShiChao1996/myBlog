# Tutorial: Animations in Flitter

将要了解：
* 如何在组件中使用 animation library 的基础类
* 如何链化 Tween Animation
* 何时使用 animatedWidget 和 animatedBuilder

## 必要的概念和类

* Animation 类，动画的核心类，‘插值来引导动画’
* animation 对象只关心当前动画**状态**，不关心屏幕上有什么
* AnimationController 管理动画
* CurvedAnimation 定义一个非线性动画
* Tween 在一个给定的范围内插值，比如定义颜色从红色到蓝色
* Listener、StatusListener 用来监控动画状态改变

#### Animation<double>
> Animation 类是一个抽象类，它有当前动画的值和状态。最常用的是 Animation<double>，
它通常在一个确定的范围生成插值数。Animation 的输出可以是一个线性的、非线性的、分步的、函数。

#### CurvedAnimation 
```dart
final CurvedAnimation curve =
  new CurvedAnimation(parent: controller, curve: Curves.easeIn);
// Curves 对象包含了一些常用的曲线
```

#### AnimationController
> AnimationController 也是一个特殊的 animation 对象。它默认生成 0.0 ~ 1.0 之间的线性的数值
```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 2000), vsync: this);
```
> AnimationController 在Animation<double> 基础上添加了控制动画的函数。
你可以使用 `forward()` 开始动画，生成的数值会绑定到屏幕上刷新。

> 在创建 AnimationController 时传入了一个 `vsync`， 它的存在阻止了不在屏幕显示区上的动画浪费不必要的资源


#### Tween
> AnimationController 生成 0.0 ~ 1.0 范围的数，Tween 可以生产任意范围的数。
```dart
final Tween doubleTween = new Tween<double>(begin: -200.0, end: 0.0);
```
tween 唯一的作用是生产包含一个范围内的数的 `map`。它不保存任何状态，`provide()`提供值， `value`是当前值。

###### Tween.animate()
> tween.animate() 返回一个 `Animation`
```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
Animation<int> alpha = new IntTween(begin: 0, end: 255).animate(controller);
```
```dart
final AnimationController controller = new AnimationController(
    duration: const Duration(milliseconds: 500), vsync: this);
final Animation curve =
    new CurvedAnimation(parent: animation, curve: Curves.easeOut);
Animation<int> alpha = new IntTween(begin: 0, end: 255).animate(curve);
```

#### Animation notifications
> 一个 Animation 对象可以有 Listeners 和 StatusListeners，
用于监听动画状态


## Animation examples

```dart
import 'package:flutter/animation.dart';
import 'package:flutter/material.dart';

class LogoApp extends StatefulWidget {
  _LogoAppState createState() => new _LogoAppState();
}

class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
  Animation<double> animation;
  AnimationController controller;

  initState() {
    super.initState();
    controller = new AnimationController(
        duration: const Duration(milliseconds: 2000), vsync: this);
    animation = new Tween(begin: 0.0, end: 300.0).animate(controller)
      ..addListener(() {
        setState(() {
          // the state that has changed here is the animation object’s value
        });
      });
    controller.forward();
  }

  Widget build(BuildContext context) {
    return new Center(
      child: new Container(
        margin: new EdgeInsets.symmetric(vertical: 10.0),
        height: animation.value,
        width: animation.value,
        child: new FlutterLogo(),
      ),
    );
  }

  dispose() {
    controller.dispose();
    super.dispose();
  }
}

void main() {
  runApp(new LogoApp());
}
```

####