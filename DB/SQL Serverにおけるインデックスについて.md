# SQL Server におけるインデックスについて

## 参考

- [SQL インデックスの使い方](https://zenn.dev/tsubasa28/articles/075f44c2fa56927be436)
- [SQLServer【Index 編】#S2*05*検索条件ありで検索](https://anderson02.com/sqlserver/sqlserver-index/sqlserver-index-2-05/)
- [SQL Server のインデックスの理解を深める](https://qiita.com/fuk101/items/2e6a225a97a14f0f2850)
- [SQL Server のインデックスを作成して検索を高速化する](https://tech-blog.cloud-config.jp/2022-07-11-sql-server-index/)
- [SQLServer のインデックスについてざっくりとまとめてみた](https://qiita.com/kz_morita/items/41291516ff3ee2650554)

## 概要

- 結論：
  - SQL Server の場合は以下の場合分けをしてインデックスを張る
    - PK カラムに対して`クラスタ化インデックス`
    - それ以外のカラムに対して`非クラスタ化インデックス`
- インデックスのメリット、デメリット
  - メリット
    - 検索、ソート、結合で多用されるカラムに対して、インデックスを作成するとパフォーマンスが向上する。
  - デメリット
    - データを追加する処理が重くなる
      - テーブルととは別の領域にデータを保持するため、テーブルにデータを追加するとインデックスの方にもデータが追加されます。また、ソートなどを行なっている場合は、データを追加するごとにソートも再度実行されます。なので、結果としてデータを追加する処理が重くなってしまいます。
- どんなときに使うのか？
  - インデックスの効果が高いカラム
    - 検索
      - WHERE 句で多用するカラム
    - ソート
      - ORDER BY 句で多用するカラム
    - 結合
      - JOIN の結合条件で多用するカラム
  - インデックスを作成するべきではないパターン
    - 下記のようなパターンでインデックスを作成すると、インデックスのデメリットの影響でパフォーマンスが低下してしまいます。
      - 格納されているデータが少ないテーブル
      - 格納されるデータの種類が少ないカラム
      - 検索、ソート、結合があまり行われないカラム

## 操作編

### キャッシュクリア

```sql
DBCC DROPCLEANBUFFERS
DBCC FREEPROCCACHE
select * from Shain
```

### 実行プラン

- ![](https://anderson02.com/wp-content/uploads/2020/08/1-24.png)
- ![](https://anderson02.com/wp-content/uploads/2020/08/1-25.png)
- テーブルスキャン
  - テーブルスキャンとは，「テーブルをすべて読みました」という意味
  - テーブルスキャンの場合、条件指定で 1 レコードだけ読み込む場合も全レコードを読み込むので遅い
  - ![](https://anderson02.com/wp-content/uploads/2020/08/1-26.png)
- インデックスシーク
  - 「Seek」とは「ピンポイントでデータを見つけました」という意味
  - ![](https://anderson02.com/wp-content/uploads/2020/08/1-37.png)
