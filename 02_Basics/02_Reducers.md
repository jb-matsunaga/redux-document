# Reducers
アクションは何かが起こったという事実を記述しますが、アプリケーションの状態がどのように変化するかを指定しません。 これはReducersの仕事です。

## 状態形状の設計
Reduxでは、すべてのアプリケーション状態が単一のオブジェクトとして格納されます。 コードを書く前にその形状を考えることは良い考えです。 オブジェクトとしてのあなたのアプリの状態の最小限の表現は何ですか？

Todoアプリでは、2つの異なるものを保存したい：

- 現在選択されている可視性フィルタ。
- todosの実際のリスト。

いくつかのデータとUI状態を状態ツリーに格納する必要があることがよくあります。 これは問題ありませんが、データをUI状態と区別して保存してください。

```javascript
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
```

関係に関する注意

より複雑なアプリケーションでは、異なるエンティティが互いに参照するようにします。 ネストしないで、できるだけ正規化した状態を保つことをお勧めします。 IDが格納されているオブジェクト内のすべてのエンティティをキーとして保持し、IDを使用して他のエンティティまたはリストからそのIDを参照します。 アプリケーションの状態をデータベースと考えてください。 このアプローチは、法律家の文書に詳細に記載されています。 たとえば、状態の中にtodosById：{id - > todo}とtodos：array <id>を保持する方が、実際のアプリケーションではより良いアイデアになりますが、例を単純にしています。

## アクションの処理
状態オブジェクトがどのように見えるかを決定したので、そのためのReducerを書く準備ができました。 レデューサーは、前の状態とアクションを取り、次の状態を返す純粋な関数です。

```javascript
(previousState, action) => newState
```
これは、あなたがArray.prototype.reduce（reducer、？initialValue）に渡す関数のタイプであるため、Reducerと呼ばれています。 Reducerが純粋にとどまることは非常に重要です。

[Array.prototype.reduce()](https://github.com/jb-matsunaga/redux-document/blob/master/others/01_arrayPrototypeReduce.md)

reducerの中で以下のことをやってはいけません：
- 引数のstate, actionインスタンスの値を変更する
- 副作用をおこす(APIを呼んだり、ルーティングを変えるなどなど)
- 毎回値が変わるもの(Date.now() や Math.random())を扱う

>[Redux入門 3日目 Reduxの基本・Reducers(公式ドキュメント和訳)](http://qiita.com/kiita312/items/7fdce94912d6d9c801f8)

私たちは、高度なウォークスルーで副作用を実行する方法を探求します。 今のところ、reducerは純粋でなければならないことを覚えておいてください。 同じ引数が与えられた場合、次の状態を計算して返す必要があります。 驚く様な事じゃない。 副作用はありません。 API呼び出しはありません。 突然変異はありません。 ただの計算だけ。

[advanced walkthrough](http://redux.js.org/docs/advanced/)

これを解消して、先ほど定義したactionを理解するために徐々に教授してreducerを書き始めましょう。

初期状態を指定することから始めます。 Reduxは最初に定義されていない状態でreducerを呼びます。 これは私たちのアプリの初期状態を返すチャンスです：

```javascript
import { VisibilityFilters } from './actions'

const initialState = {
    visibilityFilter: VisibilityFilters.SHOW_ALL,
    todos: []
}

function todoApp(state, action) {
    if(typeof state === 'undefined') {
        return initialState
    }
    // 今は何もactionを処理しない
    // そして与えられたstateを返すだけ
    return state
}
```

1つのきちんとしたトリックは、これをよりコンパクトな方法で書くために、ES6のデフォルト引数の構文を使用することです。

[ES6 default arguments syntax](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/default_parameters)

```javascript
function todoApp(state = initialState, action) {
    // 今は何もactionを処理しない
    // そして与えられたstateを返すだけ
    return state
}
```

SET VISIBILITY FILTERを処理しましょう。 状態の可視性フィルタを変更するだけです。 簡単：

```javascript
function todoApp(state = initialState, action) {
   switch (action.type) {
       case SET_VISIBILITY_FILTER:
           return Object.assign({}, state, {
               visibilityFilter: action.filter
           })
        default:
            return state
   }
}
```

>*Note that:*
1. 私たちは状態を変えません。 Object.assign（）でコピーを作成します。 Object.assign（state、{visibilityFilter：action.filter}）も間違っています：最初の引数を変更します。 空のオブジェクトを最初のパラメータとして指定する必要があります。 また、オブジェクトスプレッドオペレータプロポーザルが{... state、... newState}を書き込むようにすることもできます。
[Object.assign()](https://github.com/jb-matsunaga/redux-document/blob/master/others/objectAssign.md)

2. デフォルトの場合、前の状態に戻ります。 未知の行動があれば前の状態に戻すことが重要です。

>*Note on Object.assign*
>Object.assign（）はES6の一部ですが、ほとんどのブラウザではまだ実装されていません。 polyfill、Babelプラグイン、または_.assign（）のような別のライブラリのヘルパーを使用する必要があります。
>*Note on switch and Boilerplate*
>switch文は実際の定型文ではありません。 Fluxの本当の定型文は、アップデートを発行する必要性、StoreをDispatcherに登録する必要性、Storeをオブジェクトにする必要性（普遍的なアプリケーションが必要な場合に発生する問題）です。 Reduxは、イベントエミッタの代わりに純粋なレデューサを使用することで、これらの問題を解決します。
>多くの人が、ドキュメント内でswitch文を使用するかどうかに基づいて、フレームワークを選択することは残念です。 スイッチが気に入らない場合は、「定型化を減らす」のように、ハンドラ・マップを受け入れるカスタムのcreateReducer関数を使用できます。

## その他のアクションの処理
処理するアクションが2つ追加されています。 SET_VISIBILITY_FILTERと同じように、ADD_TODOとTOGGLE_TODOアクションをインポートしてから、ADD_TODOを処理するためにレデューサーを拡張します。

```javascript
import { VisibilityFilters, ADD_TODO, TOGGLE_TODO } from './actions';


function todoApp(state = initialState, action) {
    switch(action.type) {
        case SET_VISIBILITY_FILTER:
            return Object.assign({}, state, {
                visibilityFilter: action.filter
            })
        case ADD_TODO:
            return Object.assign({}, state, {
                todos: [
                    ...state.todos,
                    {
                        text: action.text,
                        completed: false
                    }
                ]
            })
        default:
            return state
    }
}
```

前と同じように、状態やフィールドに直接書き込むことはなく、代わりに新しいオブジェクトを返します。新しいtodosは、最後の単一の新しいアイテムと連結された古いtodosに等しいです。アクションからのデータを使用して新しいtodoが構築されました。

最後に、TOGGLE_TODOハンドラの実装は完全な驚きではありません。


```javascript
case TOGGLE_TODO:
    return Object.assign({}, state, {
        todos: state.todos.map((todo, index) => {
            if (index === action.index) {
                return Object.assign({}, todo, {
                    completed: !todo.completed
                })
            }
            return todo
    })
})
```
突然変異に頼らずに配列内の特定のアイテムを更新したいので、インデックスにあるアイテム以外の同じアイテムを持つ新しい配列を作成する必要があります。そのような操作を書くことが多いと思われる場合は、immutability-helper、updeep、さらにはImmutableのようなヘルパーを使用して詳細な更新をネイティブにサポートすることをお勧めします。最初にクローンを作成しない限り、状態の内部に何も割り当てないことを忘れないでください。

## Splitting Reducers
ここまでは私たちのコードです。かなり冗長です。

```javascript
function todoApp(state = initialState, action) {
    switch (action.type) {
        case SET_VISIBILITY_FILTER:
        return Object.assign({}, state, {
            visibilityFilter: action.filter
        })
        case ADD_TODO:
        return Object.assign({}, state, {
            todos: [
                ...state.todos,
                {
                    text:action.text,
                    completed: false
                }
            ]
        })
        case TOGGLE_TODO:
        return Object.assign({}, state, {
            todos: state.todos.map((todo, index) => {
                if (index === action.index) {
                     return Object.assign({}, state, {
                        completed: !todo.completed
                    })
                }
                return todo
            })
        })
        default:
        return state
    }
}
```

理解しやすくする方法はありますか？ todosとvisibilityFilterは完全に独立して更新されているようです。時には状態フィールドが互いに依存し、より多くの配慮が必要になることもありますが、私たちの場合、簡単に更新することを別々の関数に分割することができます：

```javascript
function todos(state = [], action) {
    switch (action.type) {
        case ADD_TODO:
        return [
            ...state,
            {
                text: action.text,
                completed: false
            }
        ]
        case TOGGLE_TODO:
        return state.map((todo, index) => {
            if (index === action.index) {
                return Object.assign({}, todo, {
                    completed: !todo.completed
                })
            }
            return todo
        })
        default:
        return state
    }
}

function todoApp(state = initialState, action) {
    switch (action.type) {
        case SET_VISIBILITY_FILTER:
        return Obhect.assign({}, state, {
            visibilityFilter: action.filter
        })
        case ADD_TODO:
        case TOGGLE_TODO:
        return Object.assign({}, state, {
            todos: todos(state.todos, action)
        })
        default:
        return state
    }
}
```

todosはstateも受け入れますが、配列であることに注意してください。今すぐtodoAppは管理する状態のスライスを与え、todosはそのスライスだけを更新する方法を知っています。これはレデューサー構成と呼ばれ、Reduxアプリケーションを構築する基本的なパターンです。

Reducerの構成をもっと調べてみましょう。私たちはまた、視界を管理するReducerを抽出できますか？私たちはできる。

インポートの下で、ES6 Object Destructuringを使用してSHOW_ALLを宣言しましょう：

```javascript
const { SHOW_ALL } = VisibilityFilters
```

Then:

```javascript
function visibilityFilter(state = SHOW_ALL, action) {
    switch (action.type) {
        case SET_VISIBILITY_FILTER:
        return action.filter
        default:
        return state
    }
}
```

今度は、主なReducerを、状態の一部を管理するReducerを呼び出す関数として書き直し、それらを単一のオブジェクトに結合することができます。また、完全な初期状態を知る必要もありません。最初に未定義のときに子レデューサーが初期状態に戻っていれば十分です。

```javascript
function todos(state = [], action) {
    switch (action.type) {
        case ADD_TODO:
        return [
            ...state,
            {
                text: action.text,
                completed: false
            }
        ]
        case TOGGLE_TODO:
        return state.map((todo, index) => {
            if (index === action.index) {
                return Object.assign({}, todo, {
                    completed: !todo.completed
                })
            }
            return todo
        })
        default:
        return state
    }
}

function visibilityFilter (state = SHOW_ALL, action) {
    switch (action.type) {
        case SET_VISIBILITY_FILTER:
        return action.filter
        default:
        return state
    }
}

function todoApp(state = {}, action) {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
}
```

*これらのReducerのそれぞれは、グローバルstateのそれ自体の部分を管理していることに注意してください。stateパラメータはすべてのReducerで異なり、それが管理するstateの一部に対応します。*

これはすでによく見ています！アプリが大きくなると、レデューサーを別々のファイルに分割し、それらを完全に独立させ、異なるデータドメインを管理することができます。

最後にReduxはcombineReducers（）というユーティリティーを提供しています。これは、上記のtodoAppと同じ定型ロジックを実行します。その助けを借りて、todoAppを次のように書き直すことができます：

```javascript
import { combineReducers } from 'redux'

const todoApp = combineReducers({
    visibilityFilter,
    todos
})

export default todoApp
```

これは以下と等価であることに注意してください。

```javascript
export default function todoApp(state = {}, action) {
    return {
        visibilityFilter: visibilityFilter(state.visibilityFilter, action),
        todos: todos(state.todos, action)
    }
}
```

また、異なるキーや関数を別々に呼び出すこともできます。複合Reducerを書くためのこれらの2つの方法は同等です：

```javascript
comst reducer = combineReducers({
    a: doSomethingWithA,
    b: processB,
    c: c
})
```
```javascript
function reducer(state = {}, action) {
    return {
        a: doSomethingWithA(state.a, action),
        b: processB(state.b, action),
        c: c(state.c, action)
    }
}
```
すべてのcombineReducers（）は、キーに従って選択された状態のスライスを使用してレデューサーを呼び出す関数を生成し、その結果を再び単一のオブジェクトに結合します。それは魔法ではない。 combineReducers（）は、他のレデューサーと同様に、レデューサーのすべてが状態を変更しない場合、新しいオブジェクトを作成しません。




