# kvm虚拟机ip查询

```shell
nmap -sP 10.1.12.0/24 | grep -A 1 Virtual
```



### 符合查询

```shell
#!/bin/bash

NAMES=`virsh list --name`

for name in $NAMES
do
        echo $name
        mac=`virsh dumpxml $name |grep "<mac address" | awk -F "['']" '{print $2}'`
        echo $mac
        ip=`nmap -sP 10.1.12.0/24 | grep -A 1 -i $mac`
        echo $ip
done	
```

