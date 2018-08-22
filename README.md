# ucloud_disk_fio
## Ucloud 云平台 磁盘压测
### 1.安装压测工具
```bash
yum -y install fio
```

### 2.编写脚本进行压测
```bash
disk_fio.sh:
#!/bin/bash

NUMS="100 200 300 400"

for SIZE in $NUMS;do
  LOGFILE="disk_iops_$SIZE"

  echo "=================================" | tee $LOGFILE
  echo "测试 ${SIZE}G 磁盘IOPS:" | tee -a $LOGFILE
  
  echo | tee -a $LOGFILE
  
  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb:随机写:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=randwrite -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=suijiduxie-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb:随机读:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=randread -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=suijidu-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb:顺序写:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=write -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=shunxuxie-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb:顺序读:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=read -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=shunxudu-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  #echo "---------------------------------" | tee -a $LOGFILE
  #echo "测试/dev/vdb:同步IO随机写:" | tee -a $LOGFILE
  #fio --filename=/data1/test_file -direct=1 -rw=randwrite -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=psync -group_reporting -name=tongbusuijiduxie-vdb | tee -a $LOGFILE
  #echo "---------------------------------" | tee -a $LOGFILE

  #echo | tee -a $LOGFILE

  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb: 4k,100%读写:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=randrw --refill_buffers --norandommap --randrepeat=0 --rwmixread=100 -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=4k100-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  echo "---------------------------------" | tee -a $LOGFILE
  echo "测试/dev/vdb: 4k,70%读,30%写:" | tee -a $LOGFILE
  fio --filename=/data1/test_file -direct=1 -rw=randrw --refill_buffers --norandommap --randrepeat=0 --rwmixread=70 -bs=4k -size=${SIZE}G -numjobs=4 -runtime=1200 -iodepth=4 -ioengine=libaio -group_reporting -name=4k7030-vdb | tee -a $LOGFILE
  echo "---------------------------------" | tee -a $LOGFILE

  echo | tee -a $LOGFILE

  echo "测试结束!" | tee -a $LOGFILE
  echo "=================================" | tee -a $LOGFILE
done
```

### 3.压测服务器类型
```text
所有磁盘都是400G,软RAID是由4块100G磁盘组成.

普通云主机 普通云盘
普通云主机 云SSD
普通云主机 软RAID0
高性能云主机 本地SSD
高性能云主机 云SSD
高性能云主机 软RAID0
```

### 4.压测结果
```text
普通云主机:
read iops： 云盘ssd 好于 普通云盘 23.17 倍；云盘ssd 好于 云盘ssd raid0  13 倍； 云盘ssd raid0 好于 普通云盘 1.7 倍
write iops: 云盘ssd 好于 普通云盘 23.09 倍；云盘ssd 好于 云盘ssd raid0  13 倍； 云盘ssd raid0 好于 普通云盘 1.7 倍
 
高性能云主机:
read iops： 本地ssd 好于 云盘ssd 2.22 倍；本地ssd 好于 云盘ssd raid0 1.77  倍； 云盘ssd raid0 好于 云盘ssd 1.25 倍
write iops: 本地ssd 好于 云盘ssd 2.22 倍；本地ssd 好于 云盘ssd raid0 1.78  倍； 云盘ssd raid0 好于 云盘ssd 1.24 倍
 
高性能云主机 本地ssd 和 普通云主机 普通云盘 比较:
read iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 52.59 倍
write iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 52.66 倍
 
高性能云主机 本地ssd 和 普通云主机 普通云盘 RAID0 比较:
read iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 RAID0 29.69 倍
write iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 RAID0 29.71 倍
 
高性能云主机 本地ssd 和 普通云主机 云盘ssd 比较:
read iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 云盘ssd 2.26 倍
write iops: 高性能云主机 本地ssd 好于 普通云主机 普通云盘 云盘ssd 2.28 倍
 
高性能云主机 云盘ssd RAID0 和 普通云主机 普通云盘 比较:
read iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 29.57 倍
write iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 29.53  倍
 
高性能云主机 云盘ssd RAID0 和  普通云主机 普通云盘 RAID0 比较:
read iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 RAID0 16.69 倍
write iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 RAID0 16.66 倍
 
高性能云主机 云盘ssd RAID0 和 普通云主机 云盘ssd 比较:
read iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 云盘ssd 1.27 倍
write iops: 高性能云主机 云盘ssd RAID0 好于 普通云主机 普通云盘 云盘ssd 1.27 倍
 
高性能云主机 云盘ssd 和 普通云主机 普通云盘 比较:
read iops: 高性能云主机 云盘ssd 好于 普通云主机 普通云盘 23.62 倍
write iops: 高性能云主机 云盘ssd 好于 普通云主机 普通云盘 23.65 倍
 
高性能云主机 云盘ssd 和  普通云主机 普通云盘 RAID0 比较:
read iops: 高性能云主机 云盘ssd 好于 普通云主机 普通云盘 RAID0 13.33 倍
write iops: 高性能云主机 云盘ssd 好于 普通云主机 普通云盘 RAID0 13.34  倍
 
高性能云主机 云盘ssd 和 普通云主机 云盘ssd 比较:
read iops: 高性能云主机 云盘ssd 好于 普通云主机 云盘ssd 1.01 倍
write iops: 高性能云主机 云盘ssd 好于 普通云主机 云盘ssd 1.02  倍
```

