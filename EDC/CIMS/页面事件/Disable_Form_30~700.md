访视内有控制访视下表单的问询（如“是否进行本次访视”），可使用一条逻辑同时激活多个访视，注意是激活的是提交表单的当前访视

```SQL

IF ({visit_no} >= 30 and {visit_no} < 800) --限定触发访视
BEGIN
	DECLARE @VAR1 NVARCHAR(max)
		   ,@VAR2 NVARCHAR(max)
		   ,@VAR3 NVARCHAR(max)
		   ,@VAR4 NVARCHAR(max)
		   ,@VAR5 NVARCHAR(max)
		   ,@VAR6 NVARCHAR(max);
	
	--获取激活的表单
	SET @VAR1 = (
			SELECT form_id + ','
			FROM (
				SELECT CONVERT(NVARCHAR(10), visit_no) + '.' + form_id AS form_id
				FROM CRF_Structure
				WHERE visit_no = {visit_no}
					AND form_id NOT IN (
						'001', '035', '036', '050', '053', '054'
					) --剔除通过其他逻辑激活的表单
				) a1
			ORDER BY form_id
			FOR XML PATH('')
			);
	
	--剔除末尾多余的","
	SET @VAR2 = LEFT(@VAR1, LEN(@VAR1) - 1); 
	
	--拼接上激活语句
	SET @VAR3 = 'ShowVisitForm|' + @VAR2 + ''
	SET @VAR4 = 'SELECT ''' + @VAR3 + ''';'
	
	--拼接上失活语句
	SET @VAR5 = 'HiddenVisitForm|' + @VAR2 + ''
	SET @VAR6 = 'SELECT ''' + @VAR5 + ''';'

	IF (getDataPointValue('[001.SVOCCUR]',Visit_No={visit_no},VAL)='Y')
	BEGIN
		EXEC (@VAR4) --满足条件时显示表单
	END
	ELSE
	BEGIN
		EXEC (@VAR6) --不满足条件时隐藏表单
	END
END

```