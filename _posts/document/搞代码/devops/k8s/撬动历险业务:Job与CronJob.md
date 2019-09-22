# 撬动历险业务:Job与CronJob

job是一种只会执行一次的pod,restartPolicy在Job对象里只允许被设置为Never和OnFailure;然而在Deployment对象里,restartPolicy则只允许被设置为Always.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: resouer/ubuntu-bc 
        command: ["sh", "-c", "echo 'scale=10000; 4*a(1)' | bc -l "]
      restartPolicy: Never
  backoffLimit: 4
```

## 处理失败的业务

如果定义**resetarPolicy=Never**,那么离线作业失败之后,Job Controller就不会去尝试创建新的Pod,但是会**不断的创建一个新的Pod**,默认情况下会试6次

如果定义**restartPolicy=OnFailurte**,那么离线作业失败后,Job Controller就不会去尝试创建新的Pod,但是他会不断尝试重启Pod里面的容器

在job控制器里可以设置两个字段,最多的创建次数以及最大的运行时间

```yaml
spec:
 backoffLimit: 5
 activeDeadlineSeconds: 100
```

## 并行控制Job

通过**parallelism**和**completions**兰控制并发数量以及最小完成数

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  parallelism: 2
  completions: 4
  template:
    spec:
      containers:
      - name: pi
        image: resouer/ubuntu-bc
        command: ["sh", "-c", "echo 'scale=5000; 4*a(1)' | bc -l "]
      restartPolicy: Never
  backoffLimit: 4
```
可以看到同一时间存在两个,但是最大完成数是四个
```shell
nginx-set-59bd9cff8-tcw8g           1/1     Running             0          5d18h
pi-limit-8qmql                      0/1     ContainerCreating   0          4s
pi-limit-bqd8z                      0/1     ContainerCreating   0          4s
web-0                               1/1     Running             0          4d23h
```

