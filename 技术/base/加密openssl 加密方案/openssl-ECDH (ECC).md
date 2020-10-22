# 应研 ECDH (ECC)
## 版本 v2
## 1 生成密钥
### 1.1 生成私钥
- 命令
	
		openssl ecparam -name secp256k1 -genkey -noout -out eccPrikey.pem
	- 参数
		
			-noout 就没有私钥前面的参数格式
- 查看私钥文件总共 {223字节}
 
		# cat eccPrikey.pem
		-----BEGIN EC PRIVATE KEY-----
		MHQCAQEEIK1KSmUuopNkLvVBC+jy5WsHqKfLU4bSBfNP/qXknhhooAcGBSuBBAAK
		oUQDQgAE87ZeW9S1gaasgUitNe6hvewFNPN50hEp6wNgt4h5ThRRn9KLArhFVqxv
		71VNXvcK+XJrr+a3Kcjj+U6vOKd2sQ==
		-----END EC PRIVATE KEY-----
- 转化为 hex{236个字符}

		30740201010420ad4a4a652ea293642ef5410be8f2e56b07a8a7cb5386d205f34ffea5e49e1868a00706052b8104000aa14403420004f3b65e5bd4b581a6ac8148ad35eea1bdec0534f379d21129eb0360b788794e14519fd28b02b84556ac6fef554d5ef70af9726bafe6b729c8e3f94eaf38a776b1

### 1.2 生成公钥
- 命令
	
		openssl ec -in eccPrikey.pem -pubout -out eccPubkey.pem
- 查看公钥{174字节}

		-----BEGIN PUBLIC KEY-----
		MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAE87ZeW9S1gaasgUitNe6hvewFNPN50hEp
		6wNgt4h5ThRRn9KLArhFVqxv71VNXvcK+XJrr+a3Kcjj+U6vOKd2sQ==
		-----END PUBLIC KEY-----
- 转化为hex{176个字符}

		3056301006072a8648ce3d020106052b8104000a03420004f3b65e5bd4b581a6ac8148ad35eea1bdec0534f379d21129eb0360b788794e14519fd28b02b84556ac6fef554d5ef70af9726bafe6b729c8e3f94eaf38a776b1

## 2 生成测试文件
- 命令，测试文件 {12字节}

		vi hello.txt

		"ecctestyang"
- md5

		8a00aea6ba5dd41a11d38d195e1e5fa0

## 3 文件加密
### 3.1 生成随机文件密钥 sympass0
- 命令，生成加密文件的文件密钥{7字节}

		vi sympass0
		"123456"

### 3.2 BFS 生成临时公私钥对
#### 3.2.1 生成临时私钥
- 命令，私钥{223字节}

		openssl ecparam -name secp256k1 -genkey -noout -out temPrikey.pem

		-----BEGIN EC PRIVATE KEY-----
		MHQCAQEEIHXhy+XrBf8dKeHnMj09xnw3sqJf+ln2C9rVKvyr+/KAoAcGBSuBBAAK
		oUQDQgAEaPYDyebV9zv2YEbFysdH9CSL2BQFydJvJHzDeKlW32jzXenldJ4WRRIH
		jTWHhTVJOh/1UcOvZWL9ZjJxZybEow==
		-----END EC PRIVATE KEY-----

- hex{236字符}

		3074020101042075e1cbe5eb05ff1d29e1e7323d3dc67c37b2a25ffa59f60bdad52afcabfbf280a00706052b8104000aa1440342000468f603c9e6d5f73bf66046c5cac747f4248bd81405c9d26f247cc378a956df68f35de9e5749e164512078d35878535493a1ff551c3af6562fd6632716726c4a3

#### 3.2.2 生成临时公钥
- 命令，公钥{174字节}

		openssl ec -in temPrikey.pem -pubout -out temPubkey.pem
		-----BEGIN PUBLIC KEY-----
		MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEfX8lnOfbPWVgvSO6Qt/1rggucjeJGqqd
		ZH+Du6/MRwIY5el7ZdN7iEQKugfJmQAAfuihuCcIYpu7d1+805708A==
		-----END PUBLIC KEY-----

- hex{176字符}

		3056301006072a8648ce3d020106052b8104000a034200047d7f259ce7db3d6560bd23ba42dff5ae082e7237891aaa9d647f83bbafcc470218e5e97b65d37b88440aba07c99900007ee8a1b82708629bbb775fbcd39ef4f0

### 3.3 生成共享密钥(万能密钥){用户公钥+临时私钥} => sharekeyEn
- 命令

		openssl pkeyutl -derive -out sharekeyEn -inkey temPrikey.pem -peerkey eccPubkey.pem
### 3.4 利用共享密钥对文件密钥(sympass0)进行加密得到加密文件密钥(sympassen0)
- 命令

		openssl enc -aes-256-cbc -in sympass0 -out sympassen0 -pass file:sharekeyEn
- 加密文件密钥 sympassen0 转化为 base64

		base64(sympassen0)
		U2FsdGVkX1+ZaXV2ZQHGfMymfAr8lR4Qn70KtfC1C6k=
- hex{64字符}

		53616c7465645f5f996975766501c67ccca67c0afc951e109fbd0ab5f0b50ba9


### 3.5 拼接{临时公钥+加密文件密钥} 得到正式加密文件密钥组( sympassen )
16进制模式进行拼接

	(临时共钥 hex+sympaas0 hex)=sympassen
	(176+64)=210字符

- hex(210字符)

		3056301006072a8648ce3d020106052b8104000a034200047d7f259ce7db3d6560bd23ba42dff5ae082e7237891aaa9d647f83bbafcc470218e5e97b65d37b88440aba07c99900007ee8a1b82708629bbb775fbcd39ef4f0/53616c7465645f5f996975766501c67ccca67c0afc951e109fbd0ab5f0b50ba9


### 3.6 密钥 sympass0 对文件加密 AES
- 命令

		openssl enc -aes-256-cbc -in hello.txt -out hello.txt_en -pass file:sympass0
		返回: hello.txt_en , sympassen
- 分析		
- 加密前文件数据为 12个字节

		MD5-After:  7211b7ccdc900ec507178505371dad1f
- 加密后文件数据 `hello.txt_en` 为32字节

		MD5-Before: 8a00aea6ba5dd41a11d38d195e1e5fa0

## 4 文件解密
## 4.1 读取 sympassen
拆分加密文件密钥组 sympassen

	 {加密过程临时公钥}+{加密文件密钥}
### 4.1.1 获取临时公钥
- 公钥 hex

		3056301006072a8648ce3d020106052b8104000a034200047d7f259ce7db3d6560bd23ba42dff5ae082e7237891aaa9d647f83bbafcc470218e5e97b65d37b88440aba07c99900007ee8a1b82708629bbb775fbcd39ef4f0

- 换算成Base64

		MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEaPYDyebV9zv2YEbFysdH9CSL2BQFydJvJHzDeKlW32jzXenldJ4WRRIHjTWHhTVJOh/1UcOvZWL9ZjJxZybEow==

- 组装公钥PEM：temPubkey.pem

		-----BEGIN PUBLIC KEY-----
		MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAEfX8lnOfbPWVgvSO6Qt/1rggucjeJGqqdZH+Du6/MRwIY5el7ZdN7iEQKugfJmQAAfuihuCcIYpu7d1+805708A==
		-----END PUBLIC KEY-----

### 4.1.2 获取AES加密文件密钥
- 加密文件密钥 hex(sympassen0_rec:)

		53616c7465645f5f996975766501c67ccca67c0afc951e109fbd0ab5f0b50ba9
- 转换成 base64
	
		U2FsdGVkX1+ZaXV2ZQHGfMymfAr8lR4Qn70KtfC1C6k=
- 获取加密文件密钥(sympassen0_rec)

		echo U2FsdGVkX1+ZaXV2ZQHGfMymfAr8lR4Qn70KtfC1C6k= | openssl base64 -d -out sympassen0_rec
		
		创建文件 sympassen0Base，将 base 64 的字符串输入，然后输出
		{{openssl base64 -d -in sympassen0Base -out sympassen0_rec}}

### 4.2 基于用户私钥+拆分的临时公钥，得到共享密钥
	openssl pkeyutl -derive -out sharekeyDe -inkey eccPrikey.pem -peerkey temPubkey.pem_rec
### 4.3 解密文件密钥，即 sympassen0 ==> sympass0_de
	openssl enc -d -aes-256-cbc -in sympassen0_rec -out sympass0_de -pass file:sharekeyDe

	cat sympass0_de
	"123456"
### 4.4 基于解密的 sympass0_en 解密文件
	openssl enc -d -aes-256-cbc -in hello.txt_en -out hello.txt_de -pass file:sympass0_de
	cat hello.txt.de
	"ecctestyang"
	

## 特别备注
1. ecdh 针对不同长度的 key，加密后的长度是不等

	在 bfs 中，统一采用 128 bits 的 AES 密钥长度，即 16 bytes，hex 模式为 32bytes。在这种模式下， sympassen （hex模式） = 176(共享公钥长度) + 96(密钥加密后长度） = 272 bytes

2. 因为 ecdh 中引入了共享密钥的概念，所以相同的 key 两次加密后的值是不同的（每次采用的共享密钥不同）。而考虑到引入共享密钥来实现 ecdh 算法，生成的时候需要共享私钥，而破解时候需要共享公钥，所以需要把共享公钥作为加密后的 AES 密码的一部分保存下来（也就是 sympassen的前176个字节）

3. 使用 `./afs-x64 demo2/demoecc` 可以演示 ecc 的加密文件上传和下载解密整个流程,下面为该demo的源码 和 部分运行结果：

		//演示RSA::1024/ECC::ECDH+AES加密文件上传
		ut_status uf_demo_help(const char *argv_0, const char *topic)
		{
			ut_bool is_show = UM_TRUE;
			//ut_bool is_show = UM_FALSE;
			char sys_cmd_in[UM_MAX_CMD];
			char sys_cmd_out[UM_MAX_CMD];
			char field[50] = "arfs";
			//char field[50] = "afs";
			int i, j, k;
			int kp_count = 0;
			int ok_count = 0;
			int ko_count = 0;
			int i_count = 2;  	// <=20
			int j_count = 3;	  	// <=30
			for(i=1; i<=i_count; i++)
			{
				char asymalgo[20] = "";
				if(!strcmp("demo1", topic) || !strcmp("demorsa", topic))
				{
					strcpy(asymalgo, "RSA");
					sprintf(sys_cmd_in, "%s \";_f=crtrsakeyp;\"", argv_0);
				}
				else if(!strcmp("demo2", topic) || !strcmp("demoecc", topic))
				{
					strcpy(asymalgo, "ECC");
					sprintf(sys_cmd_in, "%s \";_f=crtecckeyp;\"", argv_0);
				}
				else
				{
					printf("Error topic(%s)\n", topic);
					uf_sleep(100);
					continue;
				}
		
				uf_call_system(sys_cmd_in, sys_cmd_out);
				char asymprivatekey[1400];
				char asympubkey[500];
				uf_load_str(sys_cmd_out, "asymprivatekey", asymprivatekey);
				uf_load_str(sys_cmd_out, "asympubkey", asympubkey);
				if(is_show)
				{
					printf("==============(%d)==============\n", i);
					printf("#1\n");
					printf("sys_cmd_in=%s\n", sys_cmd_in);
					printf("sys_cmd_out=%s\n", sys_cmd_out);
					printf("asymprivatekey_len=%d\n", uf_strlen(asymprivatekey));
					printf("asympubkey_len=%d\n", uf_strlen(asympubkey));
					uf_sleep(1);
					printf("\n");
				}
				kp_count++;
		
				if(!strcmp("RSA", asymalgo))
				{
					if(strlen(asymprivatekey)<1200 || strlen(asymprivatekey)>1230)
					{
						printf("Error asymprivatekey_len(%d)\n", uf_strlen(asymprivatekey));
						uf_sleep(100);
						continue;
					}
					if(strlen(asympubkey)!=324)
					{
						printf("Error asympubkey_len(%d)\n", uf_strlen(asympubkey));
						uf_sleep(100);
						continue;
					}
				}
				else if(!strcmp("ECC", asymalgo))
				{
					if(strlen(asymprivatekey)!=236)
					{
						printf("Error asymprivatekey_len(%d)\n", uf_strlen(asymprivatekey));
						uf_sleep(100);
						continue;
					}
					if(strlen(asympubkey)!=176)
					{
						printf("Error asympubkey_len(%d)\n", uf_strlen(asympubkey));
						uf_sleep(100);
						continue;
					}
				}
		
				for(j=1; j<=3; j++)
				{
					if(!strcmp("RSA", asymalgo))
					{
						printf("batch demo the encryption process(RSA::1024 and AES) -- (%d-%d).\n", i, j);
					}
					else if(!strcmp("ECC", asymalgo))
					{
						printf("batch demo the encryption process(ECC::ECDH and AES) -- (%d-%d).\n", i, j);
					}
					if(is_show)
					{
						uf_sleep(1);
						printf("\n");
					}
		
					uf_unlink("1.txt");
					for(k=0; k<2000; k++)
					{
						char rand_str[1001];
						uf_rand_str(rand_str, rand()%500+500, "abcd1234");
						sprintf(sys_cmd_in, "echo \"%s\" >> 1.txt\n", rand_str);
						uf_call_system(sys_cmd_in, sys_cmd_out);
						if(k>500 & rand()%500==0)
							break;
					}
					char txt_md5[33] = "";
					uf_get_file_md5("1.txt", txt_md5);
					if(is_show)
					{
						printf("#2\n");
						printf("#generate a txt_file\n");
						printf("txt_file_length=%d\n", uf_get_file_length("1.txt"));
						printf("txt_md5=%s\n", txt_md5);
						system("md5sum 1.txt");
						uf_sleep(1);
						printf("\n");
					}
		
					sprintf(sys_cmd_in, "%s \";_f=upload;local_file=1.txt;field=%s;\"", argv_0, field);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					char afid_pub[200] = "";
					uf_load_str(sys_cmd_out, "afid", afid_pub);
					if(is_show)
					{
						printf("#3\n");
						printf("#upload this txt_file to %s by mode(public)\n", field);
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						printf("afid_pub=%s\n", afid_pub);
						uf_sleep(1);
						printf("\n");
					}
		
					sprintf(sys_cmd_in, "%s \";_f=upload;local_file=1.txt;field=%s;mode=en-proxy;asymalgo=%s;asympubkey=%s;symalgo=AES;\"", argv_0, field, asymalgo, asympubkey);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					char afid_en[200];
					uf_load_str(sys_cmd_out, "afid", afid_en);
					if(is_show)
					{
						printf("#4\n");
						printf("#upload this txt_file to afs/arfs by mode(en-proxy)\n");
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						printf("afid_en=%s\n", afid_en);
						uf_sleep(1);
						printf("\n");
					}
		
					sprintf(sys_cmd_in, "%s \";_f=read_p;afid=%s;p_name=sympassen;field=%s;\"", argv_0, afid_en, field);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					char sympassen[300];
					uf_load_str(sys_cmd_out, "p_value", sympassen);
					if(is_show)
					{
						printf("#5\n");
						printf("#read parameter(sympassen)'s value\n");
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						printf("sympassen=%s\n", sympassen);
						printf("sympassen_len=%d\n", uf_strlen(sympassen));
						uf_sleep(1);
						printf("\n");
					}
		
					char asym_de_f[20] = "";
					if(!strcmp("RSA", asymalgo))
					{
						strcpy(asym_de_f, "dersa");
					}
					else if(!strcmp("ECC", asymalgo))
					{
						strcpy(asym_de_f, "deecc");
					}
					sprintf(sys_cmd_in, "%s \";_f=%s;sympassen=%s;asymprivatekey=%s;\"", argv_0, asym_de_f, sympassen, asymprivatekey);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					char sympass[200];
					uf_load_str(sys_cmd_out, "sympass", sympass);
					if(is_show)
					{
						printf("#6\n");
						printf("#decrypt the aes_key using %s\n", asymalgo);
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						printf("sympass=%s\n", sympass);
						printf("sympass_len=%d\n", uf_strlen(sympass));
						uf_sleep(1);
						printf("\n");
					}
		
					sprintf(sys_cmd_in, "%s \";_f=download;afid=%s;local_file=1-en.txt;\"", argv_0, afid_en);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					if(is_show)
					{
						printf("#7\n");
						printf("#download the encrypted file\n");
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						uf_sleep(1);
						printf("\n");
					}
		
					sprintf(sys_cmd_in, "%s \";_f=deaes;symalgo=AES;sympass=%s;en_file=1-en.txt;de_file=1-de.txt;\"", argv_0, sympass);
					uf_call_system(sys_cmd_in, sys_cmd_out);
					if(is_show)
					{
						printf("#8\n");
						printf("#decrypt the en_file\n");
						printf("sys_cmd_in=%s\n", sys_cmd_in);
						printf("sys_cmd_out=%s\n", sys_cmd_out);
						uf_sleep(1);
						printf("\n");
					}
		
					char de_txt_md5[33];
					uf_get_file_md5("1-de.txt", de_txt_md5);
					if(is_show)
					{
						printf("#9\n");
						printf("___txt_md5=%s\n", txt_md5);
						printf("de_txt_md5=%s\n", de_txt_md5);
					}
		
					if(!strcmp(txt_md5, de_txt_md5))
					{
						printf("OK\n");
						ok_count++;
					}
					else
					{
						printf("KO\n");
						ko_count++;
					}
		
					if(is_show)
					{
						printf("ok_count=%d\n", ok_count);
						printf("ko_count=%d\n", ko_count);
					}
		
					if(is_show)
					{
						for(k=0; k<20; k++)
						{
							printf("%d ", (20-k));
							fflush(stdout);
							uf_sleep(1);
						}
						printf("\n");
					}
		
					uf_unlink("1.txt");
					uf_unlink("1-en.txt");
					uf_unlink("1-de.txt");
				}
			}
			printf("key_pair_count=%d\n", kp_count);
			printf("ok_count=%d\n", ok_count);
			printf("ko_count=%d\n", ko_count);
		
			return 1;
		}
	
## afs ecc demo 执行
	ks@ubuntu:/ks/afs-src$ ./afs-x86 demo2
	[demo2]
	==============(1)==============
	#1
	sys_cmd_in=./afs-x86 ";_f=crtecckeyp;"
	sys_cmd_out=;_r=true;asymalgo=ECC;asymprivatekey=30740201010420ed3b2c2d6cfc0e5c8ba44923eb9bf8bf702cd3bde63b2a00b2203c028226bcf2a00706052b8104000aa14403420004ed63272af5b1337cde4141eb0cfb9c2841409f7444fdb279276f921cf7d1112486bcad5fae7c3e7d99827570c580f276646182790f2536c42c688a98a5bbe065;asympubkey=3056301006072a8648ce3d020106052b8104000a03420004ed63272af5b1337cde4141eb0cfb9c2841409f7444fdb279276f921cf7d1112486bcad5fae7c3e7d99827570c580f276646182790f2536c42c688a98a5bbe065;
	asymprivatekey_len=236
	asympubkey_len=176
	
	batch demo the encryption process(ECC::ECDH and AES) -- (1-1).
	
	#2
	# 生成1个txt文件 txt_file
	txt_file_length=564057
	txt_md5=89ac96a027b7c4db6714daebf0b0bd0a
	89ac96a027b7c4db6714daebf0b0bd0a  1.txt
	
	#3
	# 上传 txt_file 到 arfs (不加密)
	sys_cmd_in=./afs-x86 ";_f=upload;local_file=1.txt;field=arfs;"
	sys_cmd_out=;_r=true;;_t=0.028946;afid=1e00000000089b59fe5c07c18cd7966dcbe9b843bf496f17db1bf4a889ac96a027b7c4db6714daebf0b0bd0a92a5519f3ac3903fa5cf6b0b64adc3eea3bd4ec1;field=|arfs|;
	afid_pub=1e00000000089b59fe5c07c18cd7966dcbe9b843bf496f17db1bf4a889ac96a027b7c4db6714daebf0b0bd0a92a5519f3ac3903fa5cf6b0b64adc3eea3bd4ec1
	
	#4
	#上传 txt_file 到双域(en-proxy，代理加密)
	sys_cmd_in=./afs-x86 ";_f=upload;local_file=1.txt;field=arfs;mode=en-proxy;asymalgo=ECC;asympubkey=3056301006072a8648ce3d020106052b8104000a03420004ed63272af5b1337cde4141eb0cfb9c2841409f7444fdb279276f921cf7d1112486bcad5fae7c3e7d99827570c580f276646182790f2536c42c688a98a5bbe065;symalgo=AES;"
	sys_cmd_out=;_r=true;;_t=0.025542;afid=1e10000000089b60f60138edab237ed90f9af8c859456f54a3db60be58c9be7f42cc69aa58bd2c5f2248c5e372149d15b93cb19708215aa2095434277e4676c8;field=|arfs|;
	afid_en=1e10000000089b60f60138edab237ed90f9af8c859456f54a3db60be58c9be7f42cc69aa58bd2c5f2248c5e372149d15b93cb19708215aa2095434277e4676c8
	
	#5
	#读取 parameter(sympassen)'s value
	sys_cmd_in=./afs-x86 ";_f=read_p;afid=1e10000000089b60f60138edab237ed90f9af8c859456f54a3db60be58c9be7f42cc69aa58bd2c5f2248c5e372149d15b93cb19708215aa2095434277e4676c8;p_name=sympassen;field=arfs;"
	sys_cmd_out=;_r=true;p_name=sympassen;p_value=3056301006072a8648ce3d020106052b8104000a034200047d2a45d53a5be7907479e5390c11f39edb24ee52820eb2fe7c55334eeb69c3c920c64f6b8acdcd431af528891f13e5a8e160878cfead8325fa22d77bea53be2e53616c7465645f5f7911d1a1a3c7afa492274ae46f2aeeb1e6be222b24248b289712251f0754e1b7ef74f993e15827a3;field=|arfs|;parameter_value=3056301006072a8648ce3d020106052b8104000a034200047d2a45d53a5be7907479e5390c11f39edb24ee52820eb2fe7c55334eeb69c3c920c64f6b8acdcd431af528891f13e5a8e160878cfead8325fa22d77bea53be2e53616c7465645f5f7911d1a1a3c7afa492274ae46f2aeeb1e6be222b24248b289712251f0754e1b7ef74f993e15827a3;
	sympassen=3056301006072a8648ce3d020106052b8104000a034200047d2a45d53a5be7907479e5390c11f39edb24ee52820eb2fe7c55334eeb69c3c920c64f6b8acdcd431af528891f13e5a8e160878cfead8325fa22d77bea53be2e53616c7465645f5f7911d1a1a3c7afa492274ae46f2aeeb1e6be222b24248b289712251f0754e1b7ef74f993e15827a3
	sympassen_len=272
	
	#6
	#使用 ecc 解密 aes_key
	sys_cmd_in=./afs-x86 ";_f=deecc;sympassen=3056301006072a8648ce3d020106052b8104000a034200047d2a45d53a5be7907479e5390c11f39edb24ee52820eb2fe7c55334eeb69c3c920c64f6b8acdcd431af528891f13e5a8e160878cfead8325fa22d77bea53be2e53616c7465645f5f7911d1a1a3c7afa492274ae46f2aeeb1e6be222b24248b289712251f0754e1b7ef74f993e15827a3;asymprivatekey=30740201010420ed3b2c2d6cfc0e5c8ba44923eb9bf8bf702cd3bde63b2a00b2203c028226bcf2a00706052b8104000aa14403420004ed63272af5b1337cde4141eb0cfb9c2841409f7444fdb279276f921cf7d1112486bcad5fae7c3e7d99827570c580f276646182790f2536c42c688a98a5bbe065;"
	sys_cmd_out=;_r=true;sympass=6e3107b7934ea272a57197655f0ff685;
	sympass=6e3107b7934ea272a57197655f0ff685
	sympass_len=32
	
	#7
	#下载加密文件
	sys_cmd_in=./afs-x86 ";_f=download;afid=1e10000000089b60f60138edab237ed90f9af8c859456f54a3db60be58c9be7f42cc69aa58bd2c5f2248c5e372149d15b93cb19708215aa2095434277e4676c8;local_file=1-en.txt;"
	sys_cmd_out=;_r=true;file_length=564064;field=|arfs|;
	
	#8
	#解密加密文件
	sys_cmd_in=./afs-x86 ";_f=deaes;symalgo=AES;sympass=6e3107b7934ea272a57197655f0ff685;en_file=1-en.txt;de_file=1-de.txt;"
	sys_cmd_out=;_r=true;de_file_lenght=564057;
	
	#9 得到文件数据
	___txt_md5=89ac96a027b7c4db6714daebf0b0bd0a
	de_txt_md5=89ac96a027b7c4db6714daebf0b0bd0a
	OK
	ok_count=1
	ko_count=0

	