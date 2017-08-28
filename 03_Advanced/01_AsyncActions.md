# Async Actions
基本ガイドでは、簡単なToDoアプリケーションを作成しました。それは完全に同期していた。アクションがディスパッチされるたびに、状態は直ちに更新されました。

このガイドでは、異なる非同期アプリケーションを構築します。 Reddit APIを使用して、選択したサブディレクトリの現在の見出しを表示します。非同期性はどのようにReduxの流れに適合しますか？

## Actions
非同期APIを呼び出すと、呼び出しを開始した瞬間と応答（またはタイムアウト）を受け取った瞬間の2つの重要な瞬間があります。

これらの2つの瞬間は、通常、アプリケーションの状態を変更する必要があります。これを行うには、減速機で同時に処理される通常のアクションをディスパッチする必要があります。通常、どのAPIリクエストでも少なくとも3種類のアクションをディスパッチする必要があります：

- 要求が開始されたことをReducerに通知するアクション。
  - Reducerは、状態でisFetchingフラグをトグルすることによってこのアクションを処理することができます。このようにして、UIはスピナーを表示する時間を知っています。

- 要求が正常に終了したことを減速機に通知するアクション。
 - Reducerは、新しいデータを管理したり、リセットしたりする状態にマージすることによって、このアクションを処理することができます。 UIはスピナーを隠し、取得したデータを表示します。

 - 要求が失敗したことを減速機に通知するアクション。
  - ReducerはisFetchingをリセットすることでこのアクションを処理できます。さらに、UIが表示できるようにエラーメッセージを格納したい場合があります。

  自分のactionに専用のステータスフィールドを使用することができます：

```javascript
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

あるいは、別々の型を定義することもできます：

```javascript
{ type: 'FETCH_POSTS_REQUEST' }
{ type: 'FETCH_POSTS_FAILURE', error: 'Oops' }
{ type: 'FETCH_POSTS_SUCCESS', response: { ... } }
```

フラグ付きの単一のアクションタイプを使用するか複数のアクションタイプを使用するかは、あなた次第です。あなたのチームで決定する必要があるしきたりです。複数のタイプには間違いの余地が少なくなりますが、あなたはredux-actionのようなヘルパーライブラリを使ってアクションクリエイターとレデューサーを生成しても問題ありません。

どのようなしきたりを選んでも、アプリケーション全体にこだわることができます。 このチュートリアルでは、別々のタイプを使用します。

## Synchronous Action Creators
まず、サンプルアプリケーションで必要ないくつかの同期アクションタイプとアクションクリエータを定義してみましょう。ここで、ユーザーは表示するサブレディを選択できます。

*actions.js*
```javascript
export const SELECT_SUBREDDIT = 'SELECT_SUBREDDIT'

export function selectSubreddit(subreddit) {
  return {
    type: SELECT_SUBREDDIT,
    subreddit
  }
}
```
また、「更新」ボタンを押して更新することもできます。
```javascript
export const INVALIDATE_SUBREDDIT = 'INVALIDATE_SUBREDDIT'

export function invalidateSubreddit(subreddit) {
  return {
    type: INVALIDATE_SUBREDDIT,
    subreddit
  }
}
```
これらは、ユーザーとの対話によって管理されるアクションでした。また、ネットワーク要求によって管理される別の種類のアクションもあります。後でそれらをディスパッチする方法がわかりますが、今のところ定義したいだけです。
```javascript
export const REQUEST_POSTS = 'REQUEST_POSTS'

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```
SELECT_SUBREDDITまたはINVALIDATE_SUBREDDITとは別にすることが重要です。アプリケーションが次第に複雑になっていく中で、ユーザーの操作とは独立して（たとえば、最も人気のあるサブディレクトリをプリフェッチしたり、古いデータを一度に更新するなど）、一部のデータを取得することができます。ルートの変更に応じて取得することもできます。早いうちに特定のUIイベントにフェッチすることは賢明ではありません。




