# 应研 openssl 
## 生成密钥对
- RSA 生成 1024 密钥(788/900字节)
	- 生成
	
			ks@ubuntu-x64:/ks$ openssl genrsa -out rsa_private.key 1024
			Generating RSA private key, 1024 bit long modulus (2 primes)
			........................................+++++
			.+++++
			e is 65537 (0x010001)
	- 查看文件	
		
			ks@ubuntu:/ks/openssl-rsa$ ls -l rsa_private.key
			-rw-rw-r-- 1 ks ks 887 Sep 28 12:02 rsa_private.key
	- 查看内容	
		
			ks@ubuntu-x64:/ks$ cat rsa_private.key
			-----BEGIN RSA PRIVATE KEY-----
			MIICWwIBAAKBgQCiZX+xB9DuezmSPkzRHAGNrkzfDQGNC5jIx9dwZFQQq/aLNZyn
			RjrlVReY4rdPB/yFeQyyyHpiGoac73VyXeoxc0p6gvw1gSjaKb0+j7NXtRGRtfNv
			f3I2nmyOF9foo9CddrOKnAwd+hupCWkBm74bA9LueSSYNx+ujYngJ4YfGQIDAQAB
			AoGAFi9Zf1y6/SlWVxUtt09lIu7Rz5LeSF9yMtOiKdd66IAlgSUEzpE4kIXMYHVw
			Z1qG89DynCXpGifXhe7sk4Ai8wC/I+Hlkl6AJMObxRZBtHsmPYNP2bzdMDrK0q0p
			eU0TEvxnrcPg1OOSY5o8n208iSBWa5YwpnxKUzeIb9WJMpUCQQDOs29emQAivDWy
			IiEyJbysh9xmH5EwkgPjrdEc12LZCRPyHSq8mpA8oSsbSReWmY1pUZ6XADac9ylg
			g7GIxGknAkEAySD2PTGl2srolavZzDA50ZazENM854o+HD+RfWKkqF7TOA0tPvVS
			Blsl7eO0jcPPX3GAea6rhM3dDG/pT0PdvwJAC5c+SIS14DzDmhCV4fUBxn772fIt
			NxcJBsxpbH+beGYs6ye9jMPyEKRNAYOwwe4sYtqD1R3a8iRd+p6H5w50gwJAXLQL
			qzL6DjmeMHZwQdQsojUCvguPM+2dKSoHpjJUUKK8WkaAh1dNgk560gY1T25kY8qN
			/AgNWH4Gl9fxQq62QwJAYGHl39YdB1QXqbvOnhNkzxg9/IG+h6/Qk0hOd66+SN6z
			wMZALBa80FTPxa7jjgWyPWY2/lGOGiOFOYUJJj3+VQ==
			-----END RSA PRIVATE KEY-----
- RSA 生成 2048 密钥(1675 字节)
	- 生成
	
			ks@ubuntu:/ks/openssl-rsa$ openssl genrsa -out rsa_private.key
			Generating RSA private key, 2048 bit long modulus
			.....................................................+++
			.............+++
			e is 65537 (0x10001)
	- 查看文件		
	
			ks@ubuntu:/ks/openssl-rsa$ ls -l rsa_private.key
			-rw-rw-r-- 1 ks ks 1675 Sep 28 12:00 rsa_private.key
	- 查看内容		
	
			ks@ubuntu:/ks/openssl-rsa$ cat rsa_private.key
			-----BEGIN RSA PRIVATE KEY-----
			MIIEogIBAAKCAQEAsMcpFv1tLJnlvpo1fc+ABC3vPIpTLm1Wb0eLZ5bEXZyQnh6U
			azfWEP/Lwe1QiSjl0q3PgRuK34+5m4kiGpIoLw6aCsGus9Skj7LgFlN1BKnTgnhi
			GIbQBjuZAmg18rMa1y4iPaX1EJYv/2rJSwVYw2MSCdp7r/lo/F2M53W9fUl9E78x
			8fhuuDQ01/zGB/2ZX9Poi5xv/f4sF9FZsm8kefNgusUspUeFSMV6taluVED1ZkXF
			Vsyqhwiay01qlJxB9hRQZTLN/BKOHexfYOHj3r5FZbKybmyHH4wNyn4Pe3qy83bN
			HzklXKXeLh4oBsVXKy2QlaHiGY4wXYlSzJBb/QIDAQABAoIBAHfHmsnfEbhk1szd
			RF1o2b77ONz2hQIyR5zDB2w7NfaP/VWGNt6nSL7f87oFCMrHEWI9Lxq2BNUSV5tR
			we/rFJs985ZSQgPLk21isH+wMNvmDlCbLSydPBrdCwIchmFahldDBSRsbK6+dOtF
			+fqBNvG9ER3oEmLcRgypfq3ek3Rt1Ke7YNIt7CckLrtuYRMpJuFmDc74nF7Y4/wl
			U09oNbSahFO4E7RwpLIjhx+9RCoVYNv4VhzH559z9GD5vam+X4Tqx/WYkDBm5qMs
			M9Wym6ogHzduMalRSG+ZclqaLhXssXy4LZzrf6DaOHaz9WGLKtSYjduXVE4X9PSw
			Q+fsqKkCgYEA16YffDriaCE8S86BlNc0d5/iLxIttlt76EEg6DLQ4jY+7CCTcvsv
			hSKORfYaPEnwrNRtUc6kT8YzAPozrcVTabzoDubES7G02hUFQxcms2nckED2TN5T
			dSf/eTS10Gcb9zRLpqxlFJwxLEPbXOpP6BNVkLDBu1oeoNOHcit8DXMCgYEA0dsQ
			8UgVK6zuWklirS/CGSsWPpfwO8y/VZ+XajgIB0W+ksxHMoe4jIWuuuF0ien5g9Zo
			SYiD1LuIdGtznCjoJaDNc4sWrMLn1n9YaOTLlq1Vaie2zGPXJ519O/oZ94YhvkN/
			/1IuPPqOZjQ2M8KUVQZ1yN9G1oCkXKdSLifZlM8CgYAgtfUurevr6aawxQlI33/4
			6Uqw4ywF7kiUSVTJg/PFbH8M6EAVf96MblpwaE1AeeLFBI/0icjIxQK1kj5GjJkD
			gAEfSYfbB4CsV+XtcFSRgGxRgVka/dpg+gk5hHJTw2AtlkQnax0GDE81LLHYnT4z
			sMMY6IbqeVgOAygXfFsb2wKBgH+Idl9rwxHr4l6UQpelcnwBJ1+azrBI2e6bMlWm
			/5oL1Kk54/rdaFZd17ZS1ZQK0UxBvNcTU6qw3ViDBJtecNaNRs51NK6BNIuykHQO
			t3s2k7YTtI+6DUNR/o24jJdUuKT8OIld1CPS0T9jc505HxQc/O+6YT5yp3B3bwYF
			ycEZAoGALgCaGnRzHz2O3Pm/UzBMvmGEDyePRmtAN20DodktoTgBaRZ9kB8/gngh
			zikSXLrrmAk629Z2y31k0Rqw1BU5BFvyg6I1H7RX1YUxpa+4PZOicV19QWSP7MXo
			oM7iPLAaUp+qDcaDplbvFm1tYakqmxuxeSU4VeR7ntxuJMpW2OQ=
			-----END RSA PRIVATE KEY-----
- 通过私钥生成公钥(272/276 字节)
	- 生成
	
			ks@ubuntu-x64:/ks$ openssl rsa -in rsa_private.key -pubout -out rsa_public.key
	writing RSA key
	- 查看内容
	
			ks@ubuntu-x64:/ks$ cat rsa_public.key
			-----BEGIN PUBLIC KEY-----
			MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCiZX+xB9DuezmSPkzRHAGNrkzf
			DQGNC5jIx9dwZFQQq/aLNZynRjrlVReY4rdPB/yFeQyyyHpiGoac73VyXeoxc0p6
			gvw1gSjaKb0+j7NXtRGRtfNvf3I2nmyOF9foo9CddrOKnAwd+hupCWkBm74bA9Lu
			eSSYNx+ujYngJ4YfGQIDAQAB
			-----END PUBLIC KEY-----

## RSA 签名
### SHA1
- 生成签名

		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -sign rsa_private.key -sha1 -out sha1_rsa_hello.sign hello.txt
- 验证签名

		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -verify rsa_public.key -sha1 -signature sha1_rsa_hello.sign hello.txt
		Verified OK

### MD5
- 生成签名

		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -sign rsa_private.key -md5 -out md5_rsa_hello.sign hello.txt
- 验证签名

		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -verify rsa_public.key -md5 -signature md5_rsa_hello.sign hello.txt
		Verified OK

### 验证
- 拷贝

		ks@ubuntu:/ks/openssl-rsa$ cp sha1_rsa_hello.sign sha1_rsa_hello1.sign
		ks@ubuntu:/ks/openssl-rsa$ cp hello.txt hello1.txt
		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -verify rsa_public.key -sha1 -signature sha1_rsa_hello1.sign hello1.txt
		Verified OK
- 修改一下内容，再验证

		ks@ubuntu:/ks/openssl-rsa$ vi hello1.txt
		ks@ubuntu:/ks/openssl-rsa$ openssl dgst -verify rsa_public.key -sha1 -signature sha1_rsa_hello1.sign hello1.txt
		Verification Failure
- 查看目录

		ks@ubuntu:/ks/openssl-rsa$ ls -l
		total 36
		-rw-rw-r-- 1 ks ks  117 Sep 25 08:49 hello.txt
		-rw-rw-r-- 1 ks ks  128 Sep 28 12:39 md5_rsa_hello.sign
		-rw-rw-r-- 1 ks ks 6373 Sep 28  2019 openssl-rsa.txt
		-rw-rw-r-- 1 ks ks  887 Sep 28 12:04 rsa_private.key
		-rw-rw-r-- 1 ks ks  272 Sep 28 12:04 rsa_public.key
		-rw-rw-r-- 1 ks ks  128 Sep 28 12:39 sha1_rsa_hello.sign

## RSA 加密
	vi hello.txt
	117 bytes ok （最多只能加密117个字节的文件 ）
	118 bytes KO

- 加密

		ks@ubuntu-x64:/ks$ openssl rsautl -encrypt -in hello.txt -inkey rsa_public.key -pubin -out hello.txt.en
- 解密

		ks@ubuntu-x64:/ks$ openssl rsautl -decrypt -in hello.txt.en -inkey rsa_private.key -out hello.txt.de

- 查看目录

	原始 hello.txt 为 117 字节，加密后 hello.txt.en 为 128 字节，解密后 hello.txt.de 为 117 字节

		ks@ubuntu-x64:/ks/openssl-rsa$ ls -l
		total 24
		-rw-rw-r-- 1 ks ks  117 Sep 25 00:49 hello.txt
		-rw-rw-r-- 1 ks ks  117 Sep 25 00:51 hello.txt.de
		-rw-rw-r-- 1 ks ks  128 Sep 25 00:49 hello.txt.en
		-rw-rw-r-- 1 ks ks 2212 Sep 25 01:05 openssl-rsa.txt
		-rw------- 1 ks ks  887 Sep 25 00:33 test.key
		-rw-rw-r-- 1 ks ks  272 Sep 25 00:34 test_pub.key

		ks@ubuntu-x64:/ks/openssl-rsa$ md5sum hello.txt
		91b23211f799d7f2ba84c321208a910d  hello.txt
		
		ks@ubuntu-x64:/ks/openssl-rsa$ md5sum hello.txt.en
		320e92ae71650974d85c417de770e708  hello.txt.en
		
		ks@ubuntu-x64:/ks/openssl-rsa$ md5sum hello.txt.de
		91b23211f799d7f2ba84c321208a910d  hello.txt.de