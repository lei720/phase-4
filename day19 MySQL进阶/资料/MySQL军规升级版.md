# 58 到家 MySQL 军规升级版

## 基础规范

- 表存储引擎必须使用 `InnoDB`
- 表字符集默认使用 `utf8`，必要时候使用 `utf8mb4`
    * 通用，无乱码风险，汉字 3 字节，英文 1 字节
    * `utf8mb4` 是 `utf8` 的超集，有存储 4 字节例如表情符号时，使用它
- 禁止使用存储过程，视图，触发器，Event
    * 对数据库性能影响较大，互联网业务，能让站点层和服务层干的事情，不要交到数据库层
    * 调试，排错，迁移都比较困难，扩展性较差
- 禁止在数据库中存储大文件，例如照片，可以将大文件存储在对象存储系统（OSS），数据库中存储路径
- 禁止在线上环境做数据库压力测试
- 测试，开发，线上数据库环境必须隔离

## 命名规范

- 库名，表名，列名必须用小写，采用下划线分隔 tb_book  t_book  
    * abc，Abc，ABC 都是给自己埋坑
- 库名，表名，列名必须见名知义，长度不要超过 32 字符
    * tmp，wushan 谁 TM 知道这些库是干嘛的
- 库备份必须以 bak 为前缀，以日期为后缀
- 从库必须以 `-s` 为后缀
- 备库必须以 `-ss` 为后缀

## 表设计规范

- 单实例表个数必须控制在 `2000` 个以内
- 单表分表个数必须控制在 `1024` 个以内
- 表必须有主键，推荐使用 `UNSIGNED` 整数为主键
    * 删除无主键的表，如果是 `row` 模式的主从架构，从库会挂住
- 禁止使用物理外键，如果要保证完整性，应由应用程式实现
    * 外键使得表之间相互耦合，影响 `update/delete` 等 SQL 性能，有可能造成死锁，高并发情况下容易成为数据库瓶颈
- 建议将大字段，访问频度低的字段拆分到单独的表中存储，分离冷热数据（具体参考：[《如何实施数据库垂直拆分》](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959773&idx=1&sn=7e4ad0dcd050f6662dfaf39d9de36f2c&chksm=bd2d04018a5a8d17b92098b4840aac23982e32d179cdd957e4c55011f6a08f6bd31f9ba5cfee&scene=21#wechat_redirect)）

## 列设计规范

- 根据业务区分使用 `tinyint/int/bigint`，分别会占用 `1/4/8` 字节
- 根据业务区分使用 `char/varchar`
    * 字段长度固定，或者长度近似的业务场景，适合使用 `char`，能够减少碎片，查询性能高
    * 字段长度相差较大，或者更新较少的业务场景，适合使用 `varchar`，能够减少空间
- 根据业务区分使用 `datetime/timestamp`
    * 前者占用 5 个字节，后者占用 4 个字节，存储年使用 `YEAR`，存储日期使用 `DATE`，存储时间使用 `datetime`
- 必须把字段定义为 `NOT NULL` 并设默认值“”
    * NULL 的列使用索引，索引统计，值都更加复杂，MySQL 更难优化
    * NULL 需要更多的存储空间
    * NULL 只能采用 `IS NULL` 或者 `IS NOT NULL` ，而在 `=/!=/in/not in` 时有大坑
- 使用 `INT UNSIGNED` 存储 `IPv4` ，不要用 `char(15)`
- 使用 `varchar(20)` 存储手机号，不要使用整数
    * 牵扯到国家代号，可能出现 `+/-/()` 等字符，例如 `+86`
    * 手机号不会用来做数学运算
    * `varchar` 可以模糊查询，例如 `like‘138%’`
- 使用 `TINYINT` 来代替 `ENUM`
    * `ENUM` 增加新值要进行 `DDL` 操作
    
## 索引规范

- 唯一索引使用 `uniq_[字段名]` 来命名
- 非唯一索引使用 `idx_[字段名]` 来命名
- 单张表索引数量建议控制在 5 个以内，如果业务需要，表中需要更多的索引，那就大胆去建索引，为了提升性能，索引多几个没关系。（建索引一定要按照公司的发布流程来走）
    * 互联网高并发业务，太多索引会影响写性能
    * 生成执行计划时，如果索引太多，会降低性能，并可能导致 MySQL 选择不到最优索引
    * 异常复杂的查询需求，可以选择 `ES` 等更为适合的方式存储
- 组合索引字段数不建议超过 5 个
    * 如果 5 个字段还不能极大缩小 row 范围，八成是设计有问题
- 不建议在频繁更新的字段上建立索引
- 非必要不要进行 `JOIN` 查询，如果要进行 `JOIN` 查询，被 `JOIN` 的字段必须类型相同，并建立索引
    * 踩过因为 `JOIN` 字段类型不一致，而导致全表扫描的坑么？
- 理解组合索引最左前缀原则，避免重复建设索引，如果建立了(a,b,c)，相当于建立了(a), (a,b), (a,b,c)

## SQL 规范

- 禁止使用 `select *`，只获取必要字段
    * `select *` 会增加 `cpu/io/内存/带宽` 的消耗
    * 指定字段能有效利用索引覆盖
    * 指定字段查询，在表结构变更时，能保证对应用程序无影响
- `insert` 必须指定字段，禁止使用 `insert into T values()`
    * 指定字段插入，在表结构变更时，能保证对应用程序无影响
- 隐式类型转换会使索引失效，导致全表扫描
- 禁止在 `where` 条件列使用函数或者表达式
    * 导致不能命中索引，全表扫描
- 禁止负向查询以及 `%` 开头的模糊查询
    * 导致不能命中索引，全表扫描
- 禁止大表 `JOIN` 和子查询
- 同一个字段上的 `OR` 必须改写问 `IN`，`IN` 的值必须少于 50 个
- 应用程序必须捕获 SQL 异常
    * 方便定位线上问题

## 说明

本军规适用于并发量大，数据量大的典型互联网业务，可直接带走参考，不谢。