# 1.基本介绍

## 1.1概念

<img src="https://upload-images.jianshu.io/upload_images/1616232-320d2db914ffd5e3.png?imageMogr2/auto-orient/strip|imageView2/2/w/859/format/webp" alt="img" style="zoom:50%;" />

## 1.2作用

SpringDataJPA 则是致力于减少数据访问层的开发量，开发者唯一要做的就是声明持久层的接口，其他都交给SpringDataJPA来帮你完成。



# 2.使用方法

## 2.1导包

```
<dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--SpringDataJpa的依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>2.2.6.RELEASE</version>
        </dependency>
        <!--mysql的依赖-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- hibernate对jpa的支持包 -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.0.7.Final</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
```



## 2.2建立映射关系

**关键注解：**

- @Table -- 建立实体和表的映射
- @Column -- 建立实体属性和表中的列的关系

**代码：**

```
@Entity
@Table(name = "cst_customer")
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId; //客户的主键

    @Column(name = "cust_name")
    private String custName;//客户名称

    @Column(name="cust_source")
    private String custSource;//客户来源

    @Column(name="cust_level")
    private String custLevel;//客户级别

    @Column(name="cust_industry")
    private String custIndustry;//客户所属行业

    @CreatedDate
    private Date createdDate;
```



## 2.3简单条件查询

### 2.3.1直接用JPA封装好的方法

**（1） JpaRepository封装的方法**

​		save（）方法

**（2）JpaSpecificationExecutor封装的方法**



### 2.3.2遵循规范、只写接口

​	   规范：主要针对查询。在**xxxxRepository**中，按照约定的规范，可以只写查询方法的接口，而不用写实现。

- 查询某一个实体类或者集合，按照 Spring Data 的规范，查询方法以 find | read | get 开头， 涉及条件查询时，条件的属性用条件关键字连接。要注意的是：条件属性以首字母大写。

​	   举例，根据custName查询Customer的信息，返回List -- findByCustName（String name）：

​		注意：这种方式写的查询方法，默认返回的都是集合（List、Set）

```java
public interface CustomerRepository extends JpaRepository<Customer,Long> ,JpaSpecificationExecutor<Customer> {

    //根据custName查询，返回List
    List<Customer> findByCustName(String name);

    //根据创建日期查询，返回list
    List<Customer> findByCreatedDateAfter(Date dateTime);
}
```



- **关键字**

  ![img](https://upload-images.jianshu.io/upload_images/1616232-5ad4318b907ddfec.png?imageMogr2/auto-orient/strip|imageView2/2/w/825/format/webp)

​    ![img](https://upload-images.jianshu.io/upload_images/1616232-0fafa549fce95f63.png?imageMogr2/auto-orient/strip|imageView2/2/w/843/format/webp)

### 2.3.3@Query自定义语句

**（1）JPQL查询和原生SQL查询**

​        默认使用JPQL查询：

```java
	@Query("select c from Customer c where c.custPhone = ?1 and "
        + "c.custLevel = ?2")
	List<Customer> findByPhoneAndCustLevel(String phone, String level);
```

​		加上：**nativeQuery = true**，使用原生SQL查询：

```sql
   @Query(value = "select * from cst_customer c where c.cust_phone = ?1 and "
           + "c.cust_level = ?2", nativeQuery = true)
   List<Customer> findByPhoneAndCustLevel(String phone, String level);
}
```

**（2）自定义查询语句**

​	如（1）中所示

**（3）自定义删除、修改语句**

单独使用@Query注解只是查询，如涉及到修改、删除则需要再加上**@Modifying**注解

注意：Modifying queries can only use void or int/Integer as return type

**修改语句 -- 返回int：**

```java
@Transactional()
@Modifying
@Query("update Customer c set c.custSource = ?1 where c.custName = ?2")
int updateCustSourceByCustName(String custSource, String custName);
```

**删除语句 -- 返回void：**

```java
@Transactional()
@Modifying
@Query("delete Customer c where c.custId = ?1")
void deleteCustomerByCustId(long custId);
```

## 2.4分页查询

使用方法：   

①CustomerRepository -- 多传一个Pageable:

```java
@Query("select c from Customer c where c.custName like %?1%")
Page<Customer> findByUserNameLike(String custName, Pageable pageable);
```

② 构造一个PageRequest（Pageale的实现类），返回一个Page

```java
@Test
public void test11(){
    PageRequest pageRequest = PageRequest.of(0, 5, Sort.Direction.DESC, "custName");
    //也可以这样构造 pageRequest
    //PageRequest pageRequest = PageRequest.of(0, 5, Sort.by(Sort.Direction.DESC, 		
    							//"custName"));
    Page<Customer> page = customerRepository.findByUserNameLike("", pageRequest);
}
```



## 2.5多表查询

![image-20200601190522238](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20200601190522238.png)

## 2.6动态查询

使用JpaSpecification( ) 进行动态查询，核心：构造specification -- 实现toPredict( ) 方法

```java
@Test
    public void test12(){
        //构造Specification对象
        Specification<Customer> spec = (Specification<Customer>) (root, criteriaQuery, criteriaBuilder) -> {
            //查询属性
            Path<Object> custName = root.get("custName");
            Path<Object> custSource = root.get("custSource");
            //查询方式(模糊匹配)
            Predicate like = criteriaBuilder.like(custName.as(String.class), "a%");
            Predicate boss = criteriaBuilder.equal(custSource, "boss");
            Predicate and = criteriaBuilder.and(like, boss);
            return and;
        };
        //构造Sort对象
        Sort sort = Sort.by(Sort.Direction.DESC, "custName"); 
        //构造分页对象
        PageRequest pageRequest = PageRequest.of(0, 2, sort);
        //打印查询结果
        Page<Customer> all = customerRepository.findAll(spec, pageRequest);
        //System.out.println(all);
    }
```

