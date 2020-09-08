# SQLチートシート (第2部　/　全4部)

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 参考文献

 Microsoft SQL データベース関数
  （Docs＞SQL＞リファレンス＞Transact-SQL (T-SQL) リファレンス＞関数）
  https://docs.microsoft.com/ja-jp/sql/t-sql/functions/functions?view=sql-server-ver15

 逆引きSQL構文集
  http://www.sql-reference.com/index.html#string


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

### 演算子

	+	数値+数値
		日付+数値  <= SQLserverでは、DATEADD (DAY , 数値, 日付 )

	-	数値-数値
		日付-数値
		日付-日付  <= SQLserverでは、DATEDIFF ( datepart , startdate , enddate )  

	*	数値*数値
	
	/	数値/数値

	||	文字列||文字列  <= SQLserverでは、連結演算子はプラス記号 (+) 

	
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
			, REPLACE([文字], ' ', '') AS [置換後文字列]  -- ★置換前の部分文字列に半角空白をセットすれば全角も拾ってくれる
		    , LEN ( REPLACE([文字], ' ', '')) AS [置換後文字数]
		  FROM [dokoQL].[dbo].[受注_tbl];


	SUBSTRING (文字列を表す列, 抽出を開始する位置, 抽出する文字の数 )	 → 抽出された部分文字列。SQLServer
	SUBSTR (文字列を表す列, 抽出を開始する位置, 抽出する文字の数 )	 → 抽出された部分文字列
	
	事例
	  費目列の１～３文字目に'費'の文字があるものだけを抽出する
		SELECT * FROM [dokoQL].[dbo].[家計簿_tbl]
				WHERE SUBSTRING([費目],1,3) LIKE '%費%';
		/* ★LIKE演算子の文法は　式 LIKE パターン文字列 。
		     関数で切り出した文字列をLIKE演算子のパターン文字列と一致しているか照合している */
	
	
	CONCAT(文字列, 文字列 [, 文字列…] )	 → 連結後の文字列
	文字列1 + 文字列2	 → 連結後の文字列。SQLServer
	
	
#### 数値の関数
	ROUND ( 数値を表す列, 有効とする桁数 )	→ 数値を丸めた値。★偶数丸めであり、四捨五入ではない。
	/*★	「偶数丸め」は四捨五入とほぼ同一ですが、次の１点が違います。
			「丸め単位の丁度まんなかで、どっちつかずの場合は、偶数側を採用する」
			したがって、1.25 を 0.01 の位で丸めると 1.2 になり、1.35 は 1.4 になります。
	*/
	
	事例
	  出金額を百円単位に丸める（10円の桁で丸めて100円の桁に切り上げるか、切り捨てる）
		SELECT [出金額], ROUND([出金額],-2) AS [百円単位の出金額]
			FROM [dokoQL].[dbo].[家計簿_tbl]
	
	
	TRUNC ( 数値を表す列, 有効とする桁数 )	→ 切り捨てた値。★SQLServerはサポートしてない
	FLOOR ( 数値を表す列 )	→ 小数点を切り捨てた値。SQLServer

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
	     COUNT(列)は、指定列の値がNULLの行を含めないで行数をカウントする
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
	SELECT 選択列リスト
	FROM テーブル名
	(WHERE 条件式)
	(GROUP BY グループ化列名)
	(HAVING 集計結果に対する条件式)
	(ORDER BY 並べ替え列名)
	
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 副問合せ

### 副問い合わせとは
	ほかのSQL文の一部分として登場するSELECT文。
	丸かっこでくくって記述する。
	SQL文がネスト構造になっている。
	副照会、サブクエリとも呼ぶ。
	

	事例
	/* 最も大きな出費の費目と金額を求める。
	   その１．副問い合わせを使わない方法。メモとSQLの修正が必要。
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
	３．SQLが、表の値の代わりとして、副問い合わせの検索結果を用いるパターン
		※単一の値：スカラー。値が１個だけ返ってくる。
		※複数の値：ベクター。値が１次元配列。
		※表の値：マトリックス。値が２次元配列。


### 単一行副問合せ（単一の値の代わりに副問い合わせの検索結果を用いる）

#### 単一行副問合せとは
		検索結果が1行1列の1つの値となる副問合せ
		SELECT文の選択列リストやFROM句、UPDATE文のSET句、
		また1つの値との判定を行うWHERE句の条件式などに記述することができる

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


### 複数行副問合せ（複数の値の代わりに副問い合わせの検索結果を用いる）

#### 複数行副問合せとは
		検索結果がn行1列の複数の値となる副問合せ（nは1以上）
		複数の値との判定を行うWHERE句の条件式や、SELECT文のFROM句に記述することができる

	事例
	/* INで副問合せを利用する
	*/
	SELECT *
		FROM [dokoQL].[dbo].[家計簿集計_tbl]
		WHERE [費目] IN ( SELECT DISTINCT [費目] FROM [dokoQL].[dbo].[家計簿_tbl] ); --家計簿_TBLに載っている費目で取得する
	   /* ★費目を指定しなくても取得できる。記述ミスや費目の追加変更に対応できる */

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
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## 複数テーブルの結合




