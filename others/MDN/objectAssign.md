# Object.assign()
[Object.assign()](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

*オブジェクトの割り当て*

Object.assign（）メソッドは、すべての列挙可能なプロパティの値を1つ以上のソースオブジェクトからターゲットオブジェクトにコピーするために使用されます。 ターゲットオブジェクトを返します。

## 構文
```javascript
Object.assign(target, ...sources)
```
### Parameters
#### target
ターゲットオブジェクト
#### sources
ソースオブジェクト
#### Return value
ターゲットオブジェクト

##説明
ターゲットオブジェクト内のプロパティは、同じキーを持つソースのプロパティによって上書きされます。 それ以降のソースのプロパティは、同様に以前のもののプロパティを上書きします。

Object.assign（）メソッドは、列挙可能なプロパティと独自のプロパティをソースオブジェクトからターゲットオブジェクトにのみコピーします。 それはソース上で[[Get]]を使い、ターゲット上で[[Set]]を使うので、getterとsetterを呼び出すでしょう。 したがって、新しいプロパティをコピーまたは定義するだけでなく、プロパティを割り当てます。 これにより、マージソースにゲッターが含まれている場合、新しいプロパティをプロトタイプにマージすることが不適切になることがあります。 列挙可能性を含むプロパティ定義をプロトタイプにコピーするには、Object.getOwnPropertyDescriptor（）およびObject.defineProperty（）を代わりに使用する必要があります。

StringプロパティとSymbolプロパティの両方がコピーされます。

エラーが発生した場合（たとえば、プロパティが書き込み不可能な場合）、TypeErrorが発生し、エラーが発生する前にプロパティが追加されていれば、ターゲットオブジェクトを変更できます。

**Object.assign（）はnullまたは未定義のソース値を投げないことに注意してください。**

## Examples
###　オブジェクトのクローン作成
```javascript
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); 
// { a: 1 }
```
## ディープクローンの警告
深いクローンを作成するには、Object.assign（）がプロパティ値をコピーするため、他の選択肢を使用する必要があります。 ソース値がオブジェクトへの参照である場合、その参照値のみがコピーされます。
```javascript
function test() {
    'use strict';

    let obj1 = { a: 0, b: {c: 0}};
    let obj2 = Object.assign({}, obj1);
    console.log(JSON.stringify(obj2));
    // {"a":0,"b":{"c":0}}

    obj1.a = 1;
    console.log('obj1.a = 1;');
    console.log(JSON.stringify(obj1));
    // {"a":1,"b":{"c":0}}
    console.log(JSON.stringify(obj2));
    // {"a":0,"b":{"c":0}}

    obj2.a = 2;
    console.log('obj2.a = 2;');
    console.log(JSON.stringify(obj1));
    // {"a":1,"b":{"c":0}}
    console.log(JSON.stringify(obj2));
    // {"a":2,"b":{"c":0}}

    obj2.b.c = 3;
    // ソース値がオブジェクトへの参照である場合、その参照値のみがコピー
    console.log('obj2.b.c = 3;');
    console.log(JSON.stringify(obj1));
    // {"a":1,"b":{"c":3}} <-----------??????
    console.log(JSON.stringify(obj2));
    // {"a":2,"b":{"c":3}}

    //ディープクローン
    obj1 = { a: 0, b: {c: 0}};
    let obj3 = JSON.parse(JSON.stringify(obj1));
    obj1.a = 4;
    obj1.b.c = 4;
    console.log('obj1 = { a: 0, b: {c: 0}};');
    console.log(JSON.stringify(obj3));
    // {"a":0,"b":{"c":0}}
}

test();
```
## オブジェクトのマージ
```javascript
let o1 = { a: 1 };
let o2 = { b: 2 };
let o3 = { c: 3 };

const obj = Object.assign(o1, o2, o3);
console.log(obj); 
// {a: 1, b: 2, c: 3}
console.log(o1); 
// {a: 1, b: 2, c: 3}
console.log(o2); 
// {b: 2}
```

## 同じプロパティを持つオブジェクトのマージ
```javascript
var o1 = { a: 1, b: 1, c: 1 };
var o2 = { b: 2, c: 2 };
var o3 = { c: 3 };

var obj = Object.assign({}, o1, o2, o3);
console.log(obj); 
// {a: 1, b: 2, c: 3}
```
プロパティは、後でパラメータの順序で同じプロパティを持つ他のオブジェクトによって上書きされます。

## シンボル型のプロパティのコピー
```javascript
var o1 = { a: 1 };
var o2 = { [Symbol('foo')]: 2 };

var obj = Object.assign({}, o1, o2);
console.log(obj); 
// {a: 1, Symbol(foo): 2}
Object.getOwnPropertySymbols(obj);
// [Symbol(foo)]
```

[more](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)