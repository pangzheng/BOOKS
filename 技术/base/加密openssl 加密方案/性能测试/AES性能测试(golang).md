# 大文件加密测试
### 一、Openssl AES 测试
### 命令参考

openssl enc -aes-128-cbc -in plain.txt -out encrypt.txt  -K 1223 -iv f123 -p

#### 100M 执行结果
dd if=/dev/urandom of=file bs=1m count=100

```
------start openssl-aes-128 encryption-------
------ECB Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m0.197s
user	0m0.040s
sys	0m0.129s
------CBC Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.244s
user	0m0.098s
sys	0m0.122s
------CTR Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.203s
user	0m0.041s
sys	0m0.129s
------GCM Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157

real	0m0.303s
user	0m0.072s
sys	0m0.132s



#############################################################
------start openssl-aes-128 Decrypt-------
------ECB Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m0.309s
user	0m0.045s
sys	0m0.141s
------CBC Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.260s
user	0m0.044s
sys	0m0.140s
------CTR Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.322s
user	0m0.042s
sys	0m0.140s
------GCM Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157
bad decrypt

real	0m0.429s
user	0m0.074s
sys	0m0.141s
```



### 300M 执行结果

dd if=/dev/urandom of=file300 bs=1m count=300
 
```
------start openssl-aes-128 encryption-------
------ECB Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m0.970s
user	0m0.120s
sys	0m0.467s
------CBC Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.421s
user	0m0.314s
sys	0m0.458s
------CTR Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.587s
user	0m0.116s
sys	0m0.388s
------GCM Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157

real	0m1.123s
user	0m0.196s
sys	0m0.782s



#############################################################
------start openssl-aes-128 Decrypt-------
------ECB Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m1.014s
user	0m0.139s
sys	0m0.520s
------CBC Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m0.946s
user	0m0.141s
sys	0m0.481s
------CTR Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.081s
user	0m0.129s
sys	0m0.485s
------GCM Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157
bad decrypt

real	0m1.283s
user	0m0.222s
sys	0m0.850s
```
 
### 500M 执行结果

dd if=/dev/urandom of=file500 bs=1m count=500

```
------start openssl-aes-128 encryption-------
------ECB Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m1.418s
user	0m0.201s
sys	0m0.766s
------CBC Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.278s
user	0m0.497s
sys	0m0.613s
------CTR Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.145s
user	0m0.193s
sys	0m0.620s
------GCM Encryption-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157

real	0m1.710s
user	0m0.328s
sys	0m1.006s


#############################################################
------start openssl-aes-128 Decrypt-------
------ECB Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50

real	0m1.275s
user	0m0.216s
sys	0m0.728s
------CBC Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.630s
user	0m0.223s
sys	0m0.797s
------CTR Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238616971576A466653

real	0m1.591s
user	0m0.207s
sys	0m0.754s
------GCM Decrypt-------
salt=CD5E7C6DFF7F0000
key=4142434445464748494A4B4C4D4E4F50
iv =463252746761523861697157
bad decrypt

real	0m1.842s
user	0m0.337s
sys	0m1.054s

```
**加密**

| 模式/文件大小    | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | 
| ECB  | 0.197s |   0.970s | 1.418s | 128bit  |
| CBC  | 0.244s |   1.421s | 1.278s | 128bit  |
| CTR  | 0.203s |   0.587s | 1.145s | 128bit  |
| GCM  | 0.303s |   1.123s | 1.710s | 128bit  |

**解密**

|  模式/文件大小   | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | ----|
| ECB  | 0.309s |  1.014s   | 1.275s |128bit  |
| CBC  | 0.260s |   0.946s  | 1.630s |128bit  |
| CTR  | 0.322s |   1.081s  | 1.591s |128bit  |
| GCM  | 0.429s |   1.283s | 1.842s | 128bit  |

### Golang AES 测试

#### 100M 执行结果

```
=== RUN   TestAesEn
ECB加密用时: 258.404634ms
CBC加密用时: 225.06961ms
CTR加密用时: 227.276087ms
GCM加密用时: 109.812014ms
CCM加密用时: 483.719632ms
--- PASS: TestAesEn (1.35s)

=== RUN   TestAesDe
ECB解密用时: 327.92431ms
CBC解密用时: 241.212073ms
CTR解密用时: 180.918607ms
GCM解密用时: 83.013743ms
CCM解密用时: 433.267293ms
--- PASS: TestAesDe (1.37s)
PASS


```

### 300M 执行结果

```
=== RUN   TestAesEn
ECB加密用时: 789.188054ms
CBC加密用时: 1.104479706s
CTR加密用时: 763.308434ms
GCM加密用时: 381.056635ms
CCM加密用时: 1.754520015s
--- PASS: TestAesEn (4.98s)


=== RUN   TestAesDe
ECB解密用时: 750.230355ms
CBC解密用时: 613.781025ms
CTR解密用时: 883.814733ms
GCM解密用时: 259.265905ms
CCM解密用时: 1.374585459s
--- PASS: TestAesDe (4.43s)
PASS

```

### 500M 执行结果

```
=== RUN   TestAesEn
ECB加密用时: 2.752199793s
CBC加密用时: 1.900598159s
CTR加密用时: 2.438564752s
GCM加密用时: 786.245166ms
CCM加密用时: 2.670718817s
--- PASS: TestAesEn (10.89s)


=== RUN   TestAesDe
ECB解密用时: 902.67282ms
CBC解密用时: 1.509868317s
CTR解密用时: 1.274119469s
GCM解密用时: 1.166910633s
CCM解密用时: 2.403446345s
--- PASS: TestAesDe (9.17s)
PASS

```


**加密**

|  模式/文件大小   | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | 
| ECB  | 0.258s |   0.789s | 2.752s | 128bit  |
| CBC  | 0.225s |   1.104s | 1.900s | 128bit  |
| CTR  | 0.227s |   0.763s | 2.438s | 128bit  |
| GCM  | 0.109s |   0.381s | 0.786s | 128bit  |
| CCM  | 0.483s |   1.754s | 2.670s | 128bit  |


**解密**

|  模式/文件大小   | 100M  |   300M  |  500M | key数据大小 |
|  ----  | ----  |   ----  | ----- |----  | ----|
| ECB  | 0.327s |   0.750s  | 0.902s |128bit  |
| CBC  | 0.241s |   0.613s  | 1.509s |128bit  |
| CTR  | 0.180s |   0.883s  | 1.274s |128bit  |
| GCM  | 0.83s  |   0.259s  | 1.166s |128bit  |
| CCM  | 0.433s |   1.374s  | 2.403s |128bit  |



- 加密

	![](../pic/openssl-golang-en.png)
- 解密
	
	![](../pic/openssl-golang-de.png)
	