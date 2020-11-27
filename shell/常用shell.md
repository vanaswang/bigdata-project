# 创建用户
```shell
# 创建用户需使用root登录，设置部署用户名，请自行修改，后面以dolphinscheduler为例
useradd admin;

# 设置用户密码，请自行修改，后面以dolphinscheduler123为例
echo "admin" | passwd --stdin admin

# 配置sudo免密
echo 'admin  ALL=(ALL)  NOPASSWD: NOPASSWD: ALL' >> /etc/sudoers
sed -i 's/Defaults    requirett/#Defaults    requirett/g' /etc/sudoers

```

```
注意：
 - 因为是以 sudo -u {linux-user} 切换不同linux用户的方式来实现多租户运行作业，所以部署用户需要有 sudo 权限，而且是免密的。
 - 如果发现/etc/sudoers文件中有"Default requiretty"这行，也请注释掉
 - 如果用到资源上传的话，还需要在`HDFS或者MinIO`上给该部署用户分配读写的权限
```



# 集群同步脚本

```shell
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname=`basename $p1`
echo fname=$fname

#3 获取上级目录到绝对路径
pdir=`cd -P $(dirname $p1); pwd`
echo pdir=$pdir

#4 获取当前用户名称
user=`whoami`

#5 循环
for host in node001 node002 node003 
do
        echo ------------------- $host --------------
        rsync -av $pdir/$fname $user@$host:$pdir
done
```





# 集群整体操作脚本

```shell
#! /bin/bash

for host in node001 node002 node003 
do
        echo --------- $host ----------
        ssh $host "source /etc/profile ; $*"
done
```

