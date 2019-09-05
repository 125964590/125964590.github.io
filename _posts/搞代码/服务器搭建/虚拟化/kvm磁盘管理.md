# kvm磁盘管理

查询虚拟机状态

> ```shell
> $ virsh list
> ```

查看虚拟机磁盘挂在情况

> ```shell
> $ virsh domblklist jbzm-mac
> ```

给虚拟机挂在磁盘

>```shell
>$ virsh attach-disk jbzm-mac /home/jbzm/kvm_iso/mac.img vdc
>```

给当前的磁盘添加容量

> ```shell
> $ qemu-img resize /home/jbzm/kvm_iso/jbzm-mac.img +200G
> ```

查看磁盘详细信息

> ```shell
> $ qemu-img info /home/jbzm/kvm_iso/jbzm-mac.img
> ```

解除磁盘挂载

> ```shell
> $ virsh detach-disk jbzm-mac vdc
> ```

