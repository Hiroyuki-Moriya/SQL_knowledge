# SQL�`�[�g�V�[�g (��2���@/�@�S4��)

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �Q�l����

 Microsoft SQL �f�[�^�x�[�X�֐�
  �iDocs��SQL�����t�@�����X��Transact-SQL (T-SQL) ���t�@�����X���֐��j
  https://docs.microsoft.com/ja-jp/sql/t-sql/functions/functions?view=sql-server-ver15

 �t����SQL�\���W
  http://www.sql-reference.com/index.html#string


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## ���Ɗ֐�

### �v�Z���Ƃ�
	�������͌��ʂ��^���U�ɂȂ����
	���ʂ��l�ɂȂ���́B�������̈ꕔ�Ƃ��ėp�����邱�Ƃ�����B

	�I��񃊃X�g�Ŏg���P�[�X
	��@SELECT [�o���z],[�o���z]+100 AS '+100','SQL' AS '�r�p�k'
		  FROM [dokoQL].[dbo].[�ƌv��_tbl]
		�o���z	'+100'	'�r�p�k'
		308		408		SQL

### ���Z�q

	+	���l+���l
		���t+���l  <= SQLserver�ł́ADATEADD (DAY , ���l, ���t )

	-	���l-���l
		���t-���l
		���t-���t  <= SQLserver�ł́ADATEDIFF ( datepart , startdate , enddate )  

	*	���l*���l
	
	/	���l/���l

	||	������||������  <= SQLserver�ł́A�A�����Z�q�̓v���X�L�� (+) 

	
### CASE���Z�q

#### CASE���Z�q�̗��p�\��(1)
		CASE �]�������⎮
		   WHEN �l1 THEN �l1�̎��ɕԂ��l
		  (WHEN �l2 THEN �l2�̎��ɕԂ��l) �c
		  (ELSE �f�t�H���g�l)
		END

	����
	/*
	  ��ڗ�Əo���z��ɐV����'�o��̕���'���������B
	  �o��̕��ނɃZ�b�g����l�͔�ڂ̒l�ɏ]���ă��e�������Z�b�g����B
	  �����ΏۂƂ���̂͏o���z��0�𒴂��郌�R�[�h�Ƃ���B
	*/

	SELECT [���],[�o���z],    -- <== �Ō�̃J���}�͐V������(NULL)��AS�ŗ񖼂�t���邽�ߕK�v
			CASE [���]
			   WHEN '���Z��' THEN '�Œ��'
			   WHEN '�������M��' THEN '�Œ��'
			   ELSE '�ϓ���'
			END
		AS [�o��̕���]
		FROM [dokoQL].[dbo].[�ƌv��_tbl]
			WHERE [�o���z] > 0


#### CASE���Z�q�̗��p�\��(2)
		CASE 
		   WHEN ����1 THEN ����1�̎��ɕԂ��l
		  (WHEN ����2 THEN ����2�̎��ɕԂ��l) �c
		  (ELSE �f�t�H���g�l)
		END

	����
	/*
	  ��ڗ�ƁA�����z��ƁA�V����'�����̕���'������ĕ\������B
	  �����̕��ނɃZ�b�g����l�͓����z�̒l�ɏ]���ă��e�������Z�b�g����B
	  �����ΏۂƂ���͓̂����z��0�𒴂��郌�R�[�h�Ƃ���B
	*/

	SELECT [���],[�����z],    -- <== �Ō�̃J���}�͐V������(NULL)��AS�ŗ񖼂�t���邽�ߕK�v
			CASE
			   WHEN [�����z] < 5000 THEN '��������'
			   WHEN [�����z] < 100000 THEN '�ꎞ����'
			   WHEN [�����z] < 300000 THEN '�����o���[�I'
			   ELSE '�z��O�̎����ł��I'
			END
		AS [�����̕���]
		FROM [dokoQL].[dbo].[�ƌv��_tbl]
			WHERE [�����z] > 0

	����
	/* 
	�@�@�N���10�Ζ��̔N��Ɋ���A
	�@�@���ʋL����|�󂵂āA
	�@�@�N��Ɛ��ʖ|����R�����łȂ���
	�@�u�����v�Ƃ��ĕ\������
	*/
	SELECT RTRIM([���[���A�h���X]) AS [���[���A�h���X]
		, 
		 CASE 
			WHEN [�N��] BETWEEN 20 AND 29 THEN '20��'
			WHEN [�N��] BETWEEN 30 AND 39 THEN '30��'
			WHEN [�N��] BETWEEN 40 AND 49 THEN '40��'
			WHEN [�N��] BETWEEN 50 AND 59 THEN '50��'
		 END
		+ ':' + 
		 CASE
			WHEN [����] = 'M' THEN '�j��'
			WHEN [����] = 'F' THEN '����'
		 END
		AS [����]
	  FROM [dokoQL].[dbo].[�񓚎�_tbl];

	����
	/* 
	  ���[���A�h���X�̉��Q���ō������ʂ��A
	  �����ɃZ�b�g����
	*/
	UPDATE [dokoQL].[dbo].[�񓚎�_tbl]
		SET [����] = CASE SUBSTRING ( RTRIM([���[���A�h���X])
									, LEN(
										RTRIM([���[���A�h���X]))-1
									, 2)
					   WHEN 'jp' THEN	'���{'
			   		   WHEN 'uk' THEN	'�C�M���X'
			   		   WHEN 'cn' THEN	'����'
			   		   WHEN 'fr' THEN	'�t�����X'
			   		   WHEN 'vn' THEN	'�x�g�i��'
					END


#### ���[�U��`�֐�
		���炩���ߗp�ӂ��ꂽ�֐��ł͂Ȃ��A
		�K�v�Ƃ��鏈���������ŋL�q���쐬�����֐�

#### �X�g�A�h�v���V�[�W��
		���s���镡����SQL�����܂Ƃ߃v���O�����̂悤�Ȃ��̂Ƃ���DBMS���ɕۑ����A
		�f�[�^�x�[�X�̊O������Ăяo������

#### Transact-SQL
		���[�U��`�֐���X�g�A�h�v���V�[�W�����L�q����Ƃ��̐�p����B
		SQL Server��PL/SQL�ŗp������B

#### ������֐�
	LENGTH (�������\����)	 �� ������̒�����\�����l
	LEN (�������\����)	 �� ������̒�����\�����l�BSQLServer

	����
	  10����(10�o�C�g)�ȉ��̃����������擾����
		SELECT [����], LEN([����]) AS [�����̒���]
			FROM [dokoQL].[dbo].[�ƌv��_tbl]
				WHERE LEN([����]) <= 10;


	TRIM (�������\����)	 �� ���E����󔒂���������������B��SQLServer�̓T�|�[�g���ĂȂ�
	LTRIM (�������\����)	 �� �����̋󔒂���������������BSQLServer
	RTRIM (�������\����)	 �� �E���̋󔒂���������������BSQLServer


	REPLACE (�u���Ώۂ̗�, �u���O�̕���������, �u����̕��������� )	 �� �u���������ꂽ��̕���������BSQLServer

	����
	  �������'�w��'��'������'�ɒu��������
		UPDATE  [dokoQL].[dbo].[�ƌv��_tbl]
				SET [����] = REPLACE([����],'�w��','������');

	����
	/*�@[����]�̂����󔒂��܂܂Ȃ���������[�u���㕶����]�ɕ\������
	    [����]�̂����󔒂��܂܂Ȃ���������[�u���㕶����]�ɕ\������
	*/
		SELECT [����]
			, REPLACE([����], ' ', '') AS [�u���㕶����]  -- ���u���O�̕���������ɔ��p�󔒂��Z�b�g����ΑS�p���E���Ă����
		    , LEN ( REPLACE([����], ' ', '')) AS [�u���㕶����]
		  FROM [dokoQL].[dbo].[��_tbl];


	SUBSTRING (�������\����, ���o���J�n����ʒu, ���o���镶���̐� )	 �� ���o���ꂽ����������BSQLServer
	SUBSTR (�������\����, ���o���J�n����ʒu, ���o���镶���̐� )	 �� ���o���ꂽ����������
	
	����
	  ��ڗ�̂P�`�R�����ڂ�'��'�̕�����������̂����𒊏o����
		SELECT * FROM [dokoQL].[dbo].[�ƌv��_tbl]
				WHERE SUBSTRING([���],1,3) LIKE '%��%';
		/* ��LIKE���Z�q�̕��@�́@�� LIKE �p�^�[�������� �B
		     �֐��Ő؂�o�����������LIKE���Z�q�̃p�^�[��������ƈ�v���Ă��邩�ƍ����Ă��� */
	
	
	CONCAT(������, ������ [, ������c] )	 �� �A����̕�����
	������1 + ������2	 �� �A����̕�����BSQLServer
	
	
#### ���l�̊֐�
	ROUND ( ���l��\����, �L���Ƃ��錅�� )	�� ���l���ۂ߂��l�B�������ۂ߂ł���A�l�̌ܓ��ł͂Ȃ��B
	/*��	�u�����ۂ߁v�͎l�̌ܓ��Ƃقړ���ł����A���̂P�_���Ⴂ�܂��B
			�u�ۂߒP�ʂ̒��x�܂�Ȃ��ŁA�ǂ��������̏ꍇ�́A���������̗p����v
			���������āA1.25 �� 0.01 �̈ʂŊۂ߂�� 1.2 �ɂȂ�A1.35 �� 1.4 �ɂȂ�܂��B
	*/
	
	����
	  �o���z��S�~�P�ʂɊۂ߂�i10�~�̌��Ŋۂ߂�100�~�̌��ɐ؂�グ�邩�A�؂�̂Ă�j
		SELECT [�o���z], ROUND([�o���z],-2) AS [�S�~�P�ʂ̏o���z]
			FROM [dokoQL].[dbo].[�ƌv��_tbl]
	
	
	TRUNC ( ���l��\����, �L���Ƃ��錅�� )	�� �؂�̂Ă��l�B��SQLServer�̓T�|�[�g���ĂȂ�
	FLOOR ( ���l��\���� )	�� �����_��؂�̂Ă��l�BSQLServer

	����
	  �o���z��~�P�ʁi�����_�؂�̂āj�ŕ\������
		SELECT [�o���z], FLOOR([�o���z]/1000) AS [��~�P�ʂ̏o���z]
			FROM [dokoQL].[dbo].[�ƌv��_tbl]


	POWER ( ���l��\����, ���悷�邩���w�肷�鐔�l )	�� ���l���w�肵���񐔂����悶������

	����
	  �o���z�̂R���\������
		SELECT [�o���z], POWER([�o���z], 3) AS [�o���z��3��]
			FROM [dokoQL].[dbo].[�ƌv��_tbl]
			WHERE[�o���z]<1000

#### ���t�̊֐�
	CURRENT_DATE	�� ���݂̓��t��SQLServer�̓T�|�[�g���ĂȂ�
	CURRENT_TIME	�� ���݂̎�����SQLServer�̓T�|�[�g���ĂȂ�
	GETDATE()	�� SQL Server �̃C���X�^���X�����s����Ă���R���s���[�^�[�̓��t�Ǝ���
	CURRENT_TIMESTAMP	�� �����s�v
	SYSDATETIME ( )	��1�b�����̗L�������Ŕ�r�����GETDATE��萸�x�������Bdate�^��time�^�̕ϐ��Ɋ��蓖�Ă���B

	����
	  ���t�������擾���āA���t�����̂ݕ\������
		SELECT GETDATE() AS [GETDATE],
				DATENAME(day,GETDATE()) AS [DATEPART]

	  ���t�������擾���āA�o�^����
		INSERT INTO [dokoQL].[dbo].[�ƌv��_tbl]
			VALUES (GETDATE(), '�H��','�h�[�i�c�𔃂���',0,260);
		/* GETDATE()�̖߂�l��\������Ɠ��t�^��YYYY-MM-DD HH:MM:SS�ŕ\������Ă���B
		   �ƌv��_TBL�̂P��ڂ�[���t]�œ��t�^�Ȃ̂ŁAGETDATE()�̒l�����̂܂ܓ������B
		*/

	���� Microsoft SQL Server���
	A. ���݂̃V�X�e�����t�Ǝ������擾����
		SELECT SYSDATETIME()  -- 2007-04-30 13:10:02.0474381  
		    ,SYSDATETIMEOFFSET()  -- 2007-04-30 13:10:02.0474381 -07:00
		    ,SYSUTCDATETIME()     -- 2007-04-30 20:10:02.0474381  
		    ,CURRENT_TIMESTAMP    -- 2007-04-30 13:10:02.047  
		    ,GETDATE()            -- 2007-04-30 13:10:02.047  
		    ,GETUTCDATE();        -- 2007-04-30 20:10:02.047  

	B. ���݂̃V�X�e�����t���擾����
		SELECT CONVERT (DATE, SYSDATETIME())      -- 2007-05-03
		    ,CONVERT (DATE, SYSDATETIMEOFFSET())  -- 2007-05-03
		    ,CONVERT (DATE, SYSUTCDATETIME())     -- 2007-05-04
		    ,CONVERT (DATE, CURRENT_TIMESTAMP)    -- 2007-05-03
		    ,CONVERT (DATE, GETDATE())            -- 2007-05-03
		    ,CONVERT (DATE, GETUTCDATE());        -- 2007-05-04
  
	C. ���݂̃V�X�e���������擾����
		SELECT CONVERT (TIME, SYSDATETIME())     -- 13:18:45.3490361
		    ,CONVERT (TIME, SYSDATETIMEOFFSET()) -- 13:18:45.3490361 
		    ,CONVERT (TIME, SYSUTCDATETIME())    -- 20:18:45.3490361
		    ,CONVERT (TIME, CURRENT_TIMESTAMP)   -- 13:18:45.3470000
		    ,CONVERT (TIME, GETDATE())           -- 13:18:45.3470000
		    ,CONVERT (TIME, GETUTCDATE());       -- 20:18:45.3470000
  


#### �ϊ��̊֐�
	CAST (�ϊ�����l AS �ϊ�����^ )	�� �ϊ���̒l

	����
	  �o���z�ɒP�ʂ�"�~"��A�����ĕ\������
		SELECT CAST ( [�o���z] AS VARCHAR(20) ) + '�~'
			FROM [dokoQL].[dbo].[�ƌv��_tbl]
		/* ���o���z��INT�^�Ȃ̂ŁA���l�ƕ����Ƃ����^�̈قȂ�l��'+'���Z�q�ŘA�����邱�Ƃ͂ł��Ȃ��B
		  �����ŁACAST()�Ō^�𐔎����當���֕ϊ����ĘA������
		*/

	  ���t�������擾���āA�����ɍ��킹�ĕ\������
		SELECT
			CURRENT_TIMESTAMP                      AS [CURRENT_TIMESTAMP], -- 2020-08-18 11:03:20.487
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,111) AS [yyyy/mm/dd],        -- 2020/08/18
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,112) AS [yyyymmdd],          -- 20200818
			CONVERT(VARCHAR,CURRENT_TIMESTAMP,120) AS [yyyy-mm-dd hh:mm:ss]   -- 2020-08-18 11:03:20
		/* ��CONVERT�́A���t�𕶎���ɕϊ����邽�߂Ɏg�p���Ă���B
		     �䂦�ɁA�o�͒l�̌^�͕�����ŁA���t�ł͂Ȃ��B
		*/


	COALESCE ( ��⎮1, ��⎮2, ��⎮3, �c)	�� �����̂����A�ŏ��Ɍ��ꂽNULL�łȂ������B�R�A���X�֐�

	����
		SELECT COALESCE ( 'A'  , 'B'  , 'C'  );	-- ���ʂ�A
		SELECT COALESCE ( NULL , 'B'  , 'C'  );	-- ���ʂ�B
		SELECT COALESCE ( NULL , 'B'  , NULL );	-- ���ʂ�B
		SELECT COALESCE ( NULL , NULL , 'C'  );	-- ���ʂ�C

	����
	  ������̒l��NULL�Ȃ�΁A���ӊ��N���b�Z�[�W��\������
	  	INSERT INTO [dokoQL].[dbo].[�ƌv��_tbl]
			VALUES ('2018-02-11', '���{��y��',NULL,0,2800);
	  	INSERT INTO [dokoQL].[dbo].[�ƌv��_tbl]
			VALUES ('2018-02-14', '���۔�',NULL,0,5000);
		SELECT [���t], [���]
			  ,COALESCE ([����], '(��������NULL�ł�)') AS [����]
			  ,[�����z], [�o���z]
			FROM [dokoQL].[dbo].[�ƌv��_tbl]

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �W�v�ƃO���[�v��

### �W�v�֐��̓���
	�W�v�֐��́A�����Ώۂ̑S�s���ЂƂ܂Ƃ߂Ɉ����A�P�񂾂��W�v�����������Ȃ��B
	�W�v�֐��̌��ʂ́A�K���P�s�ɂȂ�
	�W�v�֐��́ASELECT���̑I��񃊃X�g���ASELECT����ORDER BY�傩�ASELECT����HAVING��̒��Ŏg�p����B
	�W�v�֐��́ASELECT����WHERE��̒��ł͎g�p�ł��Ȃ��B
	

	�o���z���W�v����P�[�X
	��@SELECT SUM( [�o���z] ) AS [�o���z�̍��v]
		  FROM [dokoQL].[dbo].[�ƌv��_tbl]

### ��\�I�ȏW�v�֐�
	����	�֐���	����
	�W�v	SUM()	�e�s�̒l�̍��v�����߂�BDISTINCT���w�肷��Əd���s�������ďW�v�ł���
	�@�V	MAX()	�e�s�̒l�̍ő�l�����߂�
	�@�V	MIN()	�e�s�̒l�̍ŏ��l�����߂�
	�@�V	AVG()	�e�s�̒l�̕��ϒl�����߂�BDISTINCT���w�肷��Əd���s�������ĕ��ςł���
	�v��	COUNT()	�s�����J�E���g����BDISTINCT���w�肷��Əd���s�������ăJ�E���g�ł���
	/* ��COUNT(*)�́A �P����NULL�̍s���܂߂čs�����J�E���g����
	     COUNT(��)�́A�w���̒l��NULL�̍s���܂߂Ȃ��ōs�����J�E���g����
	*/	

	����
	/* ���X�g6-2 ���܂��܂ȏW�v */
	SELECT
	 SUM( [�o���z] ) AS [���v�o���z]
	,AVG( [�o���z] ) AS [���Ϗo���z]
	,MAX( [�o���z] ) AS [�ł��傫�ȏo���z]
	,MIN( [�o���z] ) AS [�ł����z�̏o���z]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]

	
	����
	/* ���X�g6-3 �H��̍s���𐔂��� */
	SELECT
	 COUNT( * ) AS [�H��̍s��]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]
	WHERE [���] = '�H��'

	����
	/* ���X�g6-3plus �d�������l���������W�v */
	SELECT
	 COUNT( DISTINCT [���] ) AS [��ڂ̎�ސ�]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]

	����
	/* ���X�g6-5 NULL���[���Ƃ��ĕ��ς����߂� */
	SELECT
	 AVG( COALESCE( [�o���z],0) ) AS [�o���z�̕���]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]


### �O���[�v�����ďW�v�����{�\��
	SELECT �O���[�v���̊�񖼁c,�W�v�֐�
	FROM �e�[�u����
	(WHERE �i�荞�ݏ���)
	GROUP BY �O���[�v���̊�񖼁c

	����
/* ���X�g6-7 ��ڂŃO���[�v�����Ă��ꂼ��̍��v�����߂� */
	SELECT
		[���]
		,SUM( [�o���z] ) AS [��ڕʂ̏o���z���v]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]
	GROUP BY [���]  -- ����ڗ�ŃO���[�v������


### �O���[�v�����Ă���i�荞�ފ�{�\��
	SELECT �O���[�v���̊�񖼁c,�W�v�֐�
	FROM �e�[�u����
	(WHERE �w���̕\�ɑ΂���x�i�荞�ݏ���)
	GROUP BY �O���[�v���̊�񖼁c
	HAVING �W�v���ʂɑ΂���i�荞�ݏ���
	/* ��WHERE��́A ���̃e�[�u������������Ƃ��ɍi�荞�ށB�ΏۊO�����ʕ\�ɏo���Ȃ�
	     HAVING��́A�W�v��ɏW�v���ʂ��i�荞��
	*/	

	����
	/* ���X�g6-9 �W�v���ʂōi�荞�� */
	SELECT
		[���]
		,SUM( [�o���z] ) AS [��ڕʂ̏o���z���v]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]
	GROUP BY [���]
	HAVING SUM( [�o���z] ) > 0

	����
	/* �����̏o��̂����A���ς�5,000�~�ȏ�̔�ڂƁA���̍ő�l��m�肽�� */
	SELECT
		[���]
		,MAX ( [�o���z] ) AS [�ő�o���z]
	FROM [dokoQL].[dbo].[�ƌv��_tbl]
	GROUP BY [���]
	HAVING AVG ( [�o���z] ) >= 5000;


### SELECT���̑S�e
	SELECT �I��񃊃X�g
	FROM �e�[�u����
	(WHERE ������)
	(GROUP BY �O���[�v����)
	(HAVING �W�v���ʂɑ΂��������)
	(ORDER BY ���בւ���)
	
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## ���⍇��




/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �����e�[�u���̌���




