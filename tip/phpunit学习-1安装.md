## 环境

* vagrant(centos7)
* php7.2
* composer

## 安装

```shell
composer require --dev phpunit/phpunit
```

项目根目录中新增 tests 文件夹，并在 composer.json 文件添加 autoload-dev 关联。

```json
{
    "autoload": {
        "psr-4": {
            "app\\": "application/"
        }
    },
    "require-dev": {
        "phpunit/phpunit": "^7.5"
    },
    "autoload-dev": {
        "psr-4": {
            "test\\": "tests/"
        }
    }
}
```

## 配置

项目根目录创建 phpunit.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<phpunit backupStaticAttributes="false"
         colors="true"
         convertErrorsToExceptions="true"
         convertNoticesToExceptions="true"
         convertWarningsToExceptions="true"
         processIsolation="false"
         stopOnFailure="false"
         bootstrap="./vendor/autoload.php">
    <testsuites>
        <testsuite name="project testing">
            <directory suffix="Test.php">./tests</directory>
        </testsuite>
    </testsuites>
</phpunit>
```

## 初体验

在 tests 文件夹中新增 HelloTest.php

```php
<?php

namespace test;

class HelloTest extends \PHPUnit\Framework\TestCase
{
    public function testEcho()
    {
        $msg = 'hello world !!!';

        $this->assertNotEmpty($msg);
    }
}
```

运行查看结果（需要在 Linux 中才可以运行）

```shell
./vendor/bin/phpunit
```
