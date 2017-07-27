# Basics
レデューサー、ミドルウェア、ストアエンハンサーについてのすばらしい話に全然騙されないでください.Reduxは信じられないほど簡単です。 Fluxアプリケーションを作成したことがあれば、自宅にいるような気分になります。 Fluxを初めてお使いの方も簡単です！

このガイドでは、簡単なTodoアプリケーションを作成するプロセスについて説明します。

## Actions
まず、いくつかのアクションを定義しましょう。

アクションとは、アプリケーションからストアにデータを送信する情報のペイロードです。 それらは店舗の唯一の情報源です。 store.dispatch（）を使用してストアに送信します。

新しいToDo項目の追加を表すアクションの例を次に示します。

```javascript
const ADD_TODO = 'ADD_TODO'
{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

アクションはプレーンなJavaScriptオブジェクトです。 アクションには、実行されるアクションのタイプを示すtypeプロパティが必要です。 型は通常、文字列定数として定義する必要があります。 あなたのアプリが十分な大きさになると、別のモジュールにそれらを移動することができます。

```javascript
import { ADD_TODO, REMOVE_TODO } from '../actionTypes'
```

### ボイラープレートについての注意
アクションタイプの定数を別々のファイルに定義する必要はなく、まったく定義する必要もありません。 小規模なプロジェクトでは、アクションタイプに文字列リテラルを使用するほうが簡単かもしれません。 しかし、より大きなコードベースでは定数を明示的に宣言することにはいくつかの利点があります。 あなたのコードベースをきれいに保つためのより実用的なヒントについては、Reducing Boilerplateを読んでください。

[Reducing Boilerplate](http://redux.js.org/docs/recipes/ReducingBoilerplate.html)

タイプ以外のアクションオブジェクトの構造は、あなた次第です。 興味がある場合は、Flux Standard Actionを参照して、アクションの構築方法に関する推奨事項を確認してください。

[flux-standard-action](https://github.com/acdlite/flux-standard-action)

私たちはtodoをオフにしているユーザーを完了したとして説明するもう1つのアクションタイプを追加します。 特定のtodoは、配列に格納するため、インデックスを使用して参照します。 実際のアプリケーションでは、新しいものが作成されるたびに一意のIDを生成する方が賢明です。

```javascript
{
  type: TOGGLE_TODO,
  index: 5
}
```

可能な限り各アクションにはほとんどデータを渡さないことをお勧めします。 たとえば、todoオブジェクト全体よりもインデックスを渡すほうがよいでしょう。

最後に、現在表示されているtodosを変更するためのもう1つのアクションタイプを追加します。

```javascript
{
  type: SET_VISIBILITY_FILTER,
  filter: SHOW_COMPLETED
}
```

