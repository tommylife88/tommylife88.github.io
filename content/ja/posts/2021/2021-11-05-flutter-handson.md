---
title: Flutterハンズオン
date: 2021-11-05
draft: false
tags:
- flutter
- Dart
categories:
- flutter
---

`Flutter`ハンズオン。ちょっと触れる機会があったので備忘録として。

Googleによって開発されたOSSなUIのSDK。単一のコードベースからクロスプラットフォームアプリケーションを開発するために利用できる。

公式のドキュメントが充実していて、そこから理解していくのが間違いない。

## get started

https://docs.flutter.dev/get-started

## Tutorials

https://docs.flutter.dev/get-started/codelab

https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2#0

https://codelabs.developers.google.com/codelabs/flutter#0

## Samples

https://flutter.github.io/samples/#

## Youtube

https://www.youtube.com/channel/UCwXdFgeE9KYzlDdR7TG9cMw

### Flutter Widget of the Weak

https://www.youtube.com/playlist?list=PLjxrf2q8roU23XGwz3Km7sQZFTdB996iG

### Flutter Package of the Weak

https://www.youtube.com/playlist?list=PLjxrf2q8roU1quF6ny8oFHJ2gBdrYN_AK

## VSCode

エディタはVSCode一択か。

### Extensions

https://marketplace.visualstudio.com/items?itemName=marcelovelasquez.flutter-tree

https://marketplace.visualstudio.com/items?itemName=alexisvt.flutter-snippets

https://marketplace.visualstudio.com/items?itemName=Nash.awesome-flutter-snippets

https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens

https://marketplace.visualstudio.com/items?itemName=circlecodesolution.ccs-flutter-color

### Settings

https://dartcode.org/docs/recommended-settings/

```json:.vscode/settings.json
{
    // Causes the debug view to automatically appear when a breakpoint is hit. This
    // setting is global and not configurable per-language.
    "debug.openDebug": "openOnDebugBreak",

    "[dart]": {
        // Automatically format code on save and during typing of certain characters
        // (like `;` and `}`).
        "editor.formatOnSave": true,
        "editor.formatOnType": true,

        // Draw a guide line at 80 characters, where Dart's formatting will wrap code.
        "editor.rulers": [80],

        // Disables built-in highlighting of words that match your selection. Without
        // this, all instances of the selected text will be highlighted, interfering
        // with Dart's ability to highlight only exact references to the selected variable.
        "editor.selectionHighlight": false,

        // By default, VS Code prevents code completion from popping open when in
        // "snippet mode" (editing placeholders in inserted code). Setting this option
        // to `false` stops that and allows completion to open as normal, as if you
        // weren't in a snippet placeholder.
        "editor.suggest.snippetsPreventQuickSuggestions": false,

        // By default, VS Code will pre-select the most recently used item from code
        // completion. This is usually not the most relevant item.
        //
        // "first" will always select top item
        // "recentlyUsedByPrefix" will filter the recently used items based on the
        //     text immediately preceding where completion was invoked.
        "editor.suggestSelection": "first",

        // Allows pressing <TAB> to complete snippets such as `for` even when the
        // completion list is not visible.
        "editor.tabCompletion": "onlySnippets",

        // By default, VS Code will populate code completion with words found in the
        // current file when a language service does not provide its own completions.
        // This results in code completion suggesting words when editing comments and
        // strings. This setting will prevent that.
        "editor.wordBasedSuggestions": false,
    }
}
```

追加で・・・  
コード上のガイド表示
```json:.vscode/settings.json
{
    // ガイド表示
    "dart.previewFlutterUiGuides": true,
    // 描画の遅延を防ぐ
    "dart.previewFlutterUiGuidesCustomTracking": true,
    // クロージングラベルは表示する
    "dart.closingLabels": true,
}
```
## Dart

https://dart.dev/

## With Firebase

https://firebase.google.com/docs/flutter/setup?hl=ja&platform=android