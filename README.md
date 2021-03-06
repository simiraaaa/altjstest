# なにこれ?

ここから遊べる
http://simiraaaa.github.io/altjstest/

AltJSを作ってみたかったので、作った。
主に演算子オーバーロードが実装したかった。
しかし、pegjsで出力したparserは思いの外、速度が遅いので、実用的かどうか微妙。

# 追加機能

- 演算子関数(演算子オーバーロード)
- 関数リテラル `#`
- this リテラル `@`
- return リテラル `^^`
- 逆代入式 `式 ]> 変数名` `式 ]>> 変数(メンバ可能)`
- 逆代入付初期化 `var 逆代入式`
- その他細かい構文調整
  
# 構文

JavaScriptそのものをオーバーロードしたような言語なので、基本的にJSをそのまま書いても動く。

## 演算子関数(演算子オーバーロード)

`変数 演算子: 引数`で呼び出す。
引数はなくても良いが、その場合は、明示的`;`を書かない限り改行しても、次の式が引数となる。

演算子関数の優先順位は `*` より高く `++`より低い。

### 演算子関数の定義

オブジェクトのメンバとして、演算子と同名のメンバを用意することで使用でき量になる。

例: 

```js

var obj = {
  +: #(v) {
    ^^{
      x: @x + v.x,
      y: @y + v.y,
    };
  },
  
  +=: #(v){
    @x += v.x;
    @y += v.y;
    ^^@;
  },
  
  >>: #(){
    console.log(@x, @y);
    ^^@;
  },
  
  x:0,
  y:0,
};

obj['-'] = #(v){
  ^^{
    x: @x - v.x,
    y: @y - v.y,
  }
};



```

例のようにオブジェクトリテラル内では演算子を文字列リテラルで囲まなくても良い

### 演算子関数の呼び出し

```js

// さっきのobjが使えるものとする

obj +=: {x: 1, y: 2};
obj >>: ; // 1, 2
var obj2 = obj +: {x:2, y:3};

// メソッドチェーンも可能
var obj3 = obj +=: obj2 >>: -: {x: 2, y: 1};

```

### 演算子関数で使用可能な演算子

- `===`
- `!==`
- `>>>=`
- `<<=`
- `>>=`
- `**=`
- `>>>`
- `==`
- `|=`
- `&=`
- `^=`
- `+=`
- `-=`
- `*=`
- `/=`
- `%=`
- `<=`
- `>=`
- `!=`
- `<<`
- `>>`
- `||`
- `&&`
- `**`
- `+`
- `-`
- `*`
- `/`
- `%`
- `<`
- `>`
- `=`
- `|`
- `&`
- `~`
- `^`
- `?`
- `!`

## 逆代入式
式を評価してから代入する変数を指定する
簡単に言えば、普通の代入の逆になるだけ。
`a = 1`は`1 ]> a`と書き換えることができる。

複雑な式になるとカッコがたくさん出てきて読みづらくなるが、そういうときに使える。

`a = (b = c + d) * 3`
これを逆代入式で書き換えると
`c + d ]> b * 3 ]> a`
となる。

cとdを足したものをb代入し、それを3倍したものをaに代入するという感じに、思考したまま、より直感的に記述できる。

また、単純に変数を用意せずに`()`で囲んだことにしたい場合は`@`を使用する。

`(1 + 2 + 3 + 4) * 10`
`1 + 2 + 3 + 4 ]>@ * 10`

この記述であれば、`()`で囲むためにカーソルを戻す必要がなくなる。
読みやすさを考慮する場合は、適宜改行などを行うと良い。

`((1 + 2 + 3 + 4) * 10).toString();`
```
1 + 2 + 3 + 4
]> @ * 10
]> @.toString();
```

一時変数`@`に上の行の式を代入したと考えると理解しやすくなる。
ただし、`@`は変数ではなく`()`で囲んだことにしているだけなので、二度目以降に使用することはできない(`]>`の直後に使用しなかった場合は`this`と同じなる)

この書き方は一見長く見えるが、慣れればタイプ数の割には、早くコードを書くことができると思う。

## 逆代入式(メンバ変数への代入)

メンバ変数に代入したい場合は `]>>`を使用する。
通常の逆代入演算子 `]>` を使用すると、メンバ変数への代入はできない。
例えば次のようなコードを書いた場合。
`'a,b,c'.split(',') ]> obj.values;`

JavaScriptにコンパイルされると次のようになる。
`(obj = 'a,b,c'.split(',')).values;`

`obj.values`に対して代入を行いたい場合は、`]>>`という演算子を使用する。

`'a,b,c'.split(',') ]>> obj.values;`
こう書くことで
`obj.values = 'a,b,c'.split(',');`
というふうにコンパイルされる。


## 逆代入付初期化

逆代入式を使用するときに、変数宣言を同時に行うことができる。
構文は逆代入式に`var `をつけるだけ。
逆代入の対象となるのは、 `]>`を使用したもので `]>@`ではないものである。
定義済みの変数を使用する場合は メンバ変数の代入でなくても `]>>`を使用することで、宣言をしない。

例

```js

var a = 10,
b = a * 20,
c = b * 30;
obj.value = a * b * c;

```

これを書き直すと

```js
var 10
]> a * 20
]> b * 30
]> c * b * a
]>> obj.value;
```

となる。

実際にこの書き方をした場合は、次のようにコンパイルされる。

```js

var a, b, c;
obj.value = (c = (b = (a = 10) * 20) * 30) * b * a;

```

この書き方が有用になるときは、式の途中で前回の計算結果を連続的に評価する場合である。


## issue

結構いい感じになってきたので、Parserを自前で実装したい.

