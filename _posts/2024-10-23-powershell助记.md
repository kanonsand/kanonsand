### 比较
不能直接用 > < =,需要用-gt -lt -eq等来代替，否则结果会有问题

## 文件hash，md5，sha256等均支持
Get-FileHash /home/user/Desktop/test.json -Algorithm md5

## 设置别名
set-alias pwd get-location


## pushd popd和linux相同
pwd
Pushd ..
popd
pushd ..

## head前两行
get-content -Head 2 ./test.json

## select-string (grep),并且只打印匹配内容
(cat test.json |Select-String -Pattern '(?<=uid":")\d+' -AllMatches ).Matches.Value
拿到匹配结果整个对象，然后获取Matches，再获取其Value

## 获取当前路径下的文件并按照某个属性排序
Get-ChildItem|Sort-Object -Property size -Descending
Get-ChildItem|Sort-Object -Property lastwritetime -Descending|Format-Table unixmode
Get-ChildItem|select name|Sort-Object -Unique

还可以用指定的表达式进行排序
cat /tmp/cc|Sort-Object -Property @{Expression={($_ -split '\s+')[0]};Des     cending=$true}

## 获取指定格式的时间
Get-Date -Format 'yyyy-MM-dd HH:mm:ss'

## 生成随机数
Get-Random -Minimum 0 -Maximum 100

## 操作剪切板
Get-Clipboard
Set-Clipboard ""

## map
```powershell
PS /home/user/Desktop> @{aa="aaa";bb="bb"} |select keys  

Keys
----
{bb, aa}

PS /home/user/Desktop> @{aa="aaa";bb="bb"} |% {$_.Keys}
bb
aa
```

## substring
Get-Process |% {$_.name}|% {if($_.length -ge 4){$_.substring(0,4)}else{$_.substring(0,$_.length)}}

## 正则替换
注意正则替换时pattern和替换文本之间有个逗号
```powershell
PS /home/user/Desktop> "aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','${B} ${A}'   bbb aaa               
PS /home/user/Desktop> "aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','$B $A'    
$B $A
PS /home/user/Desktop> "aaa bbb" -replace '(\w+) (\w+)','\2 \1'         
\2 \1
PS /home/user/Desktop> "aaa bbb" -replace '(\w+) (\w+)','${2} ${1}'   
bbb aaa
PS /home/user/Desktop> "aaa bbb" -replace '(\w+) (\w+)','$2 $1'    
bbb aaa
```
注意到非命名分组时，$1和${1}均可，命名分组时则必须使用${name}
特别注意，一定要用单引号，否则$xx会被解析为变量,如下输出空字符串
```powershell
"aaa bbb" -replace '(\w+) (\w+)',"$2 $1"
```
如果确定要在双引号中使用，则要用`将$进行转义
```powershell
"aaa bbb" -replace '(\w+) (\w+)',"`$2 `$1"
bbb aaa
```

## 类似awk处理文本
```powershell
#读取cipher，每一行按照空格分成字符串数组sp,sp[0]为key，sp[1]为value放进map
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap.add($sp[0],$sp[1])} {$mymap}
#map.add(k,v)和map[k]=[v]的区别是add方法在已存在的情况下会报错，第二个则不会
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]=$sp[1]} {$mymap}
#完整的写法需要指定-Begin -Process -End，默认可以不明确指定，写出来更清晰
cat ./cipher |% -Begin {$mymap=@{}} -Process {$sp=$_ -split '\s+';$mymap[$sp[0]]=$sp[1]} -End {$mymap}

#powershell会自动推断类型，这里将每行第一个单词进行计数
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]++} {$mymap}

cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]--} {$mymap}
```

## 批量重命名文件
Rename-Item NewName
```powershell
Get-ChildItem|Rename-Item -NewName {$_.Name -replace '$','.jpg'}
```

## list 操作
### map filter 
map，filter可以用ForEach和Where模拟
```powershell
PS /home/user/Desktop> $arr=@(1,2,3)
PS /home/user/Desktop> $arr
1
2
3
PS /home/user/Desktop> $arr.ForEach{$_}
1
2
3
PS /home/user/Desktop> $arr.ForEach{"$_ aa"}
1 aa
2 aa
3 aa
PS /home/user/Desktop> $arr.Where{$_ -ge 2}
2
3
PS /home/user/Desktop> $arr.ForEach{$_+1}.where{$_ -ge 2}
2
3
4
#这条和上面等价，上面的省略了小括号
PS /home/user/Desktop> $arr.ForEach({$_+1}).where({$_ -ge 2})
2
3
4
``` 

### join
```powershell
PS > $arr -join ","
```

### add
没有显式指定类型的list不能调用add方法，但是可以调用+=方法来添加元素
```powershell
PS /home/user/Desktop> $arr=@(1,2,3)
PS /home/user/Desktop> $arr+=3       
PS /home/user/Desktop> $arr
1
2
3
3
PS /home/user/Desktop> $arr.Add(4)
MethodInvocationException: Exception calling "Add" with "1" argument(s): "Collection was of a fixed size."
```

### 强类型list
默认的list是无类型的，也就是Object数组，可以放入任意类型的数据
```powershell
PS /home/user/Desktop> $arr=@(1,2,3,4)
PS /home/user/Desktop> $arr.GetType()   

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array
PS /home/user/Desktop> $arr+="a"     
PS /home/user/Desktop> $arr
1
2
3
3
a
```

默认的list等价于如下的构造
```powershell
PS /home/user/Desktop> $arr=@(1,2,3)                                      
PS /home/user/Desktop> $arr.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

PS /home/user/Desktop> $arr=[Object[]]@(1,2,3)
PS /home/user/Desktop> $arr.GetType()         

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array
```

可以在声明时指定list的类型
```powershell
PS /home/user/Desktop> $arr=[Int[]]@(1,2,3)
PS /home/user/Desktop> $arr.GetType()      

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32[]                                  System.Array

#int数组同样不支持add
PS /home/user/Desktop> $arr.Add(4)   
MethodInvocationException: Exception calling "Add" with "1" argument(s): "Collection was of a fixed size."

#支持+=
PS /home/user/Desktop> $arr+=4    
PS /home/user/Desktop> $arr
1
2
3
4

#+=可以加入不同类型，同时arr的类型会改变，相当于返回了一个新的arr
PS /home/user/Desktop> $arr=[Int[]]@(1,2,3)
PS /home/user/Desktop> $arr.gettype()      

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Int32[]                                  System.Array

PS /home/user/Desktop> $arr+="a"           
PS /home/user/Desktop> $arr.gettype()      

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array


#非泛型的arraylist，可以add任意类型，+=后变成Object[]
PS /home/user/Desktop> $arr=[System.Collections.ArrayList]@(1,2,3)
PS /home/user/Desktop> $arr.getType()                             

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     ArrayList                                System.Object

PS /home/user/Desktop> $arr.Add("a") 
3
PS /home/user/Desktop> $arr
1
2
3
a
#调用+=后类型会变成Object[]
PS /home/user/Desktop> $arr+=4      
PS /home/user/Desktop> $arr
1
2
3
a
4
PS /home/user/Desktop> $arr.gettype()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

#指定泛型类型的list,只能add指定类型，+=返回一个Object[]
PS /home/user/Desktop> $arr=[System.Collections.Generic.List[int]]@(1,2,3)
PS /home/user/Desktop> $arr.GetType()                                     

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     List`1                                   System.Object

PS /home/user/Desktop> $arr.Add(1)   
PS /home/user/Desktop> $arr.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     List`1                                   System.Object

PS /home/user/Desktop> $arr.Add("a") 
MethodException: Cannot convert argument "item", with value: "a", for "Add" to type "System.Int32": "Cannot convert value "a" to type "System.Int32". Error: "The input string 'a' was not in a correct format.""
PS /home/user/Desktop> $arr+=1      
PS /home/user/Desktop> $arr.GetType()

IsPublic IsSerial Name                                     BaseType
-------- -------- ----                                     --------
True     True     Object[]                                 System.Array

```
***总结：任意类型的list都可以使用+=操作，可以添加任何类型，返回一个新的Object[]，而add方法不改变当前list的类型，对于数组该方法报错，对于有类型的list该方法只能添加对应的类型***


### 预定义list的大小
默认的list其实是数组，可以在初始化时指定大小
```powershell
PS /home/user/Desktop> $arr=[Object[]]::new(4)
PS /home/user/Desktop> $null -eq $arr[0]
True

PS /home/user/Desktop> $arr=[Int[]]::new(4)   
PS /home/user/Desktop> 0 -eq $arr[0]       
True

#数组支持*操作，可以将当前数组复制数倍，同理支持*=操作
PS /home/user/Desktop> $arr=[Int[]]@(1,2)
PS /home/user/Desktop> $arr*=4           
PS /home/user/Desktop> $arr              
1
2
1
2
1
2
1
2
```
### 访问list中的某一段
```powershell
PS /home/user/Desktop> $arr=0..4              
PS /home/user/Desktop> $arr[0..3]             
0
1
2
3
PS /home/user/Desktop> $arr[3..0]
3
2
1
0
PS /home/user/Desktop> $arr[0,2,3]
0
2
3
PS /home/user/Desktop> $arr[-1]   
4
PS /home/user/Desktop> $arr[-1..2]
4
0
1
2
```


### 二维list
```powershell
PS /home/user/Desktop> $arr=@(@(1,2),@(3,4))
PS /home/user/Desktop> $arr[0][0]           
1
```

cat /tmp/ffd |Group-Object
cat /tmp/ffd |Group-Object|get-member
cat /tmp/ffd |Group-Object |select count,name
Get-Alias
Get-Alias|Select-String -Pattern group
Get-Alias|Select-String -Pattern "group"
Get-Alias|Select-String group
Get-Alias|Select-String where
Get-Alias|Get-Member
Get-Alias|select name
Get-Alias
Get-Alias |group {$_.name}
get-alias |select -First 5
$null
Get-Alias|Get-TypeData
Get-Alias|Get-Member
Get-Alias| % {$_.name}
Get-Alias| select -First 1
Get-Alias| select -First 1|get-member
Get-Alias| select -First 1|select type
Get-Alias| select -First 1|select module
Get-Alias| select -First 1|select modulename
Get-Alias| select -First 1|select options
Get-Alias| select -First 1|select source
Get-Alias| select -First 1|ConvertTo-Json
Get-Alias| select -First 1|get-member
Get-Alias| select -First 1|select name,resolvedcommand
Get-Alias| select -First 1|% {$_.getType()}
@(1,2,3)[0]
@(1,2,3).getType()
@(1,2,3).size
@(1,2,3).count
@(1,2,3).size()
@(1,2,3)|get-member
[System.Collections.ArrayList]@(1,2,3)
[System.Collections.ArrayList]@(1,2,3).getType()
[System.Collections.ArrayList]@(1,2,3)|% {$_.getType()}
([System.Collections.ArrayList]@(1,2,3)).getType()
([System.Collections.ArrayList]@(1,2,3)).getType().name
([System.Collections.ArrayList]@(1,2,3)).Add(4)
${}.getType()
@{}.getType()
@{}|% {$_.add("a","a"),$_}
@{}|% {$_.add("a","a"),$_}|% {$_.getType()}
@{a="a"}
@{a="a"}.add("a","b")
@{a="a"}["a"]="b"
[System.Collections.Generic.HashSet]@(1,2,3)
[System.Collections.Generic.HashSet[String]]@(1,2,3)
([System.Collections.Generic.HashSet[String]]@(1,2,3)).getType()
"aaa bbb" -split  '\s+'
$sp="aaa bbb" -split  '\s+';$sp[0]
$sp
Remove-Variable $sp
Remove-Variable sp
$sp
$sp="aaa bbb" -split  'a b'
$sp="aaa bbb" -split  '\s'
"aaa bbb" -split  '\s'
"aaa bbb" -split  'a b'
cat ./cipher
cat ./cipher |% {$sp=$_ -split '\s+';$sp[0] $sp[1]}
cat ./cipher |% {$sp=$_ -split '\s+';$sp[0]+" "+$sp[1]}
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap.add($sp[0],$sp[1]}
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap.add($sp[0],$sp[1])} {$mymap}
$mymap
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap.[$sp[0]]=$sp[1]} {$mymap}
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]=$sp[1]} {$mymap}
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]++} {$mymap}
cat ./cipher |% {$mymap=@{}} {$sp=$_ -split '\s+';$mymap[$sp[0]]--} {$mymap}
"aaa bbb" -replace '(\w+) (\w+)' '$2 $1'
"aaa bbb" -replace '(\w+) (\w+)' '\2 \1'
"aaa bbb" -replace '(\w+) (\w+)' '`$2 `$1'
"aaa bbb" -replace '(\w+) (\w+)','$2 $1'
"aaa bbb" -replace '(\w+) (\w+)','\2 \1'
"aaa bbb" -match '(\w)\1\1 (\w)\2\2'
"aaa bbb" -match '(?<=a+)\w+'
"aaa bbb" -match '(?<=a+)\w+$'
"aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','$B $A'
"aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','\B \A'
"aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','$<B> \A'
"aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','$B $A'
"aaa bbb" -replace '(?<A>\w+) (?<B>\w+)','${B} ${A}'
${mymap}
${mymap}.Keys
${mymap}.Keys |% {$mymap[$_]}
openssl s_client -connect 10.134.4.225:443  -ciphersuites TLS_RSA_WITH_AES_128_CBC_SHA -tls1_1
$$
$HOME
$Temp:
$Temp
$Host
$IsLinux
$IsWindows
$IsMacOS
(1,2,3).getType()
(1,$null,3).getType()
(1,null,3).getType()
(1,$null,3).getType()
@(1,$null,3).getType()
1,2,3
1,2,3 | % {$_+5}
1,2,3 | % {$_+5/2}
1,2,3 | % {$_+5%2}
1,2,3 | % {($_+5)%2}
$PID
$PSCmdlet
$PSHOME
Switch("5"){case "2": "2";default "5"}
Switch("5"){"2": "2";default "5"}
Switch("5"){"2" "2";default "5"}
Switch("5"){"2" {"2"} ;default {"5"}}
Switch("2"){"2" {"2"} ;default {"5"}}
ConvertFrom-Csv -Delimiter ',' ./title.csv
cat ./title.csv |ConvertFrom-Csv
cat ./title.csv |ConvertFrom-Csv |%{$_.getType()}
cat ./title.csv |ConvertFrom-Csv |%{$_.toString()}
cat ./title.csv |ConvertFrom-Csv |%{$_.toString}
cat ./title.csv |ConvertFrom-Csv 
cat ./title.csv |ConvertFrom-Csv |Get-Member
cat ./title.csv |ConvertFrom-Csv |select id,data
Import-Csv -Path ./title.csv
Import-Csv -Path ./title.csv|get-member
Import-Csv -Path ./title.csv|get-member -Type Property
Import-Csv -Path ./title.csv|get-member -Type Properties
Import-Csv -Path ./title.csv|get-member -Type Properties |get-member
Import-Csv -Path ./title.csv|get-member -Type Properties |select name,typename
Import-Csv -Path ./title.csv|get-member -Type Properties |select name,membertype
Import-Csv -Path ./title.csv|get-member -Type Properties |select name,definition
cat ./title.csv
Import-Csv -Path ./title.csv|select -First 1
Import-Csv -Path ./title.csv
Import-Csv -Path ./title.csv |select -First 1|Get-Member
cat ./title.csv |select -First 1
cat ./title.csv |select -First 1|ConvertFrom-Csv
Import-Csv -Path ./vul.csv |select -First 1|Get-Member
Import-Csv -Path ./vul.csv 
Import-Csv -Path ./vul.csv |select keys
Import-Csv -Path ./vul.csv |select -First 1 
Import-Csv -Path ./vul.csv |select -First 1  |select type
Import-Csv -Path ./vul.csv |select -First 1  | $_.getType()
Import-Csv -Path ./vul.csv |select -First 1  |% $_.getType()
Import-Csv -Path ./vul.csv |select -First 1  |% {$_.getType()}
Import-Csv -Path ./vul.csv |select -First 1  |get-member
Import-Csv -Path ./vul.csv |(select -First 1 )
Import-Csv -Path ./vul.csv |select -First 1
Import-Csv -Path ./vul.csv |select -First 1|get-member
Import-Csv -Path ./vul.csv |select -First 1|% {$_}
Import-Csv -Path ./vul.csv |select -First 1|% {$_.properties}
Import-Csv -Path ./vul.csv |select -First 1|get-member
Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty
Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name
Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name|%
Stitle-arr=(Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name)
$title-arr=(Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name)
$title_arr=(Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name)
$title_arr
$title_arr[0]
$title_arr[1]
$title_arr[1].getType
$title_arr[1].getType()
$title_arr[1].name
$title_arr=(Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select name -ExpandProperty)
$title_arr=(Import-Csv -Path ./vul.csv |select -First 1|get-member -Type NoteProperty|select  -ExpandProperty name)
$title_arr
$title_arr.getType
$title_arr.getType()
import-Csv -Path ./vul.csv |select -First 1
import-Csv -Path ./vul.csv |select -First 1|% {$_.大类}
import-Csv -Path ./vul.csv |select -First 1|get-member
import-Csv -Path ./vul.csv |select -First 1|% {$_.noteProperty}
import-Csv -Path ./vul.csv |select -First 1|%{$_.key}
import-Csv -Path ./vul.csv |%{$_.key}
import-Csv -Path ./vul.csv |ConvertTo-Json
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |get-member
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json
$my_json=import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json
$my_json.Properties
$my_json.psobject.Properties
$my_json.psobject.Properties |% {@($_.name,$_.value)}
$my_json.psobject.Properties |% {@($_.name)}
$my_json.psobject.Properties |% {@($_.value)}
$my_json.psobject.Properties
$my_json.psobject.Properties.syncroot
$my_json.psobject.Properties.SyncRoot
$my_json.psobject.Properties.isreadonly
$my_json
$my_json |select -First 1
$my_json=($my_json |select -First 1)
$my_json
$my_json.psobject.Properties
$my_json.psobject.Properties |% {@($_.name,$_.value)}
$my_json.psobject.Properties |% {$temp_map=@{}} {$temp_map[$_.name]=$_.value} {$temp_map}
impo
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.properties}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}|%{$_.count}
import-Csv -Path ./title.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}|%{$_.count}
import-Csv -Path ./title.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}
(import-Csv -Path ./title.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}).getType()
(import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}).getType()
(import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$_.psobject.properties}).count
(import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json ).count
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |get-member
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |% {$_.psobject}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |% {$_.psobject.properties}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |% {$tmap=@{}} {foreach ($pro in $_.psobject.properties){$tmap[$pro.name]=$pro.value}}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json|select -First 1 |% {$tmap=@{}} {foreach ($pro in $_.psobject.properties){$tmap[$pro.name]=$pro.value}} {$tmap}
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$tmap=@{}} {foreach ($pro in $_.psobject.properties){$tmap[$pro.name]=$pro.value}} {$tmap}
$tlist=[System.Collections.ArrayList]@()
$tlist.gettype()
import-Csv -Path ./vul.csv |ConvertTo-Json|ConvertFrom-Json |% {$tlist+=$_}
$tlist
$tlist|% {$tmap=@{}} {foreach ($pro in $_.psobject.properties){$tmap[$pro.name]=$pro.value}} {$tmap}
$tlist.count
$tlist |% {$_.getType()}
$tlist|%  {$tmap=@{};foreach ($pro in $_.psobject.properties){$tmap[$pro.name]=$pro.value};$tmap} 
[System.Web.HttpUtility]::UrlEncode("http://中文.com")
New-Object System.Random
$rand=New-Object System.Random
$rand
$rand.NextInt64()
$rand.NextInt64(10)
$rand |% {$_.NextInt64(15)}
$rand |% {$_.NextInt64(15)}|% {$_.getType()}
[System.DateTime]::Now
[System.DateTime]::UnixEpoch
[System.DateTime]::Now.Ticks
[System.DateTimeOffset]::Now.ToUnixTimeMilliseconds()
trans patricia
Get-Command
Get-Command *process
Get-Command *json
Get-Command *file
[System.IO.File]::Exists("/tmp/fda")
Get-Command *dir
help New-Item
trans junction
Get-Command *item
get-alias
(ls).getType()
ls
cat ./ncfg.sql
(cat ./ncfg.sql).getType()
(cat ./ncfg.sql).count
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; @($arr[0],$arr[2])}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; @($arr[1],$arr[2])}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr[1]+" ff "+$arr[2]}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; "a"+$arr[0]+"a"}
vi ./ncfg.sql
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; "a"+$arr[0]+"a"}
cat ./ncfg.sql| % {$_ -split '\s+'}
cat ./ncfg.sql| % {$_ -split '\s+'} | % {$_ -join "-xx"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr -join "-xx-"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr -like "."}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr -match "\w+"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$arr.count}
cat ./ncfg.sql| % {$arr=$_ -split '\s+';$arr.count}
vi ./ncfg.sql
cat ./ncfg.sql| % {$arr=$_ -split '\s+';$arr.count}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$arr.count}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)","$2 $1$3"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","$2 $1$3"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","$1"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)(.*)$","$1"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str} |% {$_.getType()}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "insert","ff"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)","ff"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","$2 $1 $3"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","${2} $1 $3"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","ff"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","\1 \1 \3"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$","${2} ${1} ${0}"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(?<A>\w+)\s+(?<B>\w+)(?<C>.*)$","${A}"}
cat ./ncfg.sql| % {$arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$",'${2} $1 $3'}
pkill -e msedge
top
Get-Variable -Scope local
Get-Variable -Scope global
Remove-Variable arr
Get-Variable ^ |Format-List *
Get-Variable ^ |Format-Table
Get-Variable ^ |Format-Table *
$arr
cat ./ncfg.sql| % {$private:arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$",'${2} $1 $3'}
$arr
cat ./ncfg.sql| % {$private:arr=$_ -split '\s+'; $arr=$arr -match "\w+";$str=$arr -join " ";$str -replace "^(\w+)\s+(\w+)(.*)$",'${2} $1 $3'} -End {Remove-Variable arr}
$arr
help new-event
@(@(1,2,3),@(3,2,1))
vi /tmp/cc
cat /tmp/cc
cat /tmp/cc|% {$arr=$_ -split '\s+';$arr}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..$arr.Count){map[$i]=$arr[$i];}$map}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..$arr.Count){$map[$i]=$arr[$i];}$map}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i]=$arr[$i];}$map}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i]=$arr[$i];}$map}|Sort-Object -Property @{Expression=0;Descending=$true}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i]=$arr[$i];}$map}|Sort-Object -Property @{Expression="0";Descending=$true}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i]=$arr[$i];}$map}|Sort-Object -Property @{Expression="0";Descending=$false}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i.ToString()]=$arr[$i];}$map}|Sort-Object -Property @{Expression="0";Descending=$false}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i.ToString()]=$arr[$i];}$map}|Sort-Object -Property @{Expression="0";Descending=$trye}
cat /tmp/cc|% {$arr=$_ -split '\s+';$map=@{};foreach ($i in 0..($arr.Count-1)){$map[$i.ToString()]=$arr[$i];}$map}|Sort-Object -Property @{Expression="0";Descending=$true}
cat /tmp/cc|Sort-Object -Property @{Expression={($_ -split '\s+')[0]};Descending=$true}
& "cat /tmp/cc"
Invoke-Expression "cat /tmp/cc"
$pss=Read-Host "enter pass" -AsSecureString
$pss
$pss.ToString(`
)
$pss |Format-List *
$pss=Read-Host "enter pass"
$pss
1..10
(1..10)
(1..<10)
cat /tmp/cc|% {$one=($_ -split '\s+' |select -First 1);$one}
cat /tmp/cc|% {$one=$($_ -split '\s+' |select -First 1);$one}
$pss=(cat;cat)
$pss=$(cat;cat)
$pss=(date;date)
$pss=$(date;date)
$pss
Get-Variable -Name pss
Get-Variable -Name PSS
Get-Variable -Name PSS|select value
Get-Variable -Name PSS|select value|Get-Member
Get-Variable -Name PSS|select value|Out-String
Get-Variable -Name PSS -ValueOnly
top
"""a'fdadcad"""
"a"
Get-Process
Get-Process|group -Property ProcessName
Get-Process|group -Property ProcessName|select -First 1
Get-Process|group -Property ProcessName|select -First 1| % {$_.getType()}
Get-Process|group -Property ProcessName|select -First 1| get-member
Get-Process|group -Property ProcessName|select -First 1| select name,count
Get-Process|group -Property ProcessName|select -First 1| select name,count|% {$_.getType()}
Get-Process|group -Property ProcessName|select -First 1| select name,count|% {$_.name}
Get-Process|group -Property ProcessName| select name,count
Get-Process|group -Property ProcessName| % {@{ $_.Name=$_.Count}}
Get-Process|group -Property ProcessName| % {@{ name=$_.Name,count=$_.Count}}
Get-Process|group -Property ProcessName| % {@{ name=$_.Name;count=$_.Count}}
Get-Process|group -Property ProcessName| % {@{ name=$_.Name;count=$_.Count}}|Sort-Object -Property name -Descending
Get-Process|group -Property ProcessName| % {@{ name=$_.Name;count=$_.Count}}|Sort-Object -Property count -Descending
Get-Process|group -Property ProcessName| % {@{ name=$_.Name;count=$_.Count}}|Sort-Object -Property count -Descending|select count
Get-Process|group -Property ProcessName| % {@{ name=$_.Name;count=$_.Count}}|Sort-Object -Property count -Descending|select count|uniq
"aaa" -match '\w+'
([System.Collections.ArrayList]@(1,2,3)) -join "aaa"
Get-Alias where
get-alias ?
get-alias ^
r
h
history
alias ft
alias fl
"aaa" -match '\W+'
"   " -match '\W+'
"   " -match '\v\W+'
"   " -match '\b\W+'
"   " -match '\b\w+'
"aaa" -match '\b\w+'
"aaa " -match '\w+\b\W+'
if (-not '') {"a"} else {"B"}
if (-not 'x') {"a"} else {"B"}
"a" -match '[A]'
"a" -imatch '[A]'
"a" -icmatch '[A]'
"a" -cmatch '[A]'
"a" -contains "a"
"dfaa" -contains "a"
"dfaa" -contains "aa"
"dfaa","aa" -contains "aa"
("dfaa","aa").getType()
(1,2) -eq 3
2 -eq 3
1 -eq "1"
"1".getType()
1.getType()
class MyTest {`
[String] $name;[Int] $age}
[MyTest]@{name="fff";age=10}
[MyTest]@{name="fff",age=10}
 1 -ge 2,3
"a" -like 'a*'
"a" -like '%*'
"a" -like '%'
"a" -like '?*'
echo 'a..f'
echo "a..f"
echo "{a..f}"
a..f
New-TimeSpan
(New-TimeSpan).Seconds
help New-TimeSpan
trans Simultaneously
trans junction
trans fidelity
trans peculiarity
trans enormous
trans immensely
Get-Process | select -First 1 |Get-Member
Get-Process | select -First 1 |Get-Member -Type Propert
Get-Process | select -First 1 |Get-Member -Type Property
trans evident
trans philosophy
trans intact
,2
(,2).getType()
trans unreval
trans unravel
$tt=[System.Collections.ArrayList]@(1,2);$tt.add(3)
$tt
,$tt
($tt).getType()
(,$tt).getType()
(,$tt).count
(,@(1,2)).getType()
(,@(1,2)).count
(,@(1,2))[0]
(@(1,2))[0]
1 -ge 0 -and 2 -ge 0
(1 -ge 0) -and (2 -ge 0)
((1 -ge 0) -and (2 -ge 0))
if(1 -le 0) {"ff"} else if(2 -ge 0) {"GG"} else {"xx"}
if(1 -le 0) {"ff"} elseif(2 -ge 0) {"GG"} else {"xx"}
Get-Help about_flow_control
Get-Help About_Flow_Control
switch(2){2 2;3 3}
switch(2){2 {2};3 {3}}
switch(2){2 {2;break};3 {3}}
switch(2){2 {2};{$_ -ge 0} {3}}
switch(2){2 {2;break};{$_ -ge 0} {3}}
for($i=0;$i -lt 5 ;$i++){$i}
$i
Remove-Variable i
for($local:i=0;$i -lt 5 ;$i++){$i}
$i
(-not 1 -ge 0)
( 1 -ge 0)
(-not ( 1 -ge 0))
1..10 -split 3
1 -band 2
1 -and 2
Pause
help pause
@"aa
@'ff
@"`
aa`
bb`
cc`
"@
"aaa`rbbb"
"aaa`nbbb"
"aaa1r`nbbb"
"aaa`r`nbbb"
"aaa`r`nbbb" -match '\w+\r\w+'
"aaa`r`nbbb" -match '\w+\r\n\w+'
trans stem
trans ubiquitous
"xx"*50
"`$a"
'$a'
"{0,b16}" -f 3
"{0,:b16}" -f 3
"{0,:b}" -f 3
"{b}" -f 3
"{0:b}" -f 3
"{0:b16}" -f 3
"{0,4:b16}" -f 3
"{0,4:g}" -f 3
"{0:g}" -f 3
"{0,-4:g} {1,-5:b}" -f 3,4
"{0,-4:g} {1,5:b}" -f 3,4
"{0,-4:p} {1,5:b}" -f 3,4
"{0,-4:p2} {1,5:b}" -f 3,4
"{0,-4:p0} {1,5:b}" -f 3,4
"{0,-4:p0} {1,5:b}" -f 0.3,4
"{0,-4:p0} {1,5:b4}" -f 0.3,4
"{0,-4:x} {1,5:b4}" -f 33,4
"{0:x}" 456
"{0:x}" -f 456
"{0:x4}" -f 456
"{0:g4}" -f 456
"{0:g4}" -f 45600
"{0:g}" -f 45600
"{0:d}" -f 45600
"{0:d8}" -f 45600
"{0:D8}" -f 45600
trans exponential
"{0:yyyy-MM-dd HH:mm:ss}" -f [System.DateTime]::now
"{0,16:yyyy-MM-dd HH:mm:ss}" -f [System.DateTime]::now
"{0,30:yyyy-MM-dd HH:mm:ss}" -f [System.DateTime]::now
[DateTime]"2023-12-34 11:00:39"
[DateTime]"2023-12-34"
[DateTime]"2023-12-24 11:00:39"
[DateTime]"2023-12-24 11:00:39" - [DateTime]"2023-11-11 00:00:00"
"{0:g}" -f $([DateTime]"2023-12-24 11:00:39" - [DateTime]"2023-11-11 00:00:00")
"{0:g}" -f $([DateTime]"2023-12-24 11:00:39" - [DateTime]"2022-11-11 00:00:00")
([DateTime]"2023-12-24 11:00:39").GetDateTimeFormats()
(([DateTime]"2023-12-24 11:00:39").GetDateTimeFormats).definition
[DateTime].GetMethod("getdatetimeformats")
[DateTime].GetMethod("GetDateTimeFormats")
[DateTime]"2023-12-24 11:00:39"|Get-Member -Name getDateTimeFormats
([DateTime]"2023-12-24 11:00:39"|Get-Member -Name getDateTimeFormats).Definition
switch(2){2 {2;break};{$_ -ge 0} {3}}
switch(2){2 {2};{$_ -ge 0} {3}}
1,2,2,3 -eq 2
1,2,2,3 -ge 2
2 -ge 1,2,3
"{0} - {1} -- {2}" -f 1,2,3
"{0:b} - {1:x} -- {2:p}" -f 3,3,3
"{0:b} - {1:x} -- {2:p}" -f 3,16,3
"{0:b} - {1:x} -- {2:p}" -f 3,17,3
"{0:b} - {1:x} -- {2:p}" -f 3,77,3
"{0:b4} - {1:x} -- {2:p}" -f 3,77,3
"{0:b4} - {1:x4} -- {2:p}" -f 3,77,3
"{0:b4} - {1:x4} -- {2:p2}" -f 3,77,3
"{0:b4} - {1:x4} -- {2:p2} - {3:p0}" -f 3,77,3,0.1
$args
$args[0]
$args[1]
"hellow world" -replace '\b(\w)' '\U\1\E'
"hellow world" -replace '\b(\w)' '\U$1\E'
"hellow world" -replace '\b(\w)','\U$1\E'
"hellow world" -replace '\b(\w)','\U$1'
"hellow world" -replace '\b(\w)','`U$1'
([Regex]::Replace).definition
[Regex]
([Regex]|Get-member -name replace -Type Method)
([Regex]|Get-Member -name replace)
-split "hello world"
-split "hello world `n d"
Get-Help about_split
update-help
Get-Help about_split
trans delimeter
trans delimiter
ls
cat ./vul.csv
ls
cat nginx
cat nginx |select -First 1
Get-Content ./nginx
Get-Content ./nginx |select -First 1
help Get-Content
Get-Content ./nginx -Delimiter /
@(1,2,3) -join 'ff'
@(1,"a",3) -join 'ff'
"aa" -eq "AA"
"aa" -ieq "AA"
"aa" -ceq "AA"
" aaa".Trim()
" aaa".TrimEnd()
Get-Date "1012-11-22 12:00:00"
Get-Date "1012-11-22 12:00:00" -Format 'yyyy-MM-dd HH:mm:ss'
if (2 -eq 3) {"f"}
if (2 -eq 2) {"f"}
trans due
trans staple
trans synopsis
Measure-Command {Start-Sleep -Seconds 5}
ls
ls | % -Begin {$arr=[System.Collections.ArrayList]@()} -Process {$arr.Add($_)} -End {$arr -join ","}
ls | % -Begin {$arr=[System.Collections.ArrayList]@()} -Process {[void]$arr.Add($_)} -End {$arr -join ","}
trans modulus
[int]3/2
([int]3/2)
[int](3/2)
[Math]::Truncate(13/7)
trans statistical
ls |Measure-Object -Property {$_.Length} -Average
ls |Measure-Object -Property {$_.Length} -Average -Sum
ls |Measure-Object -Property {$_.Length} -Average -Sum -Maximum -Minimum
ls |Measure-Object -Property {$_.Length} -Average -Sum -Maximum -Minimum -StandardDeviation
ls |Measure-Object -Line
[Convert]::ToBase64String("abc")
[Convert]::ToBase64String("abc".ToByte())
trans administrative
1kb
1k
1mb
1gb
1tb
"{0:x4}" -f 2345
"{0:x4}" -f 787
"{0:x4}" -f 16
"{0:x4}" -f 77
"{0:X4}" -f 77
@(ls)
@(ls).getType()
(ls).getType()
(ls).count
@(ls).count
(ls) -join ","
trans jag
@{a="a";b="B"}
@{a="a";b="B"}|Sort-Object name
@{a="a";b="B"}|Sort-Object -Property name
@{a="a";b="B"}|Sort-Object -Property name -Descending
@{a="a";b="B"}|Sort-Object Name
$($mm=@{};$mm["a"]="a";$mm["b"]="b";$mm)
$($mm=@{};$mm["a"]="a";$mm["b"]="b";$mm) |Sort-Object name
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm) |Sort-Object name
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm) |Sort-Object value
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm) |Sort-Object {$_.name}
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm) |% {$_.name}
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm).Keys
$($mm=@{};$mm["a"]=1;$mm["b"]=2;$mm).GetEnumerator() |Sort-Object name
Get-date
$(Get-date).DayOfYear
$(Get-date).DayOfWeek
$(Get-date).Millisecond
[System.DateTimeOffset](Get-Date)
([System.DateTimeOffset](Get-Date)).ToUnixTimeMilliseconds()`

([System.DateTimeOffset]$(Get-Date)).ToUnixTimeMilliseconds()
(3..100)|Get-Random
[System.Random]::new()
[System.Random]::new().NextInt64(200)
ls
${nginx}
${./nginx}
Get-Content ./nginx|select -First 1
Get-Content ./nginx|select -First 1 |Get-Member
Get-Content ./nginx|% {$_.readcount}
cd ../Downloads/
xxd './dp_lic (1).info'
file './dp_lic (1).info'
size './dp_lic (1).info'
ls -lrth './dp_lic (1).info'
get-content ./nginx -raw
cd
cd ./Desktop/
get-content ./nginx -raw
get-content ./nginx -raw |select -First 1
(get-content ./nginx -raw |select -First 1) -match '^location.*\}'
(get-content ./nginx -raw |select -First 1) -match '(?s)^location.*\}'
(get-content ./nginx -raw |select -First 1) -match '(?s)^Location.*\}'
(get-content ./nginx -raw |select -First 1) -cmatch '(?s)^Location.*\}' 
(get-content ./nginx -raw |select -First 1) -cmatch '(?s)^(?i)Location.*\}' 
(get-content ./nginx -raw |select -First 1) -cmatch '(?s)^(?:(?i)L)ocation.*\}' 
(get-content ./nginx -raw |select -First 1) -cmatch '(?s)^(?:(?i)L)Ocation.*\}' 
Join-Path tmp
Join-Path tmp aaa
Join-Path tmp aaa fff
$env:HOME
Get-Alias ?
Join-Path tmp,root a
Join-Path /tmp,/root a
vi /tmp/cc
Get-Content /tmp/cc
Get-Content /tmp/cc -Delimiter a
(Get-Content /tmp/cc -Delimiter a).count
Get-Content /tmp/cc -Delimiter '(?=a)'
Get-Content /tmp/cc -TotalCount
Select-String
Select-String 'abc' /tmp/cc
Select-String 'abc' /tmp/cc -AllMatches
trans emphasis
Select-String 'abc' /tmp/cc -AllMatches -NoEmphasis
Select-String 'abc' /tmp/cc -AllMatches |select -ExpandProperty matches
Select-String -Pattern '(abc)' /tmp/cc -AllMatches |select -ExpandProperty matches
Select-String -Pattern '(abc)' /tmp/cc -AllMatches |select -ExpandProperty matches |% {$_.Groups[0]}
Select-String -Pattern '(abc)' /tmp/cc -AllMatches |select -ExpandProperty matches |% {$_.Groups[1]}
Select-String -Pattern '(abc)\1\1' /tmp/cc -AllMatches |select -ExpandProperty matches |% {$_.Groups[1]}
Select-String -Pattern '(abc)\1\1' /tmp/cc -AllMatches |select -ExpandProperty matches |% {$_.Groups[0]}
Select-String -Pattern '(abc)\1\1' /tmp/cc -AllMatches |select -ExpandProperty matches |% {$_.Groups[0].value}
trans intricacy
Select-String -Pattern '(abc)\1\1' /tmp/cc -AllMatches |select  matches
trans tailor
"大小" |Out-File /tmp/gg
cat /tmp/gg
trans moderate
trans burden
cd
cd ./Downloads/
Get-Content ./dp_lic.info |Format-Hex
Get-Content ./dp_lic.info -AsByteStream|Format-Hex
xxd ./dp_lic.info
sshp 87
help Write-Progress 
for ($i=0;$i -le 100;$i++) {Write-Progress -Activity "test" -Status "$i%" -PercentComplete $i;Start-Sleep -Seconds 1}
$Host
$Host.UI
$Host.UI.RawUI
$Host.UI.RawUI.WindowTitle = Get-Location
$Host.UI.WriteLine("xfg")
Get-Clipboard
Set-Clipboard "xxx"
$error
$error[0]
trans diverse
$error[0]|Format-List
$error[0]|Format-List -Force
$Error.getType()
trans inquire
trans synonyms
&(cat)
&(ls)
&("ls")
Get-Item
Get-Item .
Get-Item *
Get-Item *./*
Get-Item ./*
Get-Item ./* -stream
cd
cd ./Desktop/
Get-Content ./nginx
Get-Item * -stream *
New-PSSession -UserName root -hostname 10.134.4.87
New-PSSession -UserName root -hostname 10.134.4.87 -port 22
error[0]|Format-Table -Force
$error[0]|Format-Table -Force
$error[0]|Format-Table -Force -Expand
$error[0]|Format-Table -Force -Expand *
$error[0]|Format-List -force
$error[1]|Format-List -force
$error[2]|Format-List -force
$error[3]|Format-List -force
trans drawback
trans demultiplexer
trans promptly
trans simultaneously
trans indication
trans participant
Set-Clipboard ""
exit
exit
ConvertFrom-Json ./table.json
Get-Content ./table.json|ConvertFrom-Json
Get-Content ./table.json|ConvertFrom-Json |get-member *
Get-Content ./table.json
Get-Content ./table.json|ConvertFrom-Json |% {$_}
Get-Content ./table.json|ConvertFrom-Json |% {$_.keys}
Get-Content ./table.json|ConvertFrom-Json |% {$_.psobject}
Get-Content ./table.json|ConvertFrom-Json |% {$_.psobject.properties}
Get-Content ./table.json|ConvertFrom-Json |% {@{$_.psobject.properties.name=$_.psobject.properties.value}}
Get-Content ./table.json|ConvertFrom-Json |% {@{key=$_.psobject.properties.name,value=$_.psobject.properties.value}}
Get-Content ./table.json|ConvertFrom-Json |% {@{key=$_.psobject.properties.name;value=$_.psobject.properties.value}}
Get-Content ./table.json|ConvertFrom-Json |% {@{key=$_.psobject.properties.name;value=$_.psobject.properties.value}}|ConvertTo-Json
Get-Content ./table.json|ConvertFrom-Json |% {@{key=$_.psobject.properties.name;value=$_.psobject.properties.value}}|ConvertTo-Json -Compress
help Rename-Item 
Get-ChildItem
exit
Get-ChildItem -path *xlsx
Get-ChildItem -path *xlsx -path docx
Get-ChildItem -path *xlsx,*docx
free -h
exit
"aabb" |% {$_.ToCharArray()}
"aabb" |% {$arr=$_.ToCharArray();$arr[0]}
"aabb" |% {$arr=$_[0]}
"aabb" |% {$arr=$_.ToCharArray();$result=[System.Collections.ArrayList]@();foreach ($c in $arr) { $result.Add($c)};$result}
"aabb" |% {$arr=$_.ToCharArray();$result=[System.Collections.ArrayList]@();foreach ($c in $arr) { $result.Add($c)};$result[0]}
"aabb" |% {$arr=$_.ToCharArray();$result=[System.Collections.ArrayList]@();foreach ($c in $arr) { [Void]$result.Add($c)};$result[0]}
free -h
trans vertex
trans nevertheless
cd
cd ./Downloads/
find -name '*pdf'
cd
cd ./IdeaProjects/
cargo new rust-compile
top
free -h
top
pkill -e msedge
top
free -h
trans grok
trans moderator
trans awkward
@{a="a";b="B"}|Sort-Object Name
@{a="a";b="B";1="1"}|Sort-Object Name
@{d="d";a="a";b="B";1="1"}|Sort-Object Name
[System.Collections.ArrayList]@(123)
[System.Collections.ArrayList]@(123,345)
@(1,"a",3).getType()
[System.Collections.ArrayList]@(1,"a",3).getType()
([System.Collections.ArrayList]@(1,"a",3)).getType()
[System.Collections.Hashtable]@{"a"=1}
([System.Collections.Hashtable]@{"a"=1}).getType()
Get-FileHash -h
get-help Get-FileHash 
get-help Get-FileHash -examples
ls
Get-FileHash cc md5
Get-FileHash cc md5|select hash
Get-FileHash cc md5|select hash|Get-Member
Get-FileHash cc md5|select hash|% {$_.psobj}
Get-FileHash cc md5|select hash|% {$_.psobject}
Get-FileHash cc md5|select hash|% {$_.psobject}|get-member
Get-FileHash cc md5|select hash|% {$_.psobject.properties}|get-member
Get-FileHash cc md5|select hash|% {$_.psobject.properties.name}
Get-FileHash cc md5|select hash|% {$_.psobject.properties.value}
Get-Alias ?
Get-Alias select
h
r
Get-Content cc
Get-Content cc|% -Begin {$list=[System.Collections.ArrayList]@()} -Process {if($list.Count -eq 2) {$str=$list -join "-";$list.Clear();$list.Add($_);$str}else{[Void]$list.Add($_)}}
Get-Content cc|% -Begin {$list=[System.Collections.ArrayList]@()} -Process {if($list.Count -eq 2) {$str=$list -join "-";$list.Clear();[void]$list.Add($_);$str}else{[Void]$list.Add($_)}}
Get-Content cc|% -Begin {$list=[System.Collections.ArrayList]@()} -Process {[void] $list.add($_);if($list.Count -eq 2) {$str=$list -join "-";$list.Clear();$str}}
cat ./stack
cat ./stack |% -Begin {$list=[System.Collections.ArrayList]@()} -Process {[void]$list.Add($_);if($_ -match '^$'){$str=$list -join "";$list.Clear();$str}}
@(aaaaa)
@("aaaaa")
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)'
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -SimpleMatch
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -Raw
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -List
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches.value
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches[0].value
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches| % {$_.getType()}
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches| Get-Member
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |get-member
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches| % {$_.psObject.properties}
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' |select matches| % {$_.psObject.properties.value}
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' | % {$_.psObject.properties.value}
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' | % {$_.psObject.properties}
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches
@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches|select matches.value
(@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches).Matches.value
(@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches).Matches
(@("aaaaa")|Select-String -Pattern '(?<=a+)a(?=$)' -AllMatches).Matches.getType()
(@("aaaaa")|Select-String -Pattern '(?<=a+)a' -AllMatches).Matches.getType()
(@("aaaaa")|Select-String -Pattern '(?<=a+)a' -AllMatches).Matches.value
(@("aaaaa")|Select-String -Pattern '(?<=a+)a').Matches.value
top
ps -ef|grep thund
kill -9 183435
kill -9 183643
kill -9 183435
ps -ef|grep bird
cat /tmp/dfs
cat /tmp/dfs|Invoke-Expression
Invoke-Expression 'cat /tmp/dfs'
@'aaa'@
@"`
aaa`
bbb`
ccdfad`
"@
exit
Get-ChildItem
Get-ChildItem | ? {$_.Directory}
Get-ChildItem | select -First 1|get-member
Get-ChildItem | % {$_.GetType()}
Get-ChildItem | % {$_.Name}
Get-ChildItem -Directory
Test-Path -Path ./RustDesk/ -Type Leaf
Test-Path -Path ./RustDesk/ -Type Container
Get-ChildItem | select -First 1
Get-ChildItem | select -First 1| %{$_.getPath()}
Get-ChildItem | select -First 1| Get-Member
[File]"/tmp/cc"
New-Object File "/tmp/cc"
[System.IO.File]"/tmp/cc"
ne
New-Object -TypeName System.IO.File -ArgumentList "/tmp/cc"
[System.IO.File].GetMethods()
[System.IO.File].GetMethods() |select name
[System.IO.File].GetMethods() |select name,isstatic
[System.IO.File].GetMethods() |select -First 1 |Get-Member
[System.IO.File].GetMethods() |select -First 1 |Get-Member|select nam3
[System.IO.File].GetMethods() |select -First 1 |Get-Member|select name
[System.IO.File].GetMethods() |select -First 1 |Get-Member|select name|Select-String -Pattern ".*return.*"
[System.IO.File].GetMethods() |select -First 1 |Get-Member|select name|Select-String -Pattern ".*param.*"
[System.IO.File].GetMethods() |select name,isstatic,parameters
[System.IO.File].GetMethods() |% {$_.GetParameters()}
[System.IO.File].GetMethods() |select -First 1|% {$_.GetParameters()}
trans washington
dig omono.org
ping 45.78.60.34
[DateTime]"2023-12-24 11:00:39"|Get-Member -Name getDateTimeFormats
[DateTime]"2023-12-24 11:00:39"|Get-Member 
"{0:X4}" -f 77
"{0:x4}" -f 77
help Set-Content
([DateTime]"2023-12-24 11:00:39" ).AddDays(-180)
"{d}" -f ([DateTime]"2023-12-24 11:00:39" ).AddDays(-180)
"{0:d}" -f ([DateTime]"2023-12-24 11:00:39" ).AddDays(-180)
"{0:yyyy-MM-dd}" -f ([DateTime]"2023-12-24 11:00:39" ).AddDays(-180)
"{0:yyyy-MM-dd HH:mm:ss}" -f ([DateTime]"2023-12-24 11:00:39" ).AddDays(-180)
([DateTime]"2023-12-24 11:00:39" ).ToUniversalTime()
([System.DateTimeOffset]([DateTime]"2023-12-24 11:00:39" )).ToUnixTimeMilliseconds()
[DateTime].GetMethods()
[DateTime].GetMethods() |Select-String -Pattern ".*offset.*"
[DateTime].GetMethods() |select name|Select-String -Pattern ".*offset.*"
[DateTime].GetMethods() |select name
[DateTime].GetMethods() |Select-String -Pattern ".*parse.*"
(New-TimeSpan -Start @(Get-Date) -End '2025-11-11 11:11:11')
(New-TimeSpan -Start @(Get-Date) -End [DateTime]'2025-11-11 11:11:11')
(New-TimeSpan -Start [DateTime]@(Get-Date) -End [DateTime]'2025-11-11 11:11:11')
@(Get-Date)
help Get-Date
(New-TimeSpan -Start [DateTime]$(Get-Date) -End [DateTime]'2025-11-11 11:11:11')
(New-TimeSpan -Start $(Get-Date) -End [DateTime]'2025-11-11 11:11:11')
(New-TimeSpan -Start $(Get-Date) -End '2025-11-11 11:11:11')
h
(New-TimeSpan -Start $(Get-Date) -End '2025-11-11 11:11:11')
(New-TimeSpan -Start $(Get-Date) -End [DateTime]'2025-11-11 11:11:11')
(New-TimeSpan -Start $(Get-Date) -End [DateTime]'2025-11-11 11:11:11').TotalDays
(New-TimeSpan -Start $(Get-Date) -End [DateTime]'2025-11-11 11:11:11').TotalDays()
(New-TimeSpan -Start $(Get-Date) -End '2025-11-11 11:11:11').TotalDays()
(New-TimeSpan -Start $(Get-Date) -End '2025-11-11 11:11:11').Days
