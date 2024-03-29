# SQL Server における権限について（ユーザ、ロール、スキーマ）

## ログインユーザとデータベースユーザ】

SQL Server にはログインユーザとデータベースユーザがあり、それらは異なる概念である。
ログインユーザはサーバにアクセスするために必要で、データベースユーザは特定のデータベースにアクセスするために必要。

### ログインユーザとデータベースユーザとのマッピング情報

確認するには、以下の手順を実行します。

- SSMS（SQL Server Management Studio）を起動します。
- ログインユーザのプロパティを開きます。
- 左側のメニューから「ユーザー マッピング」を選択します。
- 「ユーザー マッピング」ページで、ログインユーザがアクセス可能なデータベースと、そのデータベースでマッピングされたデータベースユーザを確認できます。

また、以下のクエリを使用して、ログインユーザとデータベースユーザのマッピング情報を取得することもできます。

```sql
SELECT name as LoginName, dp.name as DBUserName, default_schema_name
FROM sys.database_principals AS dp
JOIN sys.server_principals AS sp ON sp.sid = dp.sid
JOIN sys.database_role_members AS drm ON drm.member_principal_id = dp.principal_id
JOIN sys.database_principals AS dpr ON dpr.principal_id = drm.role_principal_id
WHERE sp.name = 'login_name'
```

## ロールについて

SQL Server には、

- 固定サーバロール
- 固定データベースロール
- ユーザ定義ロール

の 3 つの種類のロールがある。
ロールは、ユーザに権限を付与するために使用される。

## スキーマについて

スキーマは、データベース内のオブジェクトの所有者であり、名前空間を提供する。
スキーマを使用すると、オブジェクト名の衝突を回避できる。

## ユーザとロール、およびスキーマの関係について

ユーザはデータベースにログインするために必要であり、それぞれのユーザは特定のスキーマを所有することができる。
ユーザは、ロールに追加されることができ、ロールは、複数のユーザに権限を付与するために使用される。
ロールには権限を付与するために使用されることが多く、スキーマはオブジェクトの所有者を識別するために使用される。

## ChatGPT との会話

まきはら
sqlserver のスキーマとユーザの違いについて教えてください。

> SQL Server において、スキーマとは、データベース内のオブジェクト（テーブル、ビュー、ストアドプロシージャなど）を論理的にグループ化するためのコンテナであり、スキーマには異なるユーザーがアクセスすることができます。
> 一方、ユーザーは、データベースにアクセスするための認証情報を表し、各ユーザーはデータベース内のオブジェクトにアクセスするための権限を持っています。
> つまり、スキーマはオブジェクトをグループ化するためのコンテナであり、ユーザーはオブジェクトにアクセスするための認証情報であるという違いがあります。ただし、SQL Server では、スキーマとユーザーは密接に関連しており、スキーマは通常、特定のユーザーが所有していることが多いです。

まきはら
具体例を使って sqlserver のスキーマについて教えてください。また dbo というキーワードを用いて説明してください。

> 例えば、以下のようなスキーマを考えてみましょう。

```sql
CREATE SCHEMA sales AUTHORIZATION dbo
```

> このスキーマは "sales" という名前で作成され、認証情報として "dbo" が設定されています。ここで "dbo" は、"Database Owner" の略で、データベースを作成したユーザーを表します。"dbo" は、データベース内のすべてのオブジェクトに対して、デフォルトの所有者として設定されます。
> このスキーマには、"sales" というテーブルが含まれるとします。この場合、テーブルの完全な名前は "sales.sales" となります。つまり、"sales" スキーマ内の "sales" テーブルを表します。また、別のユーザーが "sales" スキーマ内のテーブルにアクセスするには、そのユーザーに "sales" スキーマへのアクセス権限を付与する必要があります。
> 例えば、別のユーザーに "sales" スキーマへのアクセス権限を付与するには、次のようにします。

```sql
GRANT SELECT ON SCHEMA::sales TO otheruser
```

> このコマンドは、"sales" スキーマ内のすべてのオブジェクトに対する SELECT 権限を、"otheruser" という名前のユーザーに付与します。これにより、"otheruser" は "sales.sales" テーブルなど、"sales" スキーマ内のすべてのオブジェクトにアクセスできるようになります。

まきはら
ではあるユーザが特定のテーブルにアクセスするには必ず何らかのスキーマを利用してアクセスする必要があるということですか？

> はい、SQL Server において、特定のテーブルにアクセスするためには、そのテーブルが属するスキーマに対するアクセス権限が必要です。つまり、ユーザーはスキーマ単位でアクセス権限を付与されるため、そのスキーマ内のすべてのオブジェクトにアクセスすることができます。
> ただし、デフォルトのスキーマが設定されている場合、ユーザーはそのスキーマ内のオブジェクトに対するアクセス権限を自動的に持っています。デフォルトのスキーマは、ユーザーが作成されたときに設定され、後で変更することができます。
> 例えば、以下のようにして、"user1" という名前のユーザーを作成し、そのデフォルトのスキーマを "sales" に設定することができます。

```sql
CREATE USER user1 WITH DEFAULT_SCHEMA = sales
```

> この場合、"user1" は "sales" スキーマ内のすべてのオブジェクトに対するアクセス権限を持っています。ただし、他のスキーマ内のオブジェクトにアクセスするためには、そのスキーマに対する明示的なアクセス権限が必要です。

まきはら
つまり、１ユーザに対して複数のスキーマを設定でき、また、１スキーマに対して複数のテーブルを設定できるということであっていますか？

> はい、SQL Server では、1 つのユーザーに対して複数のスキーマを設定することができます。また、1 つのスキーマに複数のテーブルを含めることができます。
> 複数のスキーマを設定することで、それぞれのスキーマに異なるアクセス権限を付与することができます。例えば、特定のグループのユーザーには "sales" スキーマへの読み取り権限を与え、管理者には "sales" スキーマのすべての権限を与えることができます。
> また、1 つのスキーマに複数のテーブルを含めることができるため、関連するテーブルをグループ化して管理することができます。例えば、"sales" スキーマ内には "customers"、"orders"、"order_items" などのテーブルを含めることができます。

まきはら
スキーマは具体的にはどんな要素から構成されますか？テーブルとそれへのアクセス権限の種類だけですか？他にも構成要素はありますか？

> SQL Server において、スキーマは主にデータベース内のオブジェクト（テーブル、ビュー、ストアドプロシージャ、関数など）の名前空間として機能します。スキーマは、これらのオブジェクトに対するアクセス権限を管理するための単位として機能します。
> スキーマに含まれる要素は、以下の通りです。
> テーブル、ビュー、ストアドプロシージャ、関数などのオブジェクト：スキーマには、さまざまな種類のオブジェクトを含めることができます。これらのオブジェクトは、スキーマに属するため、スキーマ名を接頭辞として指定することで参照されます。
> アクセス権限：スキーマは、含まれるオブジェクトに対するアクセス権限を管理するための単位として機能します。データベース管理者は、スキーマ単位でユーザーに対するアクセス権限を設定することができます。

ユーザー、ロール、スキーマ所有者：スキーマは、それを所有するユーザーやロールを定義することができます。スキーマ所有者は、スキーマ内のすべてのオブジェクトに対する完全なアクセス権限を持ちます。

したがって、スキーマは単にテーブルとそれへのアクセス権限だけでなく、データベース内のオブジェクトを名前空間として構成し、それらに対するアクセス権限を管理するための単位として機能します。

まきはら
ありがとうございます。スキーマについてよくわかりました。
デフォルトのスキーマは何かありますか？たしか dbo など目にしたことがあります。
はい、SQL Server にはデフォルトのスキーマがあります。SQL Server において、新しいユーザーを作成すると、データベース内に自動的に "dbo" という名前のスキーマが作成されます。"dbo" は "database owner" を意味し、デフォルトのスキーマとして使用されることが多いです。

"dbo" スキーマには、データベースの管理と管理タスクを実行するためのストアドプロシージャやシステムオブジェクトなど、データベースの基本的なオブジェクトが含まれます。また、新しいテーブルを作成するときに、テーブルが明示的に作成されるスキーマを指定しない場合、自動的に "dbo" スキーマが使用されます。

ただし、デフォルトのスキーマは変更することができます。たとえば、あるユーザーが "my_schema" という名前のスキーマに対する読み取り書き込みアクセス権限を持つ場合、そのユーザーが新しいテーブルを作成する場合には、自動的に "my_schema" スキーマが使用されます。

まきはら
デフォルトスキーマには guest, sys, INFORMATION_SCHEMA などもあるようですがこれらは何ですか？
"guest"、"sys"、"INFORMATION_SCHEMA" などのスキーマは、SQL Server によって作成されたデフォルトのスキーマです。

"guest" スキーマは、ゲストユーザーによる一時オブジェクトの作成や、ユーザーがアクセスできる一時テーブルなどを含むスキーマです。

"sys" スキーマには、SQL Server システムの基本的なオブジェクトが含まれています。たとえば、"sys.objects" ビューは、データベース内のすべてのオブジェクトのメタデータを格納するために使用されます。

"INFORMATION_SCHEMA" スキーマは、データベースに関する情報を提供するための規定のスキーマです。このスキーマには、データベースのテーブル、ビュー、列、インデックス、制約などに関する情報を取得するためのビューが含まれています。たとえば、"INFORMATION_SCHEMA.TABLES" ビューは、データベース内のすべてのテーブルに関する情報を提供します。

これらのスキーマは、ユーザーによって作成されたスキーマと同様に、アクセス権限の設定などを変更することができます。ただし、これらのスキーマには、SQL Server システムとの互換性を保つために、変更を加えない方が望ましい場合があります。

まきはら
ゲストユーザーとは何ですか？
ゲストユーザーは、データベースにログインしたが、データベース内にユーザーアカウントが作成されていないユーザーを指します。たとえば、データベースにアクセスするために、Windows アカウントでログインする場合、そのユーザーに対応するデータベース内のユーザーアカウントが存在しない場合、そのユーザーはゲストユーザーとして扱われます。ゲストユーザーは、一時オブジェクトを作成したり、一時テーブルにアクセスしたりするために使用されることがあります。ただし、一般的には、セキュリティ上の理由から、ゲストユーザーアカウントは削除されることが推奨されています。

まきはら
ユーザー、ロール、スキーマ所有者について教えてください。
SQL Server において、ユーザー、ロール、スキーマ所有者は、データベースに対するアクセス制御を実現するための重要な役割を担っています。

ユーザーは、データベースにアクセスするために必要な認証情報を持つアカウントです。データベース内のユーザーは、それぞれに固有のセキュリティ識別子（SID）を持ち、データベース内で一意に識別されます。ユーザーは、データベースオブジェクトへのアクセス許可が与えられます。

ロールは、複数のユーザーにアクセス許可を一括して付与するための仕組みです。ロールは、一般的にはアプリケーションやデータベース内の業務ロールに基づいて作成され、ロールにアクセス許可を付与することで、ロールに属するすべてのユーザーにアクセス許可を付与することができます。SQL Server には、事前定義されたロール（db_owner、db_datareader、db_datawriter など）があり、カスタムロールも作成できます。

スキーマ所有者は、スキーマ内のオブジェクトに対するアクセス許可を管理するための役割を持ちます。スキーマ所有者は、スキーマを作成したユーザーにデフォルトで割り当てられますが、スキーマ所有者を他のユーザーやロールに変更することもできます。スキーマ所有者は、そのスキーマ内のすべてのオブジェクトに対するアクセス許可を持ちます。

これらの役割は、データベースに対するアクセス制御を簡単かつ柔軟に実現するために重要です。適切に構成することで、セキュリティを強化し、データの保護を確保することができます。

まきはら
ユーザの種類にはどのようなものがありますか？デフォルトのユーザや特別なユーザなどはありますか？あと、 sa とは何ですか？
SQL Server には、様々な種類のユーザーが存在します。主なユーザーの種類は以下の通りです。

Windows ユーザー - Windows ドメインまたはローカルコンピュータのユーザーアカウントを使用して認証されます。

SQL Server 認証ユーザー - ユーザー名とパスワードの組み合わせを使用して認証されます。この種類のユーザーは、SQL Server が提供する認証方法の一つです。

Guest ユーザー - ゲストユーザーは、SQL Server のセキュリティモデルで特別な位置付けを持ちます。ゲストユーザーは、一般的にデータベースへの接続を試みるすべてのユーザーに割り当てられますが、デフォルトでは、ほとんどの権限が削除されています。

Public ユーザー - パブリックユーザーは、データベースにアクセスできるすべてのユーザーに割り当てられます。パブリックユーザーには、権限がなく、パブリックユーザーに付与された権限は、すべてのユーザーに適用されます。

SQL Server には、デフォルトのユーザーと特別なユーザーがいくつか存在します。たとえば、dbo（データベース所有者）ユーザーは、データベースを作成したユーザーによって所有され、そのユーザーが削除された場合は、別のユーザーによって取り込まれます。また、sa（システム管理者）ユーザーは、SQL Server の最高管理者アカウントであり、SQL Server のすべての権限を持つユーザーです。sa ユーザーは、SQL Server がインストールされたときに自動的に作成され、最初の SQL Server 認証ユーザーとして割り当てられます。

まきはら
ユーザ名とスキーマ名は同じものに合わせた方がいいですか？
ユーザー名とスキーマ名は、必ずしも同じ名前である必要はありませんが、同じ名前にすることが一般的に推奨されます。なぜなら、ユーザーが所有するスキーマと同じ名前を持つユーザーを作成することで、特定のテーブルやビューにアクセスする際に、スキーマ名を省略できるため、SQL コードが簡潔になります。

ただし、ユーザー名とスキーマ名が異なる場合は、必要な権限を設定することで、ユーザーが他のスキーマのオブジェクトにアクセスできるようにすることができます。しかし、スキーマ名を省略できないため、SQL コードがやや複雑になる場合があります。

まきはら
sql server にはログインユーザとデータベースを利用するユーザの２種類のユーザがあるのですか？またそれらの関係を教えてください。
はい、SQL Server にはログインとデータベースユーザーの 2 つの種類のユーザーがあります。

ログインユーザーは SQL Server のインスタンスに接続するために必要なユーザーです。ログインユーザーは SQL Server のインスタンスにアクセスするための認証情報を提供し、SQL Server に対するアクセス権限を付与されます。ログインユーザーは、Windows 認証を使用する Windows アカウントまたは SQL Server 認証を使用する SQL Server アカウントのいずれかである場合があります。

データベースユーザーは、データベース内でのアクセス権限を持つユーザーです。データベースユーザーは、データベース内に存在するスキーマ、テーブル、ビューなどのオブジェクトに対するアクセス権限を持っています。データベースユーザーは、ログインユーザーにマッピングされており、ログインユーザーのアクセス権限に基づいて、データベース内のオブジェクトに対するアクセス権限を付与されます。つまり、ログインユーザーがデータベースに接続する際に、対応するデータベースユーザーが自動的に作成されます。

ログインユーザーとデータベースユーザーは 1 対 1 で対応するわけではなく、複数のログインユーザーを 1 つのデータベースユーザーにマッピングすることもできます。また、ログインユーザーに対してデータベースユーザーを作成することができますが、ログインユーザーが存在しなくてもデータベースユーザーを作成することができます。ただし、ログインユーザーが存在しない場合、データベースユーザーはデータベース内のオブジェクトに対してアクセスするための認証情報を持たないため、アプリケーションからアクセスすることはできません。

まきはら
１つのログインユーザに対して複数のデータベースユーザをマッピングすることも可能ですか？
はい、1 つのログインユーザに対して複数のデータベースユーザをマッピングすることができます。ただし、ログインユーザにはデータベースごとに異なるデータベースユーザを割り当てることができます。データベースユーザーは、データベースごとに定義されたデータベースセキュリティ主体であり、ログインユーザーとデータベースユーザーの間のマッピングは、ユーザーマッピングを使用して設定できます。

まきはら
ログインユーザとデータベースユーザとのマッピング情報はどうやって確認をすればいいですか？
ログインユーザとデータベースユーザとのマッピング情報を確認するには、以下の手順を実行します。

SSMS（SQL Server Management Studio）を起動します。
ログインユーザのプロパティを開きます。
左側のメニューから「ユーザー マッピング」を選択します。
「ユーザー マッピング」ページで、ログインユーザがアクセス可能なデータベースと、そのデータベースでマッピングされたデータベースユーザを確認できます。
また、以下のクエリを使用して、ログインユーザとデータベースユーザのマッピング情報を取得することもできます。

vbnet
Copy code
SELECT name as LoginName, dp.name as DBUserName, default_schema_name
FROM sys.database_principals AS dp
JOIN sys.server_principals AS sp ON sp.sid = dp.sid
JOIN sys.database_role_members AS drm ON drm.member_principal_id = dp.principal_id
JOIN sys.database_principals AS dpr ON dpr.principal_id = drm.role_principal_id
WHERE sp.name = 'login_name'
ただし、クエリを実行するには、ログイン名を 'login_name' に置き換える必要があります。
