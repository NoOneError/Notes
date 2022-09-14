# 1.安装redis

## docker方式

第一步 pull redis

```shell
命令：docker pull redis
```

第二步 创建redis管理目录，方便后期管理

```shell
命令：
mkdir /data/redis
mkdir /data/redis/data
```

第三步 redis 启动

```shell
命令：
docker run -p 6379:6379 --name redis \
-v /data/redis/con/redis.conf:/etc/redis/redis.conf \
-v /data/redis/data:/data \
-d redis redis-server /etc/redis/redis.conf --appendonly yes 
注：解析-p 6379:6379，-p 端口：映射端口；为了好看所以做了换行，执行的时候还是需要改成一行，每行一个空格隔开就可以了;
```

第四步 查看是否启成功

```shell
命令：docker ps
```

第五步  进入redis内部

```shell
docker exec -it "id" redis-cli
```

# 2.redis.KEY

### del（删除）

`格式：`del    key1   [key2 ....]

​			删除给定的一个或多个key。

​			不存在的key会被忽略

`可用版本：`>=1.0.0

`时间复杂度：`

​			O(N)， N 为被删除的 key 的数量。

 			删除单个字符串类型的 key ，时间复杂度为 O(1)。

​			 删除单个列表、集合、有序集合或哈希表类型的 key ，时间复杂度为 O(M)， M 为以 上数据结构内的元素数量。

`返回值：`

​			==被删除key的数量==

```sh
# 删除单个 key
redis> SET name huangz
OK
redis> DEL name
(integer) 1
# 删除一个不存在的 key
redis> EXISTS phone
(integer) 0
redis> DEL phone # 失败，没有 key 被删除
(integer) 0
# 同时删除多个 key
redis> SET name "redis"
OK
redis> SET type "key-value store"
OK
redis> SET website "redis.com"
OK
redis> DEL name type website
(integer) 3
```

### keys（查找key）

`格式：`keys pattern

​			查找所有符合给定模式 pattern 的 key 。

​			 KEYS * 匹配数据库中所有 key 。

​			 KEYS h?llo 匹配 hello ， hallo 和 hxllo 等。

​			 KEYS h*llo 匹配 hllo 和 heeeeello 等。

​			 KEYS h[ae]llo 匹配 hello 和 hallo ，但不匹配 hillo 。

 			特殊符号用 \ 隔开

 **警告：KEYS 的速度非常快，但在一个大的数据库中使用它仍然可能造成性能问题，如 果你需要从一个数据集中查找特定的 key ，你最好还是用 Redis 的集合结构(set)来代替**。

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(N)， N 为数据库中 key 的数量。

 `返回值：`  符合给定模式的 key 列表。

```sh
redis> MSET one 1 two 2 three 3 four 4 # 一次设置 4 个 key
OK
redis> KEYS *o*
1) "four"
2) "two"
3) "one"
redis> KEYS t??
1) "two"
redis> KEYS t[w]*
1) "two"
redis> KEYS * # 匹配数据库内所有 key
1) "four"
2) "three"
3) "two"
4) "one"
```

### randomkey（随机获取key）

`格式：`randomkey 

​			 从当前数据库中随机返回(不删除)一个 key 。

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(1)

 `返回值：  `当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil 。

```sh
# 数据库不为空
redis> MSET fruit "apple" drink "beer" food "cookies" # 设置多个 key
OK
redis> RANDOMKEY
"fruit"
redis> RANDOMKEY
"food"
redis> KEYS * # 查看数据库内所有 key，证明 RANDOMKEY 并不删除 key
1) "food"
2) "drink"
3) "fruit"
# 数据库为空
redis> FLUSHDB # 删除当前数据库所有 key
OK
redis> RANDOMKEY
(nil)
```

### ttl / pttl（获取key剩余时间）

`格式：`ttl key 

​				 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 **/**pttl ,以毫秒为单位返回

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(1) 

`返回值：`  当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。

 **注：在 Redis 2.8 以前，当 key 不存在，或者 key 没有设置剩余生存时间时，命令 都返回 -1 。**

```sh
# 不存在的 key
redis> FLUSHDB
OK
redis> TTL key
(integer) -2
# key 存在，但没有设置剩余生存时间
redis> SET key value
OK
redis> TTL key
(integer) -1
# 有剩余生存时间的 key
redis> EXPIRE key 10086
(integer) 1
redis> TTL key
(integer) 10084

```

### exists（判断key）

`格式：`exists key 

​			 检查给定 key 是否存在。 

`可用版本：`  >= 1.0.0 

`时间复杂度： ` O(1) 

`返回值：`  若 key 存在，返回 1 ，否则返回 0 。

```sh
redis> SET db "redis"
OK
redis> EXISTS db
(integer) 1
redis> DEL db
(integer) 1
redis> EXISTS db
(integer) 0
```

### move（转移key）

​	`格式：`move key db 

​			将当前数据库的 key 移动到给定的数据库 db 当中。 

​			如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 key ，或者 key 不存在于当前数据库，那么 MOVE 没有任何效果。

==因此，也可以利用这一特性，将 MOVE 当作锁(locking)原语(primitive)。==

`可用版本：`  >= 1.0.0

 `时间复杂度：`  O(1) 

`返回值：`  移动成功返回 1 ，失败则返回 0 。



### rename（重命名key）

`格式：`rename key newkey

​			将 key 改名为 newkey 。 

​			当 key 和 newkey 相同，或者 key 不存在时，返回一个错误。

​			 当 newkey 已经存在时， RENAME 命令将覆盖旧值。

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(1) 

`返回值：`  改名成功时提示 OK ，失败时候返回一个错误。

```sh
# key 存在且 newkey 不存
redis> SET message "hello world"
OK
redis> RENAME message greeting
OK
redis> EXISTS message # message 不复存在
(integer) 0
redis> EXISTS greeting # greeting 取而代之
(integer) 1
# 当 key 不存在时，返回错误
redis> RENAME fake_key never_exists
(error) ERR no such key
# newkey 已存在时， RENAME 会覆盖旧 newkey
redis> SET pc "lenovo"
OK
redis> SET personal_computer "dell"
OK
redis> RENAME pc personal_computer
OK
redis> GET pc
(nil)
redis:1> GET personal_computer # 原来的值 dell 被覆盖了
"lenovo"
```

### renamenx（重命名key）

`格式：`renamenx key newkey 

​			当且仅当 newkey 不存在时，将 key 改名为 newkey 。

​			 当 key 不存在时，返回一个错误。

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(1) 

`返回值：`

​			  修改成功时，返回 1 。

​			 如果 newkey 已经存在，返回 0 。

```sh
# newkey 不存在，改名成功
redis> SET player "MPlyaer"
OK
redis> EXISTS best_player
(integer) 0
redis> RENAMENX player best_player
(integer) 1
# newkey 存在时，失败
redis> SET animal "bear"
OK
redis> SET favorite_animal "butterfly"
OK
redis> RENAMENX animal favorite_animal
(integer) 0
redis> get animal
"bear"
redis> get favorite_animal
"butterfly"

```



### type（获取key类型）

`格式：`type key  

​			返回 key 所储存的值的类型。 

`可用版本：`  >= 1.0.0 

`时间复杂度：`  O(1)

 `返回值：` 

​			 none (key 不存在)

​			 string (字符串) 

​			list (列表) 

​			set (集合)

​			 zset (有序集) 

​			hash (哈希表)

```sh
# 字符串
redis> SET weather "sunny"
OK
redis> TYPE weather
string
# 列表
redis> LPUSH book_list "programming in scala"
(integer) 1
redis> TYPE book_list
list
# 集合
redis> SADD pat "dog"
(integer) 1
redis> TYPE pat
set
```

### **expire**（设置key过期时间）

`格式：`expire key seconds

​			为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。 在 Redis 中，带有生存时间的 key 被称为『可挥发』(volatile)的。

​			 生存时间可以通过使用 ==DEL== 命令来删除整个 key 来移除，或者被 **SET** 和 **GETSET**命 令覆写(overwrite)，这意味着，如果一个命令只是==修改(alter)==一个带生存时间的 key 的 值而不是用一个新的 key 值来代替(replace)它的话，那么生存时间不会被改变。

​			 比如说，对一个 key 执行 INCR 命令，对一个列表进行 LPUSH 命令，或者对一个哈希 表执行 HSET 命令，这类操作都不会修改 key 本身的生存时间。

​			 另一方面，如果使用 RENAME 对一个 key 进行**改名**，那么改名后的 key 的**生存时间和 改名前一样**。 RENAME 命令的另一种可能是，尝试将一个带生存时间的 key 改名成另一个带生存时间 的 another_key ，这时旧的 another_key (以及它的生存时间)会被删除，然后旧的 key 会 改名为 another_key ，因此，新的 another_key 的生存时间也和原本的 key 一样。 

​			使用 ==PERSIST== 命令可以在不删除 key 的情况下，移除 key 的生存时间，让 key 重新 成为一个『持久化』(persistent) key 。

`更新生存时间 `

​			 可以对一个已经带有生存时间的 key 执行 ==EXPIRE== 命令，新指定的生存时间会取代旧 的生存时间。

`过期时间的精确度 `

​			 在 Redis 2.4 版本中，过期时间的延迟在 1 秒钟之内 —— 也即是，就算 key 已经 过期，但它还是可能在过期之后一秒钟之内被访问到，而在新的 Redis 2.6 版本中，延迟 被降低到 1 毫秒之内。

`Redis 2.1.3 之前的不同之处`

​			  在 Redis 2.1.3 之前的版本中，修改一个带有生存时间的 key 会导致整个 key 被删 除，这一行为是受当时复制(replication)层的限制而作出的，现在这一限制已经被修复。 

`可用版本：`  >= 1.0.0

`时间复杂度：`  O(1) 

`返回值：`

​			  设置成功返回 1 。 当 key 不存在或者不能为 key 设置生存时间时(比如在低于 2.1.3 版本的 Redis 中 你尝试更新 key 的生存时间)，返回 0 。

```sh
redis> SET cache_page "www.google.com"
OK
redis> EXPIRE cache_page 30 # 设置过期时间为 30 秒
(integer) 1
redis> TTL cache_page # 查看剩余生存时间
(integer) 23
redis> EXPIRE cache_page 30000 # 更新过期时间
(integer) 1
redis> TTL cache_page
(integer) 29996
```

`模式：`导航会话  

​			假设你有一项 web 服务，打算根据用户最近访问的 N 个页面来进行物品推荐，并且假 设用户停止阅览超过 60 秒，那么就清空阅览记录(为了减少物品推荐的计算量，并且保持 推荐物品的新鲜度)。 这些最近访问的页面记录，我们称之为『导航会话』(Navigation session)，可以用 INCR  和 RPUSH 命令在 Redis 中实现它：每当用户阅览一个网页的时候，执行以下代码：

```sh
MULTI
 RPUSH pagewviews.user:<userid> http://.....
 EXPIRE pagewviews.user:<userid> 60
EXEC
```

​			如果用户停止阅览超过 60 秒，那么它的导航会话就会被清空，当用户重新开始阅览的 时候，系统又会重新记录导航会话，继续进行物品推荐。

### PEXPIRE（设置key过期时间）

`格式：`pexpire key milliseconds  

​			这个命令和 EXPIRE 命令的作用类似，但是它以毫秒为单位设置 key 的生存时间，而 不像 EXPIRE 命令那样，以秒为单位。搭配**pttl**使用

### **persist**（移除key过期时间）

`格式：`persist key  

​			移除给定 key 的生存时间，将这个 key 从『可挥发』的(带生存时间 key )转换成『持 久化』的(一个不带生存时间、永不过期的 key )。 

`可用版本：`  >= 2.2.0 

`时间复杂度：`  O(1) 

`返回值：`  当生存时间移除成功时，返回 1 . 如果 key 不存在或 key 没有设置生存时间，返回 0 。

```sh
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 10 # 为 key 设置生存时间
(integer) 1
redis> TTL mykey
(integer) 10
redis> PERSIST mykey # 移除 key 的生存时间
(integer) 1
redis> TTL mykey
(integer) -1
```

### sort（排序）

`格式：`sort key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]]  [ASC | DESC] [ALPHA] [STORE destination] 

​			返回或保存给定列表、集合、有序集合 key 中经过排序的元素。 

​			排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。 

`一般 SORT 用法`  

​			最简单的 SORT 使用方法是 SORT key 。 

​			假设 today_cost 是一个保存数字的列表， SORT 命令默认会返回该列表值的递增(从 小到大)排序结果。

```sh
# 将数据一一加入到列表中
redis> LPUSH today_cost 30
(integer) 1
redis> LPUSH today_cost 1.5
(integer) 2
redis> LPUSH today_cost 10
(integer) 3
redis> LPUSH today_cost 8
(integer) 4
# 排序
redis> SORT today_cost
1) "1.5"
2) "8"
3) "10"
4) "30
```

`当数据集中保存的是字符串值时，你可以用 ALPHA 修饰符(modifier)进行排序。`

```sh
# 将数据一一加入到列表中
redis> LPUSH website "www.reddit.com"
(integer) 1
redis> LPUSH website "www.slashdot.com"
(integer) 2
redis> LPUSH website "www.infoq.com"
(integer) 3
# 默认排序
redis> SORT website
1) "www.infoq.com"
2) "www.slashdot.com"
3) "www.reddit.com"
# 按字符排序
redis> SORT website ALPHA
1) "www.infoq.com"
2) "www.reddit.com"
3) "www.slashdot.com"
```

​			如果你正确设置了 !LC_COLLATE 环境变量的话，Redis 能识别 UTF-8 编码。 

​			排序之后返回的元素数量可以通过 LIMIT 修饰符进行限制。

​			 LIMIT 修饰符接受两个参数： offset 和 count 。

​			 offset 指定要跳过的元素数量， count 指定跳过 offset 个指定的元素之后，要返回 多少个对象。 

​			以下例子返回排序结果的前 5 个对象( offset 为 0 表示没有元素被跳过)。

```sh
# 将数据一一加入到列表中
redis> LPUSH rank 30
(integer) 1
redis> LPUSH rank 56
(integer) 2
redis> LPUSH rank 42
(integer) 3
redis> LPUSH rank 22
(integer) 4
redis> LPUSH rank 0
(integer) 5
redis> LPUSH rank 11
(integer) 6
redis> LPUSH rank 32
(integer) 7
redis> LPUSH rank 67
(integer) 8
redis> LPUSH rank 50
(integer) 9
redis> LPUSH rank 44
(integer) 10
redis> LPUSH rank 55
(integer) 11
# 排序
redis> SORT rank LIMIT 0 5 # 返回排名前五的元素
1) "0"
2) "11"
3) "22"
4) "30"
5) "32

#修饰符可以组合使用。以下例子返回降序(从大到小)的前 5 个对象。
redis> SORT rank LIMIT 0 5 DESC
1) "78"
2) "67"
3) "56"
4) "55"
5) "50"
```

使用外部 key 进行排序



# 3.SpringBoot整合redis

## module、pom、yaml

- 导入redis依赖

```xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
            <version>1.4.1.RELEASE</version>
        </dependency>
```



- 自定义配置文件

```yaml
------------自定义yml
spring:
  #redis配置信息
  redis:
    #redis 服务器地址
    hostName: 47.100.220.41
    #redis端口
    port: 6379
    #redis 密码
    password:
    #客户端超时时间单位是毫秒 默认是2000
    timeout: 5000
    #最大空闲数
    maxIdle: 20
    #连接池的最大数据库连接数
    maxActive: -1
    #控制一个pool可分配多少个jedis实例,用来替换上面的maxActive
    maxTotal: 100
    #最大建立连接等待时间。如果超过此时间将接到异常
    maxWaitMillis: 100
    #连接的最小空闲时间
    minEvictableIdleTimeMillis: 864000000
    #每次释放连接的最大数目
    numTestsPerEvictionRun: 10
    #逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程
    timeBetweenEvictionRunsMillis: 300000
    #是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
    testOnBorrow: true
    #在空闲时检查有效性
    testWhileIdle: false
    #数据库
    database: 0
```



- 编写配置类

```java
import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.RedisStandaloneConfiguration;
import org.springframework.data.redis.connection.jedis.JedisClientConfiguration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;
import redis.clients.jedis.JedisPoolConfig;

import java.nio.charset.Charset;

@Configuration
public class RedisConfig{

        @Value("${spring.redis.hostName}")
        private String hostName;

        @Value("${spring.redis.port}")
        private Integer port;

        @Value("${spring.redis.password}")
        private String password;

        @Value("${spring.redis.timeout}")
        private Integer timeout;

        @Value("${spring.redis.maxIdle}")
        private Integer maxIdle;

        @Value("${spring.redis.maxActive}")
        private String maxActive;

        @Value("${spring.redis.maxTotal}")
        private Integer maxTotal;

        @Value("${spring.redis.maxWaitMillis}")
        private Long maxWaitMillis;

        @Value("${spring.redis.minEvictableIdleTimeMillis}")
        private Long minEvictableIdleTimeMillis;

        @Value("${spring.redis.numTestsPerEvictionRun}")
        private Integer numTestsPerEvictionRun;

        @Value("${spring.redis.timeBetweenEvictionRunsMillis}")
        private Long timeBetweenEvictionRunsMillis;

        @Value("${spring.redis.testOnBorrow}")
        private boolean testOnBorrow;

        @Value("${spring.redis.testWhileIdle}")
        private boolean testWhileIdle;

        @Value("${spring.redis.database}")
        private Integer database;

        /**
         * JedisPoolConfig 连接池
         * @return
         */
        @Bean
        public JedisPoolConfig jedisPoolConfig() {
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            // 最大空闲数
            jedisPoolConfig.setMaxIdle(maxIdle);
            // 连接池的最大数据库连接数
            jedisPoolConfig.setMaxTotal(maxTotal);
            // 最大建立连接等待时间
            jedisPoolConfig.setMaxWaitMillis(maxWaitMillis);
            // 逐出连接的最小空闲时间 默认1800000毫秒(30分钟)
            jedisPoolConfig.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
            // 每次逐出检查时 逐出的最大数目 如果为负数就是 : 1/abs(n), 默认3
            jedisPoolConfig.setNumTestsPerEvictionRun(numTestsPerEvictionRun);
            // 逐出扫描的时间间隔(毫秒) 如果为负数,则不运行逐出线程, 默认-1
            jedisPoolConfig.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
            // 是否在从池中取出连接前进行检验,如果检验失败,则从池中去除连接并尝试取出另一个
            jedisPoolConfig.setTestOnBorrow(testOnBorrow);
            // 在空闲时检查有效性, 默认false
            jedisPoolConfig.setTestWhileIdle(testWhileIdle);
            return jedisPoolConfig;
        }

        /**
         * jedis连接工厂
         * @param jedisPoolConfig
         * @return
         */
        @Bean
        public RedisConnectionFactory redisConnectionFactory(JedisPoolConfig jedisPoolConfig) {
            RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
            //设置redis服务器的host或者ip地址
            redisStandaloneConfiguration.setHostName(hostName);
            redisStandaloneConfiguration.setPort(port);
            redisStandaloneConfiguration.setPassword(password);
            redisStandaloneConfiguration.setDatabase(database);
            //获得默认的连接池构造
            //这里需要注意的是，edisConnectionFactoryJ对于Standalone模式的没有（RedisStandaloneConfiguration，JedisPoolConfig）的构造函数，对此
            //我们用JedisClientConfiguration接口的builder方法实例化一个构造器，还得类型转换
            JedisClientConfiguration.JedisPoolingClientConfigurationBuilder jpcf = (JedisClientConfiguration.JedisPoolingClientConfigurationBuilder) JedisClientConfiguration.builder();
            //修改我们的连接池配置
            jpcf.poolConfig(jedisPoolConfig);
            //通过构造器来构造jedis客户端配置
            JedisClientConfiguration jedisClientConfiguration = jpcf.build();
            return new JedisConnectionFactory(redisStandaloneConfiguration, jedisClientConfiguration);
        }

        @Bean
        public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory){
            return RedisCacheManager.create(connectionFactory);
        }

        /**
         * 实例化 RedisTemplate
         * @param redisConnectionFactory
         * @return
         */
        @Bean(name = "redisTemplate")
        public RedisTemplate<String, Object> functionDomainRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
            RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
            redisTemplate.setConnectionFactory(redisConnectionFactory);
            // 使用Jackson2JsonRedisSerialize 替换默认序列化
            Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

            ObjectMapper objectMapper = new ObjectMapper();
            objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
            objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
            jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

            //设置value的序列化规则和 key的序列化规则
            RedisSerializer stringSerializer = new StringRedisSerializer(Charset.forName("UTF8"));
            redisTemplate.setKeySerializer(stringSerializer );
            redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
            redisTemplate.setHashKeySerializer(stringSerializer );
            redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
            redisTemplate.afterPropertiesSet();
            return redisTemplate;
        }
}

```



## 编写工具类(==工具类直接用==)

```java
@Service
public class RedisUtils {
    @Autowired
    private RedisTemplate redisTemplate;

    /**
     * 写入缓存
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 写入缓存设置时效时间
     *
     * @param key
     * @param value
     * @return
     */
    public boolean set(final String key, Object value, Long expireTime, TimeUnit timeUnit) {
        boolean result = false;
        try {
            ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
            operations.set(key, value);
            redisTemplate.expire(key, expireTime, timeUnit);
            result = true;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    /**
     * 批量删除对应的value
     *
     * @param keys
     */
    public void remove(final String... keys) {
        for (String key : keys) {
            remove(key);
        }
    }

    /**
     * 批量删除key
     *
     * @param pattern
     */
    public void removePattern(final String pattern) {
        Set<Serializable> keys = redisTemplate.keys(pattern);
        if (keys.size() > 0) {
            redisTemplate.delete(keys);
        }
    }

    /**
     * 删除对应的value
     *
     * @param key
     */
    public void remove(final String key) {
        if (exists(key)) {
            redisTemplate.delete(key);
        }
    }

    /**
     * 判断缓存中是否有对应的value
     *
     * @param key
     * @return
     */
    public boolean exists(final String key) {
        return redisTemplate.hasKey(key);
    }

    /**
     * 读取缓存
     *
     * @param key
     * @return
     */
    public Object get(final String key) {
        Object result = null;
        ValueOperations<Serializable, Object> operations = redisTemplate.opsForValue();
        result = operations.get(key);
        return result;
    }

    /**
     * 哈希 添加
     *
     * @param key
     * @param hashKey
     * @param value
     */
    public void hmSet(String key, Object hashKey, Object value) {
        HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
        hash.put(key, hashKey, value);
    }

    /**
     * 哈希获取数据
     *
     * @param key
     * @param hashKey
     * @return
     */
    public Object hmGet(String key, Object hashKey) {
        HashOperations<String, Object, Object> hash = redisTemplate.opsForHash();
        return hash.get(key, hashKey);
    }

    /**
     * 列表添加
     *
     * @param k
     * @param v
     */
    public void lPush(String k, Object v) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        list.rightPush(k, v);
    }

    /**
     * 列表获取
     *
     * @param k
     * @param l
     * @param l1
     * @return
     */
    public List<Object> lRange(String k, long l, long l1) {
        ListOperations<String, Object> list = redisTemplate.opsForList();
        return list.range(k, l, l1);
    }

    /**
     * 集合添加
     *
     * @param key
     * @param value
     */
    public void add(String key, Object value) {
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        set.add(key, value);
    }

    /**
     * 集合获取
     *
     * @param key
     * @return
     */
    public Set<Object> setMembers(String key) {
        SetOperations<String, Object> set = redisTemplate.opsForSet();
        return set.members(key);
    }

    /**
     * 有序集合添加
     *
     * @param key
     * @param value
     * @param scoure
     */
    public void zAdd(String key, Object value, double scoure) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        zset.add(key, value, scoure);
    }

    /**
     * 有序集合获取
     *
     * @param key
     * @param scoure
     * @param scoure1
     * @return
     */
    public Set<Object> rangeByScore(String key, double scoure, double scoure1) {
        ZSetOperations<String, Object> zset = redisTemplate.opsForZSet();
        return zset.rangeByScore(key, scoure, scoure1);
    }

}
```

