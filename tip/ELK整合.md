# ELK 整合

## 环境说明

* windows 7
* java 1.8
* elasticSearch
* logstash
* kibana
* reids
* vagrant
  * logstash
  * java 1.8

## ELK 整合

## 分词器使用

## 基本使用

index(索引) type(类型) document(文档)

可以以表词来理解

index(数据库) type(表) document(表数据)

建表

POST /xjw/

POST /xjw/brand/_mapping
{
    "properties": {
        "id": {"type": "long"},
        "name": {"type": "string"},
        "created": {"type": "long"},
    }
}

字段类型

分词器设置


增

POST /xjw/brand/1
{
    "id": 1,
    "name": "黑人牙膏",
    "created": 8761231231
}

改

PUT /xjw/brand/1
{
    "id": 1,
    "name": "黑妹牙膏",
    "created": 8761231231
}

删

DELETE /xjw/brand/1

查

GET /xjw/brand/1

高级查询

模糊搜索

多条件搜索

过滤指定条件数据

分页

# nssm

## 说明

## 使用 nssm 设置 Logstash 和 Kibana 为 windows 服务
[参考文章](https://www.alibabacloud.com/help/zh/doc-detail/49021.htm)



## FQA

### 服务 CMD 命令操作

```

# 启动服务
net start elasticsearch

# 关闭服务
net stop elasticsearch

# 删除服务
sc delete elasticsearch

```

重点

1、导入数据库数据加速
    多线程
    es批量导入
        导入速度十分可观，更快更平稳了


2、相关数据使用观察者模式进行及时更新