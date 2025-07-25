# 四、spring对IOC的实现
## 4.3 set注入专题
### 4.3.1 注入外部Bean
在之前4.2.1中使用的案例就是注入外部Bean的方式。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userDaoBean" class="com.powernode.spring6.dao.UserDao"/>

    <bean id="userServiceBean" class="com.powernode.spring6.service.UserService">
        <property name="userDao" ref="userDaoBean"/>
    </bean>

</beans>
```
外部Bean的特点：bean定义到外面，在property标签中使用ref属性进行注入。通常这种方式是常用。
### 4.3.2 注入内部Bean
内部Bean的方式：在bean标签中嵌套bean标签。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userServiceBean" class="com.powernode.spring6.service.UserService">
        <property name="userDao">
            <bean class="com.powernode.spring6.dao.UserDao"/>
        </property>
    </bean>

</beans>
```
```java
@Test
public void testInnerBean(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-inner-bean.xml");
    UserService userService = applicationContext.getBean("userServiceBean", UserService.class);
    userService.save();
}
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1664443574869-143a6d21-9b3f-4eaa-bd9c-6d9b0e9908b5.png#averageHue=%2383715a&clientId=u37420c9c-34da-4&from=paste&height=117&id=u0c96e384&originHeight=117&originWidth=480&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9774&status=done&style=shadow&taskId=uf7b96fb5-bb62-4e68-b696-58962292b60&title=&width=480)
这种方式作为了解。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=dbFbQ&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.3 注入简单类型
我们之前在进行注入的时候，对象的属性是另一个对象。
```java
public class UserService{
    
    private UserDao userDao;
    
    public void setUserDao(UserDao userDao){
        this.userDao = userDao;
    }
    
}
```
那如果对象的属性是int类型呢？
```java
public class User{
    
    private int age;
    
    public void setAge(int age){
        this.age = age;
    }
    
}
```
可以通过set注入的方式给该属性赋值吗？

- 当然可以。因为只要能够调用set方法就可以给属性赋值。

**编写程序给一个User对象的age属性赋值20：**
第一步：定义User类，提供age属性，提供age属性的setter方法。
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    private int age;

    public void setAge(int age) {
        this.age = age;
    }
    
    @Override
    public String toString() {
        return "User{" +
                "age=" + age +
                '}';
    }
}

```
第二步：编写spring配置文件：spring-simple-type.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userBean" class="com.powernode.spring6.beans.User">
        <!--如果像这种int类型的属性，我们称为简单类型，这种简单类型在注入的时候要使用value属性，不能使用ref-->
        <!--<property name="age" value="20"/>-->
        <property name="age">
            <value>20</value>
        </property>
    </bean>
</beans>
```
第三步：编写测试程序
```java
@Test
public void testSimpleType(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-simple-type.xml");
    User user = applicationContext.getBean("userBean", User.class);
    System.out.println(user);
}
```
第四步：运行测试程序
![1664444974497(1).png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1664444978134-0cdbca10-4322-4e31-a2f8-a177d3ae2e75.png#averageHue=%23857561&clientId=u37420c9c-34da-4&from=paste&height=117&id=u11b5fd73&originHeight=117&originWidth=481&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7836&status=done&style=shadow&taskId=uc3933259-1cdc-4781-9342-b54b31b24a2&title=&width=481)
**需要特别注意：如果给简单类型赋值，使用value属性或value标签。而不是ref。**
简单类型包括哪些呢？可以通过Spring的源码来分析一下：BeanUtils类
```java
public class BeanUtils{
    
    //.......
    
    /**
	 * Check if the given type represents a "simple" property: a simple value
	 * type or an array of simple value types.
	 * <p>See {@link #isSimpleValueType(Class)} for the definition of <em>simple
	 * value type</em>.
	 * <p>Used to determine properties to check for a "simple" dependency-check.
	 * @param type the type to check
	 * @return whether the given type represents a "simple" property
	 * @see org.springframework.beans.factory.support.RootBeanDefinition#DEPENDENCY_CHECK_SIMPLE
	 * @see org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#checkDependencies
	 * @see #isSimpleValueType(Class)
	 */
	public static boolean isSimpleProperty(Class<?> type) {
		Assert.notNull(type, "'type' must not be null");
		return isSimpleValueType(type) || (type.isArray() && isSimpleValueType(type.getComponentType()));
	}

	/**
	 * Check if the given type represents a "simple" value type: a primitive or
	 * primitive wrapper, an enum, a String or other CharSequence, a Number, a
	 * Date, a Temporal, a URI, a URL, a Locale, or a Class.
	 * <p>{@code Void} and {@code void} are not considered simple value types.
	 * @param type the type to check
	 * @return whether the given type represents a "simple" value type
	 * @see #isSimpleProperty(Class)
	 */
	public static boolean isSimpleValueType(Class<?> type) {
		return (Void.class != type && void.class != type &&
				(ClassUtils.isPrimitiveOrWrapper(type) ||
				Enum.class.isAssignableFrom(type) ||
				CharSequence.class.isAssignableFrom(type) ||
				Number.class.isAssignableFrom(type) ||
				Date.class.isAssignableFrom(type) ||
				Temporal.class.isAssignableFrom(type) ||
				URI.class == type ||
				URL.class == type ||
				Locale.class == type ||
				Class.class == type));
	}
    
    //........
}
```
**通过源码分析得知，简单类型包括：**

- **基本数据类型**
- **基本数据类型对应的包装类**
- **String或其他的CharSequence子类**
- **Number子类**
- **Date子类**
- **Enum子类**
- **URI**
- **URL**
- **Temporal子类**
- **Locale**
- **Class**
- **另外还包括以上简单值类型对应的数组类型。**

**经典案例：给数据源的属性注入值：**
假设我们现在要自己手写一个数据源，我们都知道所有的数据源都要实现javax.sql.DataSource接口，并且数据源中应该有连接数据库的信息，例如：driver、url、username、password等。
```java
package com.powernode.spring6.beans;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.logging.Logger;

/**
* 
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource
 * @since 1.0
 **/
public class MyDataSource implements DataSource {
    private String driver;
    private String url;
    private String username;
    private String password;

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }
    
    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    @Override
    public Connection getConnection() throws SQLException {
        return null;
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {

    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }
}

```
我们给driver、url、username、password四个属性分别提供了setter方法，我们可以使用spring的依赖注入完成数据源对象的创建和属性的赋值吗？看配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
    <bean id="dataSource" class="com.powernode.spring6.beans.MyDataSource">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
  
</beans>
```
测试程序：
```java
@Test
public void testDataSource(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-datasource.xml");
    MyDataSource dataSource = applicationContext.getBean("dataSource", MyDataSource.class);
    System.out.println(dataSource);
}
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1664445950974-57eb35af-c4ad-47c7-9a70-341d61596656.png#averageHue=%23947f67&clientId=u37420c9c-34da-4&from=paste&height=108&id=u30991232&originHeight=108&originWidth=1422&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18204&status=done&style=shadow&taskId=u56059cbc-8c37-4b68-b6ed-f70b9b65480&title=&width=1422)
你学会了吗？
**接下来，我们编写一个程序，把所有的简单类型全部测试一遍：**
编写一个类A：
```java
package com.powernode.spring6.beans;

import java.net.URI;
import java.net.URL;
import java.time.LocalDate;
import java.util.Date;
import java.util.Locale;

/**
 * @author 动力节点
 * @version 1.0
 * @className A
 * @since 1.0
 **/
public class A {
    private byte b;
    private short s;
    private int i;
    private long l;
    private float f;
    private double d;
    private boolean flag;
    private char c;

    private Byte b1;
    private Short s1;
    private Integer i1;
    private Long l1;
    private Float f1;
    private Double d1;
    private Boolean flag1;
    private Character c1;

    private String str;

    private Date date;

    private Season season;

    private URI uri;

    private URL url;

    private LocalDate localDate;

    private Locale locale;

    private Class clazz;
    
    // 生成setter方法
    // 生成toString方法
}

enum Season {
    SPRING, SUMMER, AUTUMN, WINTER
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="a" class="com.powernode.spring6.beans.A">
        <property name="b" value="1"/>
        <property name="s" value="1"/>
        <property name="i" value="1"/>
        <property name="l" value="1"/>
        <property name="f" value="1"/>
        <property name="d" value="1"/>
        <property name="flag" value="false"/>

        <property name="c" value="a"/>
        <property name="b1" value="2"/>
        <property name="s1" value="2"/>
        <property name="i1" value="2"/>
        <property name="l1" value="2"/>
        <property name="f1" value="2"/>
        <property name="d1" value="2"/>
        <property name="flag1" value="true"/>
        <property name="c1" value="a"/>

        <property name="str" value="zhangsan"/>
        <!--注意：value后面的日期字符串格式不能随便写，必须是Date对象toString()方法执行的结果。-->
        <!--如果想使用其他格式的日期字符串，就需要进行特殊处理了。具体怎么处理，可以看后面的课程！！！！-->
        <property name="date" value="Fri Sep 30 15:26:38 CST 2022"/>
        <property name="season" value="WINTER"/>
        <property name="uri" value="/save.do"/>
        <!--spring6之后，会自动检查url是否有效，如果无效会报错。-->
        <property name="url" value="http://www.baidu.com"/>
        <property name="localDate" value="EPOCH"/>
        <!--java.util.Locale 主要在软件的本地化时使用。它本身没有什么功能，更多的是作为一个参数辅助其他方法完成输出的本地化。-->
        <property name="locale" value="CHINESE"/>
        <property name="clazz" value="java.lang.String"/>
    </bean>
</beans>
```
编写测试程序：
```java
@Test
public void testAllSimpleType(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-all-simple-type.xml");
    A a = applicationContext.getBean("a", A.class);
    System.out.println(a);
}
```
执行结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1664524514681-c680e261-eeaf-4536-b5db-e4f28361a03d.png#averageHue=%2396846a&clientId=u42a1b96f-60c3-4&from=paste&height=120&id=uc207cea9&originHeight=120&originWidth=1561&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15664&status=done&style=shadow&taskId=ucde64917-cff9-4d4c-897a-a8925574083&title=&width=1561)
**需要注意的是：**

- **如果把Date当做简单类型的话，日期字符串格式不能随便写。格式必须符合Date的toString()方法格式。显然这就比较鸡肋了。如果我们提供一个这样的日期字符串：2010-10-11，在这里是无法赋值给Date类型的属性的。**
- **spring6之后，当注入的是URL，那么这个url字符串是会进行有效性检测的。如果是一个存在的url，那就没问题。如果不存在则报错。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=qtWOr&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.4 级联属性赋值（了解）
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Clazz
 * @since 1.0
 **/
public class Clazz {
    private String name;

    public Clazz() {
    }

    public Clazz(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Clazz{" +
                "name='" + name + '\'' +
                '}';
    }
}

```
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Student
 * @since 1.0
 **/
public class Student {
    private String name;
    private Clazz clazz;

    public Student() {
    }

    public Student(String name, Clazz clazz) {
        this.name = name;
        this.clazz = clazz;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setClazz(Clazz clazz) {
        this.clazz = clazz;
    }

    public Clazz getClazz() {
        return clazz;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", clazz=" + clazz +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="clazzBean" class="com.powernode.spring6.beans.Clazz"/>

    <bean id="student" class="com.powernode.spring6.beans.Student">
        <property name="name" value="张三"/>

        <!--要点1：以下两行配置的顺序不能颠倒-->
        <property name="clazz" ref="clazzBean"/>
        <!--要点2：clazz属性必须有getter方法-->
        <property name="clazz.name" value="高三一班"/>
    </bean>
</beans>
```
```java
@Test
public void testCascade(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-cascade.xml");
    Student student = applicationContext.getBean("student", Student.class);
    System.out.println(student);
}
```
运行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665196735272-17e84eb6-8dcd-4c69-9ed7-e35c7c8ad36d.png#averageHue=%238f7e66&clientId=ufc7e21e2-2cbb-4&from=paste&height=124&id=PZQ9M&originHeight=124&originWidth=572&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12335&status=done&style=shadow&taskId=ub86e8af3-b20e-4505-a208-3614f1fe2dc&title=&width=572)
**要点：**

- **在spring配置文件中，如上，注意顺序。**
- **在spring配置文件中，clazz属性必须提供getter方法。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=KUphh&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.5 注入数组
**当数组中的元素是简单类型**：
```java
package com.powernode.spring6.beans;

import java.util.Arrays;

public class Person {
    private String[] favariteFoods;

    public void setFavariteFoods(String[] favariteFoods) {
        this.favariteFoods = favariteFoods;
    }

    @Override
    public String toString() {
        return "Person{" +
                "favariteFoods=" + Arrays.toString(favariteFoods) +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.powernode.spring6.beans.Person">
        <property name="favariteFoods">
            <array>
                <value>鸡排</value>
                <value>汉堡</value>
                <value>鹅肝</value>
            </array>
        </property>
    </bean>
</beans>
```
```java
@Test
public void testArraySimple(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-array-simple.xml");
    Person person = applicationContext.getBean("person", Person.class);
    System.out.println(person);
}
```
**当数组中的元素是非简单类型：一个订单中包含多个商品。**
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Goods
 * @since 1.0
 **/
public class Goods {
    private String name;

    public Goods() {
    }

    public Goods(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Goods{" +
                "name='" + name + '\'' +
                '}';
    }
}

```
```java
package com.powernode.spring6.beans;

import java.util.Arrays;

/**
 * @author 动力节点
 * @version 1.0
 * @className Order
 * @since 1.0
 **/
public class Order {
    // 一个订单中有多个商品
    private Goods[] goods;

    public Order() {
    }

    public Order(Goods[] goods) {
        this.goods = goods;
    }

    public void setGoods(Goods[] goods) {
        this.goods = goods;
    }

    @Override
    public String toString() {
        return "Order{" +
                "goods=" + Arrays.toString(goods) +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="goods1" class="com.powernode.spring6.beans.Goods">
        <property name="name" value="西瓜"/>
    </bean>

    <bean id="goods2" class="com.powernode.spring6.beans.Goods">
        <property name="name" value="苹果"/>
    </bean>

    <bean id="order" class="com.powernode.spring6.beans.Order">
        <property name="goods">
            <array>
                <!--这里使用ref标签即可-->
                <ref bean="goods1"/>
                <ref bean="goods2"/>
            </array>
        </property>
    </bean>

</beans>
```
测试程序：
```java
@Test
public void testArray(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-array.xml");
    Order order = applicationContext.getBean("order", Order.class);
    System.out.println(order);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665193773078-0a30da90-244d-460e-bc2c-901d99873668.png#averageHue=%238d7a63&clientId=ufc7e21e2-2cbb-4&from=paste&height=112&id=u9f40aef5&originHeight=112&originWidth=620&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12496&status=done&style=shadow&taskId=u40fc5b1e-c4d1-44d5-8964-becbb878422&title=&width=620)
**要点：**

- **如果数组中是简单类型，使用value标签。**
- **如果数组中是非简单类型，使用ref标签。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=h8Mdj&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.6 注入List集合
List集合：有序可重复
```java
package com.powernode.spring6.beans;

import java.util.List;

/**
 * @author 动力节点
 * @version 1.0
 * @className People
 * @since 1.0
 **/
public class People {
    // 一个人有多个名字
    private List<String> names;

    public void setNames(List<String> names) {
        this.names = names;
    }

    @Override
    public String toString() {
        return "People{" +
                "names=" + names +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="peopleBean" class="com.powernode.spring6.beans.People">
        <property name="names">
            <list>
                <value>铁锤</value>
                <value>张三</value>
                <value>张三</value>
                <value>张三</value>
                <value>狼</value>
            </list>
        </property>
    </bean>
</beans>
```
```java
@Test
public void testCollection(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-collection.xml");
    People peopleBean = applicationContext.getBean("peopleBean", People.class);
    System.out.println(peopleBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665199057422-a5f94a7c-711e-4846-9d69-2caf05d11e88.png#averageHue=%238c7b63&clientId=ufc7e21e2-2cbb-4&from=paste&height=116&id=u7bb357d8&originHeight=116&originWidth=499&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12244&status=done&style=shadow&taskId=ue9649c7c-07e0-4b72-8d22-38a524b1d51&title=&width=499)
**注意：注入List集合的时候使用list标签，如果List集合中是简单类型使用value标签，反之使用ref标签。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=oagnt&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.7 注入Set集合
Set集合：无序不可重复
```java
package com.powernode.spring6.beans;

import java.util.List;
import java.util.Set;

/**
 * @author 动力节点
 * @version 1.0
 * @className People
 * @since 1.0
 **/
public class People {
    // 一个人有多个电话
    private Set<String> phones;

    public void setPhones(Set<String> phones) {
        this.phones = phones;
    }
    
    //......
    
    @Override
    public String toString() {
        return "People{" +
                "phones=" + phones +
                ", names=" + names +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="peopleBean" class="com.powernode.spring6.beans.People">
        <property name="phones">
            <set>
                <!--尽管这里注入了多个，但是set会自动去重，只会显示3个-->
                <value>110</value>
                <value>110</value>
                <value>120</value>
                <value>120</value>
                <value>119</value>
                <value>119</value>
            </set>
        </property>
    </bean>
</beans>
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665199868843-bc721edd-e89a-4298-b41a-7c5ac8c93530.png#averageHue=%23917f66&clientId=ufc7e21e2-2cbb-4&from=paste&height=119&id=uab47bd18&originHeight=119&originWidth=737&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14030&status=done&style=shadow&taskId=ua0f55010-1dde-4237-a389-bdb339091f0&title=&width=737)
**要点：**

- **使用\<set>标签**
*  **set集合中元素是简单类型的使用value标签，反之使用ref标签。**


![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=woKpa&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.8 注入Map集合
```java
package com.powernode.spring6.beans;

import java.util.List;
import java.util.Map;
import java.util.Set;

/**
 * @author 动力节点
 * @version 1.0
 * @className People
 * @since 1.0
 **/
public class People {
    // 一个人有多个住址
    private Map<Integer, String> addrs;

    public void setAddrs(Map<Integer, String> addrs) {
        this.addrs = addrs;
    }
    
    //......
    
    @Override
    public String toString() {
        return "People{" +
                "addrs=" + addrs +
                ", phones=" + phones +
                ", names=" + names +
                '}';
    }

}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="peopleBean" class="com.powernode.spring6.beans.People">
        <property name="addrs">
            <map>
                <!--如果key不是简单类型，使用 key-ref 属性-->
                <!--如果value不是简单类型，使用 value-ref 属性-->
                <entry key="1" value="北京大兴区"/>
                <entry key="2" value="上海浦东区"/>
                <entry key="3" value="深圳宝安区"/>
            </map>
        </property>
    </bean>
</beans>
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665200352800-0b980533-2d1f-4222-8aaf-1b253cb19a39.png#averageHue=%23917c61&clientId=ufc7e21e2-2cbb-4&from=paste&height=121&id=u49b67347&originHeight=121&originWidth=1231&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20036&status=done&style=shadow&taskId=u17844f89-fdf0-45fc-b532-425734c5d39&title=&width=1231)
**要点：**

- **使用\<map>标签**
- **如果key是简单类型，使用 key 属性，反之使用 key-ref 属性。**
- **如果value是简单类型，使用 value 属性，反之使用 value-ref 属性。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=tfNae&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.9 注入Properties
java.util.Properties继承java.util.Hashtable，所以Properties也是一个Map集合。
```java
package com.powernode.spring6.beans;

import java.util.List;
import java.util.Map;
import java.util.Properties;
import java.util.Set;

/**
 * @author 动力节点
 * @version 1.0
 * @className People
 * @since 1.0
 **/
public class People {

    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }
    
    //......

    @Override
    public String toString() {
        return "People{" +
                "properties=" + properties +
                ", addrs=" + addrs +
                ", phones=" + phones +
                ", names=" + names +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="peopleBean" class="com.powernode.spring6.beans.People">
        <property name="properties">
            <props>
                <prop key="driver">com.mysql.cj.jdbc.Driver</prop>
                <prop key="url">jdbc:mysql://localhost:3306/spring</prop>
                <prop key="username">root</prop>
                <prop key="password">123456</prop>
            </props>
        </property>
    </bean>
</beans>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665201002733-ae9273da-1fb9-495c-907e-e6c8489a7ec5.png#averageHue=%23968269&clientId=ufc7e21e2-2cbb-4&from=paste&height=121&id=u0920d032&originHeight=121&originWidth=1390&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17881&status=done&style=shadow&taskId=u88c0bcbc-5a5a-46c3-ba4c-0eb0c365cbb&title=&width=1390)
**要点：**

- **使用\<props>标签嵌套\<prop>标签完成。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=HHmRb&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.10 注入null和空字符串
注入空字符串使用：<value/> 或者 value=""
注入null使用：<null/> 或者 不为该属性赋值

- 我们先来看一下，怎么注入空字符串。
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Vip
 * @since 1.0
 **/
public class Vip {
    private String email;

    public void setEmail(String email) {
        this.email = email;
    }

    @Override
    public String toString() {
        return "Vip{" +
                "email='" + email + '\'' +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="vipBean" class="com.powernode.spring6.beans.Vip">
        <!--空串的第一种方式-->
        <!--<property name="email" value=""/>-->
        <!--空串的第二种方式-->
        <property name="email">
            <value/>
        </property>
    </bean>

</beans>
```
```java
@Test
public void testNull(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-null.xml");
    Vip vipBean = applicationContext.getBean("vipBean", Vip.class);
    System.out.println(vipBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665202057919-ba40e1bb-405b-455e-9719-0c56f1f9144b.png#averageHue=%2384745f&clientId=ufc7e21e2-2cbb-4&from=paste&height=100&id=u9f7695ac&originHeight=100&originWidth=456&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9234&status=done&style=shadow&taskId=uc32bf6ca-c7f5-4213-981e-ca7d0fb83c0&title=&width=456)

- 怎么注入null呢？

第一种方式：不给属性赋值
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="vipBean" class="com.powernode.spring6.beans.Vip" />

</beans>
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665202218759-f35ed2da-e136-4ef0-92e5-8ae4ea029620.png#averageHue=%23887862&clientId=ufc7e21e2-2cbb-4&from=paste&height=116&id=ufa4ea0b9&originHeight=116&originWidth=454&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9606&status=done&style=shadow&taskId=u9ad79395-c316-45f2-86cc-f03852ce598&title=&width=454)
第二种方式：使用<null/>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="vipBean" class="com.powernode.spring6.beans.Vip">
        <property name="email">
            <null/>
        </property>
    </bean>

</beans>
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665202218759-f35ed2da-e136-4ef0-92e5-8ae4ea029620.png#averageHue=%23887862&clientId=ufc7e21e2-2cbb-4&from=paste&height=116&id=n18Ze&originHeight=116&originWidth=454&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9606&status=done&style=shadow&taskId=u9ad79395-c316-45f2-86cc-f03852ce598&title=&width=454)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=NS9TN&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.3.11 注入的值中含有特殊符号
XML中有5个特殊字符，分别是：<、>、'、"、&
以上5个特殊符号在XML中会被特殊对待，会被当做XML语法的一部分进行解析，如果这些特殊符号直接出现在注入的字符串当中，会报错。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665209144602-479d8a8d-d79d-4da8-bb13-6a9c7e76a949.png#averageHue=%23807d54&clientId=ufc7e21e2-2cbb-4&from=paste&height=383&id=ua27034c5&originHeight=383&originWidth=904&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64612&status=done&style=shadow&taskId=u25e3c1ae-9d23-4055-ae86-1eee56cf26e&title=&width=904)
解决方案包括两种：

- 第一种：特殊符号使用转义字符代替。
- 第二种：将含有特殊符号的字符串放到：<![CDATA[]]> 当中。因为放在CDATA区中的数据不会被XML文件解析器解析。

5个特殊字符对应的转义字符分别是：

| **特殊字符** | **转义字符** |
| --- | --- |
| > | &gt; |
| < | &lt; |
| ' | &apos; |
| " | &quot; |
| & | &amp; |

先使用转义字符来代替：
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Math
 * @since 1.0
 **/
public class Math {
    private String result;

    public void setResult(String result) {
        this.result = result;
    }

    @Override
    public String toString() {
        return "Math{" +
                "result='" + result + '\'' +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="mathBean" class="com.powernode.spring6.beans.Math">
        <property name="result" value="2 &lt; 3"/>
    </bean>
</beans>
```
```java
@Test
public void testSpecial(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-special.xml");
    Math mathBean = applicationContext.getBean("mathBean", Math.class);
    System.out.println(mathBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665210216232-b41609a9-2ead-4179-9f0c-dd9a9a4e033e.png#averageHue=%23877761&clientId=ufc7e21e2-2cbb-4&from=paste&height=113&id=ueeed1dd1&originHeight=113&originWidth=486&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10497&status=done&style=shadow&taskId=uf47e1e75-4185-454f-8c84-9be54adcdf8&title=&width=486)
我们再来使用CDATA方式：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="mathBean" class="com.powernode.spring6.beans.Math">
        <property name="result">
            <!--只能使用value标签-->
            <value><![CDATA[2 < 3]]></value>
        </property>
    </bean>

</beans>
```
**注意：使用CDATA时，不能使用value属性，只能使用value标签。**
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665210754241-75f884a4-a77b-4918-b398-9f7be2a58873.png#averageHue=%23867560&clientId=ufc7e21e2-2cbb-4&from=paste&height=112&id=u015b970a&originHeight=112&originWidth=492&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10483&status=done&style=shadow&taskId=u9d6ba99c-0427-4885-ad07-0bf77106322&title=&width=492)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=rz77u&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 4.4 p命名空间注入
目的：简化配置。
使用p命名空间注入的前提条件包括两个：

- 第一：在XML头部信息中添加p命名空间的配置信息：xmlns:p="[http://www.springframework.org/schema/p"](http://www.springframework.org/schema/p")
- 第二：p命名空间注入是基于setter方法的，所以需要对应的属性提供setter方法。
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className Customer
 * @since 1.0
 **/
public class Customer {
    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Customer{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="customerBean" class="com.powernode.spring6.beans.Customer" p:name="zhangsan" p:age="20"/>

</beans>
```
```java
@Test
public void testP(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-p.xml");
    Customer customerBean = applicationContext.getBean("customerBean", Customer.class);
    System.out.println(customerBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665215638858-c5ae8aef-43ec-455d-90a3-ac3f97c92746.png#averageHue=%238d7c66&clientId=ufc7e21e2-2cbb-4&from=paste&height=118&id=u4aeacd2a&originHeight=118&originWidth=473&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11448&status=done&style=shadow&taskId=u08f3e033-d49d-44e1-b717-e751097bdec&title=&width=473)
把setter方法去掉：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665215713205-fcebda06-c4bb-486b-a2d7-6a238088625b.png#averageHue=%23352c2b&clientId=ufc7e21e2-2cbb-4&from=paste&height=220&id=uf42f4afe&originHeight=220&originWidth=1058&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19291&status=done&style=shadow&taskId=u9c7f0649-555f-48d3-816e-a105727b293&title=&width=1058)
所以p命名空间实际上是对set注入的简化。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=QEHc4&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 4.5 c命名空间注入
c命名空间是简化构造方法注入的。
使用c命名空间的两个前提条件：
第一：需要在xml配置文件头部添加信息：xmlns:c="http://www.springframework.org/schema/c"
第二：需要提供构造方法。
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyTime
 * @since 1.0
 **/
public class MyTime {
    private int year;
    private int month;
    private int day;

    public MyTime(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    @Override
    public String toString() {
        return "MyTime{" +
                "year=" + year +
                ", month=" + month +
                ", day=" + day +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--<bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:year="1970" c:month="1" c:day="1"/>-->

    <bean id="myTimeBean" class="com.powernode.spring6.beans.MyTime" c:_0="2008" c:_1="8" c:_2="8"/>

</beans>
```
```java
@Test
public void testC(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-c.xml");
    MyTime myTimeBean = applicationContext.getBean("myTimeBean", MyTime.class);
    System.out.println(myTimeBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665216171317-2dc02c42-3f3e-42f5-80e7-c578888e2fbb.png#averageHue=%2394856d&clientId=ufc7e21e2-2cbb-4&from=paste&height=118&id=ue696a583&originHeight=118&originWidth=487&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11561&status=done&style=shadow&taskId=ua699a0a9-fe16-4e30-9d2b-71d4a4b99ee&title=&width=487)
把构造方法注释掉：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665216339051-c5eecc20-6801-46dd-a331-e5b33c66c7ed.png#averageHue=%23352b2b&clientId=ufc7e21e2-2cbb-4&from=paste&height=216&id=u54a2f985&originHeight=216&originWidth=1176&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19996&status=done&style=shadow&taskId=u2436bd3d-d8e6-4a92-9b23-68bb0fcc84c&title=&width=1176)
所以，c命名空间是依靠构造方法的。
**注意：不管是p命名空间还是c命名空间，注入的时候都可以注入简单类型以及非简单类型。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=nXLbi&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 4.6 util命名空间
使用util命名空间可以让**配置复用**。
使用util命名空间的前提是：在spring配置文件头部添加配置信息。如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665218059794-30411b76-a22c-4339-ab60-acad8f02ab28.png#averageHue=%23312a2a&clientId=ufc7e21e2-2cbb-4&from=paste&height=212&id=udeece73c&originHeight=212&originWidth=1494&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44224&status=done&style=shadow&taskId=u39d74a7a-b50e-4d8e-b74b-c63406de34f&title=&width=1494)
```java
package com.powernode.spring6.beans;

import java.util.Properties;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource1
 * @since 1.0
 **/
public class MyDataSource1 {
    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    @Override
    public String toString() {
        return "MyDataSource1{" +
                "properties=" + properties +
                '}';
    }
}

```
```java
package com.powernode.spring6.beans;

import java.util.Properties;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource2
 * @since 1.0
 **/
public class MyDataSource2 {
    private Properties properties;

    public void setProperties(Properties properties) {
        this.properties = properties;
    }

    @Override
    public String toString() {
        return "MyDataSource2{" +
                "properties=" + properties +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">

    <util:properties id="prop">
        <prop key="driver">com.mysql.cj.jdbc.Driver</prop>
        <prop key="url">jdbc:mysql://localhost:3306/spring</prop>
        <prop key="username">root</prop>
        <prop key="password">123456</prop>
    </util:properties>

    <bean id="dataSource1" class="com.powernode.spring6.beans.MyDataSource1">
        <property name="properties" ref="prop"/>
    </bean>

    <bean id="dataSource2" class="com.powernode.spring6.beans.MyDataSource2">
        <property name="properties" ref="prop"/>
    </bean>
</beans>
```
```java
@Test
public void testUtil(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-util.xml");

    MyDataSource1 dataSource1 = applicationContext.getBean("dataSource1", MyDataSource1.class);
    System.out.println(dataSource1);

    MyDataSource2 dataSource2 = applicationContext.getBean("dataSource2", MyDataSource2.class);
    System.out.println(dataSource2);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665218430727-c81e399e-294e-4bb5-b98b-2c8875b0884f.png#averageHue=%23968168&clientId=ufc7e21e2-2cbb-4&from=paste&height=140&id=ud2836a05&originHeight=140&originWidth=1518&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29754&status=done&style=shadow&taskId=ubb9d8e9e-3a21-4a14-8fe5-ff81df06522&title=&width=1518)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=truN5&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 4.7 基于XML的自动装配
Spring还可以完成自动化的注入，自动化注入又被称为自动装配。它可以根据**名字**进行自动装配，也可以根据**类型**进行自动装配。
### 4.7.1 根据名称自动装配
```java
package com.powernode.spring6.dao;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserDao
 * @since 1.0
 **/
public class UserDao {

    public void insert(){
        System.out.println("正在保存用户数据。");
    }
}

```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserService
 * @since 1.0
 **/
public class UserService {

    private UserDao aaa;

    // 这个set方法非常关键
    public void setAaa(UserDao aaa) {
        this.aaa = aaa;
    }

    public void save(){
        aaa.insert();
    }
}

```
Spring的配置文件这样配置：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.powernode.spring6.service.UserService" autowire="byName"/>
    
    <bean id="aaa" class="com.powernode.spring6.dao.UserDao"/>

</beans>
```
这个配置起到关键作用：

- UserService Bean中需要添加autowire="byName"，表示通过名称进行装配。
- UserService类中有一个UserDao属性，而UserDao属性的名字是aaa，**对应的set方法是setAaa()**，正好和UserDao Bean的id是一样的。这就是根据名称自动装配。
```java
@Test
public void testAutowireByName(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-autowire.xml");
    UserService userService = applicationContext.getBean("userService", UserService.class);
    userService.save();
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665535913374-7031648f-fad4-4fa1-a3f1-68dcf2318bef.png#averageHue=%23f5f4f3&clientId=ubfe41891-11ea-4&from=paste&height=116&id=u09b0d555&originHeight=116&originWidth=471&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9778&status=done&style=shadow&taskId=u99de35e6-3c78-4628-b282-8fe94b88194&title=&width=471)
我们来测试一下，byName装配是和属性名有关还是和set方法名有关系：
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserService
 * @since 1.0
 **/
public class UserService {
    // 这里没修改
    private UserDao aaa;

    /*public void setAaa(UserDao aaa) {
        this.aaa = aaa;
    }*/

    // set方法名变化了
    public void setDao(UserDao aaa){
        this.aaa = aaa;
    }

    public void save(){
        aaa.insert();
    }
}

```
在执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665536092171-afa2acd5-68c8-4289-95bd-ab8c0f88a66d.png#averageHue=%23faf7f6&clientId=ubfe41891-11ea-4&from=paste&height=246&id=u31a28635&originHeight=246&originWidth=1329&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37747&status=done&style=shadow&taskId=u205c3850-03b9-4bc5-96ea-2dd028afe91&title=&width=1329)
通过测试得知，aaa属性并没有赋值成功。也就是并没有装配成功。
我们将spring配置文件修改以下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <bean id="userService" class="com.powernode.spring6.service.UserService" autowire="byName"/>
  <!--这个id修改了-->
  <bean id="dao" class="com.powernode.spring6.dao.UserDao"/>
  
</beans>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665536194436-6efd0c08-72da-437e-b3ad-143cdb00834d.png#averageHue=%23f4f3f1&clientId=ubfe41891-11ea-4&from=paste&height=112&id=u30228306&originHeight=112&originWidth=462&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9786&status=done&style=shadow&taskId=uda675890-f3bf-4882-807f-6e06230e554&title=&width=462)
这说明，如果根据名称装配(byName)，底层会调用set方法进行注入。
例如：setAge() 对应的名字是age，setPassword()对应的名字是password，setEmail()对应的名字是email。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=o5EOt&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 4.7.2 根据类型自动装配
```java
package com.powernode.spring6.dao;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountDao
 * @since 1.0
 **/
public class AccountDao {
    public void insert(){
        System.out.println("正在保存账户信息");
    }
}

```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.AccountDao;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountService
 * @since 1.0
 **/
public class AccountService {
    private AccountDao accountDao;

    public void setAccountDao(AccountDao accountDao) {
        this.accountDao = accountDao;
    }

    public void save(){
        accountDao.insert();
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--byType表示根据类型自动装配-->
    <bean id="accountService" class="com.powernode.spring6.service.AccountService" autowire="byType"/>

    <bean class="com.powernode.spring6.dao.AccountDao"/>

</beans>
```
```java
@Test
public void testAutowireByType(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-autowire.xml");
    AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
    accountService.save();
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665537096983-d3c25b4c-21e1-499f-b348-6f829bc84a48.png#averageHue=%23f4f3f2&clientId=ubfe41891-11ea-4&from=paste&height=109&id=ucf231dcd&originHeight=109&originWidth=514&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9362&status=done&style=shadow&taskId=u73dc5c4e-c505-4247-8652-02ac58e7020&title=&width=514)
我们把UserService中的set方法注释掉，再执行：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665537145356-cf979b68-e11b-4b4f-b1b4-7c20649aa199.png#averageHue=%23faf8f7&clientId=ubfe41891-11ea-4&from=paste&height=235&id=uea831f0c&originHeight=235&originWidth=1444&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38307&status=done&style=shadow&taskId=u74719bf8-872a-4eb2-a90a-10c3f6943b1&title=&width=1444)
可以看到无论是byName还是byType，在装配的时候都是基于set方法的。所以set方法是必须要提供的。提供构造方法是不行的，大家可以测试一下。这里就不再赘述。
如果byType，根据类型装配时，如果配置文件中有两个类型一样的bean会出现什么问题呢？
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountService" class="com.powernode.spring6.service.AccountService" autowire="byType"/>

    <bean id="x" class="com.powernode.spring6.dao.AccountDao"/>
    <bean id="y" class="com.powernode.spring6.dao.AccountDao"/>

</beans>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665537341888-57af14a1-eeb4-4070-8713-b4368003251d.png#averageHue=%23faf7f6&clientId=ubfe41891-11ea-4&from=paste&height=254&id=uee149cb5&originHeight=254&originWidth=1583&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57785&status=done&style=shadow&taskId=ud9ec74f0-3975-42b6-9535-c4c92b16535&title=&width=1583)
测试结果说明了，当byType进行自动装配的时候，配置文件中某种类型的Bean必须是唯一的，不能出现多个。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=I89vS&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 4.8 Spring引入外部属性配置文件
我们都知道编写数据源的时候是需要连接数据库的信息的，例如：driver url username password等信息。这些信息可以单独写到一个属性配置文件中吗，这样用户修改起来会更加的方便。当然可以。
第一步：写一个数据源类，提供相关属性。
```java
package com.powernode.spring6.beans;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.logging.Logger;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource
 * @since 1.0
 **/
public class MyDataSource implements DataSource {
    @Override
    public String toString() {
        return "MyDataSource{" +
                "driver='" + driver + '\'' +
                ", url='" + url + '\'' +
                ", username='" + username + '\'' +
                ", password='" + password + '\'' +
                '}';
    }

    private String driver;
    private String url;
    private String username;
    private String password;

    public void setDriver(String driver) {
        this.driver = driver;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    //......
}

```
第二步：在类路径下新建jdbc.properties文件，并配置信息。
```properties
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/spring
username=root
password=root123
```
第三步：在spring配置文件中引入context命名空间。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```
第四步：在spring中配置使用jdbc.properties文件。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="jdbc.properties"/>
    
    <bean id="dataSource" class="com.powernode.spring6.beans.MyDataSource">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
    </bean>
</beans>
```
测试程序：
```java
@Test
public void testProperties(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-properties.xml");
    MyDataSource dataSource = applicationContext.getBean("dataSource", MyDataSource.class);
    System.out.println(dataSource);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665540422630-223562fd-e97b-40fe-96e7-df2c8744e2c2.png#averageHue=%23f7f6f4&clientId=ubfe41891-11ea-4&from=paste&height=117&id=udb7c7212&originHeight=117&originWidth=1527&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18541&status=done&style=shadow&taskId=u910a7d81-51e6-40d2-ab18-67b9262e8e5&title=&width=1527)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=tfkd9&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 五、Bean的作用域
## 5.1 singleton
默认情况下，Spring的IoC容器创建的Bean对象是单例的。来测试一下：
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className SpringBean
 * @since 1.0
 **/
public class SpringBean {
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="sb" class="com.powernode.spring6.beans.SpringBean" />
    
</beans>
```
```java
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");

    SpringBean sb1 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb1);

    SpringBean sb2 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb2);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665220014331-a1e4cac5-c749-4b67-bab8-43d6957c35e4.png#averageHue=%23927d67&clientId=ufc7e21e2-2cbb-4&from=paste&height=142&id=u054fef63&originHeight=142&originWidth=599&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20728&status=done&style=shadow&taskId=ufb3a6c56-51b9-43f5-803d-c4ca68308aa&title=&width=599)
通过测试得知：Spring的IoC容器中，默认情况下，Bean对象是单例的。
这个对象在什么时候创建的呢？可以为SpringBean提供一个无参数构造方法，测试一下，如下：
```java
package com.powernode.spring6.beans;

/**
 * @author 动力节点
 * @version 1.0
 * @className SpringBean
 * @since 1.0
 **/
public class SpringBean {
    public SpringBean() {
        System.out.println("SpringBean的无参数构造方法执行。");
    }
}

```
将测试程序中getBean()所在行代码注释掉：
```java
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
}
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665220250081-036fe814-8328-4b58-adf4-b3a37eb0cb4d.png#averageHue=%238e7f66&clientId=ufc7e21e2-2cbb-4&from=paste&height=114&id=ube1053bf&originHeight=114&originWidth=482&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12907&status=done&style=shadow&taskId=u22871dcd-91c0-46b1-b30e-3d03327e8c6&title=&width=482)
通过测试得知，默认情况下，Bean对象的创建是在初始化Spring上下文的时候就完成的。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=fob5g&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 5.2 prototype
如果想让Spring的Bean对象以多例的形式存在，可以在bean标签中指定scope属性的值为：**prototype**，这样Spring会在每一次执行getBean()方法的时候创建Bean对象，调用几次则创建几次。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="sb" class="com.powernode.spring6.beans.SpringBean" scope="prototype" />

</beans>
```
```java
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");

    SpringBean sb1 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb1);

    SpringBean sb2 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb2);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665220593171-19b1a750-551c-441d-8f8f-9c7aa7601e77.png#averageHue=%23927c63&clientId=ufc7e21e2-2cbb-4&from=paste&height=198&id=udb383895&originHeight=198&originWidth=616&originalType=binary&ratio=1&rotation=0&showTitle=false&size=32800&status=done&style=shadow&taskId=u3dea7c4e-0cc4-4a5e-ad06-76207e442c6&title=&width=616)
我们可以把测试代码中的getBean()方法所在行代码注释掉：
```java
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665220698959-ff4ad909-670e-4745-960d-5c215e2af71e.png#averageHue=%238d7b65&clientId=ufc7e21e2-2cbb-4&from=paste&height=153&id=u098e4067&originHeight=153&originWidth=541&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10979&status=done&style=shadow&taskId=u2c63f077-00be-49c9-a310-71c3fc2aa7b&title=&width=541)
可以看到这一次在初始化Spring上下文的时候，并没有创建Bean对象。
那你可能会问：scope如果没有配置，它的默认值是什么呢？默认值是singleton，单例的。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="sb" class="com.powernode.spring6.beans.SpringBean" scope="singleton" />

</beans>
```
```java
@Test
public void testScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");

    SpringBean sb1 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb1);

    SpringBean sb2 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb2);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665221074412-9b48c6e3-44f0-492c-a712-37b4baa24341.png#averageHue=%23937e66&clientId=ufc7e21e2-2cbb-4&from=paste&height=173&id=u956152e2&originHeight=173&originWidth=573&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26951&status=done&style=shadow&taskId=uad922a99-49bc-4c58-a11e-9d2b89f6856&title=&width=573)
通过测试得知，没有指定scope属性时，默认是singleton单例的。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=q2lBK&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 5.3 其它scope
**scope属性的值不止两个，它一共包括8个选项：**

- singleton：默认的，单例。
- prototype：原型。每调用一次getBean()方法则获取一个新的Bean对象。或每次注入的时候都是新对象。
- request：一个请求对应一个Bean。**仅限于在WEB应用中使用**。
- session：一个会话对应一个Bean。**仅限于在WEB应用中使用**。
- global session：**portlet应用中专用的**。如果在Servlet的WEB应用中使用global session的话，和session一个效果。（portlet和servlet都是规范。servlet运行在servlet容器中，例如Tomcat。portlet运行在portlet容器中。）
- application：一个应用对应一个Bean。**仅限于在WEB应用中使用。**
- websocket：一个websocket生命周期对应一个Bean。**仅限于在WEB应用中使用。**
- 自定义scope：很少使用。

接下来咱们自定义一个Scope，线程级别的Scope，在同一个线程中，获取的Bean都是同一个。跨线程则是不同的对象：（以下内容作为了解）

- 第一步：自定义Scope。（实现Scope接口）
   - spring内置了线程范围的类：org.springframework.context.support.SimpleThreadScope，可以直接用。
- 第二步：将自定义的Scope注册到Spring容器中。
```xml
<bean class="org.springframework.beans.factory.config.CustomScopeConfigurer">
  <property name="scopes">
    <map>
      <entry key="myThread">
        <bean class="org.springframework.context.support.SimpleThreadScope"/>
      </entry>
    </map>
  </property>
</bean>
```

- 第三步：使用Scope。
```xml
<bean id="sb" class="com.powernode.spring6.beans.SpringBean" scope="myThread" />
```
编写测试程序：
```java
@Test
public void testCustomScope(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-scope.xml");
    SpringBean sb1 = applicationContext.getBean("sb", SpringBean.class);
    SpringBean sb2 = applicationContext.getBean("sb", SpringBean.class);
    System.out.println(sb1);
    System.out.println(sb2);
    // 启动线程
    new Thread(new Runnable() {
        @Override
        public void run() {
            SpringBean a = applicationContext.getBean("sb", SpringBean.class);
            SpringBean b = applicationContext.getBean("sb", SpringBean.class);
            System.out.println(a);
            System.out.println(b);
        }
    }).start();
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665297614749-ae92b97d-85fd-4792-af87-e72d35784187.png#averageHue=%23343331&clientId=u3fe1442a-4567-4&from=paste&height=260&id=u600551ec&originHeight=260&originWidth=587&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43796&status=done&style=shadow&taskId=u14c4b472-78e4-43c6-b593-93b39a42d3f&title=&width=587)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=zGjYi&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 六、GoF之工厂模式

- 设计模式：一种可以被重复利用的解决方案。
- GoF（Gang of Four），中文名——四人组。
- 《Design Patterns: Elements of Reusable Object-Oriented Software》（即《设计模式》一书），1995年由 Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 合著。这几位作者常被称为"四人组（Gang of Four）"。
- 该书中描述了23种设计模式。我们平常所说的设计模式就是指这23种设计模式。
- 不过除了GoF23种设计模式之外，还有其它的设计模式，比如：JavaEE的设计模式（DAO模式、MVC模式等）。
- GoF23种设计模式可分为三大类：
   - **创建型**（5个）：解决对象创建问题。
      - 单例模式
      - 工厂方法模式
      - 抽象工厂模式
      - 建造者模式
      - 原型模式
   - **结构型**（7个）：一些类或对象组合在一起的经典结构。
      - 代理模式
      - 装饰模式
      - 适配器模式
      - 组合模式
      - 享元模式
      - 外观模式
      - 桥接模式
   - **行为型**（11个）：解决类或对象之间的交互问题。
      - 策略模式
      - 模板方法模式
      - 责任链模式
      - 观察者模式
      - 迭代子模式
      - 命令模式
      - 备忘录模式
      - 状态模式
      - 访问者模式
      - 中介者模式
      - 解释器模式
- 工厂模式是解决对象创建问题的，所以工厂模式属于创建型设计模式。这里为什么学习工厂模式呢？这是因为Spring框架底层使用了大量的工厂模式。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=CRGjo&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 6.1 工厂模式的三种形态
工厂模式通常有三种形态：

- 第一种：**简单工厂模式（Simple Factory）：不属于23种设计模式之一。简单工厂模式又叫做：静态 工厂方法模式。简单工厂模式是工厂方法模式的一种特殊实现。**
- 第二种：工厂方法模式（Factory Method）：是23种设计模式之一。
- 第三种：抽象工厂模式（Abstract Factory）：是23种设计模式之一。
## 6.2 简单工厂模式
简单工厂模式的角色包括三个：

- 抽象产品 角色
- 具体产品 角色
- 工厂类 角色

简单工厂模式的代码如下：
抽象产品角色：
```java
package com.powernode.factory;

/**
 * 武器（抽象产品角色）
 * @author 动力节点
 * @version 1.0
 * @className Weapon
 * @since 1.0
 **/
public abstract class Weapon {
    /**
     * 所有的武器都有攻击行为
     */
    public abstract void attack();
}
```
具体产品角色：
```java
package com.powernode.factory;

/**
 * 坦克（具体产品角色）
 * @author 动力节点
 * @version 1.0
 * @className Tank
 * @since 1.0
 **/
public class Tank extends Weapon{
    @Override
    public void attack() {
        System.out.println("坦克开炮！");
    }
}

```
```java
package com.powernode.factory;

/**
 * 战斗机（具体产品角色）
 * @author 动力节点
 * @version 1.0
 * @className Fighter
 * @since 1.0
 **/
public class Fighter extends Weapon{
    @Override
    public void attack() {
        System.out.println("战斗机投下原子弹！");
    }
}

```
```java
package com.powernode.factory;

/**
 * 匕首（具体产品角色）
 * @author 动力节点
 * @version 1.0
 * @className Dagger
 * @since 1.0
 **/
public class Dagger extends Weapon{
    @Override
    public void attack() {
        System.out.println("砍他丫的！");
    }
}

```
工厂类角色：
```java
package com.powernode.factory;

/**
 * 工厂类角色
 * @author 动力节点
 * @version 1.0
 * @className WeaponFactory
 * @since 1.0
 **/
public class WeaponFactory {
    /**
     * 根据不同的武器类型生产武器
     * @param weaponType 武器类型
     * @return 武器对象
     */
    public static Weapon get(String weaponType){
        if (weaponType == null || weaponType.trim().length() == 0) {
            return null;
        }
        Weapon weapon = null;
        if ("TANK".equals(weaponType)) {
            weapon = new Tank();
        } else if ("FIGHTER".equals(weaponType)) {
            weapon = new Fighter();
        } else if ("DAGGER".equals(weaponType)) {
            weapon = new Dagger();
        } else {
            throw new RuntimeException("不支持该武器！");
        }
        return weapon;
    }
}

```
测试程序（客户端程序）：
```java
package com.powernode.factory;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        Weapon weapon1 = WeaponFactory.get("TANK");
        weapon1.attack();

        Weapon weapon2 = WeaponFactory.get("FIGHTER");
        weapon2.attack();

        Weapon weapon3 = WeaponFactory.get("DAGGER");
        weapon3.attack();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665304945315-8bd0c855-6eff-44a8-8051-42a2c1edb712.png#averageHue=%23987f5e&clientId=u3fe1442a-4567-4&from=paste&height=173&id=u234858d4&originHeight=173&originWidth=384&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12115&status=done&style=shadow&taskId=uf0965380-b300-4c9c-8700-d47c0722e98&title=&width=384)
简单工厂模式的优点：

- 客户端程序不需要关心对象的创建细节，需要哪个对象时，只需要向工厂索要即可，初步实现了责任的分离。客户端只负责“消费”，工厂负责“生产”。生产和消费分离。

简单工厂模式的缺点：

- 缺点1：工厂类集中了所有产品的创造逻辑，形成一个无所不知的全能类，有人把它叫做上帝类。显然工厂类非常关键，不能出问题，一旦出问题，整个系统瘫痪。
- 缺点2：不符合OCP开闭原则，在进行系统扩展时，需要修改工厂类。

**Spring中的BeanFactory就使用了简单工厂模式。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=cZwbk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 6.3 工厂方法模式
工厂方法模式既保留了简单工厂模式的优点，同时又解决了简单工厂模式的缺点。
工厂方法模式的角色包括：

- **抽象工厂角色**
- **具体工厂角色**
- 抽象产品角色
- 具体产品角色

代码如下：
```java
package com.powernode.factory;

/**
 * 武器类（抽象产品角色）
 * @author 动力节点
 * @version 1.0
 * @className Weapon
 * @since 1.0
 **/
public abstract class Weapon {
    /**
     * 所有武器都有攻击行为
     */
    public abstract void attack();
}

```
```java
package com.powernode.factory;

/**
 * 具体产品角色
 * @author 动力节点
 * @version 1.0
 * @className Gun
 * @since 1.0
 **/
public class Gun extends Weapon{
    @Override
    public void attack() {
        System.out.println("开枪射击！");
    }
}

```
```java
package com.powernode.factory;

/**
 * 具体产品角色
 * @author 动力节点
 * @version 1.0
 * @className Fighter
 * @since 1.0
 **/
public class Fighter extends Weapon{
    @Override
    public void attack() {
        System.out.println("战斗机发射核弹！");
    }
}

```
```java
package com.powernode.factory;

/**
 * 武器工厂接口(抽象工厂角色)
 * @author 动力节点
 * @version 1.0
 * @className WeaponFactory
 * @since 1.0
 **/
public interface WeaponFactory {
    Weapon get();
}

```
```java
package com.powernode.factory;

/**
 * 具体工厂角色
 * @author 动力节点
 * @version 1.0
 * @className GunFactory
 * @since 1.0
 **/
public class GunFactory implements WeaponFactory{
    @Override
    public Weapon get() {
        return new Gun();
    }
}

```
```java
package com.powernode.factory;

/**
 * 具体工厂角色
 * @author 动力节点
 * @version 1.0
 * @className FighterFactory
 * @since 1.0
 **/
public class FighterFactory implements WeaponFactory{
    @Override
    public Weapon get() {
        return new Fighter();
    }
}

```
客户端程序：
```java
package com.powernode.factory;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        WeaponFactory factory = new GunFactory();
        Weapon weapon = factory.get();
        weapon.attack();

        WeaponFactory factory1 = new FighterFactory();
        Weapon weapon1 = factory1.get();
        weapon1.attack();
    }
}

```
执行客户端程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665362593949-73061a01-391f-4926-ba24-575903fd11bb.png#averageHue=%23312f2e&clientId=ue2397093-2e4b-4&from=paste&height=129&id=u2f6b7660&originHeight=129&originWidth=304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10662&status=done&style=shadow&taskId=ub809e05d-18e9-4117-9550-04b28e52a38&title=&width=304)
如果想扩展一个新的产品，只要新增一个产品类，再新增一个该产品对应的工厂即可，例如新增：匕首
```java
package com.powernode.factory;

/**
 * @author 动力节点
 * @version 1.0
 * @className Dagger
 * @since 1.0
 **/
public class Dagger extends Weapon{
    @Override
    public void attack() {
        System.out.println("砍丫的！");
    }
}

```
```java
package com.powernode.factory;

/**
 * @author 动力节点
 * @version 1.0
 * @className DaggerFactory
 * @since 1.0
 **/
public class DaggerFactory implements WeaponFactory{
    @Override
    public Weapon get() {
        return new Dagger();
    }
}

```
客户端程序：
```java
package com.powernode.factory;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        WeaponFactory factory = new GunFactory();
        Weapon weapon = factory.get();
        weapon.attack();

        WeaponFactory factory1 = new FighterFactory();
        Weapon weapon1 = factory1.get();
        weapon1.attack();

        WeaponFactory factory2 = new DaggerFactory();
        Weapon weapon2 = factory2.get();
        weapon2.attack();
    }
}

```
执行结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665362890109-5db8f42d-677b-450d-bc76-6842abe9640a.png#averageHue=%23997f5c&clientId=ue2397093-2e4b-4&from=paste&height=162&id=u7031050e&originHeight=162&originWidth=286&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11869&status=done&style=shadow&taskId=u4680145a-70fd-4503-a3db-d4108e22bd5&title=&width=286)
我们可以看到在进行功能扩展的时候，不需要修改之前的源代码，显然工厂方法模式符合OCP原则。
工厂方法模式的优点：

- 一个调用者想创建一个对象，只要知道其名称就可以了。 
- 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。
- 屏蔽产品的具体实现，调用者只关心产品的接口。

工厂方法模式的缺点：

- 每次增加一个产品时，都需要增加一个具体类和对象实现工厂，使得系统中类的个数成倍增加，在一定程度上增加了系统的复杂度，同时也增加了系统具体类的依赖。这并不是什么好事。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=oiHyc&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 6.4 抽象工厂模式（了解）
抽象工厂模式相对于工厂方法模式来说，就是工厂方法模式是针对一个产品系列的，而抽象工厂模式是针对多个产品系列的，即工厂方法模式是一个产品系列一个工厂类，而抽象工厂模式是多个产品系列一个工厂类。
抽象工厂模式特点：抽象工厂模式是所有形态的工厂模式中最为抽象和最具一般性的一种形态。抽象工厂模式是指当有多个抽象角色时，使用的一种工厂模式。抽象工厂模式可以向客户端提供一个接口，使客户端在不必指定产品的具体的情况下，创建多个产品族中的产品对象。它有多个抽象产品类，每个抽象产品类可以派生出多个具体产品类，一个抽象工厂类，可以派生出多个具体工厂类，每个具体工厂类可以创建多个具体产品类的实例。每一个模式都是针对一定问题的解决方案，工厂方法模式针对的是一个产品等级结构；而抽象工厂模式针对的是多个产品等级结果。
抽象工厂中包含4个角色：

- 抽象工厂角色
- 具体工厂角色
- 抽象产品角色
- 具体产品角色

抽象工厂模式的类图如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665370116084-46b714b8-95d2-45c5-89b6-564057c45694.png#averageHue=%23f9f9f9&clientId=ue2397093-2e4b-4&from=paste&height=663&id=uc5dac596&originHeight=663&originWidth=1153&originalType=binary&ratio=1&rotation=0&showTitle=false&size=44427&status=done&style=shadow&taskId=u24331dba-1131-47f8-b34e-677139adbc8&title=&width=1153)
抽象工厂模式代码如下：
第一部分：武器产品族
```java
package com.powernode.product;

/**
 * 武器产品族
 * @author 动力节点
 * @version 1.0
 * @className Weapon
 * @since 1.0
 **/
public abstract class Weapon {
    public abstract void attack();
}

```
```java
package com.powernode.product;

/**
 * 武器产品族中的产品等级1
 * @author 动力节点
 * @version 1.0
 * @className Gun
 * @since 1.0
 **/
public class Gun extends Weapon{
    @Override
    public void attack() {
        System.out.println("开枪射击！");
    }
}

```
```java
package com.powernode.product;

/**
 * 武器产品族中的产品等级2
 * @author 动力节点
 * @version 1.0
 * @className Dagger
 * @since 1.0
 **/
public class Dagger extends Weapon{
    @Override
    public void attack() {
        System.out.println("砍丫的！");
    }
}

```
第二部分：水果产品族
```java
package com.powernode.product;

/**
 * 水果产品族
 * @author 动力节点
 * @version 1.0
 * @className Fruit
 * @since 1.0
 **/
public abstract class Fruit {
    /**
     * 所有果实都有一个成熟周期。
     */
    public abstract void ripeCycle();
}

```
```java
package com.powernode.product;

/**
 * 水果产品族中的产品等级1
 * @author 动力节点
 * @version 1.0
 * @className Orange
 * @since 1.0
 **/
public class Orange extends Fruit{
    @Override
    public void ripeCycle() {
        System.out.println("橘子的成熟周期是10个月");
    }
}

```
```java
package com.powernode.product;

/**
 * 水果产品族中的产品等级2
 * @author 动力节点
 * @version 1.0
 * @className Apple
 * @since 1.0
 **/
public class Apple extends Fruit{
    @Override
    public void ripeCycle() {
        System.out.println("苹果的成熟周期是8个月");
    }
}

```
第三部分：抽象工厂类
```java
package com.powernode.factory;

import com.powernode.product.Fruit;
import com.powernode.product.Weapon;

/**
 * 抽象工厂
 * @author 动力节点
 * @version 1.0
 * @className AbstractFactory
 * @since 1.0
 **/
public abstract class AbstractFactory {
    public abstract Weapon getWeapon(String type);
    public abstract Fruit getFruit(String type);
}

```
第四部分：具体工厂类
```java
package com.powernode.factory;

import com.powernode.product.Dagger;
import com.powernode.product.Fruit;
import com.powernode.product.Gun;
import com.powernode.product.Weapon;

/**
 * 武器族工厂
 * @author 动力节点
 * @version 1.0
 * @className WeaponFactory
 * @since 1.0
 **/
public class WeaponFactory extends AbstractFactory{

    public Weapon getWeapon(String type){
        if (type == null || type.trim().length() == 0) {
            return null;
        }
        if ("Gun".equals(type)) {
            return new Gun();
        } else if ("Dagger".equals(type)) {
            return new Dagger();
        } else {
            throw new RuntimeException("无法生产该武器");
        }
    }

    @Override
    public Fruit getFruit(String type) {
        return null;
    }
}

```
```java
package com.powernode.factory;

import com.powernode.product.*;

/**
 * 水果族工厂
 * @author 动力节点
 * @version 1.0
 * @className FruitFactory
 * @since 1.0
 **/
public class FruitFactory extends AbstractFactory{
    @Override
    public Weapon getWeapon(String type) {
        return null;
    }

    public Fruit getFruit(String type){
        if (type == null || type.trim().length() == 0) {
            return null;
        }
        if ("Orange".equals(type)) {
            return new Orange();
        } else if ("Apple".equals(type)) {
            return new Apple();
        } else {
            throw new RuntimeException("我家果园不产这种水果");
        }
    }
}

```
第五部分：客户端程序
```java
package com.powernode.client;

import com.powernode.factory.AbstractFactory;
import com.powernode.factory.FruitFactory;
import com.powernode.factory.WeaponFactory;
import com.powernode.product.Fruit;
import com.powernode.product.Weapon;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 客户端调用方法时只面向AbstractFactory调用方法。
        AbstractFactory factory = new WeaponFactory(); // 注意：这里的new WeaponFactory()可以采用 简单工厂模式 进行隐藏。
        Weapon gun = factory.getWeapon("Gun");
        Weapon dagger = factory.getWeapon("Dagger");

        gun.attack();
        dagger.attack();

        AbstractFactory factory1 = new FruitFactory(); // 注意：这里的new FruitFactory()可以采用 简单工厂模式 进行隐藏。
        Fruit orange = factory1.getFruit("Orange");
        Fruit apple = factory1.getFruit("Apple");

        orange.ripeCycle();
        apple.ripeCycle();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665370862172-3753f689-d7c7-40d8-8a50-1e4144f2be97.png#averageHue=%23302f2d&clientId=ue2397093-2e4b-4&from=paste&height=202&id=u96334482&originHeight=202&originWidth=346&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16080&status=done&style=shadow&taskId=u9660924d-c327-4950-b959-efa0c4447f5&title=&width=346)
抽象工厂模式的优缺点：

- 优点：当一个产品族中的多个对象被设计成一起工作时，它能保证客户端始终只使用同一个产品族中的对象。
- 缺点：产品族扩展非常困难，要增加一个系列的某一产品，既要在AbstractFactory里加代码，又要在具体的里面加代码。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=NcFOX&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 七、Bean的实例化方式
Spring为Bean提供了多种实例化方式，通常包括4种方式。（也就是说在Spring中为Bean对象的创建准备了多种方案，目的是：更加灵活）

- 第一种：通过构造方法实例化
- 第二种：通过简单工厂模式实例化
- 第三种：通过factory-bean实例化
- 第四种：通过FactoryBean接口实例化
## 7.1 通过构造方法实例化
我们之前一直使用的就是这种方式。默认情况下，会调用Bean的无参数构造方法。
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    public User() {
        System.out.println("User类的无参数构造方法执行。");
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userBean" class="com.powernode.spring6.bean.User"/>

</beans>
```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className SpringInstantiationTest
 * @since 1.0
 **/
public class SpringInstantiationTest {

    @Test
    public void testConstructor(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User user = applicationContext.getBean("userBean", User.class);
        System.out.println(user);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665373293580-f86edf28-3303-44bd-a9a1-855dcd575e0d.png#averageHue=%238c7860&clientId=ue2397093-2e4b-4&from=paste&height=147&id=uece10800&originHeight=147&originWidth=516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18320&status=done&style=shadow&taskId=u00117d3f-b761-4f9e-851c-743a1f16c41&title=&width=516)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=yeSfh&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 7.2 通过简单工厂模式实例化
第一步：定义一个Bean
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Vip
 * @since 1.0
 **/
public class Vip {
}

```
第二步：编写简单工厂模式当中的工厂类
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className VipFactory
 * @since 1.0
 **/
public class VipFactory {
    public static Vip get(){
        return new Vip();
    }
}

```
第三步：在Spring配置文件中指定创建该Bean的方法（使用factory-method属性指定）
```xml
<bean id="vipBean" class="com.powernode.spring6.bean.VipFactory" factory-method="get"/>
```
第四步：编写测试程序
```java
@Test
public void testSimpleFactory(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Vip vip = applicationContext.getBean("vipBean", Vip.class);
    System.out.println(vip);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665373835672-90312dd8-81e3-4d0a-8b2d-06f90a3e9203.png#averageHue=%23978971&clientId=ue2397093-2e4b-4&from=paste&height=103&id=ucbd70ed3&originHeight=103&originWidth=505&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13066&status=done&style=shadow&taskId=u92f4799b-47c9-4a27-a6b3-3853b3e75b2&title=&width=505)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=EOPxO&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 7.3 通过factory-bean实例化
这种方式本质上是：通过工厂方法模式进行实例化。
第一步：定义一个Bean
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Order
 * @since 1.0
 **/
public class Order {
}

```
第二步：定义具体工厂类，工厂类中定义实例方法
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderFactory
 * @since 1.0
 **/
public class OrderFactory {
    public Order get(){
        return new Order();
    }
}

```
第三步：在Spring配置文件中指定factory-bean以及factory-method
```xml
<bean id="orderFactory" class="com.powernode.spring6.bean.OrderFactory"/>
<bean id="orderBean" factory-bean="orderFactory" factory-method="get"/>
```
第四步：编写测试程序
```java
@Test
public void testSelfFactoryBean(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Order orderBean = applicationContext.getBean("orderBean", Order.class);
    System.out.println(orderBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665374686365-c2f211e2-8594-4994-8b37-09259057ad8d.png#averageHue=%238e7b65&clientId=ue2397093-2e4b-4&from=paste&height=121&id=uafa4f581&originHeight=121&originWidth=492&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12681&status=done&style=shadow&taskId=u14bd9486-794a-4c69-9656-6d17f045150&title=&width=492)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=pl4su&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 7.4 通过FactoryBean接口实例化
以上的第三种方式中，factory-bean是我们自定义的，factory-method也是我们自己定义的。
在Spring中，当你编写的类直接实现FactoryBean接口之后，factory-bean不需要指定了，factory-method也不需要指定了。
factory-bean会自动指向实现FactoryBean接口的类，factory-method会自动指向getObject()方法。
第一步：定义一个Bean
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Person
 * @since 1.0
 **/
public class Person {
}

```
第二步：编写一个类实现FactoryBean接口
```java
package com.powernode.spring6.bean;

import org.springframework.beans.factory.FactoryBean;

/**
 * @author 动力节点
 * @version 1.0
 * @className PersonFactoryBean
 * @since 1.0
 **/
public class PersonFactoryBean implements FactoryBean<Person> {

    @Override
    public Person getObject() throws Exception {
        return new Person();
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }

    @Override
    public boolean isSingleton() {
        // true表示单例
        // false表示原型
        return true;
    }
}

```
第三步：在Spring配置文件中配置FactoryBean
```xml
<bean id="personBean" class="com.powernode.spring6.bean.PersonFactoryBean"/>
```
测试程序：
```java
@Test
public void testFactoryBean(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Person personBean = applicationContext.getBean("personBean", Person.class);
    System.out.println(personBean);

    Person personBean2 = applicationContext.getBean("personBean", Person.class);
    System.out.println(personBean2);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665382305162-81e16b33-be0f-4256-ae49-89a59c946823.png#averageHue=%23363433&clientId=ue2397093-2e4b-4&from=paste&height=134&id=u5096bbb8&originHeight=134&originWidth=522&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19204&status=done&style=shadow&taskId=ue23685a0-5a97-4a48-a847-ba4fd44d062&title=&width=522)
**FactoryBean在Spring中是一个接口。被称为“工厂Bean”。“工厂Bean”是一种特殊的Bean。所有的“工厂Bean”都是用来协助Spring框架来创建其他Bean对象的。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Yx0oj&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 7.5 BeanFactory和FactoryBean的区别
### 7.5.1 BeanFactory
Spring IoC容器的顶级对象，BeanFactory被翻译为“Bean工厂”，在Spring的IoC容器中，“Bean工厂”负责创建Bean对象。
BeanFactory是工厂。
### 7.5.2 FactoryBean
FactoryBean：它是一个Bean，是一个能够**辅助Spring**实例化其它Bean对象的一个Bean。
在Spring中，Bean可以分为两类：

- 第一类：普通Bean
- 第二类：工厂Bean（记住：工厂Bean也是一种Bean，只不过这种Bean比较特殊，它可以辅助Spring实例化其它Bean对象。）
## 7.6 注入自定义Date
我们前面说过，java.util.Date在Spring中被当做简单类型，简单类型在注入的时候可以直接使用value属性或value标签来完成。但我们之前已经测试过了，对于Date类型来说，采用value属性或value标签赋值的时候，对日期字符串的格式要求非常严格，必须是这种格式的：Mon Oct 10 14:30:26 CST 2022。其他格式是不会被识别的。如以下代码：
```java
package com.powernode.spring6.bean;

import java.util.Date;

/**
 * @author 动力节点
 * @version 1.0
 * @className Student
 * @since 1.0
 **/
public class Student {
    private Date birth;

    public void setBirth(Date birth) {
        this.birth = birth;
    }

    @Override
    public String toString() {
        return "Student{" +
                "birth=" + birth +
                '}';
    }
}

```
```xml
<bean id="studentBean" class="com.powernode.spring6.bean.Student">
  <property name="birth" value="Mon Oct 10 14:30:26 CST 2002"/>
</bean>
```
```java
@Test
public void testDate(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    Student studentBean = applicationContext.getBean("studentBean", Student.class);
    System.out.println(studentBean);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665383744481-75de8e77-ac4e-46b8-90dc-1cd5264f66f2.png#averageHue=%238c7a63&clientId=ue2397093-2e4b-4&from=paste&height=109&id=u5dcba6e9&originHeight=109&originWidth=529&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12763&status=done&style=shadow&taskId=u440ccfca-0f20-4c75-a9bb-e9bba89015d&title=&width=529)
如果把日期格式修改一下：
```xml
<bean id="studentBean" class="com.powernode.spring6.bean.Student">
  <property name="birth" value="2002-10-10"/>
</bean>
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665383871708-89cd2ac9-6d31-40fc-a4a8-27d27cd35ad2.png#averageHue=%232e2c2b&clientId=ue2397093-2e4b-4&from=paste&height=203&id=u2c539d66&originHeight=203&originWidth=1304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11880&status=done&style=shadow&taskId=u582a716c-fbd7-4fb7-b582-483723b0b40&title=&width=1304)
这种情况下，我们就可以使用FactoryBean来完成这个骚操作。
编写DateFactoryBean实现FactoryBean接口：
```java
package com.powernode.spring6.bean;

import org.springframework.beans.factory.FactoryBean;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * @author 动力节点
 * @version 1.0
 * @className DateFactoryBean
 * @since 1.0
 **/
public class DateFactoryBean implements FactoryBean<Date> {

    // 定义属性接收日期字符串
    private String date;

    // 通过构造方法给日期字符串属性赋值
    public DateFactoryBean(String date) {
        this.date = date;
    }

    @Override
    public Date getObject() throws Exception {
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
        return sdf.parse(this.date);
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}

```
编写spring配置文件：
```xml
<bean id="dateBean" class="com.powernode.spring6.bean.DateFactoryBean">
  <constructor-arg name="date" value="1999-10-11"/>
</bean>

<bean id="studentBean" class="com.powernode.spring6.bean.Student">
  <property name="birth" ref="dateBean"/>
</bean>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665384407377-3fb70ee0-09d6-4d44-904a-10af88cb1a2a.png#averageHue=%238e7c64&clientId=ue2397093-2e4b-4&from=paste&height=111&id=u5c409f3e&originHeight=111&originWidth=532&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12080&status=done&style=shadow&taskId=uc711c26d-daa6-49df-802c-7c4a55e8f40&title=&width=532)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=nosVo&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 八、Bean的生命周期
## 8.1 什么是Bean的生命周期
Spring其实就是一个管理Bean对象的工厂。它负责对象的创建，对象的销毁等。
所谓的生命周期就是：对象从创建开始到最终销毁的整个过程。
什么时候创建Bean对象？
创建Bean对象的前后会调用什么方法？
Bean对象什么时候销毁？
Bean对象的销毁前后调用什么方法？
## 8.2 为什么要知道Bean的生命周期
其实生命周期的本质是：在哪个时间节点上调用了哪个类的哪个方法。
我们需要充分的了解在这个生命线上，都有哪些特殊的时间节点。
只有我们知道了特殊的时间节点都在哪，到时我们才可以确定代码写到哪。
我们可能需要在某个特殊的时间点上执行一段特定的代码，这段代码就可以放到这个节点上。当生命线走到这里的时候，自然会被调用。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=tGNfW&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 8.3 Bean的生命周期之5步
Bean生命周期的管理，可以参考Spring的源码：**AbstractAutowireCapableBeanFactory类的doCreateBean()方法。**
Bean生命周期可以粗略的划分为五大步：

- 第一步：实例化Bean
- 第二步：Bean属性赋值
- 第三步：初始化Bean
- 第四步：使用Bean
- 第五步：销毁Bean

![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665388735200-444405f6-283d-4b3a-8cdf-8c3e01743618.png#averageHue=%23f6f6f6&clientId=ue2397093-2e4b-4&from=paste&height=142&id=u7c6b9a1a&originHeight=142&originWidth=851&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11129&status=done&style=shadow&taskId=u288cbb6f-b738-43ff-ac53-6eb841c29fc&title=&width=851)
编写测试程序：
定义一个Bean
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    private String name;

    public User() {
        System.out.println("1.实例化Bean");
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("2.Bean属性赋值");
    }

    public void initBean(){
        System.out.println("3.初始化Bean");
    }

    public void destroyBean(){
        System.out.println("5.销毁Bean");
    }

}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
    init-method属性指定初始化方法。
    destroy-method属性指定销毁方法。
    -->
    <bean id="userBean" class="com.powernode.spring6.bean.User" init-method="initBean" destroy-method="destroyBean">
        <property name="name" value="zhangsan"/>
    </bean>

</beans>
```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className BeanLifecycleTest
 * @since 1.0
 **/
public class BeanLifecycleTest {
    @Test
    public void testLifecycle(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User userBean = applicationContext.getBean("userBean", User.class);
        System.out.println("4.使用Bean");
        // 只有正常关闭spring容器才会执行销毁方法
        ClassPathXmlApplicationContext context = (ClassPathXmlApplicationContext) applicationContext;
        context.close();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665392526476-d0efb004-51bf-4eef-bc8c-d3b6a315ee39.png#averageHue=%2331302f&clientId=ue2397093-2e4b-4&from=paste&height=229&id=u52c8e190&originHeight=229&originWidth=340&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18954&status=done&style=shadow&taskId=u4d91507f-2f03-486a-9b77-0280f1f20b1&title=&width=340)
需要注意的：

- 第一：只有正常关闭spring容器，bean的销毁方法才会被调用。
- 第二：ClassPathXmlApplicationContext类才有close()方法。
- 第三：配置文件中的init-method指定初始化方法。destroy-method指定销毁方法。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=aAzZn&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 8.4 Bean生命周期之7步
在以上的5步中，第3步是初始化Bean，如果你还想在初始化前和初始化后添加代码，可以加入“Bean后处理器”。
编写一个类实现BeanPostProcessor类，并且重写before和after方法：
```java
package com.powernode.spring6.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * @author 动力节点
 * @version 1.0
 * @className LogBeanPostProcessor
 * @since 1.0
 **/
public class LogBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Bean后处理器的before方法执行，即将开始初始化");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Bean后处理器的after方法执行，已完成初始化");
        return bean;
    }
}

```
在spring.xml文件中配置“Bean后处理器”：
```xml
<!--配置Bean后处理器。这个后处理器将作用于当前配置文件中所有的bean。-->
<bean class="com.powernode.spring6.bean.LogBeanPostProcessor"/>
```
**一定要注意：在spring.xml文件中配置的Bean后处理器将作用于当前配置文件中所有的Bean。**
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665393219244-4763ce2a-1cec-4b67-b3b4-54c2d28bc46a.png#averageHue=%23302f2e&clientId=ue2397093-2e4b-4&from=paste&height=283&id=u5f23f762&originHeight=283&originWidth=544&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33201&status=done&style=shadow&taskId=u7ee591f2-bf31-4f9c-8ab0-bbcafcf4933&title=&width=544)
如果加上Bean后处理器的话，Bean的生命周期就是7步了：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665393936765-0ea5dcdd-859a-4ac5-9407-f06022c498b9.png#averageHue=%23f6f6f6&clientId=ue2397093-2e4b-4&from=paste&height=170&id=u040e6fe3&originHeight=170&originWidth=1015&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15217&status=done&style=shadow&taskId=u35a0a713-9831-44bb-87af-8016c399b84&title=&width=1015)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=CKxZ0&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 8.5 Bean生命周期之10步
如果根据源码跟踪，可以划分更细粒度的步骤，10步：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665394697870-15de433a-8d50-4b31-9b75-b2ca7090c1c6.png#averageHue=%23f6f6f6&clientId=ue2397093-2e4b-4&from=paste&height=884&id=ud2630984&originHeight=884&originWidth=432&originalType=binary&ratio=1&rotation=0&showTitle=false&size=42470&status=done&style=shadow&taskId=u36985516-859f-4030-b591-be4f30d3dc6&title=&width=432)
上图中检查Bean是否实现了Aware的相关接口是什么意思？
Aware相关的接口包括：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware

- 当Bean实现了BeanNameAware，Spring会将Bean的名字传递给Bean。
- 当Bean实现了BeanClassLoaderAware，Spring会将加载该Bean的类加载器传递给Bean。
- 当Bean实现了BeanFactoryAware，Spring会将Bean工厂对象传递给Bean。

测试以上10步，可以让User类实现5个接口，并实现所有方法：

- BeanNameAware
- BeanClassLoaderAware
- BeanFactoryAware
- InitializingBean
- DisposableBean

代码如下：
```java
package com.powernode.spring6.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.*;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware, InitializingBean, DisposableBean {
    private String name;

    public User() {
        System.out.println("1.实例化Bean");
    }

    public void setName(String name) {
        this.name = name;
        System.out.println("2.Bean属性赋值");
    }

    public void initBean(){
        System.out.println("6.初始化Bean");
    }

    public void destroyBean(){
        System.out.println("10.销毁Bean");
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("3.类加载器：" + classLoader);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("3.Bean工厂：" + beanFactory);
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("3.bean名字：" + name);
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("9.DisposableBean destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("5.afterPropertiesSet执行");
    }
}

```
```java
package com.powernode.spring6.bean;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

/**
 * @author 动力节点
 * @version 1.0
 * @className LogBeanPostProcessor
 * @since 1.0
 **/
public class LogBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("4.Bean后处理器的before方法执行，即将开始初始化");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("7.Bean后处理器的after方法执行，已完成初始化");
        return bean;
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665395466500-d95b1a58-24e1-46f0-b72a-aa7764b0336a.png#averageHue=%232e2e2d&clientId=ue2397093-2e4b-4&from=paste&height=443&id=uc58b1500&originHeight=443&originWidth=1001&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64711&status=done&style=shadow&taskId=u108c9520-781d-4773-a8e7-932627d0c76&title=&width=1001)
**通过测试可以看出来：**

- **InitializingBean的方法早于init-method的执行。**
- **DisposableBean的方法早于destroy-method的执行。**

对于SpringBean的生命周期，掌握之前的7步即可。够用。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=TbC9c&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 8.6 Bean的作用域不同，管理方式不同
Spring 根据Bean的作用域来选择管理方式。

- 对于singleton作用域的Bean，Spring 能够精确地知道该Bean何时被创建，何时初始化完成，以及何时被销毁；
- 而对于 prototype 作用域的 Bean，Spring 只负责创建，当容器创建了 Bean 的实例后，Bean 的实例就交给客户端代码管理，Spring 容器将不再跟踪其生命周期。

我们把之前User类的spring.xml文件中的配置scope设置为prototype：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--
    init-method属性指定初始化方法。
    destroy-method属性指定销毁方法。
    -->
    <bean id="userBean" class="com.powernode.spring6.bean.User" init-method="initBean" destroy-method="destroyBean" scope="prototype">
        <property name="name" value="zhangsan"/>
    </bean>

    <!--配置Bean后处理器。这个后处理器将作用于当前配置文件中所有的bean。-->
    <bean class="com.powernode.spring6.bean.LogBeanPostProcessor"/>

</beans>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665395807399-a3b71b4d-d871-4230-8fe4-b939ed03b301.png#averageHue=%232e2e2d&clientId=ue2397093-2e4b-4&from=paste&height=448&id=u5aa7090d&originHeight=448&originWidth=1009&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62362&status=done&style=shadow&taskId=ua7cc0d31-918e-409c-9919-af50bd7e0fb&title=&width=1009)
通过测试一目了然。只执行了前8步，第9和10都没有执行。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=i34Va&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 8.7 自己new的对象如何让Spring管理
有些时候可能会遇到这样的需求，某个java对象是我们自己new的，然后我们希望这个对象被Spring容器管理，怎么实现？
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
}

```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.Test;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;

/**
 * @author 动力节点
 * @version 1.0
 * @className RegisterBeanTest
 * @since 1.0
 **/
public class RegisterBeanTest {

    @Test
    public void testBeanRegister(){
        // 自己new的对象
        User user = new User();
        System.out.println(user);

        // 创建 默认可列表BeanFactory 对象
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        // 注册Bean
        factory.registerSingleton("userBean", user);
        // 从spring容器中获取bean
        User userBean = factory.getBean("userBean", User.class);
        System.out.println(userBean);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666262245551-b00bbba7-4107-4d44-8441-fbcd6f799293.png#averageHue=%23f6f4f1&clientId=u041108cd-a231-4&from=paste&height=146&id=u5e4b0447&originHeight=146&originWidth=525&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17970&status=done&style=shadow&taskId=u9977d128-17b7-4b7f-87e6-b43bcf4d012&title=&width=525)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=n0RFE&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 九、Bean的循环依赖问题
## 9.1 什么是Bean的循环依赖
A对象中有B属性。B对象中有A属性。这就是循环依赖。我依赖你，你也依赖我。
比如：丈夫类Husband，妻子类Wife。Husband中有Wife的引用。Wife中有Husband的引用。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665452274046-82594b87-2974-4e08-a6ab-2218d001d14f.png#averageHue=%23f5f5f5&clientId=ue12e8566-378b-4&from=paste&height=162&id=u6d2f623e&originHeight=162&originWidth=347&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4376&status=done&style=shadow&taskId=uaa31889d-a21b-4589-a039-281453ab56a&title=&width=347)
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Husband
 * @since 1.0
 **/
public class Husband {
    private String name;
    private Wife wife;
}

```
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Wife
 * @since 1.0
 **/
public class Wife {
    private String name;
    private Husband husband;
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=YL0Qp&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 9.2 singleton下的set注入产生的循环依赖
我们来编写程序，测试一下在singleton+setter的模式下产生的循环依赖，Spring是否能够解决？
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Husband
 * @since 1.0
 **/
public class Husband {
    private String name;
    private Wife wife;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setWife(Wife wife) {
        this.wife = wife;
    }

    // toString()方法重写时需要注意：不能直接输出wife，输出wife.getName()。要不然会出现递归导致的栈内存溢出错误。
    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife.getName() +
                '}';
    }
}

```
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Wife
 * @since 1.0
 **/
public class Wife {
    private String name;
    private Husband husband;

    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setHusband(Husband husband) {
        this.husband = husband;
    }

    // toString()方法重写时需要注意：不能直接输出husband，输出husband.getName()。要不然会出现递归导致的栈内存溢出错误。
    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband.getName() +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="husbandBean" class="com.powernode.spring6.bean.Husband" scope="singleton">
        <property name="name" value="张三"/>
        <property name="wife" ref="wifeBean"/>
    </bean>
    <bean id="wifeBean" class="com.powernode.spring6.bean.Wife" scope="singleton">
        <property name="name" value="小花"/>
        <property name="husband" ref="husbandBean"/>
    </bean>
</beans>
```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.Husband;
import com.powernode.spring6.bean.Wife;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className CircularDependencyTest
 * @since 1.0
 **/
public class CircularDependencyTest {

    @Test
    public void testSingletonAndSet(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        Husband husbandBean = applicationContext.getBean("husbandBean", Husband.class);
        Wife wifeBean = applicationContext.getBean("wifeBean", Wife.class);
        System.out.println(husbandBean);
        System.out.println(wifeBean);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665453201014-160bb88e-08d4-4d37-a1d9-44d4911a32df.png#averageHue=%238b7760&clientId=ue12e8566-378b-4&from=paste&height=149&id=u5b4b34dd&originHeight=149&originWidth=516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=16412&status=done&style=shadow&taskId=uf18aa1ed-430e-4ab5-9523-c0b0e54ba30&title=&width=516)
**通过测试得知：在singleton + set注入的情况下，循环依赖是没有问题的。Spring可以解决这个问题。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=smNUU&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 9.3 prototype下的set注入产生的循环依赖
我们再来测试一下：prototype+set注入的方式下，循环依赖会不会出现问题？
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="husbandBean" class="com.powernode.spring6.bean.Husband" scope="prototype">
        <property name="name" value="张三"/>
        <property name="wife" ref="wifeBean"/>
    </bean>
    <bean id="wifeBean" class="com.powernode.spring6.bean.Wife" scope="prototype">
        <property name="name" value="小花"/>
        <property name="husband" ref="husbandBean"/>
    </bean>
</beans>
```
执行测试程序：发生了异常，异常信息如下：
Caused by: org.springframework.beans.factory.**BeanCurrentlyInCreationException**: Error creating bean with name 'husbandBean': Requested bean is currently in creation: Is there an unresolvable circular reference?
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:265)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:325)
	... 44 more
翻译为：创建名为“husbandBean”的bean时出错：请求的bean当前正在创建中：是否存在无法解析的循环引用？
通过测试得知，当循环依赖的**所有Bean**的scope="prototype"的时候，产生的循环依赖，Spring是无法解决的，会出现**BeanCurrentlyInCreationException**异常。
大家可以测试一下，以上两个Bean，如果其中一个是singleton，另一个是prototype，是没有问题的。
为什么两个Bean都是prototype时会出错呢？
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665454469042-69668f45-5d71-494f-8537-18142d354abd.png#averageHue=%232f2c2b&clientId=ue12e8566-378b-4&from=paste&height=480&id=u51bd1a99&originHeight=480&originWidth=1140&originalType=binary&ratio=1&rotation=0&showTitle=false&size=92057&status=done&style=shadow&taskId=u3b8b4f66-7f8c-4735-ac25-5aba78db2d5&title=&width=1140)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=DKByo&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 9.4 singleton下的构造注入产生的循环依赖
我们再来测试一下singleton + 构造注入的方式下，spring是否能够解决这种循环依赖。
```java
package com.powernode.spring6.bean2;

/**
 * @author 动力节点
 * @version 1.0
 * @className Husband
 * @since 1.0
 **/
public class Husband {
    private String name;
    private Wife wife;

    public Husband(String name, Wife wife) {
        this.name = name;
        this.wife = wife;
    }

    // -----------------------分割线--------------------------------
    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Husband{" +
                "name='" + name + '\'' +
                ", wife=" + wife +
                '}';
    }
}

```
```java
package com.powernode.spring6.bean2;

/**
 * @author 动力节点
 * @version 1.0
 * @className Wife
 * @since 1.0
 **/
public class Wife {
    private String name;
    private Husband husband;

    public Wife(String name, Husband husband) {
        this.name = name;
        this.husband = husband;
    }

    // -------------------------分割线--------------------------------
    public String getName() {
        return name;
    }

    @Override
    public String toString() {
        return "Wife{" +
                "name='" + name + '\'' +
                ", husband=" + husband +
                '}';
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="hBean" class="com.powernode.spring6.bean2.Husband" scope="singleton">
        <constructor-arg name="name" value="张三"/>
        <constructor-arg name="wife" ref="wBean"/>
    </bean>

    <bean id="wBean" class="com.powernode.spring6.bean2.Wife" scope="singleton">
        <constructor-arg name="name" value="小花"/>
        <constructor-arg name="husband" ref="hBean"/>
    </bean>
</beans>
```
```java
@Test
public void testSingletonAndConstructor(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring2.xml");
    Husband hBean = applicationContext.getBean("hBean", Husband.class);
    Wife wBean = applicationContext.getBean("wBean", Wife.class);
    System.out.println(hBean);
    System.out.println(wBean);
}
```
执行结果：发生了异常，信息如下：
Caused by: org.springframework.beans.factory.**BeanCurrentlyInCreationException**: Error creating bean with name 'hBean': Requested bean is currently in creation: Is there an unresolvable circular reference?
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.beforeSingletonCreation(DefaultSingletonBeanRegistry.java:355)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:227)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:324)
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199)
	at org.springframework.beans.factory.support.BeanDefinitionValueResolver.resolveReference(BeanDefinitionValueResolver.java:325)
	... 56 more
和上一个测试结果相同，都是提示产生了循环依赖，并且Spring是无法解决这种循环依赖的。
为什么呢？
**主要原因是因为通过构造方法注入导致的：因为构造方法注入会导致实例化对象的过程和对象属性赋值的过程没有分离开，必须在一起完成导致的。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=f0oOk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 9.5 Spring解决循环依赖的机理
Spring为什么可以解决set + singleton模式下循环依赖？
根本的原因在于：这种方式可以做到将“实例化Bean”和“给Bean属性赋值”这两个动作分开去完成。
实例化Bean的时候：调用无参数构造方法来完成。**此时可以先不给属性赋值，可以提前将该Bean对象“曝光”给外界。**
给Bean属性赋值的时候：调用setter方法来完成。
两个步骤是完全可以分离开去完成的，并且这两步不要求在同一个时间点上完成。
也就是说，Bean都是单例的，我们可以先把所有的单例Bean实例化出来，放到一个集合当中（我们可以称之为缓存），所有的单例Bean全部实例化完成之后，以后我们再慢慢的调用setter方法给属性赋值。这样就解决了循环依赖的问题。
那么在Spring框架底层源码级别上是如何实现的呢？请看：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665456331018-18c45ae3-fa4c-4cd8-aabf-d9bace567693.png#averageHue=%23fcf9f8&clientId=ue12e8566-378b-4&from=paste&height=666&id=u56928305&originHeight=666&originWidth=1433&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98860&status=done&style=shadow&taskId=u7ea2c6fd-a463-45ab-b788-d07a902827c&title=&width=1433)
在以上类中包含三个重要的属性：
_**Cache of singleton objects: bean name to bean instance. **_**单例对象的缓存：key存储bean名称，value存储Bean对象【一级缓存】**
_**Cache of early singleton objects: bean name to bean instance. **_**早期单例对象的缓存：key存储bean名称，value存储早期的Bean对象【二级缓存】**
_**Cache of singleton factories: bean name to ObjectFactory. **_**单例工厂缓存：key存储bean名称，value存储该Bean对应的ObjectFactory对象【三级缓存】**
这三个缓存其实本质上是三个Map集合。
我们再来看，在该类中有这样一个方法addSingletonFactory()，这个方法的作用是：将创建Bean对象的ObjectFactory对象提前曝光。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665460724682-2222366d-cc07-43db-a8d0-fb27712b20a4.png#averageHue=%23fdfaf9&clientId=ue12e8566-378b-4&from=paste&height=463&id=u9c53eab2&originHeight=463&originWidth=1104&originalType=binary&ratio=1&rotation=0&showTitle=false&size=74936&status=done&style=shadow&taskId=ua1bb4340-c729-4663-9e06-baabf662874&title=&width=1104)
再分析下面的源码：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665460240687-3d0794c4-e6ed-4653-9463-767a7f943ff9.png#averageHue=%23fcf7f5&clientId=ue12e8566-378b-4&from=paste&height=934&id=u20f86432&originHeight=934&originWidth=1195&originalType=binary&ratio=1&rotation=0&showTitle=false&size=166762&status=done&style=shadow&taskId=u385636c1-8cf1-4a02-89d3-653f2316c5a&title=&width=1195)
从源码中可以看到，spring会先从一级缓存中获取Bean，如果获取不到，则从二级缓存中获取Bean，如果二级缓存还是获取不到，则从三级缓存中获取之前曝光的ObjectFactory对象，通过ObjectFactory对象获取Bean实例，这样就解决了循环依赖的问题。
**总结：**
**Spring只能解决setter方法注入的单例bean之间的循环依赖。ClassA依赖ClassB，ClassB又依赖ClassA，形成依赖闭环。Spring在创建ClassA对象后，不需要等给属性赋值，直接将其曝光到bean缓存当中。在解析ClassA的属性时，又发现依赖于ClassB，再次去获取ClassB，当解析ClassB的属性时，又发现需要ClassA的属性，但此时的ClassA已经被提前曝光加入了正在创建的bean的缓存中，则无需创建新的的ClassA的实例，直接从缓存中获取即可。从而解决循环依赖问题。**
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=GmDZk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十、回顾反射机制
## 10.1 分析方法四要素
我们先来看一下，不使用反射机制调用一个方法需要几个要素的参与。
有一个这样的类：
```java
package com.powernode.reflect;

/**
 * @author 动力节点
 * @version 1.0
 * @className SystemService
 * @since 1.0
 **/
public class SystemService {
    
    public void logout(){
        System.out.println("退出系统");
    }

    public boolean login(String username, String password){
        if ("admin".equals(username) && "admin123".equals(password)) {
            return true;
        }
        return false;
    }
}

```
编写程序调用方法：
```java
package com.powernode.reflect;

/**
 * @author 动力节点
 * @version 1.0
 * @className ReflectTest01
 * @since 1.0
 **/
public class ReflectTest01 {
    public static void main(String[] args) {

        // 创建对象
        SystemService systemService = new SystemService();

        // 调用方法并接收方法的返回值
        boolean success = systemService.login("admin", "admin123");

        System.out.println(success ? "登录成功" : "登录失败");
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665469402327-e4cdba7c-2441-4c37-bb6d-c7138ac19bc4.png#averageHue=%23f8f7f6&clientId=ue12e8566-378b-4&from=paste&height=101&id=u4a89f6d1&originHeight=101&originWidth=314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7453&status=done&style=shadow&taskId=u34beeead-9df4-43ea-baf8-fa39ba3f5f2&title=&width=314)
通过以上第16行代码可以看出，调用一个方法，一般涉及到4个要素：

- 调用哪个对象的（systemService）
- 哪个方法（login）
- 传什么参数（"admin", "admin123"）
- 返回什么值（success）

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=d78qG&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 10.2 获取Method
要使用反射机制调用一个方法，首先你要获取到这个方法。
在反射机制中Method实例代表的是一个方法。那么怎么获取Method实例呢？
有这样一个类：
```java
package com.powernode.reflect;

/**
 * @author 动力节点
 * @version 1.0
 * @className SystemService
 * @since 1.0
 **/
public class SystemService {

    public void logout(){
        System.out.println("退出系统");
    }

    public boolean login(String username, String password){
        if ("admin".equals(username) && "admin123".equals(password)) {
            return true;
        }
        return false;
    }
    
    public boolean login(String password){
        if("110".equals(password)){
            return true;
        }
        return false;
    }
}

```
我们如何获取到 logout()、login(String,String)、login(String) 这三个方法呢？
要获取方法Method，首先你需要获取这个类Class。
```java
Class clazz = Class.forName("com.powernode.reflect.SystemService");
```
当拿到Class之后，调用getDeclaredMethod()方法可以获取到方法。
假如你要获取这个方法：login(String username, String password)
```java
Method loginMethod = clazz.getDeclaredMethod("login", String.class, String.class);
```
假如你要获取到这个方法：login(String password)
```java
Method loginMethod = clazz.getDeclaredMethod("login", String.class);
```
获取一个方法，需要告诉Java程序，你要获取的方法的名字是什么，这个方法上每个形参的类型是什么。这样Java程序才能给你拿到对应的方法。
这样的设计也非常合理，因为在同一个类当中，方法是支持重载的，也就是说方法名可以一样，但参数列表一定是不一样的，所以获取一个方法需要提供方法名以及每个形参的类型。
假设有这样一个方法：
```java
public void setAge(int age){
    this.age = age;
}
```
你要获取这个方法的话，代码应该这样写：
```java
Method setAgeMethod = clazz.getDeclaredMethod("setAge", int.class);
```
其中setAge是方法名，int.class是形参的类型。
如果要获取上面的logout方法，代码应该这样写：
```java
Method logoutMethod = clazz.getDeclaredMethod("logout");
```
因为这个方法形式参数的个数是0个。所以只需要提供方法名就行了。你学会了吗？

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Dq23a&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 10.3 调用Method
要让一个方法调用的话，就关联到四要素了：

- 调用哪个对象的
- 哪个方法
- 传什么参数
- 返回什么值
```java
package com.powernode.reflect;

/**
 * @author 动力节点
 * @version 1.0
 * @className SystemService
 * @since 1.0
 **/
public class SystemService {

    public void logout(){
        System.out.println("退出系统");
    }

    public boolean login(String username, String password){
        if ("admin".equals(username) && "admin123".equals(password)) {
            return true;
        }
        return false;
    }

    public boolean login(String password){
        if("110".equals(password)){
            return true;
        }
        return false;
    }
}

```
假如我们要调用的方法是：login(String, String)
第一步：创建对象（四要素之首：调用哪个对象的）
```java
Class clazz = Class.forName("com.powernode.reflect.SystemService");
Object obj = clazz.newInstance();
```
第二步：获取方法login(String,String)（四要素之一：哪个方法）
```java
Method loginMethod = clazz.getDeclaredMethod("login", String.class, String.class);
```
第三步：调用方法
```java
Object retValue = loginMethod.invoke(obj, "admin", "admin123");
```
解说四要素：

- 哪个对象：obj
- 哪个方法：loginMethod
- 传什么参数："admin", "admin123"
- 返回什么值：retValue
```java
package com.powernode.reflect;

import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className ReflectTest02
 * @since 1.0
 **/
public class ReflectTest02 {
    public static void main(String[] args) throws Exception{
        Class clazz = Class.forName("com.powernode.reflect.SystemService");
        Object obj = clazz.newInstance();
        Method loginMethod = clazz.getDeclaredMethod("login", String.class, String.class);
        Object retValue = loginMethod.invoke(obj, "admin", "admin123");
        System.out.println(retValue);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665471501974-88a80910-1c8e-495b-956f-d6b7a82bf5b4.png#averageHue=%23faf9f9&clientId=ue12e8566-378b-4&from=paste&height=139&id=u943855ba&originHeight=139&originWidth=398&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7435&status=done&style=shadow&taskId=ued2fb744-f833-4edf-a934-69cf0d8b4b5&title=&width=398)
那如果调用既没有参数，又没有返回值的logout方法，应该怎么做？
```java
package com.powernode.reflect;

import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className ReflectTest03
 * @since 1.0
 **/
public class ReflectTest03 {
    public static void main(String[] args) throws Exception{
        Class clazz = Class.forName("com.powernode.reflect.SystemService");
        Object obj = clazz.newInstance();
        Method logoutMethod = clazz.getDeclaredMethod("logout");
        logoutMethod.invoke(obj);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665471647046-386be2b3-e848-4a3d-82ea-7faf1c802a04.png#averageHue=%23f9f9f8&clientId=ue12e8566-378b-4&from=paste&height=119&id=u8ecf54d8&originHeight=119&originWidth=308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7529&status=done&style=shadow&taskId=ua18a09c4-532b-4ded-8e58-f807696b6cb&title=&width=308)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=GJ9wJ&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 10.4 假设你知道属性名
假设有这样一个类：
```java
package com.powernode.reflect;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```
你知道以下这几条信息：

- 类名是：com.powernode.reflect.User
- 该类中有String类型的name属性和int类型的age属性。
- 另外你也知道该类的设计符合javabean规范。（也就是说属性私有化，对外提供setter和getter方法）

你如何通过反射机制给User对象的name属性赋值zhangsan，给age属性赋值20岁。
```java
package com.powernode.reflect;

import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserTest
 * @since 1.0
 **/
public class UserTest {
    public static void main(String[] args) throws Exception{
        // 已知类名
        String className = "com.powernode.reflect.User";
        // 已知属性名
        String propertyName = "age";

        // 通过反射机制给User对象的age属性赋值20岁
        Class<?> clazz = Class.forName(className);
        Object obj = clazz.newInstance(); // 创建对象

        // 根据属性名获取setter方法名
        String setMethodName = "set" + propertyName.toUpperCase().charAt(0) + propertyName.substring(1);

        // 获取Method
        Method setMethod = clazz.getDeclaredMethod(setMethodName, int.class);

        // 调用Method
        setMethod.invoke(obj, 20);

        System.out.println(obj);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665472287604-1994754e-51c1-4bd4-8a50-2fc0b0995ea6.png#averageHue=%23f8f7f6&clientId=ue12e8566-378b-4&from=paste&height=111&id=uc648e8cb&originHeight=111&originWidth=390&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9298&status=done&style=shadow&taskId=u6b15839b-6ea5-441f-89f4-9eede39631c&title=&width=390)
给User的name属性赋值zhangsan，这个大家可以尝试自己完成哦！！！

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=L9Vgx&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 十一、手写Spring框架
Spring IoC容器的实现原理：工厂模式 + 解析XML + 反射机制。
我们给自己的框架起名为：myspring（我的春天）
## 第一步：创建模块myspring
采用Maven方式新建Module：myspring
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665475207334-bd779f04-b490-4237-9ab1-306989458f22.png#averageHue=%23f3f1f1&clientId=ue12e8566-378b-4&from=paste&height=602&id=u9efa03bf&originHeight=602&originWidth=772&originalType=binary&ratio=1&rotation=0&showTitle=false&size=40674&status=done&style=shadow&taskId=u4db572af-dfb7-4fe3-8647-149614b6e24&title=&width=772)
打包方式采用jar，并且引入dom4j和jaxen的依赖，因为要使用它解析XML文件，还有junit依赖。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.myspringframework</groupId>
    <artifactId>myspring</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.dom4j</groupId>
            <artifactId>dom4j</artifactId>
            <version>2.1.3</version>
        </dependency>
        <dependency>
            <groupId>jaxen</groupId>
            <artifactId>jaxen</artifactId>
            <version>1.2.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=CRCXF&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第二步：准备好我们要管理的Bean
准备好我们要管理的Bean（**这些Bean在将来开发完框架之后是要删除的**）
注意包名，不要用org.myspringframework包，因为这些Bean不是框架内置的。是将来使用我们框架的程序员提供的。
```java
package com.powernode.myspring.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className Address
 * @since 1.0
 **/
public class Address {
    private String city;
    private String street;
    private String zipcode;

    public Address() {
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getStreet() {
        return street;
    }

    public void setStreet(String street) {
        this.street = street;
    }

    public String getZipcode() {
        return zipcode;
    }

    public void setZipcode(String zipcode) {
        this.zipcode = zipcode;
    }

    @Override
    public String toString() {
        return "Address{" +
                "city='" + city + '\'' +
                ", street='" + street + '\'' +
                ", zipcode='" + zipcode + '\'' +
                '}';
    }
}

```
```java
package com.powernode.myspring.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    private String name;
    private int age;
    private Address addr;

    public User() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Address getAddr() {
        return addr;
    }

    public void setAddr(Address addr) {
        this.addr = addr;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", addr=" + addr +
                '}';
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=tGjSx&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第三步：准备myspring.xml配置文件
将来在框架开发完毕之后，这个文件也是要删除的。因为这个配置文件的提供者应该是使用这个框架的程序员。
文件名随意，我们这里叫做：myspring.xml
文件放在类路径当中即可，我们这里把文件放到类的根路径下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans>

    <bean id="userBean" class="com.powernode.myspring.bean.User">
        <property name="name" value="张三"/>
        <property name="age" value="20"/>
        <property name="addr" ref="addrBean"/>
    </bean>
    
    <bean id="addrBean" class="com.powernode.myspring.bean.Address">
        <property name="city" value="北京"/>
        <property name="street" value="大兴区"/>
        <property name="zipcode" value="1000001"/>
    </bean>

</beans>
```
使用value给简单属性赋值。使用ref给非简单属性赋值。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=OWVKA&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第四步：编写ApplicationContext接口
ApplicationContext接口中提供一个getBean()方法，通过该方法可以获取Bean对象。
注意包名：这个接口就是myspring框架中的一员了。
```java
package org.myspringframework.core;

/**
 * @author 动力节点
 * @version 1.0
 * @className ApplicationContext
 * @since 1.0
 **/
public interface ApplicationContext {
    /**
     * 根据bean的id获取bean实例。
     * @param beanId bean的id
     * @return bean实例
     */
    Object getBean(String beanId);
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=qzdTl&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第五步：编写ClassPathXmlApplicationContext
ClassPathXmlApplicationContext是ApplicationContext接口的实现类。该类从类路径当中加载myspring.xml配置文件。
```java
package org.myspringframework.core;

/**
 * @author 动力节点
 * @version 1.0
 * @className ClassPathXmlApplicationContext
 * @since 1.0
 **/
public class ClassPathXmlApplicationContext implements ApplicationContext{
    @Override
    public Object getBean(String beanId) {
        return null;
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=nyB4L&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第六步：确定采用Map集合存储Bean
确定采用Map集合存储Bean实例。Map集合的key存储beanId，value存储Bean实例。Map<String,Object>
在ClassPathXmlApplicationContext类中添加Map<String,Object>属性。
并且在ClassPathXmlApplicationContext类中添加构造方法，该构造方法的参数接收myspring.xml文件。
同时实现getBean方法。
```java
package org.myspringframework.core;

import java.util.HashMap;
import java.util.Map;

/**
 * @author 动力节点
 * @version 1.0
 * @className ClassPathXmlApplicationContext
 * @since 1.0
 **/
public class ClassPathXmlApplicationContext implements ApplicationContext{
    /**
     * 存储bean的Map集合
     */
    private Map<String,Object> beanMap = new HashMap<>();

    /**
     * 在该构造方法中，解析myspring.xml文件，创建所有的Bean实例，并将Bean实例存放到Map集合中。
     * @param resource 配置文件路径（要求在类路径当中）
     */
    public ClassPathXmlApplicationContext(String resource) {

    }

    @Override
    public Object getBean(String beanId) {
        return beanMap.get(beanId);
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=WMYp9&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第七步：解析配置文件实例化所有Bean
在ClassPathXmlApplicationContext的构造方法中解析配置文件，获取所有bean的类名，通过反射机制调用无参数构造方法创建Bean。并且将Bean对象存放到Map集合中。
```java
/**
* 在该构造方法中，解析myspring.xml文件，创建所有的Bean实例，并将Bean实例存放到Map集合中。
* @param resource 配置文件路径（要求在类路径当中）
*/
public ClassPathXmlApplicationContext(String resource) {
    try {
        SAXReader reader = new SAXReader();
        Document document = reader.read(ClassLoader.getSystemClassLoader().getResourceAsStream(resource));
        // 获取所有的bean标签
        List<Node> beanNodes = document.selectNodes("//bean");
        // 遍历集合
        beanNodes.forEach(beanNode -> {
            Element beanElt = (Element) beanNode;
            // 获取id
            String id = beanElt.attributeValue("id");
            // 获取className
            String className = beanElt.attributeValue("class");
            try {
                // 通过反射机制创建对象
                Class<?> clazz = Class.forName(className);
                Constructor<?> defaultConstructor = clazz.getDeclaredConstructor();
                Object bean = defaultConstructor.newInstance();
                // 存储到Map集合
                beanMap.put(id, bean);
            } catch (Exception e) {
                e.printStackTrace();
            }
        });
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=iJUEZ&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第八步：测试能否获取到Bean
编写测试程序。
```java
package com.powernode.myspring.test;

import org.junit.Test;
import org.myspringframework.core.ApplicationContext;
import org.myspringframework.core.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className MySpringTest
 * @since 1.0
 **/
public class MySpringTest {
    @Test
    public void testMySpring(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("myspring.xml");
        Object userBean = applicationContext.getBean("userBean");
        Object addrBean = applicationContext.getBean("addrBean");
        System.out.println(userBean);
        System.out.println(addrBean);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665478707450-7e52f70c-97b2-4e6d-b96f-cc5bea8b51a4.png#averageHue=%23f8f7f6&clientId=ud56a2a07-f6fd-4&from=paste&height=150&id=u2c1d3f33&originHeight=150&originWidth=649&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17012&status=done&style=shadow&taskId=uf5600011-32e6-4ec6-8824-7431503ca2f&title=&width=649)
通过测试Bean已经实例化成功了，属性的值是null，这是我们能够想到的，毕竟我们调用的是无参数构造方法，所以属性都是默认值。
下一步就是我们应该如何给Bean的属性赋值呢？

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=uLQFa&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第九步：给Bean的属性赋值
通过反射机制调用set方法，给Bean的属性赋值。
继续在ClassPathXmlApplicationContext构造方法中编写代码。
```java
package org.myspringframework.core;

import org.dom4j.Document;
import org.dom4j.Element;
import org.dom4j.Node;
import org.dom4j.io.SAXReader;

import java.lang.reflect.Constructor;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * @author 动力节点
 * @version 1.0
 * @className ClassPathXmlApplicationContext
 * @since 1.0
 **/
public class ClassPathXmlApplicationContext implements ApplicationContext{
    /**
     * 存储bean的Map集合
     */
    private Map<String,Object> beanMap = new HashMap<>();

    /**
     * 在该构造方法中，解析myspring.xml文件，创建所有的Bean实例，并将Bean实例存放到Map集合中。
     * @param resource 配置文件路径（要求在类路径当中）
     */
    public ClassPathXmlApplicationContext(String resource) {
        try {
            SAXReader reader = new SAXReader();
            Document document = reader.read(ClassLoader.getSystemClassLoader().getResourceAsStream(resource));
            // 获取所有的bean标签
            List<Node> beanNodes = document.selectNodes("//bean");
            // 遍历集合（这里的遍历只实例化Bean，不给属性赋值。为什么要这样做？）
            beanNodes.forEach(beanNode -> {
                Element beanElt = (Element) beanNode;
                // 获取id
                String id = beanElt.attributeValue("id");
                // 获取className
                String className = beanElt.attributeValue("class");
                try {
                    // 通过反射机制创建对象
                    Class<?> clazz = Class.forName(className);
                    Constructor<?> defaultConstructor = clazz.getDeclaredConstructor();
                    Object bean = defaultConstructor.newInstance();
                    // 存储到Map集合
                    beanMap.put(id, bean);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
            // 再重新遍历集合，这次遍历是为了给Bean的所有属性赋值。
            // 思考：为什么不在上面的循环中给Bean的属性赋值，而在这里再重新遍历一次呢？
            // 通过这里你是否能够想到Spring是如何解决循环依赖的：实例化和属性赋值分开。
            beanNodes.forEach(beanNode -> {
                Element beanElt = (Element) beanNode;
                // 获取bean的id
                String beanId = beanElt.attributeValue("id");
                // 获取所有property标签
                List<Element> propertyElts = beanElt.elements("property");
                // 遍历所有属性
                propertyElts.forEach(propertyElt -> {
                    try {
                        // 获取属性名
                        String propertyName = propertyElt.attributeValue("name");
                        // 获取属性类型
                        Class<?> propertyType = beanMap.get(beanId).getClass().getDeclaredField(propertyName).getType();
                        // 获取set方法名
                        String setMethodName = "set" + propertyName.toUpperCase().charAt(0) + propertyName.substring(1);
                        // 获取set方法
                        Method setMethod = beanMap.get(beanId).getClass().getDeclaredMethod(setMethodName, propertyType);
                        // 获取属性的值，值可能是value，也可能是ref。
                        // 获取value
                        String propertyValue = propertyElt.attributeValue("value");
                        // 获取ref
                        String propertyRef = propertyElt.attributeValue("ref");
                        Object propertyVal = null;
                        if (propertyValue != null) {
                            // 该属性是简单属性
                            String propertyTypeSimpleName = propertyType.getSimpleName();
                            switch (propertyTypeSimpleName) {
                                case "byte": case "Byte":
                                    propertyVal = Byte.valueOf(propertyValue);
                                    break;
                                case "short": case "Short":
                                    propertyVal = Short.valueOf(propertyValue);
                                    break;
                                case "int": case "Integer":
                                    propertyVal = Integer.valueOf(propertyValue);
                                    break;
                                case "long": case "Long":
                                    propertyVal = Long.valueOf(propertyValue);
                                    break;
                                case "float": case "Float":
                                    propertyVal = Float.valueOf(propertyValue);
                                    break;
                                case "double": case "Double":
                                    propertyVal = Double.valueOf(propertyValue);
                                    break;
                                case "boolean": case "Boolean":
                                    propertyVal = Boolean.valueOf(propertyValue);
                                    break;
                                case "char": case "Character":
                                    propertyVal = propertyValue.charAt(0);
                                    break;
                                case "String":
                                    propertyVal = propertyValue;
                                    break;
                            }
                            setMethod.invoke(beanMap.get(beanId), propertyVal);
                        }
                        if (propertyRef != null) {
                            // 该属性不是简单属性
                            setMethod.invoke(beanMap.get(beanId), beanMap.get(propertyRef));
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                });
            });

        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public Object getBean(String beanId) {
        return beanMap.get(beanId);
    }
}

```
重点处理：当property标签中是value怎么办？是ref怎么办？
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665481050714-a41f73d9-67bb-40b9-9137-601a0775450d.png#averageHue=%23f9f7f6&clientId=ud56a2a07-f6fd-4&from=paste&height=147&id=udfde2846&originHeight=147&originWidth=969&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25735&status=done&style=shadow&taskId=u4a07ea37-3e47-4a10-a42a-19f2547abf6&title=&width=969)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=WsjEk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 第十步：打包发布
将多余的类以及配置文件删除，使用maven打包发布。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665481384984-b9b107a7-6566-473a-95df-fc7fcb613f18.png#averageHue=%23f9f8f7&clientId=ud56a2a07-f6fd-4&from=paste&height=222&id=uc26e4de5&originHeight=222&originWidth=395&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7445&status=done&style=shadow&taskId=ua39a7c0a-fa64-442a-bb7f-3e6dd9c2bf3&title=&width=395)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665481462831-bbd5bfd3-d647-4c04-990a-9c39a4116d21.png#averageHue=%23f9f9f8&clientId=ud56a2a07-f6fd-4&from=paste&height=173&id=u5e804933&originHeight=173&originWidth=533&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13326&status=done&style=shadow&taskId=u8a00a74f-aecc-4600-a3d8-8c359026ca3&title=&width=533)
## 第十一步：站在程序员角度使用myspring框架
新建模块：myspring-test
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665481605553-46ba6264-a360-4700-a696-1aa536c44cf1.png#averageHue=%23f3f2f2&clientId=ud56a2a07-f6fd-4&from=paste&height=604&id=u11f4822d&originHeight=604&originWidth=778&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39797&status=done&style=shadow&taskId=uf7fae99d-0f9f-412b-b003-d05456f6acc&title=&width=778)
引入myspring框架的依赖：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>myspring-test</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.myspringframework</groupId>
            <artifactId>myspring</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```
编写Bean
```java
package com.powernode.myspring.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserDao
 * @since 1.0
 **/
public class UserDao {
    public void insert(){
        System.out.println("UserDao正在插入数据");
    }
}

```
```java
package com.powernode.myspring.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserService
 * @since 1.0
 **/
public class UserService {
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        System.out.println("UserService开始执行save操作");
        userDao.insert();
        System.out.println("UserService执行save操作结束");
    }
}

```
编写myspring.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans>

    <bean id="userServiceBean" class="com.powernode.myspring.bean.UserService">
        <property name="userDao" ref="userDaoBean"/>
    </bean>

    <bean id="userDaoBean" class="com.powernode.myspring.bean.UserDao"/>

</beans>
```
编写测试程序
```java
package com.powernode.myspring.test;

import com.powernode.myspring.bean.UserService;
import org.junit.Test;
import org.myspringframework.core.ApplicationContext;
import org.myspringframework.core.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className MySpringTest
 * @since 1.0
 **/
public class MySpringTest {

    @Test
    public void testMySpring(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("myspring.xml");
        UserService userServiceBean = (UserService) applicationContext.getBean("userServiceBean");
        userServiceBean.save();
    }
}

```
执行结果
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665482096446-2015f7a8-3e86-4d74-a26b-a3d417c250fa.png#averageHue=%23f8f7f5&clientId=ud56a2a07-f6fd-4&from=paste&height=167&id=u1daaa772&originHeight=167&originWidth=425&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19370&status=done&style=shadow&taskId=u97aace1a-88b7-4a63-86f2-aa0d860ea1b&title=&width=425)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=KbjQI&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十二、Spring IoC注解式开发
## 12.1 回顾注解
注解的存在主要是为了简化XML的配置。**Spring6倡导全注解开发**。
我们来回顾一下：

- 第一：注解怎么定义，注解中的属性怎么定义？
- 第二：注解怎么使用？
- 第三：通过反射机制怎么读取注解？

**注解怎么定义，注解中的属性怎么定义？**
```java
package com.powernode.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Component {
    String value();
}
```
以上是自定义了一个注解：Component
该注解上面修饰的注解包括：Target注解和Retention注解，这两个注解被称为元注解。
Target注解用来设置Component注解可以出现的位置，以上代表表示Component注解只能用在类和接口上。
Retention注解用来设置Component注解的保持性策略，以上代表Component注解可以被反射机制读取。
String value(); 是Component注解中的一个属性。该属性类型String，属性名是value。
**注解怎么使用？**
```java
package com.powernode.bean;

import com.powernode.annotation.Component;

@Component(value = "userBean")
public class User {
}

```
用法简单，语法格式：@注解类型名(属性名=属性值, 属性名=属性值, 属性名=属性值......)
userBean为什么使用双引号括起来，因为value属性是String类型，字符串。
另外如果属性名是value，则在使用的时候可以省略属性名，例如：
```java
package com.powernode.bean;

import com.powernode.annotation.Component;

//@Component(value = "userBean")
@Component("userBean")
public class User {
}

```
**通过反射机制怎么读取注解？**
接下来，我们来写一段程序，当Bean类上有Component注解时，则实例化Bean对象，如果没有，则不实例化对象。
我们准备两个Bean，一个上面有注解，一个上面没有注解。
```java
package com.powernode.bean;

import com.powernode.annotation.Component;

@Component("userBean")
public class User {
}
```
```java
package com.powernode.bean;

public class Vip {
}
```
假设我们现在只知道包名：com.powernode.bean。至于这个包下有多少个Bean我们不知道。哪些Bean上有注解，哪些Bean上没有注解，这些我们都不知道，如何通过程序全自动化判断。
```java
package com.powernode.test;

import com.powernode.annotation.Component;

import java.io.File;
import java.net.URL;
import java.util.Arrays;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Map;

/**
 * @author 动力节点
 * @version 1.0
 * @className Test
 * @since 1.0
 **/
public class Test {
    public static void main(String[] args) throws Exception {
        // 存放Bean的Map集合。key存储beanId。value存储Bean。
        Map<String,Object> beanMap = new HashMap<>();

        String packageName = "com.powernode.bean";
        String path = packageName.replaceAll("\\.", "/");
        URL url = ClassLoader.getSystemClassLoader().getResource(path);
        File file = new File(url.getPath());
        File[] files = file.listFiles();
        Arrays.stream(files).forEach(f -> {
            String className = packageName + "." + f.getName().split("\\.")[0];
            try {
                Class<?> clazz = Class.forName(className);
                if (clazz.isAnnotationPresent(Component.class)) {
                    Component component = clazz.getAnnotation(Component.class);
                    String beanId = component.value();
                    Object bean = clazz.newInstance();
                    beanMap.put(beanId, bean);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        });

        System.out.println(beanMap);
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665543007882-24036142-350b-4209-bb20-46a61e35716d.png#averageHue=%23f9f8f7&clientId=ubfe41891-11ea-4&from=paste&height=108&id=u4615e829&originHeight=108&originWidth=600&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12954&status=done&style=shadow&taskId=u2000bb56-3c76-4281-a6df-4bbd6dbd0a2&title=&width=600)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=nHU12&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 12.2 声明Bean的注解
负责声明Bean的注解，常见的包括四个：

- @Component
- @Controller
- @Service
- @Repository

源码如下：
```java
package com.powernode.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(value = {ElementType.TYPE})
@Retention(value = RetentionPolicy.RUNTIME)
public @interface Component {
    String value();
}

```
```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}

```
```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}

```
```java
package org.springframework.stereotype;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.core.annotation.AliasFor;

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Repository {
    @AliasFor(
        annotation = Component.class
    )
    String value() default "";
}

```
通过源码可以看到，@Controller、@Service、@Repository这三个注解都是@Component注解的别名。
也就是说：这四个注解的功能都一样。用哪个都可以。
只是为了增强程序的可读性，建议：

- 控制器类上使用：Controller
- service类上使用：Service
- dao类上使用：Repository

他们都是只有一个value属性。value属性用来指定bean的id，也就是bean的名字。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665545099269-ebd7e446-bc2f-4442-89b8-3f513e546a8b.png#averageHue=%23fcfbf9&clientId=u999e1312-db7a-4&from=paste&height=362&id=ue3348a0b&originHeight=362&originWidth=723&originalType=binary&ratio=1&rotation=0&showTitle=false&size=37314&status=done&style=shadow&taskId=u1ccac9ac-0fa8-4792-bd2f-d433a8ffd3e&title=&width=723)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=NQJC7&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 12.3 Spring注解的使用
如何使用以上的注解呢？

- 第一步：加入aop的依赖
- 第二步：在配置文件中添加context命名空间
- 第三步：在配置文件中指定扫描的包
- 第四步：在Bean类上使用注解

**第一步：加入aop的依赖**
我们可以看到当加入spring-context依赖之后，会关联加入aop的依赖。所以这一步不用做。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665545268001-e3fb24f3-6688-4f52-a8c7-7c3084fa10a2.png#averageHue=%23faf8f5&clientId=u999e1312-db7a-4&from=paste&height=206&id=u99ab43ce&originHeight=206&originWidth=434&originalType=binary&ratio=1&rotation=0&showTitle=false&size=14246&status=done&style=shadow&taskId=u9a6c4b51-5411-4887-aa32-aadd701f90c&title=&width=434)
**第二步：在配置文件中添加context命名空间**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```
**第三步：在配置文件中指定要扫描的包**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6.bean"/>
</beans>
```
**第四步：在Bean类上使用注解**
```java
package com.powernode.spring6.bean;

import org.springframework.stereotype.Component;

@Component(value = "userBean")
public class User {
}

```
编写测试程序：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AnnotationTest {
    @Test
    public void testBean(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        User userBean = applicationContext.getBean("userBean", User.class);
        System.out.println(userBean);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665545669944-c067eacb-f65b-45ab-b68b-2320647cdfb4.png#averageHue=%23f3f2f0&clientId=u999e1312-db7a-4&from=paste&height=114&id=u818265a9&originHeight=114&originWidth=537&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12818&status=done&style=shadow&taskId=ub07ca18c-5947-43fc-958f-78164acfa8f&title=&width=537)
**如果注解的属性名是value，那么value是可以省略的。**
```java
package com.powernode.spring6.bean;

import org.springframework.stereotype.Component;

@Component("vipBean")
public class Vip {
}

```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.Vip;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AnnotationTest {
    @Test
    public void testBean(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        Vip vipBean = applicationContext.getBean("vipBean", Vip.class);
        System.out.println(vipBean);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665545860738-8bae2a45-efa8-40eb-9213-0dbd2ae1b54a.png#averageHue=%23f1efed&clientId=u999e1312-db7a-4&from=paste&height=107&id=u18d31f9a&originHeight=107&originWidth=496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12984&status=done&style=shadow&taskId=uaf671349-651d-42ec-b4c6-d49ae4538dd&title=&width=496)
**如果把value属性彻底去掉，spring会被Bean自动取名吗？会的。并且默认名字的规律是：Bean类名首字母小写即可。**
```java
package com.powernode.spring6.bean;

import org.springframework.stereotype.Component;

@Component
public class BankDao {
}
```
也就是说，这个BankDao的bean的名字为：bankDao
测试一下
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.BankDao;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AnnotationTest {
    @Test
    public void testBean(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        BankDao bankDao = applicationContext.getBean("bankDao", BankDao.class);
        System.out.println(bankDao);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665546100844-e0ffc213-8126-419a-ab67-7f433ad43105.png#averageHue=%23f2f0ee&clientId=u999e1312-db7a-4&from=paste&height=110&id=u1c31bf25&originHeight=110&originWidth=540&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13208&status=done&style=shadow&taskId=u9d91d502-d4e0-47a5-9253-31562560c5f&title=&width=540)
我们将Component注解换成其它三个注解，看看是否可以用：
```java
package com.powernode.spring6.bean;

import org.springframework.stereotype.Controller;

@Controller
public class BankDao {
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665546198246-f9d6adc1-ecc8-4e8c-babf-49f2ed7b87cd.png#averageHue=%23f2f0ee&clientId=u999e1312-db7a-4&from=paste&height=109&id=u34fc7e45&originHeight=109&originWidth=507&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13171&status=done&style=shadow&taskId=u211caeee-724d-410c-9440-f1136f928e4&title=&width=507)
剩下的两个注解大家可以测试一下。
**如果是多个包怎么办？有两种解决方案：**

- **第一种：在配置文件中指定多个包，用逗号隔开。**
- **第二种：指定多个包的共同父包。**

先来测试一下逗号（英文）的方式：
创建一个新的包：bean2，定义一个Bean类。
```java
package com.powernode.spring6.bean2;

import org.springframework.stereotype.Service;

@Service
public class Order {
}

```
配置文件修改：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6.bean,com.powernode.spring6.bean2"/>
</beans>
```
测试程序：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.BankDao;
import com.powernode.spring6.bean2.Order;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AnnotationTest {
    @Test
    public void testBean(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        BankDao bankDao = applicationContext.getBean("bankDao", BankDao.class);
        System.out.println(bankDao);
        Order order = applicationContext.getBean("order", Order.class);
        System.out.println(order);
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665546710304-8ebbe95d-1d1d-44fa-9605-9dad43e487b7.png#averageHue=%23f3f1ee&clientId=u999e1312-db7a-4&from=paste&height=136&id=uef2bb625&originHeight=136&originWidth=533&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19061&status=done&style=shadow&taskId=ua5845489-0baf-4d46-9312-549c3f66211&title=&width=533)
我们再来看看，指定共同的父包行不行：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6"/>
</beans>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665546777022-4eb8c5e3-22ed-4baf-8722-a5fa98df253d.png#averageHue=%23f3f1ee&clientId=u999e1312-db7a-4&from=paste&height=142&id=u1924e1f2&originHeight=142&originWidth=519&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19344&status=done&style=shadow&taskId=ua561508b-6f89-463d-b7c2-ace912fb097&title=&width=519)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Pd2IE&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 12.4 选择性实例化Bean
假设在某个包下有很多Bean，有的Bean上标注了Component，有的标注了Controller，有的标注了Service，有的标注了Repository，现在由于某种特殊业务的需要，只允许其中所有的Controller参与Bean管理，其他的都不实例化。这应该怎么办呢？
```java
package com.powernode.spring6.bean3;

import org.springframework.stereotype.Component;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;

@Component
public class A {
    public A() {
        System.out.println("A的无参数构造方法执行");
    }
}

@Controller
class B {
    public B() {
        System.out.println("B的无参数构造方法执行");
    }
}

@Service
class C {
    public C() {
        System.out.println("C的无参数构造方法执行");
    }
}

@Repository
class D {
    public D() {
        System.out.println("D的无参数构造方法执行");
    }
}

@Controller
class E {
    public E() {
        System.out.println("E的无参数构造方法执行");
    }
}

@Controller
class F {
    public F() {
        System.out.println("F的无参数构造方法执行");
    }
}

```
我只想实例化bean3包下的Controller。配置文件这样写：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.powernode.spring6.bean3" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    
</beans>
```
use-default-filters="true" 表示：使用spring默认的规则，只要有Component、Controller、Service、Repository中的任意一个注解标注，则进行实例化。
**use-default-filters="false"** 表示：不再spring默认实例化规则，即使有Component、Controller、Service、Repository这些注解标注，也不再实例化。
<context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/> 表示只有Controller进行实例化。
```java
@Test
public void testChoose(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-choose.xml");
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665556059297-de0d7dbc-aa37-46a3-9b1d-1d4c246b0ffc.png#averageHue=%23f7f6f4&clientId=u999e1312-db7a-4&from=paste&height=174&id=u46e6828e&originHeight=174&originWidth=520&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18046&status=done&style=shadow&taskId=uef3c6593-409c-4fbe-b4c5-9105a4c73d6&title=&width=520)
也可以将use-default-filters设置为true（不写就是true），并且采用exclude-filter方式排出哪些注解标注的Bean不参与实例化：
```xml
<context:component-scan base-package="com.powernode.spring6.bean3">
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Service"/>
  <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665556372417-14f2208c-4151-4bcd-9f22-80db5e3ed837.png#averageHue=%23f4f3f1&clientId=u999e1312-db7a-4&from=paste&height=111&id=u1ebe1eec&originHeight=111&originWidth=492&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11122&status=done&style=shadow&taskId=ud5a1de4a-95e7-459f-b1b0-a45af1307ff&title=&width=492)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=BdKAR&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 12.5 负责注入的注解
@Component @Controller @Service @Repository 这四个注解是用来声明Bean的，声明后这些Bean将被实例化。接下来我们看一下，如何给Bean的属性赋值。给Bean属性赋值需要用到这些注解：

- @Value
- @Autowired
- @Qualifier
- @Resource
### 12.5.1 @Value
当属性的类型是简单类型时，可以使用@Value注解进行注入。
```java
package com.powernode.spring6.bean4;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {
    @Value(value = "zhangsan")
    private String name;
    @Value("20")
    private int age;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```
开启包扫描：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6.bean4"/>
</beans>
```
```java
@Test
public void testValue(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-injection.xml");
    Object user = applicationContext.getBean("user");
    System.out.println(user);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665557109935-e0300b67-fd35-4d66-99d1-dac41cb0f13d.png#averageHue=%23f3f2f0&clientId=u999e1312-db7a-4&from=paste&height=113&id=u5fa90bbd&originHeight=113&originWidth=508&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11012&status=done&style=shadow&taskId=uec092d2d-3ae1-4366-a952-c62149cdb1f&title=&width=508)
通过以上代码可以发现，我们并没有给属性提供setter方法，但仍然可以完成属性赋值。
如果提供setter方法，并且在setter方法上添加@Value注解，可以完成注入吗？尝试一下：
```java
package com.powernode.spring6.bean4;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {
    
    private String name;

    private int age;

    @Value("李四")
    public void setName(String name) {
        this.name = name;
    }

    @Value("30")
    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665557275282-82ba995b-6395-4d32-b322-d976ac3299d1.png#averageHue=%23f3f1ef&clientId=u999e1312-db7a-4&from=paste&height=106&id=udca1ce29&originHeight=106&originWidth=485&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11245&status=done&style=shadow&taskId=uebe9436d-a844-440b-8e6a-e14e1f6a19d&title=&width=485)
通过测试可以得知，@Value注解可以直接使用在属性上，也可以使用在setter方法上。都是可以的。都可以完成属性的赋值。
为了简化代码，以后我们一般不提供setter方法，直接在属性上使用@Value注解完成属性赋值。
出于好奇，我们再来测试一下，是否能够通过构造方法完成注入：
```java
package com.powernode.spring6.bean4;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class User {

    private String name;

    private int age;

    public User(@Value("隔壁老王") String name, @Value("33") int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665557643220-1010bea9-5578-4388-8868-4beb11dfbe95.png#averageHue=%23f2f0ee&clientId=u999e1312-db7a-4&from=paste&height=110&id=u38cd30d6&originHeight=110&originWidth=444&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11523&status=done&style=shadow&taskId=ucc4904b2-3db5-488a-896d-1b762a1d7a4&title=&width=444)
通过测试得知：@Value注解可以出现在属性上、setter方法上、以及构造方法的形参上。可见Spring给我们提供了多样化的注入。太灵活了。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=ZYpUs&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 12.5.2 @Autowired与@Qualifier
@Autowired注解可以用来注入**非简单类型**。被翻译为：自动连线的，或者自动装配。
单独使用@Autowired注解，**默认根据类型装配**。【默认是byType】
看一下它的源码：
```java
package org.springframework.beans.factory.annotation;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {
	boolean required() default true;
}
```
源码中有两处需要注意：

- 第一处：该注解可以标注在哪里？
   - 构造方法上
   - 方法上
   - 形参上
   - 属性上
   - 注解上
- 第二处：该注解有一个required属性，默认值是true，表示在注入的时候要求被注入的Bean必须是存在的，如果不存在则报错。如果required属性设置为false，表示注入的Bean存在或者不存在都没关系，存在的话就注入，不存在的话，也不报错。

**我们先在属性上使用@Autowired注解：**
```java
package com.powernode.spring6.dao;

public interface UserDao {
    void insert();
}

```
```java
package com.powernode.spring6.dao;

import org.springframework.stereotype.Repository;

@Repository //纳入bean管理
public class UserDaoForMySQL implements UserDao{
    @Override
    public void insert() {
        System.out.println("正在向mysql数据库插入User数据");
    }
}

```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service // 纳入bean管理
public class UserService {

    @Autowired // 在属性上注入
    private UserDao userDao;
    
    // 没有提供构造方法和setter方法。

    public void save(){
        userDao.insert();
    }
}
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6.dao,com.powernode.spring6.service"/>
</beans>
```
```java
@Test
public void testAutowired(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-injection.xml");
    UserService userService = applicationContext.getBean("userService", UserService.class);
    userService.save();
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665561365140-b0200308-0c25-4a29-96be-5a93594e2d2b.png#averageHue=%23f2f0ee&clientId=u999e1312-db7a-4&from=paste&height=97&id=u70f1ad5a&originHeight=97&originWidth=487&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11448&status=done&style=shadow&taskId=u76fc018c-6c0d-4446-85a2-5ce21b2b28b&title=&width=487)
以上构造方法和setter方法都没有提供，经过测试，仍然可以注入成功。
**接下来，再来测试一下@Autowired注解出现在setter方法上：**
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    @Autowired
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665562770986-e19377a6-af3e-4082-9463-16c795742ad5.png#averageHue=%23f1efed&clientId=u999e1312-db7a-4&from=paste&height=93&id=ue84a5069&originHeight=93&originWidth=491&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11554&status=done&style=shadow&taskId=uc6cfe36b-be24-48ff-a987-885055ae1ef&title=&width=491)
**我们再来看看能不能出现在构造方法上：**
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    @Autowired
    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665562985700-7820d3d8-cf43-43af-8c81-46f301ea2835.png#averageHue=%23f3f1ef&clientId=u999e1312-db7a-4&from=paste&height=111&id=u234df523&originHeight=111&originWidth=470&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11539&status=done&style=shadow&taskId=ufc7d932f-8541-4b06-83db-237a969a04a&title=&width=470)
**再来看看，这个注解能不能只标注在构造方法的形参上：**
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    public UserService(@Autowired UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665563225083-172d5675-cfcb-4f63-9b83-ce85b29b953e.png#averageHue=%23f4f2f0&clientId=u999e1312-db7a-4&from=paste&height=109&id=ub32f4d51&originHeight=109&originWidth=487&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11684&status=done&style=shadow&taskId=ue050ee11-60ad-40d2-a87d-7136331fa26&title=&width=487)
**还有更劲爆的，当有参数的构造方法只有一个时，@Autowired注解可以省略。**
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665563320900-df9e4cb3-c046-4f5c-b482-42951f18fb16.png#averageHue=%23f4f2f0&clientId=u999e1312-db7a-4&from=paste&height=110&id=ua7390918&originHeight=110&originWidth=496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11721&status=done&style=shadow&taskId=u28692c80-0962-4b19-af49-762cb4791d5&title=&width=496)
**当然，如果有多个构造方法，@Autowired肯定是不能省略的。**
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
    
    public UserService(){
        
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665563410134-267b2484-54a3-4204-8e02-a9499ecbe614.png#averageHue=%23faf8f7&clientId=u999e1312-db7a-4&from=paste&height=218&id=u3fa8319b&originHeight=218&originWidth=1391&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36652&status=done&style=shadow&taskId=u8cd9a27d-178e-40b6-9a9e-6a363d63752&title=&width=1391)
到此为止，我们已经清楚@Autowired注解可以出现在哪些位置了。
@Autowired注解默认是byType进行注入的，也就是说根据类型注入的，如果以上程序中，UserDao接口还有另外一个实现类，会出现问题吗？
```java
package com.powernode.spring6.dao;

import org.springframework.stereotype.Repository;

@Repository //纳入bean管理
public class UserDaoForOracle implements UserDao{
    @Override
    public void insert() {
        System.out.println("正在向Oracle数据库插入User数据");
    }
}

```
当你写完这个新的实现类之后，此时IDEA工具已经提示错误信息了：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665563729880-0421bc02-19ca-4353-8a10-5b0ef9972b90.png#averageHue=%23faf9f8&clientId=u999e1312-db7a-4&from=paste&height=674&id=ub89c21a1&originHeight=674&originWidth=1030&originalType=binary&ratio=1&rotation=0&showTitle=false&size=84797&status=done&style=shadow&taskId=u7f117040-8072-4719-91ad-2c7720183ee&title=&width=1030)
错误信息中说：不能装配，UserDao这个Bean的数量大于1.
怎么解决这个问题呢？**当然要byName，根据名称进行装配了。**
@Autowired注解和@Qualifier注解联合起来才可以根据名称进行装配，在@Qualifier注解中指定Bean名称。
```java
package com.powernode.spring6.dao;

import org.springframework.stereotype.Repository;

@Repository // 这里没有给bean起名，默认名字是：userDaoForOracle
public class UserDaoForOracle implements UserDao{
    @Override
    public void insert() {
        System.out.println("正在向Oracle数据库插入User数据");
    }
}

```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    @Autowired
    @Qualifier("userDaoForOracle") // 这个是bean的名字。
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665564055076-ffda3ad0-f957-4216-bf6c-957d62724d5f.png#averageHue=%23f2f0ed&clientId=u999e1312-db7a-4&from=paste&height=97&id=u149f27ad&originHeight=97&originWidth=484&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11655&status=done&style=shadow&taskId=u1d416395-88a7-4458-be3a-e0c8cba234e&title=&width=484)
总结：

- @Autowired注解可以出现在：属性上、构造方法上、构造方法的参数上、setter方法上。
- 当带参数的构造方法只有一个，@Autowired注解可以省略。
- @Autowired注解默认根据类型注入。如果要根据名称注入的话，需要配合@Qualifier注解一起使用。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=tjDZU&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 12.5.3 @Resource
@Resource注解也可以完成非简单类型注入。那它和@Autowired注解有什么区别？

- @Resource注解是JDK扩展包中的，也就是说属于JDK的一部分。所以该注解是标准注解，更加具有通用性。(JSR-250标准中制定的注解类型。JSR是Java规范提案。)
- @Autowired注解是Spring框架自己的。
- **@Resource注解默认根据名称装配byName，未指定name时，使用属性名作为name。通过name找不到的话会自动启动通过类型byType装配。**
- **@Autowired注解默认根据类型装配byType，如果想根据名称装配，需要配合@Qualifier注解一起用。**
- @Resource注解用在属性上、setter方法上。
- @Autowired注解用在属性上、setter方法上、构造方法上、构造方法参数上。

@Resource注解属于JDK扩展包，所以不在JDK当中，需要额外引入以下依赖：【**如果是JDK8的话不需要额外引入依赖。高于JDK11或低于JDK8需要引入以下依赖。**】
```xml
<dependency>
  <groupId>jakarta.annotation</groupId>
  <artifactId>jakarta.annotation-api</artifactId>
  <version>2.1.1</version>
</dependency>
```
一定要注意：**如果你用Spring6，要知道Spring6不再支持JavaEE，它支持的是JakartaEE9。（Oracle把JavaEE贡献给Apache了，Apache把JavaEE的名字改成JakartaEE了，大家之前所接触的所有的  javax.*  包名统一修改为  jakarta.*包名了。）**
```xml
<dependency>
  <groupId>javax.annotation</groupId>
  <artifactId>javax.annotation-api</artifactId>
  <version>1.3.2</version>
</dependency>
```
@Resource注解的源码如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665565515435-2ad5614a-8572-4c6f-80c1-efa236dbe35f.png#averageHue=%23fcf8f6&clientId=u999e1312-db7a-4&from=paste&height=426&id=u6038d063&originHeight=426&originWidth=1066&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71322&status=done&style=shadow&taskId=u62cc11b0-bd1d-4e76-8123-cbf4e5556cb&title=&width=1066)
测试一下：
```java
package com.powernode.spring6.dao;

import org.springframework.stereotype.Repository;

@Repository("xyz")
public class UserDaoForOracle implements UserDao{
    @Override
    public void insert() {
        System.out.println("正在向Oracle数据库插入User数据");
    }
}
```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Resource(name = "xyz")
    private UserDao userDao;

    public void save(){
        userDao.insert();
    }
}
```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665622877352-0ae69e3c-e7f3-452d-a405-392901612465.png#averageHue=%23f3f1ef&clientId=u27fdab07-dcc6-4&from=paste&height=112&id=u10e932c1&originHeight=112&originWidth=467&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11805&status=done&style=shadow&taskId=u6c533807-043a-4995-862f-5f10be7c2e4&title=&width=467)
**我们把UserDaoForOracle的名字xyz修改为userDao，让这个Bean的名字和UserService类中的UserDao属性名一致：**
```java
package com.powernode.spring6.dao;

import org.springframework.stereotype.Repository;

@Repository("userDao")
public class UserDaoForOracle implements UserDao{
    @Override
    public void insert() {
        System.out.println("正在向Oracle数据库插入User数据");
    }
}

```
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Resource
    private UserDao userDao;

    public void save(){
        userDao.insert();
    }
}

```
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665623044796-c4051a04-c56b-4ce9-b627-333ab7ca7b6a.png#averageHue=%23f3f1ef&clientId=u27fdab07-dcc6-4&from=paste&height=103&id=u0824fe26&originHeight=103&originWidth=479&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11673&status=done&style=shadow&taskId=u030d8a0e-b7df-40f1-bf91-5f93056777a&title=&width=479)
通过测试得知，当@Resource注解使用时没有指定name的时候，还是根据name进行查找，这个name是属性名。
接下来把UserService类中的属性名修改一下：
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    @Resource
    private UserDao userDao2;

    public void save(){
        userDao2.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665623273523-aff8ef45-b484-4462-bacc-fba7e14c8fee.png#averageHue=%23fbf8f7&clientId=u27fdab07-dcc6-4&from=paste&height=248&id=u0c222c16&originHeight=248&originWidth=1585&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23269&status=done&style=shadow&taskId=u40b6ff92-b1cc-4056-a77c-9b171aaab93&title=&width=1585)
根据异常信息得知：显然当通过name找不到的时候，自然会启动byType进行注入。以上的错误是因为UserDao接口下有两个实现类导致的。所以根据类型注入就会报错。
我们再来看@Resource注解使用在setter方法上可以吗？
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    @Resource
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
注意这个setter方法的方法名，setUserDao去掉set之后，将首字母变小写userDao，userDao就是name
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665623530366-79b8e09d-2559-4657-83eb-0b722261045f.png#averageHue=%23f3f1ef&clientId=u27fdab07-dcc6-4&from=paste&height=105&id=uce4c028f&originHeight=105&originWidth=482&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11743&status=done&style=shadow&taskId=u9b8d145f-59f7-4816-9e5f-1cdd64c8d24&title=&width=482)
当然，也可以指定name：
```java
package com.powernode.spring6.service;

import com.powernode.spring6.dao.UserDao;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private UserDao userDao;

    @Resource(name = "userDaoForMySQL")
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

    public void save(){
        userDao.insert();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665623611980-a66591e7-bd29-4327-a43c-6c6492c8612f.png#averageHue=%23f2f0ee&clientId=u27fdab07-dcc6-4&from=paste&height=96&id=ue56a8b89&originHeight=96&originWidth=489&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11431&status=done&style=shadow&taskId=u535fea63-6aa6-4a33-bc17-453192a7468&title=&width=489)
一句话总结@Resource注解：默认byName注入，没有指定name时把属性名当做name，根据name找不到时，才会byType注入。byType注入时，某种类型的Bean只能有一个。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=lKhm2&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 12.6 全注解式开发
所谓的全注解开发就是不再使用spring配置文件了。写一个配置类来代替配置文件。
```java
package com.powernode.spring6.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScans;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan({"com.powernode.spring6.dao", "com.powernode.spring6.service"})
public class Spring6Configuration {
}

```
编写测试程序：不再new ClassPathXmlApplicationContext()对象了。
```java
@Test
public void testNoXml(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Spring6Configuration.class);
    UserService userService = applicationContext.getBean("userService", UserService.class);
    userService.save();
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665624710824-61ee0ae9-ae96-49bf-b189-4a1f358e084a.png#averageHue=%23f4f3f1&clientId=u27fdab07-dcc6-4&from=paste&height=109&id=u4341dc61&originHeight=109&originWidth=504&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11527&status=done&style=shadow&taskId=ue07c15ec-5d6b-4591-9dcc-d9e6ab347a2&title=&width=504)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=oDQG8&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十三、JdbcTemplate
JdbcTemplate是Spring提供的一个JDBC模板类，是对JDBC的封装，简化JDBC代码。
当然，你也可以不用，可以让Spring集成其它的ORM框架，例如：MyBatis、Hibernate等。
接下来我们简单来学习一下，使用JdbcTemplate完成增删改查。
## 13.1 环境准备
数据库表：t_user
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665633536319-466a1b96-90ff-4a87-82ad-fb14f32a8d12.png#averageHue=%23f5f4f3&clientId=u27fdab07-dcc6-4&from=paste&height=188&id=u76c20684&originHeight=188&originWidth=800&originalType=binary&ratio=1&rotation=0&showTitle=false&size=19315&status=done&style=shadow&taskId=u8108b33a-018c-4933-a192-577b7934751&title=&width=800)
IDEA中新建模块：spring6-007-jdbc
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665633731889-aa224eae-8ff7-47af-aaf5-70500c7cf37e.png#averageHue=%23f3f2f2&clientId=u27fdab07-dcc6-4&from=paste&height=601&id=u3f985b79&originHeight=601&originWidth=777&originalType=binary&ratio=1&rotation=0&showTitle=false&size=40078&status=done&style=shadow&taskId=u1df9c873-ae00-458c-8806-a820a25e420&title=&width=777)
引入相关依赖：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>spring6-007-jdbc</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <repositories>
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
        <!--新增的依赖:mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
        <!--新增的依赖：spring jdbc，这个依赖中有JdbcTemplate-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```
准备实体类：表t_user对应的实体类User。
```java
package com.powernode.spring6.bean;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
public class User {
    private Integer id;
    private String realName;
    private Integer age;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", realName='" + realName + '\'' +
                ", age=" + age +
                '}';
    }

    public User() {
    }

    public User(Integer id, String realName, Integer age) {
        this.id = id;
        this.realName = realName;
        this.age = age;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getRealName() {
        return realName;
    }

    public void setRealName(String realName) {
        this.realName = realName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```
编写Spring配置文件：
JdbcTemplate是Spring提供好的类，这类的完整类名是：org.springframework.jdbc.core.JdbcTemplate
我们怎么使用这个类呢？new对象就可以了。怎么new对象，Spring最在行了。直接将这个类配置到Spring配置文件中，纳入Bean管理即可。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate"></bean>
</beans>
```
我们来看一下这个JdbcTemplate源码：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665641540149-8f44a8b1-35b6-4c8a-bd27-f08ebd911e01.png#averageHue=%23fdfcfa&clientId=u27fdab07-dcc6-4&from=paste&height=610&id=u5bdbb672&originHeight=610&originWidth=993&originalType=binary&ratio=1&rotation=0&showTitle=false&size=91978&status=done&style=shadow&taskId=u1a85f911-01f1-4843-9622-a53b517b2a3&title=&width=993)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665641567361-50fd782b-cea4-4ca2-9818-01696aca0eb0.png#averageHue=%23fdfbf9&clientId=u27fdab07-dcc6-4&from=paste&height=404&id=uaf0bcb51&originHeight=404&originWidth=1159&originalType=binary&ratio=1&rotation=0&showTitle=false&size=64579&status=done&style=shadow&taskId=uf4435305-b2c6-4467-b7c8-0f1b42e6579&title=&width=1159)
可以看到JdbcTemplate中有一个DataSource属性，这个属性是数据源，我们都知道连接数据库需要Connection对象，而生成Connection对象是数据源负责的。所以我们需要给JdbcTemplate设置数据源属性。
所有的数据源都是要实现javax.sql.DataSource接口的。这个数据源可以自己写一个，也可以用写好的，比如：阿里巴巴的德鲁伊连接池，c3p0，dbcp等。我们这里自己先手写一个数据源。
```java
package com.powernode.spring6.jdbc;

import javax.sql.DataSource;
import java.io.PrintWriter;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.SQLFeatureNotSupportedException;
import java.util.logging.Logger;

/**
 * @author 动力节点
 * @version 1.0
 * @className MyDataSource
 * @since 1.0
 **/
public class MyDataSource implements DataSource {
    // 添加4个属性
    private String driver;
    private String url;
    private String username;
    private String password;

    // 提供4个setter方法
    public void setDriver(String driver) {
        this.driver = driver;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    // 重点写怎么获取Connection对象就行。其他方法不用管。
    @Override
    public Connection getConnection() throws SQLException {
        try {
            Class.forName(driver);
            Connection conn = DriverManager.getConnection(url, username, password);
            return conn;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public Connection getConnection(String username, String password) throws SQLException {
        return null;
    }

    @Override
    public PrintWriter getLogWriter() throws SQLException {
        return null;
    }

    @Override
    public void setLogWriter(PrintWriter out) throws SQLException {

    }

    @Override
    public void setLoginTimeout(int seconds) throws SQLException {

    }

    @Override
    public int getLoginTimeout() throws SQLException {
        return 0;
    }

    @Override
    public Logger getParentLogger() throws SQLFeatureNotSupportedException {
        return null;
    }

    @Override
    public <T> T unwrap(Class<T> iface) throws SQLException {
        return null;
    }

    @Override
    public boolean isWrapperFor(Class<?> iface) throws SQLException {
        return false;
    }
}

```
写完数据源，我们需要把这个数据源传递给JdbcTemplate。因为JdbcTemplate中有一个DataSource属性：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myDataSource" class="com.powernode.spring6.jdbc.MyDataSource">
        <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="myDataSource"/>
    </bean>
</beans>
```
到这里环境就准备好了。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=hAJlH&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.2 新增
编写测试程序：
```java
package com.powernode.spring6.test;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.JdbcTemplate;

/**
 * @author 动力节点
 * @version 1.0
 * @className JdbcTest
 * @since 1.0
 **/
public class JdbcTest {
    @Test
    public void testInsert(){
        // 获取JdbcTemplate对象
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
        // 执行插入操作
        // 注意：insert delete update的sql语句，都是执行update方法。
        String sql = "insert into t_user(id,real_name,age) values(?,?,?)";
        int count = jdbcTemplate.update(sql, null, "张三", 30);
        System.out.println("插入的记录条数：" + count);
    }
}

```
update方法有两个参数：

- 第一个参数：要执行的SQL语句。（SQL语句中可能会有占位符 ? ）
- 第二个参数：可变长参数，参数的个数可以是0个，也可以是多个。一般是SQL语句中有几个问号，则对应几个参数。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=YsNCl&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.3 修改
```java
@Test
public void testUpdate(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 执行更新操作
    String sql = "update t_user set real_name = ?, age = ? where id = ?";
    int count = jdbcTemplate.update(sql, "张三丰", 55, 1);
    System.out.println("更新的记录条数：" + count);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665642952562-a030937f-b3f5-4018-b92e-02d25ab390d7.png#averageHue=%23f4f2f1&clientId=u27fdab07-dcc6-4&from=paste&height=107&id=ufb078681&originHeight=107&originWidth=483&originalType=binary&ratio=1&rotation=0&showTitle=false&size=10078&status=done&style=shadow&taskId=u6474de2e-0d18-4cff-ab28-57eddf050ac&title=&width=483)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=bmNVf&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.4 删除
```java
@Test
public void testDelete(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 执行delete
    String sql = "delete from t_user where id = ?";
    int count = jdbcTemplate.update(sql, 1);
    System.out.println("删除了几条记录：" + count);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665643115536-fb7949b8-9bcd-4d45-8032-40c3658911fd.png#averageHue=%23f2f1ef&clientId=u27fdab07-dcc6-4&from=paste&height=102&id=u9cd70054&originHeight=102&originWidth=474&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9558&status=done&style=shadow&taskId=uc6e5d8c4-1963-44ba-8754-715418696e7&title=&width=474)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=FGML1&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.5 查询一个对象
```java
@Test
public void testSelectOne(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 执行select
    String sql = "select id, real_name, age from t_user where id = ?";
    User user = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(User.class), 2);
    System.out.println(user);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665643791948-c5a4f422-4c49-426a-88f3-004907f696dd.png#averageHue=%23f3f2f0&clientId=u27fdab07-dcc6-4&from=paste&height=124&id=ufeed9ab1&originHeight=124&originWidth=480&originalType=binary&ratio=1&rotation=0&showTitle=false&size=12264&status=done&style=shadow&taskId=u605f6777-09a8-4a9b-9bbb-45724b3acc4&title=&width=480)
queryForObject方法三个参数：

- 第一个参数：sql语句
- 第二个参数：Bean属性值和数据库记录行的映射对象。在构造方法中指定映射的对象类型。
- 第三个参数：可变长参数，给sql语句的占位符问号传值。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=vdptN&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.6 查询多个对象
```java
@Test
public void testSelectAll(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 执行select
    String sql = "select id, real_name, age from t_user";
    List<User> users = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class));
    System.out.println(users);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665644613140-cc029b6f-7dbe-40fa-896c-ba8674da014c.png#averageHue=%23f5f3f2&clientId=u27fdab07-dcc6-4&from=paste&height=109&id=ua5ef8f52&originHeight=109&originWidth=846&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13755&status=done&style=shadow&taskId=ud2cd8d9c-41ef-4f1e-bf97-1f45c8b14d1&title=&width=846)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=D3YvB&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.7 查询一个值
```java
@Test
public void testSelectOneValue(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 执行select
    String sql = "select count(1) from t_user";
    Integer count = jdbcTemplate.queryForObject(sql, int.class); // 这里用Integer.class也可以
    System.out.println("总记录条数：" + count);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665644735494-3aff4263-8172-4e33-b5e4-564dda88a704.png#averageHue=%23f5f4f2&clientId=u27fdab07-dcc6-4&from=paste&height=110&id=u7a66fa0a&originHeight=110&originWidth=493&originalType=binary&ratio=1&rotation=0&showTitle=false&size=9704&status=done&style=shadow&taskId=u2b2e531b-1b4d-4fbc-a55e-478fb85d6cf&title=&width=493)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=k9Ycq&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.8 批量添加
```java
@Test
public void testAddBatch(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 批量添加
    String sql = "insert into t_user(id,real_name,age) values(?,?,?)";

    Object[] objs1 = {null, "小花", 20};
    Object[] objs2 = {null, "小明", 21};
    Object[] objs3 = {null, "小刚", 22};
    List<Object[]> list = new ArrayList<>();
    list.add(objs1);
    list.add(objs2);
    list.add(objs3);

    int[] count = jdbcTemplate.batchUpdate(sql, list);
    System.out.println(Arrays.toString(count));
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665645369060-9a473341-e4d2-4c26-bdd4-7d83fd93cc82.png#averageHue=%23f5f4f3&clientId=u27fdab07-dcc6-4&from=paste&height=118&id=u4d53536e&originHeight=118&originWidth=476&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7957&status=done&style=shadow&taskId=u79ea3a82-39ec-461b-bcf0-a8d37e01b39&title=&width=476)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=ljJG5&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.9 批量修改
```java
@Test
public void testUpdateBatch(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 批量修改
    String sql = "update t_user set real_name = ?, age = ? where id = ?";
    Object[] objs1 = {"小花11", 10, 2};
    Object[] objs2 = {"小明22", 12, 3};
    Object[] objs3 = {"小刚33", 9, 4};
    List<Object[]> list = new ArrayList<>();
    list.add(objs1);
    list.add(objs2);
    list.add(objs3);

    int[] count = jdbcTemplate.batchUpdate(sql, list);
    System.out.println(Arrays.toString(count));
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665645613528-7edf2796-3f40-4c6d-bedc-9022fb16d7da.png#averageHue=%23f5f4f3&clientId=u27fdab07-dcc6-4&from=paste&height=108&id=u1cdb8d08&originHeight=108&originWidth=496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7866&status=done&style=shadow&taskId=ubd026f6d-b7c8-44dd-b28b-0fecae788ab&title=&width=496)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Jelyx&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.10 批量删除
```java
@Test
public void testDeleteBatch(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    // 批量删除
    String sql = "delete from t_user where id = ?";
    Object[] objs1 = {2};
    Object[] objs2 = {3};
    Object[] objs3 = {4};
    List<Object[]> list = new ArrayList<>();
    list.add(objs1);
    list.add(objs2);
    list.add(objs3);
    int[] count = jdbcTemplate.batchUpdate(sql, list);
    System.out.println(Arrays.toString(count));
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665645815657-de657db3-cb20-4758-a40e-2c049175d89e.png#averageHue=%23f6f5f4&clientId=u27fdab07-dcc6-4&from=paste&height=122&id=u3e9e0665&originHeight=122&originWidth=490&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7981&status=done&style=shadow&taskId=u3292aab1-ae6a-4cae-b03d-813f9520427&title=&width=490)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=rQfsJ&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.11 使用回调函数
使用回调函数，可以参与的更加细节：
```java
@Test
public void testCallback(){
    // 获取JdbcTemplate对象
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    JdbcTemplate jdbcTemplate = applicationContext.getBean("jdbcTemplate", JdbcTemplate.class);
    String sql = "select id, real_name, age from t_user where id = ?";

    User user = jdbcTemplate.execute(sql, new PreparedStatementCallback<User>() {
        @Override
        public User doInPreparedStatement(PreparedStatement ps) throws SQLException, DataAccessException {
            User user = null;
            ps.setInt(1, 5);
            ResultSet rs = ps.executeQuery();
            if (rs.next()) {
                user = new User();
                user.setId(rs.getInt("id"));
                user.setRealName(rs.getString("real_name"));
                user.setAge(rs.getInt("age"));
            }
            return user;
        }
    });
    System.out.println(user);
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665646365875-6ff081a4-74a0-469d-a3ed-b579235743ee.png#averageHue=%23f3f2f0&clientId=u27fdab07-dcc6-4&from=paste&height=115&id=u0f03c359&originHeight=115&originWidth=488&originalType=binary&ratio=1&rotation=0&showTitle=false&size=11901&status=done&style=shadow&taskId=u8dda3d94-2511-4db0-9c37-449bac390e1&title=&width=488)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=wqKtz&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 13.12 使用德鲁伊连接池
之前数据源是用我们自己写的。也可以使用别人写好的。例如比较牛的德鲁伊连接池。
第一步：引入德鲁伊连接池的依赖。（毕竟是别人写的）
```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.1.8</version>
</dependency>
```
第二步：将德鲁伊中的数据源配置到spring配置文件中。和配置我们自己写的一样。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="druidDataSource"/>
    </bean>
</beans>
```
测试结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665647176481-660d65ae-f65a-4448-a34d-81ec96ee3b08.png#averageHue=%23f9f6f5&clientId=u27fdab07-dcc6-4&from=paste&height=174&id=uf518af07&originHeight=174&originWidth=1017&originalType=binary&ratio=1&rotation=0&showTitle=false&size=24562&status=done&style=shadow&taskId=uc49a8ec5-431f-4ab5-9059-2fe235736eb&title=&width=1017)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=s8wcV&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 十四、GoF之代理模式
## 14.1 对代理模式的理解
**生活场景1**：牛村的牛二看上了隔壁村小花，牛二不好意思直接找小花，于是牛二找来了媒婆王妈妈。这里面就有一个非常典型的代理模式。牛二不能和小花直接对接，只能找一个中间人。其中王妈妈是代理类，牛二是目标类。王妈妈代替牛二和小花先见个面。（现实生活中的婚介所）【在程序中，对象A和对象B无法直接交互时。】
**生活场景2**：你刚到北京，要租房子，可以自己找，也可以找链家帮你找。其中链家是代理类，你是目标类。你们两个都有共同的行为：找房子。不过链家除了满足你找房子，另外会收取一些费用的。(现实生活中的房产中介)【在程序中，功能需要增强时。】
**西游记场景**：八戒和高小姐的故事。八戒要强抢民女高翠兰。悟空得知此事之后怎么做的？悟空幻化成高小姐的模样。代替高小姐与八戒会面。其中八戒是客户端程序。悟空是代理类。高小姐是目标类。那天夜里，在八戒眼里，眼前的就是高小姐，对于八戒来说，他是不知道眼前的高小姐是悟空幻化的，在他内心里这就是高小姐。所以悟空代替高小姐和八戒亲了嘴儿。这是非常典型的代理模式实现的保护机制。**代理模式中有一个非常重要的特点：对于客户端程序来说，使用代理对象时就像在使用目标对象一样。【在程序中，目标需要被保护时】**
**业务场景**：系统中有A、B、C三个模块，使用这些模块的前提是需要用户登录，也就是说在A模块中要编写判断登录的代码，B模块中也要编写，C模块中还要编写，这些判断登录的代码反复出现，显然代码没有得到复用，可以为A、B、C三个模块提供一个代理，在代理当中写一次登录判断即可。代理的逻辑是：请求来了之后，判断用户是否登录了，如果已经登录了，则执行对应的目标，如果没有登录则跳转到登录页面。【在程序中，目标不但受到保护，并且代码也得到了复用。】
代理模式是GoF23种设计模式之一。属于结构型设计模式。
代理模式的作用是：为其他对象提供一种代理以控制对这个对象的访问。在某些情况下，一个客户不想或者不能直接引用一个对象，此时可以通过一个称之为“代理”的第三者来实现间接引用。代理对象可以在客户端和目标对象之间起到中介的作用，并且可以通过代理对象去掉客户不应该看到的内容和服务或者添加客户需要的额外服务。 通过引入一个新的对象来实现对真实对象的操作或者将新的对象作为真实对象的一个替身，这种实现机制即为代理模式，通过引入代理对象来间接访问一个对象，这就是代理模式的模式动机。
代理模式中的角色：

- 代理类（代理主题）
- 目标类（真实主题）
- 代理类和目标类的公共接口（抽象主题）：客户端在使用代理类时就像在使用目标类，不被客户端所察觉，所以代理类和目标类要有共同的行为，也就是实现共同的接口。

代理模式的类图：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665651817094-af9ecbad-24ae-4c11-9fa2-efe46653df25.png#averageHue=%23f6f6f6&clientId=u27fdab07-dcc6-4&from=paste&height=345&id=u718151a5&originHeight=345&originWidth=517&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18753&status=done&style=shadow&taskId=uc26943d3-18ca-4a96-ba06-6566e4e3e47&title=&width=517)
代理模式在代码实现上，包括两种形式：

- 静态代理
- 动态代理

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=d2REz&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 14.2 静态代理
现在有这样一个接口和实现类：
```java
package com.powernode.mall.service;

/**
 * 订单接口
 * @author 动力节点
 * @version 1.0
 * @className OrderService
 * @since 1.0
 **/
public interface OrderService {
    /**
     * 生成订单
     */
    void generate();

    /**
     * 查看订单详情
     */
    void detail();

    /**
     * 修改订单
     */
    void modify();
}

```
```java
package com.powernode.mall.service.impl;

import com.powernode.mall.service.OrderService;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderServiceImpl
 * @since 1.0
 **/
public class OrderServiceImpl implements OrderService {
    @Override
    public void generate() {
        try {
            Thread.sleep(1234);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已生成");
    }

    @Override
    public void detail() {
        try {
            Thread.sleep(2541);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单信息如下：******");
    }

    @Override
    public void modify() {
        try {
            Thread.sleep(1010);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已修改");
    }
}

```
其中Thread.sleep()方法的调用是为了模拟操作耗时。
项目已上线，并且运行正常，只是客户反馈系统有一些地方运行较慢，要求项目组对系统进行优化。于是项目负责人就下达了这个需求。首先需要搞清楚是哪些业务方法耗时较长，于是让我们统计每个业务方法所耗费的时长。如果是你，你该怎么做呢？
第一种方案：直接修改Java源代码，在每个业务方法中添加统计逻辑，如下：
```java
package com.powernode.mall.service.impl;

import com.powernode.mall.service.OrderService;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderServiceImpl
 * @since 1.0
 **/
public class OrderServiceImpl implements OrderService {
    @Override
    public void generate() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(1234);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已生成");
        long end = System.currentTimeMillis();
        System.out.println("耗费时长"+(end - begin)+"毫秒");
    }

    @Override
    public void detail() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(2541);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单信息如下：******");
        long end = System.currentTimeMillis();
        System.out.println("耗费时长"+(end - begin)+"毫秒");
    }

    @Override
    public void modify() {
        long begin = System.currentTimeMillis();
        try {
            Thread.sleep(1010);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已修改");
        long end = System.currentTimeMillis();
        System.out.println("耗费时长"+(end - begin)+"毫秒");
    }
}
```
需求可以满足，但显然是违背了OCP开闭原则。这种方案不可取。
第二种方案：编写一个子类继承OrderServiceImpl，在子类中重写每个方法，代码如下：
```java
package com.powernode.mall.service.impl;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderServiceImplSub
 * @since 1.0
 **/
public class OrderServiceImplSub extends OrderServiceImpl{
    @Override
    public void generate() {
        long begin = System.currentTimeMillis();
        super.generate();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }

    @Override
    public void detail() {
        long begin = System.currentTimeMillis();
        super.detail();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }

    @Override
    public void modify() {
        long begin = System.currentTimeMillis();
        super.modify();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }
}
```
这种方式可以解决，但是存在两个问题：

- 第一个问题：假设系统中有100个这样的业务类，需要提供100个子类，并且之前写好的创建Service对象的代码，都要修改为创建子类对象。
- 第二个问题：由于采用了继承的方式，导致代码之间的耦合度较高。

这种方案也不可取。
第三种方案：使用代理模式（这里采用静态代理）
可以为OrderService接口提供一个代理类。
```java
package com.powernode.mall.service;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderServiceProxy
 * @since 1.0
 **/
public class OrderServiceProxy implements OrderService{ // 代理对象

    // 目标对象
    private OrderService orderService;

    // 通过构造方法将目标对象传递给代理对象
    public OrderServiceProxy(OrderService orderService) {
        this.orderService = orderService;
    }

    @Override
    public void generate() {
        long begin = System.currentTimeMillis();
        // 执行目标对象的目标方法
        orderService.generate();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }

    @Override
    public void detail() {
        long begin = System.currentTimeMillis();
        // 执行目标对象的目标方法
        orderService.detail();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }

    @Override
    public void modify() {
        long begin = System.currentTimeMillis();
        // 执行目标对象的目标方法
        orderService.modify();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }
}

```
这种方式的优点：符合OCP开闭原则，同时采用的是关联关系，所以程序的耦合度较低。所以这种方案是被推荐的。
编写客户端程序：
```java
package com.powernode.mall;

import com.powernode.mall.service.OrderService;
import com.powernode.mall.service.OrderServiceProxy;
import com.powernode.mall.service.impl.OrderServiceImpl;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        OrderService target = new OrderServiceImpl();
        // 创建代理对象
        OrderService proxy = new OrderServiceProxy(target);
        // 调用代理对象的代理方法
        proxy.generate();
        proxy.modify();
        proxy.detail();
    }
}

```
运行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665711099963-e31eb7f2-4355-43c6-985a-2ed9223a7aee.png#averageHue=%23fbfaf9&clientId=ue2440d3d-c252-4&from=paste&height=250&id=uca6f5849&originHeight=250&originWidth=357&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18052&status=done&style=shadow&taskId=u328feba1-2083-461e-9990-566503accf0&title=&width=357)
以上就是代理模式中的静态代理，其中OrderService接口是代理类和目标类的共同接口。OrderServiceImpl是目标类。OrderServiceProxy是代理类。
大家思考一下：如果系统中业务接口很多，一个接口对应一个代理类，显然也是不合理的，会导致类爆炸。怎么解决这个问题？动态代理可以解决。因为在动态代理中可以在内存中动态的为我们生成代理类的字节码。代理类不需要我们写了。类爆炸解决了，而且代码只需要写一次，代码也会得到复用。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=zkGtL&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 14.3 动态代理
在程序运行阶段，在内存中动态生成代理类，被称为动态代理，目的是为了减少代理类的数量。解决代码复用的问题。
在内存当中动态生成类的技术常见的包括：

- JDK动态代理技术：只能代理接口。
- CGLIB动态代理技术：CGLIB(Code Generation Library)是一个开源项目。是一个强大的，高性能，高质量的Code生成类库，它可以在运行期扩展Java类与实现Java接口。它既可以代理接口，又可以代理类，**底层是通过继承的方式实现的**。性能比JDK动态代理要好。**（底层有一个小而快的字节码处理框架ASM。）**
- Javassist动态代理技术：Javassist是一个开源的分析、编辑和创建Java字节码的类库。是由东京工业大学的数学和计算机科学系的 Shigeru Chiba （千叶 滋）所创建的。它已加入了开放源代码JBoss 应用服务器项目，通过使用Javassist对字节码操作为JBoss实现动态"AOP"框架。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=HkwXL&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 14.3.1 JDK动态代理
我们还是使用静态代理中的例子：一个接口和一个实现类。
```java
package com.powernode.mall.service;

/**
 * 订单接口
 * @author 动力节点
 * @version 1.0
 * @className OrderService
 * @since 1.0
 **/
public interface OrderService {
    /**
     * 生成订单
     */
    void generate();

    /**
     * 查看订单详情
     */
    void detail();

    /**
     * 修改订单
     */
    void modify();
}

```
```java
package com.powernode.mall.service.impl;

import com.powernode.mall.service.OrderService;

/**
 * @author 动力节点
 * @version 1.0
 * @className OrderServiceImpl
 * @since 1.0
 **/
public class OrderServiceImpl implements OrderService {
    @Override
    public void generate() {
        try {
            Thread.sleep(1234);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已生成");
    }

    @Override
    public void detail() {
        try {
            Thread.sleep(2541);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单信息如下：******");
    }

    @Override
    public void modify() {
        try {
            Thread.sleep(1010);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("订单已修改");
    }
}

```
我们在静态代理的时候，除了以上一个接口和一个实现类之外，是不是要写一个代理类UserServiceProxy呀！在动态代理中UserServiceProxy代理类是可以动态生成的。这个类不需要写。我们直接写客户端程序即可：
```java
package com.powernode.mall;

import com.powernode.mall.service.OrderService;
import com.powernode.mall.service.impl.OrderServiceImpl;

import java.lang.reflect.Proxy;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 第一步：创建目标对象
        OrderService target = new OrderServiceImpl();
        // 第二步：创建代理对象
        OrderService orderServiceProxy = Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), 调用处理器对象);
        // 第三步：调用代理对象的代理方法
        orderServiceProxy.detail();
        orderServiceProxy.modify();
        orderServiceProxy.generate();
    }
}
```
以上第二步创建代理对象是需要大家理解的：
OrderService orderServiceProxy = Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), 调用处理器对象);
这行代码做了两件事：

- 第一件事：在内存中生成了代理类的字节码
- 第二件事：创建代理对象

Proxy类全名：java.lang.reflect.Proxy。这是JDK提供的一个类（所以称为JDK动态代理）。主要是通过这个类在内存中生成代理类的字节码。
其中newProxyInstance()方法有三个参数：

- 第一个参数：类加载器。在内存中生成了字节码，要想执行这个字节码，也是需要先把这个字节码加载到内存当中的。所以要指定使用哪个类加载器加载。
- 第二个参数：接口类型。代理类和目标类实现相同的接口，所以要通过这个参数告诉JDK动态代理生成的类要实现哪些接口。
- 第三个参数：调用处理器。这是一个JDK动态代理规定的接口，接口全名：java.lang.reflect.InvocationHandler。显然这是一个回调接口，也就是说调用这个接口中方法的程序已经写好了，就差这个接口的实现类了。

所以接下来我们要写一下java.lang.reflect.InvocationHandler接口的实现类，并且实现接口中的方法，代码如下：
```java
package com.powernode.mall.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className TimerInvocationHandler
 * @since 1.0
 **/
public class TimerInvocationHandler implements InvocationHandler {
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
}
```
InvocationHandler接口中有一个方法invoke，这个invoke方法上有三个参数：

- 第一个参数：Object proxy。代理对象。设计这个参数只是为了后期的方便，如果想在invoke方法中使用代理对象的话，尽管通过这个参数来使用。
- 第二个参数：Method method。目标方法。
- 第三个参数：Object[] args。目标方法调用时要传的参数。

我们将来肯定是要调用“目标方法”的，但要调用目标方法的话，需要“目标对象”的存在，“目标对象”从哪儿来呢？我们可以给TimerInvocationHandler提供一个构造方法，可以通过这个构造方法传过来“目标对象”，代码如下：
```java
package com.powernode.mall.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className TimerInvocationHandler
 * @since 1.0
 **/
public class TimerInvocationHandler implements InvocationHandler {
    // 目标对象
    private Object target;

    // 通过构造方法来传目标对象
    public TimerInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        return null;
    }
}

```
有了目标对象我们就可以在invoke()方法中调用目标方法了。代码如下：
```java
package com.powernode.mall.service;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className TimerInvocationHandler
 * @since 1.0
 **/
public class TimerInvocationHandler implements InvocationHandler {
    // 目标对象
    private Object target;

    // 通过构造方法来传目标对象
    public TimerInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 目标执行之前增强。
        long begin = System.currentTimeMillis();
        // 调用目标对象的目标方法
        Object retValue = method.invoke(target, args);
        // 目标执行之后增强。
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
        // 一定要记得返回哦。
        return retValue;
    }
}

```
到此为止，调用处理器就完成了。接下来，应该继续完善Client程序：
```java
package com.powernode.mall;

import com.powernode.mall.service.OrderService;
import com.powernode.mall.service.TimerInvocationHandler;
import com.powernode.mall.service.impl.OrderServiceImpl;

import java.lang.reflect.Proxy;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        OrderService target = new OrderServiceImpl();
        // 创建代理对象
        OrderService orderServiceProxy = (OrderService) Proxy.newProxyInstance(target.getClass().getClassLoader(),
                                                                                target.getClass().getInterfaces(),
                                                                                new TimerInvocationHandler(target));
        // 调用代理对象的代理方法
        orderServiceProxy.detail();
        orderServiceProxy.modify();
        orderServiceProxy.generate();
    }
}

```
大家可能会比较好奇：那个InvocationHandler接口中的invoke()方法没看见在哪里调用呀？
注意：当你调用代理对象的代理方法的时候，注册在InvocationHandler接口中的invoke()方法会被调用。也就是上面代码第24 25 26行，这三行代码中任意一行代码执行，注册在InvocationHandler接口中的invoke()方法都会被调用。
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665715879232-21eb379f-c3a4-4ffc-868f-441079541feb.png#averageHue=%23faf9f8&clientId=ue2440d3d-c252-4&from=paste&height=257&id=u72b69159&originHeight=257&originWidth=366&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18921&status=done&style=shadow&taskId=u0b1617f4-76ec-4081-9ca3-bd136386c89&title=&width=366)
学到这里可能会感觉有点懵，折腾半天，到最后这不是还得写一个接口的实现类吗？没省劲儿呀？
你要这样想就错了!!!!
我们可以看到，不管你有多少个Service接口，多少个业务类，这个TimerInvocationHandler接口是不是只需要写一次就行了，代码是不是得到复用了！！！！
而且最重要的是，以后程序员只需要关注核心业务的编写了，像这种统计时间的代码根本不需要关注。因为这种统计时间的代码只需要在调用处理器中编写一次即可。
到这里，JDK动态代理的原理就结束了。
不过我们看以下这个代码确实有点繁琐，对于客户端来说，用起来不方便：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665716434406-4e092df4-b1a7-4d16-bbc1-1f134b8f51f7.png#averageHue=%23fcf9f8&clientId=ue2440d3d-c252-4&from=paste&height=296&id=u40396b71&originHeight=296&originWidth=1314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=49308&status=done&style=shadow&taskId=u4ceb735a-a652-4879-83af-454bfff2cb3&title=&width=1314)
我们可以提供一个工具类：ProxyUtil，封装一个方法：
```java
package com.powernode.mall.util;

import com.powernode.mall.service.TimerInvocationHandler;

import java.lang.reflect.Proxy;

/**
 * @author 动力节点
 * @version 1.0
 * @className ProxyUtil
 * @since 1.0
 **/
public class ProxyUtil {
    public static Object newProxyInstance(Object target) {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), 
                target.getClass().getInterfaces(), 
                new TimerInvocationHandler(target));
    }
}
```
这样客户端代码就不需要写那么繁琐了：
```java
package com.powernode.mall;

import com.powernode.mall.service.OrderService;
import com.powernode.mall.service.TimerInvocationHandler;
import com.powernode.mall.service.impl.OrderServiceImpl;
import com.powernode.mall.util.ProxyUtil;

import java.lang.reflect.Proxy;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 创建目标对象
        OrderService target = new OrderServiceImpl();
        // 创建代理对象
        OrderService orderServiceProxy = (OrderService) ProxyUtil.newProxyInstance(target);
        // 调用代理对象的代理方法
        orderServiceProxy.detail();
        orderServiceProxy.modify();
        orderServiceProxy.generate();
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665716690080-a1fac908-d6d8-4605-b24d-2fd9fd19a7e6.png#averageHue=%23f9f7f6&clientId=ue2440d3d-c252-4&from=paste&height=252&id=uee021dd8&originHeight=252&originWidth=293&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18020&status=done&style=shadow&taskId=u81ecf60f-ca43-45a5-8bff-d0c5c0d7ea2&title=&width=293)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=SsGwa&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 14.3.2 CGLIB动态代理
CGLIB既可以代理接口，又可以代理类。底层采用继承的方式实现。所以被代理的目标类不能使用final修饰。
使用CGLIB，需要引入它的依赖：
```xml
<dependency>
  <groupId>cglib</groupId>
  <artifactId>cglib</artifactId>
  <version>3.3.0</version>
</dependency>
```
我们准备一个没有实现接口的类，如下：
```java
package com.powernode.mall.service;

/**
 * @author 动力节点
 * @version 1.0
 * @className UserService
 * @since 1.0
 **/
public class UserService {

    public void login(){
        System.out.println("用户正在登录系统....");
    }

    public void logout(){
        System.out.println("用户正在退出系统....");
    }
}
```
使用CGLIB在内存中为UserService类生成代理类，并创建对象：
```java
package com.powernode.mall;

import com.powernode.mall.service.UserService;
import net.sf.cglib.proxy.Enhancer;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 创建字节码增强器
        Enhancer enhancer = new Enhancer();
        // 告诉cglib要继承哪个类
        enhancer.setSuperclass(UserService.class);
        // 设置回调接口
        enhancer.setCallback(方法拦截器对象);
        // 生成源码，编译class，加载到JVM，并创建代理对象
        UserService userServiceProxy = (UserService)enhancer.create();

        userServiceProxy.login();
        userServiceProxy.logout();

    }
}
```
和JDK动态代理原理差不多，在CGLIB中需要提供的不是InvocationHandler，而是：net.sf.cglib.proxy.MethodInterceptor
编写MethodInterceptor接口实现类：
```java
package com.powernode.mall.service;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className TimerMethodInterceptor
 * @since 1.0
 **/
public class TimerMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object target, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        return null;
    }
}
```
MethodInterceptor接口中有一个方法intercept()，该方法有4个参数：
第一个参数：目标对象
第二个参数：目标方法
第三个参数：目标方法调用时的实参
第四个参数：代理方法
在MethodInterceptor的intercept()方法中调用目标以及添加增强：
```java
package com.powernode.mall.service;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

/**
 * @author 动力节点
 * @version 1.0
 * @className TimerMethodInterceptor
 * @since 1.0
 **/
public class TimerMethodInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object target, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        // 前增强
        long begin = System.currentTimeMillis();
        // 调用目标
        Object retValue = methodProxy.invokeSuper(target, objects);
        // 后增强
        long end = System.currentTimeMillis();
        System.out.println("耗时" + (end - begin) + "毫秒");
        // 一定要返回
        return retValue;
    }
}

```
回调已经写完了，可以修改客户端程序了：
```java
package com.powernode.mall;

import com.powernode.mall.service.TimerMethodInterceptor;
import com.powernode.mall.service.UserService;
import net.sf.cglib.proxy.Enhancer;

/**
 * @author 动力节点
 * @version 1.0
 * @className Client
 * @since 1.0
 **/
public class Client {
    public static void main(String[] args) {
        // 创建字节码增强器
        Enhancer enhancer = new Enhancer();
        // 告诉cglib要继承哪个类
        enhancer.setSuperclass(UserService.class);
        // 设置回调接口
        enhancer.setCallback(new TimerMethodInterceptor());
        // 生成源码，编译class，加载到JVM，并创建代理对象
        UserService userServiceProxy = (UserService)enhancer.create();

        userServiceProxy.login();
        userServiceProxy.logout();

    }
}
```
对于高版本的JDK，如果使用CGLIB，需要在启动项中添加两个启动参数：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665719287350-761d69a9-2666-40b3-9332-91d695f1eb86.png#averageHue=%23f1efee&clientId=u71f508e1-87c2-4&from=paste&height=513&id=u454347c1&originHeight=513&originWidth=712&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36664&status=done&style=shadow&taskId=u88589ad8-76f4-42fd-997f-5f989550926&title=&width=712)

- --add-opens java.base/java.lang=ALL-UNNAMED
- --add-opens java.base/sun.net.util=ALL-UNNAMED

执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665719690752-0b38d1ec-f4fd-4a8e-878c-3496da353fe2.png#averageHue=%23f9f8f7&clientId=u71f508e1-87c2-4&from=paste&height=201&id=u60ff1d9f&originHeight=201&originWidth=328&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15058&status=done&style=shadow&taskId=uf9053cca-6107-4765-9b3d-94828282e01&title=&width=328)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=uXyZw&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十五、面向切面编程AOP
IoC使软件组件松耦合。AOP让你能够捕捉系统中经常使用的功能，把它转化成组件。
AOP（Aspect Oriented Programming）：面向切面编程，面向方面编程。（AOP是一种编程技术）
AOP是对OOP的补充延伸。
AOP底层使用的就是动态代理来实现的。
Spring的AOP使用的动态代理是：JDK动态代理 + CGLIB动态代理技术。Spring在这两种动态代理中灵活切换，如果是代理接口，会默认使用JDK动态代理，如果要代理某个类，这个类没有实现接口，就会切换使用CGLIB。当然，你也可以强制通过一些配置让Spring只使用CGLIB。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=eNvR7&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.1 AOP介绍
一般一个系统当中都会有一些系统服务，例如：日志、事务管理、安全等。这些系统服务被称为：**交叉业务**
这些**交叉业务**几乎是通用的，不管你是做银行账户转账，还是删除用户数据。日志、事务管理、安全，这些都是需要做的。
如果在每一个业务处理过程当中，都掺杂这些交叉业务代码进去的话，存在两方面问题：

- 第一：交叉业务代码在多个业务流程中反复出现，显然这个交叉业务代码没有得到复用。并且修改这些交叉业务代码的话，需要修改多处。
- 第二：程序员无法专注核心业务代码的编写，在编写核心业务代码的同时还需要处理这些交叉业务。

使用AOP可以很轻松的解决以上问题。
请看下图，可以帮助你快速理解AOP的思想：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665732609757-d8ae52ba-915e-49cf-9ef4-c7bcada0d601.png#averageHue=%23f5f1dc&clientId=u71f508e1-87c2-4&from=paste&height=472&id=uef8a4f68&originHeight=472&originWidth=872&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23630&status=done&style=shadow&taskId=u986fcc80-b6e4-46d8-a79e-2c5855dd6ad&title=&width=872)
**用一句话总结AOP：将与核心业务无关的代码独立的抽取出来，形成一个独立的组件，然后以横向交叉的方式应用到业务流程当中的过程被称为AOP。**
**AOP的优点：**

- **第一：代码复用性增强。**
- **第二：代码易维护。**
- **第三：使开发者更关注业务逻辑。**



![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=e32lY&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.2 AOP的七大术语
```java
public class UserService{
    public void do1(){
        System.out.println("do 1");
    }
    public void do2(){
        System.out.println("do 2");
    }
    public void do3(){
        System.out.println("do 3");
    }
    public void do4(){
        System.out.println("do 4");
    }
    public void do5(){
        System.out.println("do 5");
    }
    // 核心业务方法
    public void service(){
        do1();
        do2();
        do3();
        do5();
    }
}
```

- **连接点 Joinpoint**
   - 在程序的整个执行流程中，**可以织入**切面的位置。方法的执行前后，异常抛出之后等位置。
- **切点 Pointcut**
   - 在程序执行流程中，**真正织入**切面的方法。（一个切点对应多个连接点）
- **通知 Advice**
   - 通知又叫增强，就是具体你要织入的代码。
   - 通知包括：
      - 前置通知
      - 后置通知
      - 环绕通知
      - 异常通知
      - 最终通知
- **切面 Aspect**
   - **切点 + 通知就是切面。**
- 织入 Weaving
   - 把通知应用到目标对象上的过程。
- 代理对象 Proxy
   - 一个目标对象被织入通知后产生的新对象。
- 目标对象 Target
   - 被织入通知的对象。

通过下图，大家可以很好的理解AOP的相关术语：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665735638342-44194599-66e2-4c02-a843-8a8b3ba5b0c8.png#averageHue=%23f3f0da&clientId=u71f508e1-87c2-4&from=paste&height=489&id=u20b88b4d&originHeight=489&originWidth=608&originalType=binary&ratio=1&rotation=0&showTitle=false&size=41603&status=done&style=shadow&taskId=u27b5a949-c5ad-4ccd-9b5d-244ab41e6e3&title=&width=608)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Q0wOr&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.3 切点表达式
切点表达式用来定义通知（Advice）往哪些方法上切入。
切入点表达式语法格式：
```
execution([访问控制权限修饰符] 返回值类型 [全限定类名]方法名(形式参数列表) [异常])
```
访问控制权限修饰符：

- 可选项。
- 没写，就是4个权限都包括。
- 写public就表示只包括公开的方法。

返回值类型：

- 必填项。
- * 表示返回值类型任意。

全限定类名：

- 可选项。
- 两个点“..”代表当前包以及子包下的所有类。
- 省略时表示所有的类。

方法名：

- 必填项。
- *表示所有方法。
- set*表示所有的set方法。

形式参数列表：

- 必填项
- () 表示没有参数的方法
- (..) 参数类型和个数随意的方法
- (*) 只有一个参数的方法
- (*, String) 第一个参数类型随意，第二个参数是String的。

异常：

- 可选项。
- 省略时表示任意异常类型。

理解以下的切点表达式：
```java
execution(public * com.powernode.mall.service.*.delete*(..))
```
```java
execution(* com.powernode.mall..*(..))
```
```java
execution(* *(..))
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=c1i63&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.4 使用Spring的AOP
Spring对AOP的实现包括以下3种方式：

- **第一种方式：Spring框架结合AspectJ框架实现的AOP，基于注解方式。**
- **第二种方式：Spring框架结合AspectJ框架实现的AOP，基于XML方式。**
- 第三种方式：Spring框架自己实现的AOP，基于XML配置方式。

实际开发中，都是Spring+AspectJ来实现AOP。所以我们重点学习第一种和第二种方式。
什么是AspectJ？（Eclipse组织的一个支持AOP的框架。AspectJ框架是独立于Spring框架之外的一个框架，Spring框架用了AspectJ） 
AspectJ项目起源于帕洛阿尔托（Palo Alto）研究中心（缩写为PARC）。该中心由Xerox集团资助，Gregor Kiczales领导，从1997年开始致力于AspectJ的开发，1998年第一次发布给外部用户，2001年发布1.0 release。为了推动AspectJ技术和社团的发展，PARC在2003年3月正式将AspectJ项目移交给了Eclipse组织，因为AspectJ的发展和受关注程度大大超出了PARC的预期，他们已经无力继续维持它的发展。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=jMcAu&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 15.4.1 准备工作
使用Spring+AspectJ的AOP需要引入的依赖如下：
```xml
<!--spring context依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-context</artifactId>
  <version>6.0.0-M2</version>
</dependency>
<!--spring aop依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aop</artifactId>
  <version>6.0.0-M2</version>
</dependency>
<!--spring aspects依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>6.0.0-M2</version>
</dependency>
```
Spring配置文件中添加context命名空间和aop命名空间
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

</beans>
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=TxCRE&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 15.4.2 基于AspectJ的AOP注解式开发
#### 实现步骤
第一步：定义目标类以及目标方法
```java
package com.powernode.spring6.service;

// 目标类
public class OrderService {
    // 目标方法
    public void generate(){
        System.out.println("订单已生成！");
    }
}
```
第二步：定义切面类
```java
package com.powernode.spring6.service;

import org.aspectj.lang.annotation.Aspect;

// 切面类
@Aspect
public class MyAspect {
}
```
第三步：目标类和切面类都纳入spring bean管理
在目标类OrderService上添加**@Component**注解。
在切面类MyAspect类上添加**@Component**注解。
第四步：在spring配置文件中添加组建扫描
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--开启组件扫描-->
    <context:component-scan base-package="com.powernode.spring6.service"/>
</beans>
```
第五步：在切面类中添加通知
```java
package com.powernode.spring6.service;

import org.springframework.stereotype.Component;
import org.aspectj.lang.annotation.Aspect;

// 切面类
@Aspect
@Component
public class MyAspect {
    // 这就是需要增强的代码（通知）
    public void advice(){
        System.out.println("我是一个通知");
    }
}

```
第六步：在通知上添加切点表达式
```java
package com.powernode.spring6.service;

import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;
import org.aspectj.lang.annotation.Aspect;

// 切面类
@Aspect
@Component
public class MyAspect {
    
    // 切点表达式
    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    // 这就是需要增强的代码（通知）
    public void advice(){
        System.out.println("我是一个通知");
    }
}
```
**注解@Before表示前置通知。**
第七步：在spring配置文件中启用自动代理
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--开启组件扫描-->
    <context:component-scan base-package="com.powernode.spring6.service"/>
    <!--开启自动代理-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
</beans>
```
<aop:aspectj-autoproxy  proxy-target-class="true"/> 开启自动代理之后，凡事带有@Aspect注解的bean都会生成代理对象。
proxy-target-class="true" 表示采用cglib动态代理。
proxy-target-class="false" 表示采用jdk动态代理。默认值是false。即使写成false，当没有接口的时候，也会自动选择cglib生成代理类。
测试程序：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.service.OrderService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPTest {
    @Test
    public void testAOP(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-aspectj-aop-annotation.xml");
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
        orderService.generate();
    }
}

```
运行结果：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665843923087-e1116f09-2470-46cb-b21a-1526f62cab50.png#averageHue=%233d3e3b&clientId=u34f8a484-08bc-4&from=ui&height=109&id=u792f21b4&originHeight=210&originWidth=532&originalType=binary&ratio=1&rotation=0&showTitle=false&size=45191&status=done&style=shadow&taskId=u5b9f2f5f-c8e8-44be-9073-3a439f1bf0a&title=&width=276)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=G9jzr&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 通知类型
通知类型包括：

- 前置通知：@Before 目标方法执行之前的通知
- 后置通知：@AfterReturning 目标方法执行之后的通知
- 环绕通知：@Around 目标方法之前添加通知，同时目标方法执行之后添加通知。
- 异常通知：@AfterThrowing 发生异常之后执行的通知
- 最终通知：@After 放在finally语句块中的通知

接下来，编写程序来测试这几个通知的执行顺序：
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

// 切面类
@Component
@Aspect
public class MyAspect {

    @Around("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @AfterReturning("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterAdvice(){
        System.out.println("最终通知");
    }

}
```
```java
package com.powernode.spring6.service;

import org.springframework.stereotype.Component;

// 目标类
@Component
public class OrderService {
    // 目标方法
    public void generate(){
        System.out.println("订单已生成！");
    }
}

```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.service.OrderService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPTest {
    @Test
    public void testAOP(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-aspectj-aop-annotation.xml");
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
        orderService.generate();
    }
}
```
执行结果：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665892617792-22cc74a2-6876-4cd1-bb17-87d3b5211cae.png#averageHue=%23333333&clientId=u34f8a484-08bc-4&from=ui&height=228&id=u64d09acb&originHeight=378&originWidth=656&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73879&status=done&style=shadow&taskId=u5653f4f8-bba0-49ae-8455-6e4175b3973&title=&width=395)
通过上面的执行结果就可以判断他们的执行顺序了，这里不再赘述。
结果中没有异常通知，这是因为目标程序执行过程中没有发生异常。我们尝试让目标方法发生异常：
```java
package com.powernode.spring6.service;

import org.springframework.stereotype.Component;

// 目标类
@Component
public class OrderService {
    // 目标方法
    public void generate(){
        System.out.println("订单已生成！");
        if (1 == 1) {
            throw new RuntimeException("模拟异常发生");
        }
    }
}
```
再次执行测试程序，结果如下：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665892847715-75045cd0-63b1-47f9-a77e-05911dc72339.png#averageHue=%23323130&clientId=u34f8a484-08bc-4&from=ui&height=247&id=ub9420670&originHeight=464&originWidth=858&originalType=binary&ratio=1&rotation=0&showTitle=false&size=121416&status=done&style=shadow&taskId=u28bbe456-b066-4f33-9bfe-5a815207ab9&title=&width=456)
通过测试得知，当发生异常之后，最终通知也会执行，因为最终通知@After会出现在finally语句块中。
出现异常之后，**后置通知**和**环绕通知的结束部分**不会执行。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=BJ5eL&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 切面的先后顺序
我们知道，业务流程当中不一定只有一个切面，可能有的切面控制事务，有的记录日志，有的进行安全控制，如果多个切面的话，顺序如何控制：**可以使用@Order注解来标识切面类，为@Order注解的value指定一个整数型的数字，数字越小，优先级越高**。
再定义一个切面类，如下：
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Order(1) //设置优先级
public class YourAspect {

    @Around("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("YourAspect环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("YourAspect环绕通知结束");
    }

    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void beforeAdvice(){
        System.out.println("YourAspect前置通知");
    }

    @AfterReturning("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterReturningAdvice(){
        System.out.println("YourAspect后置通知");
    }

    @AfterThrowing("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterThrowingAdvice(){
        System.out.println("YourAspect异常通知");
    }

    @After("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterAdvice(){
        System.out.println("YourAspect最终通知");
    }
}

```
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

// 切面类
@Component
@Aspect
@Order(2) //设置优先级
public class MyAspect {

    @Around("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @AfterReturning("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterAdvice(){
        System.out.println("最终通知");
    }

}
```
执行测试程序：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665893738167-b3c55a19-6129-4615-813f-9b8dc0f17f40.png#averageHue=%23313131&clientId=u34f8a484-08bc-4&from=ui&height=288&id=u37fcbfae&originHeight=626&originWidth=982&originalType=binary&ratio=1&rotation=0&showTitle=false&size=144865&status=done&style=shadow&taskId=u6264f387-9dcc-49ac-865b-a0ac4af8167&title=&width=451)
通过修改@Order注解的整数值来切换顺序，执行测试程序：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665893833282-2cbc59cc-15a5-44c4-bb20-cbdac65a750d.png#averageHue=%23313131&clientId=u34f8a484-08bc-4&from=ui&height=323&id=ubdf4d786&originHeight=648&originWidth=994&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145756&status=done&style=shadow&taskId=uabd655b1-4b27-47f9-8ee1-fd1dfe91e7f&title=&width=496)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=QgWal&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 优化使用切点表达式
观看以下代码中的切点表达式：
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

// 切面类
@Component
@Aspect
@Order(2)
public class MyAspect {

    @Around("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @Before("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @AfterReturning("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void afterAdvice(){
        System.out.println("最终通知");
    }

}

```
缺点是：

- 第一：切点表达式重复写了多次，没有得到复用。
- 第二：如果要修改切点表达式，需要修改多处，难维护。

可以这样做：将切点表达式单独的定义出来，在需要的位置引入即可。如下：
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

// 切面类
@Component
@Aspect
@Order(2)
public class MyAspect {
    
    @Pointcut("execution(* com.powernode.spring6.service.OrderService.*(..))")
    public void pointcut(){}

    @Around("pointcut()")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        System.out.println("环绕通知开始");
        // 执行目标方法。
        proceedingJoinPoint.proceed();
        System.out.println("环绕通知结束");
    }

    @Before("pointcut()")
    public void beforeAdvice(){
        System.out.println("前置通知");
    }

    @AfterReturning("pointcut()")
    public void afterReturningAdvice(){
        System.out.println("后置通知");
    }

    @AfterThrowing("pointcut()")
    public void afterThrowingAdvice(){
        System.out.println("异常通知");
    }

    @After("pointcut()")
    public void afterAdvice(){
        System.out.println("最终通知");
    }

}

```
使用@Pointcut注解来定义独立的切点表达式。
注意这个@Pointcut注解标注的方法随意，只是起到一个能够让@Pointcut注解编写的位置。
执行测试程序：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665893833282-2cbc59cc-15a5-44c4-bb20-cbdac65a750d.png#averageHue=%23313131&clientId=u34f8a484-08bc-4&from=ui&height=323&id=SQAHo&originHeight=648&originWidth=994&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145756&status=done&style=shadow&taskId=uabd655b1-4b27-47f9-8ee1-fd1dfe91e7f&title=&width=496)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=hmODS&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 全注解式开发AOP
就是编写一个类，在这个类上面使用大量注解来代替spring的配置文件，spring配置文件消失了，如下：
```java
package com.powernode.spring6.service;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan("com.powernode.spring6.service")
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class Spring6Configuration {
}
```
测试程序也变化了：
```java
@Test
public void testAOPWithAllAnnotation(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Spring6Configuration.class);
    OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
    orderService.generate();
}
```
执行结果如下：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665893833282-2cbc59cc-15a5-44c4-bb20-cbdac65a750d.png#averageHue=%23313131&clientId=u34f8a484-08bc-4&from=ui&height=323&id=LSG4d&originHeight=648&originWidth=994&originalType=binary&ratio=1&rotation=0&showTitle=false&size=145756&status=done&style=shadow&taskId=uabd655b1-4b27-47f9-8ee1-fd1dfe91e7f&title=&width=496)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=edX9r&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 15.4.3 基于XML配置方式的AOP（了解）
第一步：编写目标类
```java
package com.powernode.spring6.service;

// 目标类
public class VipService {
    public void add(){
        System.out.println("保存vip信息。");
    }
}
```
第二步：编写切面类，并且编写通知
```java
package com.powernode.spring6.service;

import org.aspectj.lang.ProceedingJoinPoint;

// 负责计时的切面类
public class TimerAspect {
    
    public void time(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long begin = System.currentTimeMillis();
        //执行目标
        proceedingJoinPoint.proceed();
        long end = System.currentTimeMillis();
        System.out.println("耗时"+(end - begin)+"毫秒");
    }
}

```
第三步：编写spring配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <!--纳入spring bean管理-->
    <bean id="vipService" class="com.powernode.spring6.service.VipService"/>
    <bean id="timerAspect" class="com.powernode.spring6.service.TimerAspect"/>

    <!--aop配置-->
    <aop:config>
        <!--切点表达式-->
        <aop:pointcut id="p" expression="execution(* com.powernode.spring6.service.VipService.*(..))"/>
        <!--切面-->
        <aop:aspect ref="timerAspect">
            <!--切面=通知 + 切点-->
            <aop:around method="time" pointcut-ref="p"/>
        </aop:aspect>
    </aop:config>
</beans>
```
测试程序：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.service.VipService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AOPTest3 {

    @Test
    public void testAOPXml(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring-aop-xml.xml");
        VipService vipService = applicationContext.getBean("vipService", VipService.class);
        vipService.add();
    }
}

```
执行结果：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665902800121-49540c48-d6c2-4909-874d-e1a485e67ea5.png#averageHue=%23333333&clientId=u34f8a484-08bc-4&from=ui&height=135&id=uc93a20cf&originHeight=306&originWidth=794&originalType=binary&ratio=1&rotation=0&showTitle=false&size=73341&status=done&style=shadow&taskId=u9a1ba488-afcf-493f-8b12-389153b34bf&title=&width=351)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Eiujl&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.5 AOP的实际案例：事务处理
项目中的事务控制是在所难免的。在一个业务流程当中，可能需要多条DML语句共同完成，为了保证数据的安全，这多条DML语句要么同时成功，要么同时失败。这就需要添加事务控制的代码。例如以下伪代码：
```java
class 业务类1{
    public void 业务方法1(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
    public void 业务方法2(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
    public void 业务方法3(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
}

class 业务类2{
    public void 业务方法1(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
    public void 业务方法2(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
    public void 业务方法3(){
        try{
            // 开启事务
            startTransaction();
            
            // 执行核心业务逻辑
            step1();
            step2();
            step3();
            ....
            
            // 提交事务
            commitTransaction();
        }catch(Exception e){
            // 回滚事务
            rollbackTransaction();
        }
    }
}
//......
```
可以看到，这些业务类中的每一个业务方法都是需要控制事务的，而控制事务的代码又是固定的格式，都是：
```java
try{
    // 开启事务
    startTransaction();

    // 执行核心业务逻辑
    //......

    // 提交事务
    commitTransaction();
}catch(Exception e){
    // 回滚事务
    rollbackTransaction();
}
```
这个控制事务的代码就是和业务逻辑没有关系的“**交叉业务**”。以上伪代码当中可以看到这些交叉业务的代码没有得到复用，并且如果这些交叉业务代码需要修改，那必然需要修改多处，难维护，怎么解决？可以采用AOP思想解决。可以把以上控制事务的代码作为环绕通知，切入到目标类的方法当中。接下来我们做一下这件事，有两个业务类，如下：
```java
package com.powernode.spring6.biz;

import org.springframework.stereotype.Component;

@Component
// 业务类
public class AccountService {
    // 转账业务方法
    public void transfer(){
        System.out.println("正在进行银行账户转账");
    }
    // 取款业务方法
    public void withdraw(){
        System.out.println("正在进行取款操作");
    }
}

```
```java
package com.powernode.spring6.biz;

import org.springframework.stereotype.Component;

@Component
// 业务类
public class OrderService {
    // 生成订单
    public void generate(){
        System.out.println("正在生成订单");
    }
    // 取消订单
    public void cancel(){
        System.out.println("正在取消订单");
    }
}
```
注意，以上两个业务类已经纳入spring bean的管理，因为都添加了@Component注解。
接下来我们给以上两个业务类的4个方法添加事务控制代码，使用AOP来完成：
```java
package com.powernode.spring6.biz;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

@Aspect
@Component
// 事务切面类
public class TransactionAspect {
    
    @Around("execution(* com.powernode.spring6.biz..*(..))")
    public void aroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
        try {
            System.out.println("开启事务");
            // 执行目标
            proceedingJoinPoint.proceed();
            System.out.println("提交事务");
        } catch (Throwable e) {
            System.out.println("回滚事务");
        }
    }
}

```
你看，这个事务控制代码是不是只需要写一次就行了，并且修改起来也没有成本。编写测试程序：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.biz.AccountService;
import com.powernode.spring6.biz.OrderService;
import com.powernode.spring6.service.Spring6Configuration;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AOPTest2 {
    @Test
    public void testTransaction(){
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Spring6Configuration.class);
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);
        AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
        // 生成订单
        orderService.generate();
        // 取消订单
        orderService.cancel();
        // 转账
        accountService.transfer();
        // 取款
        accountService.withdraw();
    }
}
```
执行结果：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665899075177-6660d491-06d4-4296-b69a-b54716179c9d.png#averageHue=%23313131&clientId=u34f8a484-08bc-4&from=ui&height=365&id=ue7ff086e&originHeight=676&originWidth=846&originalType=binary&ratio=1&rotation=0&showTitle=false&size=125447&status=done&style=shadow&taskId=u8dd026e7-67bb-486c-bc8b-8ce1fb7e15c&title=&width=457)
通过测试可以看到，所有的业务方法都添加了事务控制的代码。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=qeZ3v&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 15.6 AOP的实际案例：安全日志
需求是这样的：项目开发结束了，已经上线了。运行正常。客户提出了新的需求：凡事在系统中进行修改操作的，删除操作的，新增操作的，都要把这个人记录下来。因为这几个操作是属于危险行为。例如有业务类和业务方法：
```java
package com.powernode.spring6.biz;

import org.springframework.stereotype.Component;

@Component
//用户业务
public class UserService {
    public void getUser(){
        System.out.println("获取用户信息");
    }
    public void saveUser(){
        System.out.println("保存用户");
    }
    public void deleteUser(){
        System.out.println("删除用户");
    }
    public void modifyUser(){
        System.out.println("修改用户");
    }
}
```

```java
package com.powernode.spring6.biz;

import org.springframework.stereotype.Component;

// 商品业务类
@Component
public class ProductService {
    public void getProduct(){
        System.out.println("获取商品信息");
    }
    public void saveProduct(){
        System.out.println("保存商品");
    }
    public void deleteProduct(){
        System.out.println("删除商品");
    }
    public void modifyProduct(){
        System.out.println("修改商品");
    }
}


```
注意：已经添加了@Component注解。
接下来我们使用aop来解决上面的需求：编写一个负责安全的切面类
```java
package com.powernode.spring6.biz;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class SecurityAspect {

    @Pointcut("execution(* com.powernode.spring6.biz..save*(..))")
    public void savePointcut(){}

    @Pointcut("execution(* com.powernode.spring6.biz..delete*(..))")
    public void deletePointcut(){}

    @Pointcut("execution(* com.powernode.spring6.biz..modify*(..))")
    public void modifyPointcut(){}

    @Before("savePointcut() || deletePointcut() || modifyPointcut()")
    public void beforeAdivce(JoinPoint joinpoint){
        System.out.println("XXX操作员正在操作"+joinpoint.getSignature().getName()+"方法");
    }
}

```
```java
@Test
public void testSecurity(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Spring6Configuration.class);
    UserService userService = applicationContext.getBean("userService", UserService.class);
    ProductService productService = applicationContext.getBean("productService", ProductService.class);
    userService.getUser();
    userService.saveUser();
    userService.deleteUser();
    userService.modifyUser();
    productService.getProduct();
    productService.saveProduct();
    productService.deleteProduct();
    productService.modifyProduct();
}
```
执行结果：
![5F9597E7-7930-4384-95C2-CF64C9DDA9F3.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1665901327786-9bfab382-61a3-4d1e-abe5-728b242eb3a2.png#averageHue=%23333333&clientId=u34f8a484-08bc-4&from=ui&height=425&id=ue0b3f754&originHeight=852&originWidth=844&originalType=binary&ratio=1&rotation=0&showTitle=false&size=203543&status=done&style=shadow&taskId=ucddfd42f-50b8-425d-9ace-c36b67eef63&title=&width=421)
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Wcrz9&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十六、Spring对事务的支持
## 16.1 事务概述

- 什么是事务
   - 在一个业务流程当中，通常需要多条DML（insert delete update）语句共同联合才能完成，这多条DML语句必须同时成功，或者同时失败，这样才能保证数据的安全。
   - 多条DML要么同时成功，要么同时失败，这叫做事务。
   - 事务：Transaction（tx）
- 事务的四个处理过程：
   - 第一步：开启事务 (start transaction)
   - 第二步：执行核心业务代码
   - 第三步：提交事务（如果核心业务处理过程中没有出现异常）(commit transaction)
   - 第四步：回滚事务（如果核心业务处理过程中出现异常）(rollback transaction)
- 事务的四个特性：
   - A 原子性：事务是最小的工作单元，不可再分。
   - C 一致性：事务要求要么同时成功，要么同时失败。事务前和事务后的总量不变。
   - I 隔离性：事务和事务之间因为有隔离性，才可以保证互不干扰。
   - D 持久性：持久性是事务结束的标志。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=YToTk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 16.2 引入事务场景
以银行账户转账为例学习事务。两个账户act-001和act-002。act-001账户向act-002账户转账10000，必须同时成功，或者同时失败。（一个减成功，一个加成功， 这两条update语句必须同时成功，或同时失败。）
连接数据库的技术采用Spring框架的JdbcTemplate。
采用三层架构搭建：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666495641174-069ee06f-097c-4f44-9a29-ca3e701d666b.png#averageHue=%23f7f8f5&clientId=uae187c3b-e934-4&from=paste&height=366&id=u78262625&originHeight=508&originWidth=919&originalType=binary&ratio=1&rotation=0&showTitle=false&size=99941&status=done&style=shadow&taskId=u091a372f-ef95-48b9-8fda-dcae80e1468&title=&width=663)
模块名：spring6-013-tx-bank（依赖如下）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>spring6-013-tx-bank</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!--仓库-->
    <repositories>
        <!--spring里程碑版本的仓库-->
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <!--依赖-->
    <dependencies>
        <!--spring context-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--spring jdbc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
      <!--德鲁伊连接池-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.13</version>
        </dependency>
      <!--@Resource注解-->
        <dependency>
            <groupId>jakarta.annotation</groupId>
            <artifactId>jakarta.annotation-api</artifactId>
            <version>2.1.1</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=AfKBK&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第一步：准备数据库表
表结构：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666496097440-75d21db2-588b-4f6a-bd40-149c3de6f27d.png#averageHue=%23f5f3f2&clientId=uae187c3b-e934-4&from=paste&height=183&id=ue8b0909d&originHeight=183&originWidth=792&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18901&status=done&style=shadow&taskId=ubf854639-fb59-4c00-b536-55f060658ad&title=&width=792)
表数据：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666496136146-5cc1d848-0ad4-425d-a1fc-8b59b5d0b91f.png#averageHue=%23f3f2f0&clientId=uae187c3b-e934-4&from=paste&height=134&id=u2efb26dc&originHeight=134&originWidth=339&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7935&status=done&style=shadow&taskId=uff2b8130-c329-47be-b8a7-d27d204ff1c&title=&width=339)
### 第二步：创建包结构
com.powernode.bank.pojo
com.powernode.bank.service
com.powernode.bank.service.impl
com.powernode.bank.dao
com.powernode.bank.dao.impl

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=F9pL2&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第三步：准备POJO类
```java
package com.powernode.bank.pojo;

/**
 * @author 动力节点
 * @version 1.0
 * @className Account
 * @since 1.0
 **/
public class Account {
    private String actno;
    private Double balance;

    @Override
    public String toString() {
        return "Account{" +
                "actno='" + actno + '\'' +
                ", balance=" + balance +
                '}';
    }

    public Account() {
    }

    public Account(String actno, Double balance) {
        this.actno = actno;
        this.balance = balance;
    }

    public String getActno() {
        return actno;
    }

    public void setActno(String actno) {
        this.actno = actno;
    }

    public Double getBalance() {
        return balance;
    }

    public void setBalance(Double balance) {
        this.balance = balance;
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=iKa7x&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第四步：编写持久层
```java
package com.powernode.bank.dao;

import com.powernode.bank.pojo.Account;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountDao
 * @since 1.0
 **/
public interface AccountDao {

    /**
     * 根据账号查询余额
     * @param actno
     * @return
     */
    Account selectByActno(String actno);

    /**
     * 更新账户
     * @param act
     * @return
     */
    int update(Account act);

}

```
```java
package com.powernode.bank.dao.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import jakarta.annotation.Resource;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountDaoImpl
 * @since 1.0
 **/
@Repository("accountDao")
public class AccountDaoImpl implements AccountDao {

    @Resource(name = "jdbcTemplate")
    private JdbcTemplate jdbcTemplate;

    @Override
    public Account selectByActno(String actno) {
        String sql = "select actno, balance from t_act where actno = ?";
        Account account = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(Account.class), actno);
        return account;
    }

    @Override
    public int update(Account act) {
        String sql = "update t_act set balance = ? where actno = ?";
        int count = jdbcTemplate.update(sql, act.getBalance(), act.getActno());
        return count;
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=N5Lo3&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第五步：编写业务层
```java
package com.powernode.bank.service;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountService
 * @since 1.0
 **/
public interface AccountService {

    /**
     * 转账
     * @param fromActno
     * @param toActno
     * @param money
     */
    void transfer(String fromActno, String toActno, double money);
}

```
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountServiceImpl
 * @since 1.0
 **/
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    @Override
    public void transfer(String fromActno, String toActno, double money) {
        // 查询账户余额是否充足
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new RuntimeException("账户余额不足");
        }
        // 余额充足，开始转账
        Account toAct = accountDao.selectByActno(toActno);
        fromAct.setBalance(fromAct.getBalance() - money);
        toAct.setBalance(toAct.getBalance() + money);
        int count = accountDao.update(fromAct);
        count += accountDao.update(toAct);
        if (count != 2) {
            throw new RuntimeException("转账失败，请联系银行");
        }
    }
}

```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=HhQOV&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第六步：编写Spring配置文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.powernode.bank"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

</beans>
```

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=LLuPT&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 第七步：编写表示层（测试程序）
```java
package com.powernode.spring6.test;

import com.powernode.bank.service.AccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className BankTest
 * @since 1.0
 **/
public class BankTest {
    @Test
    public void testTransfer(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
        try {
            accountService.transfer("act-001", "act-002", 10000);
            System.out.println("转账成功");
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666497683531-b14430f2-b90e-4555-8552-1de9747c9fcc.png#averageHue=%23f7f4f3&clientId=uae187c3b-e934-4&from=paste&height=167&id=u3b1a7771&originHeight=167&originWidth=545&originalType=binary&ratio=1&rotation=0&showTitle=false&size=18404&status=done&style=shadow&taskId=u37ae7808-00d8-4639-ae1c-21a20cebd45&title=&width=545)
数据变化：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666497727323-b2ca34c9-99c6-4b23-8d3b-8dbe3009d3e9.png#averageHue=%23f4f3f1&clientId=uae187c3b-e934-4&from=paste&height=146&id=u349bac3e&originHeight=146&originWidth=366&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8216&status=done&style=shadow&taskId=ua8cb3724-50c9-4fe1-884c-aefbbab9044&title=&width=366)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=X4z5y&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 模拟异常
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountServiceImpl
 * @since 1.0
 **/
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    @Override
    public void transfer(String fromActno, String toActno, double money) {
        // 查询账户余额是否充足
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new RuntimeException("账户余额不足");
        }
        // 余额充足，开始转账
        Account toAct = accountDao.selectByActno(toActno);
        fromAct.setBalance(fromAct.getBalance() - money);
        toAct.setBalance(toAct.getBalance() + money);
        int count = accountDao.update(fromAct);
        
        // 模拟异常
        String s = null;
        s.toString();

        count += accountDao.update(toAct);
        if (count != 2) {
            throw new RuntimeException("转账失败，请联系银行");
        }
    }
}

```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666497808309-c50af959-1a57-480c-9f31-76f6ce3b555a.png#averageHue=%23f9f5f3&clientId=uae187c3b-e934-4&from=paste&height=222&id=ufadbc5cf&originHeight=222&originWidth=1121&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50039&status=done&style=shadow&taskId=uddcc7f27-6fa7-478e-be1f-763193c1efe&title=&width=1121)
数据库表中数据：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666497824308-bdd8f11f-8f99-4195-81c4-c37721627f4c.png#averageHue=%23f2f1ef&clientId=uae187c3b-e934-4&from=paste&height=136&id=u953308f7&originHeight=136&originWidth=298&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7521&status=done&style=shadow&taskId=ud299b8c4-ea8d-485c-b51f-60a57ab6d2b&title=&width=298)
**丢了1万。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=NKgSU&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 16.3 Spring对事务的支持
### Spring实现事务的两种方式

- 编程式事务
   - 通过编写代码的方式来实现事务的管理。
- 声明式事务
   - 基于注解方式
   - 基于XML配置方式
### Spring事务管理API
Spring对事务的管理底层实现方式是基于AOP实现的。采用AOP的方式进行了封装。所以Spring专门针对事务开发了一套API，API的核心接口如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666504216275-1b6a9ac4-6958-4cdf-9323-7a79a08d059d.png#averageHue=%23fbf9db&clientId=uae187c3b-e934-4&from=paste&height=200&id=ue4d38259&originHeight=200&originWidth=573&originalType=binary&ratio=1&rotation=0&showTitle=false&size=26513&status=done&style=shadow&taskId=ude8d8f0a-04ae-4499-9d7f-fc4811d1299&title=&width=573)
PlatformTransactionManager接口：spring事务管理器的核心接口。在**Spring6**中它有两个实现：

- DataSourceTransactionManager：支持JdbcTemplate、MyBatis、Hibernate等事务管理。
- JtaTransactionManager：支持分布式事务管理。

如果要在Spring6中使用JdbcTemplate，就要使用DataSourceTransactionManager来管理事务。（Spring内置写好了，可以直接用。）

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=L4o3V&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 声明式事务之注解实现方式

- 第一步：在spring配置文件中配置事务管理器。
```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>
```

- 第二步：在spring配置文件中引入tx命名空间。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
```

- 第三步：在spring配置文件中配置“事务注解驱动器”，开始注解的方式控制事务。
```xml
<tx:annotation-driven transaction-manager="transactionManager"/>
```

- 第四步：在service类上或方法上添加@Transactional注解

在类上添加该注解，该类中所有的方法都有事务。在某个方法上添加该注解，表示只有这个方法使用事务。
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountServiceImpl
 * @since 1.0
 **/
@Service("accountService")
@Transactional
public class AccountServiceImpl implements AccountService {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    @Override
    public void transfer(String fromActno, String toActno, double money) {
        // 查询账户余额是否充足
        Account fromAct = accountDao.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new RuntimeException("账户余额不足");
        }
        // 余额充足，开始转账
        Account toAct = accountDao.selectByActno(toActno);
        fromAct.setBalance(fromAct.getBalance() - money);
        toAct.setBalance(toAct.getBalance() + money);
        int count = accountDao.update(fromAct);

        // 模拟异常
        String s = null;
        s.toString();

        count += accountDao.update(toAct);
        if (count != 2) {
            throw new RuntimeException("转账失败，请联系银行");
        }
    }
}

```
当前数据库表中的数据：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666505321919-85dd9adb-bceb-49ef-826f-5a3ddf7699a0.png#averageHue=%23f3f2f0&clientId=uae187c3b-e934-4&from=paste&height=130&id=ub41a6cea&originHeight=130&originWidth=362&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8347&status=done&style=shadow&taskId=u144a0931-0247-4320-aee2-fd431d1bcd8&title=&width=362)
执行测试程序：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666505358758-2a264b1c-3435-4f90-a42f-801001170a2b.png#averageHue=%23f8f4f2&clientId=uae187c3b-e934-4&from=paste&height=240&id=u601972d5&originHeight=240&originWidth=1149&originalType=binary&ratio=1&rotation=0&showTitle=false&size=57873&status=done&style=shadow&taskId=u8197db4e-7a4a-48cc-a5a5-58122ade31a&title=&width=1149)
虽然出现异常了，再次查看数据库表中数据：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666505321919-85dd9adb-bceb-49ef-826f-5a3ddf7699a0.png#averageHue=%23f3f2f0&clientId=uae187c3b-e934-4&from=paste&height=130&id=F4ohV&originHeight=130&originWidth=362&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8347&status=done&style=shadow&taskId=u144a0931-0247-4320-aee2-fd431d1bcd8&title=&width=362)
通过测试，发现数据没有变化，事务起作用了。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=S232S&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 事务属性
#### 事务属性包括哪些
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666506552984-8a4f9d42-73ba-4ded-853d-564d27340db5.png#averageHue=%23fdf7f5&clientId=uae187c3b-e934-4&from=paste&height=838&id=u30f642cf&originHeight=838&originWidth=849&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93371&status=done&style=shadow&taskId=u72868c13-3ce4-41b4-a721-875a05c55c6&title=&width=849)
事务中的重点属性：

- 事务传播行为
- 事务隔离级别
- 事务超时
- 只读事务
- 设置出现哪些异常回滚事务
- 设置出现哪些异常不回滚事务

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=ryvw1&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 事务传播行为
什么是事务的传播行为？
在service类中有a()方法和b()方法，a()方法上有事务，b()方法上也有事务，当a()方法执行过程中调用了b()方法，事务是如何传递的？合并到一个事务里？还是开启一个新的事务？这就是事务传播行为。
事务传播行为在spring框架中被定义为枚举类型：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666505960049-06173489-15fc-4d16-94f3-1a9025f85d8c.png#averageHue=%23fbfaf7&clientId=uae187c3b-e934-4&from=paste&height=324&id=u26bc5c0e&originHeight=324&originWidth=694&originalType=binary&ratio=1&rotation=0&showTitle=false&size=43060&status=done&style=shadow&taskId=ub319df56-8dd6-40bb-bc15-0226b2d8165&title=&width=694)
一共有七种传播行为：

- REQUIRED：支持当前事务，如果不存在就新建一个(默认)**【没有就新建，有就加入】**
- SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行**【有就加入，没有就不管了】**
- MANDATORY：必须运行在一个事务中，如果当前没有事务正在发生，将抛出一个异常**【有就加入，没有就抛异常】**
- REQUIRES_NEW：开启一个新的事务，如果一个事务已经存在，则将这个存在的事务挂起**【不管有没有，直接开启一个新事务，开启的新事务和之前的事务不存在嵌套关系，之前事务被挂起】**
- NOT_SUPPORTED：以非事务方式运行，如果有事务存在，挂起当前事务**【不支持事务，存在就挂起】**
- NEVER：以非事务方式运行，如果有事务存在，抛出异常**【不支持事务，存在就抛异常】**
- NESTED：如果当前正有一个事务在进行中，则该方法应当运行在一个嵌套式事务中。被嵌套的事务可以独立于外层事务进行提交或回滚。如果外层事务不存在，行为就像REQUIRED一样。**【有事务的话，就在这个事务里再嵌套一个完全独立的事务，嵌套的事务可以独立的提交和回滚。没有事务就和REQUIRED一样。】**

在代码中设置事务的传播行为：
```java
@Transactional(propagation = Propagation.REQUIRED)
```
可以编写程序测试一下传播行为：
```java
@Transactional(propagation = Propagation.REQUIRED)
public void save(Account act) {

    // 这里调用dao的insert方法。
    accountDao.insert(act); // 保存act-003账户

    // 创建账户对象
    Account act2 = new Account("act-004", 1000.0);
    try {
        accountService.save(act2); // 保存act-004账户
    } catch (Exception e) {

    }
    // 继续往后进行我当前1号事务自己的事儿。
}
```
```java
@Override
//@Transactional(propagation = Propagation.REQUIRED)
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void save(Account act) {
    accountDao.insert(act);
    // 模拟异常
    String s = null;
    s.toString();

    // 事儿没有处理完，这个大括号当中的后续也许还有其他的DML语句。
}
```
**一定要集成Log4j2日志框架，在日志信息中可以看到更加详细的信息。**
#### 事务隔离级别
事务隔离级别类似于教室A和教室B之间的那道墙，隔离级别越高表示墙体越厚。隔音效果越好。
数据库中读取数据存在的三大问题：（三大读问题）

- **脏读：读取到没有提交到数据库的数据，叫做脏读。**
- **不可重复读：在同一个事务当中，第一次和第二次读取的数据不一样。**
- **幻读：读到的数据是假的。**

事务隔离级别包括四个级别：

- 读未提交：READ_UNCOMMITTED
   - 这种隔离级别，存在脏读问题，所谓的脏读(dirty read)表示能够读取到其它事务未提交的数据。
- 读提交：READ_COMMITTED
   - 解决了脏读问题，其它事务提交之后才能读到，但存在不可重复读问题。
- 可重复读：REPEATABLE_READ
   - 解决了不可重复读，可以达到可重复读效果，只要当前事务不结束，读取到的数据一直都是一样的。但存在幻读问题。
- 序列化：SERIALIZABLE
   - 解决了幻读问题，事务排队执行。不支持并发。

大家可以通过一个表格来记忆：

| **隔离级别** | **脏读** | **不可重复读** | **幻读** |
| --- | --- | --- | --- |
| 读未提交 | **有** | **有** | **有** |
| 读提交 | 无 | **有** | **有** |
| 可重复读 | 无 | 无 | **有** |
| 序列化 | 无 | 无 | 无 |

在Spring代码中如何设置隔离级别？
隔离级别在spring中以枚举类型存在：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666508609641-2c838566-7334-4cf1-b452-0fed9aaebf3d.png#averageHue=%23fcfbfa&clientId=uae187c3b-e934-4&from=paste&height=217&id=u9ea08dc2&originHeight=217&originWidth=390&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25744&status=done&style=shadow&taskId=u275afa85-e7b6-45c9-9fc9-b5164d26048&title=&width=390)
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```
测试事务隔离级别：READ_UNCOMMITTED 和 READ_COMMITTED
怎么测试：一个service负责插入，一个service负责查询。负责插入的service要模拟延迟。
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author 动力节点
 * @version 1.0
 * @className IsolationService1
 * @since 1.0
 **/
@Service("i1")
public class IsolationService1 {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    // 1号
    // 负责查询
    // 当前事务可以读取到别的事务没有提交的数据。
    //@Transactional(isolation = Isolation.READ_UNCOMMITTED)
    // 对方事务提交之后的数据我才能读取到。
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void getByActno(String actno) {
        Account account = accountDao.selectByActno(actno);
        System.out.println("查询到的账户信息：" + account);
    }

}

```
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.dao.AccountDao;
import com.powernode.bank.pojo.Account;
import jakarta.annotation.Resource;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * @author 动力节点
 * @version 1.0
 * @className IsolationService2
 * @since 1.0
 **/
@Service("i2")
public class IsolationService2 {

    @Resource(name = "accountDao")
    private AccountDao accountDao;

    // 2号
    // 负责insert
    @Transactional
    public void save(Account act) {
        accountDao.insert(act);
        // 睡眠一会
        try {
            Thread.sleep(1000 * 20);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}

```
测试程序
```java
@Test
public void testIsolation1(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    IsolationService1 i1 = applicationContext.getBean("i1", IsolationService1.class);
    i1.getByActno("act-004");
}

@Test
public void testIsolation2(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
    IsolationService2 i2 = applicationContext.getBean("i2", IsolationService2.class);
    Account act = new Account("act-004", 1000.0);
    i2.save(act);
}
```
通过执行结果可以清晰的看出隔离级别不同，执行效果不同。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=o1P62&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 事务超时
代码如下：
```java
@Transactional(timeout = 10)
```
以上代码表示设置事务的超时时间为10秒。
**表示超过10秒如果该事务中所有的DML语句还没有执行完毕的话，最终结果会选择回滚。**
默认值-1，表示没有时间限制。
**这里有个坑，事务的超时时间指的是哪段时间？**
**在当前事务当中，最后一条DML语句执行之前的时间。如果最后一条DML语句后面很有很多业务逻辑，这些业务代码执行的时间不被计入超时时间。**
```java
@Transactional(timeout = 10) // 设置事务超时时间为10秒。
public void save(Account act) {
    accountDao.insert(act);
    // 睡眠一会
    try {
        Thread.sleep(1000 * 15);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
```java
@Transactional(timeout = 10) // 设置事务超时时间为10秒。
public void save(Account act) {
    // 睡眠一会
    try {
        Thread.sleep(1000 * 15);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    accountDao.insert(act);
}
```
**当然，如果想让整个方法的所有代码都计入超时时间的话，可以在方法最后一行添加一行无关紧要的DML语句。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=nPfS1&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 只读事务
代码如下：
```java
@Transactional(readOnly = true)
```
将当前事务设置为只读事务，在该事务执行过程中只允许select语句执行，delete insert update均不可执行。
该特性的作用是：**启动spring的优化策略。提高select语句执行效率。**
如果该事务中确实没有增删改操作，建议设置为只读事务。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=Nl7Jj&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
#### 设置哪些异常回滚事务
代码如下：
```java
@Transactional(rollbackFor = RuntimeException.class)
```
表示只有发生RuntimeException异常或该异常的子类异常才回滚。
#### 设置哪些异常不回滚事务
代码如下：
```java
@Transactional(noRollbackFor = NullPointerException.class)
```
表示发生NullPointerException或该异常的子类异常不回滚，其他异常则回滚。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=IEOQt&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 事务的全注解式开发
编写一个类来代替配置文件，代码如下：
```java
package com.powernode.bank;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.sql.DataSource;

/**
 * @author 动力节点
 * @version 1.0
 * @className Spring6Config
 * @since 1.0
 **/
@Configuration
@ComponentScan("com.powernode.bank")
@EnableTransactionManagement
public class Spring6Config {

    @Bean
    public DataSource getDataSource(){
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/spring6");
        dataSource.setUsername("root");
        dataSource.setPassword("root");
        return dataSource;
    }

    @Bean(name = "jdbcTemplate")
    public JdbcTemplate getJdbcTemplate(DataSource dataSource){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource);
        return jdbcTemplate;
    }

    @Bean
    public DataSourceTransactionManager getDataSourceTransactionManager(DataSource dataSource){
        DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager();
        dataSourceTransactionManager.setDataSource(dataSource);
        return dataSourceTransactionManager;
    }

}

```
测试程序如下：
```java
@Test
public void testNoXml(){
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Spring6Config.class);
    AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
    try {
        accountService.transfer("act-001", "act-002", 10000);
        System.out.println("转账成功");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666511446141-925a1a0e-05ab-4306-996f-532878d5c5a3.png#averageHue=%23f8f4f2&clientId=uae187c3b-e934-4&from=paste&height=285&id=ud9a316f2&originHeight=285&originWidth=1125&originalType=binary&ratio=1&rotation=0&showTitle=false&size=68631&status=done&style=shadow&taskId=u6f98b110-c9e0-4886-a737-bbc965769fe&title=&width=1125)
数据库表中数据：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666511460275-5ede53ce-9ad1-4bce-935a-32436a46c83a.png#averageHue=%23f4f2f1&clientId=uae187c3b-e934-4&from=paste&height=146&id=u4b73eebc&originHeight=146&originWidth=318&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8042&status=done&style=shadow&taskId=u45c7cde0-4d4b-4941-ad13-53606b3f4c8&title=&width=318)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=I2ACU&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
### 声明式事务之XML实现方式
配置步骤：

- 第一步：配置事务管理器
- 第二步：配置通知
- 第三步：配置切面

记得添加aspectj的依赖：
```xml
<!--aspectj依赖-->
<dependency>
  <groupId>org.springframework</groupId>
  <artifactId>spring-aspects</artifactId>
  <version>6.0.0-M2</version>
</dependency>
```
Spring配置文件如下：
**记得添加aop的命名空间。**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd
                           http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="com.powernode.bank"/>

    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/spring6"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置通知-->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="save*" propagation="REQUIRED" rollback-for="java.lang.Throwable"/>
            <tx:method name="del*" propagation="REQUIRED" rollback-for="java.lang.Throwable"/>
            <tx:method name="update*" propagation="REQUIRED" rollback-for="java.lang.Throwable"/>
            <tx:method name="transfer*" propagation="REQUIRED" rollback-for="java.lang.Throwable"/>
        </tx:attributes>
    </tx:advice>

    <!--配置切面-->
    <aop:config>
        <aop:pointcut id="txPointcut" expression="execution(* com.powernode.bank.service..*(..))"/>
        <!--切面 = 通知 + 切点-->
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>

</beans>
```
将AccountServiceImpl类上的@Transactional注解删除。
编写测试程序：
```java
@Test
public void testTransferXml(){
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring2.xml");
    AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
    try {
        accountService.transfer("act-001", "act-002", 10000);
        System.out.println("转账成功");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```
执行结果：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666510211960-60399e1d-ae1c-4e73-9593-3ce0086bf143.png#averageHue=%23f8f4f2&clientId=uae187c3b-e934-4&from=paste&height=266&id=ub662fbc9&originHeight=266&originWidth=1116&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63969&status=done&style=shadow&taskId=u31cf1aa3-8722-42ee-9b00-c3192b7f968&title=&width=1116)
数据库表中记录：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666510230350-5150f5ca-3812-40d6-8817-adc102516e7e.png#averageHue=%23f4f3f2&clientId=uae187c3b-e934-4&from=paste&height=150&id=ub644a306&originHeight=150&originWidth=356&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8520&status=done&style=shadow&taskId=ue28c57cd-8f46-4649-b48f-0a8b3c694b3&title=&width=356)
通过测试可以看到配置XML已经起作用了。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=GkDqr&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 十七、Spring6整合JUnit5
## 17.1 Spring对JUnit4的支持
准备工作：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>spring6-015-junit</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!--仓库-->
    <repositories>
        <!--spring里程碑版本的仓库-->
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <dependencies>
        <!--spring context依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--spring对junit的支持相关依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--junit4依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```
```java
package com.powernode.spring6.bean;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * @author 动力节点
 * @version 1.0
 * @className User
 * @since 1.0
 **/
@Component
public class User {

    @Value("张三")
    private String name;

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public User() {
    }

    public User(String name) {
        this.name = name;
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
    <context:component-scan base-package="com.powernode.spring6.bean"/>
</beans>
```
单元测试：
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

/**
 * @author 动力节点
 * @version 1.0
 * @className SpringJUnit4Test
 * @since 1.0
 **/
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring.xml")
public class SpringJUnit4Test {

    @Autowired
    private User user;

    @Test
    public void testUser(){
        System.out.println(user.getName());
    }
}

```
执行结果如下：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666602069724-ec6288cc-bb7b-417e-995e-8e1978ee6943.png#averageHue=%23f8f4f1&clientId=uf3bdf93c-900b-4&from=paste&height=240&id=u13070b41&originHeight=240&originWidth=559&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29892&status=done&style=shadow&taskId=u053c1ac2-b862-4efe-87f8-7e1e95bd741&title=&width=559)
Spring提供的方便主要是这几个注解：
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring.xml")
在单元测试类上使用这两个注解之后，在单元测试类中的属性上可以使用@Autowired。比较方便。

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=G0nL0&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 17.2 Spring对JUnit5的支持
引入JUnit5的依赖，Spring对JUnit支持的依赖还是：spring-test，如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>spring6-015-junit</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!--仓库-->
    <repositories>
        <!--spring里程碑版本的仓库-->
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <dependencies>
        <!--spring context依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--spring对junit的支持相关依赖-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <!--junit5依赖-->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>5.9.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```
```java
package com.powernode.spring6.test;

import com.powernode.spring6.bean.User;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit.jupiter.SpringExtension;


@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:spring.xml")
public class SpringJUnit5Test {

    @Autowired
    private User user;

    @Test
    public void testUser(){
        System.out.println(user.getName());
    }
}
```
在JUnit5当中，可以使用Spring提供的以下两个注解，标注到单元测试类上，这样在类当中就可以使用@Autowired注解了。
@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:spring.xml")
![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=ftBI4&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
# 十八、Spring6集成MyBatis3.5
## 18.1 实现步骤

- 第一步：准备数据库表
   - 使用t_act表（账户表）
- 第二步：IDEA中创建一个模块，并引入依赖
   - spring-context
   - spring-jdbc
   - mysql驱动
   - mybatis
   - mybatis-spring：**mybatis提供的与spring框架集成的依赖**
   - 德鲁伊连接池
   - junit
- 第三步：基于三层架构实现，所以提前创建好所有的包
   - com.powernode.bank.mapper
   - com.powernode.bank.service
   - com.powernode.bank.service.impl
   - com.powernode.bank.pojo
- 第四步：编写pojo
   - Account，属性私有化，提供公开的setter getter和toString。
- 第五步：编写mapper接口
   - AccountMapper接口，定义方法
- 第六步：编写mapper配置文件
   - 在配置文件中配置命名空间，以及每一个方法对应的sql。
- 第七步：编写service接口和service接口实现类
   - AccountService
   - AccountServiceImpl
- 第八步：编写jdbc.properties配置文件
   - 数据库连接池相关信息
- 第九步：编写mybatis-config.xml配置文件
   - 该文件可以没有，大部分的配置可以转移到spring配置文件中。
   - 如果遇到mybatis相关的系统级配置，还是需要这个文件。
- 第十步：编写spring.xml配置文件
   - 组件扫描
   - 引入外部的属性文件
   - 数据源
   - SqlSessionFactoryBean配置
      - 注入mybatis核心配置文件路径
      - 指定别名包
      - 注入数据源
   - Mapper扫描配置器
      - 指定扫描的包
   - 事务管理器DataSourceTransactionManager
      - 注入数据源
   - 启用事务注解
      - 注入事务管理器
- 第十一步：编写测试程序，并添加事务，进行测试

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=N8XM5&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 18.2 具体实现

- 第一步：准备数据库表

连接数据库的工具有很多，除了之前我们使用的navicat for mysql之外，也可以使用IDEA工具自带的DataBase插件。可以根据下图提示自行配置：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666659555476-977c1aec-6bcb-4b2b-a5d1-932a8b66cbac.png#averageHue=%23f7f5f4&clientId=u73ffb7ec-cc40-4&from=paste&height=228&id=u89ada0af&originHeight=228&originWidth=487&originalType=binary&ratio=1&rotation=0&showTitle=false&size=15896&status=done&style=shadow&taskId=uf45ae4a7-5989-4d31-b17a-cb777e6d77e&title=&width=487)
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666659459681-56e377a7-3b9e-4649-b29d-9da3c81fe46f.png#averageHue=%23f5f4f0&clientId=u73ffb7ec-cc40-4&from=paste&height=208&id=u6d2a9b6f&originHeight=208&originWidth=392&originalType=binary&ratio=1&rotation=0&showTitle=false&size=13524&status=done&style=shadow&taskId=u755ccb44-a8bc-4423-8691-2b23427f89d&title=&width=392)

- 第二步：IDEA中创建一个模块，并引入依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.powernode</groupId>
    <artifactId>spring6-016-sm</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <!--仓库-->
    <repositories>
        <!--spring里程碑版本的仓库-->
        <repository>
            <id>repository.spring.milestone</id>
            <name>Spring Milestone Repository</name>
            <url>https://repo.spring.io/milestone</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>6.0.0-M2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.30</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.11</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>2.0.7</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.13</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

</project>
```

- 第三步：基于三层架构实现，所以提前创建好所有的包

![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666660021872-5935b222-7e72-41d9-a9e1-c532ca29ef10.png#averageHue=%23fcfafa&clientId=u73ffb7ec-cc40-4&from=paste&height=151&id=u1dc2848e&originHeight=151&originWidth=237&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3561&status=done&style=shadow&taskId=ud1da6287-9713-4210-bc5c-7124e009c3d&title=&width=237)

- 第四步：编写pojo
```java
package com.powernode.bank.pojo;

/**
 * @author 动力节点
 * @version 1.0
 * @className Account
 * @since 1.0
 **/
public class Account {
    private String actno;
    private Double balance;

    @Override
    public String toString() {
        return "Account{" +
                "actno='" + actno + '\'' +
                ", balance=" + balance +
                '}';
    }

    public Account() {
    }

    public Account(String actno, Double balance) {
        this.actno = actno;
        this.balance = balance;
    }

    public String getActno() {
        return actno;
    }

    public void setActno(String actno) {
        this.actno = actno;
    }

    public Double getBalance() {
        return balance;
    }

    public void setBalance(Double balance) {
        this.balance = balance;
    }
}

```

- 第五步：编写mapper接口
```java
package com.powernode.bank.mapper;

import com.powernode.bank.pojo.Account;

import java.util.List;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountMapper
 * @since 1.0
 **/
public interface AccountMapper {

    /**
     * 保存账户
     * @param account
     * @return
     */
    int insert(Account account);

    /**
     * 根据账号删除账户
     * @param actno
     * @return
     */
    int deleteByActno(String actno);

    /**
     * 修改账户
     * @param account
     * @return
     */
    int update(Account account);

    /**
     * 根据账号查询账户
     * @param actno
     * @return
     */
    Account selectByActno(String actno);

    /**
     * 获取所有账户
     * @return
     */
    List<Account> selectAll();
}

```

- 第六步：编写mapper配置文件

一定要注意，按照下图提示创建这个目录。注意是斜杠不是点儿。在resources目录下新建。并且要和Mapper接口包对应上。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666660299388-b2e278e1-497d-4357-835c-ca95bfd87f0e.png#averageHue=%23f6f0ee&clientId=u73ffb7ec-cc40-4&from=paste&height=269&id=ua3500e97&originHeight=269&originWidth=536&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28659&status=done&style=shadow&taskId=u36c618c7-8cdc-4268-9d47-80daab48502&title=&width=536)
如果接口叫做AccountMapper，配置文件必须是AccountMapper.xml
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.powernode.bank.mapper.AccountMapper">
    <insert id="insert">
        insert into t_act values(#{actno}, #{balance})
    </insert>
    <delete id="deleteByActno">
        delete from t_act where actno = #{actno}
    </delete>
    <update id="update">
        update t_act set balance = #{balance} where actno = #{actno}
    </update>
    <select id="selectByActno" resultType="Account">
        select * from t_act where actno = #{actno}
    </select>
    <select id="selectAll" resultType="Account">
        select * from t_act
    </select>
</mapper>
```

- 第七步：编写service接口和service接口实现类

注意编写的service实现类纳入IoC容器管理：
```java
package com.powernode.bank.service;

import com.powernode.bank.pojo.Account;

import java.util.List;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountService
 * @since 1.0
 **/
public interface AccountService {
    /**
     * 开户
     * @param act
     * @return
     */
    int save(Account act);

    /**
     * 根据账号销户
     * @param actno
     * @return
     */
    int deleteByActno(String actno);

    /**
     * 修改账户
     * @param act
     * @return
     */
    int update(Account act);

    /**
     * 根据账号获取账户
     * @param actno
     * @return
     */
    Account getByActno(String actno);

    /**
     * 获取所有账户
     * @return
     */
    List<Account> getAll();

    /**
     * 转账
     * @param fromActno
     * @param toActno
     * @param money
     */
    void transfer(String fromActno, String toActno, double money);
}

```
```java
package com.powernode.bank.service.impl;

import com.powernode.bank.mapper.AccountMapper;
import com.powernode.bank.pojo.Account;
import com.powernode.bank.service.AccountService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author 动力节点
 * @version 1.0
 * @className AccountServiceImpl
 * @since 1.0
 **/
@Transactional
@Service("accountService")
public class AccountServiceImpl implements AccountService {

    @Autowired
    private AccountMapper accountMapper;

    @Override
    public int save(Account act) {
        return accountMapper.insert(act);
    }

    @Override
    public int deleteByActno(String actno) {
        return accountMapper.deleteByActno(actno);
    }

    @Override
    public int update(Account act) {
        return accountMapper.update(act);
    }

    @Override
    public Account getByActno(String actno) {
        return accountMapper.selectByActno(actno);
    }

    @Override
    public List<Account> getAll() {
        return accountMapper.selectAll();
    }

    @Override
    public void transfer(String fromActno, String toActno, double money) {
        Account fromAct = accountMapper.selectByActno(fromActno);
        if (fromAct.getBalance() < money) {
            throw new RuntimeException("余额不足");
        }
        Account toAct = accountMapper.selectByActno(toActno);
        fromAct.setBalance(fromAct.getBalance() - money);
        toAct.setBalance(toAct.getBalance() + money);
        int count = accountMapper.update(fromAct);
        count += accountMapper.update(toAct);
        if (count != 2) {
            throw new RuntimeException("转账失败");
        }
    }
}

```

- 第八步：编写jdbc.properties配置文件

放在类的根路径下
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/spring6
jdbc.username=root
jdbc.password=root
```

- 第九步：编写mybatis-config.xml配置文件

放在类的根路径下，只开启日志，其他配置到spring.xml中。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
</configuration>
```

- 第十步：编写spring.xml配置文件

**注意：当你在spring.xml文件中直接写标签内容时，IDEA会自动给你添加命名空间**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">
  
    <!--组件扫描-->
    <context:component-scan base-package="com.powernode.bank"/>
  
    <!--外部属性配置文件-->
    <context:property-placeholder location="jdbc.properties"/>

    <!--数据源-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>

    <!--SqlSessionFactoryBean-->
    <bean class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--mybatis核心配置文件路径-->
        <property name="configLocation" value="mybatis-config.xml"/>
        <!--注入数据源-->
        <property name="dataSource" ref="dataSource"/>
        <!--起别名-->
        <property name="typeAliasesPackage" value="com.powernode.bank.pojo"/>
    </bean>

    <!--Mapper扫描器-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.powernode.bank.mapper"/>
    </bean>

    <!--事务管理器-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--开启事务注解-->
    <tx:annotation-driven transaction-manager="txManager"/>

</beans>
```

- 第十一步：编写测试程序，并添加事务，进行测试
```java
package com.powernode.spring6.test;

import com.powernode.bank.service.AccountService;
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * @author 动力节点
 * @version 1.0
 * @className SMTest
 * @since 1.0
 **/
public class SMTest {

    @Test
    public void testSM(){
        ApplicationContext applicationContext = new ClassPathXmlApplicationContext("spring.xml");
        AccountService accountService = applicationContext.getBean("accountService", AccountService.class);
        try {
            accountService.transfer("act-001", "act-002", 10000.0);
            System.out.println("转账成功");
        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("转账失败");
        }
    }

}
```
**最后大家别忘了测试事务！！！！**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=zRUXk&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 18.3 Spring配置文件的import
spring配置文件有多个，并且可以在spring的核心配置文件中使用import进行引入，我们可以将组件扫描单独定义到一个配置文件中，如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--组件扫描-->
    <context:component-scan base-package="com.powernode.bank"/>

</beans>
```
然后在核心配置文件中引入：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!--引入其他的spring配置文件-->
    <import resource="common.xml"/>

</beans>
```
**注意：在实际开发中，service单独配置到一个文件中，dao单独配置到一个文件中，然后在核心配置文件中引入，养成好习惯。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=WIaTY&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)

# 十九、Spring中的八大模式
## 19.1 简单工厂模式
BeanFactory的getBean()方法，通过唯一标识来获取Bean对象。是典型的简单工厂模式（静态工厂模式）；
## 19.2 工厂方法模式
FactoryBean是典型的工厂方法模式。在配置文件中通过factory-method属性来指定工厂方法，该方法是一个实例方法。
## 19.3 单例模式
Spring用的是双重判断加锁的单例模式。请看下面代码，我们之前讲解Bean的循环依赖的时候见过：
![image.png](https://cdn.nlark.com/yuque/0/2022/png/21376908/1666663352271-4ba8d737-1e32-4f0e-b01a-aa305ad3abea.png#averageHue=%23fcfbfa&clientId=u73ffb7ec-cc40-4&from=paste&height=839&id=u96f97bc2&originHeight=839&originWidth=1190&originalType=binary&ratio=1&rotation=0&showTitle=false&size=137222&status=done&style=shadow&taskId=u92873aaa-765a-4341-8094-6fa96146e4a&title=&width=1190)

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=GQQOl&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 19.4 代理模式
Spring的AOP就是使用了动态代理实现的。
## 19.5 装饰器模式
JavaSE中的IO流是非常典型的装饰器模式。
Spring 中配置 DataSource 的时候，这些dataSource可能是各种不同类型的，比如不同的数据库：Oracle、SQL Server、MySQL等，也可能是不同的数据源：比如apache 提供的org.apache.commons.dbcp.BasicDataSource、spring提供的org.springframework.jndi.JndiObjectFactoryBean等。
这时，能否在尽可能少修改原有类代码下的情况下，做到动态切换不同的数据源？此时就可以用到装饰者模式。
Spring根据每次请求的不同，将dataSource属性设置成不同的数据源，以到达切换数据源的目的。
**Spring中类名中带有：Decorator和Wrapper单词的类，都是装饰器模式。**

![标头.jpg](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg#averageHue=%23f9f8f8&clientId=uc5a67c34-8a0d-4&from=paste&height=78&id=t7CTr&originHeight=78&originWidth=1400&originalType=binary&ratio=1&rotation=0&showTitle=false&size=23158&status=done&style=shadow&taskId=u98709943-fd0b-4e51-821c-a3fc0aef219&title=&width=1400)
## 19.6 观察者模式
定义对象间的一对多的关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。Spring中观察者模式一般用在listener的实现。
Spring中的事件编程模型就是观察者模式的实现。在Spring中定义了一个ApplicationListener接口，用来监听Application的事件，Application其实就是ApplicationContext，ApplicationContext内置了几个事件，其中比较容易理解的是：ContextRefreshedEvent、ContextStartedEvent、ContextStoppedEvent、ContextClosedEvent
## 19.7 策略模式
策略模式是行为性模式，调用不同的方法，适应行为的变化 ，强调父类的调用子类的特性 。
getHandler是HandlerMapping接口中的唯一方法，用于根据请求找到匹配的处理器。
比如我们自己写了AccountDao接口，然后这个接口下有不同的实现类：AccountDaoForMySQL，AccountDaoForOracle。对于service来说不需要关心底层具体的实现，只需要面向AccountDao接口调用，底层可以灵活切换实现，这就是策略模式。
## 19.8 模板方法模式
Spring中的JdbcTemplate类就是一个模板类。它就是一个模板方法设计模式的体现。在模板类的模板方法execute中编写核心算法，具体的实现步骤在子类中完成。
