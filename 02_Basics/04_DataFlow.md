# [Data Flow](http://redux.js.org/docs/basics/DataFlow.html)
Reduxアーキテクチャは、単一方向の厳密なデータフローを中心に展開されています。

これは、アプリケーションのすべてのデータが同じライフサイクルパターンに従うことを意味し、アプリケーションのロジックをより予測しやすく理解しやすくします。また、データの正規化が促進されるため、相互に認識されない同じデータの複数の独立したコピーが作成されることはありません。

あなたがまだ納得していない場合は、単方向データフローのための説得力のある議論のために、MotivationとFluxのケースをお読みください。 ReduxはFluxではありませんが、同じ主な利点があります。

Reduxアプリケーションのデータライフサイクルは、次の4つのステップに従います。

1. *`store.dispatch(action)`を呼び出します。*　　
  actionは、何が起こったかを記述する明白なオブジェクトです。例えば：
  ```javascript
  { type: 'LIKE_ARTICLE', articleId: 42 }
  { type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Mary' } }
  { type: 'ADD_TODO', text: 'Read the Redux docs.' }
  ```
  actionを非常に簡単なニューススニペットと考えてください。 "Maryは記事42を好きでした"または "Read the Redux docs." todosのリストに追加されました。  
  コンポーネントやXHRコールバック、スケジュールされた間隔など、アプリケーションのどこからでもstore.dispatch（action）を呼び出すことができます。

2. *Redux storeは、指定したreducer機能を呼び出します。*  
  storeは、2つの引数、つまり現在のstateツリーとactionをreducerに渡します。たとえば、todoアプリでは、root reducerは次のようなものを受け取ることがあります：

  ```javascript
  // 現在のアプリケーションのstate（todosと選択されたフィルタのリスト）
  let previousState = {
    visibleTodoFilter: 'SHOW_ALL',
    todos: [
      {
        text: 'Read the docs.',
        complete: false
      }
    ]
  }
  // 実行されているaction（Todoを追加）
  let action = {
    type: 'ADD_TODO',
    text: 'Understand the flow.'
  }
  // reducerは次のアプリケーションstateを返します
  let nextState = todoApp(previousState, action)
  ```
  reducerは純粋な関数であることに注意してください。次のstateだけを計算します。それは完全に予測可能でなければなりません。同じ入力を何度も呼び出すと、同じ出力が生成されるはずです。 APIコールやルーター移行などの副作用は実行しないでください。これらは、actionがdispatchされる前に発生します。

3. *root reducerは、複数のreducerの出力を単一のstateツリーに組み合わせることができる。*  
  root reducerの構造は、あなた次第です。 ReduxにはcombinedReducers()ヘルパ関数が付属しています。root reducerをstate treeの1つのブランチを管理する別々の関数に「分割」するのに便利です。  
  combineReducers()の働きは次のとおりです。 todoのリストと現在選択されているフィルタ設定の2つのreducerがあるとしましょう：  

  ```javascript
  const todos = (state = [], action) => {
    // どうにかそれを計算する...
    return nextState
  }

  const visibleTodoFilter = (state = 'SHOW_ALL', action) => {
    // どうにかそれを計算する...
    return nextState
  }

  let todoApp = combineReducers({
    todos,
    visibleTodoFilter
  })
  ```
  actionを発行するとき、`combineReducers`によって返された`todoApp`は両方のreducersを呼び出します：  
  ```javascript
  let nextTodos = todos(state.todos, action)
  let nextVisibleTodoFilter = visibleTodoFilter(state.visibleTodoFilter, action)
  ```
  次に、両方の結果セットを単一のstateツリーに結合します。
  ```javascript
  return {
    todos: nextTodos,
    visibleTodoFilter: nextVisibleTodoFilter
  }
  ```
  `combineReducers()`は便利なヘルパーユーティリティですが、それを使う必要はありません。あなた自身のroot reducerを書き留めてください！

4. *Redux storeは、root reducerによって返された完全なstateツリーを保存します。*  
この新しいツリーはあなたのアプリの次のstateになりました！ `store.subscribe(listener)`で登録されたすべてのlistenerが呼び出されます。listenerは`store.getState()`を呼び出して現在のstateを取得できます。  
これで、UIを更新して新しいstateを反映させることができます。 React Reduxのようなバインディングを使用する場合、これは`component.setState(newState)`が呼び出されるポイントです。

## Next Steps
Reduxの仕組みを知ったので、それをReactアプリケーションconnectしましょう。  
> *上級ユーザー向けの注意事項*  
基本的な概念をすでに理解していて、以前にこのチュートリアルを完了している場合は、高度なチュートリアルで非同期フローをチェックして、ミドルウェアが非同期アクションをレデューサーに到達する前に変換する方法を忘れないようにしてください。
