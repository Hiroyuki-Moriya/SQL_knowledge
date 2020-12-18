# SQLチートシート (第3部　/　全4部)

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 参考文献

 スッキリわかるSQL入門 第２版
 dokoQL デスクトップ版
  https://dokoQL/d/
    ID：SUKKIRI=2800YEN
    PW：（何も入力しない）

 Microsoft SQL Transact-SQL リファレンス (データベース エンジン)
  （Docs＞SQL＞リファレンス＞Transact-SQL (T-SQL) リファレンス）
  https://docs.microsoft.com/ja-jp/sql/t-sql/language-reference?view=sql-server-ver15

 Microsoft SQL データベース関数
  （Docs＞SQL＞リファレンス＞Transact-SQL (T-SQL) リファレンス＞関数）
  https://docs.microsoft.com/ja-jp/sql/t-sql/functions/functions?view=sql-server-ver15

 逆引きSQL構文集
  http://www.sql-reference.com/index.html#string

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## トランザクション

### トランザクション制御
	トランザクションとは、DBMSに対する指示で、1つ以上のSQL文をひとかたまりで扱うように指示するときの「かたまり」。
	DBMSによるトランザクション制御
	・トランザクションの途中で、処理が中断されないようにする。
	・トランザクションの途中に、ほかの人の処理が割り込めないようにする。
	・DBMSは、トランザクションに含まれるすべてのSQL文について、
		必ず「すべての実行が完了している」か
		「１つも実行されていない」のどちらかの状態になるよう制御する。原子性(atomicity)

### トランザクションを使うための指示
	BEGIN 開始指示。この指示以降のSQL文を１つのトランザクションとする。
	COMMIT 終了指示。この指示までを１つのトランザクションとし、変更を確定する。コミット。
	ROLLBACK 終了指示。この指示までを１つのトランザクションとし、変更を取り消す。ロールバック。

	事例
	/*　このSQL文では処理１と処理２を不可分なものとして扱う。
	　　もし処理１を実行した直後に障害が発生した場合、
	　　自動的にロールバックが行われ、処理１の実行は取り消される。
	*/
		BEGIN;  -- この行より下がトランザクション
		
		-- 処理１：アーカイブテーブルへコピー
		INSERT INTO 家計簿アーカイブ_TBL
		SELECT * FROM 家計簿_TBL WHERE 日付 <= '2018-01-31';
		-- 処理２：家計簿テーブルから削除
		DELETE FROM 家計簿_TBL WHERE 日付 <= '2018-01-31';

		COMMIT;  -- この行より上がトランザクション


### ＤＢ更新で発生する障害
	・ダーティリード(dirty read)。まだコミットされていない未確定の変更を、ほかの人が読めてしまう。
	・反復不能読み取り(non-repeatable read)。あるテーブルに対してSELECT文を実行した後、ほかの人がUPDATE文でデータを書き換えると、次回	SELECTした際に検索結果が異なってしまう。
	ファントムリード(phantom read)。２回のSELECT文の間にほかの人がINSERT文で行を追加すると、２回目のSELECT文で“結果行数”が変わってしまう。

### トランザクション分離レベル(transaction isolation level)
	・ロック(lock)。あるトランザクションが現在読み書きしている行に鍵をかけ、ほかの人のトランザクションからは読み書きできないようにすること。
	・DBMSでどの程度厳密にトランザクションを分離するかを指定するレベル。正確なデータ操作とパフォーマンスは二律背反の関係にあることに注意。

	分離レベル        障害の可能性（×：恐れあり、〇発生しない）
	READ UNCOMMITTED…ダーティ×、反復不能×、ファントム×
	READ COMMITTED  …ダーティ〇、反復不能×、ファントム×
	REPEATABLE READ …ダーティ〇、反復不能〇、ファントム×
	SERIALIABLE     …ダーティ〇、反復不能〇、ファントム〇

	事例
	/*　トランザクション分離レベルの指定ができるSQL文
	*/
		SET TRANSACTION ISOLATION LEVEL 分離レベル名
		
		SET CURRENT ISOLATION 分離レベル名

### ロックの種類
	行ロック。ある特定の１行だけをロックする
	表ロック。ある特定のテーブル全体をロックする
	データベースロック。データベース全体をロックする
	
	排他ロック(exclusive lock)。ほかからのロックを一切許可しない。
	共有ロック(shared lock)。ほかからの共有ロックを許す。データ読み取り時に利用する。

	事例
	/*　行ロックの取得 - SELECT ～ FOR UPDATE (NOWAIT)
	　　通常のSELECT文は共有ロックがかかる
	　　SELECT ～ FOR UPDATEにすると排他ロックがかかる
	　　NOWAITオプションを指定すると、DBMSはロックが解除されるまでトランザクションを待機状態にせず、
	　　DBMSはロック解除を待機せずにすぐさまロック失敗のエラーを返す（処理を待たせたくないアプリ向き）
	*/
		BEGIN;  -- この行より下がトランザクション
		
		-- 処理１
		SELECT * FROM 家計簿_TBL
			WHERE 日付 >= '2018-02-01' FOR UPDATE; -- 2月以降のデータを明示的にロックする
		-- 集計処理１
		SELECT ～;
		-- 集計処理２
		SELECT ～;
		-- 集計処理３
		SELECT ～;

		COMMIT;  -- ロックが解除される。　この行より上がトランザクション

	事例
	/*　表ロックの取得 - LOCK TABLE テーブル名 IN モード名 MODE (NOWAIT)
	　　モード名　排他ロックは EXCLUSIVE 、共有ロックは SHARE
	*/
		BEGIN;  -- この行より下がトランザクション
		
		-- 処理１
		LOCK TABLE 家計簿_TBL IN EXCLUSIVE MODE; -- 表を明示的にロック
			WHERE 日付 >= '2018-02-01' FOR UPDATE; -- 2月以降のデータを明示的にロックする
		-- 集計処理１
		SELECT ～;
		-- 集計処理２
		SELECT ～;
		-- 集計処理３
		SELECT ～;

		COMMIT;  -- ロックが解除される。　この行より上がトランザクション

### デッドロック(dead lock)を予防する方法
	・トランザクションの時間を短くする
	・同じ順番でロックする


## テーブルの作成

### SQL命令の分類
	①DML(Data Manipulation Language)：データ操作言語
		・データの格納、取り出し、更新、削除などの命令
		・SELECT , INSERT , UPDATE , DELETE , EXPLAIN , LOCK TABLE など
	②TCL(Transaction Control Language)：トランザクション制御言語
		・トランザクションの開始や終了の命令
		・COMMIT , ROLLBACK , SET TRANSACTION , SAVEPOINT
	③DDL(Data Definition Language)：データ定義言語
		・テーブルなどの作成や削除、各種設定などの命令
		・CREATE , ALTER , DROP , TRUNCATE
	④DCL(Data Control Language)：データ制御言語
		・DMLやDDLの利用に関する許可や禁止を設定する命令
		・GRANT , REVOKE

### DCLの構文
	GRANT 権限名 TO ユーザ名
	REVOKE 権限名 FROM ユーザ名
	詳細は各製品のマニュアル参照

### テーブルの作成/削除/変更
	基本形の構文
		CREATE TABLE テーブル名
			(
			列名１ 列１の型名,
			列名２ 列２の型名,
			…
			列名Ⅹ 列Ⅹの型名,
			)

	省略値の指定を含むテーブル作成の構文
		CREATE TABLE テーブル名
			(
			列名 列の型名 DEFAULT 省略値,
			…
			)

	テーブルの削除
		DROP TABLE テーブル名

	テーブル定義の変更
	・列の追加
		ALTER TABLE テーブル名 ADD 列名 型 制約
	・列の削除
		ALTER TABLE テーブル名 DROP COLUMN 列名 型 制約
		※基本はDROPで列削除だがMS SQLは列削除と制約削除を区別するためにDROP COLUMNを使う

	事例
	/*	リスト10-1 家計簿テーブルの作成
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW家計簿_TBL]  -- DokoQL.dboを省略するとmaster.dboに作成される
		(
	      [日付]	DATE,
	      [費目ID]	INTEGER,
	      [メモ]	VARCHAR(100),
	      [入金額]	INTEGER,
	      [出金額]	INTEGER
		);
	※Microsoft SQLのCREATE TABLEについて
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/create-table-transact-sql?view=sql-server-ver15

	事例
	/*	リスト10-3 デフォルト値を含む家計簿テーブルの作成
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW2家計簿_TBL]
		(
	      [日付]	DATE,
	      [費目ID]	INTEGER,
	      [メモ]	VARCHAR(100) DEFAULT '不明',
	      [入金額]	INTEGER		 DEFAULT 0,
	      [出金額]	INTEGER		 DEFAULT 0
		);
		
	事例
	/*	リスト10-4 家計簿テーブルの変更
	*/
	ALTER TABLE [dokoQL].[dbo].[NEW家計簿_TBL]
	  ADD [関連日] DATE; -- 列追加

	ALTER TABLE [dokoQL].[dbo].[NEW家計簿_TBL]
	  DROP COLUMN [関連日]; -- 列削除

	※Microsoft SQLのALTER TABLEについて
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/alter-table-transact-sql?view=sql-server-ver15
	

### テーブルのデータ全件を高速削除する
	TRUNCATE TABLE [dokoQL].[dbo].[NEW家計簿_TBL]
	DELETE FROM テーブル名　とほぼ同じだが、
	次の違いがある。
	・DELETEはWHERE句で指定した行のみを削除できるが、TRUNCATEは必ず全行を削除する。
	・DELETEはロールバックに備えて記録を残して削除するが、TRUNCATEは記録を残さない。
	・DELETEは低速、TRUNCATEは高速
	・DELETEはDML(データ操作言語)、TRUNCATEはDDL(データ定義言語)に属する。
	TRUNCATEは厳密にはデータ削除ではなく、テーブル初期化の命令で、
	「テーブルをDROPした後、同じものをCREATEする」イメージ。

### 制約(constraint)
	CREATE TABLE文でテーブルを定義するときに、列定義の末尾に制約を指定できる。
	制約は複数指定することもでき、その場合はカンマで区切らずにそのまま並べて記述する。
	基本形の構文
		CREATE TABLE テーブル名
			(
			列名 型 制約の指定,
			…
			)
	
	制約は基本的に３つある
	・NOT NULLL制約：NULLの格納を許さない。DEFAULT指定と組み合わせて使う
	・UNIQUE制約：列の内容の重複を許さない。ただしNULLが格納された行が複数あっても許される。「NULLはNULLと等しくない」
	・CHECK制約：列に格納される値が妥当かを細かく判定するときに使用。条件式が真となるような値だけが許される。

	事例
	/*	リスト10-5 基本的な３つの制約を活用
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW家計簿_TBL]
		(
	      [日付]	DATE			NOT NULL,   -- NOT NULL制約
	      [費目ID]	INTEGER,
	      [メモ]	VARCHAR(100)	DEFAULT '不明' NOT NULL,
	      [入金額]	INTEGER			DEFAULT 0 CHECK( [入金額] >= 0),   -- CHECK制約
	      [出金額]	INTEGER			DEFAULT 0 CHECK( [出金額] >= 0)
		);

	CREATE TABLE [dokoQL].[dbo].[NEW費目_TBL]
		(
	      [ID]	INTEGER,
	      [名前] VARCHAR(40) UNIQUE   -- UNIQUE制約
		);

### 主キー制約(PRIMARY KEY制約)
	単なる「NULLも重複も許されない列」ではなく、主キーとしての役割があるという意味になる
	事例
	/*	リスト10-7 単独の列に主キー制約を付ける場合
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW費目2_TBL]
		(
	      [ID]	INTEGER		 PRIMARY KEY,   -- 主キー制約
	      [名前] VARCHAR(40) UNIQUE
		);

	/*	リスト10-8 複数の列に主キー制約を付ける場合
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW費目3_TBL]
		(
	      [ID]	INTEGER,   -- 主キー制約
	      [名前] VARCHAR(40) UNIQUE,
		  PRIMARY KEY ([ID],[名前])   -- ID列と名前で主キーを構成する
		);

### 参照整合性の崩壊
	参照整合性(referential integrity)：外部キーが指し示す先にきちんと行が存在してリレーションシップが成立していること
	参照整合性の崩壊を引き起こすデータ操作はつぎの４つ
	①「他の行から参照されている」行を削除する
	②「他の行から参照されている」行の主キーを変更する
	③「存在しない行を参照する」行を追加する
	④「存在しない行を参照する」行に更新する

### 外部キー制約(FOREIGN KEY制約)
	基本の構文
		CREATE TABLE テーブル名
			(
			列名 型 REFERENCES 参照先テーブル名(参照先列名),
			…
			)
	事例
	/*	リスト10-10 外部キー制約の指定
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW家計簿2_TBL]
		(
	      [日付]	DATE			NOT NULL,						 -- NOT NULL制約
	      [費目ID]	INTEGER			REFERENCES [dokoQL].[dbo].[NEW費目3_TBL]([ID]),
	      [メモ]	VARCHAR(100)	DEFAULT '不明' NOT NULL,
	      [入金額]	INTEGER			DEFAULT 0 CHECK( [入金額] >= 0), -- CHECK制約
	      [出金額]	INTEGER			DEFAULT 0 CHECK( [出金額] >= 0)
		);

	主キーの場合と同様に、CREATE TABEL文の最後にまとめて定義できる
		CREATE TABLE テーブル名
			(
			…
			FOREIGN KEY (参照元列名) REFERENCES 参照先テーブル名 (参照先列名)
			)

	事例
	/*	問題10-2
	*/
	CREATE TABLE [dokoQL].[dbo].[学部_TBL]
		(
	      [ID]   CHAR(1)      PRIMARY KEY,                 -- 学部を一意に識別する文字
	      [名前] VARCHAR(20)  UNIQUE NOT NULL,             -- 学部の名前(必須、重複不可）
	      [備考] VARCHAR(100) DEFAULT '特になし' NOT NULL  -- 特にない場合は'特になし'を設定
		);


	/*	問題10-3
	*/
	CREATE TABLE [dokoQL].[dbo].[学生_TBL]
		(
	      [学籍番号] CHAR(8) PRIMARY KEY,                  -- 学生を一意に識別する番号(必須)
	      [名前] VARCHAR(30)  NOT NULL,                    -- 学生の名前(必須)
		  [生年月日] DATE NOT NULL,                        -- 学生の生年月日(必須)
		  [血液型] CHAR(2) CHECK(                          -- 学生の血液型　※A,B,O,AB いずれかで不明な場合はNULL
									[血液型] IN ('A','B','O','AB')
								OR	[血液型] IS NULL
								),
	      [学部ID] CHAR(1) REFERENCES [dokoQL].[dbo].[学部_TBL]([ID]) -- 学部テーブルのID列の値を格納する外部キー
		);
	
## さまざまな支援機能

### インデックスの特徴
	・インデックスは指定した列に対して作られる。
	・インデックスが存在する列に対して検索が行われた場合、DBMSは自動的にインデックスの使用を試みるため、
	　高速になることが多い（検索の内容によってはインデックスの利用はできず性能も向上しないことがある）
	・インデックスには名前を付けなければならない。

### インデックスの作成と削除
	基本の構文
	  作成
		CREATE INDEX インデックス名 ON テーブル名(列名)
	  削除
		DROP INDEX インデックス名 ON テーブル名
		
	※Microsoft SQLのCREATE INDEX (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/create-index-transact-sql?view=sql-server-ver15
		
	※Microsoft SQLのDROP INDEX (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/drop-index-transact-sql?view=sql-server-ver15


	事例
	/*	リスト11-1 新家計簿テーブルにインデックスを２つ作る
	*/
	CREATE INDEX 費目ID_idx ON [dokoQL].[dbo].[新家計簿_TBL](費目ID);
	CREATE INDEX メモ_idx ON [dokoQL].[dbo].[新家計簿_TBL](メモ);
	※SSMSでは、dokoQL→テーブル→dbo.新家計簿_tbl→インデックス この下に費目ID_idxとメモ_idxが配置される。
	
	事例
	/*	リスト11-1add 新家計簿テーブルのインデックスを削除する
	*/
	DROP INDEX 費目ID_idx ON [dokoQL].[dbo].[新家計簿_TBL];
	DROP INDEX メモ_idx ON [dokoQL].[dbo].[新家計簿_TBL];

### 一般的にインデックス設定の効果が高い列
	・WHERE句に頻繁に登場する列
	・ORDER BY句に頻繁に登場する列
	・JOINの結合条件に頻繁に登場する列（外部キーの列）
	
	事例
	/*	リスト11-2 インデックスのある列をWHERE句に指定する（完全一致検索）
	*/
	SELECT * FROM [新家計簿_TBL]
		WHERE メモ = '不明';

	/*	リスト11-3 インデックスのある列をWHERE句に指定する（前方一致検索）
	*/
	SELECT * FROM [新家計簿_TBL]
		WHERE メモ LIKE '1月の%';
	★部分一致検索や後方一致検索ではインデックスを利用できない

	/*	リスト11-4 インデックスのある列をORDER BY句に指定する
	*/
	SELECT * FROM [新家計簿_TBL]
		ORDER BY [費目ID];

	/*	リスト11-5 インデックスのある列をJOINの結合条件に指定する
	*/
	SELECT * FROM [新家計簿_TBL]
		JOIN [費目_TBL]
		ON [新家計簿_TBL].[費目ID] = [費目_TBL].[ID];

### インデックスを作成することによるデメリット
	・索引情報を保存するために、ディスク容量を消費する。
	・INSERT文、UPDATE文、DELETE文のオーバヘッドが増える
	　理由は、テーブルのデータが変更されるとインデックスも書き換える必要があるから。

### ビュー(VIEW)の特徴
	・WHERE句で検索した結果表をテーブルのように扱えるので、
	何度も同じWHERE句を書かなくてよい

### ビューの作成と削除
	基本の構文
	  作成
		CREATE VIEW ビュー名 AS SELECT文;
	  削除
		DROP VIEW ビュー名;
		
	※Microsoft SQLのCREATE VIEW (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/create-view-transact-sql?view=sql-server-ver15
		
	※Microsoft SQLのDROP VIEW (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/drop-view-transact-sql?view=sql-server-ver15


	事例
	/*	リスト11-7 4月に関する新家計簿データのみを持つビューを定義
	*/
	CREATE VIEW 新家計簿4月_view AS 
		SELECT * FROM [新家計簿_tbl]
			WHERE　[日付] >= '2018-04-01'
			AND　  [日付] <= '2018-04-30';
	※SSMSでは、dokoQL→ビュー→dbo.新家計簿4月_viewが配置される。

	/*	リスト11-7add ビューを削除
	*/
	DROP VIEW 新家計簿4月_view

### ビューのメリット
	・SQL文がシンプルになる
	・見せたくない列（機密情報列など）を取り除いたビューを作って見せることができる
	・データ参照を許可する権限に応じたビューを作ることもできる

	事例
	/*	リスト11-8 新家計簿4月ビューを使ったSQL文の実行
	*/
	SELECT * FROM [新家計簿4月_view];
	SELECT DISTINCT [費目ID] FROM [新家計簿4月_view];

### ビューのデメリット
	・実行されるSQL文は、一見するよりも負荷の高い処理になる可能性がある
	理由　ビューの実体は単なる「名前を付けたSELECT文」なので、
	　　　展開すると長くて複雑で冗長なSQL文になることがある。
　　　　　ビューは実体を持たないので、ビューを参照するたびに
　　　　　実際にデータを持っているテーブルへのSELECT文が実行されることにより、
　　　　　性能上の問題になることがある。


### マテリアライズド・ビュー
	・SELECT文文による検索結果をキャッシュしているテーブルのようなもの。
	　ディスク容量は消費するが、テーブルを経由することなくデータを直接参照できるので、
	　高速に動作する。
	　インデックス作成も可能なので、性能を重視したい場合には利用検討するべき。

### 採番
	採番…追加する行に独自の番号を振るために、適切な番号を取得すること。
	採番テーブル…すでに採番した番号や最後に採番した番号を記録する専用のテーブル
	
### 採番手法その１。連番が自動的に降られる特殊な列の定義
	・CREATE TABLE文で列を定義するときに「連番を振る列である」ことを指定すればよい。
	
	基本の構文
		CREATE TABLE テーブル名
		(
		列名 型 IDENTITY PRIMARY KEY, -- 『制約の指定』の部分にIDENTITY と PRIMARY KEYを記述する
		…
		)

	事例
	/*	リスト11-9 連番の指定
	*/
	CREATE TABLE [NEW費目4_tbl]
		(
	      [ID]	INTEGER		 IDENTITY PRIMARY KEY,   -- 連番指定と主キー制約
	      [名前] VARCHAR(40) UNIQUE
		);

	※Microsoft SQLのCREATE TABLE (Transact-SQL) IDENTITY (プロパティ)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/create-table-transact-sql?view=sql-server-ver15
	（抜粋）
	IDENTITY: 新しい列が ID 列であることを示します。
	テーブルに行が新しく追加されると、データベース エンジンは列に一意な増分の値を設定します。
	ID 列は通常、PRIMARY KEY 制約と共に使用され、テーブルの一意な行識別子 (ROWID) の役割を果たします。
	IDENTITY プロパティは、 tinyint 、 smallint 、 int 、 bigint 、 decimal(p,0) 、 numeric(p,0) のいずれかの列に割り当てることができます。
	ID 列は 1 つのテーブルにつき 1 つだけ作成できます。 

### 採番手法その２。シーケンス。連番を管理する専用の道具
	シーケンス(sequence)…常に採番した最新の値を記録している
	シーケンスから値を取り出すとその値は確定するので、ロールバックしても戻らない。
	※Microsoft SQLのシーケンス番号
		https://docs.microsoft.com/ja-jp/sql/relational-databases/sequence-numbers/sequence-numbers?view=sql-server-ver15
	抜粋
		シーケンスは、シーケンスが作成された仕様に従って数値のシーケンスを生成するユーザー定義のスキーマ バインド オブジェクトです。
		数値のシーケンスは、定義された間隔で昇順または降順に生成され、要求に応じて繰り返されます。
		ID 列とは異なり、シーケンスはテーブルには関連付けられていません。 
		行の挿入時に生成される ID 列値とは異なり、アプリケーションは、 NEXT VALUE FOR 関数を呼び出すことにより、行を挿入する前に次のシーケンス番号を取得できます。
		番号がテーブルに挿入されない場合でも、シーケンス番号は、NEXT VALUE FOR が呼び出されたときに割り当てられます。
		NEXT VALUE FOR 関数は、テーブル定義内の列の既定値として使用できます。
		一度に複数のシーケンス番号の範囲を取得するには、 sp_sequence_get_range を使用します。


### シーケンスの作成と削除と取得
	基本の構文(SQL-Serverの構文。DBMSごとにかなり違う）)
	  作成
		CREATE SEQUENCE シーケンス名;
	  削除
		DROP SEQUENCE シーケンス名;
	  取得
		SELECT NEXT VALUE FOR シーケンス名;

		
	※Microsoft SQLのCREATE SEQUENCE (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/create-sequence-transact-sql?view=sql-server-ver15

	※Microsoft SQLのDROP SEQUENCE (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/statements/drop-sequence-transact-sql?view=sql-server-ver15

	※Microsoft SQLのNEXT VALUE FOR (Transact-SQL)
		https://docs.microsoft.com/ja-jp/sql/t-sql/functions/next-value-for-transact-sql?view=sql-server-ver15


	事例
	/*	リスト11-10 シーケンスの作成と取得と削除
	*/
	CREATE SEQUENCE [dbo].[費目4_seq]	-- スキーマ名とシーケンス名を記述する。
										-- データベース名の[dokoQL]を記述するとエラーになる
		AS int							-- シーケンスを整数型として定義
	    START WITH 1					-- シーケンス オブジェクトによって返される最初の値  
	    INCREMENT BY 1;					-- シーケンス オブジェクトの値の増分 
		-- その他のオプションとして最小値/最大値とそれを超過した場合の挙動などがある

	※SSMSでは、システムデータベース→master→プログラミング→シーケンス　に　dbo.費目4_seqが配置される。

	SELECT NEXT VALUE FOR [費目4_seq]　as [一回目];
	SELECT NEXT VALUE FOR [費目4_seq]　as [ニ回目];
	※実行結果は、一回目　1、二回目　3 と表示される

	DROP SEQUENCE [費目4_seq];
	*/

	事例
	/*	（次のSQLでシーケンスの値をセットするための）テーブルを定義して作成
	*/
	CREATE TABLE [dokoQL].[dbo].[NEW費目4_tbl]
		(
		    [ID]   INTEGER	 PRIMARY KEY,   -- 主キー制約(連番指定はしない）
		    [名前] VARCHAR(40) UNIQUE
		);
	/*	リスト11-13 SQL-Serverにおける費目行の追加
	*/
	INSERT INTO [dokoQL].[dbo].[NEW費目4_tbl]([ID],[名前])
		VALUES (
				( NEXT VALUE FOR [費目4_seq]), '接待交際費'
				);

	/* 追加結果の確認
	*/
	SELECT [ID],[名前]
	  FROM [dokoQL].[dbo].[NEW費目4_tbl];


### ACID特性
	・Atomicity 原子性。処理が中断しても中途半端な状態にならない。
	・Consistency 一貫性。データの内容が矛盾した状態にならない。
	・Isolation 分離性。複数の処理を同時実行しても副作用がない。
	・Durability 永続性。記録した情報は消滅せず保持され続ける。
	
### バックアップ
	Backup。データ消失に備えるしくみ。
	DR:Disaster Recover。災害復旧対策。
	
	オフラインバックアップ…DBMSを停止して行うバックアップ
	オンラインバックアップ…DBMSを稼働させながら行うバックアップ
	
	低頻度バックアップ…日次、週次、月次でデータベースの内容を。
	高頻度バックアップ…数分毎～数時間毎でログファイルの内容を。
	
	REDOログ…リドゥログ。アーカイブログ、トランザクションログとも呼ぶ。内容はそこまでに実行したSQL文。
	
	ロールバック………実行した処理を取り消す。
	　　　　　　　　　データベースの利用中に、SQLの実行失敗やデッドロックなどでたびたび実施される。
	ロールフォワード…まだ実行されていない処理を実行する。
	　　　　　　　　　障害復旧時に行われる処理。
	　　　　　　　　　ログに記載されているSQL文を再実行して、障害が発生する直前の状態までデータを更新する。
