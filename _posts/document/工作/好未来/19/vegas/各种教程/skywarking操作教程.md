# skywarking 操作教程


## 清理磁盘
该虚拟机之前的挂载磁盘用来接收mysql的数据,这里先把磁盘格式化后重新挂载

```
# 查看磁盘状态
root@test-skywarking:/# df -Th
# 解除挂载
root@test-skywarking:/# umount /dev/vdc1
# 格式化磁盘
root@test-skywarking:/# mkfs -t xfs -f /dev/vdc1
# 重新挂载
root@test-skywarking:/# mount /dev/vdc1 /home/
# 检查状态
root@test-skywarking:/# df -Th
```

## 安装skywarking
- [官方文档]()

**下面这个一定要注意,说多了都是泪库j
![](http://ww4.sinaimg.cn/large/006tNc79ly1g5r5zi7ffzj30pl032dg8.jpg)