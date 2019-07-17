# iops性能测试

## 挂载磁盘

```
mkfs.xfs /dev/vdb
mount /dev/vdb /mnt/yunpa1
```

## 安装fio

```
yum install fio
```

## 测试

### 随机读55:随机写45

测试命令

```
fio --directory=/mnt/yunpan1 --name fio_test_file --direct=1 --randrepeat=0 --rw=randrw -rwmixread=55  --bs=4k --size=512m --numjobs=4 --time_based --runtime=60 --group_reporting
```

主要结果
 
 |iops|带宽(KiB/s)
-----|-----|-----
读| 899 | 3597
写| 735 |2943

 |iops|带宽(KiB/s)
-----|-----|-----
读| 987 | 3949
写| 799 |3200

### 随机读96：随机写4

测试命令

```
fio --directory=/mnt/yunpan1 --name fio_test_file --direct=1 --randrepeat=0 --rw=randrw -rwmixread=96  --bs=4k --size=512m --numjobs=4 --time_based --runtime=60 --group_reporting
```

主要结果

 |iops|带宽(KiB/s)
-----|-----|-----
读| 1232 | 4930
写| 50 | 203

 |iops|带宽(KiB/s)
-----|-----|-----
读| 1801 | 7205
写| 75 | 303


 