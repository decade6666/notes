```SQL
--定义变量
DECLARE @sex nvarchar(max),@birdat nvarchar(max),@lbdat nvarchar(max),
		@scr float,@scru nvarchar(max),@GFR_old float,
		@subject_id nvarchar(max),
		@visit_no nvarchar(max),
		@form_id nvarchar(max),@form_seq nvarchar(max),
		@i float,
		@GFR_RES float,
		@age int, @month_diff int, @day_diff int;

--定义游标，将“人口学资料”拼接到“肾功能”
DECLARE GFR_data cursor for 
	select	a.Subject_Id --筛选号
			,a.Visit_No --访视编号
			,'044' as form_id --肾功能表单ID
			,a.Form_Seq --表单序号
			,a.LBORRES_1 --肌酐结果
			,a.LBORRESU_1 --肌酐单位
			,a.LBORRES2 --肾小球滤过率结果
			,a.LBDAT --肾功能检查日期
			,c.SEX --性别
			,c.BRTHDAT --出生日期
		from F_044 a --肾功能
		left join F_005 c on a.Subject_Id = c.Subject_Id --人口学资料
		where a.Subject_Id = '{subject_id}'
		order by	a.Subject_Id,a.Visit_No,a.Form_Seq

--打开游标，将游标数据存放在自定义变量中
open GFR_data

--fetch next：第一次调用时获取第一行
fetch next from GFR_data into @subject_id,@visit_no,@form_id,@form_seq,@scr,@scru,@GFR_old,@lbdat,@sex,@birdat

--fetch_status：0，Fetch语句成功；-1，Fetch语句失败或行不在结果集中；-2，提取的行不存在。
while @@fetch_status=0

BEGIN
	--计算前排除空值
    IF(
		@scr is not null AND (isnull(@scru,'')<>'' 
		AND LOWER(replace(@scru,' ','')) in (LOWER('mg/dL'), LOWER('μmol/L')) 
		AND isnull(@lbdat,'')<>'' 
		AND isnull(@birdat,'')<>'' 
		AND isnull(@sex,'')<>''
	)
	BEGIN
		--计算周岁年龄
		IF(ISNULL(@lbdat,'')<>'' AND ISNULL(@birdat, '')<>'')
		BEGIN
			SET @age        = YEAR(@lbdat)  - YEAR(@birdat);
			SET @month_diff = MONTH(@lbdat) - MONTH(@birdat);
			SET @day_diff   = DAY(@lbdat)   - DAY(@birdat);

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
	 
		--单位转化，参数赋值
		SET @i=case LOWER(replace(@scru,' ','')) 
			when LOWER('μmol/L') then 1
			when LOWER('mg/dL') then 88.402
		END

		--计算GFR
		IF(isnull(@age,'')<>'')
		BEGIN
			IF(@sex='F')
			BEGIN
				IF(cast(@scr*@i as numeric(20,10))<=62)
				BEGIN
					SET @GFR_RES=Cast( round( Cast( 1.018 * Cast( Cast( 141 * Cast( POWER( Cast( Cast( @scr * @i AS DECIMAL(18, 12) ) / 62 AS DECIMAL(18, 12)) , -0.329) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) * Cast( POWER(0.9938, @age) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) , 2) AS DECIMAL(18, 12))
				END
				ELSE IF(cast(@scr*@i as numeric(20,10))>62)
				BEGIN
					SET @GFR_RES=Cast( round( Cast( 1.018 * Cast( Cast( 141 * Cast( POWER( Cast( Cast( @scr * @i AS DECIMAL(18, 12) ) / 62 AS DECIMAL(18, 12)) , -1.209) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) * Cast( POWER(0.9938, @age) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) , 2) AS DECIMAL(18, 12))			
				END
			END
			ELSE IF(@sex='M')
			BEGIN
				IF(cast(@scr*@i as numeric(20,10))<=80)
				BEGIN
					SET @GFR_RES=Cast( round( Cast( Cast( 141 * Cast( POWER( Cast( Cast( @scr * @i AS DECIMAL(18, 12) ) / 80 AS DECIMAL(18, 12)) , -0.411) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) * Cast( POWER(0.993, @age) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) , 2) AS DECIMAL(18, 12))
				END
				ELSE IF(cast(@scr*@i as numeric(20,10))>80)
				BEGIN
					SET @GFR_RES=Cast( round( Cast( Cast( 141 * Cast( POWER( Cast( Cast( @scr * @i AS DECIMAL(18, 12) ) / 80 AS DECIMAL(18, 12)) , -1.209) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) * Cast( POWER(0.993, @age) AS DECIMAL(18, 12)) AS DECIMAL(18, 12)) , 2) AS DECIMAL(18, 12))
				END
			END
			ELSE
			BEGIN
				SET @GFR_RES=NULL
			END
		END
		ELSE
		BEGIN
			SET @GFR_RES=NULL
		END
	END


	--判定更新
	IF(isnull(@GFR_RES,'') <> isnull(@GFR_old,''))
	BEGIN
		--轨迹记录数据更新
		INSERT INTO dbo.LOG_SM_Field (Entry_Seq, Site_No, Subject_Id, Visit_No, Form_Id, Form_Seq, Field_name, Old_Value, New_Value, Operation_Date, Operation_User, Operation_Type, Operation_Note, Operation_IP, Operation_Environment) SELECT '{entry_seq}','{site_no}', @subject_id, @visit_no, @form_id, @form_seq, 'LBORRES2',@GFR_old,@GFR_RES,getdate(),'System','Update','For Trigger','',''
		UPDATE dbo.F_044 SET LBORRES2=@GFR_RES WHERE subject_id=@subject_id AND Visit_No=@visit_no and Form_Seq=@form_seq 
	END
	--循环下一次结果
    fetch next from GFR_data into @subject_id,@visit_no,@form_id,@form_seq,@scr,@scru,@GFR_old,@lbdat,@sex,@birdat
END
close GFR_data
deallocate GFR_data
```