# VBA 配列のまとめ

## 静的配列

### 宣言

`Dim 静的配列の名前(静的配列のサイズ) as データ型`

```vb
Dim sArray(3) As String
```

### 格納

`静的配列の名前(インデックス) = データ`

```vb
sArray(0) = "Monday"
sArray(1) = "Tuesday"
sArray(2) = "Wednesday"
sArray(3) = "Thursday"
```

## 動的配列

#### 宣言

`Dim 動的配列の名前() as データ型`
動的配列の生成では、動的配列の名前と格納するデータ型を指定します。なお、生成時にサイズは指定しません。

```vb
Dim dArray() As String
```

#### サイズ設定

ただし値を具体的に代入する前にサイズは決めとく必要がある

`ReDim Preserve 動的配列の名前(動的配列のサイズ)`
動的配列のサイズは何度でも再設定できます。

```vb
ReDim Preserve dArray(3)
```

#### 動的配列にデータを追加する。

`動的配列の名前(インデックス) = データ`

```vb
dArray(0) = "January"
dArray(1) = "February"
dArray(2) = "March"
dArray(3) = "April"
```

#### 注意

したがって以下のようなコードはエラーになる

```vb
Dim dArray() As String
'サイズ宣言がないと・・・
' ReDim Preserve dArray(3)

dArray(0) = "January" 'ここでエラー
dArray(1) = "February"
dArray(2) = "March"
dArray(3) = "April"
```

### LBound,UBound 関数（要素の下限/上限を取得する関数）

- 2 個目の引数は、配列が多次元だった場合のどの次元の要素数を指定するか、の指定（１次元配列なら指定しないか、1 を指定する。２次元配列の２次元目を使いたいなら 2 を指定）
- 使い方

```vb
For i = LBound(MyArray, 1) To UBound(MyArray, 1)
　　For j = LBound(MyArray, 2) To UBound(MyArray, 2)
　　　　MyArray(i, j) = 0
　　Next
Next i
```

### Array 関数

`Array(arglist)`
配列が格納されたバリアント型 (Variant) の値を返します。
指定した値は、バリアント型 (Variant) に格納されている配列の要素に代入されます。

```vb
Dim MyArray
MyArray = Array(10, 20, 30)
```

### Filter 関数

## セルの内容を配列に代入する

- 配列の型は`無指定`か`Variant`で行う.
- 配列用の`()`はあってもなくてもいい
- 以下どれでも OK

```vb
Dim dArray()
dArray = Range("A1:B100")
```

```vb
Dim dArray() As Variant
dArray = Range("A1:B100")
```

```vb
Dim dArray
dArray = Range("A1:B100")
```

```vb
Dim dArray As Variant
dArray = Range("A1:B100")
```

## Collection

### 配列との違い

#### 配列のメリット

- Array や Split などで動的に生成できる
- Join 関数で結合できること、
- 2 次元配列のセルとの相互転記ができること、
- 宣言時に型をカチっと決められるためオブジェクト型の配列にしたときにドットでプロパティとメソッドの入力候補が表示される

#### コレクションのメリット

- データの追加・削除・挿入が容易である
- キー文字列を設定でき、インデックスの変わりにキーを使ってデータ参照できる

## 連想配列（Dictionary）
