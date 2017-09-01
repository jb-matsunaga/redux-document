# `<Provider store>`
以下のコンポーネント階層のconnect（）呼び出しでReduxストアを使用できるようにします。通常、<Provider>内の親コンポーネントまたは祖先コンポーネントをラップすることなくconnect（）を使用することはできません。

本当に必要な場合は、すべてのconnect（）のコンポーネントに手動でstoreをpropsとして渡すことができますが、単体テストや非完全コードベースのスタブストアの場合にのみ行うことをお勧めします。通常、`<Provider>`を使用するだけです。

## Props
- store（Redux Store）：アプリケーション内の単一のReduxストア
- children（ReactElement）コンポーネント階層のルート

## Example
### Vanilla React

```javascript
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```

### React Router
```javascript
ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      <Route path="/" component={App}>
        <Route path="foo" component={Foo}/>
        <Route path="bar" component={Bar}/>
      </Route>
    </Router>
  </Provider>,
  document.getElementById('root')
)
```
