#### SuSE装iscsi-target,IP:192.168.244.134 ­

#### RHEL装iscsi-initiator,IP:192.168.244.136 ­

SuSE中添加第二块2G的硬盘，名称为hdb,分一个大小为500M的区hdb1,挂载ext3，剩下的空间不动。 ­

通过iscsi连接到RHEL后，在RHEL内名称sda,全部容量分为一个区，大小也是500M,同样挂载ext3,并且写入文件若干。 ­

然后`/etc/init.d/iscsi stop`，回到SuSE中，用`fdisk /dev/hdb`重新分区(注意此时一定要把iscsi-target服务停掉，否则fdisk不让改)，大小改为2G。 ­

重起iscsi-target服务,回到RHEL中，同样用fdisk /dev/sda重新分区，把从sda1从500M划到2G,然后运行e2fsck -f /dev/sda1(此命令为检查状态),再运行`resize2fs -p /dev/sda1`(此命令为延伸nodes)后mount ­

最终发现sda1的容量从500M扩展到2G,里面的文件也无损坏 ­

注:实际发现`blockdev --rereadpt /dev/sda`（重读partition table）这个命令不用也可以，只要注意对卷进行操作时把initiator断开，把iscsi-target服务stop就可以 ­

-2008.5.25
