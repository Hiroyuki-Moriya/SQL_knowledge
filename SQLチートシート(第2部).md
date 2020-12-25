# SQLチートシート (第2部　/　全4部)

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
	
### 演算子

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
	  
	-	数値-数値
		日付-数値
		日付-日付  <= SQLserverでは、DATEDIFF ( datepart , startdate , enddate )  

	*	数値*数値

	/	数値/数値

	||	文字列||文字列  <= SQLserverでは、連結演算子はプラス記号 (+)

		事例　文字列連結する
			/* LEVEL4-38
			口座テーブルから、種別が「別段」のデータについて、ロ座番号と名義を抽出する。
			ただし、名義の前に「カ）」を付記すること。
			*/
			SELECT [口座番号]
			--     ,'カ）' || [名義] AS [名義]     -- 正答はこれだがSQL Serverではエラーになる。
				   ,'カ）' + [名義]  AS [名義] -- SQL Serverでは加算を使う
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
