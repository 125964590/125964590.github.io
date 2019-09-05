# mysql导出到es

## 排错流程

1. 因为数据是通过爬虫获取来的,所以先查看数据库中的数据是否有断层,如果存在断层,说明数据来源改写了数据的借口.需要重新写爬虫,但是结果是数据库中的数据是最新的爬虫没有问题.
2. checkout elasticsearch
通过爬虫爬取的数据最终会存放到es中,到es中进行检查,发现es中的数据缺失,也就是说错误出现在从mysql到es的过程上.

## 解决错误
由于找到了错误的原因现在开始寻找解决错误的方案.
mysq中的数据到es是通过logstash传递的,this logstash is a pipeline ,it every timing  scan data ,发现新增的数据会上传到es中.

> ### 之前的更改
> 因为在上一个版本的时候我对logstash进行了一些更改,通过logstash进行了日志收集,将服务上的日志进行收集,之后统一处理.所以定义了多个type来保证输出到es中可以有独立的标识.

The pipeline had bug,this type for jdbc 不能找到,相应的属性
> 其实是因为,我是用的是jdbc的插件,这个并不会吧自己的变量加入到logsatsh的环境变量中,无法在其他的地方进行引用.

通过观察logstash的日志信息,打印出来的sql的id下标和数据库中的最大下标一直,这是需要考虑更改logstash的外部环境变量.
> #### 外部环境变量
> logstash在使用jdbc插件的时候可以通过定义外部环境变量的方式,把每次查询的id保存在外部的文件中.

将外部环境变量的id改成和es上数据相同的id+1就可以了

## 添加标识
在logstash中所有的input插件都提供一下字段
所有输入插件都支持以下配置选项


| 设置          | 输入类型 | 需要 |
| ------------- | -------- | ---- |
| add_field     | hash     | no   |
| codec         | codec    | no   |
| enable_metric | boolean  | no   |
| id            | string   | no   |
| tages         | array    | no   |
| type          | string   | no   |

通过设置元数据来进行判断相应的存储索引


## 这里附上修改之后的配置文件
```
input {
 stdin { }
    #jdbc config
    jdbc {
        # 该字段为通用字段,在正常情况都是用来做标注
        type => "galaxy"

        jdbc_connection_string => "jdbc:mysql://172.17.70.225:3306/galaxy"

        jdbc_user => "galaxy"

        jdbc_password => "galaxy@HuaTu2018"

        jdbc_driver_library => "/usr/mysql/mysql-connector-java-5.1.45-bin.jar"

        jdbc_driver_class => "com.mysql.jdbc.Driver"

        jdbc_paging_enabled => "true"

        jdbc_page_size => "50000"
        statement => "SELECT fb.id AS id, course_id, fb.title AS title, fb.price AS price, fb.student_limit AS student_limit, fb.student_count-fb.previous_count AS previous_count, fb.previours_price AS previours_price, fb.current_cycle_income AS current_cycle_income, fb.create_time AS create_time, fc.course_type AS course_type, fc.status AS status, fc.sale_status AS sale_status FROM fb_course_snapshot fb LEFT JOIN fb_course fc ON fb.course_id = fc.id WHERE fb.id>:sql_last_value"
        use_column_value => true
        tracking_column => "id"
        schedule => "* * * * *"
        last_run_metadata_path => "/home/es/.logstash_jdbc_last_run"
    }
 }
filter {
    date {
      match => ["create_time", "yyyy-MM-dd HH:mm:ss,SSS","ISO8601"]
      target => "@timestamp"
      locale => "cn"
    }
}
output {
  if [type] == "galaxy" {
    # 输出到控制台
     stdout { 
      codec => rubydebug
    }
    elasticsearch {
        hosts => "172.17.70.224:9200"
        index => "fb_goods"
        document_type => "fb_orders"
        document_id => "%{id}"
    }
  }
}
```