如果脱落原因与多个选项关联，使用此逻辑
```SQL
--定义变量
DECLARE @DSDECOD nvarchar(100)
		,@DSDECOD1 nvarchar(100)
		,@DSDECOD2 nvarchar(100)
		,@DSDECOD3 nvarchar(100)
		,@DSDECOD4 nvarchar(500)
		
--获取数据
SELECT @DSDECOD=CASE DSDECOD 
	WHEN '1' THEN '撤回知情同意书' 
	WHEN '2' THEN '失访' 
	ELSE '' 
	END 
FROM F_035  WHERE subject_id='{subject_id}' AND visit_no='{visit_no}' AND form_seq='{form_seq}'

SELECT @DSDECOD1=CASE DSDECOD1
	WHEN '1' THEN  '使用方案规定以外的影响疗效的合并药物/非药物治疗'
	WHEN '2' THEN  '不良事件'
	WHEN '3' THEN  '受试者用药依从性差'
	ELSE ''  
	END
FROM F_035  WHERE subject_id='{subject_id}' AND visit_no='{visit_no}' AND form_seq='{form_seq}'


SELECT @DSDECOD2=CASE DSDECOD2 
	WHEN '3' THEN '泄盲或紧急揭盲' 
	ELSE ''  
	END 
FROM F_035  WHERE subject_id='{subject_id}' AND visit_no='{visit_no}' AND form_seq='{form_seq}'

SELECT @DSDECOD3=CASE DSDECOD3
	WHEN '4' THEN '试验终止' 
	ELSE ''  
	END 
FROM F_035  WHERE subject_id='{subject_id}' AND visit_no='{visit_no}' AND form_seq='{form_seq}'

SELECT @DSDECOD4=DSTERM
FROM F_035  WHERE subject_id='{subject_id}' AND visit_no='{visit_no}' AND form_seq='{form_seq}'

--关联脱落原因
IF (ISNULL(@DSDECOD,'')<>'')
BEGIN
	UPDATE SM_Subject SET Subject_Note=@DSDECOD where subject_id='{subject_id}'
END
ELSE IF(ISNULL(@DSDECOD1,'')<>'')
BEGIN
	UPDATE SM_Subject SET Subject_Note=@DSDECOD1 WHERE subject_id='{subject_id}'
END
ELSE IF(ISNULL(@DSDECOD2,'')<>'')
BEGIN
	UPDATE SM_Subject SET Subject_Note=@DSDECOD2 WHERE subject_id='{subject_id}'
END
ELSE IF(ISNULL(@DSDECOD3,'')<>'')
BEGIN
	UPDATE SM_Subject SET Subject_Note=@DSDECOD3 WHERE subject_id='{subject_id}'
END
ELSE 
BEGIN
	UPDATE SM_Subject SET Subject_Note=@DSDECOD4 WHERE subject_id='{subject_id}'
END


```