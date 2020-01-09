# FFmpeg使用
HTTP Live Streaming 或 HLS 是由 Apple 开发的一种流行的实时流媒体和点播视频技术。HLS 使用HTTP 协议，该协议允许用户从常规Web服务器而不是专用流服务器进行流传输。

## 什么是 FFmpeg
FFmpeg 是一个多媒体框架，能解码、编码、转码、mux、demux、流、过滤和播放人类和机器创建的几乎所有内容。在计算机上安装 FFmpeg 将为您提供上述所有功能，但是本文将重点介绍 FFmpeg 的 Muxers。混合器允许您将多媒体流写入特定的文件类型

- [ffmpeg的官网地址](https://www.ffmpeg.org/)
- [ffmpeg的Github项目](https://github.com/FFmpeg/FFmpeg)

### FFmpeg 组件
有以下几部分组成：

- `FFmpeg`

	视频文件转换命令行工具,也支持经过实时电视卡抓取和编码成视频文件；
- `ffserver`

	基于HTTP、RTSP用于实时广播的多媒体服务器.也支持时间平移；
- `ffplay`

	用 SDL和FFmpeg库开发的一个简单的媒体播放器；
- `libavcodec`

	一个包含了所有FFmpeg音视频编解码器的库。为了保证最优性能和高可复用性，大多数编解码器从头开发的；
- `libavformat`

	一个包含了所有的普通音视格式的解析器和产生器的库。

### 谁在使用ffmpeg
- 使用FFMPEG作为内核视频播放器：Mplayer，ffplay，射手播放器，暴风影音，KMPlayer，QQ影音...
- 使用FFMPEG作为内核的Directshow Filter：ffdshow，lav filters...
- 使用FFMPEG作为内核的转码工具：ffmpeg，格式工厂...


## 安装 FFmpeg on Mac
	brew install ffmpeg

	安装完成提示
	
	gettext is keg-only, which means it was not symlinked into /usr/local,
	because macOS provides the BSD gettext library & some software gets confused if both are in the library path.
	
	If you need to have gettext first in your PATH run:
	  echo 'export PATH="/usr/local/opt/gettext/bin:$PATH"' >> ~/.bash_profile
	
	For compilers to find gettext you may need to set:
	  export LDFLAGS="-L/usr/local/opt/gettext/lib"
	  export CPPFLAGS="-I/usr/local/opt/gettext/include"
	
	==> libffi
	libffi is keg-only, which means it was not symlinked into /usr/local,
	because some formulae require a newer version of libffi.
	
	For compilers to find libffi you may need to set:
	  export LDFLAGS="-L/usr/local/opt/libffi/lib"
	
	For pkg-config to find libffi you may need to set:
	  export PKG_CONFIG_PATH="/usr/local/opt/libffi/lib/pkgconfig"
	
	==> openssl@1.1
	A CA file has been bootstrapped using certificates from the system
	keychain. To add additional certificates, place .pem files in
	  /usr/local/etc/openssl@1.1/certs
	
	and run
	  /usr/local/opt/openssl@1.1/bin/c_rehash
	
	openssl@1.1 is keg-only, which means it was not symlinked into /usr/local,
	because openssl/libressl is provided by macOS so don't link an incompatible version.
	
	If you need to have openssl@1.1 first in your PATH run:
	  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.bash_profile
	
	For compilers to find openssl@1.1 you may need to set:
	  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
	  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"
	
	For pkg-config to find openssl@1.1 you may need to set:
	  export PKG_CONFIG_PATH="/usr/local/opt/openssl@1.1/lib/pkgconfig"
	
	==> unbound
	To have launchd start unbound now and restart at startup:
	  sudo brew services start unbound
	==> readline
	readline is keg-only, which means it was not symlinked into /usr/local,
	because macOS provides the BSD libedit library, which shadows libreadline.
	In order to prevent conflicts when programs look for libreadline we are
	defaulting this GNU Readline installation to keg-only.
	
	For compilers to find readline you may need to set:
	  export LDFLAGS="-L/usr/local/opt/readline/lib"
	  export CPPFLAGS="-I/usr/local/opt/readline/include"
	
	For pkg-config to find readline you may need to set:
	  export PKG_CONFIG_PATH="/usr/local/opt/readline/lib/pkgconfig"
	
	==> sqlite
	sqlite is keg-only, which means it was not symlinked into /usr/local,
	because macOS provides an older sqlite3.
	
	If you need to have sqlite first in your PATH run:
	  echo 'export PATH="/usr/local/opt/sqlite/bin:$PATH"' >> ~/.bash_profile
	
	For compilers to find sqlite you may need to set:
	  export LDFLAGS="-L/usr/local/opt/sqlite/lib"
	  export CPPFLAGS="-I/usr/local/opt/sqlite/include"
	
	For pkg-config to find sqlite you may need to set:
	  export PKG_CONFIG_PATH="/usr/local/opt/sqlite/lib/pkgconfig"
	
	==> python
	Python has been installed as
	  /usr/local/bin/python3
	
	Unversioned symlinks `python`, `python-config`, `pip` etc. pointing to
	`python3`, `python3-config`, `pip3` etc., respectively, have been installed into
	  /usr/local/opt/python/libexec/bin
	
	If you need Homebrew's Python 2.7 run
	  brew install python@2
	
	You can install Python packages with
	  pip3 install <package>
	They will install into the site-package directory
	  /usr/local/lib/python3.7/site-packages
	
	See: https://docs.brew.sh/Homebrew-and-Python
	==> glib
	Bash completion has been installed to:
	  /usr/local/etc/bash_completion.d
	==> tesseract
	This formula contains only the "eng", "osd", and "snum" language data files.
	If you need all the other supported languages, `brew install tesseract-lang`.
	
## 未加密转换
### 准备
- 准备一个目录为 `normal`
- 准备一个 mp4 文件

	如下

		$ ll
		total 51960
		-rw-r--r--@  1 Prometheus  staff    25M 10  1 22:59 acecombat7.mp4
		drwxr-xr-x  15 Prometheus  staff   480B 12  4 19:42 normal
	
### 执行转换命令
	$ ffmpeg -i acecombat7.mp4 -profile:v baseline -level 3.0 -s 640x360 -start_number 0 -hls_time 10 -hls_list_size 0 -f hls normal/F14OnDCS.m3u8
	
	结果
	
	ffmpeg version 4.2.1 Copyright (c) 2000-2019 the FFmpeg developers
	  built with Apple clang version 11.0.0 (clang-1100.0.33.8)
	  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.2.1_1 --enable-shared --enable-pthreads --enable-version3 --enable-avresample --cc=clang --host-cflags='-I/Library/Java/JavaVirtualMachines/adoptopenjdk-13.jdk/Contents/Home/include -I/Library/Java/JavaVirtualMachines/adoptopenjdk-13.jdk/Contents/Home/include/darwin -fno-stack-check' --host-ldflags= --enable-ffplay --enable-gnutls --enable-gpl --enable-libaom --enable-libbluray --enable-libmp3lame --enable-libopus --enable-librubberband --enable-libsnappy --enable-libtesseract --enable-libtheora --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-libfontconfig --enable-libfreetype --enable-frei0r --enable-libass --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenjpeg --enable-librtmp --enable-libspeex --enable-libsoxr --enable-videotoolbox --disable-libjack --disable-indev=jack
	  libavutil      56. 31.100 / 56. 31.100
	  libavcodec     58. 54.100 / 58. 54.100
	  libavformat    58. 29.100 / 58. 29.100
	  libavdevice    58.  8.100 / 58.  8.100
	  libavfilter     7. 57.100 /  7. 57.100
	  libavresample   4.  0.  0 /  4.  0.  0
	  libswscale      5.  5.100 /  5.  5.100
	  libswresample   3.  5.100 /  3.  5.100
	  libpostproc    55.  5.100 / 55.  5.100
	Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'acecombat7.mp4':
	  Metadata:
	    major_brand     : isom
	    minor_version   : 512
	    compatible_brands: isomiso2mp41
	    date            : 2019
	    encoder         : Lavf58.20.100
	  Duration: 00:02:01.99, start: 0.000000, bitrate: 1739 kb/s
	    Stream #0:0(und): Video: mpeg4 (Simple Profile) (mp4v / 0x7634706D), yuv420p, 720x576 [SAR 64:45 DAR 16:9], 1602 kb/s, 30 fps, 30 tbr, 15360 tbn, 30 tbc (default)
	    Metadata:
	      handler_name    : VideoHandler
	    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 44100 Hz, stereo, fltp, 128 kb/s (default)
	    Metadata:
	      handler_name    : SoundHandler
	Stream mapping:
	  Stream #0:0 -> #0:0 (mpeg4 (native) -> h264 (libx264))
	  Stream #0:1 -> #0:1 (aac (native) -> aac (native))
	Press [q] to stop, [?] for help
	[libx264 @ 0x7f9cd8823600] using SAR=1/1
	[libx264 @ 0x7f9cd8823600] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 BMI2 AVX2
	[libx264 @ 0x7f9cd8823600] profile Constrained Baseline, level 3.0
	[libx264 @ 0x7f9cd8823600] 264 - core 155 r2917 0a84d98 - H.264/MPEG-4 AVC codec - Copyleft 2003-2018 - http://www.videolan.org/x264.html - options: cabac=0 ref=3 deblock=1:0:0 analyse=0x1:0x111 me=hex subme=7 psy=1 psy_rd=1.00:0.00 mixed_ref=1 me_range=16 chroma_me=1 trellis=1 8x8dct=0 cqm=0 deadzone=21,11 fast_pskip=1 chroma_qp_offset=-2 threads=11 lookahead_threads=1 sliced_threads=0 nr=0 decimate=1 interlaced=0 bluray_compat=0 constrained_intra=0 bframes=0 weightp=0 keyint=250 keyint_min=25 scenecut=40 intra_refresh=0 rc_lookahead=40 rc=crf mbtree=1 crf=23.0 qcomp=0.60 qpmin=0 qpmax=69 qpstep=4 ip_ratio=1.40 aq=1:1.00
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS0.ts' for writing
	Output #0, hls, to 'normal/F14OnDCS.m3u8':
	  Metadata:
	    major_brand     : isom
	    minor_version   : 512
	    compatible_brands: isomiso2mp41
	    date            : 2019
	    encoder         : Lavf58.29.100
	    Stream #0:0(und): Video: h264 (libx264), yuv420p, 640x360 [SAR 1:1 DAR 16:9], q=-1--1, 30 fps, 90k tbn, 30 tbc (default)
	    Metadata:
	      handler_name    : VideoHandler
	      encoder         : Lavc58.54.100 libx264
	    Side data:
	      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: -1
	    Stream #0:1(und): Audio: aac (LC), 44100 Hz, stereo, fltp, 128 kb/s (default)
	    Metadata:
	      handler_name    : SoundHandler
	      encoder         : Lavc58.54.100 aac
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=13.5x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS1.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=11.8x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS2.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=10.8x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS3.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=10.5x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS4.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=10.5x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS5.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=10.5x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS6.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=10.1x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS7.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=  10x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS8.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=9.83x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS9.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=9.71x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS10.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=9.62x
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS11.ts' for writing
	[hls @ 0x7f9cd882c400] Opening 'normal/F14OnDCS.m3u8.tmp' for writingd=9.55x
	frame= 3658 fps=282 q=-1.0 Lsize=N/A time=00:02:01.97 bitrate=N/A speed=9.42x
	video:15143kB audio:1913kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: unknown
	[libx264 @ 0x7f9cd8823600] frame I:21    Avg QP:21.99  size: 27284
	[libx264 @ 0x7f9cd8823600] frame P:3637  Avg QP:25.50  size:  4106
	[libx264 @ 0x7f9cd8823600] mb I  I16..4: 26.8%  0.0% 73.2%
	[libx264 @ 0x7f9cd8823600] mb P  I16..4:  2.1%  0.0%  0.4%  P16..4: 41.8% 15.8%  5.8%  0.0%  0.0%    skip:34.0%
	[libx264 @ 0x7f9cd8823600] coded y,uvDC,uvAC intra: 25.4% 54.4% 16.4% inter: 20.2% 17.9% 1.3%
	[libx264 @ 0x7f9cd8823600] i16 v,h,dc,p: 19% 49% 12% 20%
	[libx264 @ 0x7f9cd8823600] i4 v,h,dc,ddl,ddr,vr,hd,vl,hu: 30% 23% 18%  5%  5%  4%  5%  4%  4%
	[libx264 @ 0x7f9cd8823600] i8c dc,h,v,p: 56% 29% 12%  3%
	[libx264 @ 0x7f9cd8823600] ref P L0: 77.8% 13.6%  8.7%
	[libx264 @ 0x7f9cd8823600] kb/s:1017.30
	[aac @ 0x7f9cd8824e00] Qavg: 150.463
### 查看转换完毕的数据
- 查看目录

		$ ll normal/
		total 42160
		-rw-r--r--@ 1 Prometheus  staff   468B 12  4 19:21 F14OnDCS.m3u8
		-rw-r--r--@ 1 Prometheus  staff   1.5M 12  4 19:21 F14OnDCS0.ts
		-rw-r--r--  1 Prometheus  staff   1.3M 12  4 19:21 F14OnDCS1.ts
		-rw-r--r--  1 Prometheus  staff   1.2M 12  4 19:21 F14OnDCS10.ts
		...
		-rw-r--r--  1 Prometheus  staff   1.1M 12  4 19:21 F14OnDCS8.ts
		-rw-r--r--  1 Prometheus  staff   1.2M 12  4 19:21 F14OnDCS9.ts

- 查看 m3u8 文件
	
		$ cat normal/F14OnDCS.m3u8
		#EXTM3U
		#EXT-X-VERSION:3
		#EXT-X-TARGETDURATION:17
		#EXT-X-MEDIA-SEQUENCE:0
		#EXTINF:14.700000,
		F14OnDCS0.ts
		#EXTINF:8.333333,
		...
		F14OnDCS9.ts
		#EXTINF:8.333333,
		F14OnDCS10.ts
		#EXTINF:11.800000,
		F14OnDCS11.ts
		#EXT-X-ENDLIST

### 测试
可以使用 `ffplay` 测试转换结果

	$ ffplay normal/F14OnDCS.m3u8
## 合并回 mp4
	$ ffmpeg -i F14OnDCS.m3u8 -acodec copy -vcodec copy -f mp4 F14.mp4
### 查看转换完毕的数据
- 查看目录

		$ ll
		total 77576
		-rw-r--r--  1 Prometheus  staff    17M 12  4 20:09 F14.mp4
		-rw-r--r--  1 Prometheus  staff   468B 12  4 19:56 F14OnDCS.m3u8
		-rw-r--r--  1 Prometheus  staff   1.5M 12  4 19:56 F14OnDCS0.ts
- 测试

	双击打开 mp4 文件	

## 加密转换
### 基础知识
- 什么是 openssl rand

	openssl rand 用于产生指定长度个bytes的随机字符
	
		openssl rand[-out file] [-randfile(s)] [-base64] [-hex]num
	-  `-out` 

		将随机字符放到一个文件中
	- `-base64 / -hex` 

		对随机字符串进行base64编码或用hex格式显示，如
		
		- hex
		
				$ openssl rand -hex 10
				1fc085778fc6537688c2
		- base64

				openssl rand -base64 10
				nDkalLiR9sJF7A==
							
### 准备工作
- 准备一个目录为 `encryption`
- 创建一个16字节的 key

		$ openssl rand 16 > enc.key
- 查看，

		$ xxd enc.key

		00000000: a02c abfc 2e0e 466e 0528 9fa8 ea2d 473f  .,....Fn.(...-G?-
- 获取加密用 iv 并查看值

		$ openssl rand -hex 16 > enc.iv
- 查看

		$ xxd enc.iv
		
		00000000: 6136 6231 3035 3039 6332 3330 6561 3138  a6b10509c230ea18
		00000010: 3566 3337 6161 3532 3336 6365 6664 3062  5f37aa5236cefd0b
		00000020: 0a                                       .		
		$ cat enc.iv
		
		a6b10509c230ea185f37aa5236cefd0b
- 生成一个 key 文件
	- 参数 
		- Key URI 
			
			enc.key 的路径，使用 http 形式
		- Path to key file
	
			enc.key 文件
		- IV
	
			上面生成的iv
	- 例子

			http://47.75.83.3:5145/dn/short/1e00e86f2a0ea35926b3304f-solarfs.key
			enc.key
			a6b10509c230ea185f37aa5236cefd0b
- 加密压缩

		ffmpeg -y -i test.mp4 -hls_time 12 -hls_key_info_file enc.keyinfo -hls_playlist_type vod -hls_segment_filename "EN-F14%d.ts" EN-F14.m3u8
		
		参数说明
		-i 							解析文件
		-hls_time					分割时间
		-hls_key_info_file		加密数据索引
		-hls_playlist_type		播放模式(vod 点播)
		-hls_segment_filename	小段文件名字
- 播放

		ffplay -protocol_whitelist file,http,https,tcp,tls,crypto EN-F14.m3u8

## [树莓派安装](https://github.com/pangzheng/BOOKS/blob/master/%E6%8A%80%E6%9C%AF/%E6%A0%91%E8%8E%93%E6%B4%BE/%E6%A0%91%E8%8E%93%E6%B4%BE%E5%AE%89%E8%A3%85%E6%A0%91%E8%8E%93%E7%B3%BB%E7%BB%9F.md)
## 树莓派摄像头直播(m3u8)
- 首先安装 ffmpeg-支持树莓派硬件
	- 安装 264

			cd /usr/src
			git clone git://git.videolan.org/x264
			cd x264
			./configure  --enable-static --enable-strip --disable-cli --enable-shared
			make && make install 
	- 安装 ffmpeg
		- 安装 `pkg-config`

				apt install pkg-config 
		- 安装 freetype(`ERROR: freetype2 not found using pkg-config`)
		
				wget https://sourceforge.net/projects/freetype/files/freetype2/2.8.1/freetype-2.8.1.tar.gz
				tar xzvf freetype-2.8.1.tar.gz && cd freetype-2.8.1
				./configure
				make && sudo make install
		- 安装 libomxil-bellagio(`ERROR: OMX_Core.h not found`) 
				
				apt-get install libomxil-bellagio-dev
		- 安装 image2(`Requested output format 'image2' is not a suitable output format`)

				编译增加 --enable-muxer=image2
		- 安装 ffmpeg
	
				cd /usr/src
				wget  http://ffmpeg.org/releases/ffmpeg-4.2.tar.gz
				tar xzvf ffmpeg-4.2.tar.gz && cd ffmpeg-4.2
				./configure --enable-gpl --enable-version3 --enable-nonfree --enable-static --disable-shared --disable-opencl --disable-thumb --disable-pic --disable-stripping --enable-small --enable-ffmpeg --enable-ffplay --enable-ffprobe --disable-doc --disable-htmlpages --disable-podpages --disable-txtpages --disable-manpages --disable-everything --enable-libx264 --enable-encoder=libx264 --enable-decoder=h264 --enable-encoder=aac --enable-decoder=aac --enable-encoder=ac3 --enable-decoder=ac3 --enable-encoder=rawvideo --enable-decoder=rawvideo --enable-encoder=mjpeg --enable-decoder=mjpeg --enable-demuxer=concat --enable-muxer=flv --enable-demuxer=flv --enable-demuxer=live_flv --enable-muxer=hls --enable-muxer=segment --enable-muxer=stream_segment --enable-muxer=mov --enable-demuxer=mov --enable-muxer=mp4 --enable-muxer=mpegts --enable-demuxer=mpegts --enable-demuxer=mpegvideo --enable-muxer=matroska --enable-demuxer=matroska --enable-muxer=wav --enable-demuxer=wav --enable-muxer=pcm* --enable-demuxer=pcm* --enable-muxer=rawvideo --enable-demuxer=rawvideo --enable-muxer=rtsp --enable-demuxer=rtsp --enable-muxer=rtsp --enable-demuxer=sdp --enable-muxer=fifo --enable-muxer=tee --enable-parser=h264 --enable-parser=aac --enable-protocol=file --enable-protocol=tcp --enable-protocol=rtmp --enable-protocol=cache --enable-protocol=pipe --enable-filter=aresample --enable-filter=allyuv --enable-filter=scale --enable-libfreetype --enable-indev=v4l2 --enable-indev=alsa --enable-omx --enable-omx-rpi --enable-encoder=h264_omx --enable-hwaccel=h264_mmal --enable-decoder=h264_mmal --enable-mmal  --enable-muxer=image2
				make -j4 && make install
	
	- 检测 ffmpeg

			ffmpeg -version
- 查看摄像头
	- 没有插摄像头
	
			# ll /dev/video*
			crw-rw----. 1 root video 81, 0 1月   1 08:00 /dev/video10
			crw-rw----. 1 root video 81, 1 1月   1 08:00 /dev/video11
			crw-rw----. 1 root video 81, 2 1月   1 08:00 /dev/video12
	- 插了摄像头

			# ll /dev/video*
			crw-rw----. 1 root video 81, 3 1月   1 08:01 /dev/video0 <- 新增视频
			crw-rw----. 1 root video 81, 4 1月   1 08:01 /dev/video1 <- 新增音频？
			crw-rw----. 1 root video 81, 0 1月   1 08:00 /dev/video10
			crw-rw----. 1 root video 81, 1 1月   1 08:00 /dev/video11
			crw-rw----. 1 root video 81, 2 1月   1 08:00 /dev/video12
- 录制
	- 不用硬件加速

			/usr/src/ffmpeg-4.2/ffmpeg -f v4l2 -r 10 -s 640x480 -i /dev/video0 -b:v 300k -c:v libx264 -an -f segment -segment_time 5 -segment_wrap 3 -segment_list_size 3 -segment_list "/data/m3u8/stream.m3u8" "/data/m3u8/stream%03d.ts"
	- 用硬件加速(仅在树莓派系统跑通)
		
			/usr/src/ffmpeg-4.2/ffmpeg -f v4l2 -r 10 -s 640x480 -i /dev/video0 -b:v 300k -c:v h264_omx -an -f segment -segment_time 5 -segment_wrap 3 -segment_list_size 3 -segment_list "/data/m3u8/stream.m3u8" "/data/m3u8/stream%03d.ts"
			
			/usr/src/ffmpeg-4.2/ffmpeg -f v4l2 -r 24 -s 1280*720 -i /dev/video0 -b:v 256k -c:v h264_omx -an -f segment -segment_time 10 -segment_wrap 10 -segment_list_size 5 -segment_list "/data/m3u8/stream.m3u8" "/data/m3u8/stream%03d.ts"
- 截图

		ffmpeg -i stream000.ts -r 2 -q:v 2 -f image2 img-%3d.jpeg
		ffmpeg -i stream000.ts -f image2 -t 0.001 -s 1280*720 image-%3d.jpg
				
## MAC 接摄像头直播(m3u8)
- 查询摄像头位置

		$ ffmpeg -f avfoundation -list_devices true -i ""
		
		#视频输入
		[AVFoundation input device @ 0x7ff9e0c250c0] AVFoundation video devices: 
		[AVFoundation input device @ 0x7ff9e0c250c0] [0] FaceTime HD Camera
		[AVFoundation input device @ 0x7ff9e0c250c0] [1] HD Pro Webcam C920 <-- 这里
		[AVFoundation input device @ 0x7ff9e0c250c0] [2] Capture screen 0
		#音频输入
		[AVFoundation input device @ 0x7ff9e0c250c0] AVFoundation audio devices:
		[AVFoundation input device @ 0x7ff9e0c250c0] [0] Built-in Microphone
		[AVFoundation input device @ 0x7ff9e0c250c0] [1] HD Pro Webcam C920
- 录制
	- 普通录制
		
			ffmpeg -f avfoundation -framerate 30 -i "1" -r 10 -s 640x480  -b:v 300k -c:v libx264 -an -f segment -segment_time 10 -segment_wrap 10 -segment_list_size 10 -segment_list "/Users/Prometheus/test/move-m3u8/m3u8/m3u8stream.m3u8" "/Users/Prometheus/test/move-m3u8/m3u8/stream%03d.ts"
			
			-f avfoundation
			-framerate
			-i
			-r 							# 帧率
			-s							# 分辨率
			-b:v						# 视频码率
			-c:v						# 硬件编码器
			-an
			-f segment				# 打开切片设置
			-segment_time				# 每个切片 10 秒
			-segment_wrap				# 切片数量
			-segment_list_size		# 缓冲文件数量
			-segment_list 			# 缓冲设置
			
	- 加密录制

- 参数说明
	- 通用选项

			-L license
			-h 帮助
			-fromats 显示可用的格式，编解码的，协议的。。。
			-f fmt 强迫采用格式fmt
			-i filename 输入文件
			-y 覆盖输出文件（即如果test.***文件已经存在的话，不经提示就覆盖掉了）
			-t duration 设置纪录时间 hh:mm:ss[.xxx]格式的记录时间也支持
			-ss position 搜索到指定的时间 [-]hh:mm:ss[.xxx]的格式也支持。使用-ss参数的作用，可以从指定时间点开始转换任务，-ss后的时间单位为秒
			-title string 设置标题（比如PSP中显示影片的标题）
			-author string 设置作者
			-copyright string 设置版权
			-comment string 设置评论
			-target type 设置目标文件类型(vcd,svcd,dvd) 所有的格式选项（比特率，编解码以及缓冲区大小）自动设置 ，只需要输入如下的就可以了：ffmpeg -i myfile.avi -target vcd vcd.mpg
			-hq 激活高质量设置
			-itsoffset offset 设置以秒为基准的时间偏移，该选项影响所有后面的输入文件。该偏移被加到输入文件的时戳，定义一个正偏移意味着相应的流被延迟了 offset秒。 [-]hh:mm:ss[.xxx]的格式也支持。
	- 视频选项

			-b bitrate 设置比特率，缺省200kb/s
			-vb bitrate set bitrate (in bits/s)
			-vframes number 设置要编码多少帧
			-r fps 设置帧频 缺省25
			-s size 设置帧大小 格式为W*H 缺省160X128.也可以直接使用简写，也认：Sqcif qcif cif 4cif 等
			-aspect aspect 设置横纵比 4:3 16:9 或 1.3333 1.7777
			-croptop size 设置顶部切除带大小 像素单位
			-cropbottom size 
			-cropleft size 
			-cropright size
			-padtop size 设置顶部补齐的大小 像素单位
			-padbottom size 
			-padleft size 
			-padright size 
			-padcolor color 设置补齐条颜色(hex,6个16进制的数，红:绿:兰排列，比如 000000代表黑色)
			-vn 不做视频记录
			-bt tolerance 设置视频码率容忍度kbit/s （固定误差）
			-maxrate bitrate设置最大视频码率容忍度 （可变误差）
			-minrate bitreate 设置最小视频码率容忍度（可变误差）
			-bufsize size 设置码率控制缓冲区大小
			-vcodec codec 强制使用codec编解码方式，如-vcodec xvid 使用xvid压缩 如果用copy表示原始编解码数据必须被拷贝。
			-sameq 使用同样视频质量作为源（VBR）
			-pass n 选择处理遍数（1或者2）。两遍编码非常有用。第一遍生成统计信息，第二遍生成精确的请求的码率
			-passlogfile file 选择两遍的纪录文件名为file

- 高级视频选项

		-g gop_size 设置图像组大小 这里设置GOP大小，也表示两个I帧之间的间隔
		-intra 仅适用帧内编码
		-qscale q 使用固定的视频量化标度(VBR) 以<q>质量为基础的VBR，取值0.01-255，约小质量越好，即qscale 4和-qscale 6，4的质量比6高 。此参数使用次数较多，实际使用时发现，qscale是种固定量化因子，设置qscale之后，前面设置的-b好像就无效了，而是自动调整了比特率。
		-qmin q 最小视频量化标度(VBR) 设定最小质量，与-qmax（设定最大质量）共用
		-qmax q 最大视频量化标度(VBR) 使用了该参数，就可以不使用qscale参数
		-qdiff q 量化标度间最大偏差 (VBR)
		-qblur blur 视频量化标度柔化(VBR)
		-qcomp compression 视频量化标度压缩(VBR)
		-rc_init_cplx complexity 一遍编码的初始复杂度
		-b_qfactor factor 在p和b帧间的qp因子
		-i_qfactor factor 在p和i帧间的qp因子
		-b_qoffset offset 在p和b帧间的qp偏差
		-i_qoffset offset 在p和i帧间的qp偏差
		-rc_eq equation 设置码率控制方程 默认tex^qComp
		-rc_override override 特定间隔下的速率控制重载
		-me method 设置运动估计的方法 可用方法有 zero phods log x1 epzs(缺省) full
		
		-dct_algo algo 设置dct的算法 可用：
		
			0 FF_DCT_AUTO 缺省的DCT
			1 FF_DCT_FASTINT
			2 FF_DCT_INT
			3 FF_DCT_MMX
			4 FF_DCT_MLIB
			5 FF_DCT_ALTIVEC
		
		-idct_algo algo 设置idct算法。可用的有：
		
			0 FF_IDCT_AUTO 缺省的IDCT
			1 FF_IDCT_INT
			2 FF_IDCT_SIMPLE
			3 FF_IDCT_SIMPLEMMX
			4 FF_IDCT_LIBMPEG2MMX
			5 FF_IDCT_PS2
			6 FF_IDCT_MLIB
			7 FF_IDCT_ARM
			8 FF_IDCT_ALTIVEC
			9 FF_IDCT_SH4
			10 FF_IDCT_SIMPLEARM
		
		-er n 设置错误残留为n
		
			1 FF_ER_CAREFULL 缺省
			2 FF_ER_COMPLIANT
			3 FF_ER_AGGRESSIVE
			4 FF_ER_VERY_AGGRESSIVE
		
		-ec bit_mask 设置错误掩蔽为bit_mask,该值为如下值的位掩码 1 FF_EC_GUESS_MVS (default=enabled) 2 FF_EC_DEBLOCK (default=enabled)
		-bf frames 使用frames个B 帧，支持mpeg1,mpeg2,mpeg4（即如果-bf 2的话，在两个非b帧中间隔的b帧数目为2，即IBBPBBPBBP结构）
		
		-mbd mode 宏块决策
		
			0 FF_MB_DECISION_SIMPLE 使用mb_cmp
			1 FF_MB_DECISION_BITS 2 FF_MB_DECISION_RD
		
		-4mv 使用4个运动矢量 仅用于mpeg4
		-part 使用数据划分 仅用于mpeg4
		-bug param 绕过没有被自动监测到编码器的问题
		-strict strictness 跟标准的严格性
		-aic 使能高级帧内编码 h263+
		-umv 使能无限运动矢量 h263+
		-deinterlace 不采用交织方法
		-interlace 强迫交织法编码 仅对mpeg2和mpeg4有效。当你的输入是交织的并且你想要保持交织以最小图像损失的时候采用该选项。可选的方法是不交织，但是损失更大
		-psnr 计算压缩帧的psnr
		-vstats 输出视频编码统计到vstats_hhmmss.log
		-vhook module 插入视频处理模块 module 包括了模块名和参数，用空格分开
		-bitexact 使用标准比特率
		-max_qdiff 视频中所有桢（包括i/b/P）的最大Q值差距
		-b_qfactor 表示i/p与B的Q值比例因子,值越大B桢劣化越严重
		-b_qoffset 表示1/p与B的Q值比例的偏移量,值越大B桢劣化越严重.如果大于0,那么下一个B的Q=前一个P的Q乘以b_quant_factor再加上offset,如果小于0,则B的Q=负的normal_Q乘以factor加上offset.
		-i_qfactor p和i的Q值比例因子,越接近1则P越优化.
		-i_qoffset p和i的Q的偏移量

	- -metadata

		表示修改视频文件的元数据，其中 `title="${i%.*}"` 表示根据文件名修改视频文件信息中的视频标题，一般播放器会优先现实元数据的内容，可以修改的元数据还有很多。
	- -c:v

		表示使用的视频编码器，即输出视频的编码类型，h264_omx 是 h264 硬件加速编码器，`c:v` 是 `-codec:v` 的简写，还可以写作 `-vcodec` ，若把 `h264_omx` 改为 `copy` 则保持原文件的视频编码；
	- -c:a

		表示使用的音频编码器，a与上面的v相对应，还可以写作 `-codec:a` or `-acodec`  用法与 `c:v` 类似，一般都是 `acc`；
	- -b:

		表示视频文件的码流，码流越大视频的质量就越好。 
		
		- `-b:v` 表示视频的码流
		- `-b:a` 表示音频的码流，默认是200Kbit/s
		
		所以要保持原视频的质量一定要设置这个参数，否则转出来的效果惨不忍睹；
	- -f
	
		表示输出视频的封装格式，可以取 mp4、flv 等值；
	- -ss: 

		表示开始时间（默认从 0:0:00 开始）；
	- -t: 

		表示持续时间（默认是原视频全长）；
	- -s 

		表示输出视频的分辨率，参数格式为w*h或w×h
	- -r:

		表示提取图像频率，一般会用于视频截图；
	- -re

		表示以原视频的帧率读取数据。FFmpeg 一般会尽可能快地读数据，流文件推送就需要加上这个参数，如果是转推直播流的话，不用加此参数；

## 注意
- m3u8 的文件中的 ts 可以不需要加后缀也可以执行，比如直接使用 md5，但是播放的时候在 `ffplay` 需要增加额外的参数 `-allowed_extensions`(就是允许全部后缀模式，否则没后缀的不让播放)

	例如
	
		ffmpeg-4.0.2-win64-static\bin\ffplay.exe -allowed_extensions ALL playlist.m3u8

		
## 参考
- [How to Convert MP4 to HLS](https://www.keycdn.com/support/how-to-convert-mp4-to-hls)
- [官方参数](https://www.ffmpeg.org/ffmpeg-formats.html#Options-4)
- [FFmpeg的使用](https://www.jianshu.com/p/7ed3be01228b)
- [树莓派使用 HLS 实现视频流直播](https://www.cnblogs.com/HintLee/p/9499420.html)
- [使用ffmpeg视频切片并加密](https://www.cnblogs.com/codeAB/p/9184266.html)
- [Installing FFMPEG for Raspberry Pi](https://www.jeffreythompson.org/blog/2014/11/13/installing-ffmpeg-for-raspberry-pi/)
- [树莓派编译安装FFmpeg(添加H.264硬件编解码器支持)](https://www.jianshu.com/p/dec9bf9cffc9)
- [ffmpeg 下载地址](https://ffmpeg.org/download.html)