```SQL

-- 变量声明
DECLARE @AGE nvarchar(max)
DECLARE @SEX nvarchar(max)
DECLARE @WEIGHT nvarchar(max)
DECLARE @SCR nvarchar(max)
DECLARE @UNIT nvarchar(max)
DECLARE @RESULT float

-- 取值并转化类型
SET @AGE    = getDataPointValue('[004.AGE]',Visit_No=10,VAL) --年龄，若使用检查日期需重新计算年龄，参考GFR的计算步骤
SET @SEX    = getDataPointValue('[004.SEX]',Visit_No=10,VAL) --性别
SET @WEIGHT = getDataPointValue('[006.VSORRES_WEIGHT]',Visit_No=10,VAL) --体重
SET @SCR  = getDataPointValue('[188.LBORRES_5]',Visit_No=10,VAL) --肌酐结果
SET @UNIT   = UPPER(getDataPointValue('[188.LBORRESU_5]',Visit_No=10,VAL)) --肌酐单位

-- 计算前排除空值
IF     ISNULL(@AGE,'') <> '' 
   AND ISNULL(@WEIGHT,'') <> '' 
   AND ISNULL(@SEX,'') <> ''
   AND ISNULL(@SCR,'') <> '' 
   AND ISNULL(@UNIT,'') <> ''
BEGIN
    -- 判断性别和单位，计算肾小球滤过率结果，保留2位小数
    IF (@UNIT = UPPER('mg/dL') AND @SEX = 'M')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (72*Cast(@SCR as decimal(20,15))), 2)
    ELSE IF (@UNIT = UPPER('μmol/L') AND @SEX = 'M')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (72*Cast(@SCR as decimal(20,15))/88.4), 2 )
    ELSE IF (@UNIT = UPPER('mg/dL') AND @SEX = 'F')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (85*Cast(@SCR as decimal(20,15))), 2)
    ELSE IF (@UNIT = UPPER('μmol/L') AND @SEX = 'F')
        SET @RESULT = ROUND( ((140-Cast(@AGE as decimal(20,15)))*Cast(@WEIGHT as decimal(20,15))) / (85*Cast(@SCR as decimal(20,15))/88.4),2 )
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