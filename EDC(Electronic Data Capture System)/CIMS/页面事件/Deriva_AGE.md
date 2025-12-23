 若首次知情同意书签署日期的月日整体大于或等于出生日期的月日，则年龄=首次知情同意书签署年份-出生日期年份；若首次知情同意书签署日期的月日整体小于出生日期的月日，则年龄=首次知情同意书签署年份-出生日期年份-1。

```SQL
--定义变量
DECLARE @age int, @month_diff int, @day_diff int;
DECLARE @DSSTDAT nvarchar(200),@BRTHDAT nvarchar(200);

--知情同意书签署日期，如果表单类型是Log Form且确认第1行是首次签署知情，"Visit_No=XX"后面需要加上 "And Form_seq=1"
set @DSSTDAT = getDataPointValue('[009.DSSTDAT]',Visit_No=10,VAL) 
set @BRTHDAT = getDataPointValue('[006.BRTHDAT]',Visit_No=10,VAL) --出生日期

IF(ISNULL(@DSSTDAT,'')<>'' AND ISNULL(@BRTHDAT, '')<>'')
BEGIN
	SET @age        = YEAR(@DSSTDAT)  - YEAR(@BRTHDAT);
	SET @month_diff = MONTH(@DSSTDAT) - MONTH(@BRTHDAT);
	SET @day_diff   = DAY(@DSSTDAT)   - DAY(@BRTHDAT);

	IF(	@month_diff < 0 OR ( @month_diff = 0 AND @day_diff < 0 ) )
	BEGIN
		SET @age = @age - 1;
	END

	IF(@age < 0)
	BEGIN
		SET @age = NULL;
	END
	
END
ELSE
BEGIN
	SET @age = NULL
END


setDataPointValue('[006.AGE]',Visit_No=10,@age)--年龄
```