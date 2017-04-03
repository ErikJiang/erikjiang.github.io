title: "Node 任务定时器的使用"
date: 2017-04-02 14:12:24
categories: "技术" 
tags: [node,schedule,cron] 
---

## 1. 什么是任务定时器：

> 能够在指定时间定期地执行命令、脚本或者程序，可以被用于系统的自动化维护及管理的工具；

最常见的任务定时器当属 `Cron` ，该词源于希腊语 chronos，原意是时间；
在类 Unix 系统中，可以通过 `crontab` 命令设置定时任务执行的时间周期，然后 `cron` 的守护进程会在后台实时的检测是否有需要执行的任务，通常这些需要执行的任务被称为 `cron jobs`;

<!--more-->

## 2. 定时器的参数说明：

`Cron` 在类 Unix 系统中有很多的实现，如：cronie、bcron、dcron、fcron 等；
虽然实现有多种，但是基于 `cron` 风格的时序参数结构却是相似的；

这里展示了一个 cron 的时序参数格式：

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    |
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)
```

`*` 表示通配符，匹配任意，当秒为 '*' 时，代表任意秒数都会触发执行；

观察这组参数说明的过程中，发现了一个有趣的问题:

`Q`: 在 `day of week` 参数上，0 或 7 其实都可以代表星期日(Sunday)，WHY?
`A`: 原因在于 `cron` 的众多实现版本里，对于这个参数，部分实现的设置为：0 - 6 => Sunday - Saturday，而另一部分实现的设置为：1 - 7 => Monday - Sunday，为了兼容，确保两种设置都正确，所以如上；

## 3. 具体参数设置的试例：
```

* * * * *           # 例1: 每1分钟执行一次

3,15 * * * *        # 例2: 每小时的第3和第15分钟执行

3,15 8-11 * * *     # 例3: 在每天上午8点到11点的第3和第15分钟执行

3,15 8-11 */2 * *   # 例4：每隔两天的上午8点到11点的第3和第15分钟执行

3,15 8-11 * * 1     # 例5：每周一的上午8点到11点的第3和第15分钟执行

30 21 * * *         # 例6：每晚的21时30分执行

45 4 1,10,22 * *    # 例7：每月的1、10、22日的4时45分执行
 
10 1 * * 6,0        # 例8：每周六、周日的1时10分执行

0,30 18-23 * * *    # 例9：每天 18:00 至 23:00 之间每隔30分钟执行

0 23 * * 6          # 例10：每星期六晚 23:00 执行

* */1 * * *         # 例11：每隔1小时执行

* 23-7/1 * * *      # 例12：晚上23时到次日7时之间每隔1小时执行
 
```

## 4. 任务定时器在 Node 中的使用

在 Node.JS 中，可以使用 [`Node-Schedule`](https://github.com/node-schedule/node-schedule) 这个 npm 包来进行任务定时器的操作：



> 描述： A cron-like and not-cron-like job scheduler for Node.

### 4.1 安装方式：
```
npm install node-schdule
```

### 4.2 如何使用：

#### 4.2.1 任务定时器的创建：
```js
var schedule = require('node-schedule');

var j = schedule.scheduleJob('42 * * * *', function(){
    // 每小时的第42分钟被执行
  console.log('The answer to life, the universe, and everything!');
});
```
`scheduleJob()` 方法的回调函数用于实现具体定时任务执行的内容;

#### 4.2.2 任务定时器的注销：
```js
j.cancel();
```

#### 4.2.3 基于JS Date类型的时间参数设置：
```js
var schedule = require('node-schedule');
// 2012年12月21日5时30分执行
var date = new Date(2012, 11, 21, 5, 30, 0);

var j = schedule.scheduleJob(date, function(){
  console.log('The world is going to end today.');
});
```
注意： 在使用 Date 设置参数是，月份的设定范围是 0~11 ，其中0代表一月，11代表十二月；

#### 4.2.4 递归循环任务的设置：
```js
var schedule = require('node-schedule');

var rule = new schedule.RecurrenceRule();
// 每小时的第 42 分钟执行
rule.minute = 42;
// rule.dayOfWeek = 5;
// rule.month = 6;
// rule.dayOfMonth = 15;
// rule.hour = 1;
// rule.second = 0

var j = schedule.scheduleJob(rule, function(){
  console.log('The answer to life, the universe, and everything!');
});
```

#### 4.2.5 指定时间范围的设置：
```js
var rule = new schedule.RecurrenceRule();
// 每个月的星期四、五、六、日的17点整执行
rule.dayOfWeek = [0, new schedule.Range(4, 6)];
rule.hour = 17;
rule.minute = 0;

var j = schedule.scheduleJob(rule, function(){
  console.log('Today is recognized by Rebecca Black!');
});
```
`RecurrenceRule` 实例的每个cron属性可接受以数组的形式添加多个时间数值，`Range()`方法可指定一个范围的开始值及结束值;

#### 4.2.6 通过对象字面量的方式设置：
```js
// 每周日的14时30分执行
var j = schedule.scheduleJob({hour: 14, minute: 30, dayOfWeek: 0}, function(){
  console.log('Time for tea!');
});
```

#### 4.2.7 任务定时器的开始及结束时间设置：
```js
// 5秒后任务开始执行且10秒后任务结束，任务在此过程中每秒执行一次
let startTime = new Date(Date.now() + 5000);
let endTime = new Date(startTime.getTime() + 5000);
var j = schedule.scheduleJob({ 
    start: startTime, 
    end: endTime, 
    rule: '*/1 * * * * *' 
}, function(){
  console.log('Time for tea!');
});
```


---
> 参考：
http://www.cnblogs.com/zhongweiv/p/node_schedule.html
http://www.codexiu.cn/javascript/blog/16175/
https://zh.wikipedia.org/wiki/Cron
