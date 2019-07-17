# etcd故障
## 故障一
通过web界面删除”部署“，没有反应
## 故障分析
- 查看 etcd 的监控 grafana
	- the number of leader changes seen 和 the total number of failed proposals seen 

		结果都是400多
	- disk sync duration 

		获取的 etcd_disk_wal_fsync_duration_seconds_bucket 在一段时间点异常抖动
	- 客户端传输

		客户端传输同磁盘响应一起抖动	
- 网上查询(https://zhiliao.h3c.com/Theme/details/39865)

	查看IO延时落到8ms以内的有99.95%（见下标红部分292001/292136得出百分比），说明整个etcd集群是健康的，就不会影响到系统业务的运行。

		[root@cloudos18803 ~]# curl http://127.0.0.1:2379/metrics | grep fsync
		
		  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
		
		                                 Dload  Upload   Total   Spent    Left  Speed
		
		100 35236  100 352# HELP etcd_wal_fsync_durations_seconds The latency distributions of fsync called by wal.
		
		36# TYPE etcd_wal_fsync_durations_seconds histogram
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.001"} 291808
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.002"} 291899
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.004"} 291947
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.008"} 292001   <----这里的值是
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.016"} 292068
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.032"} 292117
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.064"} 292120
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.128"} 292125
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.256"} 292127
		
		  etcd_wal_fsync_durations_seconds_bucket{le="0.512"} 292131
		
		  etcd_wal_fsync_durations_seconds_bucket{le="1.024"} 292135
		
		  etcd_wal_fsync_durations_seconds_bucket{le="2.048"} 292136
		
		  etcd_wal_fsync_durations_seconds_bucket{le="4.096"} 292136
		
		  etcd_wal_fsync_durations_seconds_bucket{le="8.192"} 292136
		
		  etcd_wal_fsync_durations_seconds_bucket{le="+Inf"} 292136    <----这里的值是
		
		  etcd_wal_fsync_durations_seconds_sum 104.93636339099864
		
		  etcd_wal_fsync_durations_seconds_count 292136

	
	