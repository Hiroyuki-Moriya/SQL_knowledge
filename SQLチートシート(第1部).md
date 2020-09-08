# SQL�`�[�g�V�[�g (��1���@/�@�S4��)

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## ��{���@
	/* �R�����g */
	-- �R�����g

	SELECT �� -- ��SELECT���ɂ͑��̋�����̂ŁA�S�e�͑�2�����Q��
	  FROM �e�[�u����
	 WHERE (�C��)(���̑��C��);
	 

	UPDATE �e�[�u���� -- ��WHERE�̂Ȃ�UPDATE���߂͑S���X�V�I
	   SET ��1=���l,��2='������'
	 WHERE (�C��);

	DELETE FROM �e�[�u���� -- ��WHERE�̂Ȃ�DELETE���߂͑S���폜�I
	 WHERE (�C��); 

	INSERT INTO �e�[�u����(��1,��2)
	 VALUES(�l1,�l2) ;
	 /*
	 �E������̓_�u���R�[�e�[�V�����łȂ͂Ȃ��A�V���O���R�[�e�[�V����
	 �E1�s�ǉ��̎���insert��value��1���ÂK�v�Bi�s��insert����values��2�s�����Ȃ�
	 �E�l�����܂��Ă��Ȃ�(=null)�̗�ɂ��ẮAvalues�ɋ�''��Null�������̂ł͂Ȃ��Ainsert���̗񖼂������Ȃ��B
	 */

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �ʖ���`
	 �� AS �ʗ�
	 �e�[�u���� AS �ʃe�[�u����

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## ������

### ��r���Z�q
	  = 
	  <
	  >
	  <=
	  >=
	  <>
	 IS NULL   -- ��NULL��=���Z�q�ł͔���ł��Ȃ�
	 IS NOT NULL
	 
### LIKE
	WHERE �� LIKE �p�^�[��������
	�@-- �p�^�[��������
	  -- %�@�C�ӂ�0�����ȏ�̕�����
	  -- _ (�A���X�R)�C�ӂ̂P����
	�@-- �@����@LIKE '%1��%�f
	  -- %��_������������Ɏg���Ƃ��� ESCAPE����g�p����
	  --   ESCAPE�����ɑ���%��_�͕����Ƃ��Ĉ�����
	�@-- �@����@LIKE '%100$%' ESCAPE '$'�@-- 100%������������ɂ��Ă���
	�@--   ���ၚ �Q�����̂͂��Ȃ̂Ɂ@= '�H��' ��@like '�H��' �ł͌����ł��Ȃ��Ƃ��́@like '�H��%'�ł��܂������B

### BETWEEN
	WHERE �� BETWEEN �l1 AND �l2 -- �Ӗ��́A�l1�ȏ� ���� �l2�ȉ��@

### IN / NOT IN
	WHERE �� IN (�l1,�l2,�l3,�c)�@-- �l�̂ǂꂩ�ƈ�v�����TRUE
	  --   ����@�����z IN (1000,2000,3000) -- �����z��2000�Ȃ�Βl2�𖞂����̂ŁATRUE

	WHERE �� NOT IN (�l1,�l2,�l3,�c) -- �l�̂ǂ�Ƃ���v���Ȃ����TRUE
	  --   ����@�����z NOT IN (1000,2000,3000) -- �����z��2000�Ȃ�Βl2�𖞂����̂ŁAFALSE

### ANY / ALL
	WHERE �� ANY ���Z�q (�l1,�l2,�l3,�c)
	  -- ��r���Z�q�Ƒg�ݍ��킹�Ăǂꂩ������������TRUE
	  --   ����@WHERE �����z < (1000,2000,3000) -- �����z��2500�Ȃ�Βl3�𖞂����̂ŁATRUE

	WHERE �� ALL ���Z�q (�l1,�l2,�l3,�c)
	  -- ��r���Z�q�Ƒg�ݍ��킹�đS�Ă�����������TRUE
	  --   ����@WHERE �����z < (1000,2000,3000) -- �����z��1000�Ȃ�Βl1�𖞂����Ȃ��̂ŁAFALSE

### �_�����Z�q
	 AND
	 OR
	 NOT
	 -- �D�揇�ʂ́@NOT > AND > OR
	 -- �J�b�R��t����ΗD�悳���
	  --   ����@������1 OR ������2 AND ������3�@�Ɓ@������1 OR (������2 AND ������3)�@�́@���l�ɂȂ�B

	���������܂ޓ��t����
	�@�@2018�N3���ȑO�̃f�[�^�𒊏o����
	�@�@���t <= '2018-03-31'  -- 2018-03-31 00:00:00�Ɖ��߂���DBMS������ƁA2018-03-31 10:30:00�͊܂܂�Ȃ��Ȃ��Ă��܂�
	�@�@���t <  '2018-04-01'  -- ���̂悤�ɏ����ׂ�

 
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �f�[�^�^

### ��{�̃f�[�^�^
	 ����
	  �����l�@INTEGER�^
	  ����    DECIMAI�^ REAL�^ 
	 ������
	  �Œ蒷 CHAR�^
	  �ϒ� VARCHAR�^ 
	 ���t�Ǝ���
	         DATETIME�^
	         DATE�^
	         TIME�^

### SQL SERVER�̃f�[�^�^
	https://docs.microsoft.com/ja-jp/sql/t-sql/data-types/data-types-transact-sql?view=sql-server-ver15

#### �^��
	  decimal�^(numeric�^) �Œ蒷�̗L�������Ə����_�ȉ��ێ������������l�f�[�^�^�ł��B
	                       decimal �� numeric �͓��`�B

	  bigint�^   �����f�[�^���g�p��������f�[�^�^�B-2^63�`2^63-1
	  int�^      �����f�[�^���g�p��������f�[�^�^�B-2^31�`2^31-1(2,147,483,647)
	  smallint�^ �����f�[�^���g�p��������f�[�^�^�B-2^15�`2^15-1(32,767)
	  tinyint�^  �����f�[�^���g�p��������f�[�^�^�B0�`255

	  money�^      ���z�l�܂��͒ʉݒl��\���f�[�^�^�B-922,337,203,685,477.5808 �` 922,337,203,685,477.5807
	  smallmoney�^ ���z�l�܂��͒ʉݒl��\���f�[�^�^�B- 214,748.3648 �` 214,748.3647

	  bit�^ 1�A0�A�܂��� NULL �̒l���Ƃ鐮���^

#### �T�� ���������_���l�f�[�^�Ŏg�p����T���^
	  float�^ - 1.79E+308 ���� -2.23E-308�A0�A����� 2.23E-308 ���� 1.79E+308
	  real�^  - 3.40E + 38 ���� -1.18E - 38�A0�A����� 1.18E - 38 ���� 3.40E + 38

#### ���t�Ǝ���
	  date�^ ����̕����񃊃e�����`��YYYY-MM-DD�B������10 ����
	  time�^ 1���̎������`���܂��B ������ 24 ���Ԍ`���ŁA�^�C�� �]�[���͔F������܂���Bhh:mm:ss[.nnnnnnn] (Informatica �̏ꍇ�� nnn)
	  datetime�^  24���Ԍ`���̎��� (1 �b�����̕b���܂�) �Ƒg�ݍ��킹�����t���`�B
	  datetime2�^  datetime �^���g�����āA���t�͈͂Ɗ���̗L�������𑝂₵�A���[�U�[���K�v�ɉ����ėL���������w��ł���悤�ɂ�������
	  smalldatetime�^ ���t�������Ƒg�ݍ��킹�Ē�`���܂��B ���Ԃ� 24 �����Ƃ��A�b�͏�Ƀ[�� (: 00) �ɂ��A�b�̏������͎g�p���܂���B
	  datetimeoffset�^ �^�C�� �]�[����F������ 24 ���Ԍ`���̎����Ƒg�ݍ��킹�����t���`

#### ������
	  char�^    �Œ�T�C�Y�̕����f�[�^�^�B8000�o�C�g�y�A�܂ł̕�����B
	  varchar�^ �σT�C�Y�̕����f�[�^�^�B8000�o�C�g�y�A�܂ł̕�����B
	  text�^    �傫�Ȕ� Unicode �������i�[���邽�߂̌Œ蒷�̃f�[�^�^�B�폜�\��

#### Unicode������
	  nchar    �Œ�T�C�Y�̕����f�[�^�^�BUnicode �����f�[�^�̑S�͈͂��i�[�B4000�o�C�g�y�A�܂ł̕�����B
	  nvarchar �σT�C�Y�̕����f�[�^�^�BUnicode �����f�[�^�̑S�͈͂��i�[�B4000�o�C�g�y�A�܂ł̕�����B
	  ntext    �傫��Unicode �������i�[���邽�߂̉ϒ��̃f�[�^�^�B�폜�\��

#### ��
	  �o�C�i��������A�ȂǂȂ�


/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## NULL�l���Z�b�g������@

	Ctrl + 0�L�[���������l����ɂ���Enter�L�[�������܂��B

	CSV��NULL�l����ꂽ���ꍇ��NULL�̕�����\N���g�p����΂悢
	
	�f�[�^�̃C���|�[�g�ŁA�\�[�X��EXCEL�̃f�[�^���g���ꍇ�A���ADB���̌^��int�̏ꍇ�́AEXCEL�̃Z����NULL�Ə����΂悢�B
	�f�[�^�̃C���|�[�g�ŁA�\�[�X��EXCEL�̃f�[�^���g���ꍇ�A���ADB���̌^��char�̏ꍇ�́AEXCEL�̃Z���ɉ�������Ă͂Ȃ�Ȃ��B
	
	INSERT���ŁA�l��NULL�Ə���
		INSERT INTO �e�[�u����(��1,��2)
	 	VALUES(�l1,NULL) ;
/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## �f�[�^�C���|�[�g���@

### Excel ���� SQL Server �܂��� Azure SQL Database �Ƀf�[�^���C���|�[�g����
	https://docs.microsoft.com/ja-jp/sql/relational-databases/import-export/import-data-from-excel-to-sql?view=sql-server-ver15

	���@�P�DSQL Server �C���|�[�g����уG�N�X�|�[�g �E�B�U�[�h
	 SQL Server �C���|�[�g����уG�N�X�|�[�g �E�B�U�[�h�̃y�[�W���X�e�b�v���s���āAExcel �t�@�C�����璼�ڃf�[�^���C���|�[�g���܂��B
	 �K�v�ɉ����āA��ŃJ�X�^�}�C�Y���čė��p�ł��� SQL Server Integration Services (SSIS) �p�b�P�[�W�Ƃ��Đݒ��ۑ����܂��B
	 1.SQL Server Management Studio�ŁA SQL Server�f�[�^�x�[�X �G���W���̃C���X�^���X�ɐڑ����܂��B
	 2.[�f�[�^�x�[�X] ��W�J���܂��B
	 3.�f�[�^�x�[�X���E�N���b�N���܂��B
	 4.[�^�X�N] �ɃJ�[�\�������킹�܂��B
	 5.�ȉ��̂����ꂩ�̃I�v�V�������N���b�N���܂��B
	  5-1.�f�[�^�̃C���|�[�g
	�@�@�@���f�[�^�\�[�X��Flat File��I�сA�ϊ����SQL Server Native Client 11.0��I�ԁB
	�@�@�@�@�f�[�^�x�[�X�Ɋ����̃f�|�^�x�[�X�A�}�b�s���O�̕ҏW�ōs�ǉ���I�ԂƁA�����e�[�u���ւ̃f�[�^�ǉ����ł���B
	  5-2.�f�[�^�̃G�N�X�|�[�g
	  5-3.�t���b�g �t�@�C���̃C���|�[�g
	      ��5-3�̕��@�́A�V�K�Ƀe�[�u�����쐬���A�f�[�^���C���|�[�g����̂ŁA�����e�[�u���ւ̃f�[�^�C���|�[�g�͂ł��Ȃ��B

	���f�[�^�ϊ��s�\�ƂȂ��ĕϊ����Ȃ��ꍇ������B�i�ϊ�����N�G���[�͕ۑ��ł���j
	����P�@Double �� varchar�ϊ����Ɣ��肳�ꂽ��
	�@�@���͑��@��ID ���͒l�ɂ�3���̐��l���������Ă��Ȃ��B
	�@�@�o�͑��@�@�@�@ varchar(20)
	�@�@�΍�@�@���͑��̃Z�������𕶎���ɕύX���A���l�̓��ɃV���O���R�[�e�[�V������t���ĕ�����ł��邱�Ƃ𖾎������B

### CSV �t�@�C���� BULK INSERT ���g���ăC���|�[�g����
	https://sql55.com/query/bulk-insert.php

	1.�e�[�u�������
	�@CREATE TABLE Students (
	   StudentID INT NOT NULL PRIMARY KEY,
	   FirstName VARCHAR(50) NULL,
	   LastName VARCHAR(50) NULL,
	   BirthDay DATE NULL,
	   Gender CHAR(1) NULL
	�@);

	2.�C���|�[�g������ CSV �t�@�C����p��
	�@c:\NewStudents.txt
	   1,Taro,Yamada,1980-02-15,M
	   2,Hanako,Tanaka,1979-12-30,F
	   3,Yuko,Suzuki,1979-07-07,F
	   4,Takao,Sato,1980-03-12,M

	3.BULK INSERT ���g���� Students �e�[�u���Ƀf�[�^���C���|�[�g
	 BULK INSERT Students
	  FROM 'c:\NewStudents.txt'
	   WITH
	   (
	     FIELDTERMINATOR = ',',  -- FIELDTERMINATOR �ɂ̓f�[�^����؂镶��
	     ROWTERMINATOR = '\n'    -- ROWTERMINATOR �ɂ͂P�s�̏I������������
	   );

/* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  */
## SELECT���ɂ̂݉\�ȁg�������ʂ����H����h���߂́u���̑��C���v
	SELECT �� 
	  FROM �e�[�u����
	 WHERE (�C��)(���̑��C��);

### DISTINCT  �������ʂ���d���s�����O����B�������[���ʏ����̂ŗ��p�ɒ���
	SELECT DISTINCT �� �c -- ��DISTINCT��SELECT���̍ŏ��ɋL�q����B
	  FROM �e�[�u����

	-- ���� �ƌv��e�[�u���̖��ׂ̒�������ۂɎg���Ă����ڂ𒊏o����
	--      SELECT DISTINCT ��� FROM �ƌv��_TBL

### ORDER BY  �������ʂ̏�������ёւ���B�������[���ʏ����̂ŗ��p�ɒ���
	SELECT �� �c 
	  FROM �e�[�u����
	  ORDER BY ��1 ���я�,  ��2 ���я�, �c -- ��ASC(�����A�ȗ��l),DESC(�~��)

	-- ���� �ƌv��e�[�u���̖��ׂ̒�������z(�~��)�ŕ��ׁA���l�Ȃ�Ώo���z�i�~���j�ɕ��ׂ�
	--      SELECT * FROM �ƌv��_TBL
		    ORDER BY �����z DESC, �o���z DESC

	SELECT ��1,��2, �c 
	  FROM �e�[�u����
	  ORDER BY ��ԍ�1 ���я�,  ��ԍ�2 ���я�, �c -- ����ԍ��́ASELECT���őI�����ꂽ��̃��X�g�̏���

	-- ���� �ƌv��e�[�u���̖��ׂ̒�������z(�~��)�ŕ��ׁA���l�Ȃ�Ώo���z�i�~���j�ɕ��ׂ�
	--      SELECT ���t,���,�����z,�o���z FROM �ƌv��_TBL
		    ORDER BY 3 DESC,  4 DESC

### OFFSET - FETCH �������ʂ��猏�������肵�Ď擾����
	SELECT �� �c 
	  FROM �e�[�u����
	  ORDER BY ��1 ���я�,  ��2 ���я�
	  OFFSET �擪���珜�O����s�� ROWS -- �����O�����ɐ擪����擾����Ƃ��� 0 ROWS ���w�肷��
	  (FETCH NEXT �擾�s�� ROWS ONLY)

	-- ���� �ƌv��e�[�u���̖��ׂ��o���z�̑������ɕ��ׁA���̒�����11�`15�s�ڂ��擾����
	--      SELECT * FROM �ƌv��_TBL
		    ORDER BY �o���z DESC
		    OFFSET 10 ROWS         -- �擪10�s�����O
		    FETCH NEXT 5 ROWS ONLY -- ���̂��Ƃɑ���5�s���擾����
	/* �� OFFSET - FETCH�́ADBMS���ƂɈقȂ�̂ŁA���ӂ��K�v */

### UNION     �a�W���̏W�����Z�q�B�������ʂɂق��̌������ʂ𑫂����킹��B�������[���ʏ����̂ŗ��p�ɒ���
	SELECT ��1
	  UNION (ALL) -- ALL��t���Ȃ��Əd���s��1�s�ɂ܂Ƃ߁AALL��t����Əd���s�����̂܂ܕԂ��B
	SELECT ��2
	
	/* ���� �ƌv��DB�͌����������Ȃ茟�����Ԃ�������悤�ɂȂ����̂ŁA
	   �����Ƃ���ȊO�i�A�[�J�C�u�j�ɕ����ĉ^�p���邱�Ƃɂ����B
	   �������A�P�N�Ԃ̌��ʂ���������Ƃ������������Q�Ƃ������B */
	--  SELECT ���t,���,�����z,�o���z FROM �ƌv��_TBL
	     UNION
	    SELECT ���t,���,�����z,�o���z FROM �ƌv��A�[�J�C�u_TBL
	     ORDER BY 3 DESC, 4 DESC, 2 ASC

    /* �W�����Z�q���g���������ORDER BY����g���Ƃ��̒��ӓ_
	   �� �e�[�u���̗񐔂ƃf�[�^�^�͂҂������v�����Ă����K�v������
	      (�I��񃊃X�g�̐�������Ȃ��Ƃ��́A����Ȃ��ق��̑I��񃊃X�g��NULL��ǉ����邱�ƂŁA������v��������)
	   �� ORDER BY �͍Ō��SELECT���Ɏw�肷��
	   �� UNION��ORDER BY �͒ʏ�A��ԍ��w��ōs���B�񖼂�AS�ɂ��ʖ��̏ꍇ�͍ŏ���SELECT���̂��̂��w�肷��B
	*/

### EXCEPT / MINUS   ���W���̏W�����Z�q�B�������ʂ���ق��̌������ʂ���������(MINUS��OracleDB�̃L�[���[�h)
	SELECT ��1 -- �W��A
	  EXCEPT (ALL)      -- ALL��t���Ȃ��Əd���s��1�s�ɂ܂Ƃ߁AALL��t����Əd���s�����̂܂ܕԂ��B
	SELECT ��2 -- �W��B
	           -- ���ʂ́A�C���[�W�I�ɂ̓}�b�`���O�� 'A��' �ɑ�������

	/* ���� �ƌv��_TBL�ɂ���A�ƌv��A�[�J�C�u_TBL�ɂȂ���ڂ��Q�Ƃ������B
	--  SELECT ��� FROM �ƌv��_TBL
	     EXCEPT
	    SELECT ��� FROM �ƌv��A�[�J�C�u_TBL

	/* �W�����Z�q���g���������ORDER BY����g���Ƃ��̒��ӓ_�́AUNION�ɓ��� */

### INTERSECT �ϏW���̏W�����Z�q�B�������ʂƂق��̌������ʂŏd�����镔�����擾����
	SELECT ��1
	  INTERSECT (ALL)      -- ALL��t���Ȃ��Əd���s��1�s�ɂ܂Ƃ߁AALL��t����Əd���s�����̂܂ܕԂ��B
	SELECT ��2

	/* ���� �ƌv��_TBL�Ɖƌv��A�[�J�C�u_TBL�̗����ɂ����ڂ��Q�Ƃ������B
	--  SELECT ��� FROM �ƌv��_TBL
	     INTERSECT
	    SELECT ��� FROM �ƌv��A�[�J�C�u_TBL

	/* �W�����Z�q���g���������ORDER BY����g���Ƃ��̒��ӓ_�́AUNION�ɓ��� */
