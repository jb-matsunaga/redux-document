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

## Action Creators
アクションクリエイターは、アクションを作成する機能です。 「アクション」と「アクションクリエイター」の用語を簡単に組み合わせることができますので、適切な用語を使用するようにしてください。

Reduxアクション作成者は単にアクションを返します：

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

これにより、ポータブルで簡単にテストすることができます。

従来のFluxでは、アクションクリエイターは、呼び出されたときにディスパッチをトリガーすることがよくありました。

```javascript
function addTodoWithDispatch(text) {
  const action = {
    type: ADD_TODO,
    text
  }
  dispatch(action)
}
```

Reduxでは、これは当てはまりません。

代わりに、実際にディスパッチを開始するには、結果をdispatch（）関数に渡します。

```javascript
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

あるいは、自動的にディスパッチするバウンドアクションクリエータを作成することもできます。

```javascript
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))
```

今すぐあなたはそれらを直接呼び出すことができます：

```javascript
boundAddTodo(text)
boundCompleteTodo(index)
```

dispatch（）関数はstore.dispatch（）としてストアから直接アクセスできますが、react-reduxのconnect（）などのヘルパーを使用してアクセスする可能性が高くなります。 bindActionCreators（）を使用すると、多くのアクション作成者を自動的にdispatch（）関数にバインドできます。

アクションクリエイターは非同期で、副作用もあります。 上級チュートリアルでは、非同期アクションについて読むことができ、AJAXレスポンスを処理し、アクションクリエータを非同期コントロールフローに組み立てる方法を学ぶことができます。 アドバンスチュートリアルと非同期アクションの前提条件であるその他の重要な概念については、基本チュートリアルを完了するまで非同期アクションに進む必要はありません。

## ソースコード

```javascript
/**
 * action types
 */

export const ADD_TODO = 'ADD_TODO';
export const TOGGLE_TODO = 'TOGGLE_TODO';
export const SET_VISIBILITY_FILTER = 'SET_VISIBILITY_FILTER';

/**
 * other constants
 */

export const VisibilityFilters = {
    SHOW_ALL: 'SHOW_ALL',
    SHOW_COMPLETED: 'SHOW_COMPLETED',
    SHOW_ACTIVE: 'SHOW_ACTIVE'
}

/**
 * action creators
 */

export function addTodo(text) {
    return { type: ADD_TODO, text }
}

export function TOGGLE_TODO(index) {
    return { type: TOGGLE_TODO, index }
}

export function SET_VISIBILITY_FILTER(filter) {
    return { type: SET_VISIBILITY_FILTER, filter }
}
```
## NEXT STEP
次に、これらのアクションをディスパッチしたときに状態がどのように更新されるかを指定するリデューサをいくつか定義しましょう！