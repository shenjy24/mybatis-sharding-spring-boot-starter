## 项目简介
该`starter`简单集成了`MyBatis-Plus`和`Sharding-Sphere3`，目前仅仅是集成了简单的依赖和
简单的配置。

## 版本说明
- mybatis-plus: 3.1.1
- sharding-sphere: 3.1.0
- jdbc驱动版本: 6.0.6

说明：`spring boot`的版本最好为`2.1.2.RELEASE`，因为`mybatis-plus-starter`依赖的`spring boot`
版本为`2.1.2.RELEASE`，否则可能报错。

## 使用说明

#### 引入依赖
目前项目包暂未上传到`maven`中央仓库，所以只能下载到本地打包，然后在`spring boot`项目中引入下面依赖：
```xml
<dependency>
    <groupId>com.jonas</groupId>
    <artifactId>mybatis-sharding-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

#### mybatis-plus 配置
`mybatis-plus`的配置可以参考官网：https://mp.baomidou.com/

下面为参考配置：
```yaml
mybatis-plus:
  #配置mapper.xml路径
  mapper-locations: classpath:/mapper/**/*Mapper.xml 
  global-config:
    db-config:
      id-type: id_worker
  configuration:
    map-underscore-to-camel-case: true
  #配置枚举包路径
  type-enums-package: com.shenjy.enums
```

#### sharding-sphere 配置
`sharding-sphere`的配置可以参考官网：https://shardingsphere.apache.org/document/legacy/3.x/document/en/overview/

下面为简单的分库分表配置，`order`为逻辑表，`order_0`和`order_1`为真实表：
```yaml
#### 分库分表配置
sharding:
  jdbc:
    # 配置数据源
    datasource:
      names: master0,slave0,slave1
      master0:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb1?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
      slave0:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb2?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
      slave1:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb3?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
    config:
      sharding:
        # 配置分表逻辑
        tables:
          order:
            actual-data-nodes: ds0.order_$->{0..1}
            table-strategy:
              inline:
                sharding-column: order_id
                algorithm-expression: order_$->{order_id % 2}
        # 配置主从库
        master-slave-rules:
          ds0:
            master-data-source-name: master0
            slave-data-source-names: slave0,slave1
      props:
          # 打印sql日志
          sql:
            show: true
```

以下是主从配置，不进行分库分表
```yaml
sharding:
  jdbc:
    # 配置数据源
    datasource:
      names: master0,slave0,slave1
      master0:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb1?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
      slave0:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb2?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
      slave1:
        type: com.alibaba.druid.pool.DruidDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/tradedb3?nullNamePatternMatchesAll=true&useSSL=false
        username: root
        password: 
    config:
      masterslave:
          load-balance-algorithm-type: random
          name: ms
          master-data-source-name: master0
          slave-data-source-names: slave0,slave1
      props:
          # 打印sql日志
          sql:
            show: true
```