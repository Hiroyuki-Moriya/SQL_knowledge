# SQLチートシート (第1部　/　全4部)

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
	 ・文字列はダブルコーテーションでなはなく、シングルコーテーション
	 ・1行追加の時はinsertとvalueが1文づつ必要。i行のinsert文にvaluesは2行書けない
	 ・値が決まっていない(=null)の列については、valuesに空文''やNullを書くのではなく、insert文の列名を書かない。
	 */

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
	  ORDER BY 列名1 並び順,  列名2 並び順
	  OFFSET 先頭から除外する行数 ROWS -- ★除外せずに先頭から取得するときは 0 ROWS を指定する
	  (FETCH NEXT 取得行数 ROWS ONLY)

	-- 事例 家計簿テーブルの明細を出金額の多い順に並べ、その中から11～15行目を取得する
	--      SELECT * FROM 家計簿_TBL
		    ORDER BY 出金額 DESC
		    OFFSET 10 ROWS         -- 先頭10行を除外
		    FETCH NEXT 5 ROWS ONLY -- そのあとに続く5行を取得する
	/* ★ OFFSET - FETCHは、DBMSごとに異なるので、注意が必要 */

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
