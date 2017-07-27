# イントロダクション
## モチベーション
JavaScriptのシングルページアプリケーションの要件がますます複雑になるにつれて、私たちのコードはこれまで以上に多くの状態を管理する必要があります。 この状態には、サーバー応答とキャッシュされたデータ、およびサーバーにまだ永続化されていないローカルに作成されたデータが含まれます。 アクティブなルート、選択されたタブ、スピナー、ページネーションコントロールなどを管理する必要があるため、UIの状態も複雑になります。

この絶え間なく変化する状態を管理するのは難しいです。 あるモデルが別のモデルを更新できる場合、ビューはモデルを更新し、別のモデルを更新し、別のモデルが更新される可能性があります。 ある時点で、いつ、どうやってその状態を制御できなくなったとしても、アプリで何が起きるか理解できなくなります。 システムが不透明で非確定的な場合は、バグを再現したり、新しい機能を追加することは難しいです。

これが十分に悪くないかのように、フロントエンドの製品開発では新しい要件が一般的になります。 開発者としては、楽観的な更新、サーバー側のレンダリング、経路遷移を実行する前にデータを取得することなどが期待されます。 これまで対処していなかった複雑さを管理しようとしていることがわかりました。必然的に、あきらめる時が来ましたか？答えはノーです。

この複雑さは、人の心が推論するために非常に難しい2つのコンセプト、すなわち突然変異と非同期性を混合しているので、扱いにくいです。 私はそれらをMentosとCokeと呼んでいます。 どちらも分離で素晴らしいことができますが、一緒になって混乱を招きます。 Reactのようなライブラリは、非同期操作と直接的なDOM操作の両方を取り除いて、ビュー層でこの問題を解決しようとします。 しかし、データの状態を管理することはあなた次第です。 これはReduxが入る場所です。

Flux、CQRS、およびEvent Sourcingの手順に従って、Reduxは、どのように更新が行われるかについて一定の制限を課すことによって、状態の変異を予測可能にしようとします。 これらの制限は、Reduxの3つの原則に反映されています。

## コアコンセプト
Redux自体はとてもシンプルです。

あなたのアプリの状態が単純なオブジェクトとして記述されているとします。 たとえば、todoアプリの状態は次のようになります。

```javascript
{
  todos: [{
    text: 'Eat food',
    completed: true
  }, {
    text: 'Exercise',
    completed: false
  }],
  visibilityFilter: 'SHOW_COMPLETED'
}
```

このオブジェクトは、セッターがないことを除いて、 "モデル"のようなものです。 これは、コードのさまざまな部分が状態を任意に変更することができず、再現しにくいバグを引き起こすためです。

状態を変更するには、アクションをディスパッチする必要があります。 アクションとは、何が起こったのかを記述するプレーンなJavaScriptオブジェクト（どのような魔法を導入しないかに注意してください）です。 いくつかのアクションの例を次に示します。

```javascript
{ type: 'ADD_TODO', text: 'Go to swimming pool' }
{ type: 'TOGGLE_TODO', index: 1 }
{ type: 'SET_VISIBILITY_FILTER', filter: 'SHOW_ALL' }
```

すべての変更がアクションとして記述されていることを強制することで、アプリで何が起こっているのかを明確に理解できます。 変更されたものがあれば、その変更理由がわかります。 行動は、起こったことのブレッドクラムと同じです。 最後に、状態とアクションを結びつけるために、我々はReducerと呼ばれる関数を書く。 繰り返しになりますが、それについては何も魔法のようなものではありません。それは引数として状態とアクションを取り、関数の次の状態を返します。 大きなアプリではこのような関数を書くのは難しいので、状態の一部を管理する小さな関数を書く：

```javascript
function visibilityFilter(state = 'SHOW_ALL', action) {
  if (action.type === 'SET_VISIBILITY_FILTER') {
    return action.filter
  } else {
    return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return state.concat([{ text: action.text, completed: false }])
    case 'TOGGLE_TODO':
      return state.map(
        (todo, index) =>
          action.index === index
            ? { text: todo.text, completed: !todo.completed }
            : todo
      )
    default:
      return state
  }
}
```

そして、対応する状態キーのために2つのReducerを呼び出すことによって、アプリケーションの完全な状態を管理する別のReducerを書きます：

```javascript
function todoApp(state = {}, action) {
  return {
    todos: todos(state.todos, action),
    visibilityFilter: visibilityFilter(state.visibilityFilter, action)
  }
}
```

これは、基本的にReduxの考え方です。 Redux APIは使用していないことに注意してください。 このパターンを容易にするためのユーティリティがいくつかありますが、主な考え方は、アクションオブジェクトに応じて状態がどのように更新されるかを記述し、コードの90％がReduxを使用しない単純なJavaScriptです それ自体、そのAPI、または任意の魔法です。

# 三つの原則
Reduxは3つの基本原則で記述することができます。
## 真実の単一の源
アプリケーション全体の状態は、単一のストア内のオブジェクトツリーに格納されます。

これにより、ユニバーサルアプリケーションの作成が容易になります。サーバからの状態をシリアライズして、余分なコーディング作業をせずにクライアントに提供することができます。 また、単一の状態ツリーを使用すると、アプリケーションのデバッグや検査が容易になります。 開発サイクルを短縮するために、アプリケーションの状態を開発中も維持することができます。 すべての状態が1つのツリーに格納されていれば、従来は実装が難しかった機能の一部（たとえば、元に戻す/やり直し）が突然実現することがあります。

```javascript
console.log(store.getState())

/* Prints
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
*/
```

## 状態は読み取り専用です
状態を変更する唯一の方法は、何が起こったのかを記述するオブジェクトを生成することです。

これにより、ビューもネットワークコールバックも状態に直接書き込まれることはありません。 代わりに、彼らはstateを変える意思を表明する。 すべての変更は集中管理され、厳密な順序で1つずつ実行されるため、微妙な競合条件はありません。 アクションは単純なオブジェクトなので、デバッグやテストの目的でログに記録し、シリアル化し、保存し、後で再生することができます。

```javascript
store.dispatch({
  type: 'COMPLETE_TODO',
  index: 1
})

store.dispatch({
  type: 'SET_VISIBILITY_FILTER',
  filter: 'SHOW_COMPLETED'
})
```

## 変更は純粋な関数で行われます
状態ツリーがアクションによってどのように変換されるかを指定するには、純粋なReducerを書きます。

Reducerは、以前の状態とアクションを取り、次の状態を返す純粋な関数です。 以前の状態に変更するのではなく、新しい状態オブジェクトを返すことを忘れないでください。 単一のReducerから始めることができます。アプリが成長するにつれて、ステートツリーの特定の部分を管理するより小さなReducerに分割することができます。 Reducerは単なる関数なので、呼び出される順序を制御したり、追加データを渡したり、ページネーションなどの一般的な作業で再利用可能なReducerを作成することもできます。

```javascript
function visibilityFilter(state = 'SHOW_ALL', action) {
  switch (action.type) {
    case 'SET_VISIBILITY_FILTER':
      return action.filter
    default:
      return state
  }
}

function todos(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ]
    case 'COMPLETE_TODO':
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: true
          })
        }
        return todo
      })
    default:
      return state
  }
}

import { combineReducers, createStore } from 'redux'
const reducer = combineReducers({ visibilityFilter, todos })
const store = createStore(reducer)
```

それでおしまい！ 今、あなたはReduxがすべてについて知っています。

# 従来技術
Reduxにはさまざまな遺産があります。 それはいくつかのパターンや技術に似ていますが、重要な点で異なっています。 類似点と相違点のいくつかを以下で探求します。

## Flux 
ReduxをFlux実装と見なすことはできますか？
はいといいえ。
（Fluxのクリエイターは、あなたが知りたいと思っていればそれを心配しないでください。）

ReduxはFluxのいくつかの重要な特質に触発されました。 Fluxと同様に、Reduxは、モデル更新ロジックをアプリケーションの特定のレイヤーに集中させることを規定しています（Fluxでは "store"、Reduxでは "reducers"）。 アプリケーションコードがデータを直接突然変異させるのではなく、すべての突然変異を「アクション」と呼ばれる単純なオブジェクトとして記述するように指示します。

Fluxとは異なり、ReduxにはDispatcherの概念はありません。 これは、イベントエミッタの代わりに純粋な関数に依存するためであり、純粋な関数は簡単に作成でき、エンティティを管理する追加のエンティティは必要ありません。 Fluxをどのように表示するかによって、これは偏差または実装の詳細のいずれかとみなされます。 フラックスはしばしば（状態、作用）=>状態として記述されている。 この意味で、ReduxはFluxアーキテクチャには当てはまりますが、純粋な機能のおかげでより簡単になります。

Fluxとのもう1つの重要な相違点は、Reduxはデータを変更しないと仮定していることです。 あなたの状態には普通のオブジェクトや配列を使っても問題ありませんが、Reducerの内部でそれらを変更することはお勧めしません。 新しいオブジェクトを返す必要があります。これは、オブジェクトスプレッド演算子の提案や、不変のようなライブラリを使用すると簡単です。

性能上のコーナーケースのためにデータを突然変異させる不純なReducerを書くことは技術的に可能ですが、私たちは積極的にこれをやめないようにします。 タイムトラベル、録画/再生、ホットリロードなどの開発機能は中断されます。 さらに、Omが示すように、オブジェクトの割り当てを失ったとしても、高価な再レンダリングや再計算を避けることで、まだ勝つことができるため、ほとんどの実際のアプリケーションでパフォーマンスの問題が生じることはありません。 Reducerの純粋さのおかげで変わりました。

# Ecosystem
Reduxは小さなライブラリですが、契約やAPIは、ツールや拡張のエコシステムを生み出すために慎重に選択されています。

Reduxに関連するすべての広範なリストについては、Awesome Reduxをお勧めします。 サンプル、定型文、ミドルウェア、ユーティリティライブラリなどが含まれています。 React / Redux Linksには、ReactやReduxを学ぶ人のためのチュートリアルやその他の有用なリソースが含まれています.Redux Ecosystem Linksには、多くのRedux関連のライブラリとアドオンがリストされています。

[awesome-redux](https://github.com/xgrommx/awesome-redux)    
[react-redux-links](https://github.com/markerikson/react-redux-links)     
[redux-ecosystem-links](https://github.com/markerikson/redux-ecosystem-links)

このページでは、Reduxのメンテナーが個人的に審査したものの一部を紹介します。 これがあなたの残りの部分を試してしまうのを妨げないようにしてください！ エコシステムは急速に成長しており、すべてを見る時間が限られています。 これらを「スタッフのおすすめ」とみなし、Reduxですばらしいことを構築した場合は、PRを提出することを躊躇しないでください。





