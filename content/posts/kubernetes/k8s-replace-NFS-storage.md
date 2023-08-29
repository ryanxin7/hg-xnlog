```bash
# /etc/rsyncd: configuration file for rsync daemon mode

# See rsyncd.conf man page for more options.

# configuration example:
 uid = root
 gid = root
 use chroot = no
 max connections = 200
 timeout = 6000
 ignore errors
 read only = false
 list = false
 auth users = rsync_backup
 secrets file = /etc/rsync.passwd
 log file = /var/log/rsyncd.log
# pid file = /var/run/rsyncd.pid
# exclude = lost+found/
# transfer logging = yes
# timeout = 900
# ignore nonreadable = yes
# dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2
 [nfs-backup]
 comment = welcome to backup!
 path = /nfs-server

 [mysql-data-backup]
 comment = welcome to backup!
 path = /nfs-server2/tj-prod-mysql57-data-pvc-d4968732-0cf1-4395-9854-fd4581d2906d

 [mysql-log-backup]
 comment = welcome to backup!
 path = /nfs-server2/tj-prod-mysql57-log-pvc-7f831088-8828-4ab5-8e20-03ab5b048cdd
 [tree-file-backup]
 comment = welcome to backup!
 path = /nfs-server2/tj-prod-app-data-pvc-0770e2e6-70c4-4426-9a43-75e2cab94291
# [ftp]
#        path = /home/ftp
#        comment = ftp export area

```





```
nohup rsync -avzrogpP --append-verify rsync_backup@127.0.0.1::tree-file-backup /nfs-server2/tj-prod-app-data-pvc-51174db7-fd78-4448-b9fb-21b6f8577e58 > /var/log/rsync-log-treefile.log 2>&1 &
```



```bash
 tail -f /var/log/rsync-log-treefile.log
 
 
tree-file-server-for-pt/storage/high/2022/02/14/86b7f9c882e744a5b51be2c551f24a4b.jpg
         32,827 100%  244.71kB/s    0:00:00 (xfr#138425, ir-chk=149321/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b834f605614e07bccc92feaa9ab687.jpg
         35,251 100%  256.90kB/s    0:00:00 (xfr#138426, ir-chk=149320/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b8ac41b8394c529c00994427078f0f.png
          1,345 100%    9.80kB/s    0:00:00 (xfr#138427, ir-chk=149319/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b8b2ed44dc45c3ae182eccff7b4499.png
          1,722 100%   12.55kB/s    0:00:00 (xfr#138428, ir-chk=149318/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b932ab32e4472b8bb582c160885877.jpg
         34,690 100%  250.94kB/s    0:00:00 (xfr#138429, ir-chk=149317/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86b9ad6ecdec45c9846aee66df52769f.png
          1,978 100%   14.31kB/s    0:00:00 (xfr#138430, ir-chk=149316/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba0c2fd3a7462597c2c1e688a081d9.jpg
         37,435 100%  270.80kB/s    0:00:00 (xfr#138431, ir-chk=149315/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba59e758674905ba39073f647db788.png
          1,184 100%    8.50kB/s    0:00:00 (xfr#138432, ir-chk=149314/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86ba8bf9b31e464fa53b5cef03b29d52.jpg
         31,866 100%  227.15kB/s    0:00:00 (xfr#138433, ir-chk=149313/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bb4df3f5e8490f8f1eb421706828aa.jpg
         33,953 100%  240.27kB/s    0:00:00 (xfr#138434, ir-chk=149312/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bbc7d02a5f4076a6a2545f6cfc729d.jpg
         30,784 100%  216.28kB/s    0:00:00 (xfr#138435, ir-chk=149311/287796)
tree-file-server-for-pt/storage/high/2022/02/14/86bc058ff7e847dd9c6b6c7a69d551a8.jpg
```

