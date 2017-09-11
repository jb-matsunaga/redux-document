# connectAdvanced(selectorFactory, [connectOptions])
ReactコンポーネントをRedux storeに接続します。これは  `connect()`の基盤ですが、最終的なpropsに`state`、`props`、および`dispatch`をどのように組み合わせるかについてはあまり心配されていません。結果のデフォルトやメモの作成については想定されておらず、これらの責任は発信者に任せています。

それは渡されたコンポーネントクラスを変更しません。代わりに、使用するための新しい接続されたコンポーネントクラスを返します。

## Arguments
- *`selectorFactory(dispatch, factoryOptions): selector(state, ownProps): props`(Function):*
各インスタンスのコンストラクタの間にセレクタ関数を初期化します。このセレクター関数は、store stateの変更または新しいpropsの受け取りの結果として、コネクター・コンポーネントが新しいpropsを計算する必要があるときはいつでも呼び出されます。
`selector`の結果は、単純なオブジェクトであると予想され、ラッパーされたコンポーネントに渡されます。
`selector`への連続呼び出しが前の呼び出しと同じオブジェクト（`===`）を返す場合、コンポーネントは再レンダリングされません。適切なときに前のオブジェクトを返すのは`selector`の責任です。


- *[`connectOptions`] (Object)* 指定すると、コネクターの動作をさらにカスタマイズします。
  - *[`getDisplayName`] (Function):*ラップされたコンポーネントのdisplayNameプロパティを基準にコネクタコンポーネントのdisplayNameプロパティを計算します。通常、ラッパー関数によってオーバーライドされます。
  デフォルト値：`name => 'ConnectAdvanced('+name+')'`
  - *[`methodName`] (String):*エラーメッセージに表示されます。通常、ラッパー関数によってオーバーライドされます。デフォルト値： `'connectAdvanced'`
  - *[`renderCountProp`] (String):*定義されている場合は、この値の名前のプロパティがラップされたコンポーネントに渡された小道具に追加されます。その値は、コンポーネントがレンダリングされた回数になります。これは、不要な再レンダリングを追跡するのに役立ちます。デフォルト値：`undefined`
  - *[`shouldHandleStateChanges`] (Boolean):*コネクタコンポーネントがStoreステートの変更を反映するかどうかを制御します。 falseに設定すると、componentWillReceivePropsでのみ再描画されます。デフォルト値：`true`
  - *[`storeKey`] (String):*props/contextのkeyはstoreを得るためです。おそらく、複数のstoreを持つのが賢明でない場合にのみ必要です。デフォルト値： `'store'`
  - *[`withRef`] (Boolean):*trueの場合、ラップされたコンポーネントインスタンスにrefを格納し、getWrappedInstance（）メソッドを使用してそれを使用可能にします。デフォルト値：`false`
  - 付加的に、`connectOptions`経由で渡された余分なオプションは、`factoryOptions`引数の`selectorFactory`に渡されます。


## Returns
store stateからpropsを構築し、それをラップされたコンポーネントに渡す高次のReactコンポーネントクラス。上位コンポーネントは、コンポーネントの引数を受け取り、新しいコンポーネントを返す関数です。

### Static Properties
- *`WrappedComponent` (Component):*元のコンポーネントクラスが`connectAdvanced（...）（Component）`に渡されました。

### Static Methods
コンポーネントの元の静的メソッドはすべて持ち上げられます。

### Instance Methods
`getWrappedInstance(): ReactComponent`

ラップされたコンポーネントインスタンスを返します。 options引数の一部として{withRef：true}を渡す場合にのみ使用できます。

## Remarks
- `connectAdvanced`は高次コンポーネントを返すため、2回呼び出す必要があります。最初に、前述の引数を使用して、もう一度`connectAdvanced(selectorFactory)(MyComponent)`というコンポーネントを使用します。
- `connectAdvanced`は、渡されたReactコンポーネントを変更しません。代わりに使用する必要がある新しい接続コンポーネントが返されます。

## Examples
propsに応じて特定のユーザーのtodosを注入し、props.userIdをアクションに注入する
```javascript
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

const selectorFactory = dispatch => {
  let ownProps = {}
  let result = {}
  const actions = bindActionCreators(actionCreators, dispatch)
  return (nextState, nextOwnProps) => {
    const todos = nextState.todos[nextOwnProps.userId]
    const nextResult = { ...nextOwnProps, todos, addTodo }
    ownProps = nextOwnProps
    if (!shallowEqual(result, nextResult)) result = nextResult
    return result 
  }
}
export default connectAdvanced(selectorFactory)(TodoApp)
```

