## 说明

logstash 是一个日志收集工具，把不同格式的日志内容取出并以不同方式进行保存。

因为直接用 logstash 写到目标存在丢包的情况，所以先把日志内容全部存放在 redis 中，然后再由一个 logstash 写入到最终目标中。

我写入 elasticSearch 为例（下面简称 ES）。

## 配置

* 两个装有 logstash 的系统
* redis
* elastSearch

nginx 日志提取方式

```
input {
    file {
        type => "nginx-access"
        path => "/usr/local/nginx/logs/access.log"
    }
    file {
        type => "nginx-error"
        path => "/usr/local/nginx/logs/error.log"
    }
}

filter {
}

output {
    stdout {
        codec => rubydebug
    }
    redis {
        password => "123456"
        data_type => "list"
        key => "logstash-%{type}"
    }
}
```

存储到 elastSearch

```
input {
    redis {
        type => "nginx-access"
        host => "192.168.33.10"
        port => "6379"
        password => "123456"
        data_type => "list"
        key => "logstash-nginx-access"
    }
    redis {
        type => "nginx-error"
        host => "192.168.33.10"
        port => "6379"
        password => "123456"
        data_type => "list"
        key => "logstash-nginx-error"
    }
    redis {
        type => "php-error"
        host => "192.168.33.10"
        port => "6379"
        password => "123456"
        data_type => "list"
        key => "logstash-php-error"
    }
}

output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
    }
}
```

## logstash 尝试

1. 简化写法

收集日志到 redis

input 里设置 type，output 直接存在到一个 redis 键中。

写入 ES

input 里只有一个为源，但是 output 里直接使用 logstash-%{type}-%{+YYYY.MM.dd}

```
# 收集日志到 redis
input {
    file {
        type => "nginx-access"
        path => "/usr/local/nginx/logs/access.log"
    }
    file {
        type => "nginx-error"
        path => "/usr/local/nginx/logs/error.log"
    }
    file {
        type => "php-error"
        path => "/usr/local/php71/var/log/error.log"
    }
}

output {
    stdout {
        codec => rubydebug
    }
    redis {
        password => "123456"
        data_type => "list"
        key => "logstash-log"
    }
}

```

```
# 导入 ES
input {
    redis {
        host => "192.168.33.10"
        port => "6379"
        password => "123456"
        data_type => "list"
        key => "logstash-log"
    }
}

output {
    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts => ["127.0.0.1:9200"]
        index => "logstash-%{type}-%{+YYYY.MM.dd}"
    }
}
```

思路：因为数据里的 type 会把 input.type 值进行覆盖。所以才想到上面的办法
状态：已验证，可行



## 调试方法

ouput 里都使用 stdout 进行直接输出。这样可以知道有没有被触发。如果说旧日志已经被读完了，又没有办法让新日志进来。可以进入日志文件中复制一行日志，再保存就可以触发 logstash 的监听了

## 总结

不得吐槽 logstash 的配置文件修改后就得关闭再开。因为电脑性能不太好，每次都等到我心焦。还有就是每台服务器都需要装 logstash 以及依赖的 java 感觉服务器一多维护上会比较麻烦，还有软件占用性能总合有点高（用的上 ELK 的公司一般不会在意这些小费用的），日志内容匹配有些麻烦，有很多插件供你选择，但还是需要多多练习才能很好的掌握。

用 redis 统一收集日志时不要用带有变量的名来当 redis 键。在 logstash 读取 redis 日志时需要直接指定的键才能获取日志，每当键改变了，logstash 的配置需要跟着改变。

从 redis 保存到 ES 时，output 的 key 参数带上时间变量来当 es 的文档名。以后就可以按时间来查看什么时候收集了那些日志数据。

定义 type 时最好进行分层。如 nginx 的 access 日志和 error 日志。我用 logstash-nginx-access 和 logstash-nginx-error 来命名。使用 kibana 读取时可以使用正则匹配来查看 全部日志、error 日志、access 日志。

## FQA:

### 保存到 ES 时，input 里定义的 type，在 ouput 里用 if 判断不了。

这是因为保存的数据里也存在 type。所以 定义的 type 会被覆盖。推荐使用数据里的 type 进行判断（不覆盖但保留数据中的 type 不变的方法还没有找到）。

### 保存到 ES 时，input.redis 里的 key 参数使用不了正则

我学习的时候太过于意识化了，以为 redis 和 file 一样都可以进行正则匹配。经过多次试错，判断这里是要指定一个 reids.key 的，而且每次读取完的 redis 里的日志，都会清空处理。所以不用担心 redis 日志过多而影响 redis 性能。
