# Event Scheduler

## 前提

在工作中会遇到涉及数据库的定时任务，对于和业务强相关，逻辑比较复杂的定时任务我们一般放在服务端去实现。而对于涉及业务较少，逻辑简单的定时任务我们可以使用数据库自带的event scheduler去实现。下面介绍关于MySQL Event Scheduler的使用

**注意：以下配置基于MySQL 5.7**

## 配置

MySQL在创建event scheduler之前需要开启event scheduler配置，event是通过MySQL内部的scheduler线程去执行的，所以我们讨论event scheduler实际是在讨论scheduler线程。event scheduler有下面三种状态：

- OFF：event scheduler处于停止状态，是默认的event scheduler状态。当处于这种状态时无法通过SHOW PROCESSLIST查询到event scheduler的线程。可以设置为ON。

- ON：event scheduler处于开启状态，代表event scheduler的线程正在执行。当处于这种状态时无法通过SHOW PROCESSLIST查询到event scheduler的线程。可以设置为OFF。

  ```mysql
  mysql> SHOW PROCESSLIST\G
  *************************** 1. row ***************************
       Id: 1
     User: root
     Host: localhost
       db: NULL
  Command: Query
     Time: 0
    State: NULL
     Info: show processlist
  *************************** 2. row ***************************
       Id: 2
     User: event_scheduler
     Host: localhost
       db: NULL
  Command: Daemon
     Time: 3
    State: Waiting for next activation
     Info: NULL
  2 rows in set (0.00 sec)
  ```

- DISABLED：event scheduler处于禁用状态，和OFF类似，event scheduler停止并且无法通过SHOW PROCESSLIST查询。当处于这种状态下时无法设置为其他状态。

由于event scheduler是全局的环境变量，所以可以通过SHOW VARIABLES或者SELECT来查询event scheduler的状态。当event scheduler状态不是DISABLED时，也可以通过SET GLOBAL来设置event scheduler的ON、OFF状态：

```mysql
//查询event scheduler状态
SELECT @@GLOBAL.event_scheduler
SHOW VARIABLES LIKE "%event_scheduler%"

//设置event scheduler状态
SET GLOBAL event_scheduler = ON;
SET @@GLOBAL.event_scheduler = ON;
SET GLOBAL event_scheduler = 1;
SET @@GLOBAL.event_scheduler = 1;

SET GLOBAL event_scheduler = OFF;
SET @@GLOBAL.event_scheduler = OFF;
SET GLOBAL event_scheduler = 0;
SET @@GLOBAL.event_scheduler = 0;
```

上面设置的状态会随着MySQL的重启而恢复到原来的默认状态，所以如果需要开机自动开启event scheduler，需要设置mysql.ini：

```
[mysqld]
event_scheduler = ON
```

## Event语法

### create event

 创建Event的语法如下：

```mysql
CREATE
    [DEFINER = user]
    EVENT
    [IF NOT EXISTS]
    event_name
    ON SCHEDULE schedule
    [ON COMPLETION [NOT] PRESERVE]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT 'string']
    DO event_body;

schedule: {
    AT timestamp [+ INTERVAL interval] ...
  | EVERY interval
    [STARTS timestamp [+ INTERVAL interval] ...]
    [ENDS timestamp [+ INTERVAL interval] ...]
}

interval:
    quantity {YEAR | QUARTER | MONTH | DAY | HOUR | MINUTE |
              WEEK | SECOND | YEAR_MONTH | DAY_HOUR | DAY_MINUTE |
              DAY_SECOND | HOUR_MINUTE | HOUR_SECOND | MINUTE_SECOND}
```

Create Event主要由以下三个语句组成：

event_name：数据库中唯一事件标识，长度限制为64字符。

scheduler：事件执行的时间和频率，可以设置单次和重复。

event_body：事件执行的内容，可以是对应的SQL语句或者存储过程。

下面是最简单的一个event事件：

```mysql
CREATE EVENT myevent
    ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
    DO
      UPDATE myschema.mytable SET mycol = mycol + 1;
```

创建一个myevent的事件，当前时间延迟一个小时触发，事件内容是更新myschema.mytable表的mycol字段自增1。

具体的语句解析：

- DEFINER = user：指定event执行时的MySQL账号，user可以是'username'@'host_name'、CURRENT USER 或者CURRENT USER()，未指定情况下默认使用CURRENT_USER。

- IF NOT EXISTS：和使用CREATE TABLE效果一样，如果存在相同event name的event，不会创建，也不会报错，只会有一个警告。

- ON SCHEDULE schedule：ON SCHEDULE代表event执行的事件和频率，schedule语句有两种形式：

  - AT timestamp：是单次执行，代表event在指定时间执行一次。timestamp可以是日期，时间或者任意能够转化为时间的表达式。当timestamp的时间过期时，create event会出现显示警告。timestamp除了使用指定的时间以外，还可以使用\+ INTERVAL  interval来表示相对时间。比如要创建当前时间后3周2天执行，可以使用CURRENT_TIMESTAMP + INTERVAL 3 WEEK + INTERVAL 2 DAY。

  - EVERY：是重复执行，代表event在指定时间内执行的频率。EVERY后面直接跟interval，EVERY 6 WEEK代表每6周一个重复。EVERY后可以使用START语句来实现更加精确和灵活的执行。START timstamp \+ INTERVAL  interval代表在一个重复周期内的指定时间执行。

    比如EVERY 3 MONTH STARTS CURRENT_TIMESTAMP + INTERVAL 1 WEEK代表每3周重复，从当前开始延后一周执行。

    或者EVERY 2 WEEK STARTS CURRENT_TIMESTAMP + INTERVAL '6:15' HOUR_MINUTE代表每2周重复，在当前开始延后6小时15分执行。

    END和START类似，代表重复停止的时间。

    EVERY 12 HOUR STARTS CURRENT_TIMESTAMP + INTERVAL 30 MINUTE ENDS CURRENT_TIMESTAMP + INTERVAL 4 WEEK代表每12小时重复，在当前时间延后30分钟执行，持续4周。

- ON COMPLETION [NOT] PRESERVE：指定event过期时是否删除，默认为ON COMPLETION NOT PRESERVE。

- ENABLE | DISABLE | DISABLE ON SLAVE：代表event的状态，默认为ENABLE，可以设置为DISABLE不执行，DISABLE ON SLAVE代表在副本上不执行。可以结合ALTER EVENT来实现event的开启关闭。

- COMMENT：event的注释，不超过个字符。

- DO：event具体执行内容，可以是SQL或者存储过程。

因此完整的Create Event的语句如下：

```mysql
CREATE
    DEFINER = user
    EVENT
    IF NOT EXISTS
    myevent
    ON SCHEDULE AT CURRENT_TIMESTAMP + 1 HOUR
    ON COMPLETION PRESERVE
    ENABLE
    COMMENT 'DELETE'
    DO 
    	DELETE FROM site_activity.sessions;
```

创建一个在当前时间延后一个小时执行的event，内容是删除site_activity.sessions，在event过期后不删除。

### ALTER EVENT

编辑Event的语法如下：

```
ALTER
    [DEFINER = user]
    EVENT event_name
    [ON SCHEDULE schedule]
    [ON COMPLETION [NOT] PRESERVE]
    [RENAME TO new_event_name]
    [ENABLE | DISABLE | DISABLE ON SLAVE]
    [COMMENT 'string']
    [DO event_body]
```

ALTER EVENT的语法和CREATE EVENT类似，可以使用ALTER EVENT编辑部分event内容。

## Reference

https://dev.mysql.com/doc/refman/5.7/en/events-configuration.html

https://dev.mysql.com/doc/refman/5.7/en/create-event.html

https://dev.mysql.com/doc/refman/5.7/en/alter-event.html



