```JS
var age		= getDataPointValue('[004.AGE]',Visit_No=10,VAL).toNum(); //年龄
var weight	= getDataPointValue('[006.VSORRES_WEIGHT]',Visit_No=10,VAL).toNum(); //体重
var sex		= getDataPointValue('[004.SEX]',Visit_No=10,VAL); //性别(男/女)
var creat	= getTextValue('[LBORRES_5]').toNum(); //肌酐
var unit	= getTextValue('[LBORRESU_5]'); //肌酐单位

var ccr;

if (age != '' && weight != '' && sex != '' && creat != '' && unit != '') {

	//调试：控制台输出数据
//	console.log('AGE ：',AGE);
//	console.log('WEIGHT ：',WEIGHT);
//	console.log('sex ：',sex);
//	console.log(' CREAT：',CREAT);
//	console.log(' unit：',unit); 

	//根据单位和性别计算
	if ((unit === "mg/dL" || unit === "mg/dl") && sex == 'M') {
		ccr = ((140-age) * weight / (72 * creat)).toround(2,1);
	}
	else if ((unit === "μmol/L" || unit === "umol/L") && sex == 'M') {
		ccr = ((140-age) * weight / (72 * creat / 88.4)).toround(2,1);
	}
	else if ((unit === "mg/dL" || unit === "mg/dl") && sex == 'F') {
		ccr = ((140-age) * weight / (85 * creat)).toround(2,1);
	}
	else if ((unit === "μmol/L" || unit === "umol/L") && sex == 'F') {
		ccr = ((140-age) * weight / (85 * creat / 88.4)).toround(2,1);
	}
	else {
		ccr = null;
		console.log('警告：肌酐单位（',unit,'）不在要求范围内(mg/dL、mg/dl、umol/L或μmol/L)');
	}
}
else {
	ccr = null;
	console.log('警告：数据填写不完整，肌酐清除率无法计算');
}

setTextValue('[LBORRES1]',ccr); //肌酐清除率
```

![[Pasted image 20251215095448.png]]
![[Pasted image 20251215095722.png]]
![[Pasted image 20251215103821.png]]