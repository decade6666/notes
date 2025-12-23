
```JS
var hr = getTextValue('[EGORRES1_HR]'); //HR
var qt = getTextValue('[EGORRES1_QT]'); //QT
var qtcf;

if(hr != '' && qt != '') {
	qtcf = qt / (Math.pow(60/hr,0.33)).toRound(1,1);
}
else {
	qtcf = null;
}

setTextValue('[EGORRES1_QTCF]',qtcf);
```
![[Pasted image 20251223100509.png]]