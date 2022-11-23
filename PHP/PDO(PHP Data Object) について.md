# PDO(PHP Data Object) について

## 参考

- [PHP でデータベースに接続するときのまとめ](https://qiita.com/mpyw/items/b00b72c5c95aac573b71)

## 基本

### データベースに接続

`PDO::__construct` メソッドを使用してインスタンスを生成します．コンストラクタなので，実際には直接これを呼び出すコードは書かずに， new 演算子 を用います．

```php
$pdo = new PDO($dsn, $username, $password, $driver_options);
```

引数は以下の通りです．

#### `$dsn`

データベースに接続するために必要な情報です． (Data Source Name)
以下に各データベース製品に応じた DSN の書き方が掲載されています．

MySQL の基本的な書き方を例に挙げます．

```php
mysql:dbname=test;host=localhost;charset=utf8mb4
```

頭にデータベースの種類を指定して `:` で区切る．
各項目は `項目名=値` とし， `;` で区切る．

##### dbname

データベース名を指定します．基本的には必須です．但し，データベースを後で USE test のように SQL 文で選択する場合，省略することができます．

##### host

ホスト名または IP アドレスを指定します．ローカル環境で動かす場合，省略しても問題ない場合が多いです．localhost は自分自身のホスト名， 127.0.0.1 は自分自身の IP アドレスを指します．どちらを用いても問題が発生する可能性があるので，適宜問題のない方を選択してください．

- Linux 環境はホスト名推奨
  [upsilon さん: MySQL では localhost と 127.0.0.1 で接続方法が変わるため注意が必要](http://qiita.com/mpyw/items/b00b72c5c95aac573b71#comment-e9db50fff9bffa1dd6f8)
- Windows 環境は IP アドレス推奨
  [AH-2 - Windows で localhost への接続が遅い(解決方法)](http://www.ah-2.com/2012/01/28/win_localhost_slow.html)

##### charset

文字セットを指定します．SET NAMES は避けて，ここで指定するべきです．UTF-8 ではなく utf8 であることに注意してください．ハイフンは入りません．

> 【2016/12/17 追記】
> MySQL5.5.3 以降の場合は，4 バイトからなる絵文字なども正常に取り扱える utf8mb4 を使用することを強く推奨します。

#### `$username`

ユーザー名．ルート権限を使う場合，デフォルトでは root です．

#### `$password`

パスワード．ルート権限を使う場合，デフォルトでは空白です．

#### `$driver_options`

接続時のオプションを連想配列で渡します．

キーはあらかじめ用意されている定数を取る．
値はあらかじめ用意されている定数以外に，論理値・文字列などの一般的な値も取り得る．

```php
[
  PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
  PDO::ATTR_EMULATE_PREPARES => false,
]
```

### 接続オプション

`PDO::setAttribute` メソッドを使用します．多くのオプションはコンストラクタの `$driver_options` で指定してもこちらで指定しても大差はありませんが， `PDO::MYSQL_ATTR_INIT_COMMAND`, `PDO::ATTR_PERSISTENT` など，一部コンストラクタ専用のものがあります．

```php
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

よく使われるドライバオプションとその値

#### PDO::ATTR_ERRMODE

SQL 実行でエラーが起こった際にどう処理するかを指定します．デフォルトは PDO::ERRMODE_SILENT です．

#### PDO::ERRMODE_EXCEPTION

を設定すると例外をスローしてくれる．これを選択しておくのが一番無難．

##### PDO::ERRMODE_WARNING

は SQL で発生したエラーを PHP の Warning として報告する．

##### PDOStatement::execute メソッドの返り値が false でないかを毎回のように確認する必要がある．

PDO::ERRMODE_SILENT は何も報告しない．
PDOStatement::execute メソッドの返り値が false でないかを毎回のように確認する必要がある．
PDO::ATTR_DEFAULT_FETCH_MODE
PDOStatement::fetch メソッドや PDOStatement::fetchAll メソッドで引数が省略された場合や，ステートメントが foreach 文に直接かけられた場合のフェッチスタイルを設定します．デフォルトは PDO::FETCH_BOTH です．

PDO::FETCH_BOTH
カラム番号とカラム名の両方をキーとする連想配列で取得する．
PDO::FETCH_NUM
カラム番号をキーとする配列で取得する．
PDO::FETCH_ASSOC
カラム名をキーとする連想配列で取得する．これが一番ポピュラーな設定．
PDO::FETCH_OBJ
カラム名をプロパティとする基本オブジェクトで取得する．
PDO::ATTR_EMULATE_PREPARES
データベース側が持つ「プリペアドステートメント」という機能のエミュレーションを PDO 側で行うかどうかを設定します．

PHP5.1 のデフォルトは false
PHP5.2 以降のデフォルトは true
この設定で，いくつか PDO の挙動に違いが表れます．

プリペアドステートメントのためにデータベースサーバと通信する必要が無くなるため，エミュレーションを行ったほうがパフォーマンスは向上する．
存在しないテーブル名やカラム名を SQL 文に持つプリペアドステートメントを発行したとき，エミュレーション OFF の場合はすぐにエラーが発生するが，エミュレーション ON の場合は実際にクエリを実行するまでエラーが発生するかどうかわからない．
エミュレーションが ON の場合のみ， ; 区切りで複数の SQL 文を 1 つのクエリで実行することができる．
その他，どちらにも利点と欠点があるので，違いは追って見ていき，最後に表にまとめることにします．

PDO クラスのエミュレーションを無効にする
$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
PDO::ATTR_PERSISTENT (コンストラクタ専用)
true を設定すると，スクリプトが終了してもデータベースへの接続を維持し，次回に再利用します．規模が大きくなってくると設定する恩恵が大きくなりますが，ほそぼそと練習用サイトを作っているうちは役に立たないでしょう．

PDO::MYSQL_ATTR_USE_BUFFERED_QUERY (MySQL 専用)
true のとき， バッファクエリを使用します．デフォルト値はバージョンによって異なるようで，不明です．PHP5.3 の時代には既にデフォルトが true で今も変わっていない…？少なくとも，私が確認できる限りの最近バージョンではデフォルトは true です．

バッファクエリ:
全ての情報をデータベースサーバから取得してきておいて，PHP に 1 件ずつ取り出させる
非バッファクエリ:
1 件ごとにデータベースサーバと通信を行って，PHP に取り出させる
取得してくる情報がメモリに収まりきらない莫大な量である，といった非常に特殊なケースを除けば，バッファクエリを選択しておく方が無難です．サーバ負荷も軽減され，途中までフェッチしたところで突然例外が発生するような事態も避けられます．また，非バッファクエリには複数同時にクエリを実行できないなどの大きな欠点もあります．基本的にこれは，データベースから取得したデータで HTML を表示する用途ではなく，コマンドラインからバッチ処理を実行する用途で使われます．

PDO::MYSQL_ATTR_INIT_COMMAND (MySQL 専用) (コンストラクタ専用)
接続した直後に実行されるクエリをここに書きます．

## 基本コーディング

書き方のテンプレです．

```php
<?php

try {

    /* リクエストから得たスーパーグローバル変数をチェックするなどの処理 */

    // データベースに接続
    $pdo = new PDO(
        'mysql:dbname=testdb;host=localhost;charset=utf8mb4',
        'root',
        '',
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]
    );

    /* データベースから値を取ってきたり， データを挿入したりする処理 */

} catch (PDOException $e) {

    // エラーが発生した場合は「500 Internal Server Error」でテキストとして表示して終了する
    // - もし手抜きしたくない場合は普通にHTMLの表示を継続する
    // - ここではエラー内容を表示しているが， 実際の商用環境ではログファイルに記録して， Webブラウザには出さないほうが望ましい
    header('Content-Type: text/plain; charset=UTF-8', true, 500);
    exit($e->getMessage());

}

// Webブラウザにこれから表示するものがUTF-8で書かれたHTMLであることを伝える
// (これか <meta charset="utf-8"> の最低限どちらか1つがあればいい． 両方あっても良い．)
header('Content-Type: text/html; charset=utf-8');

?>
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>Example</title>
    </head>
    <body>
        <!-- ここではHTMLを書く以外のことは一切しない -->
    </body>
</html>
```

## クエリ

### `PDO::query` メソッドで直接クエリを実行する

ユーザー入力を伴わないクエリに関しては単に `PDO::query` メソッドを実行すればいいだけです．返り値は `PDOStatement` となります．

全員を取得する

```php
$stmt = $pdo->query('SELECT \* FROM users');
```

### `PDO::exec` メソッドで直接クエリを実行する

ユーザー入力を伴わないクエリで， INSERT や UPDATE 等で作用した件数を直接返り値に欲しい場合は `PDO::exec` メソッドを代わりに使います．特に結果を必要としない場合においてもこちらを使用すべきです．後に登場する `PDOStatement::execute` と紛らわしいので注意してください．

全員の年齢を+1 し，その対象となった人数を返り値として取得する

```php
$count = $pdo->exec('UPDATE users SET age = age + 1');
```

### `PDO::prepare` → `PDOStatement::bindValue` → `PDOStatement::execute` の 3 ステップでクエリを実行する

ユーザー入力を受け取って SQL 文を動的に生成する場合は プリペアドステートメント と プレースホルダ を使わなければなりません．

#### プレースホルダ:

直訳すると「場所取り」．何かユーザ入力を当てはめる場所としてあらかじめ確保しておくもの．

#### プリペアドステートメント:

直訳すると「予約文」．文を予約したもの．通常，「予約文」は「場所取り」を使うために作られる．もし「場所取り」が無ければ普通に `PDO::query` などで実行するだけで十分なためである．
プレースホルダには 2 種類あり，**疑問符プレースホルダ** を使う方法と， **名前付きプレースホルダ** を使う方法があります．もしこれらが混ざってしまうと

```
SQLSTATE[HY093]: Invalid parameter number: mixed named and positional parameters
```

が発生するので，どちらか一方のみを選択してください．

##### 疑問符プレースホルダ

- `?` の「番目」は 1 から始まる．
- `PDO::PARAM_STR` は省略することが出来る．
- エミュレーションが ON の場合には正しくキャストしてくれないバグのようなものが仕様として存在するため，文字列以外のものを扱う際に明示的なキャストが必要．
- `NULL` 値に関しては `PDO::PARAM_NULL` が暗黙的に使用される．

エミュレーションが ON の場合はこうする必要がある

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE gender = ? AND age = ?');
$stmt->bindValue(1, $gender);
$stmt->bindValue(2, (int)$age, PDO::PARAM_INT);
$stmt->execute();
```

エミュレーションが OFF の場合はこれでも OK

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE gender = ? AND age = ?');
$stmt->bindValue(1, $gender);
$stmt->bindValue(2, $age, PDO::PARAM_INT);
$stmt->execute();
```

##### 名前付きプレースホルダ

- `:` を頭につけ，半角英数字とアンダースコアにて構成する．
- バインド時の頭の `:` は省略することが出来る．

こちらも同様にエミュレーションが ON ならば明示的なキャストを忘れない

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE age = :age AND gender = :gender');
$stmt->bindValue(':age', (int)$age, PDO::PARAM_INT);
$stmt->bindValue(':gender', $gender);
```

頭のコロンを省略したケース

```php
$stmt->bindValue('gender', $gender);
```

- エミュレーションが ON の場合のみ，同名のプレースホルダを複数使うことが出来る．

ID が 20 で年齢も 20 歳の人を取得

```php
$n = 20;
$stmt = $pdo->prepare('SELECT * FROM users WHERE age = :n AND id = :n');
$stmt->bindValue(':n', (int)$age, PDO::PARAM_INT);
$stmt->execute();
```

なお， 値を即時にバインドするのではなく，変数を参照的にバインドしておき，実際に値をバインドするのは実行時になる PDOStatement::bindParam メソッドもありますが，原則的にこちらを使う必要はありません．エミュレーションが ON の場合，実行後にバインドした変数が文字列型にされる仕様もあるので注意してください．

### `PDO::prepare` → `PDOStatement::execute` の 2 ステップでクエリを実行する

`PDOStatement::execute` メソッドの引数に配列を渡すと，それらを全てバインドしたあとそのまま SQL を実行してくれます．但し，以下の条件に注意してください．

- NULL 値以外は全て PDO::PARAM_STR 扱いになる ．もし間違った型でバインドをしても，MySQL/SQLite はデータベース側で自動的にキャストし直してくれるが，パフォーマンスの低下やバグの原因になるので，可能な限り避けたほうがいい．PostgreSQL の場合はエラーになる．
- また， 既に PDOStatement::bindValue メソッドで値がバインドされていた場合でも，それらは全て無視される．これを用いる場合，全てのバインドをこの引数で行わなければならない．

#### 疑問符プレースホルダ

`PDOStatement::bindValue` メソッドを用いたときと異なり，? の「番目」が 0 から始まることに注意してください．

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE city = ? AND gender = ?');
$stmt->execute([$city, $gender]);
```

キーを適切に設定すれば順番を変えて指定することも可能

```php
$stmt->execute([1 => $gender, 0 => $city]);
```

#### 名前付きプレースホルダ

```php
$stmt = $pdo->prepare('SELECT * FROM users WHERE city = :city AND gender = :gender');
$stmt->execute([':city' => $city, ':gender' => $gender]);
```

頭のコロンは省略することが出来る

```php
$stmt->execute(['city' => $city, 'gender' => $gender]);
```

compact 関数を活用する例

```php
$stmt->execute(compact('city', 'gender'));
```

## 結果を取得する

### `PDOStatement::fetch`

カーソルをずらしながら，指定したフェッチモードで 1 行ずつ取得していきます．

引数を省略した場合はデフォルトフェッチモードが使用される．
全ての取得が終わると常に false を返す．
デフォルトフェッチモードを PDO::FETCH_ASSOC に設定した際の例を示します．

一番基本的な方法

```php
while ($row = $stmt->fetch()) {
    printf("%s lives in %s<br />\n", $row['name'], $row['city']);
}
```

`vprintf()`関数を使った応用

```php
while ($row = $stmt->fetch()) {
    vprintf("%s lives in %s<br />\n", $row);
}
```

`PDOStatement` クラスは `Traversable` インターフェースを実装しているため，デフォルトフェッチモードを使う場合，while 文の代わりに `foreach` 文でもっとスマートに書くことができます．但し，HTML 表示のために変数を準備する場合には，多くは配列として持っておく方が都合がいいので，後に紹介する `PDOStatement::fetchAll` メソッドの使用を検討してください．

```php
foreach ($stmt as $row) {
    printf("%s lives in %s<br />\n", $row['name'], $row['city']);
}
```

0 から始まるオフセットを取り出すことも出来る

```php
foreach ($stmt as $i => $row) {
  printf("[%d] %s lives in %s<br />\n", $i, $row['name'], $row['city']);
}
```

### `PDOStatement::fetchObject`

連想配列の代わりに stdClass オブジェクトとして取得します．PDOStatement::fetch で PDO::FETCH_OBJ を指定するケースと等価ですが，こちらの方が短く書くことができます．

```php
while ($row = $stmt->fetchObject()) {
  printf("%s lives in %s<br />\n", $row->name, $row->city);
}
```

### `PDOStatement::fetchColumn`

特定の 1 カラムのみを文字列として取得します．PDOStatement::fetch で PDO::FETCH_COLUMN を指定するケースと等価ですが，こちらの方が短く書くことができます．

先頭から数えてそのカラムが何番目にあるかを第 1 引数として渡す．「番目」は 0 から始まる．省略した場合は 0 を指定したとみなされる．
値に 0 が含まれる可能性がある場合は， false !== の条件判定をしなければならない．

```php
while (false !== $value = $stmt->fetchColumn()) {
    echo "{$value}<br />\n";
}
```

### `PDOStatement::fetchAll`

一気に全件取得して 2 次元配列とします．

引数を省略した場合はデフォルトフェッチモードが使用される．

```php
$rows = $stmt->fetchAll();
var_dump($rows);
```

特定のカラムだけ一気に全件取得して 1 次元配列としたい場合，第 1 引数に PDO::FETCH_COLUMN を指定し，第 2 引数にカラムの「番目」を渡す．「番目」は 0 から始まる．省略した場合は 0 を指定したとみなされる．
$values = $stmt->fetchAll(PDO::FETCH_COLUMN);
var_dump($values);

### `PDOStatement::setFetchMode`

PDO オブジェクト自体にデフォルトフェッチモードを指定する方法を紹介しましたが，このメソッドを利用すれば個別に発行された PDOStatement オブジェクトに対して後からフェッチモードを指定することができます．引数の渡し方がモードによって異なるので，詳しくはマニュアルを参照してください．

0 番目のカラムを foreach で取得していくケース

```php
$stmt->setFetchMode(PDO::FETCH_COLUMN, 0);
foreach ($stmt as $i => $name) { ... }
```

実は… PDO::query を用いる場合にも同じ形式でフェッチモードを指定することができます．

0 番目のカラムを foreach で取得していくケース

```php
foreach ($pdo->query($sql, PDO::FETCH_COLUMN, 0) as $i => $name) { ... }
```

### `UPDATE`, `INSERT` で作用した行数を取得する

こちらの結果に対してフェッチしようとした場合，SQLSTATE[HY000]: General error が発生します．こちらに対して用いることができるのは PDOStatement::rowCount メソッドのみです．

```php
printf("%d 行に作用しました<br >\n", $stmt->rowCount());
```

### `SELECT` で該当した行数を取得する

基本的に，以下の方法に従います．

件数だけでなく結果セットも一緒に欲しい場合， PDOStatement::fetchAll メソッドで一気に配列として取得し，それに対して PHP の count 関数を使う．
件数だけが欲しい場合は， SELECT COUNT(\*) WHERE ... といったクエリを発行し，その結果を PDOStatement::fetchColumn メソッドで得る．
これらの方法が最も推奨されますが，他の方法が無いわけではありません．以下に補足説明を示します．但しこれらは MySQL や PostgreSQL についてのみ当てはまり，SQLite には当てはまりません．

バッファクエリ使用時
SELECT に対しても常に PDOStatement::rowCount メソッドを使うことができる．
非バッファクエリ使用時
PDOStatement::fetchAll メソッドを実行した後，つまり全てのフェッチが終わった後であれば SELECT に対しても PDOStatement::rowCount メソッドを使うことができる．
結果のフェッチを途中で中断したまま次のクエリ実行に移行するときは， PDOStatement::closeCursor メソッドを使ってカーソルを閉じる必要がある．1 つ目の結果セットを PDOStatement に保持したまま 2 つ目の SQL を実行することはできないので，その場合はあらかじめ PDOStatement::fetchAll メソッドでデータを退避させておく必要がある．
データベース接続の切断
データベース処理の最後に

$pdo = null;
と書いているコードが散見されますが，ほとんどの場合ではこの記述は不要です．必要になるのは，非バッファクエリを使うようなバッチ処理のシーンのみです．

## トランザクションの基本的な利用法

使用されるメソッド
処理の実体はデータベースごとに固有の実装が為されていますが，インタフェースは PDO のメソッドとして抽象化されており，SQL 文をデータベースごとに書き分けたりする必要はなくなります．

- `PDO::beginTransaction`
- `PDO::commit`
- `PDO::rollBack`
- `PDO::lastInsertId` (必要に応じて)

### ポイント

Try ブロックを 2 重 に設け，例外発生時にはロールバックを実行し，捕捉した例外を必要に応じて外側の Try ブロックに向けてスローする．
同じ形式のプリペアドステートメントの生成は 1 回だけにして，それを使いまわすようにした方がパフォーマンスは向上する．
例

```php
try {

    // データベースに接続
    $pdo = new PDO(
        'mysql:dbname=testdb;host=localhost;charset=utf8mb4',
        'root',
        '',
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]
    );

    // パラメータ
    $from = 'A';
    $to = 'B';
    $transfer_amount = 12000;

    // プリペアドステートメントを用意
    $stmt = $pdo->prepare('UPDATE account SET credit = credit + (?) WHERE id = ?');

    // トランザクション処理を開始
    $pdo->beginTransaction();
    try {
        // Aの預金残高を減らす
        $stmt->bindValue(1, -1 * $transfer_amount, PDO::PARAM_INT);
        $stmt->bindValue(2, $from);
        $stmt->execute();
        // Bの預金残高を増やす
        $stmt->bindValue(1, +1 * $transfer_amount, PDO::PARAM_INT);
        $stmt->bindValue(2, $to);
        $stmt->execute();
        // コミット
        $pdo->commit();
    } catch (PDOException $e) {
        // ロールバック
        $pdo->rollBack();
        // 外側のTryブロックに対してスロー
        throw $e;
    }

} catch (PDOException $e) {

    // 例外メッセージを表示
    header('Content-Type: text/plain; charset=UTF-8', true, 500);
    exit($e->getMessage());

}
```
