同步基本信息：脱落原因
```SQL
--定义变量;
DECLARE @DSDECOD NVARCHAR(500), @DSDECOD1 NVARCHAR(500), @DSDECOD2 NVARCHAR(500);

--是否脱落选择否且脱落原因不为空时关联脱落原因，否则赋空值;
IF (
		getDataPointValue('[046.DSYN]', Visit_No = 900, VAL) = 'N'
		AND getDataPointValue('[046.DSDECOD]', Visit_No = 900, VAL) <> ''
		AND getDataPointValue('[046.DSDECOD]', Visit_No = 900, VAL) IS NOT NULL
	)--排空
BEGIN
	 --脱落原因字段;
	SET @DSDECOD = getDataPointValue('[046.DSDECOD]', Visit_No = 900, VAL);

	--脱落原因的键名;
	SELECT @DSDECOD1 = Code_Text
	FROM CRF_Codelist
	WHERE Form_id = '046'
		AND Code_RefName = 'clDSDECOD'
		AND Code_Value = @DSDECOD;
	
	--其他脱落原因描述;
	SET @DSDECOD2 = getDataPointValue('[046.DSTERM]', Visit_No = 900, VAL);
	
	--选择其他时关联其他脱落原因描述，否则关联单选的键名;
	IF(getDataPointValue('[046.DSDECOD]', Visit_No = 900, VAL) = 'C.99')
	BEGIN
		setSystemSubjectValue(Subject_Note = @DSDECOD2)
	END
	ELSE
	BEGIN
		setSystemSubjectValue(Subject_Note = @DSDECOD1)
	END
END
ELSE
BEGIN
	setSystemSubjectValue(Subject_Note = '')
END
```