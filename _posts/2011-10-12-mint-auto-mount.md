---
layout: post
title: Mint启动时自动挂载win7分区
category: [Linux-Mint,mount]
comments: false
---

* content
{:toc}

###  查看已经mount例子
```bash
mount |grep /dev/sda
#结果例如显示：
/dev/sda5 on /media/yourname/program type fuseblk (rw,nosuid,nodev,relatime.....)
/dev/sda6 on /media/yourname/work type fuseblk (rw,nosuid,nodev,relatime....)
```

### 查看用户id
```bash
id yourname
#如：
uid=0(root) gid=0(root) groups=0(root)
```

### 修改mount启动文件挂载到所创建相应的文件夹
```bash
sudo gedit /etc/fstab
/dev/sda5 /media/yourname/program ntfs rw,nosuid,nodev,relatime,user_id=1000,group_id=1000,noatime 0 0
/dev/sda6 /media/yourname/work ntfs rw,nosuid,nodev,relatime,user_id=1000,group_id=1000,noatime 0 0
```

### 重启系统即可
