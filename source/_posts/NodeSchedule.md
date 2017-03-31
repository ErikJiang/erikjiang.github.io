# Node 任务定时器的使用

标签： node schedule cron

---

## 1. 什么是任务定时器：

> 能够让用户在指定时间段周期性地运行命令、脚本或者程序，可以被用于系统的自动化维护及管理的工具；

## 2. 定时器的参数说明：
最常见的任务定时器当属命令行工具 `Cron` ；
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

在 Node.JS 中，可以使用 node-schedule 这个 npm 包来进行任务定时器的操作：
[`Node Schedule` ][1]https://github.com/node-schedule/node-schedule

安装方式：
```
npm install node-schdule
```

> 参考：
http://nodeclass.com/articles/78767
http://www.cnblogs.com/zhongweiv/p/node_schedule.html

---


Examples with the cron format:

```js
var schedule = require('node-schedule');

var j = schedule.scheduleJob('42 * * * *', function(){
  console.log('The answer to life, the universe, and everything!');
});
```

Execute a cron job when the minute is 42 (e.g. 19:42, 20:42, etc.).

And:

```js
var j = schedule.scheduleJob('0 17 ? * 0,4-6', function(){
  console.log('Today is recognized by Rebecca Black!');
});
```

Execute a cron job every 5 Minutes = */5 * * * *

#### Unsupported Cron Features

Currently, `W` (nearest weekday), `L` (last day of month/week), and `#` (nth weekday
of the month) are not supported. Most other features supported by popular cron
implementations should work just fine.

[cron-parser] is used to parse crontab instructions.

### Date-based Scheduling

Say you very specifically want a function to execute at 5:30am on December 21, 2012.
Remember - in JavaScript - 0 - January, 11 - December.

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);

var j = schedule.scheduleJob(date, function(){
  console.log('The world is going to end today.');
});
```

You can invalidate the job with the `cancel()` method:

```js
j.cancel();
```

To use current data in the future you can use binding:

```js
var schedule = require('node-schedule');
var date = new Date(2012, 11, 21, 5, 30, 0);
var x = 'Tada!';
var j = schedule.scheduleJob(date, function(y){
  console.log(y);
}.bind(null,x));
x = 'Changing Data';
```
This will log 'Tada!' when the scheduled Job runs, rather than 'Changing Data',
which x changes to immediately after scheduling.

### Recurrence Rule Scheduling

You can build recurrence rules to specify when a job should recur. For instance,
consider this rule, which executes the function every hour at 42 minutes after the hour:

```js
var schedule = require('node-schedule');

var rule = new schedule.RecurrenceRule();
rule.minute = 42;

var j = schedule.scheduleJob(rule, function(){
  console.log('The answer to life, the universe, and everything!');
});
```

You can also use arrays to specify a list of acceptable values, and the `Range`
object to specify a range of start and end values, with an optional step parameter.
For instance, this will print a message on Thursday, Friday, Saturday, and Sunday at 5pm:

```js
var rule = new schedule.RecurrenceRule();
rule.dayOfWeek = [0, new schedule.Range(4, 6)];
rule.hour = 17;
rule.minute = 0;

var j = schedule.scheduleJob(rule, function(){
  console.log('Today is recognized by Rebecca Black!');
});
```

#### RecurrenceRule properties

- `second`
- `minute`
- `hour`
- `date`
- `month`
- `year`
- `dayOfWeek`

> **Note**: It's worth noting that the default value of a component of a recurrence rule is
> `null` (except for second, which is 0 for familiarity with cron). *If we did not
> explicitly set `minute` to 0 above, the message would have instead been logged at
> 5:00pm, 5:01pm, 5:02pm, ..., 5:59pm.* Probably not what you want.

#### Object Literal Syntax

To make things a little easier, an object literal syntax is also supported, like
in this example which will log a message every Sunday at 2:30pm:

```js
var j = schedule.scheduleJob({hour: 14, minute: 30, dayOfWeek: 0}, function(){
  console.log('Time for tea!');
});
```

#### Set StartTime and EndTime

It will run after 5 seconds and stop after 10 seconds in this example.
The ruledat supports the above.

```js
let startTime = new Date(Date.now() + 5000);
let endTime = new Date(startTime.getTime() + 5000);
var j = schedule.scheduleJob({ start: startTime, end: endTime, rule: '*/1 * * * * *' }, function(){
  console.log('Time for tea!');
});
```


## Contributing

This module was originally developed by [Matt Patenaude], and is now maintained by
[Tejas Manohar] and [other wonderful contributors].

We'd love to get your contributions. Individuals making significant and valuable
contributions are given commit-access to the project to contribute as they see fit.

Before jumping in, check out our [Contributing] page guide!

## Copyright and license

Copyright 2015 Matt Patenaude.

Licensed under the **[MIT License] [license]**.


[cron]: http://unixhelp.ed.ac.uk/CGI/man-cgi?crontab+5
[Contributing]: https://github.com/node-schedule/node-schedule/blob/master/CONTRIBUTING.md
[Matt Patenaude]: https://github.com/mattpat
[Tejas Manohar]: http://tejas.io
[license]: https://github.com/node-schedule/node-schedule/blob/master/LICENSE
[Tejas Manohar]: https://github.com/tejasmanohar
[other wonderful contributors]: https://github.com/node-schedule/node-schedule/graphs/contributors
[cron-parser]: https://github.com/harrisiirak/cron-parser


  [1]: https://github.com/node-schedule/node-schedule

