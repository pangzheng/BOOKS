# AES-CBC
## 数据准备
- 明文内容

		 % cat hello.txt
		openssl aes-256-cbc -in hello.txt –out encrypto.txt –iv f123 –K 123 –p solarfs solarfs solarfs solarfs solar
- 明文字节长度(117)	
	
		% ls -l
		total 8
		-rw-r--r--  1 Prometheus  staff  117  8 31 12:16 hello.txt
- md5

		% md5 hello.txt
		MD5 (hello.txt) = a3857d03d7b70be85b8e69bd989597db
 
## x64
- 指定 k,iv
	- 加密
	
		指定情况下，salt 时无效的，注意这里 k 是根据算法而定，算法是 128 就是 128 位，256就是 256 位。但是因为明文是固定128 位长度，所以 IV 也是 128 位，而 k,iv 会分别以 0 进行补位长度
		
		- 命令行
			
				openssl enc -aes-256-cbc -in hello.txt -out hello.txt.aes.cbc.en.x64 -iv f123 -K 1223 -p
				
				salt=CD8EE96CFF7F0000(无效)
				key=1223000000000000000000000000000000000000000000000000000000000000
				iv =F1230000000000000000000000000000
				
				hexdump -C hello.txt.aes.cbc.en.x64
				00000000  8f f9 2c bd c5 67 26 7d  58 e1 74 ef 4b f2 d9 f5  |..,..g&}X.t.K...|
				00000010  01 64 8d 09 b9 93 49 5d  aa 6f b1 9e 3f b9 d1 41  |.d....I].o..?..A|
				00000020  3e 23 ff fe e4 45 73 2a  ad fb 2c 39 74 fe 5e c3  |>#...Es*..,9t.^.|
				00000030  96 2a 6b 63 73 71 92 20  62 da 9a 3d 43 29 fa c6  |.*kcsq. b..=C)..|
				00000040  1e 90 44 7d 88 c5 e9 a7  58 a0 c3 11 3c c5 d9 e3  |..D}....X...<...|
				00000050  ce fd 31 68 2e 5e f9 7c  84 33 29 a1 92 d0 19 7c  |..1h.^.|.3)....||
				00000060  56 f4 e2 92 dd 98 31 6f  b2 30 16 7f 20 da 90 d1  |V.....1o.0.. ...|
				00000070  05 b7 f9 3e 0e 19 5e ac  4b e8 c3 a9 4e a2 62 bd  |...>..^.K...N.b.|
				00000080
		- 密文长度(128)
	
				-rw-r--r--  1 Prometheus  staff  128  8 31 12:22 hello.txt.aes.cbc.en.x64
		- md5
	
				% md5 hello.txt.aes.cbc.en.x64
				MD5 (hello.txt.aes.cbc.en.x64) = 0c385487c0289ee15a17e224696902f9
	- 解密
		- 命令行

				% openssl enc -d -aes-256-cbc -in hello.txt.aes.cbc.en.x64 -out hello.txt.aes.cbc.dn.x64 -iv f123 -K 1223 -p
				salt=CD8EE96CFF7F0000
				key=1223000000000000000000000000000000000000000000000000000000000000
				iv =F1230000000000000000000000000000
		- 解密文件长度(117)

				-rw-r--r--  1 Prometheus  staff  117  8 31 14:54 hello.txt.aes.cbc.dn.x64
		- md5

				% md5 hello.txt.aes.cbc.dn.x64(解密文件)
				MD5 (hello.txt.aes.cbc.dn.x64) = a3857d03d7b70be85b8e69bd989597db
- 使用 salt 随机生成加密

	随机生成 8个字节的 `salt` ，然后基于 `salt+passwd` 生成 `k,iv`，使用 `hexdump` 可以看见在密文 8-16 字节是 salt，注意看两次执行的 salt  不一样，但是密文1-8字节都是固定的。17-最后字节是加密后的文件
	
	- `salt + passwd`  生成 `k,iv` 算法是不一定的
		- openssl 0.* 使用 md5 算法
			
				hash1_128 = md5(passwd+salt)
				hash2_128 = md5(hash1_128+passwd+salt) 
				hash3_128 = md5(hash2_128+passwd+salt) 
				key = hash1_128+hash2_128
				IV = hash3_128
		- openshift11.1* 使用 sha256

				key = sha256(passwd+salt)
				IV = sha256(key+passwd+salt)[0:16]
	- 加密(执行第一遍)
		- 命令行
		
				% openssl enc -aes-256-cbc -in hello.txt -out hello.txt.aes.cbc.en.salt.x64 -pass pass:123456 -p
				salt=058ADA12A1294E62
				key=A51369BBC01F4B0E9770396654970866A9A2636E74264E515899423967FACE60
				iv =7BD9D1B6EC5C5E0767F100977D229682
		
				% hexdump -C hello.txt.aes.cbc.en.salt.x64
				00000000  53 61 6c 74 65 64 5f 5f(固定)  05 8a da 12 a1 29 4e 62(salt)  |Salted__.....)Nb|
				00000010  59 9a 83 03 75 38 03 e1  2c 2d ec 2f 56 e7 a0 e7  |Y...u8..,-./V...|
				00000020  6b 6d 86 60 bf 68 d8 e8  57 6f 59 83 83 65 7e ae  |km.`.h..WoY..e~.|
				00000030  6b 36 c4 2b 88 27 73 76  a0 78 ff b4 27 ab 7e 1a  |k6.+.'sv.x..'.~.|
				00000040  31 92 a5 7a 01 97 0a df  84 20 40 9a f3 ba 66 bb  |1..z..... @...f.|
				00000050  14 ed cf c2 9c 75 d0 33  e3 3b 04 88 4d 99 9f db  |.....u.3.;..M...|
				00000060  b1 9b 2e e0 fe 4a 05 7a  8b 10 91 42 a9 51 4e 92  |.....J.z...B.QN.|
				00000070  65 e1 4c 83 9f a7 50 09  42 82 16 7e 33 2b 35 4e  |e.L...P.B..~3+5N|
				00000080  85 0b 48 ba 68 4e bd 2b  16 4d 51 90 15 e6 71 3d  |..H.hN.+.MQ...q=|
				00000090		
		- 密文长度(144)

				-rw-r--r--  1 Prometheus  staff  144  8 31 12:34 hello.txt.aes.cbc.en.salt.x64
		- md5	
			
				% md5 hello.txt.aes.cbc.en.salt.x64
				MD5 (hello.txt.aes.cbc.en.salt.x64) = 733a05a60588b1811331650799cc307e
	- 解密
		- 命令行

				% openssl enc -d -aes-256-cbc -in hello.txt.aes.cbc.en.salt.x64 -out hello.txt.aes.cbc.dn.salt.x64 -pass pass:123456 -p
				salt=058ADA12A1294E62
				key=A51369BBC01F4B0E9770396654970866A9A2636E74264E515899423967FACE60
				iv =7BD9D1B6EC5C5E0767F100977D229682
		- 解密文件长度(117)

				-rw-r--r--  1 Prometheus  staff  117  8 31 15:03 hello.txt.aes.cbc.dn.salt.x64
		- md5
		
				% md5 hello.txt.aes.cbc.dn.salt.x64
				MD5 (hello.txt.aes.cbc.dn.salt.x64) = a3857d03d7b70be85b8e69bd989597db
	- 加密(再执行一遍)
		- 命令
		
				% openssl enc -aes-256-cbc -in hello.txt -out hello.txt.aes.cbc.en.salt.x64.2 -pass pass:123456 -p
				salt=DD2F2B9B9EF9F2AC
				key=23C56AEC214467D5D5D174005527488182EC4A2316042928593E89C6D1B797E4
				iv =878A32BD6834DE1AAC2A293DDA266F7B
				
				% hexdump -C hello.txt.aes.cbc.en.salt.x64.2
				00000000  53 61 6c 74 65 64 5f 5f(固定)  dd 2f 2b 9b 9e f9 f2 ac(salt)  |Salted__./+.....|
				00000010  92 c8 4a 16 ca 36 e9 fa  0b 66 1c 47 df bf 7a 0b  |..J..6...f.G..z.|
				00000020  7d c4 25 a5 d2 32 19 50  0a 11 c4 14 f7 9e b5 f5  |}.%..2.P........|
				00000030  5f 51 d7 c2 45 a2 be 3b  b6 92 ac 41 f2 70 06 f4  |_Q..E..;...A.p..|
				00000040  80 a5 ae e1 8c 5c 7a 9e  4f df 49 67 d9 53 17 51  |.....\z.O.Ig.S.Q|
				00000050  16 ae f3 ef 24 2f 7a 52  97 71 a1 3e 71 51 79 91  |....$/zR.q.>qQy.|
				00000060  68 3f 36 9d 44 e7 4b f8  c3 29 b6 28 76 46 36 37  |h?6.D.K..).(vF67|
				00000070  e6 20 0f 97 99 06 14 35  7e a1 8a d1 9a 5e 91 ff  |. .....5~....^..|
				00000080  6b 4f 08 62 2c c2 a3 f8  aa 59 bd 26 e2 03 88 d8  |kO.b,....Y.&....|
				00000090
		- 密文长度(144)

				-rw-r--r--  1 Prometheus  staff  144  8 31 14:09 hello.txt.aes.cbc.en.salt.x64.2
		- md5

				% md5 hello.txt.aes.cbc.en.salt.x64.2
				MD5 (hello.txt.aes.cbc.en.salt.x64.2) = 8308ccd51c9792360efd0852d8e05031
	- 解密
		- 命令行

				% openssl enc -d -aes-256-cbc -in hello.txt.aes.cbc.en.salt.x64.2 -out hello.txt.aes.cbc.dn.salt.x64.2 -pass pass:123456 -p
				salt=DD2F2B9B9EF9F2AC
				key=23C56AEC214467D5D5D174005527488182EC4A2316042928593E89C6D1B797E4
				iv =878A32BD6834DE1AAC2A293DDA266F7B
		- 解密文件长度(117)

				-rw-r--r--  1 Prometheus  staff  117  8 31 15:06 hello.txt.aes.cbc.dn.salt.x64.2
		- md5
		
				% md5 hello.txt.aes.cbc.dn.salt.x64.2
				MD5 (hello.txt.aes.cbc.dn.salt.x64.2) = a3857d03d7b70be85b8e69bd989597db
- openssl 命令行参数
	- `-p`
	
		-p 表示打印出加密用的salt, key, iv. salt就是所谓的加盐.
	
## 最终 md5 对比
- 原始文件md5
	
		% md5 hello.txt
		MD5 (hello.txt) = a3857d03d7b70be85b8e69bd989597db
- 加密 x64
	
	如下统计可以看出，`salt` 模式比固定的 `k,iv` 模式大 16 个字节，这个字节就是 `8 字节 magic number + 8 字节 salt`
	
	- 固定加密(128字节)
	
			% ls -l 
			-rw-r--r--  1 Prometheus  staff  128  8 31 12:22 hello.txt.aes.cbc.en.x64
	
			% md5 hello.txt.aes.cbc.en.x64
			MD5 (hello.txt.aes.cbc.en.x64) = 0c385487c0289ee15a17e224696902f9
	- 随机 salt(144字节)
	
			% ls -l 
			-rw-r--r--  1 Prometheus  staff  144  8 31 12:34 hello.txt.aes.cbc.en.salt.x64
	
			% md5 hello.txt.aes.cbc.en.salt.x64
			MD5 (hello.txt.aes.cbc.en.salt.x64) = 733a05a60588b1811331650799cc307e
	- 随机 salt2
			
			% ls -l 
			-rw-r--r--  1 Prometheus  staff  144  8 31 14:09 hello.txt.aes.cbc.en.salt.x64.2
			
			% md5 hello.txt.aes.cbc.en.salt.x64.2
			MD5 (hello.txt.aes.cbc.en.salt.x64.2) = 8308ccd51c9792360efd0852d8e05031					
- 解密 x86/64
	- 固定加密
	
			% md5 hello.txt.aes.cbc.dn.x64(解密文件)
			MD5 (hello.txt.aes.cbc.dn.x64) = a3857d03d7b70be85b8e69bd989597db
	- 随机 salt
		
			% md5 hello.txt.aes.cbc.dn.salt.x64
			MD5 (hello.txt.aes.cbc.dn.salt.x64) = a3857d03d7b70be85b8e69bd989597db
	- 随机 salt2

			% md5 hello.txt.aes.cbc.dn.salt.x64.2
			MD5 (hello.txt.aes.cbc.dn.salt.x64.2) = a3857d03d7b70be85b8e69bd989597db		

