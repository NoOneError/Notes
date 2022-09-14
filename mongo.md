



## 1.安装mongodb

1.获取MongoDB镜像

```css
docker pull mongo:latest
```

2.运行容器

```css
docker run -p 27017:27017 -v /home/docker/mongodb:/data/db --name mongodb -d mongo
```

容器账户
 默认没有账户密码
 3.进入容器

```css
docker exec -it mongodb bash
```

无密码情况进入MongoDB的方式

4.如果是第一次使用MongoDB，首先先创建用户

```shell
> use admin
switched to db admin
> db.createUser({user:"admin", pwd:"123456", roles:[{role:"readWrite", db:"admin"}]});
Successfully added user: {
	"user" : "admin",
	"roles" : [
		{
			"role" : "readWrite",
			"db" : "admin"
		}
	]
}

role:"",是赋予用户什么权限
db:"admin",是为用户创建数据库

---------删除用户
db.dropUser("root")
```

> MongoDB权限介绍

|         权限         |                             说明                             |
| :------------------: | :----------------------------------------------------------: |
|         read         |                    允许用户读取指定数据库                    |
|      readWrite       |                    允许用户读写指定数据库                    |
|       dbAdmin        | 允许用户在指定数据库中执行管理函数，如索引创建、删除、查看统计或访问system.profile |
|      userAdmin       | 允许用户向system.users集合写入，可以在指定数据库中创建、删除和管理用户 |
|     clusterAdmin     | 必须在admin数据库中定义，赋予用户所有分片和复制集相关函数的管理权限 |
|   readAnyDatabase    |     必须在admin数据库中定义，赋予用户所有数据库的读权限      |
| readWriteAnyDatabase |    必须在admin数据库中定义，赋予用户所有数据库的读写权限     |
| userAdminAnyDatabase |  必须在admin数据库中定义，赋予用户所有数据库的userAdmin权限  |
|  dbAdminAnyDatabase  |   必须在admin数据库中定义，赋予用户所有数据库的dbAdmin权限   |
|         root         |         必须在admin数据库中定义，超级账号，超级权限          |

## 2.mongodb基础命令

进入数据库
 `mongo -u 用户名 -p 密码`
 显示数据库
 `show dbs`
 选择和创建数据库
 `use 数据库名`
 数据库名不存在自动创建新数据库
 显示表(集合)
 `show collections`

修改集合名字

`db.getCollection("").renameCollection("")`

删除集合

`db.getCollection("").drop()`

批量删除集合

```shell
db.getCollectionNames().forEach(function(c){
//选择部分规律，例如集合名字带xxxx
	if(c.match("xxxx")){
		db.getCollection("").drop()
	}
});
```

 插入文档
 `db.集合名称.insert(数据)`
 查看表下文档数据
 `db.集合名称.find()`
 按条件查询(参数为json)
 `db.集合名称.find(参数)`
 返回指定条数记录
 `db.集合名称.find().limit(条数)`
 修改文档(条件和修改后的数据为json)
 `db.集合名称.update(条件,{$set:修改后数据})`
 删除文档(条件为json)
 `db.集合名称.remove(条件)`
 删除全部文档
 `db.集合名称.remove({})`
 统计条数
 `db.集合名称.count()`
 按条件统计条数
 `db.集合名称.count(条件)`

## 3.SpringBoot整合mongodb

1、引入相关依赖

```java
 <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/> <!-- 比较新的版本有出入 -->
    </parent>    

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

2、配置数据库连接信息

```yaml
spring:    ------自定义
  mongo:
    host: 47.100.220.41
    username: admin
    password: 123456
    port: 27017
    authentication-database: admin
```

3、springboot连接mongo自定义配置

- 读取配置信息

```java
@Data
@Component
@ConfigurationProperties(prefix = "spring.mongodb")
public class MongodbProperties {
    private String host;
    private int port;
    private String username;
    private String password;
    private String database;
}
```

- 自定义mongoTemplate

```java
import com.mongodb.MongoClient;
import com.mongodb.MongoClientOptions;
import com.mongodb.MongoCredential;
import com.mongodb.ServerAddress;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.mongodb.MongoDbFactory;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.SimpleMongoDbFactory;

@Configuration
public class MongodbConfig {

    @Autowired
    MongodbProperties mongodb;

    @Bean
    public MongoClient mongoClient(){
        //配置连接mongo客户端，一共三个参数
        //主机端口，账号密码，超时时间等的配置
        return new MongoClient(new ServerAddress(mongodb.getHost(),mongodb.getPort()),
                MongoCredential.createCredential(mongodb.getUsername(),mongodb.getDatabase(),mongodb.getPassword().toCharArray()),
                MongoClientOptions.builder()
                        .socketTimeout(3000)
                        .minHeartbeatFrequency(25)
                        .heartbeatSocketTimeout(3000)
                        .build());
    }

    @Bean
    public MongoDbFactory mongoDbFactory(){
        //注册一个MongoDbFactory，连接到指定数据库
        return new SimpleMongoDbFactory(mongoClient(),"admin");
    }
    @Bean
    public MongoTemplate mongoTemplate(){
        //将MongoDbFactory作为参数，注册MongoTemplate
        return new MongoTemplate(mongoDbFactory());
    }

}
```

## 4.CRUD

==SpringBoot操作mongodb有两种方式，mongoTemplate 和 MongoRepository==

创建数据类

```java
@Data
@Document("old_user") //指定mongodb存储c
public class User {
    @Id
    private Long id;

    private String username;

    private String password;
}
```



1、**MongoRepository**

注意`MongoRepository`后面接的泛型`<User, String>`第一个为实体类，第二个为主键`ID`。

```java
@Repository
public interface UserRepository extends MongoRepository<User, String> {
}
```

在controller中操作如下:

```java
@RestController
@RequestMapping("/user")
public class UserController {
    
    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @GetMapping("")
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }

    @GetMapping("/{userId}")
    public User getByUserId(@PathVariable String userId) {
        return userRepository.findById(userId).orElse(new User());
    }

    @PostMapping("")
    public User addNewUser(@RequestBody User user) {
        return userRepository.save(user);
    }

    @DeleteMapping("/{userId}")
    public String delete(@PathVariable String userId) {
        User user = new User();
        user.setUserId(userId);
        userRepository.deleteById(userId);
        return "deleted: " + userId;
    }

    @PutMapping("")
    public User update(@RequestBody User user) {
        return userRepository.save(user);
    }
}
```



2、==mongoTemplate==

mongoTemplate（springBoot会自动配置**自动配置需要注意配置文件的前缀，spring.data.mongodb**，也可以手动配置），上面已经手动配置了！

```java
    @Autowired
    private MongoTemplate template;

    @Override
    public List<User> findAll() {
        return template.findAll(User.class);
    }

    @Override
    public User findById(String userId) {
        return template.findById(userId,User.class);
    }

    @Override
    public User save(User user) {
        return template.save(user);
    }

    @Override
    public void deleteById(String userId) {
        Query query = new Query();
        query.addCriteria(Criteria.where("userId").is(userId));
        template.remove(query, User.class);
    }

 	@Override
    public void updateUser(User user) {
        //原来的数据
        Query query = new Query(Criteria.where("_id").is(user.getId()));
        //修改之后的数据
        Update update = new Update();
        update.set("url", user.getPort());
        update.set("name", user.getName());
        update.set("remarks", user.getK());
        mongoTemplate.updateFirst(query, update, User.class);
        mongoTemplate.updateMulti(query, update, User.class); //批量
        
        mongoTemplate.upsert(user);//没有
    }

```



## 5、复杂查询

查每个人，成绩最好的分数（）类似

```java
/**
     * 复杂一点的查询。
     * 分组，排序，过滤
     */
    //object是泛型，是定义的数据库映射类
    private List<Object> getArchiveDataList() {
        TypedAggregation<Object> tagg = TypedAggregation.newAggregation(Object.class,
                Arrays.asList(
                        //筛选条件 过滤条件 这里举例了通过'createTime'进行范围查询
                        //后面atTime的作用是MongoDB的时区不一样，时区+8小时处理
                        TypedAggregation.match(Criteria.where("createTime").is("")),

                        //分组过滤条件，sort的顺序有要求，在分组方法前就是先排序后分组，反之道理一样
                        TypedAggregation.sort(Sort.by(Sort.Order.desc("createTime"))),
                        //排序字段 通过userId进行分组，意义：取某个人的最新一条数据，first，as里最后包含展示的字段
                        //first和as的作用是 first为MongoDB查询出的列值，as为将值放到类成员变量
                        // 中，如果不使用任何的first和as，那么返回的映射类信息所有成员变量都是null
                        TypedAggregation.group("userId")
                                .first("userId").as("userId")
                                .first("name").as("name")


                        //挑选需要字段
//                        TypedAggregation.project("activeCode", "startDate"),
                )
        );

        AggregationResults<Object> result = mongoTemplate.aggregate(tagg, Object.class);

        return result.getMappedResults();
    }

```

```java

//分页
private Page<Object> page(List<Object> list,Integer pageNum,Integer pageSize){
    PageRequest pageRequest = PageRequest.of(pageNum,pageSize,Sort.by(Sort.Direction.DESC,"排序的字段"));
     int totalElements = list.size();
            if(pageable==null)
            {
                pageable = PageRequest.of(0,10);
            }
            int fromIndex = pageable.getPageSize()*pageable.getPageNumber();
            int toIndex = pageable.getPageSize()*(pageable.getPageNumber()+1);
            if(toIndex>totalElements) toIndex = totalElements;
　　　　　　　//如果list的内容取回后基本不变化，可以考虑放入类对象变量里或者其他缓存里，然后判断list是否可用，不用每次都去取list，适合取数的时候无法真分页只能伪分页情况。
            //类似取mysql数据库的情况，因为数据取数本身支持分页，所以list的数据每次都取当前页就可以，但是需要先要计算所有记录数，然后传入totalElements变量
            List<ESIndexObject> indexObjects = list.subList(fromIndex,toIndex);#获取当前页数据
            Page<ESIndexObject> page = new PageImpl<>(indexObjects,pageable,totalElements);
    return page;
}
```

## 6、自定义查询

详细见 elasticSearch的自定义查询章节，都是spring datade
