# facade

因为涉及一些私人云服务信息，整套微服务架构项目设置了隐私模式，「这个项目提供对外入口以及功能介绍」

如果您感兴趣的话，请将您的账号发送到邮箱:syiea_jwy@126.com，我将为您设置权限。

# summary

整套架构"MY"的项目名称以DOTA2中的英雄名称命名。
目前包含可插拔组件：

|project      | description
|-------------|------------
|zeus         | parent
|medusa       | 框架核心实现
|wisp         | 公共项目
|karl         | 配置中心模块扩展
|warlock      | jdbc模块扩展
|witch        | cache模块（redis）扩展
|rubick       | log模块扩展
|doom         | admin 微服务管理模块
|broodmother  | 脚手架
|meepo        | 统一分布式ID发号器server
|arcwarden    | 统一分布式ID发号器client

## medusa

"MY"架构核心代码实现，自动装配入口: MyCoreAutoConfiguration

#### 可插拔功能：
- consul扩展：流量分流、consul发现服务管理等功能支持。 
  - 默认开启状态，你可以通过设置my.extension.consul=false来关闭此功能
  - ！如果关闭了此功能，将会对"my"微服务架构造成不可预见破坏，请勿自主决定关闭此功能
- WebMvc扩展：统一输出、请求log、异常标准化等功能支持。 
  - 默认开启状态，你可以通过设置my.extension.mvc=false来关闭此功能。
  - ！如果关闭此功能，将失去如上功能支持
  - 设置logging.level.com.jwy.medusa.mvc.MyAccessLogFilter=debug来log输出请求日志
- Feign扩展：实现异常机制传递(参考下一点)以及必要的"头"信息过滤
    - 默认开启状态，你可以设置my.extension.feign=false来关闭此功能
    - ！如果关闭了此功能，将会对"my"微服务架构造成不可预见破坏，请勿自主决定关闭此功能
- 异常机制：业务异常标准化、在内部rpc(feign)请求时的异常传递功能实现。
  - 异常标准化依赖WebMvc同步实现
  - 异常传递机制默认关闭状态。包括server(服务方)的发送开关，以及client(调用房)的请求开关。异常机制可以保证在feign调用的时候，由server产出的异常被client捕获。！开启异常传递机制会由性能影响，请在合适的场景使用，具体描述请参考相关类注释。想要开启功能需要如下配置：
    - my.extension.mvc=true(默认)
    - my.extension.feign=true(默认)
    - my.exception.transfer.send=true
    - my.exception.transfer.receive=true
- SaaS基础内容("多租户)实现
  - 默认开启状态，你可以设置my.extension.saas=false来关闭。
- 流量灰度/路由功能实现
  - 默认开启状态，你可以设置my.extension.feature=false来关闭。
  - ！灰度路由功能同时依赖负载均衡扩展，确保my.extension.load-balance=true(默认)来开启
- ApiDoc生成，默认开启状态，设置springdoc.api-docs.enabled=false关闭(请在prod环境下关闭)
- Authentication JWT访问权限验证, 默认开启状态，设置my.authorization.jwt.enabled=false关闭
- 系统上下文工具扩展：MyContextUtils。注入此工具类可以获取"MY"架构内的上下文内容，包括：
  - 统一JSON工具
  - Feature、SaaS Tenant、UserInfo 等上下文内容
  - Spring上下文工具
  
### 系统配置

| item                               | description
|------------------------------------|------------
| sys.restTemplate.connection.timeout|restTemplate创建连接超时时间：默认5s，单位是millisecond
| sys.restTemplate.read.timeout|restTemplate请求超时时间：默认10s，单位是millisecond
 
#### 关于Authentication验证模块

    "My"框架默认支持并开启JWT Authentication验证，即在请求的时候需要添加Authentication header并设置正确的token才准许访问

    你可以通过设置 my.authorization.jwt.enabled=false来关闭此功能
    你可以通过设置 my.jwt.token=xxx 来指定token
    你剋有通过设置 my.authorization.path.ignores=/xx,/xx来忽略指定的请求头

#### 技术选型

"MY"微服务架构使用如下技术实现重点功能：

- 使用consul为发现服务中心
- 使用Spring WebMVC 为提供web服务
- 使用feign提供内部RPC调用
- 使用load balance实现负载均衡
- 使用sleuth提供链路追踪
- 使用actuator提供服务状态信息
- 使用SpringDoc提供API可视化
- 使用prometheus提供监控
- 使用commons 、guava提供统一通用工具实现

关于Circuit Breaker相关内容目前在"网关"层主要处理

## zeus

parent项目，负责定义框架/插件版本

继承"spring-boot-starter-parent"实现jar包版本控制

如下统一信息子项目无须再设置：

- 统一utf8 
- 统一jdk:8 
- 统一基础spring cloud 版本 维持spring boot 2.7 对应的最新版
- 统一java工具：guava & apache common

## karl

架构-服务配置

- 引入apollo
- 添加environment.properties 、customized.properties

通常我们的项目配置文件由如下三部分构成：
- application.properties: 项目基础配置，不需要同步至apollo
- environment.properties: 环境配置，比如数据库、redis、consul等中间件配置, 对应apollo的namespace
- customized.properties: 自定义配置，比如自己些的@Value属性信息, 对应apollo的namespace

## warlock

framework jdbc模块

1. 引入合适的ORM(jpa)  以及相关插件
2. 确定表-实体之间的自动映射策略
3. ...

定义ID Generator Entity:
```
- AbstractEntity: 定义公共基础字段：比如创建时间
- AbstractAutoIncIdEntity: 设定ID自增策略
- AbstractSpecifyIdEntity: 设定ID有程序设定
```
 
## witch

framework cache模块-> redis

```
- 提供自定义template：
    - 读写分离模式 RedisTemplate
    - 仅主节点模式 MyMainStringRedisTemplate
- 自定义使用jackson序列化
- 提供分布式锁RedisLock，支持ttl与wait

```

## meepo

暂时定义meepo为一个"轻量级"的服务，而除了源生SnowFlake的实现之外，都需要额外的支持，比如数据库

- 排除掉baidu的uid-generator，因其基本上与使用SnowFlake实现无区别，且未有维护
- 排除掉美团leaf，在高QPS场景可以使用
- 排除掉滴滴Tinyid，与美团leaf相似，我认为还是美团leaf好点

需要关心两个配置：

```
- sys.snowflake.workid： 机器的编号，默认为1。！有多个节点部署的话必须要"手动配置"，不能重复
- sys.snowflake.datacenterId：数据中心的编号，默认为1。多个节点部署的话也要手动配置。
```

### 如何处理时钟回拨问题？

- 记录最近一次生成的ID
- 每次生成新的都要比较，如果不是递增顺序的话，则返回最近的id+1
  - 每次生成的ID步长为"10"
  - 不适用于"平均" qps > 1000的场景

### 升级路线

- 自研实现分段式数据库id分发
- 采用美团leaf

## arcwarden

ID Generator Client

meepo是ID Generator server，需要搭配使用.只要引用就默认支持这个模块，没有开关

使用方式如下：

```
...
@Autowired
private IdGeneratorClient idGeneratorClient;

...
long id = idGeneratorClient.nextId();
...
```
查看IdGeneratorClient的注释以更好的应用

## wisp

公共依赖项目，主要包含各个项目的公共"无状态"内容，比如rpc之间的requestDto与responseDto

请遵循以下规则在此项目中添加内容：

- 保持"公共"属性，至少有>=2项目使用的内容
- 设计尽量"抽象"，以提高复用概率
- 无状态，不要涉及具体业务上下文调用、线程创建等
- utils不要重复造轮子，优先使用apache、guava通用工具类
  
项目结构：
```
  -- constant: 常量定义放到这里
  -- enm: enum定义放到这里
  -- exception: exception定义放到这里
  -- pojo: 对应的各种"dto"定义放到这里
  -- util: 工具类放到这里
```

