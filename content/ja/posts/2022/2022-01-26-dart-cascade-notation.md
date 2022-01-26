---
title: Dartのカスケード記法（../?..）
date: 2022-01-26
draft: false
tags:
- Dart
categories:
- Dart
---

Dartの`..`や`?..`って何？ってよく聞かれるので。

[【公式】Cascade notation（カスケード記法）](https://dart.dev/guides/language/language-tour#cascade-notation)って呼ばれるもの。

## カスケード記法

機械翻訳すると、Dartのカスケード記法は以下のように説明されている。

> 同じオブジェクトに対して一連の操作を行うことができます。関数呼び出しに加えて、同じオブジェクトのフィールドにアクセスすることもできます。これにより、一時変数を作成する手間が省け、より流動的なコードを記述できるようになります。

例えば、
```dart
var paint = Paint();
paint.color = Colors.black;
paint.strokeCap = StrokeCap.round;
paint.strokeWidth = 5.0;
```
をカスケードで書き直すと、
```dart
var paint = Paint()
  ..color = Colors.black
  ..strokeCap = StrokeCap.round
  ..strokeWidth = 5.0;
```
とかける。  
直前に作った`paint`オブジェクトに対する関数呼び出しや、フィールドへのアクセスを短く記述できる。  
カスケード記法に続くコードは、返されるかもしれない値を無視して、このオブジェクトを操作する。

## null-shortingカスケード

カスケードが動作するオブジェクトがnullになる可能性がある場合は、`?..`で記述できる。

{{< notice info "対応バージョン" >}} 
2.12以降
{{< /notice >}}

```dart
var button = querySelector('#confirm');
button?.text = 'Confirm';
button?.classes.add('important');
button?.onClick.listen((e) => window.alert('Confirmed!'));
```
は、
```dart
querySelector('#confirm') // Get an object.
  ?..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'))
```
と書ける。

## 注意

戻り値`void`の関数にカスケードするとエラーになる。

```dart
var sb = StringBuffer();
sb.write('foo')
  ..write('bar'); // Error: method 'write' isn't defined for 'void'.
```

この場合、`sb.write`の戻り値が`void`なのでカスケードできない。理屈が分かればなんのこっちゃない。