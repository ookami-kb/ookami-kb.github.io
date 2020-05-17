---
layout: post
title:  "Flutter: how to draw text along an arc"
date:   2019-12-12 11:51:25 +0200
categories: flutter tutorial
---

For one of our side projects (highly experimental and written in Flutter for Web, by the way) I needed to implement something like this:

![](https://developers.mews.com/content/images/2019/12/Screenshot_2019-12-01_at_21.32.52.png)

The code is actually the same for mobile and web, so, for the sake of simplicity (and because they look nicer), I'll share screenshots from the mobile app.

The problem is that Flutter doesn't support drawing text along a custom path (and it [doesn't look like it will](https://github.com/flutter/flutter/issues/16477), at least not in the near future). So I decided to implement the functionality on my own. Drawing text along any custom path would be quite complex, but luckily I only had to implement text along an arc. I'd like to share with you one way to do that.

For this implementation, your first geometry course should be more than enough (though, shame on me, I've realised that I've forgot almost all of mine). It won't be the best possible implementation, but for our use case it will be good enough.

Let's start with defining the interface of our widget (let it be `ArcText`):

```dart
class ArcText extends StatelessWidget {
  const ArcText({
    Key key,
    @required this.radius,
    @required this.text,
    @required this.textStyle,
    this.startAngle = 0,
  }) : super(key: key);

  final double radius;
  final String text;
  final double startAngle;
  final TextStyle textStyle;
}
```

`startAngle` is the initial angle where the text will start, and `radius` is the arc's radius – the text will be drawn along its side. We can use it like this:

```dart
Container(
  decoration: BoxDecoration(
    border: Border.all(),
    color: Colors.white,
  ),
  width: 300,
  height: 300,
  child: ArcText(
    radius: 100,
    text: 'Hello, Mews! I am ArcText widget. I can draw circular text.',
    textStyle: TextStyle(fontSize: 18, color: Colors.black),
    startAngle: -pi / 2,
  ),
)
```

For the text rendering we'll be using `CustomPainter`. This is where the magic happens:

```dart
class ArcText extends StatelessWidget {
  const ArcText({
    Key key,
    @required this.radius,
    @required this.text,
    @required this.textStyle,
    this.startAngle = 0,
  }) : super(key: key);

  final double radius;
  final String text;
  final double startAngle;
  final TextStyle textStyle;

  @override
  Widget build(BuildContext context) => CustomPaint(
        painter: _Painter(
          radius,
          text,
          textStyle,
          initialAngle: startAngle,
        ),
      );
}
```

Before starting to implement the `_Painter` class let's think about how it will work:


![](https://developers.mews.com/content/images/2019/12/Group.png)


The idea here is that we will draw each letter on top of a chord. The radius of the circle the cord belongs to is our defined radius. `d` – the chord's length – equals the letter's width. That means that after drawing each letter we can rotate the canvas to a specified angle and move it to a distance `d` along the x axis. It's easier to transform the canvas than to calculate new coordinates.

Let's start implementing the `_Painter` class:

```dart
class _Painter extends CustomPainter {
  _Painter(this.radius, this.text, this.textStyle, {this.initialAngle = 0});

  final num radius;
  final String text;
  final double initialAngle;
  final TextStyle textStyle;

  @override
  void paint(Canvas canvas, Size size) {}

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}
```

For rendering we will need: the radius, the text itself, the text style (so that we can get a width of each letter) and the initial angle. The `shouldRepaint` method defines whether the `paint` method needs to be called and, in the simplest case (when we don't have any complex calculations there), it can always return `true`.

Now we can continue implementing `paint`:

```dart
@override
void paint(Canvas canvas, Size size) {
  canvas.translate(size.width / 2, size.height / 2);
  canvas.drawCircle(Offset.zero, radius, Paint()..style=PaintingStyle.stroke);
  canvas.translate(0, -radius);
}
```

In this code snippet we're moving the canvas so that circle radius is in the center of a container, drawing the circle (it's just a helper, we'll remove it later), and moving the canvas again to a `-radius` along the y axis – later it will be easy to move and rotate it. We should get the following picture:

![](https://developers.mews.com/content/images/2019/12/Screenshot_2019-12-01_at_18.51.06.png)

Let's add text rendering:

```dart
@override
void paint(Canvas canvas, Size size) {
  canvas.translate(size.width / 2, size.height / 2);
  canvas.drawCircle(Offset.zero, radius, Paint()..style=PaintingStyle.stroke);
  canvas.translate(0, -radius);

  double angle = 0;
  for (int i = 0; i < text.length; i++) {
    angle = _drawLetter(canvas, text[i], angle);
  }
}

double _drawLetter(Canvas canvas, String letter, double prevAngle) {
  _textPainter.text = TextSpan(text: letter, style: textStyle);
  _textPainter.layout(
    minWidth: 0,
    maxWidth: double.maxFinite,
  );

  final double d = _textPainter.width;
  final double alpha = 2 * math.asin(d / (2 * radius));

  final newAngle = _calculateRotationAngle(prevAngle, alpha);
  canvas.rotate(newAngle);

  _textPainter.paint(canvas, Offset(0, -_textPainter.height));
  canvas.translate(d, 0);

  return alpha;
}

double _calculateRotationAngle(double prevAngle, double alpha) =>
    (alpha + prevAngle) / 2;
```

As I said, the idea is rather simple: with each letter drawn we're rotating the canvas to a calculated angle, so that the chord of the current letter is parallel to the x axis; then we draw the letter and move the canvas along the x axis to a distance equal to the letter width.

We can use our knowledge of geometry to find a chord's angle and the new angle to rotate the canvas.

A chord's length can be found from the following equation:

$$d=2r\sin {\frac {\alpha }{2}}$$

So it can be transformed like this to find the chord's central angle:

$$\alpha=2\arcsin({\frac d {2r}})$$

The angle to rotate the canvas can be found with this simple formula:

$$\Delta=(\frac{\alpha+\beta} 2)$$

`⍺` is the central angle of the previous chord, `β` is the central angle of the current chord.

Now it looks like this:


![](https://developers.mews.com/content/images/2019/12/Screenshot_2019-12-01_at_22.07.00.png)

Now we can remove the helper circle and take the initial angle into account:

```dart
@override
void paint(Canvas canvas, Size size) {
  canvas.translate(size.width / 2, size.height / 2 - radius);

  if (initialAngle != 0) {
    final d = 2 * radius * math.sin(initialAngle / 2);
    final rotationAngle = _calculateRotationAngle(0, initialAngle);
    canvas.rotate(rotationAngle);
    canvas.translate(d, 0);
  }

  double angle = initialAngle;
  for (int i = 0; i < text.length; i++) {
    angle = _drawLetter(canvas, text[i], angle);
  }
}
```

![](https://developers.mews.com/content/images/2019/12/Screenshot_2019-12-01_at_21.32.52.png)

That's all! Here's the final source code:

```dart
import 'dart:math' as math;

import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

class ArcText extends StatelessWidget {
  const ArcText({
    Key key,
    @required this.radius,
    @required this.text,
    @required this.textStyle,
    this.startAngle = 0,
  }) : super(key: key);

  final double radius;
  final String text;
  final double startAngle;
  final TextStyle textStyle;

  @override
  Widget build(BuildContext context) => CustomPaint(
        painter: _Painter(
          radius,
          text,
          textStyle,
          initialAngle: startAngle,
        ),
      );
}

class _Painter extends CustomPainter {
  _Painter(this.radius, this.text, this.textStyle, {this.initialAngle = 0});

  final num radius;
  final String text;
  final double initialAngle;
  final TextStyle textStyle;

  final _textPainter = TextPainter(textDirection: TextDirection.ltr);

  @override
  void paint(Canvas canvas, Size size) {
    canvas.translate(size.width / 2, size.height / 2 - radius);

    if (initialAngle != 0) {
      final d = 2 * radius * math.sin(initialAngle / 2);
      final rotationAngle = _calculateRotationAngle(0, initialAngle);
      canvas.rotate(rotationAngle);
      canvas.translate(d, 0);
    }

    double angle = initialAngle;
    for (int i = 0; i < text.length; i++) {
      angle = _drawLetter(canvas, text[i], angle);
    }
  }

  double _drawLetter(Canvas canvas, String letter, double prevAngle) {
    _textPainter.text = TextSpan(text: letter, style: textStyle);
    _textPainter.layout(
      minWidth: 0,
      maxWidth: double.maxFinite,
    );

    final double d = _textPainter.width;
    final double alpha = 2 * math.asin(d / (2 * radius));

    final newAngle = _calculateRotationAngle(prevAngle, alpha);
    canvas.rotate(newAngle);

    _textPainter.paint(canvas, Offset(0, -_textPainter.height));
    canvas.translate(d, 0);

    return alpha;
  }

  double _calculateRotationAngle(double prevAngle, double alpha) =>
      (alpha + prevAngle) / 2;

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => true;
}
```
    
By the way, it's also available as a Flutter package [here](https://pub.dev/packages/flutter_arc_text/).

> Originally published at [developers.mews.com](https://developers.mews.com/flutter-how-to-draw-text-along-arc/)
