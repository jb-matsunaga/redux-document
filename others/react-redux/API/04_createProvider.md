# createProvider([storeKey])
コンテキストの渡されたキーにRedux storeを設定する新しい`<Provider>`を作成します。おそらく、複数のstoreを持つのが賢明でない場合にのみ必要です。また、同じ`storeKey`を`connect`の`options`引数に渡す必要もあります。

## Arguments
- *[`storeKey`] (String):*ストアを設定するコンテキストのキー。デフォルト値： `'store'`

## Examples
複数のstoreを作成する前に、以下のFAQをご覧ください。
[複数の店舗を作ることはできますか？](http://redux.js.org/docs/faq/StoreSetup.html#can-or-should-i-create-multiple-stores-can-i-import-my-store-directly-and-use-it-in-components-myself)
```javascript
import { connect, createProvider } from 'react-redux'

const STORE_KEY = 'componentStore'

export const Provider = createProvider(STORE_KEY)

const connectExtended = (
  mapStateToProps,
  mapDispatchToProps,
  mergeProps,
  options = {}
) {
  options.storeKey = STORE_KEY
  return connect (
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    options
  )
}

export { connectExtended as connect }
```
これで、上記の`Provider`をインポートし、`connect`して使用することができます。
