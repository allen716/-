

# 1. 理论介绍

## 1.1 概述

它是Spring基于**ORM框架**、**JPA规范**封装的一套持久层框架，可使开发者用极简的代码即可实现对持久层的操作（数据库的访问、操作）。

**关键词：**ORM，JPA，持久层框架

## 1.2 orm思想

### 1.2.1 概念

概念orm: Object Relational Mapping -- 对象关系映射

每张数据的表 对应 一个Java实体类

### 1.2.2举个栗子

customer这张表：

![image-20200816172333198](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20200816172333198.png)

**核心：**把对数据库的操作转化成对对象的操作

**orm的优势：**不用sql直接编码，能够像操作对象一样从数据库获取数据  

​						-- 项目组成员的例子

## 1.3 JPA

JPA：Java Persistence API

**本质：**是一个持久层的规范（仅仅定义了一些接口），Hibernate3.2+、TopLink 10.1.3以及OpenJPA都提供了JPA的实现

![img](https://pic1.zhimg.com/80/v2-d1ae2c30ef5be40eea5c96b6a638984d_720w.jpg?source=1940ef5c)

SpringDataJpa是用Hibernate来实现

**使用：**按照约定好的【方法命名规则】写dao层接口，就可以在不写接口实现的情况下，实现对数据库的访问和操作

# 2.层次结构

## 2.1 继承树

<img src="https://upload-images.jianshu.io/upload_images/1616232-320d2db914ffd5e3.png?imageMogr2/auto-orient/strip|imageView2/2/w/859/format/webp" alt="img" style="zoom:50%;" />

## 2.2 内部封装

CrudRepository:

![image-20200816180913890](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20200816180913890.png)

PagingAndSortingRepository：

![image-20200816181034373](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20200816181034373.png)



# 3.使用场景

## 3.0 准备工作

- 导包

- 实体关联对应的表：

<img src="https://iweb.zh.cmbchina.com/getIwebImage.action?fileid=A3C8EF55603E929FA5A8DA166D3942FA2203740686_IMAGE&amp;length=307501&amp;name=1597576463802.JPEG&amp;thumbId=8966000479FF010FA71EC296C94A4ADD2203740686_THUMB&amp;thumbSize=7632&amp;openid=E4774BB0EACC31F95543C56E0F1842BF&amp;email=uici8lf3ays3Rnz766sk8iVcPPisvd7TsOxpDDPIdCgGBDnAmUumU4zRV1mhtGIXJV4ejy149oRoFOpUkZVFDA%3D%3D&amp;encry=1" alt="下载" style="zoom:67%;" />

- 声明JPA

  ```java
  public interface BranchJpaRepository extends JpaRepository<BranchEntity, Long>
  ```

    将JPA--实体---数据库表 关联起来

## 3.1 简单条件查询

场景：单表查询

### 3.1.1 只写接口不用写具体实现

- 在**xxxxRepository**中，按照约定的规范，可以只写查询方法的接口，而不用写实现。


- 查询某一个实体类或者集合，按照 Spring Data 的规范，查询方法以 find | read | get 开头， 涉及条件查询时，条件的属性用条件关键字连接。要注意的是：条件属性以首字母大写。

​		注意：这种方式写的查询方法，默认返回的都是集合（List、Set）

- 举例说明：

```java
public interface BranchJpaRepository extends JpaRepository<BranchEntity, Long> {   	 		List<BranchEntity> findByReleaseJourneyIdAndReleaseUnitId(String releaseJourneyId, 			String releaseUnitId);
 }
```

- **关键字说明**

  ![img](https://upload-images.jianshu.io/upload_images/1616232-5ad4318b907ddfec.png?imageMogr2/auto-orient/strip|imageView2/2/w/825/format/webp)

​    ![img](https://upload-images.jianshu.io/upload_images/1616232-0fafa549fce95f63.png?imageMogr2/auto-orient/strip|imageView2/2/w/843/format/webp)

### 3.1.3@Query自定义语句

**（1）原生SQL查询** ：

​		加上：**nativeQuery = true**，使用原生SQL查询：

```sql
   @Query(value = "select * from cst_customer c where c.cust_phone = ?1 and "
           + "c.cust_level = ?2", nativeQuery = true)
   List<Customer> findByPhoneAndCustLevel(String phone, String level);
}
```

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

## 3.2 分页查询

- 使用方法：   

  ①CustomerRepository -- 多传一个Pageable:

```java
@Query("select c from Customer c where c.custName like %?1%")
Page<Customer> findByUserNameLike(String custName, Pageable pageable);
```

​		② 构造一个PageRequest（Pageale的实现类），返回一个Page

```java
@Test
public void test11(){
    PageRequest pageRequest = PageRequest.of(0, 5, Sort.Direction.DESC);
    //查询userName包含"a"的记录，并分页展示：从第0页开始，每页5条
    Page<Customer> page = customerRepository.findByUserNameLike("a", pageRequest);
}
```

**打印SQL：**

```
Hibernate: select customer0_.cust_id as cust_id1_0_, customer0_.cust_address as cust_add2_0_, customer0_.cust_industry as cust_ind3_0_, customer0_.cust_level as cust_lev4_0_, customer0_.cust_name as cust_nam5_0_, customer0_.cust_phone as cust_pho6_0_, customer0_.cust_source as cust_sou7_0_ from cst_customer customer0_ where customer0_.cust_name like ? order by customer0_.cust_name desc limit ?

Hibernate: select count(customer0_.cust_id) as col_0_0_ from cst_customer customer0_ where customer0_.cust_name like ?
```

## 3.3 动态查询

```
JpaSpecificationExecutor --- 通过构造的条件进行查询
```

（1）自定义查询条件

- 实现Specification接口

  ![image-20200817175149059](C:\Users\kx\AppData\Roaming\Typora\typora-user-images\image-20200817175149059.png)

- 实现toPredicate方法

  root：获取需要查询的对象属性

  criteriaBuilder：构造查询条件

（2）举例说明

查询：custName以 'a' 开头，且custSource = 'boss' 的 记录

```java
public void testSpecification() {
	//构造Specification对象
        Specification<Customer> spec = (Specification<Customer>) (root, criteriaQuery, 				criteriaBuilder) -> {
            //查询属性
            Path<Object> custName = root.get("custName");
           	Path<Object> custSource = root.get("custSource");
            //查询条件
           	Predicate like = criteriaBuilder.like(custName.as(String.class), "a%");
           	Predicate boss = criteriaBuilder.equal(custSource, "boss");
           	Predicate and = criteriaBuilder.and(like, boss);
            return and;
       };
	Customer customer = customerRepository.findOne(spec);
}
```

## 3.4 多表查询

- **场景举例：**

![下载](https://iweb.zh.cmbchina.com/getIwebImage.action?fileid=9F2787F04F3D06D316F4BE3DBB28405F2203740686_IMAGE&length=30469&name=1597583936250.JPEG&thumbId=BADA683787636AA7D5D9E8CDB8404FBA2203740686_THUMB&thumbSize=2487&openid=E4774BB0EACC31F95543C56E0F1842BF&email=uici8lf3ays3Rnz766sk8iVcPPisvd7TsOxpDDPIdCgGBDnAmUumU4zRV1mhtGIXJV4ejy149oRoFOpUkZVFDA%3D%3D&encry=1)

```java
public class Living {
    Long id;

    @ManyToOne
    @JsonIgnore
    @JoinColumn(name = "actorId", foreignKey = @ForeignKey(name = "none", value 				=ConstraintMode.NO_CONSTRAINT))
    public Actor actor;

    @ManyToOne
    @JsonIgnore
    @JoinColumn(name = "regionId", foreignKey = @ForeignKey(name = "none", value 				=ConstraintMode.NO_CONSTRAINT))
    public Region region;

}
```

- 构造条件，查询出满足条件的living：

- 这里join的精髓就是,获取join对象，通过join对象进行查询条件的构建

   根据userdetai 中的 sex，actor中的actortype，还有 region的id 进行关联

```java

public void testJoin() {
	Specification<Living> specification = new Specification<Living>() {
	@Override
    public Predicate toPredicate(Root<Living> root, CriteriaQuery<?> query, 		
                                 CriteriaBuilder cb) {
        List<Predicate> list = new ArrayList<Predicate>();

        if (null!=sex) {
            Join<UserDetail, Living> join = root.join("actor", JoinType.LEFT);
            list.add(cb.equal(join.get("userDetail").get("sex"),  sex ));
        }

        if (null!=actortype) {
            Join<Actor, Living> join = root.join("actor", JoinType.LEFT);
            list.add(cb.equal(join.get("actorType"),  actortype));
        }
        if (null!=cityid) {
            Join<Region, Living> join = root.join("region", JoinType.LEFT);
            list.add(cb.equal(join.get("id"), cityid));
        }

        //Join<A, B> join = root.join("bs", JoinType.LEFT);
        //list.add(cb.equal(join.get("c").get("id"), id));
        Predicate[] p = new Predicate[list.size()];
        return cb.and(list.toArray(p));
    };
};
    //根据specification查询
	Living living = livingRepository.findAll(specification);

```

- **对比：SQL写连接查询**

```
@Query("select new 		
	com.cmbchina.ucm.project.adapter.repository.db.entity.ReleaseUnitSyncEntity("    
+ "r.id, r.groupId, r.artifactId, r.owner, rt) "    
+ "from com.cmbchina.ucm.projectset.adapter.repository.db.entity.ProjectSetEntity pset " + "join com.cmbchina.ucm.project.adapter.repository.db.entity.ActiveReleaseJourneyEntity 		pe "    
+ "on pset.id = pe.prdProjectSetId and pset.id = :projectSetId "    
+ "join com.cmbchina.ucm.project.adapter.repository.db.entity
       .ReleaseJourneyVsReleaseUnitSyncEntity pvr "
+ "on pe.id = pvr.releaseJourneyId "
+ "join com.cmbchina.ucm.project.adapter.repository.db.entity.ReleaseUnitSyncEntity r "    + "on pvr.releaseUnitId = r.id "
+ "left join ReleaseUnitTypeEntity rt on r.id = rt.id "
+ "group by r.id")
Set<ReleaseUnitSyncEntity> findReleaseUnitsByProjectSetId(@Param("projectSetId") String projectSetId);
```

- **复杂查询的小结：**

  （1）许多人多jpa 有很大的误解，认为jpa 的多表、多条件复杂查询，不如mybatis的查询

  （2）其实通过jpa 实现了这个多表多条件的复杂查询之后，我觉得JPA在这方面不逊于mybatis，直接通过操作对象即可，省去了些SQL语句的麻烦，还可以减少出错 

# 4.小心避雷

## 4.1 jpa缓存导致无法查询到更新后的数据

举个栗子：对某条记录做了查询，然后又对某个字段单独进行更新，更新后立即查询更新后的结果，查询的数据是更新前的。这个问题是由于JPA的缓存导致的。

解决方法：通过@Modifying注解的clearAutomatically（设置为true）清除缓存，这样更新后JPA就只能去数据库获取数据。但是clearAutomatically会对性能有一定的影响，所以clearAutomatically属性应该视情况使用。示例如下：

```sql
  @Modifying(clearAutomatically = true)
  @Query("update User set name = :name where id = :id")
  int updateUserNameById(@Param("name") String name,@Param("id") Long id);
```

## 4.2 双向关联导致递归死循环

​	双向关联查询数据出现死循环、栈溢出的问题

​	（1）**举个栗子**：

![下载](https://iweb.zh.cmbchina.com/getIwebImage.action?fileid=9CABDB25F4634ED67DF2E8FE34FC4F402203740686_IMAGE&length=10684&name=1597651423366.JPEG&thumbId=8285BB7101E78352FB73244D6885495E2203740686_THUMB&thumbSize=2044&openid=E4774BB0EACC31F95543C56E0F1842BF&email=uici8lf3ays3Rnz766sk8e%2FBZyyTM37lkOpzGEjpLtX4%2BMux%2F6Eo%2FcBwg5B1xdJTwKHLsV%2BGBbL4qq%2FYT0ocPg%3D%3D&encry=1)

```java
@Data 
@Entity 
@Builder 
@Table(name = "ucm_schedule") 
@NoArgsConstructor 
@AllArgsConstructor 
public class ScheduleEntity { 
   //属性
   //.......
    @OneToMany 
    @JoinColumn(name = "schedule_id", referencedColumnName = "id", insertable = false, updatable = false) 
    private List<ProjectExtendEntity> projectList; //1个排期对应多个项目
 
    public static ScheduleEntity create(String name) { 
        return ScheduleEntity.builder() 
            .id(UuidUtils.newUuid()) 
            .name(name) 
            .team(ImmutableList.of(DEFAULT_TEAM)) 
            .build(); 
    } 
} 
```

```
@Data 
@Entity 
@Builder 
@Table(name = "ucm_project_sync") 
@NoArgsConstructor 
@AllArgsConstructor 
public class ProjectExtendEntity { 
   //属性
   //.......
    @ManytoOne 
    @JoinColumn(name = "id", referencedColumnName = "schedule_id") 
    private ScheduleEntity schedule; //1个项目对应1个排期
} 
```

（2）**原因分析：**循环调用toString( )方法，导致死循环

（3）**解决办法：**

方式一：

​	只在一边进行管联。

​	如：本例中，只在多的一方使用 @OneToOne

方式二：

​	将@Data注解换成@getter或者@setter方法，不让lombok帮我们自动重写toString( )方法，或者自己重写	

​     toString( )方法

# 5. 优势/劣势

SpringDataJpa和MyBatis对比：

优势：

（1）熟练之后，Spring data jpa可以明显加快开发速度

（2）缓存的优化：有着三级缓存，一级缓存是默认开启的，二级缓存需要手动开启以及配置优化，三级缓存可以		整合业界流行的缓存技术 redis，ecache 等等一起去实现
（3）关联查询的懒加载

（4）扩展性更强。对象的维护更好，对增删改查的对象的维护要方便。

劣势：

（1）不如mybatis 灵活，mybatis 修改起来更加方便

（2）JPA的使用门槛比较高

（3）Mybatis可以进行更细致的SQL优化：查询所需字段。

​		SpringDataJpa底层使用Hibernate，它的查询会将表中的所有字段查询出来，这一点会有性能消耗

（4）多对多映射不好理解