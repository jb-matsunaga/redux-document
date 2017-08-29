# Async Actions
基本ガイドでは、簡単なToDoアプリケーションを作成しました。それは完全に同期していた。ActionがDispatchされるたびに、Stateは直ちに更新されました。

このガイドでは、異なる非同期アプリケーションを構築します。 Reddit APIを使用して、選択したサブディレクトリの現在の見出しを表示します。非同期性はどのようにReduxの流れに適合しますか？

## Actions
非同期APIを呼び出すと、呼び出しを開始した瞬間と応答（またはタイムアウト）を受け取った瞬間の2つの重要な瞬間があります。

これらの2つの瞬間は、通常、アプリケーションのStateを変更する必要があります。これを行うには、Reducerで同時に処理される通常のActionをDispatchする必要があります。通常、どのAPIリクエストでも少なくとも3種類のActionをDispatchする必要があります：

- Requestが開始されたことをReducerに通知するAction。
  - Reducerは、StateでisFetchingフラグをトグルすることによってこのActionを処理することができます。このようにして、UIはスピナーを表示する時間を知っています。

- Requestが正常に終了したことをReducerに通知するAction。
 - Reducerは、新しいデータを管理したり、リセットしたりするStateにマージすることによって、このActionを処理することができます。 UIはスピナーを隠し、取得したデータを表示します。

 - Requestが失敗したことをReducerに通知するAction。
  - ReducerはisFetchingをリセットすることでこのActionを処理できます。さらに、UIが表示できるようにエラーメッセージを格納したい場合があります。

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

フラグ付きの単一のActionタイプを使用するか複数のActionタイプを使用するかは、あなた次第です。あなたのチームで決定する必要があるしきたりです。複数のタイプには間違いの余地が少なくなりますが、あなたはredux-actionのようなヘルパーライブラリを使ってAction CreaterとReducerを生成しても問題ありません。

どのようなしきたりを選んでも、アプリケーション全体にこだわることができます。 このチュートリアルでは、別々のタイプを使用します。

## Synchronous Action Creators
まず、サンプルアプリケーションで必要ないくつかの同期ActionタイプとAction Createrを定義してみましょう。ここで、ユーザーは表示するサブレディを選択できます。

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
これらは、ユーザーとの対話によって管理されるActionでした。また、ネットワークRequestによって管理される別の種類のActionもあります。後でそれらをDispatchする方法がわかりますが、今のところ定義したいだけです。
```javascript
export const REQUEST_POSTS = 'REQUEST_POSTS'

function requestPosts(subreddit) {
  return {
    type: REQUEST_POSTS,
    subreddit
  }
}
```
*SELECT_SUBREDDITまたはINVALIDATE_SUBREDDITとは別にすることが重要です。* アプリケーションが次第に複雑になっていく中で、ユーザーの操作とは独立して（たとえば、最も人気のあるサブディレクトリをプリフェッチしたり、古いデータを一度に更新するなど）、一部のデータを取得することができます。ルートの変更に応じて取得することもできます。早いうちに特定のUIイベントにフェッチすることは賢明ではありません。

最後に、ネットワークリクエストが送信されると、RECEIVE_POSTSが送信されます。

```javascript
export const RECEIVE_POSTS = 'RECEIVE_POSTS'

function receivePosts(subreddit, json) {
  return {
    type: RECEIVE_POSTS,
    subreddit,
    posts: json.data.children.map(child => child.data),
    receiveAt: Date.now()
  }
}
```
これは今私たちが知る必要があるすべてです。ネットワーク要求とともにこれらのアクションをディスパッチする特定のメカニズムについては後で説明します。

> ### エラー処理に関する注意
> 実際のアプリケーションでは、リクエストの失敗時にアクションをディスパッチする必要があります。このチュートリアルではエラー処理を実装しませんが、実際の例では可能なアプローチの1つを示しています。

## State形状の設計
基本チュートリアルのように、実装に突入する前に、アプリケーションの状態の形状を設計する必要があります。非同期コードでは、処理する状態が増えているので、それを考える必要があります。

非同期アプリケーションの状態をどのような情報が記述しているのか、および単一のツリーでどのように構成するのかがすぐに分かるわけではないため、この部分は初心者にはしばしば混乱します。

最も一般的なユースケースであるリストから始めましょう。 Webアプリケーションは多くの場合、物事のリストを表示します。たとえば、投稿のリスト、または友人のリストです。あなたのアプリが表示できるリストの種類を把握する必要があります。あなたはそれらをキャッシュすることができ、必要な場合にのみ再度フェッチすることができるので、これらを別々に状態に保存したいとします。

私たちの "Reddit headlines"アプリの状態の形状は次のようになります：

```javascript
{
  selectedSubreddit: 'frontend',
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 12345678,
      items: [
        {
          id: 42,
          title: 'Confusion about Flux and Relay'
        },
        {
          id: 500,
          title: 'Creating a Simple Application Using React JS and Flux Architecture'
        }
      ]
    }
  }
}
```
いくつかの重要なビットがここにあります：
- 各サブディレクトリの情報を別々に保存するので、すべてのサブディレクトリをキャッシュできます。ユーザーが2回目に切り替えると、瞬時に更新されます。必要な場合を除き、更新は必要ありません。これらのアイテムがすべてメモリに格納されていることを心配する必要はありません。数万点のアイテムを扱っていて、そのユーザーがタブを閉じることはめったにない限り、クリーンアップは一切必要ありません。

- アイテムのリストごとに、isFetchingを保存してスピナーを表示し、didInvalidateを使用して、データが古くなったときにトグルすることができます。最後にフェッチされた時刻を知ることができます。実際のアプリケーションでは、fetchedPageCountやnextPageUrlのようなページネーション状態を保存する必要があります。

> ### ネストされたエンティティに関する注意
> この例では、受信したアイテムをページネーション情報とともに保存します。ただし、ネストされたエンティティが互いに参照している場合や、ユーザーがアイテムを編集できるようにしている場合は、この方法はうまく機能しません。ユーザーが取得した投稿を編集したいとしますが、この投稿は状態ツリーの複数の場所に複製されているとします。これは実装するのが本当に苦しいでしょう。

> ネストされたエンティティがある場合、またはユーザーが受信したエンティティを編集できるようにする場合は、データベースであるかのように状態を別々に保持する必要があります。ページネーション情報では、IDで参照するだけです。これにより、常に最新の状態に保つことができます。現実世界の例では、ネストされたAPIレスポンスを正規化するためにnormalizrと一緒にこのアプローチを示しています。このアプローチでは、あなたの状態は次のようになります。

> 
```javasctipt
{
  selectedSubreddit: 'frontend',
  entities: {
    users: {
      2: {
        id: 2,
        name: 'Andrew'
      }
    },
    posts: {
      42: {
        id: 42,
        title: 'Confusion about Flux and Relay',
        author: 2
      },
      100: {
        id: 100,
        title: 'Creating a Simple Application Using React JS and Flux Architecture',
        author: 2
      }
    }
  },
  postsBySubreddit: {
    frontend: {
      isFetching: true,
      didInvalidate: false,
      items: []
    },
    reactjs: {
      isFetching: false,
      didInvalidate: false,
      lastUpdated: 1439478405547,
      items: [ 42, 100 ]
    }
  }
}
```
> このガイドでは、エンティティを正規化することはしませんが、より動的なアプリケーションを検討する必要があります。



