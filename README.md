# DataV For Nemo
## 简介
[SNH48-费沁源应援会](https://weibo.com/u/5577610720?topnav=1&wvr=6&topsug=1)基于阿里云DataV的数据可视化系统

直达链接：[费沁源应援会集资监控系统](monitor.feiqinyuan.club)

## 各组件介绍
`modian_monitor.py`
* 该组件是实现数据可视化的基础，用于实时存储新产生的数据至服务器数据库


`dataCompensation.py`
* 数据补偿组件，受网络环境等因素影响，`modian_monitor.py`有极小概率会丢失订单，`dataCompensation.py`会比对摩点API接口返回数据和数据库中存储数据是否一致，并将遗漏订单及时补偿，保证服务的稳定性


`base_monitor.py`
* 用于监控竞争对手的集资情况，每隔一段时间更新数据库中竞争对手的集资数据

`auto_add_monitor.py`
* 该组件为`base_monitor.py`的升级版，`base_monitor.py`虽能及时更新数据，但使用久后暴露出一个问题，若竞争对手开设新项目，需要手动添加至监控列表，人工干预效率极低，`auto_add_monitor.py`通过监控用户主页，能自动将竞争对手新开设的项目加入监控列表，并通过日志提醒，实现整个监控体系的自动化


`jzdaily.py`

* 该组件用于存储**集资趋势图**所依赖的单日数据概况，配合`crontab`命令定时执行



## 部署文档

##### 前期准备

1. 在本地数据库中创建存放订单数据的数据表（示例文档以strawberry作为表名）

  ```
  CREATE TABLE `strawberry` (
  `pro_id` varchar(10) DEFAULT NULL,
  `user_id` varchar(10) DEFAULT NULL,
  `nickname` varchar(50) NOT NULL,
  `backer_money` double(10,2) NOT NULL,
  `pay_time` datetime NOT NULL
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

2. 在本地数据库中创建存放单日数据概况的数据表（示例文档以jzdaily作为表名）

  ```
   CREATE TABLE `jzdaily` (
    `date` date NOT NULL,
    `moneyNum` double(10,2) NOT NULL,
    `peopleNum` int(10) NOT NULL,
    `orderNum` int(10) NOT NULL
  ) ENGINE=InnoDB DEFAULT CHARSET=utf8
  ```

#### `modian_monitor.py` 的相关配置

1. 编辑`modian_monitor.py`

   * 根据实际情况，修改第**12-23** 行数据库连接的相关代码


   * 将第**78**行代码中的**pro_id**修改为你想要监控的摩点项目编号
   * 程序默认每10秒发起一次轮询，若要修改时间间隔，请将第**80**行代码`sched.add_job(newOrder, 'interval', seconds=10) `中seconds的参数值修改为你想设定的时间，同时请将第**55**行代码中的数值修改为你设定的seconds值
   * 示例文档的表名称为`strawberry` ，若在建表时修改了名称，请同时修改第**68**行代码中的表名称

2. 在终端输入`python modian_monitor.py` 命令开始监控

   （Tips:监控期间执行其他操作请另建一个终端窗口）

#### `dataCompensation.py ` 的相关配置

1. 编辑`dataCompensation.py`

   * 根据实际情况，修改第**16-27** 行数据库连接的相关代码
   * 将第**115**行代码中的**pro_id**修改为你想要监控的摩点项目编号
   * 若修改了表名，请同时修改第**39、48、89**行SQL语句中的代码

2. 在终端（Linux）输入`crontab -e`设置定时任务

   ```
   */5 * * * * /usr/local/bin/python /root/dataCompensation.py      #每5分钟执行一次
   ```

##  更新记录

**2018.04.09更新**：初次更新，共开放`auto_add_monitor.py`,`base_monitor.py`,`dataCompensation.py`,`modian_monitor.py`四个组件的源码

**2018.04.14更新**：新增`jzdaily.py`模块，用于存储当天集资概况数据至数据库

**2018.04.18更新**：完善了README中的部署文档

