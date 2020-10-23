# 					lombok学习笔记

# 1.概念

lombok是一个开源Java库，能在编译时，自动为我们生成get、set、构造方法等代码，并且能简化Java开发

注解简化开发

本文列下常用的几个注解：

@Data, @AllArgsConstructor, @NoArgsConstructor, @Builder, @Value, @NonNull, @Slf4j

# 2.常见注解

## 2.1 @Data

包含：@ToString, @EqualsAndHashCode, @Getter,@Setter,@RequiredArgsConstructor

（1）自动为所有字段添加@ToString, @EqualsAndHashCode, @Getter方法

（2）为非final字段添加@Setter,和@RequiredArgsConstructor

### 2.1.1@ToString

默认情况下，它会按顺序打印类名以及每个字段，并以逗号分隔。

生成toString()方法，默认情况下，它会按顺序（以逗号分隔）打印你的类名称以及每个字段。

使用小技巧：

**① 通过exclude设置不包含哪些字段:** 

```java
@ToString(exclude = {"company","salary"})
@AllArgsConstructor
@NoArgsConstructor
public class Person {
    String name;
    String company;
    Double salary;
}
```

测试：

```java
@Test
public void testToString() {
    Person person = new Person("allen", "cmb", 20000d);
    System.out.println(person.toString());
}
```

输出结果：

```java
Person(name=allen) //加exclude
Person(name=allen, company=cmb, salary=20000.0)  //不加exclude
```

**② 设置callSuper = true**

如果继承的有父类的话，可以设置callSuper 让其调用父类的toString()方法，打印父类中的属性

例如：@ToString(callSuper = true)

### 2.1.2@EqualsAndHashCode

从对象的字段生成`hashCode()`和`equals()`方法

同样，可以通过设置 callSuper=ture/false 来指定是否为父类中的属性生成equals和hashcode方法

首先看下：Player.java

```java
@Data
@ToString(callSuper = true)
//@SuperBuilder
@EqualsAndHashCode(callSuper = false)
public class Player extends Person {
    private String position;
    private int number;

    public Player() {
    }

    @Builder
    public Player(String name, String company, Double salary, String position, int number) {
        super(name, company, salary);
        this.position = position;
        this.number = number;
    }
}
```

测试通过，说明callsuper=false时，equals和hashcode方法中不包含父类中的属性。

反之，当callsuper=true，包含父类中的属性。

```java 
@Test
public void test3() {
    //Person person = new Person("allen", "cmb", 20000d);
    Player player1 = new Player("allen", "cmb", 20000d, "SF", 23);
    Player player2 = new Player("allen1", "cmb1", 20001d, "SF", 23);
    Assert.assertEquals(player1, player2);//true
}
```

### 2.1.3 @RequiredArgsConstructor

会生成一个包含：**常量（final）**，和 **标识了@NotNull**的变量 的构造方法。

```java
@ToString//(exclude = {"company","salary"})
@RequiredArgsConstructor
//@SuperBuilder
public class Person {
    protected String name;
    protected String company;
    protected Double salary;
    //测试@RequiredArgsConstructor，age
    protected final int age;
    //测试@RequiredArgsConstructor
    @NonNull
    protected int height;
}
```

测试:  该注解生成的构造方法为: **Peron(int age, int height);**

```java
@Test
public void testRequiredArgsConstructor() {
    //Person person = new Person("allen", "cmb", 20000d);
    Person person = new Person(25,178);
    System.out.println(person);
}
```

输出结果：

```java
Person(name=null, company=null, salary=null, age=25, height=178)
```

### 2.1.4 @Getter/ @Setter

使用@Getter和@Setter 后，可以利用lombok自动生成对应的get和set方法

(1)lombok生成的getter / setter方法默认作用域：public

```java
@ToString
@Getter//(AccessLevel.PRIVATE)
@Setter
@AllArgsConstructor
public class Person {
    protected String name;
    protected String company;
    protected Double salary;
}

@Test
    public void testGetterAndSetter() {
        Person person = new Person("allen","cmb", 15000.0);
        String company = person.getCompany();
    }
```

如果加上在@getter后面加上：AccessLevel.PRIVATE，那么在单元测试中，无法使用person.getCompany( )

（2）该注解中的onMethod_={} 属性

想试一下：把@notNull加到方法上进行限制，然而没有生效 --- 为什么？

```java
@ToString
@Getter(AccessLevel.PUBLIC)
@Setter
@AllArgsConstructor
public class Person {
    protected String name;
    protected String company;
    @Setter(value = AccessLevel.PUBLIC, onMethod_={@NotNull})
    protected Double salary = setSal();

    private Double setSal() {
        return null;
    }
```

## 2.2@AllArgsConstructor

为这个类生成：包含所有变量的构造方法

```java
    public Person(String name，String company，Double salary){
		this.name = name;
        this.company = company;
		this.salary = salary;
    }
```

## 2.3@NoArgsConstructor

为这个类生成：无参的构造方法

    public Person(){
    }
## 2.4@Builder

（1）作用：使用建造者模式创建一个对象

（2）使用场景：`@Builder`可以放在类，构造函数或方法上

```java
Person person1 = Person.builder()
        .name("allen")
        .company("cmb")
        .salary(15000d)
        .build();
```

（3）源码分析

在类上加@Builder注解：

```java
@Builder
public class Person {
    protected String name;
    protected String company;
    protected Double salary; //= setSal();
}
```

**编译后：**

技巧：① 在Person中建了一个PersonBuilder内部类，并具有和实体类形同的属性（完全copy一份）

​			② PersonBuilder类内部，xxx( )方法是给xxx属性设置值，并且返回的就是PersonBuilder，方便链式调用

​			② build( ) 方法返回Person对象：干活的是PersonBuilder，等值设置好之后，复制一份给Person

```java
public class Person {
    protected String name;
    protected String company;
    protected Double salary;

    Person(final String name, final String company, final Double salary) {
        this.name = name;
        this.company = company;
        this.salary = salary;
    }

    public static Person.PersonBuilder builder() {
        return new Person.PersonBuilder();
    }

    public static class PersonBuilder {
        private String name;
        private String company;
        private Double salary;

        PersonBuilder() {
        }

        public Person.PersonBuilder name(final String name) {
            this.name = name;
            return this;
        }

        public Person.PersonBuilder company(final String company) {
            this.company = company;
            return this;
        }

        public Person.PersonBuilder salary(final Double salary) {
            this.salary = salary;
            return this;
        }

        public Person build() {
            return new Person(this.name, this.company, this.salary);
        }

        public String toString() {
            return "Person.PersonBuilder(name=" + this.name + ", company=" + this.company + ", salary=" + this.salary + ")";
        }
    }
}
```

## 2.5@Value

（1）作用：只添加@Value注解，没有其他限制，那么类属性会被编译成final，因此只有get方法，而没有set方法

（2）实现分析

@Value = @AllArgsConstructor + @Getter + @EqualsAndHashCode + @ToString

反编译之后的源码：

```java
public final class Person {
    protected final String name;
    protected final String company;
    protected final Double salary;

    public Person(final String name, final String company, final Double salary) {
        this.name = name;
        this.company = company;
        this.salary = salary;
    }

    public String getName() {
        return this.name;
    }

    public String getCompany() {
        return this.company;
    }

    public Double getSalary() {
        return this.salary;
    }

    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Person)) {
            return false;
        } else {
            Person other;
            label44: {
                other = (Person)o;
                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name == null) {
                        break label44;
                    }
                } else if (this$name.equals(other$name)) {
                    break label44;
                }

                return false;
            }

            Object this$company = this.getCompany();
            Object other$company = other.getCompany();
            if (this$company == null) {
                if (other$company != null) {
                    return false;
                }
            } else if (!this$company.equals(other$company)) {
                return false;
            }

            Object this$salary = this.getSalary();
            Object other$salary = other.getSalary();
            if (this$salary == null) {
                if (other$salary != null) {
                    return false;
                }
            } else if (!this$salary.equals(other$salary)) {
                return false;
            }

            return true;
        }
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $name = this.getName();
        int result = result * 59 + ($name == null ? 43 : $name.hashCode());
        Object $company = this.getCompany();
        result = result * 59 + ($company == null ? 43 : $company.hashCode());
        Object $salary = this.getSalary();
        result = result * 59 + ($salary == null ? 43 : $salary.hashCode());
        return result;
    }

    public String toString() {
        return "Person(name=" + this.getName() + ", company=" + this.getCompany() + ", salary=" + this.getSalary() + ")";
    }
}
```

## 2.6@NonNull

（1）加上属性上

（2）实现分析

在使用到该属性的地方，会进行判空处理

编译后的代码：

① get方法判空

```java
@NonNull
public Double getSalary() {
    return this.salary;
}
```

② set方法为salary赋值时，会进行判空处理

```java
public void setSalary(@NonNull final Double salary) {
    if (salary == null) {
        throw new NullPointerException("salary is marked non-null but is null");
    } else {
        this.salary = salary;
    }
}
```

③ 构造方法中属性判空：

```java
public Person(final String name, final String company, @NonNull final Double salary) {
    if (salary == null) {
        throw new NullPointerException("salary is marked non-null but is null");
    } else {
        this.name = name;
        this.company = company;
        this.salary = salary;
    }
}
```

## 2.7@Slf4j

（1）@slf4j 相当于： private final Logger logger = LoggerFactory.getLogger(XXX.class);

（2）使用方法：

​        ① 在类上加上 注解

​	    ② 打印日志

```java
	logger.debug("debug message");
	logger.warn("warn message");
	logger.info("info message");
	logger.error("error message");
	logger.trace("trace message");
```
# 3.实现原理

参照这里实现原理部分：//TODO

https://blog.csdn.net/ThinkWon/article/details/101392808

# 4.引入lombok可能导致的问题

整理了下网上的说法：

（1）Lombok对于代码有很强的侵入性：在编译时修改代码

（2）如果只使用了@Data，而不使用@EqualsAndHashCode(callSuper=true)的话，会默认@EqualsAndHashCode(callSuper=false)，这时候生成的equals()方法只会比较子类的属性，不会考虑从父类继承的属性

（3）当需要升级到某个新版本的JDK的时候，若其中的特性在Lombok中不支持，我们的就会受到影响

原因：Lombok作为一个第三方工具，是由开源团队维护的，迭代速度无法保证