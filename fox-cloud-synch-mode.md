# 同步模式

## 相关介绍
> 现场的Fox-Edge和云端的Fox-Cloud，共同组成了Fox物联网的边缘计算和云端计算。


## 基本概念
- **边缘计算**：Fox-Edge被部署在现场侧的工控机设备之中，充当对现场物联网智能设备的管理

- **云端计算**：Fox-Cloud被部署在云端，将各地的Fox-Edge数据汇聚到中心，方便进行集中的数据处理。

- **数据同步**：数据由Fox-Edge节点向Fox-Cloud发起同步。Fox-Edge节点不断的将数据推送到Fox-Cloud之中。

## 同步模式

### 1.用户配置模式

##### 业务场景


在Fox-Edge上存在用户的各种配置信息，比如DeviceEntity、ChannelEntity、UserEntity等实体数据。
这类数据是由用户手动配置产生的。


##### 数据特点

此类数据由于是用户手动配置产生的，所以它同时具有下面的几个特点:
1. 数据量**非常有限**
2. 数据**不会频繁性的变化**
3. 边缘侧的数据才是数据的标准
4. 此类**数据重要、及时**，不允许数据丢失、两边不一致、同步严重滞后的状态
5. 此类数据要从边缘侧及时的发布到云端


##### 同步过程
**基本特点**：首次全量同步，后续增量同步

例如：安装完成后全量同步，长期运行阶段增量同步

- 全量同步
	``` txt
	1、Edge启动后，向Cloud询问，双方是否要进行一次初始化性质的重置？
	2、Cloud检查自身，是否有云端管理员在云端标识的重新初始化同步的标志，或者本身尚未有初始化过的标志
	3、Edge接到重置的标记，则准备进行重置性的全量同步，将自身的全量数据一次性以reset分组发送给Cloud，并标识上Edge的时间戳
	4、Cloud接收到推送数据时，发现是reset数据，那么将对本地数据进行一次全量性的增删改比对，结束后标识上Edge的时间戳，
	5、此时，Edge和Cloud就完成了初始化同步，两边数据保存一致，时间戳也保持一致，可以后续的增量同步了
	```

- 增量同步
	``` txt
	Edge在运行期不断向Cloud查询是否需要reset标记，确定是进入全量同步阶段，还是增量同步阶段，确定进入增量同步阶段后，进入下列步骤
	1、Edge向Cloud查询上次Edge标识的时间戳，如果时间戳跟Edge自己身上的一致，那么就认为两边一致，否则就进行后面的增量同步操作
	2、Edge捕捉自己数据的增删改变化，向Cloud发送insert/delete/update操作，同时带上时间戳标识
	3、Cloud接收到Edge的insert/delete/update请求，对Cloud上的数据进行增删改操作，并标识上新的时间戳
	4、不断重复上述过程，完成增量同步过程
	
	```
	
### 2.设备状态模式

##### 业务场景


在Fox-Edge上会不断对下面的物联网设备进行数据采集，此时会产生状态数据。比如DeviceValueEntity、TriggerValueEntity实体数据。
这类数据是由设备自动产生的，比如温度数据、电压数据。


##### 数据特点


此类数据由于是设备自动产生的，所以它同时具有下面的几个特点:
1. 数据量**相对有限**
2. 数据**高速变化**，**频繁变化**
3. 边缘侧的数据才是数据的标准
4. 此类**数据并不紧急**，由于会不断采样生成新的数据，所以采用定期同步的方式
5. 此类数据要从边缘侧及时的发布到云端



##### 同步过程
**基本特点**：全量的，周期性的，较长周期的，同步

例如：每小时全量同步一次

- 全量同步
	``` txt
     1、Edge周期性向Cloud查询时间戳。
	 2、Edge比对本地和Cloud的时间戳，判定是否需要进行同步，如果需要同步，就进行后面的流程
     3、向Cloud发出reset操作，Cloud接收到这个请求后，会清空自己的表数据，并标识为reset状态
     4、向Cloud循环的分页提交本地全部数据，Cloud会将数据逐个的插入到自己的数据库表中
     5、向Cloud发出complete操作，Cloud接收到这个操作后，会标识同步状态为complete
     6、至此，Edge和Cloud两边数据同步结束，Edge重新等待本地的数据和时间戳发生变化，然后重新进行上述流程
	```

### 3.设备结构模式

##### 业务场景


在Fox-Edge上会不断对下面的物联网设备进行数据采集，此时会产生设备的结构性数据。比如DeviceObjectEntity、TriggerObjectEntity实体数据。
这类数据是由设备自动产生的，比如设备的数据字典。


##### 数据特点


此类数据由于是设备自动产生的，所以它同时具有下面的几个特点:
1. 数据量**相对有限**
2. 数据**基本不会变化**
3. 边缘侧的数据才是数据的标准
4. 此类**数据相对要求及时**，由于会不断采样生成新的数据，所以采用定时比较时间戳，全量同步的方式
5. 此类数据要从边缘侧及时的发布到云端



##### 同步过程
**基本特点**：根据变化，每次全量同步

例如：一检测到本地发生变化，就发起全量同步

- 全量同步
	``` txt
	 1、Edge检测到本地数据变化，生成一个本地时间戳，然后去跟Cloud查询时间戳
     1、比对Edge和Cloud的时间戳是否相同，判定是否需要进行同步，如果需要同步，就进行后面的流程
     2、向Cloud发出重置操作，Cloud接收到这个请求后，会清空自己的表数据
     3、向Cloud循环的分页提交本地的全部数据，Cloud会将数据逐个的插入到自己的表总
     4、向Cloud发出完成操作，Cloud接收到这个操作后，会标识同步状态为完成
     5、至此，Edge和Cloud两边数据同步结束，重新等待Edge本地的数据和时间戳发生变化，然后重新进行上述流程
	```

### 4.事件记录模式

##### 业务场景


在Fox-Edge上会不断对下面的物联网设备进行数据采集，此时会产生记录类型的数据。比如DeviceRecordEntity、OperateRecordEntity实体数据。
这类数据是由设备自动产生的，比如设备的门禁记录、告警记录。也可能是用户操作产生，比如用户的操作记录。


##### 数据特点


此类数据由于是设备或者用户源源不断新产生的，所以它同时具有下面的几个特点:
1. 数据量**不断增加**
2. 数据**只会增量变化**
3. 边缘侧的数据才是数据的标准
4. 此类**数据要求及时**，所以采用触发**差额同步**的增量同步方式
5. 此类**数据比较重要**，所以发现异常导致的不一致，马上重新发起全量同步，再增量同步
6、同步的时候，会再同时保存数据在xxxEntity的主表，并且保存在xxxEntityBacckup表，用于回溯
7. 此类数据要从边缘侧及时的发布到云端



##### 同步过程
**基本特点**：根据序号的递增变化，每次增量同步差额数据

例如：Edge一检测到本地和Cloud出现序号变化，就发起差额同步

- 全量同步
	``` txt
	 1、Edge向Cloud查询时间戳，比对本地数据库的时间戳
     2、比对Edge和Cloud的时间戳是否相同，判定是否需要进行同步，如果需要同步，就进行后面的流程
     3、Edge分页查询本地数据库的差额数据，分页向Cloud推送增量数据
     4、至此，Edge和Cloud两边数据同步结束，重新等待Edge本地的数据和时间戳发生变化，然后重新进行上述流程
	```
	
- 业务场景

	``` txt
	 场景1：Edge本地没有数据，Cloud却有数据
	 说明两边出现了数据不一致的异常。
	 以Edge为基准，清空Cloud数据
	```
	
	``` txt
	 场景2：Edge本地有数据，Cloud没有数据
	 说明出现了需要全量同步的数据。
	 以Edge为基准，发布数据到Cloud
	```
	
	``` txt
	 场景3：Edge本地有数据，Cloud也有数据
	 说明出现了需要增量同步的数据。
	 以Edge为基准，查询增量差额数据，发布数据到Cloud
	```
	
### 5.日志记录模式

##### 业务场景


在Fox-Edge上会不断对下面的物联网设备的数据采集进行变化检测并记录数据，此时会产生日志类型的数据。比如DeviceHistoryEntity实体数据。
这类数据是由Fox-Edge自动产生的，比如对设备数据进行监测记录的功能


##### 数据特点


此类数据由于是Edge源源不断新产生的，所以它同时具有下面的几个特点:
1. 数据量**不断增加**
2. 数据**只会增量变化**
3. 边缘侧的数据才是数据的标准
4. 此类**数据不要求及时**，所以采用触发**差额同步**的增量同步方式
5. 此类**数据相对不重要**，所以发现两边不一致，只是认为各自清理了过多的旧数据
6. 此类数据要从边缘侧定期的发布到云端



##### 同步过程
**基本特点**：根据序号的递增变化，每次增量同步差额数据

例如：Edge一检测到本地和Cloud出现序号变化，就发起差额同步

- 增量同步
	``` txt
	 1、Edge向Cloud查询时间戳，比对本地数据库的时间戳
     2、比对Edge和Cloud的时间戳是否相同，判定是否需要进行同步，如果需要同步，就进行后面的流程
     3、Edge分页查询本地数据库的差额数据，分页向Cloud推送增量数据
     4、至此，Edge和Cloud两边数据同步结束，重新等待Edge本地的数据和时间戳发生变化，然后重新进行上述流程
	```
	
- 业务场景

	``` txt
	 场景1：Edge本地没有数据，Cloud却有数据
	 说明Edge本地被定期清理，Cloud的才是永久备份数据。
	 正常场景，不进行处理
	```
	
	``` txt
	 场景2：Edge本地有数据，Cloud没有数据
	 说明出现了需要全量同步的数据。
	 以Edge为基准，发布数据到Cloud
	```
	
	``` txt
	 场景3：Edge本地有数据，Cloud也有数据
	 说明出现了需要增量同步的数据。
	 以Edge为基准，查询增量差额数据，发布数据到Cloud
	```
	