---
title: windows下Yii2使用redis
date: 2016-08-24 11:13:18
tags:
---
### 下载redis
windows版本请参考这里
[http://redis.io/download](http://redis.io/download)
[https://github.com/MSOpenTech/redis](https://github.com/MSOpenTech/redis)<!-- more -->
### 启动redis
```
redis-server redis.conf
```
### 安装Yii2 redis扩展
```
composer require --prefer-dist yiisoft/yii2-redis
```
### 配置使用Yii2 redis
应用配置文件中添加：
```
return [
    //....
    'components' => [
        'redis' => [
            'class' => 'yii\redis\Connection',
            'hostname' => 'localhost',
            'port' => 6379,
            'database' => 0,
        ],
    ]
];
```
使用：
```
$redis = Yii::$app->redis;
$redis->set('name','shy');
$redis->get('name');
```