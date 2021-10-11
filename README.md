

## WIN常见问题

#### 端口占用

win:

```bash
netstat -ano | findstr 8080
taskkill /pid [pid] /F
```



#### maven安装本地jar包

```bash
mvn install:install-file -DgroupId=cn.hutool -DartifactId=hutool-all -Dversion=5.6.3 -Dpackaging=jar -Dfile=hutool-all-5.6.3.jar
```



#### ffmpeg

```bash
# flv转mp4
ffmpeg -i "source.flv" -c copy "target.mp4"
# 音频、视频都直接 copy，只是将 mp4 的封装格式转成了flv
ffmpeg -i out.mp4 -vcodec copy -acodec copy out.flv
# 把图片pic作为略所图
ffmpeg -i source.mp4 -i pic.jpg -map 1 -map 0 -c copy -disposition:0 attached_pic target.mp4
# 切割MP3，按时间切割
ffmpeg -i D:\music.mp3 -ss 00:20:00 -to 02:30:05 D:\target.mp3
# MP4改尺寸 -b压缩率
ffmpeg -i source.mp4 -b 4M -s 1920*1080 -max_muxing_queue_size 9999 target.mp4
# 快速剪切某段视频作为输出
ffmpeg -i D:\source.mp4 -ss 0:0:0 -to 0:23:20 -c copy D:\target.mp4
```



#### nacos单机启动，默认集群

```bash
startup.cmd -m standalone
# 访问地址: http://localhost:8848/nacos/
# 用户名、密码: nacos/nacos
```



#### sentinel控制台启动

```bash
java -Dserver.port=8849 -jar sentinel-dashboard-1.8.1.jar
# 用户名、密码: sentinel/sentinel
```

## LINUX常见问题

+ 查看某端口是否占用

  ```bash
  netstat -an|grep LIST|grep 6379
  ```

+ ubuntu开机启动纯命令行

  ```bash
  systemctl set-default multi-user.target

+ ubuntu开机启动图形界面

  ```bash
  systemctl set-default graphical.target
  ```


+ 自动获取ip

  ```bash
  sudo dhclient ens33
  ```

  

## 环境搭建常见问题&&命令

#### Docker常用命令

+ 重启

  ```bash
  systemctl daemon-reload && systemctl restart docker
  ```

  

+ 拉取镜像

  ```bash
  docker pull mysql
  ```

  

+ 查看本地镜像

  ```bash
  docker images
  ```

  

+ 查看运行中的容器

  ```bash
  docker ps [-aq] [-n 1]
  ```

  

+ 暴露端口 -p 宿主机 : 本机

  ```bash
  docker run -it -d -p 6379:6379 redis
  ```

  

+ 将本地目录创建的配置文件挂载到docker nginx的配置文件, redis同理

  ```bash
  docker run --name nginx -d -p 9000:9000
  -v /mynginx/conf/nginx.conf:/etc/nginx/nginx.conf 
  -v /mynginx/log:/var/log/nginx 
  -v /mynginx/conf.d:/etc/nginx/conf.d 
  -v /mynginx/html:/usr/share/nginx/html nginx
  ```

  

+ 退出容器

  ```bash
  CTRL+P+Q 或者 exit
  ```
  
  
  
+ 进入正在运行的容器

  ```bash
  docker attach [容器id]  或者 docker exec -it [容器id] /bin/bash
  ```
  
  
  
+ :anchor:  ​ docker容器ip地址查看

  ```bash
  docker inspect --format='{{.NetworkSettings.IPAddress}}'  [容器名称]
  ```

  

+ docker 容器开机自动启动

  ```bash
  docker update mysql --restart=always
  ```
  
  
  
+ docker 查看容器详细信息

  ```bash
  docker inspect [容器id/name]
  ```
  
  
  
+ docker 查看挂载的盘

  ```bash
  docker volume ls
  docker volume inspect [volumeId]
  ```
  
  
  
+ docker 清除无主的数据卷

  ```bash
  docker volume prune
  ```



#### Docker搭建环境、集群

+ Mysql

  ```bash
  docker run 
  -p 3306:3306 
  -v /mysql/conf:/etc/mysql/conf.d
  -v /myslq/logs:/logs
  -v /mysql/data:/var/lib/mysql
  -e MYSQL_ROOT_PASSWORD=123456
  --name mysql
  -d mysql
  ```
  
  开启远程访问
  
  ```bash
  grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option; flush privileges;
  ```
  
  
  
+ Docker搭建Nacos集群

  执行多次，分别修改成对应ip和端口，Nacos之间互相守望
  
  ```bash
  docker run -d \
  -e PREFER_HOST_MODE=hostname \
  -e MODE=cluster \
  -e NACOS_APPLICATION_PORT=8846 \
  -e NACOS_SERVERS="localhost:8846 localhost:8847 localhost:8848" \
  -e SPRING_DATASOURCE_PLATFORM=mysql \
  -e MYSQL_SERVICE_HOST=192.168.56.10 \
  -e MYSQL_SERVICE_PORT=3306 \
  -e MYSQL_SERVICE_USER=root \
  -e MYSQL_SERVICE_PASSWORD=root \
  -e MYSQL_SERVICE_DB_NAME=nacos_config \
  -e NACOS_SERVER_IP=localhost \
  -p 8846:8846 \
  --name my-nacos1 \
  nacos/nacos-server
  ```
  
+ :boom:远程nginx配置踩过的坑

  过程

  > nginx监听所在主机的端口,此处为9000 `->` 前端绑定nginx所在主机地址及对应端口 `->`  nginx监听服务所在主机的服务并按规则转发, **即前端发请求到nginx，nginx再转到对应的服务及其所在地址** `->` 后端响应

  ```nginx
  http{
      # 允许文件大小
      client_max_body_size    1024m;
  
      server{
          listen		9000;
          server_name localhost;
          location ~ /Aservice/ {
              proxy_pass http://localhost:8001;
              # 如果服务在远程地址下，则把localhost改为远程ip，远程主机ip地址最好改为静态
              # vue前端页面地址改为nginx所在地址
          }
          location ~ /Bservice/ {
              proxy_pass http://localhost:8002;
          }
      }
      # 当监听到server_name + listen（对应的ip和端口），发出的请求中以location名字开头的，将会转换为proxy_pass配置的地址，进行请求操作
      
  ```

  > 关闭nginx
  >
  > nginx -s stop



- RabbitMQ

  ```bash
  # docker启动rabbitmq并创建管理员帐户
  docker run -d -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123 --name rabbitmq --hostname=rabbitmqhostone  rabbitmq:management
  ```



+ Elasticsearch

  ```bash
  mkdir -p /elasticsearch/config
  mkdir -p /elasticsearch/data
  mkdir -p /elasticsearch/plugins
  
  echo "http.host: 0.0.0.0" >> /elasticsearch/config/elasticsearch.yml
  chmod -R 777 /elasticsearch/
  
  # docker启动elasticsearch创建配置文件并挂载到docker内，暴露两个端口，一个请求端口，一个集群通信端口
  docker run -p 9200:9200 -p 9300:9300 \
  --name elasticsearch --net elastic \
  -e "discovery.type=single-node" \
  -e ES_JAVA_OPTS="-Xms64m -Xmx128m" \
  -v /elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
  -v /elasticsearch/data:/usr/share/elasticsearch/data \
  -v /elasticsearch/plugins:/usr/share/elasticsearch/plugins \
  -d elasticsearch
  ```

  ```json
  // 查看健康状况
  post http://192.168.242.129:9200/_cat/health
  // 查看节点
  Get http://192.168.242.129:9200/_cat/nodes
  // 查看主节点
  Get http://192.168.242.129:9200/_cat/master
  // 查看索引
  Get http://192.168.242.129:9200/_cat/indices
  // 提交数据到customer索引下的external类型数据id为1号，再次提交则是更新
  Post http://192.168.242.129:9200/customer/external/1
  {
  	"name": "FFFF"
  }
  // 更新带乐观锁_seq_no
  Put http://192.168.242.129:9200/customer/external/1?if_seq_no=1&if_primary_term=1
  // _update更新会检查元数据，数据需doc对象格式，数据跟原来一样则什么都不做_version也不变
  Post http://192.168.242.129:9200/customer/external/1/_update
  {
  	"doc": {
  		"name": "FFFF",
  		"age": 20
  	}
  }
  // 不能单独删除类型，必须删除索引
  Delete http://192.168.242.129:9200/customer
  // bulk批量API,必须post
  Post http://192.168.242.129:9200/customer/external/_bulk
  {"index": {"_id": "1"}}
  {"name": "john Doe"}
  {"index": {"_id": 2}}
  {"name": "aaaaa"}
  ---
  复杂实例 Post/_bulk
  {"delete": {"_index": "website", "_type": "blog", "_id": "2"}}
  {"create": {"_index": "website", "_type": "blog", "_id": "2"}}
  {"title": "my first blog"}
  {"index": {"_index": "website", "_type": "blog"}}
  {"title": "my second blog"}
  {"update": {"_index": "website", "_type": "blog", "_id": "2"}}
  {"doc": {"title": "update my blog "}}
  ```

  

+ Kibana

  ```bash
  # 配合elasticsearch使用
  docker run --name kibana --net elastic -p 5601:5601 -e "ELASTICSEARCH_HOSTS=http://elasticsearch:9200" -d kibana
  ```

  

+ Canal

  模拟slaver向master发送请求实现同步数据

  > mysql数据解析关注的表，perl正则表达式 多个正则以，分隔，转义符需要\\\  该过滤条件只对row模式数据有效
  >
  > + 所有表 `.* or .*\\..*`
  >
  > + canal schema下的所有表 `canal\\..*`
  >
  > + canal下的canal打头的表 `canal\\.canal.*`
  >
  > + canal schema下的一张表 `canal.test`
  > + 多规则组合 `canal\\..*,mysql.table1,mysql.table2`

  ```bash
  docker run --name canal \
  -e canal.destinations=example \
  -e canal.instance.master.address=x.x.x.x:3306 \
  -e canal.instance.dbUsername=canal \
  -e canal.instance.dbPassword=canal \
  -e canal.instance.filter.regex=`.*\\..*` \
  -p 11111:11111 \
  -d canal/canal-server
  ```

+ Seata

  ```bash
  # 启动seata
  nohup /myseata/seata-server-1.4.2/bin/seata-server.sh -h 192.168.242.129 -p 8091 >log.out 2>1 &
  # 查看后台脚本
  ps -ef | grep seata-server.s
  ```

  

## 常见问题

#### 事务

1. 一个事务里包含本地调用和远程调用，如果远程调用过程中出现假失败（网络缓慢超时等，但数据已写入远程数据库）会导致本地事务回滚但远程事务回滚失效引发数据不对的问题，需要引入`分布式事务`

2. 事务传播设置(REQUIRED)最外层覆盖内层,REQUIRES_NEW新建事务

3. 同一个对象(如一个service)内事务方法互调默认失败，原因是事务是使用代理对象控制，而这绕过了代理对象

   > 解决方法：不让绕过使用代理
   >
   > 引入 `starter-aop`, 主要用到`AspectJ`
   >
   > 需要开启注解 `@EnableAspectJAutoProxy(exposeProxy = true)`, 即使没有接口也可以产生动态代理；默认是JDK按照接口生成的动态代理
   >
   > `exposeProxy`: 对外暴露代理对象

	```java
	@Transactional(timeout = 30)
	public void a(){
        SercviceImpl service = (ServiceImpl) AopContext.currentProxy();
        service.b();
        service.c();
    }
   
    @Transactional(propagation = Propagation.REQUIRED, timeout = 10)
    public void b(){}
   
    @Transactional(propagation = Propagation.REQUIRES_NEW, timeout = 20)
    public void c(){}
    ...
   ```

4. 分布式事务
   + RAFT算法(类似于领导选举)保证一致性和解决分区容错(CP),但不能保证可用(A)
   + BASE理论：强一致性，弱一致性，`最终一致性`

###### SEATA

+ 添加注解`@GloabalTransactional`到业务方法上





## SpringBoot集成组件



#### 集成Redis

+ springboot集成redis，主启动类上加`@EnableCaching`

  - 方式1：注解

    + `@Cacheable` 一般用在查询
    + `@CachePut` 一般用在新增
    + `@CacheEvict` 一般用在修改删除

  - 方式2：RedisTemplate

    + 配置文件

      ```yaml
      spring: 
        redis:
          host: 192.168.1.1
          port: 6379
          database: 0
          timeout: 1800000
          lettuce:
            pool:
              max-active: 20
              max-wait: -1
              max-idle: 5
              min-idle: 0
      ```

      

    + 配置类

      ```java
      public class RedisConfig extends CachingConfigurerSupport {
      
      	@Bean
      	public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
      		RedisTemplate<String, Object> template = new RedisTemplate<>();
      		RedisSerializer<String> redisSerializer = new StringRedisSerializer();
      		Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      		ObjectMapper om = new ObjectMapper();
      		om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      		om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
      		jackson2JsonRedisSerializer.setObjectMapper(om);
      		template.setConnectionFactory(factory);
      		//key序列化方式
      		template.setKeySerializer(redisSerializer);
      		//value序列化
      		template.setValueSerializer(jackson2JsonRedisSerializer);
      		//value hashmap序列化
      		template.setHashValueSerializer(jackson2JsonRedisSerializer);
      		return template;
      	}
      
      	@Bean
      	public CacheManager cacheManager(RedisConnectionFactory factory) {
      		RedisSerializer<String> redisSerializer = new StringRedisSerializer();
      		Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      		//解决查询缓存转换异常的问题
      		ObjectMapper om = new ObjectMapper();
      		om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      		om.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
      		jackson2JsonRedisSerializer.setObjectMapper(om);
      		// 配置序列化（解决乱码的问题）,数据过期时间600秒
      		RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
      				.entryTtl(Duration.ofSeconds(600))
      				.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
      				.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
      				.disableCachingNullValues();
      		RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
      				.cacheDefaults(config)
      				.build();
      		return cacheManager;
      	}
      }
      ```

    + 用法

      ```java
      @Autowired
      private RedisTemplate redisTemplate;
      
          redisTemplate.opsForValue().set()...;
          redisTemplate.opsForHash()...;
          ...
      ```

      

#### 集成网关

- gateway网关配置

  ```yml
  spring:
    cloud:
      nacos:
        discovery:
          server-addr: 127.0.0.1:8848
      gateway:
        # 路由转发
        routes:
          - id: admin_route
            uri: lb://renren-fast
            predicates:
              - Path=/api/**
            filters:
              # 路径重写
              - RewritePath=/api/(?<segment>.*), /renren-fast/$\{segment}
  ```
  
  
  
  ```xml
  <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-loadbalancer</artifactId>
      <version>3.0.3</version>
  </dependency>
  ```
  
  > lb://  如果报503需要添加loadbalance依赖,需要负载均衡来进行路由



#### 集成MapStruct

+ 依赖

  ```xml
  <properties>
      <org.mapstruct.version>1.4.2.Final</org.mapstruct.version>
  </properties>
  ...
  <dependencies>
      <dependency>
          <groupId>org.mapstruct</groupId>
          <artifactId>mapstruct</artifactId>
          <version>${org.mapstruct.version}</version>
      </dependency>
      <dependency>
          <groupId>org.mapstruct</groupId>
          <artifactId>mapstruct-processor</artifactId>
          <version>${org.mapstruct.version}</version>
      </dependency>
  </dependencies>
  ...
  <build>
      <plugins>
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-compiler-plugin</artifactId>
              <version>3.8.1</version>
              <configuration>
                  <source>1.8</source>
                  <target>1.8</target>
                  <annotationProcessorPaths>
                      <path>
                          <groupId>org.mapstruct</groupId>
                          <artifactId>mapstruct-processor</artifactId>
                          <version>${org.mapstruct.version}</version>
                      </path>
                      <path>
                          <groupId>org.projectlombok</groupId>
                          <artifactId>lombok</artifactId>
                          <version>1.18.18</version>
                      </path>
                  </annotationProcessorPaths>
              </configuration>
          </plugin>
      </plugins>
  </build>
  ```

+ 定义一个转换类

  ```java
  @Mapper
  public interface MemberPriceStruct {
      MemberPriceStruct INSTANCE = Mappers.getMapper(MemberPriceStruct.class);
      /**
       * 数据对象转会员价格实体类
       *
       * @param to 数据对象
       * @return 会员价格
       */
      @Mappings({
              @Mapping(constant = "1", target = "addOther"),
              @Mapping(source = "id", target = "memberLevelId"),
              @Mapping(source = "name", target = "memberLevelName"),
              @Mapping(source = "price", target = "memberPrice")
      })
      MemberPriceEntity to2MemberPrice(MemberPrice to);
      /**
       * 数据对象列表转会员价格列表
       */
      List<MemberPriceEntity> toList2MemberPriceList(List<MemberPrice> toList);
  }
  ```

  > `@Mapper`: MapStruct包里的注解
  >
  > `@Mapping`: 把`source`字段对应`target`字段，相同不必须; `constant`把一个常量赋值给`target` 