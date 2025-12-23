
# 代码
```JS
var brthdat = getTextValue('[BRTHDAT]'); //出生日期

//知情同意书签署日期，如果表单类型是Log Form且确认第1行是首次签署知情，"Visit_No=XX"后面需要加上 "And Form_seq=1"
var icfdat = getDataPointValue('[001.DSSTDAT]',Visit_No=10,VAL);

var age;

if(brthdat != '' && icfdat != '') {
	age = Datepart('yyyy',icfdat).toNum()  - Datepart('yyyy',brthdat).toNum() ;
	var month_diff = Datepart('MM',icfdat).toNum()  - Datepart('MM',brthdat).toNum() ;
	var day_diff = Datepart('dd',icfdat).toNum()  - Datepart('dd',brthdat).toNum() ;

	if (
		month_diff<0 || (month_diff == 0 && day_diff<0)
	) {
		age--;
    }

	if (age < 0) {
		age = null;
	}
	
} else {
	age = null;
}

setTextValue('[AGE]',age); //年龄

```

![[Pasted image 20251215095448.png]]
![[Pasted image 20251215095413.png]]

