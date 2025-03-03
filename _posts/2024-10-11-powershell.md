### switch语法
switch语法的break用于中断后续匹配，当只有一个匹配时break可有可无，有多个匹配时没有break则switch会返回所有的匹配项
```powershell
PS /home/user/Desktop> switch(2){2 {2;break};{$_ -ge 0} {3}}
2

switch(2){2 {2};{$_ -ge 0} {3}}      
2
3
```

### 获取方法定义
先通过get-member获取到方法，再调用.Definition方法
```powershell
([DateTime]"2023-12-24 11:00:39"|Get-Member -Name getDateTimeFormats).Definition
```


### string format
通过占位符格式化字符串，占位符从0开始,-f选项指定后续参数，需要用逗号分隔

```powershell
PS /home/user/Desktop> "{0} - {1} -- {2}" -f 1,2,3                                                    
1 - 2 -- 3
```
格式化时同时也可以指定格式化后的格式，比如b表示二进制，x表示16进制，p表示百分比
```powershell
"{0:b} - {1:x} -- {2:p}" -f 3,77,3
11 - 4d -- 300.000%
```
还可以指定格式化后的补全位数，p后面的数字表示百分比小数后的位数，为0时表示整数百分比
```powershell
"{0:b4} - {1:x4} -- {2:p2} - {3:p0}" -f 3,77,3,0.1
0011 - 004d -- 300.00% - 10%
```


