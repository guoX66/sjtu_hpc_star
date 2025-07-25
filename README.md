# sjtu HPC star-ccm多主机并行方案——2025/7/25

## 1 前期准备

参考官用户手册：[STAR-CCM+ - 上海交大超算平台用户手册](https://docs.hpc.sjtu.edu.cn/app/engineeringscience/star-ccm.html)

```
nano ~/.ssh/known_hosts
```

清空其中内容，ctrl+x退出，并按y保存。

执行以下命令生成公私钥并复制到服务器：

```
ssh-keygen -t rsa
ssh-copy-id -i ~/.ssh/id_rsa localhost
```

 编辑 ~/.ssh/config 

```
nano ~/.ssh/config
```

在头几行增加如下两行内容：

```
StrictHostKeyChecking no
UserKnownHostsFile=/dev/null
```

修改config权限：

```
chmod 600 ~/.ssh/config
```

## 2 脚本示例

思源一号集群上Slurm 脚本（#SBATCH  -N 2 表示申请两个节点）

```
#!/bin/bash

#SBATCH --job-name=v7
#SBATCH --partition=64c512g
#SBATCH -N 2
#SBATCH --ntasks-per-node=64
#SBATCH --output=%j.out
#SBATCH --error=%j.err

#source /usr/share/Modules/init/bash
module purge
module load intel intel-mpi

ulimit -s unlimited
ulimit -l unlimited

scontrol show hostname $SLURM_JOB_NODELIST | \
  while read node; do 
    echo "${node}:${SLURM_NTASKS_PER_NODE}" 
  done > machinefile

starccm+ \
    -batch run \
    -batch-report \
    -power -rsh ssh \
    -np $SLURM_NTASKS \
    -machinefile machinefile \
/dssg/home/acct-naowbr/guox66/star_projects/v7_r2.sim
```

## 3 注意事项

### 3.1 ssh免密问题

根据用户手册的介绍（[通过 SSH 登录集群 - 上海交大超算平台用户手册](https://docs.hpc.sjtu.edu.cn/login/sshlogin.html)），经过安全升级后，超算平台不再支持传统的公私钥免密，将公钥写入个人家目录的 `~/.ssh/authorized_keys` **不会** 再赋予免密登录的权限，因此可能需要前往 [超算账号管理平台](https://my.hpc.sjtu.edu.cn/) 申请免密证书。

在[超算账号管理平台](https://my.hpc.sjtu.edu.cn/)中登录以后，点击右上角用户名-个人主页-申请免密，选择上传公钥的方法，将步骤1中生成的~/.ssh/id_rsa.pub下载到本地后再上传文件至平台

![image](https://github.com/guoX66/sjtu_hpc_star/blob/main/assets/1.png)

![image](https://github.com/guoX66/sjtu_hpc_star/blob/main/assets/2.png)

点击生成证书，获得认证后将下载的 id_rsa-cert.pub 文件传到 ~/.ssh 路径下

### 3.2 star-ccm版本问题

本方案使用的star-ccm版本是 19.06.009-R8 ，如仍有问题可尝试更换star-ccm版本：

通过网盘分享的文件：Siemens.STAR-CCM+2410.1_19.06.009.R8.Double.Precision.Linux64.tar.gz
链接: https://pan.baidu.com/s/1YtzchxFspsy9GagxIaWUeA 提取码: 756b 
--来自百度网盘超级会员v6的分享


