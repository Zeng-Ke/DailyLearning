## Java的反射

[TOC]


### 1、类加载器
当Java程序要使用这个类时，如果该类还没有被加载到内存中，则系统会通过加载，链接，初始化这三步来实现对这个类的初始化。
类的加载就是指将class文件读入内存中，并为之创建一个Class对象。任何类被使用时系统都会创建一个Class对象。  
类加载器的职责就是把.class文件加载到内存中，并产生Class对象。  
类加载器的组成：  
- Bootstrap ClassLoader 根类加载器（C代码完成） ： 负责加载java运行的核心类。
- Extension ClassLoader 扩展加载器（C代码完成）： 负责加载程序中的扩展应用。
- System ClassLoader 系统加载器（Java代码完成） ：负责加载我么写的类。



### 2、反射的定义
java的反射机制是在**运行状态中**，对于任意一个类，都能知道这个类的所有属性和方法，对于任何一个对象，都能够调用它的任何一个方法和属性，这样**动态获取类的信息**以及**动态调用对象方法**的能力。  
简单的来说，反射就是解剖一个类，然后获取这个类的属性和方法，前提是要获取这个类的Class对象。  
在Java中给我们提供了一下几个类供描述编译后的各种对象。

类 | 描述
---|---
java.lang.Class | 描述编译后的class文件的对象
java.lang.reflect.Constructor|用于描述构造方法
java.lang.reflect.Field|描述字段（成员变量）
java.lang.reflect.Method| 描述成员方法





### 3、如何使用反射
> 1. 使用Class对象获取出被解剖的这个类的class对象
> 2. 使用Class类方法，获取出类中的所有成员
> 3. 将成员获取出来后，交给对应类，对应类的方法运行成员


### 4、获取类的Class对象有三种方式 ：
> 1. 通过对象的getClass获取。如 Class clz = "hello".getClass();
>   
> 2. 使用类的静态属性获取。 如 Class clz = String.class；
>
> 3. 使用Class类的静态方法获取。如 Class clz = Class.forName("java.lang.String")；
>



### 5、通过反射实例化对象
Constructor 提供了有关类的构造方法的信息，以及对它的动态访问的能力。

可以通过class提供的方法获取Constructor对象，具体如下：


方法返回值 | 方法名称|方法说明
---|---|--
Constructor<T> | getConstructor(Class<?>... parameterTypes)|返回指定参数类型、具有public访问权限的<br>构造函数Constructor对象
Constructor<?>[] |getConstructors()| 返回所有具有public访问权限的构造函数的<br>Constructor对象数组
Constructor<T>|getDeclaredConstructor(Class<?>... parameterTypes) |返回指定参数类型、所有声明（包括private）<br>构造函数Constructor对象
Constructor<?>[] |getDeclaredConstructor()|返回所有声明的（包括private构造函数<br>Constructor对象





对于以下Person对象有三种构造方法：

```java
public class Person {

        // public的空参数构造方法
        public Person() {
        }

        // 有参构造函数方法
        public Person(String name, int age) {
        }

        // private 有参构造函数
        private Person(String name) {
        }
        
    }

```



#### 5.1、 通过获取空参数构造，必须是public


```java
Class clz = Person.class;

Person person = (Person) clz.newInstance();
      

```

####  5.2、  通过无参数构造方法

```
Class clz = Person.class;
 
Constructor constructor = clz.getConstructor();
 Person person = (Persion)constructor.newInstance();
    
```

#### 5.3、 通过有参构造方法


```
 Class clz = Person.class;

 Constructor constructor = clz.getConstructor(String.class,int.class);
  Person person = (Persion)constructor1.newInstance("张三",18);
```

#### 5.4、 通过私有构造方法

```
 Class clz = Person.class;

 Constructor constructor = clz.getConstructor(String.class);
 constructor.setAccessible(true); //暴力访问
 Person person = (Persion)constructor.newInstance("张三");
```


### 6、通过反射获取类的成员变量



方法返回值| 方法名称|方法说明
---|---|--
Field | getDeclaredField(String name)|获取指定名称的（包含private修饰）字段，不包含继承的字段
Field[] | getDeclaredFields()|获取Class对象所表示的类或接口的所有（包含private修饰<br>的）字段，不包括继承的字段
Field|getField(String name)| 获取指定name名称，具有public修饰的字段，包含继承字段
Field[]|getField()| 获取修饰符为public的字段，包括继承字段



对于以下Person类 ：

```
public class Person {

    private String name = "李四";  // 私有 变量
    public int age = 20; // public 变量

    // public的空参数构造方法
    public Person() {
        System.out.println("Person类无参数构造");
    }

}
```

#### 6.1 、getDeclaredField(String name)


```
Person person = (Person) clz.newInstance();
Field field = clz.getDeclaredField("name");
field.setAccessible(true);
field.set(person, "张三");  

 System.out.println(field); // public java.lang.String com.zk.mmkvdemo.Person.name
 System.out.println(person.toString());  //Person  name =  张三  ;  age =  20

```

#### 6.2 、 getDeclareFields()


```
 Person person = (Person) clz.newInstance();
 Field[] fields = clz.getDeclaredFields();
 for (Field field : fields) {
    System.out.println(field);
}

输出：
-- private int com.zk.mmkvdemo.Person.age
-- public java.lang.String com.zk.mmkvdemo.Person.name

```

#### 6.3 、 getField(String name)


```
 Person person = (Person) clz.newInstance();
 Field field = clz.getField("age");
 field.set(person, 10);

 System.out.println(field); //public int com.zk.mmkvdemo.Person.age
 System.out.println(person.toString());//Person  name =  李四  ;  age =  10
 
 
```


#### 6.4 、 getFields()


```
 Person person = (Person) clz.newInstance();
 Field[] fields = clz.getFields();
 for (Field field : fields) {
     System.out.println(field);
 }
 
 输出：
-- private int com.zk.mmkvdemo.Person.age   
            
            
```



### 7.通过反射获取类的成员方法

Method提供了有关类和接口的单个方法的信息，以及对它的动态访问的能力。


方法返回值 | 方法名称|方法说明
---|---|--
Method | getDeclaredMethod(String name, Class<?>... parameterTypes) |返回一个指定参数的Method对象，该对象反映此class对象所表示<br>的类或接口的指定已声明方法
Method[]| getDeclaredMethod()| 返回Method对象的数组，这些对象反映此class对象所表示的类或接口<br>的所有方法，包括公共，保护，默认(包)访问和私有方法，但不包括继承方法。
Method|getMethod(String name, Class<?>... parameterTypes) |返回一个 Method 对象，它反映此 Class 对象所表示的类或接口<br>的指定public修饰成员方法。
Method[]|getMethods()|返回一个包含某些 Method 对象的数组，这些对象反映此 Class<br> 对象所表示的类或接口（包括那些由该类或接口声明的以及从<br>超类和超接口继承的那些的类或接口）的公共 member 方法。

对于以下Person类 ：


```
public class Person {

  
    // public的空参数构造方法
    public Person() {
        System.out.println("Person类无参数构造");
    }

    public void show() {
        System.out.println("show 空参数");
    }

    public void show(int a) {
        System.out.println("show   a:" + a);
    }

    private void show(String s) {
        System.out.println("show   s:" + s);
    }


}
```

#### 7.1  getDeclaredMethod(String name, Class<?>... parameterTypes)


```
 Person person = (Person) clz.newInstance();
 Method method = clz.getDeclaredMethod("show",String.class);
 method.setAccessible(true);
 method.invoke(person,"张三"); //  输出 ： show   name:============张三
 
 
```

#### 7.2  getDeclaredMethods()


```
 Person person = (Person) clz.newInstance();
 Method[] methods = clz.getDeclaredMethods();
 for (Method method : methods) {
    System.out.println(method);
}

输出：
--public static java.lang.Object com.zk.mmkvdemo.Person.access$super(com.zk.mmkvdemo.Person,java.lang.String,java.lang.Object[])
--private void com.zk.mmkvdemo.Person.show(java.lang.String)
--public void com.zk.mmkvdemo.Person.show()
--public void com.zk.mmkvdemo.Person.show(java.lang.String,int)
--public java.lang.String com.zk.mmkvdemo.Person.toString()


```

#### 7.3 、 getMethod(String name, Class... parameterTypes);


```
Person person = (Person) clz.newInstance();
Method method = clz.getMethod("show",String.class,int.class);
method.invoke(person,"张三",10);  //输出：  show   name:张三  ; age + 10
```


#### 7.4 getMethods()


```

Person person = (Person) clz.newInstance();
Method[] methods = clz.getMethods();
for (Method method : methods) {
    System.out.println(method);
}

输出：
-- public static java.lang.Object com.zk.mmkvdemo.Person.access$super(com.zk.mmkvdemo.Person,java.lang.String,java.lang.Object[])
--public boolean java.lang.Object.equals(java.lang.Object)
--public final java.lang.Class java.lang.Object.getClass()
--public int java.lang.Object.hashCode()
--public final native void java.lang.Object.notify()
--public final native void java.lang.Object.notifyAll()
--public void com.zk.mmkvdemo.Person.show()
--public void com.zk.mmkvdemo.Person.show(java.lang.String,int)
--public java.lang.String com.zk.mmkvdemo.Person.toString()
--public final native void java.lang.Object.wait() throws java.lang.InterruptedException
--public final void java.lang.Object.wait(long) throws java.lang.InterruptedException
--public final native void java.lang.Object.wait(long,int) throws java.lang.InterruptedException  



```


### 反射优缺点

>优点： 用于框架柱动态配置，插件化开发中持续集成，应用扩展  
>
>缺点： 性能低，可读性差，只能在运行时报错


### 完！




### 参考

https://blog.csdn.net/android_ku_ku/article/details/52336734
https://juejin.im/post/5b6cfdea5188251ace75f280








