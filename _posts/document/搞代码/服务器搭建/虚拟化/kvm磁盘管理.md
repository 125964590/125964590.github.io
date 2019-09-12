# kvm磁盘管理

创建磁盘的时候注意选择磁盘类型,需要和现有的磁盘类型保持一致

## 主机磁盘相关操作

### 查询虚拟机状态

> ```shell
> $ virsh list
> ```

### 查看虚拟机磁盘挂在情况

> ```shell
> $ virsh domblklist jbzm-mac
> ```

### 给虚拟机挂在磁盘

>```shell
>$ virsh attach-disk jbzm-mac /home/jbzm/kvm_iso/mac.img vdc
>```

### 给当前的磁盘添加容量

> ```shell
> $ qemu-img resize /home/jbzm/kvm_iso/jbzm-mac.img +200G
> ```

### 查看磁盘详细信息

> ```shell
> $ qemu-img info /home/jbzm/kvm_iso/jbzm-mac.img
> ```

### 解除磁盘挂载

> ```shell
> $ virsh detach-disk jbzm-mac vdc
> ```

## 虚拟机磁盘挂载

在主机创建好虚拟磁盘之后需要进行挂载

### 查看磁盘

```shell
$ fdisk -l
```

### 对磁盘进行格式化

```shell
$ mkfs.ext4 /dev/vdb
```

### 完成挂载

```shell
$ mount /dev/vdb /data
```

### 配置开机自动挂载

```shell
$ vim /etc/fstab 
/dev/vdb	/data		ext4	defaults	0 0
```

### 固定xml配置

由于给虚拟机添加了磁盘导致xml发生变更,所以我们需要将内存中的xml虚拟机配置同步到文件中

```shell
$ virsh dumpxml strang >/etc/libvirt/qemu/strang.xml
$ virsh define /etc/libvirt/qemu/strang.xml
```

默认情况下kvm的xml文件回报存在**/etc/libvirt/qemu/{server_name}.xml

