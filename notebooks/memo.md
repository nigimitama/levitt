# Levittのコードを解読してみる

## First Stage

firststa.do にある。
わりと読みやすいのでそのまま読める


## Second Stage

convert.doでSAS用にデータの前処理をしてからsecstage.sasでさらに前処理して分析している。
変数が多くて読みにくい。

### convert.do

7つのcrime categoriesにあわせて縦持ちにしている

```do
drop if city==.  * 欠損値の削除

expand 7 * 7倍にレコード数を増やす

* id代わりにnumberを生成
sort city year
generate number=1
replace number=number[_n-1]+1 if city==city[_n-1]&year==year[_n-1]
```

犯罪の内容に応じてラベルづけ

```do
replace  crime=murd if number==1
replace  crime=rape if number==2
replace  crime=robb if number==3
replace  crime=assa if number==4
replace crime=burg if number==5
replace crime=larc if number==6
replace crime=auto if number==7
```


人口10万人あたりに処理

```do
replace sworn=sworn/citypop*100000
replace civil=civil/citypop*100000
...（略）
```

### secstage.sas

```sas
proc syslin data=pol_tem2 out=pol_tem2 2sls ; 
	title 'all crimes  2sls using elections  -- unrestricted';
	instruments  %vele_in1 %vele_in2 %pele_in1 %pele_in2
			 %vdifcov %pdifcov %citdum  
			%vcitsiz %pcitsiz %vyears %pyears ;
 	endogenous  d_crim %vdswo_1 %vdswo_2 %pdswo_1 %pdswo_2;
	model d_crim= %vdswo_1 %vdswo_2 %pdswo_1 %pdswo_2 %vdifcov
			%pdifcov %citdum  
			%vcitsiz %pcitsiz %vyears %pyears / overid ;
```

例えばこのモデルは

```
犯罪の変化率（d_crim）
~ vdswo_1（暴力系犯罪 violentのときだけ0じゃなく値が入っている sworn policeの変化率）
 + vdswo_2（暴力系犯罪 violentのときだけ0じゃなく値が入っている sworn policeの変化率）
 + pdswo_1（窃盗系犯罪 property のときだけ0じゃなく値が入っている sworn policeの変化率）
 + pdswo_2（窃盗系犯罪 property のときだけ0じゃなく値が入っている sworn policeの変化率）
 + vdifcov + pdifcov (共変量；女性家主や黒人比率、失業率や福祉への歳出など）
 + citdum（都市ダミー; cc1 ~ cc59）
 + vcitsiz + pcitsiz （都市サイズ）
 + vyears + pyears（年ダミーだが生成がちょっと特殊かも）
 + [
     d_crim + vdswo_1 + vdswo_2 + pdswo_1 + pdswo_2
     ~ 
     vele_in1 + vele_in2 + pele_in1 + pele_in2 （election）
     + vdifcov + pdifcov （共変量）
     + citdum（都市ダミー）
     + vcitsiz + pcitsiz（都市サイズ）
     + vyears + pyears （年ダミー）
    ]
```



#### vele_in1, pele_in1: 操作変数

- vele_in1はviolent crime
- pele_in1はproperty crime

```sas
%macro vele_in1;	im1_el1 im2_el1 im3_el1 im4_el1
		ig1_el1 ig2_el1 ig3_el1 ig4_el1
%mend;		

%macro pele_in1;	 im5_el1 im6_el1 im7_el1 
		 ig5_el1 ig6_el1 ig7_el1 
%mend;
```

im1_el2は、`市長選(im), murder(1), 一昨年？が選挙(elec_2)`

```do
replace im1_el1=1 if elec_1==1&number==1
replace im2_el1=1 if elec_1==1&number==2
```

#### vdifcov

covariatesのdifferencial（変化率）

```sas
%macro vdifcov;	dd1_fem dd2_fem dd3_fem dd4_fem
		dd1_bla dd2_bla dd3_bla dd4_bla
		dd1_wel dd2_wel dd3_wel dd4_wel
		dd1_edu dd2_edu dd3_edu dd4_edu
		dd1_une dd2_une dd3_une dd4_une
		dd1_age dd2_age dd3_age dd4_age
%mend;
```

dd1_femはnumberが1以外のときにfemの変化 `c_fem = cityfemh-cityfem[_n-1]` をとる

```do
gen dd1_fem=c_fem
gen dd2_fem=c_fem
...
replace dd1_fem=0 if number!=1
replace dd2_fem=0 if number!=2
```

dd1_blaも同様 `c_black=citybla-citybla[_n-1]`

unempも同様（元が割合なら対数はとらない）

`c_unemp=unemp-unemp[_n-1]`

```
gen a15_24=a15_19+a20_24
gen c_a15_24=a15_24-a15_24[_n-1]
```


dd1_wel も 似ているが対数をとって変化率にしている

```do
gen c_st_edu=ln(sta_edu)-ln(sta_edu[_n-1])
gen c_st_wel=ln(sta_wel)-ln(sta_wel[_n-1])
```


### 共変量 d


```do
gen d_sworn=ln(sworn)-ln(sworn[_n-1]) if city==city[_n-1]
gen d_civil=ln(civil)-ln(civil[_n-1]) if city==city[_n-1]

gen d_swo_1=d_sworn[_n-1] if year>69& (city==city[_n-2])
gen d_civ_1=d_civil[_n-1] if year>69& (city==city[_n-2])

gen d_swo_2=d_sworn[_n-2] if year>69& (city==city[_n-3])
```

#### pdifcov

```sas
%macro pdifcov; dd5_fem dd6_fem dd7_fem 
 		dd5_bla dd6_bla dd7_bla 
 		dd5_wel dd6_wel dd7_wel
 		dd5_edu dd6_edu dd7_edu
 		dd5_une dd6_une dd7_une
 		dd5_age dd6_age dd7_age
%mend;
```


