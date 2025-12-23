```SQL

-- 变量声明
DECLARE @AGE nvarchar(max)
DECLARE @SEX nvarchar(max)
DECLARE @WEIGHT nvarchar(max)
DECLARE @CREAT nvarchar(max)
DECLARE @UNIT nvarchar(max)
DECLARE @RESULT float

-- 取值并转化类型
SET @AGE    = getDataPointValue('[004.AGE]',Visit_No=10,VAL)
SET @SEX    = getDataPointValue('[004.SEX]',Visit_No=10,VAL)
SET @WEIGHT = getDataPointValue('[006.VSORRES_WEIGHT]',Visit_No=10,VAL)
SET @CREAT  = getDataPointValue('[188.LBORRES_5]',Visit_No=10,VAL)
SET @UNIT   = UPPER(getDataPointValue('[188.LBORRESU_5]',Visit_No=10,VAL))

-- 计算前排除空值
IF     ISNULL(@AGE,'') <> '' 
   AND ISNULL(@WEIGHT,'') <> '' 
   AND ISNULL(@SEX,'') <> ''
   AND ISNULL(@CREAT,'') <> '' 
   AND ISNULL(@UNIT,'') <> ''
BEGIN
    -- 判断性别和单位，计算CCR结果，保留2位小数
    IF (@UNIT = UPPER('mg/dL') AND @SEX = 'M')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (72*Cast(@CREAT as decimal(20,15))), 2)
    ELSE IF (@UNIT = UPPER('μmol/L') AND @SEX = 'M')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (72*Cast(@CREAT as decimal(20,15))/88.4), 2 )
    ELSE IF (@UNIT = UPPER('mg/dL') AND @SEX = 'F')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (85*Cast(@CREAT as decimal(20,15))), 2)
    ELSE IF (@UNIT = UPPER('μmol/L') AND @SEX = 'F')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (85*Cast(@CREAT as decimal(20,15))/88.4),2 )
    ELSE
        SET @RESULT = NULL
    
    -- 输出结果
    setDataPointValue('[188.LBORRES1]',Visit_No=10,@RESULT)

END
ELSE
BEGIN
	    -- 数据不全，无法计算
    setDataPointValue('[188.LBORRES1]',Visit_No=10,null)
END
```