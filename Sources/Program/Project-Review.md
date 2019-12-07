## Crash问题汇总

#### Crash-首页连续下拉刷新50次App出现Crash

该问题由星巴克QA测试发现并提出，但并不是TW的scope，其它vendor提出让TW帮忙分析原因。

经过抓取日志分析，发现该问题是创建Thread数量过多导致。具体结合下拉刷新的功能代码，发现每次下拉刷新时，会一次性创建13个任务，这些任务直接使用RxJava的默认的任务调度器Schedulers.io()来执行，Schedulers.io()任务调度器虽然本身是有缓存池的，但它的任务缓存策略是无界的，如果短时间内添加的任务很多，任务又比较耗时，就会创建大量的线程。连续进行多次下拉刷新操作，短时间内创建n x 13个任务也即创建n  x 13个线程导致App出现Crash，把问题原因给到WCL并由WCL修复。

#### Crash-7.1.3 chinapex

客户的bob提出Fabric上有两个crash影响的用户数量非常多，让TW协助排查原因。

从Fabric上看到两个crash都是SQLiteDatabase异常导致的，根据堆栈看发生异常的原因是 database is locked，是数据库读写操作时数据库实例被Lock了。

1. 数据库使用的是sqlcipher，是一个开源的module，这个module并不是线程安全的，使用这个数据库的地方如果没做多线程同步保护，是会导致异常的。

2. 根据堆栈来看数据库读写异常是由AnalyticsDbDao.deleteByTime导致的，AnalyticsDbDao 是引用的第三方 chinapex-data-collect-sdk ，在这个sdk中数据存储使用的即是sqlcipher。

chinapex这个sdk在业务代码里面只仅仅是使用sendGaEvent之类的方法统计用户行为，不涉及额外的业务处理，所以这个SQLiteDatabase异常和具体的业务并不相关。和业务方bob沟通确认相关的功能并没有在使用，可以删除该sdk及相关的业务代码，7.1.3版本修改发布后，该sdk导致的两个crash被修复。

> 该Crash修复后，App Crash-Free从93.5%提升到95%。

#### Crash-7.3.1登录/资源释放异常问题

1. 登录页面Crash，会直接影响用户体验，极端情况下用户会永远无法登录。过去30天影响用户数 2850，Crash次数 4150。该问题非必现，问题原因为三方登录转到其它三方App进程后，星巴克App进程因系统资源紧张被回收。

2. 资源释放异常错误，用户在使用过程当中或者退出应用后，App会Crash，影响用户体验。过去30天影响用户数 1.8w+ ，Crash次数 3.4w+。该问题为系统级问题，和手机ROM相关，高发于Oppo手机。

这两个Crash在过去30天总共影响用户数 2w+。

> 该Crash修复后，App Crash-Free从95.6%提升到97.4%，提升约1.8个百分点。

#### Crash-7.4.0版本兼容性/概率性问题

1. Android 6.0及以下机型收银台支付时Crash，Android 6.0及以下机型调起收银台时Crash。对用户的直接影响是不能正常支付购买产品，该Crash是Android版本兼容问题，代码里面使用了Android 6.0以上版本才支持的资源文件，导致6.0及以下机型在调起收银台时，不能正常支付。

2. Android 6.0及以下机型进入Delivery页面Crash，对用户的直接影响是不能使用Delivery功能。Delivery页面使用了一个vector矢量资源文件，该资源文件中使用了新的特性在Android 6.0不支持，导致进入Delivery页面会Crash。(问题由ShuInfo引入)

3. 星享好礼页面(Reward)概率性Crash，用户在进入该页面后加载数据，数据加载完成后概率性Crash。该Crash是由于好礼券数量获取方式错误导致。(老问题，WCL引入)

4. Pickup页面选择自取门店页面(PickupStoreLocator)概率Crash，用户在选择门店时概率Crash。

> Android 7.4.1版本App Crash-Free提升到98.3%。

#### Crash-7.5.0非码支付SDK问题

非码支付SDK多线程并发Crash修复，该Crash由非码支付SDK引发，用户在下单支付的时候，概率性Crash，7.4.1及之前的版本在过去90天，共影响2.4万+用户，共发生2.7万+次crash。(当前用户数量最多的版本是7.6.0及7.5.0，所以如果该问题在未修复的情况下，影响用户数量会远大于2.4万。)

在非码支付SDK代码已高度混淆的情况下，通过Crash堆栈找到相应的调用堆栈并还原代码块，准确定位有多线程并发问题的代码并给到非码支付SDK开发，推动SDK重新发版升级并测试验证通过发版上线。

> Android 7.5.0版本App Crash-Free 98.7%(该版ShuInfo引入了另外一个用户数量比较大的Crash，所以相比7.4.1，App Crash-Free率提升不大)





微信登录，服务端返回信息不统一，后端一开始提出让前端根据不同三方平台判断，但鉴于淘宝/支付宝后端返回的是统一的，提出建议让后端做平台判断，前端保持原有逻辑。


