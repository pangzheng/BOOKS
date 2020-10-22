
#### 说明
```
读取耗时：file(blob)类型转Uint8Array  
100M文件多次读取分别耗时：130ms、127ms、119ms  
加密耗时: 调用加密算法返回耗时  
File、Key、Iv、均为Uint8Array类型  
其中salt对应asmcrypto.js的nonce且仅支持 8~12位字符
```
##### 100M（102400KB）
###### Encrypt---------------------------------------------------
```
1、	ECB Encrypt------  
key=4142434445464748494A4B4C4D4E4F50  
padding：true，无salt  
加密耗时：1750ms

Padding: false
加密耗时：1766ms

2、CBC Encrypt------
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238（iv仅支持16bit）
padding: true
加密耗时：1788ms

Padding：false
加密耗时：1791ms

3、CTR Encrypt------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF7F00
加密耗时：1911ms

4、GCM Encrypt------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF7F00
加密耗时：9411ms

5、CCM Encrypt------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF
加密耗时：4231ms
```
###### Decrypt---------------------------------------------------
```
1、ECB Decrypt ------
key=4142434445464748494A4B4C4D4E4F50
padding：true，无salt
解密耗时：1735ms

Padding: false
解密耗时：1699ms

2、CBC Decrypt ------
key=4142434445464748494A4B4C4D4E4F50
iv =4632527467615238（iv仅支持16位）
padding: true
解密耗时：1947ms

Padding：false
解密耗时：1990ms

3、CTR Decrypt ------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF7F00
解加密耗时：1797ms

4、GCM Decrypt ------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF7F00
解密耗时：9194ms

5、CCM Decrypt ------
插件算法无iv
key=4142434445464748494A4B4C4D4E4F50
salt=CD5E7C6DFF
解密耗时：4164ms
```
##### 300M及500M文件， key和Iv、salt均相同

##### 加密统计

模式/文件大小 | 100M | 300M | 500M | key数据大小
---|---|---|---|---
ECB | 1.750s | 5.447s | 8.508s | 128bit
CBC | 1.788s | 5.253s | 10.734s | 128bit
CTR | 1.911s | 5.454s | 9.002s | 128bit
GCM | 9.411s | 30.391s | 53.706s | 128bit
CCM | 4.231s | 10.266s | 21.052s | 128bit

##### 解密统计

模式/文件大小 | 100M | 300M | 500M | key数据大小
---|---|---|---|---
ECB | 1.735s | 5.478s | 9.286s | 128bit
CBC | 1.947s | 5.309s | 10.307s | 128bit
CTR | 1.797s | 5.352s | 11.043s | 128bit
GCM | 9.194s | 32.590s | 59.074s | 128bit
CCM | 4.164s | 10.028s | 20.207s | 128bit

