关系型数据库和SQL一直是数据持久化领域的首选方案。尽管近年来涌现了许多可选的数据库类型，但是关系型数据库依然是通用数据存储的首选，而且短期内不太可能撼动它的地位。 

在处理关系型数据的时候，Java开发人员有多种可选方案，其中最常见的是JDBC和JPA。现在实际业务开发中大多数人使用 myatis框架，当然spring data JPA使用的也不少。尽管JDBC和spring JdbcTemplate实际开发为了效率不会使用，但仍有必要了解。

# jdbcTemplate入门

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```

## 例子

### controller

```java
@RestController
@RequestMapping("category")
public class CategotyController {

    @Resource
    private CategoryService categoryService;

    @GetMapping("/{id}")
    public Category findOne(@PathVariable("id") int id){
        return categoryService.findOne(id);
    }

    @GetMapping("/all")
    public Iterable<Category> findAll(){
        return categoryService.findAll();
    }
}
```

### service

```java
@Service
public class CategoryService {
    @Autowired
    private JdbsCategaryRepository categaryRepository;

    public Iterable<Category> findAll(){
        return categaryRepository.findAll();
    }

    public Category findOne(int id){
        return categaryRepository.findOne(id);
    }
}
```

### dao

dao层接口

```java
public interface CategoryRepository {
    Iterable<Category> findAll();

    Category findOne(int id);
}
```

持久层实现

在applicaiton.yml配置完成mysql数据源之后，在Repository实现类中，直接注入JdbcTemplate。

JdbcTemplate提供了操作数据表的接口，用户将原生sql语句传给jdbc即可。注意sql中字段最好写驼峰格式，从结果集中获取数据也不要用驼峰字段，表明和字段名都和数据库保持一致。

```java
@Repository
public class CategaryRepositoryImpl implements CategoryRepository {

    @Resource
    private JdbcTemplate jdbc;

    @Override
    public Iterable<Category> findAll() {
        return jdbc.query("select * from ms_category", this::mapRowToCategory);
    }

    @Override
    public Category findOne(int id) {
        return jdbc.queryForObject("select id, avatar, category_name, description from ms_category where id=?",
                this::mapRowToCategory, id);
    }

    private Category mapRowToCategory(ResultSet rs, int rowNum) throws SQLException {
        return new Category(rs.getInt("id"), rs.getString("avatar"),
                rs.getString("category_name"), rs.getString("description"));
    }
}
```

关于JdbcTemplate的接口用法，如果你真的要用其做项目，再去详细了解即可。一般项目中我们都会使用下面的 JPA 或者 mybatis 。

### pojo

```java
@Data
@AllArgsConstructor
public class Category {
    private int id;

    private String avatar;

    private String categaryName;

    private String description;
}
```

# spring data JPA

## 入门

Spring Data是一个非常大的伞形项目，由多个子项目组成，其中大多数子项目都关注对不同的数据库类型进行数据持久化。比较流行的几个Spring Data项目包括： 

- Spring Data JPA：基于关系型数据库进行JPA持久化。 

- Spring Data MongoDB：持久化到Mongo文档数据库。 

- Spring Data Neo4j：持久化到Neo4j图数据库。 

- Spring Data Redis：持久化到Redis key-value存储。 

- Spring Data Cassandra：持久化到Cassandra数据库。

spring data 系列中，我们做增删改查业务一般用 JPA 最多，故本文只会介绍 JPA。

### 依赖

直接在pom.xml中引入jpa的starter

```xml
<dependency> 
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-data-jpa</artifactId> 
</dependency>
```

使用

为了将Ingredient声明为JPA实体，它必须添加@Entity注解。它的id 属性需要使用@Id注解，以便于将其指定为数据库中唯一标识该实体的属性。

### 实体类

若使用JPA，和数据库表映射的实体类需要加几个注解。先看代码，后面解释必须的几个注解的作用。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor(access= AccessLevel.PRIVATE, force=true)
@Entity
@Table(name="ms_category")
public class JpaCategory {
    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private String id;

    private String avatar;

    @NotNull
    @Size(min=5, message="Name must be at least 2 characters long")
    private String categoryName;

    private String description;
}
```

实体类中必须使用以下几个注解

- @Entity：为了将JpaCategory声明为JPA实体，它必须添加@Entity注解。
- @Id：实体类的id 属性需要使用@Id注解，以便于将其指定为数据库中唯一标识该实体的属性。
- @Table： 表明 JpaCategory 实体应该持久化到数据库中名为 ms_category 的表中。如果没有它，JPA默认会将实体持久化到名为 JpaCategory 的表中。

@GeneratedValue是为了自动生成 id，这个注解不必加，我们可以自动生成id，比如使用UUID。

@AllArgsConstructor等是Lombok提供的注解，可以少写javabean基本方法。

@NotNull等是validation相关的注解。

### Repository

创建CategoryRepository extends CrudRepository后，即可实现单表的基本查询或修改。CrudRepository定义了很多用于CRUD（创建、读取、更新、删除）操作的方法。注意，它是参数化的，第一个参数是repository要持久化的实体类型，第二个参数是实体ID属性的类型。对于 CategoryRepository 来说，参数应该是JpaCategory和String。 

```java
public interface CategoryRepository extends CrudRepository<JpaCategory, String> {
}
```

Spring Data JPA带来的好消息是，我们根本就不用编写实现类！当应用启动的时候，Spring Data JPA会在运行期自动生成实现类。这意味着，我们可以直接使用这些repository的方法，只需要像使用基于JDBC的实现那样将它们注入控制器中就可以了。

## 例子

我们使用 JPA 改写JdbcTemplate中的例子，controller层的简单代码省略掉。

### service

在service层注入CategoryRepository，可以调用 findById()  这些JPA自带的方法。

```java
@Service
public class JpaCategoryService {
    @Resource
    private CategoryRepository categoryRepository;

    public Iterable<JpaCategory> findAll() {
        return categoryRepository.findAll();
    }

    public JpaCategory findOne(String id) {
        Optional<JpaCategory> optional = categoryRepository.findById(id);
        return optional.orElse(null);
    }

    public Iterable<JpaCategory> findByCategoryName(String categoryName) {
        return categoryRepository.findByCategoryName(categoryName);
    }
}
```

### dao

```java
public interface CategoryRepository extends CrudRepository<JpaCategory, String> {
}
```

## sql实现

### 方法名生成sql

```java
    /**
     *  自动生成单表查询
     */
    Iterable<JpaCategory> findByCategoryName(String categoryName);
```

### JPA书写sql

#### 查询

尽管方法名称约定对于相对简单的查询非常有用，但是，不难想象，对于更为复杂的查询，方法名可能会面临失控的风险。在这种情况下，可以将方法定义为任何你想要的名称，并为其添加@Query注解，从而明确指明方法调用时要执行的查询。

jpa在sql使用 : 引用方法参数

```java
public interface CategoryRepository extends CrudRepository<JpaCategory, String> {
    @Query("select po from JpaCategory po where po.categoryName=:categoryName")
    Iterable<JpaCategory> findByCategoryName2(String categoryName);
}
```

也可以为参数指定名字

```java
    @Query("select po from JpaCategory po where po.categoryName=:categoryName")
    Iterable<JpaCategory> findByCategoryName2(String categoryName);
```

#### 更新

jpa中使用自定义更新或删除语句必须加 @Modifying 和 @Transactional，否则会报错。@Modifying表示@Query的sql是更新，@Transactional保证事务的一致性。

```java
    @Transactional //事务的注解
    @Modifying
    @Query("update JpaCategory n set n.description=:description where n.id =:id")
    int updateIpaCategory(String id,String description);
```

### 原生sql

JPA中当然也支持书写原生sql，只需设置nativeQuery = true即可。

```java
    @Query(value="select * from ms_category c where c.category_name=:name",
    nativeQuery = true)
    Iterable<JpaCategory> findByCategoryName4(@Param("name") String categoryName);
```

### sql引用参数

上面我们看到了sql用 : 符号获取方法的参数值，不过这种只能获取 int  String Iteger这种简单的参数。对应对象的参数，无论是JPA封装后的sql，还是原生sql，都使用 #{#Object.field} 获取对象的属性。

```java
@Query(value="select * from ms_category where description = :#{#condition.description} and category_name = :#{#condition.categoryName}",nativeQuery=true )
JpaCategory selectBySortNameAndParendId(@Param("condition") Condition condition);
```

