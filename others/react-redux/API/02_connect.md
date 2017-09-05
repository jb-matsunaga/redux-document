# connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

ReactコンポーネントをReduxストアに接続します。 connectはconnectAdvancedのまわりのファサードであり、最も一般的な使用事例に便利なAPIを提供します。

それは渡されたコンポーネントクラスを変更しません。代わりに、使用するための新しい接続されたコンポーネントクラスを返します。

## Arguments
### [mapStateToProps（state、[ownProps]）：stateProps]（Function）：
この引数が指定されている場合、新しいコンポーネントはReduxストア更新にサブスクライブします。これは、ストアが更新されるたびにmapStateToPropsが呼び出されることを意味します。 mapStateToPropsの結果は、コンポーネントのpropsにマージされる普通のオブジェクトでなければなりません。ストア更新を購読したくない場合は、mapStateToPropsの代わりにnullまたはundefinedを渡します。

mapStateToProps関数が2つのパラメーターを取ると宣言されている場合、ストアステートを最初のパラメーターとして呼び出され、2番目のパラメーターとして接続されたコンポーネントに渡され、接続されたコンポーネントが新しいpropsを受け取るたびに呼び出されます浅い等価比較によって決定される。 （2番目のパラメータは通常、規約に従ってownPropsと呼ばれます）。

> Note
> レンダリングのパフォーマンスをより詳細に制御する必要がある高度なシナリオでは、mapStateToProps（）も関数を返すことができます。この場合、その関数は特定のコンポーネントインスタンスのmapStateToProps（）として使用されます。これにより、インスタンスごとのメモを行うことができます。 [＃279](https://github.com/reactjs/react-redux/pull/279)と追加されたテストを参照してください。ほとんどのアプリはこれを必要としません。
> mapStateToProps関数は、Reduxストアの状態全体の1つの引数を取り、渡されるオブジェクトを返します。これはしばしばreselectと呼ばれます。reselectを効率的に作成し、[派生データを計算](http://redux.js.org/docs/recipes/ComputingDerivedData.html)するには、[reselect](https://github.com/reactjs/reselect)を使用します。

### [mapDispatchToProps(dispatch, [ownProps]): dispatchProps] (Object or Function):
オブジェクトが渡されると、その中の各関数はReduxアクション作成者であるとみなされます。同じ機能名を持ち、すべてのアクション作成者がディスパッチコールにラップされて直接呼び出されるオブジェクトは、コンポーネントのpropsにマージされます。

関数が渡された場合は、最初のパラメータとしてdispatchが与えられます。ディスパッチを使って自分のやり方でアクションクリエイターをバインドするオブジェクトを返すのはあなた次第です。 （ヒント：Reduxの[bindActionCreators（）](http://reactjs.github.io/redux/docs/api/bindActionCreators.html)ヘルパーを使用することができます）。

mapDispatchToProps関数が2つのパラメータを取ると宣言されている場合、最初のパラメータとしてdispatchで呼び出され、2番目のパラメータとして接続されたコンポーネントに渡され、接続されたコンポーネントが新しいpropsを受け取るたびに呼び出されます。 （2番目のパラメータは通常、規約に従ってownPropsと呼ばれます）。

独自のmapDispatchToProps関数またはアクション作成者でいっぱいのオブジェクトを指定しないと、デフォルトのmapDispatchToProps実装では、コンポーネントのプロップにディスパッチが挿入されます。

> Note:
> レンダリングのパフォーマンスをより詳細に制御する必要がある高度なシナリオでは、mapDispatchToProps（）も関数を返すことができます。この場合、その関数は特定のコンポーネントインスタンスのmapDispatchToProps（）として使用されます。これにより、インスタンスごとのメモを行うことができます。 [＃279](https://github.com/reactjs/react-redux/pull/279)と追加されたテストを参照してください。ほとんどのアプリはこれを必要としません。

### [mergeProps(stateProps, dispatchProps, ownProps): props] (Function):
指定すると、mapStateToProps（）、mapDispatchToProps（）、および親のpropsの結果が渡されます。それから返すプレーンオブジェクトは、ラップされたコンポーネントにpropsとして渡されます。この関数を指定して、propsに基づいて状態のスライスを選択するか、または行動作成者をpropsから特定の変数に結びつけることができます。これを省略すると、デフォルトでObject.assign（{}、ownProps、stateProps、およびdispatchProps）が使用されます。

### [options] (Object)
指定すると、コネクターの動作がさらにカスタマイズされます。 connect（）に渡すことができるオプションに加えて（下記参照）、connect（）は次の追加オプションを受け入れます。

- *[pure] (Boolean):* trueの場合、関連する状態/ propsオブジェクトがそれぞれの等価チェックに基づいて等しい場合、connect（）は再レンダリングとmapStateToProps、mapDispatchToProps、およびmergePropsの呼び出しを回避します。ラップされたコンポーネントが「純粋な」コンポーネントであり、そのプロップおよび選択されたReduxストアの状態以外の入力または状態に依存しないと仮定します。デフォルト値：true

- *[areStatesEqual] (Function):* 純粋な場合は、着信ストアの状態を以前の値と比較します。デフォルト値：strictEqual（===）

- *[areOwnPropsEqual] (Function):* 純粋な場合は、入ってくるpropsを以前の値と比較します。デフォルト値：shallowEqual 

- *[areStatePropsEqual] (Function):* 純粋な場合は、mapStateToPropsの結果を前の値と比較します。デフォルト値：shallowEqual

- *[areMergedPropsEqual] (Function):* 純粋な場合は、mergePropsの結果を前の値と比較します。デフォルト値：shallowEqual

- *[storeKey] (String):* ストアを読む場所からのコンテキストのキー。おそらく、複数の店舗を持つことが賢明でない場合にのみ必要です。デフォルト値： 'store'

## mapStateToPropsとmapDispatchToPropsの引数の数は、ownPropsを受け取るかどうかを決定します

>NOTE:
>関数の正式な定義に1つの必須パラメータ（関数の長さが1）が含まれている場合、`ownProps`は`mapStateToProps`および`mapDispatchToProps`に渡されません。例えば、以下のように定義された関数は、第2引数として`ownProps`を受け取りません。

```javascript
function mapStateToProps(state) {
  console.log(state); // state
  console.log(arguments[1]); // undefined
}
```

```javascript
const mapStateToProps = (state, ownProps = {}) => {
  console.log(state); // state
  console.log(ownProps); // undefined
}
```

必須パラメータまたは2つのパラメータを持たない関数はownPropsを受け取ります

```javascript
const mapStateToProps = (state, ownProps) => {
  console.log(state); // state
  console.log(ownProps); // ownProps
}
```

```javascript
function mapStateToProps() {
  console.log(arguments[0]); // state
  console.log(arguments[1]); // ownProps
}
```

```javascript
const mapStateToProps = (...args) => {
  console.log(args[0]); // state
  console.log(args[1]); // ownProps
}
```

### options.pureがtrueのときに接続を最適化する
options.pureがtrueの場合、connectはmapStateToProps、mapDispatchToProps、mergePropsへの不必要な呼び出しを回避し、最終的にレンダリングするために使用される複数の等価チェックを実行します。これには、areStatesEqual、areOwnPropsEqual、areStatePropsEqual、およびareMergedPropsEqualが含まれます。デフォルトはおそらく99％が適切ですが、パフォーマンスやその他の理由からカスタム実装でそれらを上書きすることをお勧めします。ここにいくつかの例があります：

- mapStateToProps関数が計算上高価で、あなたの状態の小さなスライスにのみ関係する場合は、areStatesEqualをオーバーライドすることもできます。 たとえば：areStatesEqual：（next、prev）=> prev.entities.todos === next.entities.todos; これは、状態スライスを除くすべての状態変化を効果的に無視します。

- storeのstateを変える不純なreducerがある場合は、常にfalseを返すように、areStatesEqualをオーバーライドすることができます（areStatesEqual：（）=> false）。 （これは、mapStateToProps関数に応じて、他の平等チェックにも影響を与える可能性があります）。

- 入ってくるpropsをホワイトリストにする方法として、areOwnPropsEqualを上書きすることができます。ホワイトリストのpropsにもmapStateToProps、mapDispatchToProps、およびmergePropsを実装する必要があります。 （たとえば、recomposeのmapPropsを使用するなど、この他の方法を実現する方が簡単かもしれません）。

- 関連するpropsが変更された場合にのみ新しいオブジェクトを返すmemoizedセレクターをmapStateToPropsが使用する場合、strictEqualを使用するためにareStatePropsEqualをオーバーライドすることができます。 これは、mapStateToPropsが呼び出されるたびに個々のpropsに対して余分な等価性チェックを行わないため、パフォーマンスはごくわずかです。

- 選択子が複雑なpropsを作成する場合は、deepEqualを実装するためにareMergedPropsEqualをオーバーライドすることができます。 ex：入れ子オブジェクト、新しい配列など（深度等価チェックは再レンダリングよりも速くなければなりません）


### Returns
提供された引数から派生したコンポーネントにstateクリエータとactionクリエータを渡す高次のリアクタコンポーネントクラス。これはconnectAdvancedによって作成され、この高次コンポーネントの詳細はここでカバーされています。

### Examples
#### dispatchするだけでstoreを聞かない
```javascript
export default connect()(TodoApp)
```

#### storeに登録せずにすべてのアクションクリエータ（addTodo、completeTodo、...）を挿入する
```javascript
import * as actionCreators from './actionCreators'
export def`ault connect(null, actionCreators)(TodoApp)
```

#### `dispatch`とグローバルstateのすべてのフィールドを注入する
>これをしないでください！ TodoAppはすべての状態の変更後に再レンダリングするため、パフォーマンスの最適化は行われません。 ビュー階層内の複数のコンポーネントに対して、より細かいconnect（）を使用する方がそれぞれが優れています 関連する状態のスライスを聞く。
```javascript
export default connect(state => state)(TodoApp)
```

#### `dispatch`と`todos`を注入する
```javascript
const mapStateToProps = (state) => {
  return { todos: state.todos }
}

export default connect(mapStateToProps)(TodoApp)
```

#### `todos`とすべての`action creators`を注入する
```javascript
import * as actionCreators from './actionCreators'

function mapStateToProps(state) {
  return { todos: state.todos }
}

export default connect(mapStateToProps, actionCreators)(TodoApp)
```

#### `todos`とすべてのアクションクリエータ（addTodo、completeTodo、...）をアクションとして挿入する
```javascript
import * as actionCreators from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return { actions: bindActionCreators(actionCreators, dispatch) }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### `todos`と特定のアクション作成者（addTodo）を挿入する
```javascript
import { addTodo } from './actionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return bindActionCreators({ addTodo }, dispatch)
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### `todos`、todoActionCreatorsをtodoActions、counterActionCreatorsをcounterActionsとして挿入する
```javascript
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    todoActions: bindActionCreators(todoActionCreators, dispatch),
    counterActions: bindActionCreators(counterActionCreators, dispatch)
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### todosとtodoActionCreatorsとcounterActionCreatorsをアクションとして一緒に挿入する
```javascript
import * as todoActionCreators from './todoActionCreators'
import * as counterActionCreators from './counterActionCreators'
import { bindActionCreators } from 'redux'

function mapStateToProps(state) {
  return { todos: state.todos }
}

function mapDispatchToProps(dispatch) {
  return {
    actions: bindActionCreators(Object.assign({}, todoActionCreators, counterActionCreators), dispatch)
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(TodoApp)
```

#### todosとすべてのtodoActionCreatorsとcounterActionCreatorsをpropsとして直接注入する

