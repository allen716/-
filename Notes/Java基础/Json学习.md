# 1.概念

**JSON：**

JavaScript Object Notation ( JS 对象简谱)， 是一种轻量级的数据交换格式。

采用完全独立于编程语言的文本格式来存储和表示数据。

**Json的本质：**

用JavaScript语法书写的一种具有特定格式的文本字符串。

**Json与JS对象的区别：**

JSON 是 JS 对象的字符串表示法，它使用文本表示一个 JS 对象的信息，本质是一个字符串

**优势：**

易于人阅读和编写，易于机器解析和生成，同时有效地提升网络传输效率



# 2.Json格式

## 2.1对象格式

```
{"name":"JSON","address":"深圳市宝安区","age":25} //JSON的对象格式的字符串
```

## 2.2数组对象

```
[{"name":"JSON","address":"深圳市宝安区","age":25}]//数据对象格式
```

对象格式和数组对象格式唯一的不同是：数组对象在对象格式的基础上加上了[]



# 3.Java对象和Json字符串之间的转换

## 3.1.Java对象转json字符串

```java
@Data
public class Cat {
    private String name;
    private Integer age;
    private String sound;
}
```

```java
public class Main {
    public static void main(String[] args) {
        Cat cat = new Cat();
        cat.setName("昭君");
        cat.setAge(1);
        cat.setSound("喵喵叫");

        //使用JSONObject
        JSONObject jsonObject = JSONObject.fromObject(cat);
        System.out.println("json对象：" + jsonObject.toString());

        //2、使用JSONArray
        JSONArray jsonArray = JSONArray.fromObject(cat);
        System.out.println("json数组对象：" + jsonArray.toString());
    }
}
```

结果如下:

```java
json对象：{"sound":"喵喵叫","name":"昭君","age":1}
json数组对象：[{"sound":"喵喵叫","name":"昭君","age":1}]
```

**两种方法都可以把java对象转化为JSON字符串，只是转化后的结构不同**

## 3.2.Json字符串转对象

（1）使用JSONObject

```java
	String objectStr = "{\"name\":\"昭君\", \"age\":1, \"sound\":\"喵喵叫\"}";
    //使用JSONObject
    JSONObject jsonObject = JSONObject.fromObject(objectStr);
    Cat cat = (Cat)JSONObject.toBean(jsonObject, Cat.class);
    System.out.println(cat);
```

输出：

```java
Cat(name=昭君, age=1, sound=喵喵叫)
```

（2）使用JSONArray


```java
	//使用JSONArray
	String arrayStr = "[{\"name\":\"昭君\", \"age\":1, \"sound\":\"喵喵叫\"}]";
	JSONArray jsonArray = JSONArray.fromObject(arrayStr);
	//获取jsonArray的第一个元素
	Object o = jsonArray.get(0);
    JSONObject jsonObject = JSONObject.fromObject(o);
    Object cat1 = JSONObject.toBean(jsonObject, Cat.class);
    System.out.println(cat1);
```
输出：

```java
Cat(name=昭君, age=1, sound=喵喵叫)
```



# 4.**list和json字符串的互转**

## 4.1list对象转换成json字符串

使用JSONArray做转化：

```java
Cat cat = new Cat();
cat.setName("昭君");
cat.setAge(1);
cat.setSound("喵喵叫");
List<Cat> cats = new ArrayList<>();
cats.add(cat);

//使用JSONArray
JSONArray jsonArray = JSONArray.fromObject(cats);
System.out.println(jsonArray.toString());
```

## 4.2json字符串转换成list对象

```java
String arrayStr = "[{\"name\":\"昭君\", \"age\":1, \"sound\":\"喵喵叫\"}," +
                "{\"name\":\"貂蝉\", \"age\":2, \"sound\":\"嘤嘤嘤\"}]";
JSONArray jsonArray = JSONArray.fromObject(arrayStr);
List<Cat> list = JSONArray.toList(jsonArray, Cat.class);
for (Cat cat: cats) {
    System.out.println(cat);
}
```

输出：

```java
Cat(name=昭君, age=1, sound=喵喵叫)
Cat(name=貂蝉, age=2, sound=嘤嘤嘤)
```

**注意：由于字符串的格式为带有“[]”的格式，所以这里选择JSONArray这个对象，它有toArray、toList方法可供使用，前者转化为java中的数组，或者转化为java中的list**



# 5.**map和json字符串的互转**

## 5.1map对象转成json字符串

```java
Cat cat = new Cat();
cat.setName("昭君");
cat.setAge(1);
cat.setSound("喵喵叫");
Map<String, Cat> map = new HashMap<>();
map.put("cat", cat);
//1.JSONObject
JSONObject jsonObject = JSONObject.fromObject(map);
System.out.println("jsonObject:" + jsonObject.toString());
//2.JSONArray
JSONArray jsonArray = JSONArray.fromObject(map);
System.out.println("jsonArray:" + jsonArray.toString());
```

输出：

```
jsonObject:{"cat":{"sound":"喵喵叫","name":"昭君","age":1}}
jsonArray:[{"cat":{"sound":"喵喵叫","name":"昭君","age":1}}]

```

## 5.2json字符串转成map对象

```java
//json字符串转成map对象
public static void main(String[] args) {
    String jsonStr = "{\"name\":\"昭君\", \"age\":1, \"sound\":\"喵喵叫\"}";
    JSONObject jsonObject = JSONObject.fromObject(jsonStr);
    Cat cat = (Cat) JSONObject.toBean(
        jsonObject, Cat.class, new HashMap<String, Object>());
    System.out.println(cat);
}
```

输出：

```java
Cat(name=昭君, age=1, sound=喵喵叫)
```

