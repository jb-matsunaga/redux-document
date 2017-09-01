# connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])

ReactコンポーネントをReduxストアに接続します。 connectはconnectAdvancedのまわりのファサードであり、最も一般的な使用事例に便利なAPIを提供します。

それは渡されたコンポーネントクラスを変更しません。代わりに、使用するための新しい接続されたコンポーネントクラスを返します。

## Arguments
### [mapStateToProps（state、[ownProps]）：stateProps]（Function）：
この引数が指定されている場合、新しいコンポーネントはReduxストア更新にサブスクライブします。これは、ストアが更新されるたびにmapStateToPropsが呼び出されることを意味します。 mapStateToPropsの結果は、コンポーネントの小道具にマージされる普通のオブジェクトでなければなりません。ストア更新を購読したくない場合は、mapStateToPropsの代わりにnullまたはundefinedを渡します。

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

