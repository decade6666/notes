**受试者状态：**


|         |              |     |     |      |     |     |      |     |         |        |     |
| ------- | ------------ | --- | --- | ---- | --- | --- | ---- | --- | ------- | ------ | --- |
| ==000== | ==筛选中==      |     | 001 | 待筛选  |     | 005 | 知情失败 |     | ==010==     | ==筛选失败==   |     |
| 011     | 待审核          |     | 015 | 审核驳回 |     | 020 | 待入组  |     | 025     | 不入组    |     |
| ==030== | ==已入组(进行中)== |     | 031 | 治疗中  |     | 032 | 随访中  |     | ==040== | ==脱落== |     |
| ==050== | ==已完成==      |     |     |      |     |     |      |     |         |        |     |


```SQL

IF (getDataPointValue('[007.DSYN]', Visit_No = 900, VAL) = 'Y')
BEGIN
	setSystemSubjectValue(Subject_Status = '050');--已完成
END
ELSE IF (getDataPointValue('[007.DSYN]', Visit_No = 900, VAL) = 'N')
BEGIN
	setSystemSubjectValue(Subject_Status = '040');--脱落
END
ELSE IF (
		getDataPointValue('[010.DSYN]', Visit_No = 20, VAL) <> '' 
		AND getDataPointValue('[010.DSYN]', Visit_No = 20, VAL) IS NOT NULL
)
BEGIN
	IF (getDataPointValue('[010.DSYN]', Visit_No = 20, VAL) = 'Y')
	BEGIN
		setSystemSubjectValue(Subject_Status = '030');--已入组（进行中）
	END
	ELSE IF (getDataPointValue('[010.DSYN]', Visit_No = 20, VAL) = 'N')
	BEGIN
		setSystemSubjectValue(Subject_Status = '010');--筛选失败
	END
END
ELSE
BEGIN
	setSystemSubjectValue(Subject_Status = '000');--筛选中
END

```