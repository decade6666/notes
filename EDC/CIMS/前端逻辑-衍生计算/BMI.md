
```JS
var height = getTextValue('[VSORRES_HEIGHT]'); //身高
var weigth = getTextValue('[VSORRES_WEIGHT]'); //体重
var bmi;

if(height != '' && weigth != '') {
	if(height <= 0 | weigth < 0) {
		bmi = null;
	}
	else {
		bmi = ((weigth / height / height) * 10000).toRound(1,1);
	}
}
else {
	bmi = null;
}

setTextValue('[VSORRES_BMI]',bmi);
```

![[Pasted image 20251215095722.png]]

