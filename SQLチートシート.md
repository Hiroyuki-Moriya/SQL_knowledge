/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
# SQLチートシート (第1部　/　全4部)
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
## 基本文法
	/* コメント */
	-- コメント

	SELECT 列名 -- ★SELECT文には他の句もつくので、全貌は第2部を参照
	  FROM テーブル名
	 WHERE (修飾)(その他修飾);


	UPDATE テーブル名 -- ★WHEREのないUPDATE命令は全件更新！
	   SET 列名1=数値,列名2='文字列'
	 WHERE (修飾);

	DELETE FROM テーブル名 -- ★WHEREのないDELETE命令は全件削除！
	 WHERE (修飾);

	INSERT INTO テーブル名(列名1,列名2)
	 VALUES(値1,値2) ;
	 /*
   ・テーブル名の後に値をセットする列名を順番に記述する。
   ・すべての列にデータをセットする場合は列名を省略することができる。
	 ・文字列はダブルコーテーションではなく、シングルコーテーションで囲う。
	 ・1行追加の時はinsertとvalueが1文づつ必要。1行のinsert文にvaluesは2行書けない
	 ・値が決まっていない(=null)の列については、valuesに空文''やNULLを書いても良いし、テーブル名の後に列名を書かない方法もある。
	 */

	☆ データ全件を削除する命令にはDELETEとTRUNCATEがある。
	☆TRUNCATEは厳密にはデータ削除ではなく、テーブル初期化の命令で、
	☆「テーブルをDROPした後、同じものをCREATEする」イメージ。
	☆詳細はSQLチートシート (第3部)を参照。


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 別名定義
	 列名 AS 別列名
	 テーブル名 AS 別テーブル名

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 条件式

### 比較演算子
	  =
	  <
	  >
	  <=
	  >=
	  <>
	 IS NULL   -- ★NULLは=演算子では判定できない
	 IS NOT NULL

### LIKE
	WHERE 列名 LIKE パターン文字列
	　-- パターン文字列
	  -- %　任意の0文字以上の文字列
	  -- _ (アンスコ)任意の１文字
	　-- 　事例　LIKE '%1月%’
	  -- %や_を検索文字列に使うときは ESCAPE句を使用する
	  --   ESCAPE文字に続く%や_は文字として扱われる
	　-- 　事例　LIKE '%100$%' ESCAPE '$'　-- 100%を検索文字列にしている
	　--   事例★ ２文字のはずなのに　= '食費' や　like '食費' では検索できないときは　like '食費%'でうまくいく。

### BETWEEN
	WHERE 列名 BETWEEN 値1 AND 値2 -- 意味は、値1以上 かつ 値2以下　

### IN / NOT IN
	WHERE 列名 IN (値1,値2,値3,…)　-- 値のどれかと一致すればTRUE
	  --   事例　入金額 IN (1000,2000,3000) -- 入金額が2000ならば値2を満たすので、TRUE

	WHERE 列名 NOT IN (値1,値2,値3,…) -- 値のどれとも一致しなければTRUE
	  --   事例　入金額 NOT IN (1000,2000,3000) -- 入金額が2000ならば値2を満たすので、FALSE

### ANY / ALL
	WHERE 列名 ANY 演算子 (値1,値2,値3,…)
	  -- 比較演算子と組み合わせてどれかが満たされればTRUE
	  --   事例　WHERE 入金額 < (1000,2000,3000) -- 入金額が2500ならば値3を満たすので、TRUE

	WHERE 列名 ALL 演算子 (値1,値2,値3,…)
	  -- 比較演算子と組み合わせて全てが満たされればTRUE
	  --   事例　WHERE 入金額 < (1000,2000,3000) -- 入金額が1000ならば値1を満たさないので、FALSE

### 同じ意味になる演算子
	NOT IN と <>ALL … すべての値と一致しないことを判定する演算子
	IN     と  =ANY … いずれかの値と一致することを判定する演算子


### 論理演算子
	 AND
	 OR
	 NOT
	 -- 優先順位は　NOT > AND > OR
	 -- カッコを付ければ優先される
	  --   事例　条件式1 OR 条件式2 AND 条件式3　と　条件式1 OR (条件式2 AND 条件式3)　は　同値になる。

	★時刻を含む日付判定
	　　2018年3月以前のデータを抽出する
	　　日付 <= '2018-03-31'  -- 2018-03-31 00:00:00と解釈するDBMSがあると、2018-03-31 10:30:00は含まれなくなってしまう
	　　日付 <  '2018-04-01'  -- このように書くべき


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## データ型

### 基本のデータ型
	 整数
	  整数値　INTEGER型
	  小数    DECIMAI型 REAL型
	 文字列
	  固定長 CHAR型
	  可変長 VARCHAR型
	 日付と時刻
	         DATETIME型
	         DATE型
	         TIME型

### SQL SERVERのデータ型
	https://docs.microsoft.com/ja-jp/sql/t-sql/data-types/data-types-transact-sql?view=sql-server-ver15

#### 真数
	  decimal型(numeric型) 固定長の有効桁数と小数点以下保持桁数を持つ数値データ型です。
	                       decimal と numeric は同義。

	  bigint型   整数データを使用する実数データ型。-2^63～2^63-1
	  int型      整数データを使用する実数データ型。-2^31～2^31-1(2,147,483,647)
	  smallint型 整数データを使用する実数データ型。-2^15～2^15-1(32,767)
	  tinyint型  整数データを使用する実数データ型。0～255

	  money型      金額値または通貨値を表すデータ型。-922,337,203,685,477.5808 ～ 922,337,203,685,477.5807
	  smallmoney型 金額値または通貨値を表すデータ型。- 214,748.3648 ～ 214,748.3647

	  bit型 1、0、または NULL の値をとる整数型

#### 概数 浮動小数点数値データで使用する概数型
	  float型 - 1.79E+308 から -2.23E-308、0、および 2.23E-308 から 1.79E+308
	  real型  - 3.40E + 38 から -1.18E - 38、0、および 1.18E - 38 から 3.40E + 38

#### 日付と時刻
	  date型 既定の文字列リテラル形式YYYY-MM-DD。文字長10 文字
	  time型 1日の時刻を定義します。 時刻は 24 時間形式で、タイム ゾーンは認識されません。hh:mm:ss[.nnnnnnn] (Informatica の場合は nnn)
	  datetime型  24時間形式の時刻 (1 秒未満の秒を含む) と組み合わせた日付を定義。
	  datetime2型  datetime 型を拡張して、日付範囲と既定の有効桁数を増やし、ユーザーが必要に応じて有効桁数を指定できるようにしたもの
	  smalldatetime型 日付を時刻と組み合わせて定義します。 時間は 24 時制とし、秒は常にゼロ (: 00) にし、秒の小数部は使用しません。
	  datetimeoffset型 タイム ゾーンを認識する 24 時間形式の時刻と組み合わせた日付を定義

#### 文字列
	  char型    固定サイズの文字データ型。8000バイトペアまでの文字列。
	  varchar型 可変サイズの文字データ型。8000バイトペアまでの文字列。
	  text型    大きな非 Unicode 文字を格納するための固定長のデータ型。削除予定

#### Unicode文字列
	  nchar    固定サイズの文字データ型。Unicode 文字データの全範囲を格納。4000バイトペアまでの文字列。
	  nvarchar 可変サイズの文字データ型。Unicode 文字データの全範囲を格納。4000バイトペアまでの文字列。
	  ntext    大きなUnicode 文字を格納するための可変長のデータ型。削除予定

#### 他
	  バイナリ文字列、などなど


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## NULL値をセットする方法

	Ctrl + 0キーを押すか値を空にしてEnterキーを押します。

	CSVでNULL値を入れたい場合はNULLの部分に\Nを使用すればよい

	データのインポートで、ソースにEXCELのデータを使う場合、かつ、DB側の型がintの場合は、EXCELのセルにNULLと書けばよい。
	データのインポートで、ソースにEXCELのデータを使う場合、かつ、DB側の型がcharの場合は、EXCELのセルに何も入れてはならない。

	INSERT文で、値にNULLと書く
		INSERT INTO テーブル名(列名1,列名2)
	 	VALUES(値1,NULL) ;

	UPDATE文で、値をNULLに更新する
		UPDATE テーブル名
		SET 列名 = NULL
		WHERE (修飾);

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## データインポート方法

### Excel から SQL Server または Azure SQL Database にデータをインポートする
	https://docs.microsoft.com/ja-jp/sql/relational-databases/import-export/import-data-from-excel-to-sql?view=sql-server-ver15

	方法１．SQL Server インポートおよびエクスポート ウィザード
	 SQL Server インポートおよびエクスポート ウィザードのページをステップ実行して、Excel ファイルから直接データをインポートします。
	 必要に応じて、後でカスタマイズして再利用できる SQL Server Integration Services (SSIS) パッケージとして設定を保存します。
	 1.SQL Server Management Studioで、 SQL Serverデータベース エンジンのインスタンスに接続します。
	 2.[データベース] を展開します。
	 3.データベースを右クリックします。
	 4.[タスク] にカーソルを合わせます。
	 5.以下のいずれかのオプションをクリックします。
	  5-1.データのインポート
	　　　★データソースにFlat Fileを選び、変換先をSQL Server Native Client 11.0を選ぶ。
	　　　　データベースに既存のデ－タベース、マッピングの編集で行追加を選ぶと、既存テーブルへのデータ追加ができる。
	  5-2.データのエクスポート
	  5-3.フラット ファイルのインポート
	      ★5-3の方法は、新規にテーブルを作成し、データをインポートするので、既存テーブルへのデータインポートはできない。

	★データ変換不可能となって変換しない場合がある。（変換するクエリーは保存できる）
	事例１　Double → varchar変換だと判定された例
	　　入力側　受注ID 入力値には3桁の数値しか入っていない。
	　　出力側　　　　 varchar(20)
	　　対策　　入力側のセル書式を文字列に変更し、数値の頭にシングルコーテーションを付けて文字列であることを明示した。

### CSV ファイルを BULK INSERT を使ってインポートする
	https://sql55.com/query/bulk-insert.php

	1.テーブルを作る
	　CREATE TABLE Students (
	   StudentID INT NOT NULL PRIMARY KEY,
	   FirstName VARCHAR(50) NULL,
	   LastName VARCHAR(50) NULL,
	   BirthDay DATE NULL,
	   Gender CHAR(1) NULL
	　);

	2.インポートしたい CSV ファイルを用意
	　c:\NewStudents.txt
	   1,Taro,Yamada,1980-02-15,M
	   2,Hanako,Tanaka,1979-12-30,F
	   3,Yuko,Suzuki,1979-07-07,F
	   4,Takao,Sato,1980-03-12,M

	3.BULK INSERT を使って Students テーブルにデータをインポート
	 BULK INSERT Students
	  FROM 'c:\NewStudents.txt'
	   WITH
	   (
	     FIELDTERMINATOR = ',',  -- FIELDTERMINATOR にはデータを区切る文字
	     ROWTERMINATOR = '\n'    -- ROWTERMINATOR には１行の終わりを示す文字
	   );

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## SELECT文にのみ可能な“検索結果を加工する”ための「その他修飾」
	SELECT 列名
	  FROM テーブル名
	 WHERE (修飾)(その他修飾);

### DISTINCT  検索結果から重複行を除外する。メモリーを大量消費するので乱用に注意
	SELECT DISTINCT 列名 … -- ★DISTINCTはSELECT文の最初に記述する。
	  FROM テーブル名

	-- 事例 家計簿テーブルの明細の中から実際に使っている費目を抽出する
	--      SELECT DISTINCT 費目 FROM 家計簿_TBL

### ORDER BY  検索結果の順序を並び替える。メモリーを大量消費するので乱用に注意
	SELECT 列名 …
	  FROM テーブル名
	  ORDER BY 列名1 並び順,  列名2 並び順, … -- ★ASC(昇順、省略値),DESC(降順)

	-- 事例 家計簿テーブルの明細の中を入金額(降順)で並べ、同値ならば出金額（降順）に並べる
	--      SELECT * FROM 家計簿_TBL
		    ORDER BY 入金額 DESC, 出金額 DESC

	SELECT 列名1,列名2, …
	  FROM テーブル名
	  ORDER BY 列番号1 並び順,  列番号2 並び順, … -- ★列番号は、SELECT文で選択された列のリストの順番

	-- 事例 家計簿テーブルの明細の中を入金額(降順)で並べ、同値ならば出金額（降順）に並べる
	--      SELECT 日付,費目,入金額,出金額 FROM 家計簿_TBL
		    ORDER BY 3 DESC,  4 DESC

### OFFSET - FETCH 検索結果から件数を限定して取得する
	SELECT 列名 …
	  FROM テーブル名
	  ORDER BY 列名1 並び順,  列名2 並び順 -- ★OFFSET句とFETCH句の前には必ずORDER BY句が必要（無ければエラー）
	  OFFSET 先頭から除外する行数 ROWS -- ★除外せずに先頭から取得するときは 0 ROWS を指定する
	  (FETCH NEXT 取得行数 ROWS ONLY)

	-- 事例 家計簿テーブルの明細を出金額の多い順に並べ、その中から11～15行目を取得する
	--      SELECT * FROM 家計簿_TBL
		    ORDER BY 出金額 DESC
		    OFFSET 10 ROWS         -- 先頭10行を除外
		    FETCH NEXT 5 ROWS ONLY -- そのあとに続く5行を取得する
	/* ★ OFFSET - FETCHは、DBMSごとに異なるので、注意が必要
		SQL Server FETCH (Transact-SQL) https://docs.microsoft.com/ja-jp/sql/t-sql/language-elements/fetch-transact-sql?view=sql-server-ver15
			引数の抜粋
				NEXT…現在の行の直後にある行を結果行として返し、この返した行に現在の行を加えます。
				　　　　カーソルに対する最初のフェッチが FETCH NEXT の場合、結果セットの先頭の行が返ります。
				FIRST…カーソル内の先頭行を返し、これを現在の行にします。
				LAST…カーソル内の最終行を返し、これを現在の行にします。
				ABSOLUTE { n| @nvar}…n または @nvar が正の値の場合は、カーソルの先頭から n 行目の行を返し、返した行を新しい現在の行にします。
	 */


### UNION     和集合の集合演算子。検索結果にほかの検索結果を足し合わせる。メモリーを大量消費するので乱用に注意
	SELECT 文1
	  UNION (ALL) -- ALLを付けないと重複行を1行にまとめ、ALLを付けると重複行をそのまま返す。
	SELECT 文2

	/* 事例 家計簿DBは件数が多くなり検索時間がかかるようになったので、
	   当月とそれ以外（アーカイブ）に分けて運用することにした。
	   ただし、１年間の結果を検索するときだけ両方を参照したい。 */
	--  SELECT 日付,費目,入金額,出金額 FROM 家計簿_TBL
	     UNION
	    SELECT 日付,費目,入金額,出金額 FROM 家計簿アーカイブ_TBL
	     ORDER BY 3 DESC, 4 DESC, 2 ASC

    /* 集合演算子を使える条件とORDER BY句を使うときの注意点
	   ★ テーブルの列数とデータ型はぴったり一致させておく必要がある
	      (選択列リストの数が合わないときは、足りないほうの選択列リストにNULLを追加することで、数を一致させられる)
	   ★ ORDER BY は最後のSELECT文に指定する
	   ★ UNIONのORDER BY は通常、列番号指定で行う。列名やASによる別名の場合は最初のSELECT文のものを指定する。
	*/

	/* 事例
		ドリルA LEVEL3-32
		口座テーブルと廃止口座テーブルに登録されている口座番号と残高の一覧を取得する。
		ただし、口座テーブルは残高がゼロのもの、廃止口座テーブルは解約時残高がゼロではないものを抽出の対象とする。
		一覧は口座番号順とする。
	*/
	SELECT [口座番号]
	      ,[残高]
	  FROM [dokoQL].[dbo].[口座_t]
	  WHERE [残高] = 0 -- ★それぞれのテーブルで条件抽出してUNIONで和集合にする
	UNION
	SELECT [口座番号]
	      ,[解約時残高]
	  FROM [dokoQL].[dbo].[廃止口座_t]
	  WHERE [解約時残高] <> 0 -- ★それぞれのテーブルで条件抽出してUNIONで和集合にする
	  ORDER BY 1 ASC;

	/* 事例
		ドリルA LEVEL3-33
		口座テーブルと廃止口座テーブルに登録されている口座番号と名義の一覧を取得する。
		一覧は名義順とし、その口座の状況がわかるように、有効な口座には「〇」を、廃止した口座には「×」を一覧に付記すること。
	*/
	SELECT [口座番号]
	      ,[名義]
	      , '〇' AS [口座区分] -- ★新しい列名「口座区分」を用意して〇印を入れる
	  FROM [dokoQL].[dbo].[口座_t]
	UNION
	SELECT [口座番号]
	      ,[名義]
	      , '×' AS [口座区分]  -- ★新しい列名「口座区分」を用意して×印を入れる
	  FROM [dokoQL].[dbo].[廃止口座_t]
	  ORDER BY 2

### EXCEPT / MINUS   差集合の集合演算子。検索結果からほかの検索結果を差し引く(MINUSはOracleDBのキーワード)
	SELECT 文1 -- 集合A
	  EXCEPT (ALL)      -- ALLを付けないと重複行を1行にまとめ、ALLを付けると重複行をそのまま返す。
	SELECT 文2 -- 集合B
	           -- 結果は、イメージ的にはマッチングの 'Aﾉﾐ' に相当する

	/* 事例 家計簿_TBLにあり、家計簿アーカイブ_TBLにない費目を参照したい。
	--  SELECT 費目 FROM 家計簿_TBL
	     EXCEPT
	    SELECT 費目 FROM 家計簿アーカイブ_TBL

	/* 集合演算子を使える条件とORDER BY句を使うときの注意点は、UNIONに同じ */

### INTERSECT 積集合の集合演算子。検索結果とほかの検索結果で重複する部分を取得する
	SELECT 文1
	  INTERSECT (ALL)      -- ALLを付けないと重複行を1行にまとめ、ALLを付けると重複行をそのまま返す。
	SELECT 文2

	/* 事例 家計簿_TBLと家計簿アーカイブ_TBLの両方にある費目を参照したい。
	--  SELECT 費目 FROM 家計簿_TBL
	     INTERSECT
	    SELECT 費目 FROM 家計簿アーカイブ_TBL

	/* 集合演算子を使える条件とORDER BY句を使うときの注意点は、UNIONに同じ */


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
# SQLチートシート (第2部　/　全4部)
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */

## 式と関数

### 計算式とは
	条件式は結果が真か偽になるもの
	結果が値になるもの。条件式の一部として用いられることもある。

	選択列リストで使うケース
	例　SELECT [出金額],[出金額]+100 AS '+100','SQL' AS 'ＳＱＬ'
		  FROM [dokoQL].[dbo].[家計簿_tbl]
		出金額	'+100'	'ＳＱＬ'
		308		408		SQL

	事例　データ登録時に使うケース
	/* LEVEL4-35
	口座テーブルに次のデータを登録する。
	ただし、キャンペーンによりSQL文にて残高に3,000円をプラスして登録すること。
	SQL文はデータごとに1つずつ作成すること。
	VALUES('0652281', 'タカギ　ノブオ', '1', 100000, '2018-04-01');
	*/
	INSERT INTO [dokoQL].[dbo].[口座_t]([口座番号] ,[名義] ,[種別] ,[残高] ,[更新日])
	  VALUES('0652281', 'タカギ　ノブオ', '1', 100000+3000, '2018-04-01');

	事例　データ修正時に使うケース
	/* LEVEL4-36
		4-35の問題で登録したデータについて、キャンペーンの価格が間違っていたことが判明した。
		該当するデータの残高それぞれから3,000円を差し引き、あらためて残高の0.3％を上乗せした金額になるよう更新する。
	*/
	UPDATE [dokoQL].[dbo].[口座_t]
	  SET [残高] = ([残高] - 3000) * 1.003
	  WHERE [口座番号] IN ('0652281','1026413','2239710');

### 算術演算子

#### 加算
	+	数値+数値
		日付+数値  <= SQLserverでは、DATEADD (DAY , 数値, 日付 )

		事例　180日後を求める。
		/* LEVEL4-37
			口座テーブルから、更新日が2016年以前のデータを対象に、口座番号、更新日、通帳期限日を抽出する。
			通帳期限日は、更新日の180日後とする。
		*/
		SELECT [口座番号],[更新日]
		--	,[更新日]+180 AS [通帳期限日]  -- 正答はこれだがSQL Serverではエラーになる。
		  	,DATEADD(DAY,180,[更新日]) AS [通帳期限日]  -- SQL Serverでは加算でなく関数を使う。
		  FROM [dokoQL].[dbo].[口座_t]
		  WHERE [更新日] < '2017-01-01';

#### 減算
	-	数値-数値
		日付-数値
		日付-日付  <= SQLserverでは、DATEDIFF ( datepart , startdate , enddate )  

#### 乗算
	*	数値*数値

#### 除算
	/	数値/数値

#### 剰余
  % 被除数%除数 戻り値が剰余になる。

#### 連結
	||	文字列||文字列  <= SQLserverでは、連結演算子はプラス記号 (+)

		事例　文字列連結する
			/* LEVEL4-38
			口座テーブルから、種別が「別段」のデータについて、ロ座番号と名義を抽出する。
			ただし、名義の前に「カ）」を付記すること。
			*/
			SELECT [口座番号]
			--     ,'カ）' || [名義] AS [名義]     -- 正答はこれだがSQL Serverではエラーになる。
				   ,'カ）' + [名義]  AS [名義] -- SQL Serverでは加算を使う。
                                      -- 列が数値型だと+は加算とみなされるので、
                                      -- 後で出てくるCAST関数で文字型に変換する。
			  FROM [dokoQL].[dbo].[口座_t]
			  WHERE [種別] = '3'

### CASE演算子

#### CASE演算子の利用構文(1)
		CASE 評価する列や式
		   WHEN 値1 THEN 値1の時に返す値
		  (WHEN 値2 THEN 値2の時に返す値) …
		  (ELSE デフォルト値)
		END

	実例
	/*
	  費目列と出金額列に新たに'出費の分類'列を加える。
	  出費の分類にセットする値は費目の値に従ってリテラルをセットする。
	  処理対象とするのは出金額が0を超えるレコードとする。
	*/

	SELECT [費目],[出金額],    -- <== 最後のカンマは新しい列(NULL)にASで列名を付けるため必要
			CASE [費目]
			   WHEN '居住費' THEN '固定費'
			   WHEN '水道光熱費' THEN '固定費'
			   ELSE '変動費'
			END
		AS [出費の分類]
		FROM [dokoQL].[dbo].[家計簿_tbl]
			WHERE [出金額] > 0


#### CASE演算子の利用構文(2)
		CASE
		   WHEN 条件1 THEN 条件1の時に返す値
		  (WHEN 条件2 THEN 条件2の時に返す値) …
		  (ELSE デフォルト値)
		END

	実例
	/*
	  費目列と、入金額列と、新たな'収入の分類'列を加て表示する。
	  収入の分類にセットする値は入金額の値に従ってリテラルをセットする。
	  処理対象とするのは入金額が0を超えるレコードとする。
	*/

	SELECT [費目],[入金額],    -- <== 最後のカンマは新しい列(NULL)にASで列名を付けるため必要
			CASE
			   WHEN [入金額] < 5000 THEN 'お小遣い'
			   WHEN [入金額] < 100000 THEN '一時収入'
			   WHEN [入金額] < 300000 THEN '給料出たー！'
			   ELSE '想定外の収入です！'
			END
		AS [収入の分類]
		FROM [dokoQL].[dbo].[家計簿_tbl]
			WHERE [入金額] > 0

	実例
	/*
	　　年齢を10歳毎の年代に括り、
	　　性別記号を翻訳して、
	　　年代と性別翻訳をコロンでつないで
	　「属性」として表示する
	*/
	SELECT RTRIM([メールアドレス]) AS [メールアドレス]
		,
		 CASE
			WHEN [年齢] BETWEEN 20 AND 29 THEN '20代'
			WHEN [年齢] BETWEEN 30 AND 39 THEN '30代'
			WHEN [年齢] BETWEEN 40 AND 49 THEN '40代'
			WHEN [年齢] BETWEEN 50 AND 59 THEN '50代'
		 END
		+ ':' +
		 CASE
			WHEN [性別] = 'M' THEN '男性'
			WHEN [性別] = 'F' THEN '女性'
		 END
	/* 別解
		 CASE  [性別]
			WHEN 'M' THEN '男性'
			WHEN 'F' THEN '女性'
	*/	 END
		AS [属性]
	  FROM [dokoQL].[dbo].[回答者_tbl];

	実例
	/*
	  メールアドレスの下２桁で国を識別し、
	  国名にセットする
	*/
	UPDATE [dokoQL].[dbo].[回答者_tbl]
		SET [国名] = CASE SUBSTRING ( RTRIM([メールアドレス])
									, LEN(
										RTRIM([メールアドレス]))-1
									, 2)
					   WHEN 'jp' THEN	'日本'
			   		   WHEN 'uk' THEN	'イギリス'
			   		   WHEN 'cn' THEN	'中国'
			   		   WHEN 'fr' THEN	'フランス'
			   		   WHEN 'vn' THEN	'ベトナム'
					END

	実例
	/*
		事由区分ごとの入室回数を取得する。
		その際、事由区分はわかりやすい表記にすること
	*/
	SELECT
		CASE [事由区分]
				WHEN '1' THEN 'メンテナンス'
				WHEN '2' THEN 'リリース作業'
				WHEN '3' THEN '障害対応'
				ELSE 'その他'
		END AS [事由]
	   ,COUNT(*) AS [入室回数]
	  FROM [dokoQL].[dbo].[入退室管理_tbl]
	  GROUP BY [事由区分]
	  ORDER BY [事由区分] ASC;

#### ユーザ定義関数
		あらかじめ用意された関数ではなく、
		必要とする処理を自分で記述し作成した関数

#### ストアドプロシージャ
		実行する複数のSQL文をまとめプログラムのようなものとしてDBMS内に保存し、
		データベースの外部から呼び出すもの

#### Transact-SQL
		ユーザ定義関数やストアドプロシージャを記述するときの専用言語。
		SQL ServerやPL/SQLで用いられる。

#### 文字列関数
	LENGTH (文字列を表す列)	 → 文字列の長さを表す数値
	LEN (文字列を表す列)	 → 文字列の長さを表す数値。SQLServer

	事例
	  10文字(10バイト)以下のメモだけを取得する
		SELECT [メモ], LEN([メモ]) AS [メモの長さ]
			FROM [dokoQL].[dbo].[家計簿_tbl]
				WHERE LEN([メモ]) <= 10;


	TRIM (文字列を表す列)	 → 左右から空白を除去した文字列。★SQLServerはサポートしてない
	LTRIM (文字列を表す列)	 → 左側の空白を除去した文字列。SQLServer
	RTRIM (文字列を表す列)	 → 右側の空白を除去した文字列。SQLServer


	REPLACE (置換対象の列, 置換前の部分文字列, 置換後の部分文字列 )	 → 置換処理された後の部分文字列。SQLServer

	事例
	  メモ列の'購入'を'買った'に置き換える
		UPDATE  [dokoQL].[dbo].[家計簿_tbl]
				SET [メモ] = REPLACE([メモ],'購入','買った');

	事例
	/*　[文字]のうち空白を含まない文字数を[置換後文字列]に表示する
	    [文字]のうち空白を含まない文字数を[置換後文字数]に表示する
	*/
		SELECT [文字]
			, REPLACE([文字], '　', '') AS [置換後文字列]  -- ★置換前の[文字]列に含まれた全角空白を取り除く
		    , LEN ( REPLACE([文字], '　', '')) AS [置換後文字数]
		  FROM [dokoQL].[dbo].[受注_tbl];


	SUBSTRING (文字列を表す列, 抽出を開始する位置, 抽出する文字の数 )	 → 抽出された部分文字列。SQLServer
	SUBSTR (文字列を表す列, 抽出を開始する位置, 抽出する文字の数 )	 → 抽出された部分文字列

	事例
	  費目列の１～３文字目に'費'の文字があるものだけを抽出する
		SELECT * FROM [dokoQL].[dbo].[家計簿_tbl]
				WHERE SUBSTRING([費目],1,3) LIKE '%費%';
		/* ★LIKE演算子の文法は　式 LIKE パターン文字列 。
		     関数で切り出した文字列をLIKE演算子のパターン文字列と一致しているか照合している */

	事例
		/* LEVEL4-43
			口座テーブルから、残高の桁数が4桁以上で、1,000円未満の端数がないデータを抽出する。
			ただし、どちらの条件も文字数を求める関数を使って判定すること。
		*/
		SELECT *
		  FROM [dokoQL].[dbo].[口座_t]
		 WHERE LEN(CAST([残高] AS VARCHAR)) >= 4
		   AND SUBSTRING(CAST([残高] AS VARCHAR), LEN(CAST([残高] AS VARCHAR))-2, 3) = '000'
		   -- 例　残高が654,321円ならば、SUBSTRINGS('654321', 6-2, 3) --　左から4文字目を開始位置にして右へ3文字を取り出している
		   -- SUBSTRING( 文字列式, 文字の開始位置, 文字数) 戻り値:文字列の指定された部分文字列
		   -- LEN(文字列式) 戻り値:文字列式の字数
		   -- CAST(文字列式 AS 変換後のデータ型)　戻り値:データ型の式を別のデータ型に変換
		   -- 別解
		--   AND RIGHT(CAST([残高] AS VARCHAR), 3) = '000'
		   -- RIGHT(文字列式,文字数) 戻り値:文字列の右から指定された数の部分文字列

	CONCAT(文字列, 文字列 [, 文字列…] )	 → 連結後の文字列
	文字列1 + 文字列2	 → 連結後の文字列。SQLServer


#### 数値の関数
	ROUND ( 数値を表す列, 有効とする桁数 )	→ 数値を丸めた値。★偶数丸めであり、四捨五入ではない。
  /* 有効とする桁数
      1  小数第2位の値を四捨五入して、小数第1位の値にする("小数第2位の位で四捨五入する"と同義)
         例 0.05 -> 0.1
      0  小数第1位の値を四捨五入して、一の位にする("小数点以下を四捨五入する"と同義)
         例 0.5 ->　1
     -1 一の位の値を四捨五入して、十の位にする("一の位で四捨五入する"と同義)
         例 5 ->　10
  */

	/*★	「偶数丸め」は四捨五入とほぼ同一ですが、次の１点が違います。
			「丸め単位の丁度まんなかで、どっちつかずの場合は、偶数側を採用する」
			したがって、1.25 を 0.01 の位で丸めると 1.2 になり、1.35 は 1.4 になります。
	*/

	事例
	  出金額を百円単位に丸める（10円の桁で丸めて100円の桁に切り上げるか、切り捨てる）
		SELECT [出金額], ROUND([出金額],-2) AS [百円単位の出金額]
			FROM [dokoQL].[dbo].[家計簿_tbl]


	TRUNC ( 数値を表す列, 有効とする桁数 )	→ 切り捨てた値。★SQLServerはサポートしてない
	FLOOR ( 数値を表す列 )	→ SQLServerがサポート。戻り値:指定された数値式以下の最大の整数。つまり小数点部分の切り捨て。
	-- Microsoft SQL 数値関数 - floor
	-- （Docs＞SQL＞リファレンス＞Transact-SQL (T-SQL) リファレンス＞FLOOR）
	--  https://docs.microsoft.com/ja-jp/sql/t-sql/functions/floor-transact-sql?view=sql-server-ver15

	事例
	  出金額千円単位（小数点切り捨て）で表示する
		SELECT [出金額], FLOOR([出金額]/1000) AS [千円単位の出金額]
			FROM [dokoQL].[dbo].[家計簿_tbl]


	POWER ( 数値を表す列, 何乗するかを指定する数値 )	→ 数値を指定した回数だけ乗じた結果

	事例
	  出金額の３乗を表示する
		SELECT [出金額], POWER([出金額], 3) AS [出金額の3乗]
			FROM [dokoQL].[dbo].[家計簿_tbl]
			WHERE[出金額]<1000

#### 日付の関数
	CURRENT_DATE	→ 現在の日付★SQLServerはサポートしてない
	CURRENT_TIME	→ 現在の時刻★SQLServerはサポートしてない
	GETDATE()	→ SQL Server のインスタンスが実行されているコンピューターの日付と時刻
	CURRENT_TIMESTAMP	→ 引数不要
	SYSDATETIME ( )	→1秒未満の有効桁数で比較するとGETDATEより精度が高い。date型とtime型の変数に割り当てられる。

	事例
	  日付を自動取得して、日付部分のみ表示する
		SELECT GETDATE() AS [GETDATE],
				DATENAME(day,GETDATE()) AS [DATEPART]

	  日付を自動取得して、登録する
		INSERT INTO [dokoQL].[dbo].[家計簿_tbl]
			VALUES (GETDATE(), '食費','ドーナツを買った',0,260);
		/* GETDATE()の戻り値を表示すると日付型のYYYY-MM-DD HH:MM:SSで表示されている。
		   家計簿_TBLの１列目は[日付]で日付型なので、GETDATE()の値をそのまま入れられる。
		*/

	事例 Microsoft SQL Serverより
	A. 現在のシステム日付と時刻を取得する
		SELECT SYSDATETIME()  -- 2007-04-30 13:10:02.0474381  
		    ,SYSDATETIMEOFFSET()  -- 2007-04-30 13:10:02.0474381 -07:00
		    ,SYSUTCDATETIME()     -- 2007-04-30 20:10:02.0474381  
		    ,CURRENT_TIMESTAMP    -- 2007-04-30 13:10:02.047  
		    ,GETDATE()            -- 2007-04-30 13:10:02.047  
		    ,GETUTCDATE();        -- 2007-04-30 20:10:02.047  

	B. 現在のシステム日付を取得する
		SELECT CONVERT (DATE, SYSDATETIME())      -- 2007-05-03
		    ,CONVERT (DATE, SYSDATETIMEOFFSET())  -- 2007-05-03
		    ,CONVERT (DATE, SYSUTCDATETIME())     -- 2007-05-04
		    ,CONVERT (DATE, CURRENT_TIMESTAMP)    -- 2007-05-03
		    ,CONVERT (DATE, GETDATE())            -- 2007-05-03
		    ,CONVERT (DATE, GETUTCDATE());        -- 2007-05-04

	C. 現在のシステム時刻を取得する
		SELECT CONVERT (TIME, SYSDATETIME())     -- 13:18:45.3490361
		    ,CONVERT (TIME, SYSDATETIMEOFFSET()) -- 13:18:45.3490361
		    ,CONVERT (TIME, SYSUTCDATETIME())    -- 20:18:45.3490361
		    ,CONVERT (TIME, CURRENT_TIMESTAMP)   -- 13:18:45.3470000
		    ,CONVERT (TIME, GETDATE())           -- 13:18:45.3470000
		    ,CONVERT (TIME, GETUTCDATE());       -- 20:18:45.3470000

	事例
		/* LEVEL4-47
			口座テーブルから更新日が2018年以降のデータを抽出する。
			その際、更新日は「2018年01月01日」のような形式で抽出すること。
		*/
		  SELECT [口座番号],[名義],[種別],[残高],   -- 正答は前ゼロがつく。
		       SUBSTRING(CAST([更新日] AS VARCHAR), 1, 4) + '年' +
			   SUBSTRING(CAST([更新日] AS VARCHAR), 6, 2) + '月' +
			   SUBSTRING(CAST([更新日] AS VARCHAR), 9, 2) + '日' AS [更新日]
		 FROM [dokoQL].[dbo].[口座_t]
		 WHERE [更新日] >= '2018-01-01'

		  SELECT [口座番号],[名義],[種別],[残高],   -- 私の回答では、前ゼロがつかない。
				  CAST( YEAR([更新日]) AS VARCHAR) + '年'
				+ CAST( MONTH([更新日]) AS VARCHAR) + '月'
				+ CAST( DAY([更新日]) AS VARCHAR) + '日'
				AS [更新日]
		  FROM [dokoQL].[dbo].[口座_t]
		  WHERE [更新日] > '2017-12-31'

		  -- Microsoft SQL 日付と時刻のデータ型および関数 (Transact-SQL)
		  -- （Docs＞SQL＞リファレンス＞Transact-SQL (T-SQL) リファレンス＞関数＞日時）
		  -- 日付と時刻の要素を返す関数
		  -- https://docs.microsoft.com/ja-jp/sql/t-sql/functions/date-and-time-data-types-and-functions-transact-sql?view=sql-server-ver15#functions-that-return-date-and-time-parts
		  /*
				DATENAME ( datepart , date )	指定された日付の指定された datepart を表す文字列を返します。
				DATEPART ( datepart , date )	指定された date の指定された datepart を表す整数を返します。
				DAY ( date )	指定された date の日の部分を表す整数を返します。
				MONTH ( date )	指定された date の月の部分を表す整数を返します。
				YEAR ( date )	指定された date の年の部分を表す整数を返します。
		  */

#### 変換の関数
	CAST (変換する値 AS 変換する型 )	→ 変換後の値

	事例
	  出金額に単位の"円"を連結して表示する
		SELECT CAST ( [出金額] AS VARCHAR(20) ) + '円'
			FROM [dokoQL].[dbo].[家計簿_tbl]
		/* ★出金額はINT型なので、数値と文字という型の異なる値を'+'演算子で連結することはできない。
		  そこで、CAST()で型を数字から文字へ変換して連結する
		*/

	  日付を自動取得して、書式に合わせて表示する
		SELECT
			CURRENT_TIMESTAMP                      AS [CURRENT_TIMESTAMP], -- 2020-08-18 11:03:20.487
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,111) AS [yyyy/mm/dd],        -- 2020/08/18
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,112) AS [yyyymmdd],          -- 20200818
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,120) AS [yyyy-mm-dd hh:mm:ss]   -- 2020-08-18 11:03:20
		/* ★CONVERTは、日付を文字列に変換するために使用している。
		     ゆえに、出力値の型は文字列で、日付ではない。
		*/


	COALESCE ( 列や式1, 列や式2, 列や式3, …)	→ 引数のうち、最初に現れたNULLでない引数。コアレス関数
  /* 列の値がnullの時に、予め設定しておいた値を返す関数 */

	事例
		SELECT COALESCE ( 'A'  , 'B'  , 'C'  );	-- 結果はA
		SELECT COALESCE ( NULL , 'B'  , 'C'  );	-- 結果はB
		SELECT COALESCE ( NULL , 'B'  , NULL );	-- 結果はB
		SELECT COALESCE ( NULL , NULL , 'C'  );	-- 結果はC

	事例
	  メモ列の値がNULLならば、注意喚起メッセージを表示する
	  	INSERT INTO [dokoQL].[dbo].[家計簿_tbl]
			VALUES ('2018-02-11', '教養娯楽費',NULL,0,2800);
	  	INSERT INTO [dokoQL].[dbo].[家計簿_tbl]
			VALUES ('2018-02-14', '交際費',NULL,0,5000);
		SELECT [日付], [費目]
			  ,COALESCE ([メモ], '(メモ欄はNULLです)') AS [メモ]
			  ,[入金額], [出金額]
			FROM [dokoQL].[dbo].[家計簿_tbl]

	/* LEVEL4-48
		口座テーブルから更新日を抽出する。
		更新日が登録されていない場合は、「設定なし」と表記すること。
	*/
	  SELECT COALESCE ( CAST([更新日] AS VARCHAR) , '設定なし' ) AS [更新日]
	  FROM [dokoQL].[dbo].[口座_t];

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 集計とグループ化

### 集計関数の特徴
	集計関数は、検索対象の全行をひとまとめに扱い、１回だけ集計処理をおこなう。
	集計関数の結果は、必ず１行になる
	集計関数は、SELECT文の選択列リストか、SELECT文のORDER BY句か、SELECT文のHAVING句の中で使用する。
	集計関数は、SELECT文のWHERE句の中では使用できない。


	出金額を集計するケース
	例　SELECT SUM( [出金額] ) AS [出金額の合計]
		  FROM [dokoQL].[dbo].[家計簿_tbl]

### 代表的な集計関数
	分類	関数名	説明
	集計	SUM()	各行の値の合計を求める。DISTINCTを指定すると重複行を除いて集計できる
	　〃	MAX()	各行の値の最大値を求める
	　〃	MIN()	各行の値の最小値を求める
	　〃	AVG()	各行の値の平均値を求める。DISTINCTを指定すると重複行を除いて平均できる
	計数	COUNT()	行数をカウントする。DISTINCTを指定すると重複行を除いてカウントできる
	/* ★COUNT(*)は、 単純にNULLの行を含めて行数をカウントする
	     COUNT(列)は、指定列の値がNULLの行を含めないで行数をカウントする（指定列がNULL値ならばカウントしない）
	*/

	事例
	/* リスト6-2 さまざまな集計 */
	SELECT
	 SUM( [出金額] ) AS [合計出金額]
	,AVG( [出金額] ) AS [平均出金額]
	,MAX( [出金額] ) AS [最も大きな出金額]
	,MIN( [出金額] ) AS [最も少額の出金額]
	FROM [dokoQL].[dbo].[家計簿_tbl]

	事例
	/* LEVEL5-52
		口座テーブルから、名義の最大値と最小値をそれぞれ求める。
	*/
	SELECT MAX(  [名義] ) AS [最大] -- 名義にはカナ氏名が入っているので、
		  ,MIN(  [名義] ) AS [最小] -- 50音順に並べた先頭と最終が表示される
	  FROM [dokoQL].[dbo].[口座_t];

	事例
	/* リスト6-3 食費の行数を数える */
	SELECT
	 COUNT( * ) AS [食費の行数]
	FROM [dokoQL].[dbo].[家計簿_tbl]
	WHERE [費目] = '食費'

	事例
	/* リスト6-3plus 重複した値を除いた集計 */
	SELECT
	 COUNT( DISTINCT [費目] ) AS [費目の種類数]
	FROM [dokoQL].[dbo].[家計簿_tbl]

	事例
	/* リスト6-5 NULLをゼロとして平均を求める */
	SELECT
	 AVG( COALESCE( [出金額],0) ) AS [出金額の平均]
	FROM [dokoQL].[dbo].[家計簿_tbl]

  事例
  /* ドリルC LEVEL5 #52 新しい列にCASEで値を入れ、新しい列を集計する。

    ある洞窟に存在する「力の扉」は、キャラクターの HP によって開けることのできる扉の数が決まっている。
    次の条件によってその数が決まるとき、現在のパーティーで開ける ことのできる扉の数を求める。
    ・HP が 100未満のキャラクター 1枚
    ・HP が 100 以上 150 未満のキャラクター 2枚
    ・HP が 150 以上 200 未満のキャラクター 3枚
    ・HP が 200 以上のキャラクター 5枚
  */
    SELECT SUM(
  		CASE
  			WHEN [HP] < 100 THEN 1
  			WHEN [HP] <= 100 AND [HP] < 150 THEN 2
  			WHEN [HP] <= 150 AND [HP] < 200 THEN 3
  			WHEN [HP] <= 200 THEN 5
  	    END
  		   ) AS [扉の数] -- 元のDBにはなかった新しい列
   FROM [dokoQL].[dbo].[パーティ_t];

 事例
 /* ドリルC LEVEL6 #53 全体の合計をもとめて、特定行の値の比率を求める。

    勇者の現在の HP が、パーティー全員の HP の何%に当たるかを求めるため、
    適切な列を用いて次の別名で抽出する。
    ただし、割合は小数点第2位を四捨五入し、小数点第1 位まで求めること。
    ・なまえ ・現在の HP ・パーティーでの割合
 */
    SELECT [名称] AS [なまえ]
         ,[HP] AS [現在のHP]
       ,ROUND(CAST([HP] AS NUMERIC) -- WHERE句で抽出する行の[HP]
        / ( SELECT SUM([HP])        -- DB全体の[HP]の合計
          FROM [dokoQL].[dbo].[パーティ_t])
        * 100, 1) AS [パーティーでの割合]
     FROM [dokoQL].[dbo].[パーティ_t]
     WHERE [職業コード] = '01'; -- 勇者

### グループ化して集計する基本構文
	SELECT グループ化の基準列名…,集計関数
	FROM テーブル名
	(WHERE 絞り込み条件)
	GROUP BY グループ化の基準列名…

	事例
/* リスト6-7 費目でグループ化してそれぞれの合計を求める */
	SELECT
		[費目]
		,SUM( [出金額] ) AS [費目別の出金額合計]
	FROM [dokoQL].[dbo].[家計簿_tbl]
	GROUP BY [費目]  -- ★費目列でグループ化する


### グループ化してから絞り込む基本構文
	SELECT グループ化の基準列名…,集計関数
	FROM テーブル名
	(WHERE 『元の表に対する』絞り込み条件)
	GROUP BY グループ化の基準列名…
	HAVING 集計結果に対する絞り込み条件
	/* ★WHERE句は、 元のテーブルを検索するときに絞り込む。対象外を結果表に出さない
	     HAVING句は、集計後に集計結果を絞り込む
	*/

	事例
	/* リスト6-9 集計結果で絞り込む */
	SELECT
		[費目]
		,SUM( [出金額] ) AS [費目別の出金額合計]
	FROM [dokoQL].[dbo].[家計簿_tbl]
	GROUP BY [費目]
	HAVING SUM( [出金額] ) > 0

	事例
	/* 今月の出費のうち、平均が5,000円以上の費目と、その最大値を知りたい */
	SELECT
		[費目]
		,MAX ( [出金額] ) AS [最大出金額]
	FROM [dokoQL].[dbo].[家計簿_tbl]
	GROUP BY [費目]
	HAVING AVG ( [出金額] ) >= 5000;


### SELECT文の全貌
	SELECT 選択列リスト              -- 列名、列名を含む関数、ASでつけた別名を書く。
	FROM テーブル名
	(WHERE 条件式)
	(GROUP BY グループ化列名)
	(HAVING 集計結果に対する条件式)  -- HAVING句には列名か列名を含む関数を書く。ASでつけた別名ではない。
	(ORDER BY 並べ替え列名)          -- ORDER句にはASでつけた別名を書いてよい

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 副問合せ

### 副問い合わせとは
	ほかのSQL文の一部分として登場するSELECT文。
	丸かっこでくくって記述する。
	SQL文がネスト構造になっている。
	副照会、サブクエリとも呼ぶ。
	副問合せは、実行すると何らかの値に置き換わる。
	副問合せは、より内側にあるものから外側に向かって順に評価されていく

	事例
	/* 最も大きな出費の費目と金額を求める。
	   その１．副問い合わせを使わない方法。２回SQL実行するので、１回目の結果のメモと２回目のSQL修正が必要。
	*/
		SELECT MAX([出金額]) FROM 家計簿;  <==①出金額をメモする。

		SELECT 費目,出金額 FROM 家計簿
			WHERE [出金額] = XXXXX;        <==②XXXをメモした出金額に書き換える。

	/* 最も大きな出費の費目と金額を求める。
	   その２．副問い合わせを使う方法
	*/
		SELECT 費目,出金額 FROM 家計簿
			WHERE [出金額] = ( SELECT MAX([出金額]) FROM 家計簿 );
	/*
	  ★その１のSQLの①のXXXを、②に書き換えたのと同じ。
	*/

### 副問い合わせはネスト構造できる
	SQL文は内部に複数の副問い合わせを持つことができる。
		UPDATE (SELECT …)
			WHERE (SELECT …)
	副問い合わせの中にさらに別の副問い合わせを記述できる。
		UPDATE (SELECT …
				(SELECT …) )

### 副問い合わせの動作順序
	内側にあるSELECT文が実行され結果になる。
	外側のSQLが実行される。

### 副問い合わせの３パターン
	SQLが、副問い合わせの結果を用いるのは３パターンに分類できる。
	１．単一行副問合せ。SQLが、単一の値の代わりとして、副問い合わせの検索結果を用いるパターン
	２．複数行副問合せ。SQLが、複数の値の代わりとして、副問い合わせの検索結果を用いるパターン
	３．結果がn行m列の表形式になる副問合せ。SQLが、表の値の代わりとして、副問い合わせの検索結果を用いるパターン
		※単一の値：スカラー。値が１個だけ返ってくる。
		※複数の値：ベクター。値が１次元配列。
		※表の値：マトリックス。値が２次元配列。


### 単一行副問合せ（単一の値の代わりに副問い合わせの検索結果を用いる）

#### 単一行副問合せとは
		検索結果が1行1列の1つの値となる副問合せ。
		SELECT文の選択列リストやFROM句、UPDATE文のSET句、
		また1つの値との判定を行うWHERE句の条件式などに記述することができる。

	事例
	/* SET句で副問い合わせを利用する
	*/
	UPDATE [dokoQL].[dbo].[家計簿集計_tbl]
		SET [平均] = ( SELECT AVG( [出金額] )	-- ①家計簿アーカイブから食費の平均値を取得する
						FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
						WHERE [出金額] > 0
							AND [費目] = '食費')
		WHERE [費目] = '食費'					-- ②家計簿集計の食費行平均値列を①の値に更新する

	事例
	/* アーカイブの食費の出金額を集計して、集計テーブルの食費の合計を更新したい
	*/
	UPDATE [dokoQL].[dbo].[家計簿集計_tbl]
		SET [合計] = ( SELECT SUM( [出金額] )
						FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
						WHERE [出金額] > 0
							AND [費目] = '食費' )
		WHERE  [費目] = '食費';

	事例
	/* SELECT句で副問い合わせを利用する
	*/
	SELECT	[日付]
			,[メモ]
			,[出金額]
			,( SELECT [合計]					-- ①家計簿集計から食費行合計列の値を取得する
				FROM [dokoQL].[dbo].[家計簿集計_tbl]
				WHERE [費目] = '食費') AS [過去の合計額]
		FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
		WHERE [費目] = '食費'					-- ②家計簿アーカイブの費目列が食費の行に①の値を表示する

	事例
	/* アーカイブの1月と12月の出金額の合計をそれぞれ知りたい
	*/
	SELECT	 G.[タイトル]
			,G.[出金額計]
		FROM (
				SELECT	'合計01月' AS [タイトル]
						,SUM ( [出金額] ) AS [出金額計]
					FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
					WHERE [日付] >= '2018-01-01'
						AND [日付] <= '2018-01-31'
				UNION
				SELECT	'合計12月' AS [タイトル]
						,SUM ( [出金額] ) AS [出金額計]
					FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
					WHERE [日付] >= '2017-12-01'
						AND [日付] <= '2017-12-31'
			) AS G;


### 複数行副問合せ（複数の値の代わりに副問い合わせの検索結果を用いる）

#### 複数行副問合せとは
		検索結果がn行1列の複数の値となる副問合せ（nは1以上）。
		複数の値との判定を行うWHERE句の条件式や、SELECT文のFROM句に記述することができる。
		複数行副問合せは、IN、ANY、ALL演算子とあわせてよく用いられる。
		複数行副問合せの結果にNULLが含まれると、NOT IN、<>ALL演算子の評価結果もNULLになる。

	事例
	/* INで副問合せを利用する(その１)
	*/
	SELECT *
		FROM [dokoQL].[dbo].[家計簿集計_tbl]
		WHERE [費目] IN ( SELECT DISTINCT [費目] FROM [dokoQL].[dbo].[家計簿_tbl] ); --家計簿_TBLに載っている費目で取得する
	   /* ★費目を指定しなくても取得できる。記述ミスや費目の追加変更に対応できる */

	事例
   	/* INで副問合せを利用する(その２)
   	*/
     /*	問題 7-3 設問2
     	頭数集計テーブルで、飼育頭数が多いほうから3つの都道府県で飼育されているデータを
     	個体識別テーブルより抽出する。
     	抽出する項目は、都道府県名、個体識別番号、雌雄とする。
     	ただし、雌雄コードではなく「雄」「雌」の日本語表記とする。
     */

     /* STEP1
     	頭数集計テーブルで、飼育頭数が多いほうから3つの都道府県で飼育されているデータ
     */
     SELECT [飼育県]
     	FROM [dokoQL].[dbo].[頭数集計_tbl]
     	ORDER BY [頭数] DESC	-- 頭数の多い順
     	OFFSET 0 ROWS			-- 先頭から選ぶ
     	FETCH NEXT 3 ROWS ONLY;	-- 3行を取得

     /* STEP2
     	個体識別テーブルより抽出する。
     	抽出する項目は、都道府県名、個体識別番号、雌雄とする。
     	ただし、雌雄コードではなく「雄」「雌」の日本語表記とする。
     */
     SELECT	 [飼育県] AS [都道府県名]
     		,[個体識別番号]
     		,CASE [雌雄コード]
     			WHEN '1' THEN '雄'
     			WHEN '2' THEN '雌'
     		END AS [雌雄]
     	FROM [dokoQL].[dbo].[個体識別_tbl];

     /* STEP3
     	最終表示はSTEP2で、STEP1はそのための抽出になるので、
     	副問合せはSTEP1と考えて、合体させる。
     	複数行副問合せのIN演算子を使える。
     */
     SELECT	 [飼育県] AS [都道府県名]
     		,[個体識別番号]
     		,CASE [雌雄コード]
     			WHEN '1' THEN '雄'
     			WHEN '2' THEN '雌'
     		END AS [雌雄]
     	FROM [dokoQL].[dbo].[個体識別_tbl]
     	WHERE [飼育県] IN  -- ★主問合せの飼育県と副問合せの飼育県TOP3(n)を照合する。
     					(
     						SELECT [飼育県]
     						FROM [dokoQL].[dbo].[頭数集計_tbl]
     						ORDER BY [頭数] DESC
     						OFFSET 0 ROWS
     						FETCH NEXT 3 ROWS ONLY
     					);

	事例
  	/* INで副問合せを利用する(その３)
  	*/
    /*	問題 7-3 設問3
    	個体識別テーブルには母牛についてもデータ登録されており、
    	母牛が乳用種である牛の一覧を個体識別テーブルより抽出したい。
    	抽出する項目は、個体識別番号、品種、出生日、母牛番号とする。
    	なお、品種はコードではなく「乳用種」「肉用種」「交雑種」の日本語表記とする。
    */

    /* STEP1
    	抽出する項目は、個体識別番号、品種、出生日、母牛番号とする。
    	なお、品種はコードではなく「乳用種」「肉用種」「交雑種」の日本語表記とする。
    */
    SELECT [個体識別番号]
          ,CASE [品種コード]
    		WHEN '01' THEN '乳用種'
    		WHEN '02' THEN '肉用種'
    		WHEN '03' THEN '交雑種'
    	   END AS [品種]
          ,[出生日]
          ,[母牛番号]
      FROM [dokoQL].[dbo].[個体識別_tbl];

    /* STEP2
    	母牛が乳用種である牛の一覧を個体識別テーブルより抽出したい。
    	※個体が母牛か否かは、一旦置いといて、乳用種の個体識別番号を抽出する
    */
    SELECT [個体識別番号]
      FROM [dokoQL].[dbo].[個体識別_tbl]
      WHERE [品種コード] = '01';

    /* STEP3
    	STEP1の母牛番号でSTEP2の副問合せをすれば、乳用種を抽出できるので、合体する。
    	複数行副問合せのIN演算子を使える。
    */
    SELECT [個体識別番号]
          ,CASE [品種コード]
    		WHEN '01' THEN '乳用種'
    		WHEN '02' THEN '肉用種'
    		WHEN '03' THEN '交雑種'
    	   END AS [品種]
          ,[出生日]
          ,[母牛番号]
      FROM [dokoQL].[dbo].[個体識別_tbl]
      WHERE [母牛番号] IN  -- ★主問合せの母牛番号(1)と副問合せの乳用種の個体識別番号(n)を照合する。
    	  (	SELECT [個体識別番号]
    		  FROM [dokoQL].[dbo].[個体識別_tbl]
    		  WHERE [品種コード] = '01'
    		);

	/* NOT IN演算子で、２表の列のアンマッチ項目を取り出す
		家計簿のうち、今月初めて発生した費目を知りたい
	*/
	SELECT DISTINCT [費目]
		FROM [dokoQL].[dbo].[家計簿_tbl]
		WHERE [費目] NOT IN ( SELECT [費目]
								FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl] );

	事例
	/* 	ALLで副問合せを利用する
		家計簿テーブルの今年の給料で、アーカイブの去年の給料よりも高い額があれば知りたい
	*/
	SELECT * FROM [dokoQL].[dbo].[家計簿_tbl]
		WHERE [費目] = '給料'
			AND [入金額]　> ALL ( SELECT [入金額]
									FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
									WHERE [費目] = '給料');

	事例
	/* ANYで副問合せを利用する
	*/
	SELECT * FROM [dokoQL].[dbo].[家計簿_tbl]
	 WHERE [費目] = '食費'
	       AND [出金額] < ANY ( SELECT [出金額] FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
								 WHERE [費目] = '食費' );
	/* ★家計簿アーカイブの食費の出金額を取得して得られる複数の値と
		家計簿テーブルの食費の出金額を比較して
		『どれよりも小さい』行を取得する */

#### 同じ意味になる演算子
	NOT IN と <>ALL … すべての値と一致しないことを判定する演算子
	IN     と  =ANY … いずれかの値と一致することを判定する演算子

#### 副問合せがNULLを含んでいる場合
	NOT IN または <>ALL で判定する副問合せの結果にNULLが含まれると、全体の結果もNULLになる。
	そこで、絶対にNULLが含まれない状況を作る必要がある。

	副問合せの結果から確実にNULLを除外する方法
	実例
	/*
		副問合せからNULLを除外する(IS NOT NULL版)
	*/
	SELECT * FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
	 WHERE [費目] IN ( SELECT [費目] FROM [dokoQL].[dbo].[家計簿_tbl]
								 WHERE [費目] IS NOT NULL );

	/*
		副問合せからNULLを除外する(COALESCE版)
	*/
	SELECT * FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
	 WHERE [費目] IN ( SELECT COALESCE([費目],'不明') FROM [dokoQL].[dbo].[家計簿_tbl]);

#### 行値式と副問合せ
	行値式とは、複数の列の組み合わせによる条件式
	例 WHERE (A, B) IN ( SELECT C, D FROM …)


### 表の結果となる副問合せ（表の代わりに副問い合わせの検索結果を用いる）

#### 表形式の結果になる副問合せとは
		検索結果がn行m列の表となる副問合せ（n、mは1以上）
		SELECT文のFROM句やINSERT文などに記述することができる

	事例
	/*
		FROM句で副問合せを利用する。
		2個の表を併合し、併合した表の項目を集計する

	*/
	SELECT SUM (SUB.[出金額])  -- ②SUBの出金額を合計して表示する
		AS [出金額合計]

		FROM (  --　①家計簿とアーカイブを併合して3列の表を作りSUBと名付ける。ただし日付を1月に絞る。
				SELECT [日付], [費目], [出金額]
					FROM [dokoQL].[dbo].[家計簿_tbl]
					UNION
				SELECT [日付], [費目], [出金額]
					FROM [dokoQL].[dbo].[家計簿アーカイブ_tbl]
					WHERE [日付] >= '2018-01-01'
					AND [日付] <= '2018-01-31' )
			AS SUB;

	事例
	/*	リスト 7-12
		INSERT文で副問合せを利用する
		1回のINSERT文で複数行のデータを登録できる。
	*/
	INSERT INTO [dokoQL].[dbo].[家計簿集計_tbl]
				([費目], [合計], [平均], [回数])
	-- ★本来ならばINSERT文のVALES句で値を入れるが、特殊構文として続くSELECT文が副問合せとしてVALUES句の代わりをする
	SELECT [費目]
			,SUM([出金額])
			,AVG([出金額])
			,0
		FROM [dokoQL].[dbo].[家計簿_tbl]
			WHERE [出金額] > 0
			GROUP BY [費目];

	事例
	/* 今月の家計簿テーブルのデータをアーカイブに入れたい。
	*/
	INSERT INTO [dokoQL].[dbo].[家計簿アーカイブ_tbl]
	SELECT * FROM [dokoQL].[dbo].[家計簿_tbl];

### 相関副問合せ

#### 相関副問合せとは
		副問合せの内部から、主問い合わせの表や列を利用する副問合せ
		DBMSの負荷が大幅に増加するので乱用に注意

#### EXISTS 演算子
		EXISTS 演算子は、副問い合せの結果が存在するかを調べるときに使用します。
		このとき、副問い合せの結果が存在するとき真になります。

	活用パターン
	SELECT 列 FROM テーブル1
		WHERE EXISTS
		( SELECT * FROM テーブル2
			WHERE テーブル1.列 = テーブル2.列);

	事例
	/*
		今月使った費目（家計簿テーブルに登場する費目）についてのみ、
		合計金額を家計簿集計テーブルから抽出したい。
	*/
	SELECT [費目], [合計]
	  FROM [dokoQL].[dbo].[家計簿集計_tbl]  -- 主問い合わせ
	  WHERE EXISTS
		( SELECT * FROM [dokoQL].[dbo].[家計簿_tbl]
			WHERE [家計簿_tbl].[費目] = [家計簿集計_tbl].[費目] ); -- 主問合わせの[費目]を副問合せで使う

#### 副問合せのコツ
	それぞれの問合わせを独立したSQL文で作成する。
	メインの問合せのSQL文の中に、先ほどのSQL文を含めていく

	事例
	/* LEVEL6-60
		次の口座について、現在の残高と、取引日に発生した取引による入出金額それぞれの合計金額を取得する。
		取得には、選択列リストにて取引テーブルを副問い合わせするSELECT文を用いること。
		・口座番号：1115600、取引日：2017-12-28
	*/
	/*　口座テーブルの表示 */
	SELECT [口座番号],[名義],[種別],[残高],[更新日]
	  FROM [dokoQL].[dbo].[口座_t]
	  WHERE  [口座番号] = '1115600';

	/*　取引テーブルの表示 */
	SELECT [取引番号],[取引事由ID],[日付],[口座番号],[入金額],[出金額]
	  FROM [dokoQL].[dbo].[取引_t]
	  WHERE  [口座番号] = '1115600'
	    AND  [日付] = '2017-12-28';

	/*　私の回答 */
	SELECT [残高]
		 ,(SELECT SUM( [入金額] )
		          FROM [dokoQL].[dbo].[取引_t]
				 WHERE  [口座番号] = '1115600'
	               AND  [日付] = '2017-12-28'
			 ) AS [入金額合計]
		 ,(SELECT SUM( [出金額] )
		          FROM [dokoQL].[dbo].[取引_t]
				 WHERE  [口座番号] = '1115600'
	               AND  [日付] = '2017-12-28'
			 ) AS [出金額合計]
	  FROM [dokoQL].[dbo].[口座_t]
	  WHERE  [口座番号] = '1115600';


	/* LEVEL6-61
		これまで1回の取引で100万円以上の入金があった口座について、口座番号、名義、残高を取得する。
		ただし、WHERE句でIN演算子を利用した副問い合わせを用いること。
	*/
	/*　口座テーブルの表示 */
	SELECT [口座番号],[名義],[種別],[残高],[更新日]
	  FROM [dokoQL].[dbo].[口座_t];

	/*　取引テーブルの表示 */
	SELECT [取引番号],[取引事由ID],[日付],[口座番号],[入金額],[出金額]
	  FROM [dokoQL].[dbo].[取引_t]
	  WHERE [入金額] >= 1000000 ;

	/*　私の回答 */
	SELECT [口座番号]
	      ,[名義]
		  ,[残高]
	  FROM [dokoQL].[dbo].[口座_t]
	  WHERE [口座番号] IN (SELECT [口座番号]
							 FROM  [dokoQL].[dbo].[取引_t]
							 WHERE [入金額] >= 1000000
							);



/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 複数テーブルの結合

### テーブルＡとテーブルＢの結合
		SELECT 選択列リスト
		FROM   左表テーブルＡのテーブル名
		JOIN   右表テーブルＢのテーブル名
		ON     両テーブルの結合条件

	事例
	/* LEVEL7-66
		次の口座について、口座情報（口座番号、名義、残高）とこれまでの取引情報（日付、入金額、出金額）を一覧として抽出する。
		一覧は、取引の古い順に表示すること。
		・口座番号：0887132
	*/

	SELECT  K.[口座番号],K.[名義],K.[残高]
	       ,T.[日付],T.[入金額],T.[出金額]
	  FROM [dokoQL].[dbo].[口座_t] AS K
	  JOIN [dokoQL].[dbo].[取引_t] AS T
	    ON K.[口座番号] = T.[口座番号] -- ★ON には2個のテーブルをつなげるキーを記述する
	 WHERE K.[口座番号] = '0887132'    -- ★WHEREには2個のテーブルの列を使って抽出条件を記述する
	 ORDER BY T.[取引番号];

### 右表の結合条件列の重複
		つなぐべき右表の行が複数ある時、
		DBMSは左表の行を複製して結合する。
		結果表の行数は、もとの左表の行数より増える。

### 右表に結合相手がない場合
		右表に結合相手の行がない場合や、
		左表の結合条件の列がNULLの場合、
		結果結果から消滅する。
		消滅してはまずい場合に左外部結合left outer joinを使う。

### 内部結合と外部結合
		内部結合(inner join)：結合すべき相手の行が見つからない場合に行が消滅してしまう“通常の”結合
		外部結合(outer join)：結合すべき相手の行が見つからない場合に強制的に行を出力する結合

### 左外部結合：左表の全行を必ず出力する
		SELECT 選択列リスト
		FROM   左表テーブルＡのテーブル名
		LEFT JOIN   右表テーブルＢのテーブル名
		ON     両テーブルの結合条件

### 右外部結合：右表の全行を必ず出力する
		SELECT 選択列リスト
		FROM   左表テーブルＡのテーブル名
		RIGHT JOIN   右表テーブルＢのテーブル名
		ON     両テーブルの結合条件

	事例
	/* LEVEL7-69
		取引テープルのデータを抽出する。
		取引事由については「取引事由ID：取引事由名」の形式で表示する。
		ただし、これまでに発生しなかった取引事由についても併せて記載されるようにすること。
	*/
	SELECT  T.[取引番号]
	       ,CAST( J.[取引事由ID] AS VARCHAR) + ':' + J.[取引事由名] AS [取引事由]
		   ,T.[日付],T.[口座番号],T.[入金額],T.[出金額]
	  FROM [dokoQL].[dbo].[取引_t] AS T
	  RIGHT JOIN [dokoQL].[dbo].[取引事由_t] AS J
	    ON T.[取引事由ID] = J.[取引事由ID]


### 完全外部結合：左右の表の全行を必ず出力する
		SELECT 選択列リスト
		FROM   左表テーブルＡのテーブル名
		FULL JOIN   右表テーブルＢのテーブル名
		ON     両テーブルの結合条件

	事例
	/* LEVEL7-70
	取引テーブルと取引事由テーブルから、取引事由の一覧を抽出する。
	一覧には取引事由IDと取引事由名を記載する。
	なお、取引事由テーブルに存在しない事由で取引されている可能性、
	および取引の実績のない事由が存在する可能性を考慮すること。
	*/
	/* 正答 */
	SELECT DISTINCT J.[取引事由ID],J.[取引事由名]
	  FROM [dokoQL].[dbo].[取引_t] AS T
	  FULL JOIN [dokoQL].[dbo].[取引事由_t] AS J
	    ON T.[取引事由ID] = J.[取引事由ID]

	/* 別解 */
	-- FULL JOINが使えない場合、以下で代替
	SELECT DISTINCT T.[取引事由ID], J.[取引事由名]
	  FROM [dokoQL].[dbo].[取引_t] AS T
	  LEFT JOIN [dokoQL].[dbo].[取引事由_t] AS J
	         ON T.[取引事由ID] = J.[取引事由ID]
	UNION
	SELECT DISTINCT J.[取引事由ID], J.[取引事由名]
	  FROM [dokoQL].[dbo].[取引_t] AS T
	 RIGHT JOIN [dokoQL].[dbo].[取引事由_t] AS J
	         ON T.[取引事由ID] = J.[取引事由ID]

### 結合の構文：テーブル名の指定
		SELECT 日付, 家計簿.メモ, 費目.メモ -- 属するテーブル名を明示する
		FROM   家計簿テーブルのテーブル名
		JOIN   費目テーブルのテーブル名
		ON     家計簿テーブル.費目ID = 費目テーブル.ID

### 結合の構文：別名を使ったSQL文
		SELECT 日付, K.メモ, H.メモ
		FROM   家計簿テーブル AS K -- テーブル名に別名を設定する
		JOIN   費目テーブル   AS H
		ON     K.費目ID = H.ID

### 結合の構文：3つのテーブルを結合するSQL文
		SELECT 日付, 費目.名前, 経費区分.名称
		FROM   家計簿テーブル
		JOIN   費目テーブル  -- まず家計簿テーブルに費目を結合する
		ON     家計簿テーブル.費目ID = 費目テーブル.ID
		JOIN   経費区分テーブル  -- その結果にさらに経費区分を結合する
		ON     費目テーブル.経費区分ID = 経費区分テーブル.ID

	事例
	/* LEVEL7-68の考察
		2016年3月1日に取引のあった口座番号の一覧を取得する。
		一覧には、口座テーブルより名義と残高も表示すること。
		解約されたロ座ももれなく一覧に記載する。
	*/

	SELECT  T.[取引番号],T.[取引事由ID],J.[取引事由名]
	       ,T.[日付],T.[口座番号],K.[名義],T.[入金額],T.[出金額]
	  FROM [dokoQL].[dbo].[取引_t] AS T
	  LEFT JOIN [dokoQL].[dbo].[取引事由_t] AS J
	    ON T.[取引事由ID] = J.[取引事由ID]
	  LEFT JOIN [dokoQL].[dbo].[口座_t] AS K
	    ON T.[口座番号] = K.[口座番号]
	 WHERE T.[日付] = '2016-03-01';

### 結合の構文：副問合せの結果と結合するSQL文
		SELECT 日付, 費目.名前, 費目.経費区分ID
		FROM   家計簿テーブル
		JOIN  (SELECT * FROM 費目テーブル
				WHERE 経費区分ID = 1
			  ) AS 費目 -- 副問合せの結果を結合
		ON     家計簿テーブル.費目ID = 費目テーブル.ID

	事例
	/* LEVEL7-74
		取引テーブルから、同一の口座で同じ日に3回以上取引された実績のある口座番号とその回数を抽出する。
		併せて、口座テーブルから名義を表示すること。
	*/
	SELECT K.[口座番号],T.[回数],K.[名義]
	  FROM [dokoQL].[dbo].[口座_t] AS K
	  JOIN (SELECT [口座番号], COUNT(*) AS [回数]
		      FROM [dokoQL].[dbo].[取引_t]
			 GROUP BY [口座番号],[日付]
	        HAVING COUNT(*) >= 3)  AS T
	    ON K.[口座番号] = T.[口座番号];


	事例
	/* LEVEL7-73
		結合相手に副問い合わせを利用するようSQL文を変更する。
	*/

	/* 変更前のSQL */
	SELECT K.[口座番号],K.[名義],K.[残高]
	      ,T.[日付] AS [取引の日付],T.[取引事由ID],T.[入金額],T.[出金額]
	    FROM [dokoQL].[dbo].[口座_t] AS K
		JOIN [dokoQL].[dbo].[取引_t] AS T
		  ON K.[口座番号] = T.[口座番号]
		WHERE K.[残高] >= 5000000
		  AND ( T.[入金額] >= 1000000 OR T.[出金額] >= 1000000 )
	      AND ( T.[日付] >= '2018-01-01');

	/* 副問合せに変更したSQL */
	SELECT K.[口座番号],K.[名義],K.[残高]
	      ,T.[日付] AS [取引の日付],T.[取引事由ID],T.[入金額],T.[出金額]
	    FROM [dokoQL].[dbo].[口座_t] AS K
		JOIN (SELECT T1.[口座番号], T1.[日付], T1.[取引事由ID], T1.[入金額], T1.[出金額]
		        FROM [dokoQL].[dbo].[取引_t] AS T1 -- SELECTの内と外では集合が違うのでわざと内にT1の別名を付けたが、
												   -- 無理に付ける必要はない。
				WHERE ( T1.[入金額] >= 1000000 OR T1.[出金額] >= 1000000 )
				  AND ( T1.[日付] >= '2018-01-01')
			)  AS T -- 変更前はJOINについていた別名
		  ON K.[口座番号] = T.[口座番号]
		WHERE K.[残高] >= 5000000;

### 結合の構文：自分自身と結合するSQL文。
		※自己結合(self join)、再帰結合(recursive join)ともいう
		SELECT A.日付, A.メモ, A.関連日付, B.メモ -- 新しい「関連日付」列を作る
		FROM   家計簿テーブル AS A
		LEFT JOIN 家計簿テーブル AS B -- 同じテーブルに別の名前を付ける
		ON     A.関連日付 = B.日付


	事例
	/* LEVEL7-75
		この銀行では、口座テーブルの名寄せを行うことになった。
		同じ名義で複数の口座番号を持つ顧客について、次の項目を持つ一覧を取得する。
		・名義、口座番号、種別、残高、更新日
		一覧は名義のアイウエオ順、口座番号の小さい順に並べること。
	*/

	/* 回答 集計関数と結合を用いた場合 */
	SELECT  K1.[名義],K1.[口座番号],K1.[種別],K1.[残高],K1.[更新日]
	  FROM [dokoQL].[dbo].[口座_t] AS K1
	  JOIN (SELECT [名義],COUNT([名義]) AS [口座数]
			  FROM [dokoQL].[dbo].[口座_t]
			 GROUP BY [名義]
	         HAVING COUNT([名義]) >= 2
			) AS K2
		 ON K1.[名義] = K2.[名義]
	  ORDER BY K1.[名義] ASC, K1.[口座番号] ASC;

	/* 別解　自己結合を用いた場合 */
	SELECT DISTINCT K1.[名義], K1.[口座番号],K1.[種別], K1.[残高], K1.[更新日]
	  FROM [dokoQL].[dbo].[口座_t] AS K1
	  JOIN [dokoQL].[dbo].[口座_t] AS K2
	    ON K1.[名義] = K2.[名義]
	 WHERE K1.[口座番号] <> K2.[口座番号]
	 ORDER BY K1.[名義], K1.[口座番号];


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
# SQLチートシート (第3部　/　全4部)
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

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
# SQLチートシート (第4部　/　全4部)
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */

## テーブルの設計

### データベース構築のINPUTとOUTPUT
	INPUT:要件の一覧表（お客様からインタビューしたもの）
	OUTPUT:一連のDDL文（データ定義言語。
	　　　　　　　　　　テーブルなどの作成や削除、各種設定などの命令(CREATE,ALTER,DROP,TRUNCATE)）

	概念設計：要件に登場する情報を使って、管理すべき情報を整理する。
	　　　　　情報間の関連や関係も整理する。データベースやシステムは考えない。
	論理設計：概念設計で明らかにした管理すべき情報を使って、RDBを使う前提で構造を整理する。
	　　　　　テーブルと列がわかれば十分。
	物理設計：特定のDBMS製品(例えばSQL Server)を使う前提に立ち、論理設計で明らかになった各テーブルを具体化する。
	　　　　　すべてのテーブルのすべての列について、型、インデックス、制約、デフォルト値など、テーブル作成に必要なすべての要素を確定する。

### 概念設計
	エンティティ(entity)…情報のかたまり。「テーブル」のようなもの。
	　　　　　　　　　　　帳簿のように形のあるものだけでなく、入出金行為のように形のないものもエンティティになる。
	属性(attribute)…テーブルの「列」のようなもの。
	関連(relationship)…テーブル間の関連

	ER図(ERD:entity-relationship diagram)…実体関連図。エンティティ、属性、関連を俯瞰してみることができる図。
	　　　　　　　　　　　　　　　　　　記法としてIE(Information Engineering)がよく使われる。
	　　　　　　　　　　　　　　　　　　[ER図の Crow's Foot 記法 (IE 記法)](https://knooto.info/erd-crows-foot/)
	　　　　　　　　　　　　　　　　　　[ER図の書き方５ステップ](https://it-koala.com/entity-relationship-diagram-1897)

	多重度（カーディナリティ）…エンティティ同士の数量的な関係。1:1、0or1:1、1:多、1:多(0以上)、1:多(1以上)

#### (参考)ER図のツールを探した
	[ER図をいい感じに作れそうなモデリングツールを探してみた](http://bashalog.c-brains.jp/18/02/14-144444.php)
	●A5:SQL Mk-2　SQLを実行したり、テーブルを編集するほかに、SQLの実行計画を取得したり、ER図を作成したりすることが出来ます。
	・MySQL WorkBench 高機能なMySQLのデータモデル開発用アプリケーション。ER図からDDLへのエクスポートが可能、DBとの接続、SQL実行も可能。
	・ERDPlus ER図を作れる、シンプルな機能と操作性のWebアプリ。Beta版ですがExportSQLからDDL生成もできるようです。
	・WWW SQL Designer hサーバーに設置可能なPHP製ツールで、ブログにも多く取り上げられています。 SQL出力可能。
	・LucidChart Webアプリ。DFDなどのモデリングツールです。 ER図からDDLを作成する機能はありません。
	・draw.io。クラウドストレージと密に融合しているWebアプリ。Google DriveやOne Drive上にファイルを保存できます。 Databaseのテンプレート内にER図があります。描画のみ。
	・Edraw MAX。MS Office製品に近いUIを持つアプリ。グラフィカルなプレゼン資料作成向き。エンティティの描画は可能ながら、属性やデータ型の追加機能などはありません。

### 論理設計
	論理設計の目的は、概念上のエンティティをリレーショナルデータベースモデルで扱いやすい形のテーブルに変形することにある。

	論理設計の流れ
	・多対多の関係は、中間テーブルを使って、２つ以上の１対多の関係に変換する。
	・キーを整理する。主キーが備えるべき３つの特性を持っていること。
		非NULL性：必ず何らかの値を持っている
		一意性：他と重複しない
		不変性：一度決定されたら値が変化することがない（主キーは、一貫して同じ１行を指し示す）
	・正規化(normalization)する。矛盾したデータを格納できないように、テーブルを複数に分割していく作業。
		『１つの事実は１箇所に(one-fact in one-place)』…整合性が崩れにくい優れたテーブル設計の原則

### 正規化の手順
	正規形(normalized form)…正規化によってテーブルが手説に分割された状態
	正規化の流れ…手元にあるエンティティ構造（テーブル構）を、非正規形から第３正規形まで順次変形していく。

#### 第一正規形への変形…繰り返し列の排除
	テーブルのすべての行のすべての列に１つずつ値が入っているべきである。
	よって、繰り返しの列やセル結合（同じ値が列や行に現れる）があってはならない。
	非正規形を第一正規形に変形するステップ
	①繰り返しの列の部分を別の表に切り出す（費目ID列と費目名列など）
	②切り出したテーブルの仮の主キーを決める
	③主キー列をコピーして複合主キーを構成する

#### 関数従属性
	関数従属性とは「ある列Ａの値が決まれば、おのずと列Ｂの値も決まる」という関係。
	このとき「列Ｂは列Ａに関数従属している」という。　イメージ的には　Fx(A)=B
	例えば、入出金行為ＩＤがわかればその日付が確定できるとすると、日付は入出金行為ＩＤに関数従属しているといえる。

	すべての非キー列は、主キーにきれいに関数従属しているべきである。

#### 第二正規形への変形…部分関数従属の排除
	主キーに対する「きたない関数従属（部分関数従属）」の排除が目的。
	複合主キーを持つテーブルの場合、非キー列は、複合主キーの全体に関数従属するべきである。
	よって、「複合主キーの一部に対してのみ関数従属する列」が含まれてはならない
	①複合主キーの一部に関数従属する列を切り出す。
	②部分関数従属先だった列をコピーする

#### 第三正規形への変形…間接的な関数従属（推移関数従属）の排除
	テーブルの非キー列は、主キーに直接、関数従属すべきである。
	よって、「主キーに関数従属する列に、さらに関数従属する列」は存在してはならない。
	①間接的に主キーに関数従属する列を切り出す
	②直接関数従属先だった列をコピーする



### 物理設計
	①最終的なテーブル名、列名（物理名）を決定する。
		物理名(physical name)…データベース内にテーブルが作られる際のテーブル名や列名
		論理名(logical name)…論理設計までの段階で利用してきた名前
		※SPROでの命名規約
		　テーブル名：英字名称の接尾辞に"_t"を付与する。
		　View：接尾辞に"_v"を付与する。
		　マスタ："m_○○○_t"とする。
		　SQLコマンド：sql_[英字]
	②列の型を決定する。
	③制約、デフォルト値を決定する。
	④インデックスを決定する。
	⑤その他（ビュー作成、性能のための巨大テーブル分割や非正規化）
		※正規化すると多数のテーブルをSQL上で結合することになり、処理性能が向上することもあるが、
		　データの整合性が崩れやすくなる。
		　
### 正規化されたデータの利用
	管理に適した形と、利用に適した形はことなる
		管理するときは…データは複数のテーブルに分割してあるほうがよい。
		利用するときは…データは１つのテーブルに結合してあるほうがよい。


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## TIPS
### テーブルの作成、バックアップ用にテーブルコピー、リストア

	データ削除や更新をする前の状態にロールバックできるようにすることを含めたテーブル作成方法と
	それを活用したバックアップテーブル作成、およびデータリストア方法

	１．テーブル作成
		EXCELに定義書とデータを作成する
		SSMS＞データベースを右クリック>タスク>データのインポート
		　参照　第一部 ＞ データインポート方法 ＞ Excel から SQL Server または Azure SQL Database にデータをインポートする
		マッピングでテーブル名と型の定義を修正し、インポートする

	２．テーブル生成スクリプトを作成する
		SSMSのテーブル＞デザインで、主キーとCHECK制約を入力
		タスク＞スクリプト生成
				・スクリプトファイル保存する
				・UNICODEを選択
		ファイル名 SQLquery[識別名]-script.sqlで保存する

	３．バックアップ用テーブルの作成
		SSMSで新規SQLを作成し、２で保存した SQLquery[識別名]-script.sql のSQL文をコピペする。
		ファイル名をバックアップ用に書き換える
			事例　ファイルが [テーブル名]_t (※ _t はサフィックス）の場合は
			　　　サフックスをキーワードにして [テーブル名]BKP_tに置換する。
		SQLを実行する。
		SQLはファイル名 SQLquery[識別名]BKP-script.sqlで保存する

	４．データのリストア
		SSMS＞データベースを右クリック>タスク>データのインポート
		　参照　第一部 ＞ データインポート方法 ＞ Excel から SQL Server または Azure SQL Database にデータをインポートする
		リストア先のテーブル名を修正し、インポートする
		※前の３で型や主キーや制約ができているので、前の１のような型の修正は不要

### テスト用環境で使用するためデータベースを他の場所へコピーする

	(移出元)(ZK0AAA85:SQL Server 2019)
	SSMSのdokoQLを右クリック＞タスク＞バックアップ
	コピーのみのバックアップにチェックを入れる
		標準のバックアップ先はココ↓
			C:\Program Files\Microsoft SQL Server\MSSQL15.SQLEXPRESS\MSSQL\Backup
			dokoQL.bak

  (移出先)SSMSが同じバージョンのとき
  dokoQL.bakをココ↓に入れる
    C:\Program Files\Microsoft SQL Server\MSSQL12.SQLEXPRESS\MSSQL\Backup
	SSMSのdokoQLを右クリック＞タスク＞復元＞データベース
	デバイスのラジオボタンを選択
  バー右の･･･をクリックして、「バックアップデバイスの選択」ウィンドゥを開く
  「追加」ボタンを押して、dokoQL.bakを選択する

	 Microsoft SQL Transact-SQL リファレンス
		(Docs ＞ SQL ＞ ビジネス継続性 ＞ バックアップと復元 ＞ 概念 ＞ バックアップ ＞ コピーのみのバックアップ）
		https://docs.microsoft.com/ja-jp/sql/relational-databases/backup-restore/copy-only-backups-sql-server?view=sql-server-ver15


  (移出先)SSMSが異なるバージョンのとき(ZK0AAA85:SQL Server 2014))
  直接復元ができないので、テーブルごとに作成スクリプトを保存し、それをリストア先で実行する。
  ①移出先のSQL Server Management Studioで、
  　各テーブルごとに右クリックからテーブルをスクリプト化 → 新規作成 → ファイル を選択する。
  　その後、適当なファイル名を付けて保存する。
  ②移出先で、保存したファイルを開き、Product_Mngのデータベースを選択して実行する。
  　テーブル作成に成功すれば「コマンドは正常に完了しました。」と表示される。
  作成できたテーブルを確認すると、正しく構造が正しく移し込まれている。

	 Qiita
		(SQL Serverのバージョンが異なるため復元に失敗する）
		https://qiita.com/RollSystems/items/b2c79a62c7e5604f216d




### 不等号の読み方
	「<」は左辺が右辺より小さいことを示す。
	「>」は左辺が右辺より大きいことを示す。
	使用例
	3 > 2 （3は2より大きい／3大なり2） -- ★「より小さい」「より大きい」はその数を含めない表現
	2 < 3 （2は3より小さい／2小なり3） -- ★必ず左辺を主語としていることに注意
	プログラミングでは「LT (less than)」「GT (greater than)」と呼ぶこともあるが、ほとんどの言語で、不等号は < と > で表される。

	「≦」「≤」「⩽」（いずれでも意味は同じ）は左辺が右辺より小さいか等しい（a < b または a = b）ことを示す。
	「≧」「≥」「⩾」は左辺が右辺より大きいか等しいことを示す。
	A ≦ B (AはB以下／A小なりイコールB) -- ★必ず左辺を主語としていることに注意
	A ≧ B (AはB以上／A大なりイコールB)
	プログラミングでは「LE (less than or equal to)」「GE (greater than or equal to)」と呼ぶこともあるが、
	ほとんどの言語で、等号付き不等号は <= と >= で表される。

	[参考:Wikipedia](https://ja.wikipedia.org/wiki/%E4%B8%8D%E7%AD%89%E5%8F%B7)
