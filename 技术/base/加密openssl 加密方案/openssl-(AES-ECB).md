# AES-ECB 
## x86
- 加密

		openssl enc -e -aes-128-ecb -K 95B8724B0763DF53469EE78B367F4588 -nosalt -in hello.txt -out hello.txt.aes.en.x86
- 解密	

		openssl enc -d -aes-128-ecb -K 95B8724B0763DF53469EE78B367F4588 -nosalt -in hello.txt.aes.en.x86 -out hello.txt.aes.de.x86

## x64
- 加密

		openssl enc -e -aes-128-ecb -K 95B8724B0763DF53469EE78B367F4588 -nosalt -in hello.txt -out hello.txt.aes.en.x64

- 解密	

		openssl enc -d -aes-128-ecb -K 95B8724B0763DF53469EE78B367F4588 -nosalt -in hello.txt.aes.en.x64 -out hello.txt.aes.de.x64

## 原始 hello.txt 为 117 字节，加密后 hello.txt.aes.en 为 128 字节 

	ks@ubuntu:/ks/openssl-aes$ ls -l
	total 24
	-rw-rw-r-- 1 ks ks  117 Sep 25 08:49 hello.txt
	-rw-rw-r-- 1 ks ks  117 Sep 25 09:09 hello.txt.aes.de.x64
	-rw-rw-r-- 1 ks ks  117 Sep 26 10:06 hello.txt.aes.de.x86
	-rw-rw-r-- 1 ks ks  128 Sep 25 09:09 hello.txt.aes.en.x64
	-rw-rw-r-- 1 ks ks  128 Sep 26 10:05 hello.txt.aes.en.x86
	-rw-rw-r-- 1 ks ks 1107 Sep 26  2019 openssl-aes.txt

- 原始文件md5
	
		ks@ubuntu-x64:/ks/openssl-aes$ md5sum hello.txt
		91b23211f799d7f2ba84c321208a910d  hello.txt

- 加密 x86/64
	- x86
	
			ks@ubuntu:/ks/openssl-aes$ md5sum hello.txt.aes.en.x86
			070566cd19dce2b4cb1d93a4eb276c40  hello.txt.aes.en.x86
	- 64
	
			ks@ubuntu:/ks/openssl-aes$ md5sum hello.txt.aes.en.x64
			070566cd19dce2b4cb1d93a4eb276c40  hello.txt.aes.en.x64
- 解密 x86/64
	- x86
	
			ks@ubuntu:/ks/openssl-aes$ md5sum hello.txt.aes.de.x86
			91b23211f799d7f2ba84c321208a910d  hello.txt.aes.de.x86
	- 64
		
			ks@ubuntu:/ks/openssl-aes$ md5sum hello.txt.aes.de.x64
			91b23211f799d7f2ba84c321208a910d  hello.txt.aes.de.x64

