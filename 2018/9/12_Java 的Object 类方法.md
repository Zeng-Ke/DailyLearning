### Java Object类内部的方法有以下几个：
* 构造函数
* hasCode()和equal()函数用来判断对象是否相等
* wait(),wait(long),wait(long,int)和notify(),notifyAll()
* toString(),getClass()
* clone()用在克隆对象
* finalize()用在垃圾回收




### 函数说明

#### 一、hashCode() 和  equal()

 * equal() 用于判断两个对象是否相同
 * hashCode() 用于获取对象的hash值
 * hashCode()相等不代表两个对象相等， 因为hasCode的值为int类型，最大为2^32，当超过2^32个对象时，就一定会产生hash冲突。 
 * equal()返回true证明两个对象一定相等。因为原生equal方法是通过两个对象的内存地址去判断是否相等。
 
**例子：** 定义一个Person类。有名字，和年龄。当两个人出现名字和年龄相同时，我们判断为同一个人。这时我们就要从写equal()方法，改变判断。


```java
 public class Person {

        public String name;
        public int age;

        public Person(String name, int age) {
            this.name = name;
            this.age = age;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            Person other = (Person) obj;
            if (age != other.age)
                return false;
            if (name == null) {
                if (other.name != null)
                    return false;
            } else if (!name.equals(other.name))
                return false;
            return true;
        }
    }


```

那对于上面的Person：  
Person p1 = new Person("张三"，10)   
Person p2 = new Person("张三"，10)

p1.equal(p2) = true。那p1 == p2 吗？ 答案： 否。
因为上面equal()返回true有两种可能： 1、内存地址一样 ；2、内容一样 。很明显上面的是例子是内容一样，内存地址并不一样。所以== 为false。

那判断两个对象是否相等只需重新equal()就可以了吗？ 答案： 否  
一般还要从写hashCode()方法。hashCode的作用使用来获取哈斯码，用于确定对象在哈斯表中的作用。就比如上面的例子，就假如要把p1和p2放到hashSet中，因为这两个判断上为同一个人，理想是因为HashSet的去重特性所以添加之后只存在一个。结果发现HashSet中它们两个都存在。那问题在哪呢？  
  
  
HashSet添加元素时如果添加元素的hashCode相等并且对象equals为true或对象==，则认为同一个元素，不添加到元素中。而上面通过hasCode()方法可以获得p1。hashCode() = 105850744,p2.hashCode() = 210406993。显然两种hashCode不相等。

> 引申出两个面试题：  

    1、两个对象，如果a.equals(b) == true,那么a和b相等？  
    
        相等。但地址不一定相等

    2、两个对象，如果hashCode一样，那么两个对象是否相等？
        不一定相等，判断两个对象是否相等，需要判断equal是否为true
        
        
  
  
#### 二. wait(), notify(),notifyAll()
wait(),notify(),notifyAll(),sychronize要搭配使用，用于多线程的同步。  
* wai的作用  
  * 1、wait()总是在一个循环中被调用，被挂起等待另一个条件的成立。wait()调用会等到其他线程调用notify()或者notifyAll()时才会返回。  
  * 2、当一个线程在执行sychronized的方法内部，调用了wait()方法后，该线程会释放该对象的锁。然后该线程会被添加到该对象的等待队列(waiting queue)中，就会一直处于闲置状态，不会被调度执行。需要注意wait()方法会强迫线程先进行释放所操作，所以在调用wait()时，该线程已获得锁，否则会抛出异常。
  
* notify(),notifyAll()  
    * 1、当一个线程调用一个对象的notify()方法时，调度器会从所有处于该对象等待队列的线程中取出任意一个线程，将其添加到入口队列中，然后在入口队列中的多个线程就会竞争对象的锁，得到锁的线程就可以持续执行。如果等待队列没有线程，notify()方法不会产生任何作用。
    * 2、notifyAll()和notify()的区别是把等待队列的所有线程添加到入口队列中。
    * 3、notifyAll()比notify()更常用，因为notify()不能指定唤醒哪一个线程。  


正确用法：  


```java
// 线程 A 的代码
synchronized(obj_A)
{
    while(!condition){ 
        obj_A.wait(); // 释放锁，并把当前线程添加到等待队列中
    }
    // do something 
}

```

```java
// 线程 B 的代码
synchronized(obj_A)
{
    if(!condition){ 
        // do something ...
        condition = true;
        obj_A.notify(); //唤醒线程A，把线程从等待队列移到入口对口中
    }
}

```
参考：  
[为什么wait()和notify()需要搭配synchonized关键字使用]( https://blog.csdn.net/lengxiao1993/article/details/52296220)


#### 三、 clone()方法

在工作种，经常会遇到有一个对象A，并且某一时刻A里面包含了一些有效属性。这是我们有可能需要一个个A完全一样的对象B，并且对对象B的改变不会影响到A。也就是说A和B是两个完全独立的对象，但B的初始值完全是由A决定的。 这是就要用到 **clone()** 方法。
colne()拷贝的使用注意点：  
*  拷贝分为 **浅拷贝**和 **深拷贝**。
*  覆盖clone()方法需要实现cloneable接口，否则会抛CloneNotSupportException异常。


##### 浅拷贝

如以下Person类：
    
```java  
    public class Person implements Cloneable {

    public String name;
    public int age;
     public School school;

    public Person(String name, int age, School school) {
        this.name = name;
        this.age = age;
    
    }

    public Object clone() {
        Object cloneObject = null;
        try {
            cloneObject = super.clone();//需要实现Cloneable接口，否则报java.lang.CloneNotSupprtedException: Class com.zk.demo.demo.ThirdActivity$Person doesn't implement Cloneable
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return cloneObject;
    }
        
      public static class School  {
        public String name;

        public School(String name) {
            this.name = name;
        }
    }


}
```

运行：  

```
   Person p1 = new Person("张三", 10,new Person.School("一中"));
   Person p2 = (Person) p1.clone();
   Log.d("====", p1 == p2 ? "true" : "false" ); //false
   Log.d("====", p1.hasCode()); //245778498
   Log.d("====", p2.hasCode()); //199037011
```
可以看到上面已经通过p1对象拷贝了一个p2对象。并且p1对象和p2对象不是相同引用。这就是浅拷贝。


##### 深拷贝
    
对于上面的例子我们再输出 
    
```
   Log.d("====", p1.school == p2.school ? "true" : "false" ); //true
   Log.d("====", p1.school.hasCode()); //46450707
   Log.d("====", p2.school.hasCode()); //46450707
```

发现虽然 p1和p2虽然不是同一个对象，但它们的属性school都是指向同一个对象。显示当我们队p2的school进行修改时，就有可能会影响到p1。这是就要用到深拷贝了。

深拷贝有两种方法。  
* 方法1 ： 对p1的非基本类型属性都进行递归拷贝。就譬如对school也要进行浅拷贝操作。而加入School里面还有非基本类型，还要继续递归下去。  
 
```
public static class School implements Cloneable {
        public String name;

        public School(String name) {
            this.name = name;
        }
       
        @Override
        public Object clone() {
            Object cloneObject = null;
            try {
                cloneObject = super.clone();//需要实现Cloneable接口，否则报java.lang.CloneNotSupportedException: Class com.zk.demo.demo
                // .ThirdActivity$Person doesn't implement Cloneable
            } catch (CloneNotSupportedException e) {
                e.printStackTrace();
            }
            return cloneObject;
        }
    }
```

运行：

```
    Person p1 = new Person("张三", 10, new Person.School("一中"));
    Person p2 = (Person) p1.clone();
    p2.school = (Person.School) p1.school.clone();
```

输出：
```
   Log.d("====", p1 == p2 ? "true" : "false" ); //false
   Log.d("====", p1.hasCode()); //210406993
   Log.d("====", p2.hasCode()); //33650358
   Log.d("====", p1.school == p2.school ? "true" : "false" ); //false
   Log.d("====", p1.school.hasCode()); //51069732
   Log.d("====", p2.school.hasCode()); //59715511
```



* 方法2： 通过流操作进行深拷贝，因为涉及到流操作，也需要实现Serializable序列化接口


```
public class Person implements Cloneable, Serializable {

    public String name;
    public int age;
    public School school;

    public Person(String name, int age, School school) {
        this.name = name;
        this.age = age;
        this.school = school;
    }

     
    //对象克隆
    @Override
    public Object clone() {
        Object object = null;
        try {
            //写
            ByteArrayOutputStream bo = new ByteArrayOutputStream();
            ObjectOutputStream oo = new ObjectOutputStream(bo);
            oo.writeObject(Person.this);

            //读
            ByteArrayInputStream bi = new ByteArrayInputStream(bo.toByteArray());
            ObjectInputStream oi = new ObjectInputStream(bi);
            object = oi.readObject();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return object;
    }

    public static class School implements Serializable {
        public String name;

        public School(String name) {
            this.name = name;
        }

    }

}
```

运行：

```
    Person p1 = new Person("张三", 10, new Person.School("一中"));
    Person p2 = (Person) p1.clone();
```

输出：
```
   Log.d("====", p1 == p2 ? "true" : "false" ); //false
   Log.d("====", p1.hasCode()); //181708430
   Log.d("====", p2.hasCode()); //190299476
   Log.d("====", p1.school == p2.school ? "true" : "false" ); //false
   Log.d("====", p1.school.hasCode()); //142122394
   Log.d("====", p2.school.hasCode()); //154459668
```

上面便是两种深拷贝的实现。

参考：  
[关于object类的clone方法浅克隆与深度克隆](https://blog.csdn.net/JQ_AK47/article/details/52675898)


####  四、finalize()


finalize用于垃圾回收的情景中。

finalize()的作用：用于在GC发生前先调用，去回收JNI调用中申请的特殊内存。下次GC发生时候保证GC后所有该对象的内存都没释放了。  


垃圾回收： 
* Java的垃圾回收器只会释放由我们new出来的内存堆块，那些不是由new出来的“特殊内存”，垃圾回收期是不会管理的。  
* 所谓的特殊内存是指通过JNI用C/C++向系统申请的内存，这些内存如果不手动去清除就会一直占据在内存中。  


finalize():
* 由上，Java的对象并不一定会被全部垃圾回收，当你不需要改对象的时候，你需要手动去处理那些“特殊内存”，java没有析构函数，所以提供了finalize()方法让我们执行清理操作。
* 当系统进行GC时会先调用finalize方法，然后再下次回收对象的内存。因为native申请的内存，GC没有办法回收，所以finalize被用来做垃圾回收前的重要操作：释放特殊内存。
* 所以finalzie()一般使用在使用了JNI情景下，需要在finalize中调用native方法释放特殊内存，一般情况不使用finalize()。
