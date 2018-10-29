## Java的注解

[TOC]

- 目录  
    - 1、注解的作用
    - 2、注解中最重要的三个类
    - 3、元注解
        - 3.1 @Target   
        - 3.2 @Retention
        - 3.3 @Documented
        - 3.4 @Inherited
    -4、注解的处理方式
        - 4.1、在运行时通过反射获取注解信息处理。
        - 4.2、使用apt来在编译时编译时期生成相应的代码。通过注解处理器（AnnotationProcessor）来处理。
    - 5、Java常用的注解
    - 6、Android 常用注解
        - 6.1 @IntDef 和 @StringDef
        - 6.2 、 @Nullable 和@NonNull
        - 6.3、 @FloateRange 和 @IntRange
        - 6.4、 @Size
        - 6.5、线程注解
        - 6.6 资源注解
    - 7、自定义注解的使用
        - 7.1 通过反射方式
        - 7.2 通过自定义注解处理器的方式。


### 1、注解的作用
>1. 生成文档。
>2. 代码分析，实现替代配置文件功能。
>3. 在编译时进行格式检查。如@Ovrride放在方法前，如果你这个方法不是覆盖了超类方法，则编译时就能检查出。

### 2、注解中最重要的三个类
Annotation,ElementType,RetentionPolicy 这三个类是注解中最重要的类。所有注解都基于这三个类。其中Annotation是接口，另外两个是枚举。

>- Annotation  :  注解的基础接口 。其它的注解都实现了这接口。
>
>- ElementType :注解的作用域枚举
>
>- RetationPolicy :注解的生命周期枚举

### 3、元注解
Java提供了四种元注解，专门**负责注解**其它的注解。

#### 3.1 @Target
    作用： 限定注解的作用域，可以用在什么地方。它的取值由EmelemtType定义。
 
 

ElementType| 说明
---|---
TYPE | 类、接口或枚举说明
FIELD | 字段说明（成员变量，包括枚举变量）
METHOD| 方法说明
PARAMETER| 参数说明
CONSUCTOR| 构造方法说明
LOCAL_VARIABLE | 局部变量
ANNONATION_TYPE| 注解类型声明
PACKAGE| 包声明
TYPE_PARAMETER|类型参数声明（1.8加入），表示这个注解可以用来标注类型参数
TYPE_USE | 类型使用声明（1.8加入）,用于标注各种类型名称，都可以进行注解


#### 3.2 @Retention
    作用： 表示在什么级别保存该注解类型信息，它的取值由RetentionPolicy定义

RetentionPolicy | 说明
---|---
SOURCE |只在源码中存在，编译成.class文件就不存在。如用于类型检查的注解
CLASS | 在源码和.class文件在都存在。像@Override，@Deprecated,@SupperWarinings都属于编译时注解
RUNTIME | 在运行阶段起作用，甚至会影响运行逻辑的注解。像Butterknife的@Bind，Dagger的@Inject这种


#### 3.3 @Documented
    作用： 表示注解会被包含在java api文档中

#### 3.4 @Inherited
    作用： 运行子类继承父类的注解。即它的所标注的Annotation具有继承性
 
 
 


### 4、注解的处理方式

#### 4.1、在运行时通过反射获取注解信息处理。
a） 先通过反射获得class对象，方法对象Method，字段对象 Field
    
```
   Class cls = Object.class;
   Method method = cls.getMethod("methodName");
   Field field = cls.getField("fieldName");
```

b） 根据需要通过Class对象，Method对象以及Field对象的方法去处理注解

-  Class 对象。对标注在类或接口上的注解处理
   

方法| 返回值|说明
---|---|--
isAnnotation()| boolean| 该对象是否为一个注解类型
isAnnotationPresent(Class<? extends Annotation> annotationClass)  | boolean|是否被指定的注解标注
getAnnotations()|Annotation[]|返回此类或接口上所有注解
getAnnotation(Class<A> annotationClass)| < A extends Annotation> |如果该类或接口被指定注解标注，则<br>返回该注解对象，否则返回null
getDeclaredAnnotations() | Annotation[] |   返回直接标注在此类或接口上的所有注释。


 - Method 对象。 对标注在方法上的注解的处理


方法 | 返回值| 说明
---|---|--
getAnnotation(Class<T> annotationClass)|< T extends Annotation > |如果该方法被指定注解标注，则返回<br>该注解对象，否则返回null
getDeclaredAnnotations() | Annotation[] |   返回直接存在于此方法上的所有注释。
	getParameterAnnotations() |Annotation[]|返回按声明顺序对此Method对象所表示方法的形参进行标注的注解的数组集合

- Field 对象。 对标注在字段（变量、枚举）上的注解的处理

方法 | 返回值| 说明
---|---|--
getAnnotation(Class<T> annotationClass)|< T extends Annotation > |如果该字段被指定注解标注，则返回该注解对象，否则返回null
getDeclaredAnnotations() | Annotation[] |   返回直接存在于此方法上的所有注释。







#### 4.2、使用apt来在编译时编译时期生成相应的代码。通过注解处理器（AnnotationProcessor）来处理。
>




### 5、Java常用的注解

>- @Override 只能标记方法，表示该方法覆盖父类的方法
>
>- @Deprecated 标记注解，其所标注的内容，表示不建议再被使用
>
>- @SupperWarinings 该注解做标注的内容产生的警告，编译器会对这些警告保持静默


### 6、Android 常用注解

#### 6.1 @IntDef  和  @StringDef
替代Java中枚举的注解，以@IntDef为例。定义和使用如下：

```java
 public static final int SUBWAY = 0;
 public static final int TAXI = 1;
 public static final int BUS = 2;


 @IntDef({SUBWAY, TAXI, BUS})
 @Retention(RetentionPolicy.SOURCE)
 public @interface Transport {

 }

 public  void  setTransport(@Transport int transport){
        
 }
```

#### 6.2 、 @Nullable 和@NonNull

可以修饰方法参数，返回值，属性变量

>@Nullable : 注解的元素可为空
>
>@NonNull ： 注解的元素不可为空。（为空时，不会报错，但会警告）


#### 6.3、  @FloateRange 和 @IntRange

用于限定范围的注解。


```
 public  void  setSize(@IntRange(from = 1,to = 10) int size){

 }
```


#### 6.4、 @Size

@Size用于限定长度

1. 用于限定字符串长度

```
public  void setText(@Size(4)String data){

}
```


2. 用于限定数组的长度

```
 public  void  setArray(@Size(2) int[] ints){
        
 }
```

3. 特殊的限定，如限定为2的倍数，最小长度，最大长度

```
 //数组长度只能为2的倍数
 public  void setArray(@Size(multiple = 2)int[] ints){
     
 }
 
 //数组的最小长度位2
 public  void setArray(@Size(min = 2)int[] ints){
     
 }
 
 //数组的最大长度为2
 public  void setArray(@Size(max = 2)int[] ints){
     
 }
 
 
```


#### 6.5、线程注解

- @MainThread ： 表示该方法只能在主线程调用。如果标记的是一个类，则这个类中的所有方法都应该是在主线程中被调用。
- @UiTread  ： ..该方法..只能..UI线程调用.......标记一个类......所有方法...Ui线程调用
- @WorkThread : ..该方法..只能..工作线程调用.......标记一个类......所有方法...工作线程调用
- @BInder ： ..该方法..只能..绑定线程调用.......标记一个类......所有方法...绑定线程调用

#### 6.6 资源注解

- @IntegerRes : R.integer类型资源
- @AnimatorRes : R.animator 类型资源
- @AnimRes :  R.anim 类型资源
- @ArrayRes ：R.array类型资源
- @ColorRes ： R.color类型资源
 。。。等等


#### 6.6 、 @Keep


标记的指定代码在混淆时不会被混淆





### 7、自定义注解的使用

#### 7.1 通过反射方式
1、 首先有以下三个注解：
 
```
    
    @Target(ElementType.TYPE) //用于注解类和接口
    @Retention(RetentionPolicy.RUNTIME) // 运行时注解仍然存在
    public @interface INTRODUCE {

        Country from() default Country.CHINA;

        String job() default "";

        enum Country {

            CHINA("中国"),
            AMMERICAN("美国"),
            ENGLAND("英国");

            public String country;


            Country(String country) {
                this.country = country;
            }
        }

    }
```



```
    @Target(ElementType.FIELD) //用于注解字段
    @Retention(RetentionPolicy.RUNTIME)
    public @interface AGE {
        int value() default 18;
    }
```


```
    @Target(ElementType.METHOD) //用于注解方法
    @Retention(RetentionPolicy.RUNTIME)
    public @interface EDUCATION {
        String education() default "";
    }
```

2、分别标注在Person类的 类，字段和方法上


```
@INTRODUCE(from = INTRODUCE.Country.CHINA, job = "程序员")
public class Person {

    @AGE(18)
    public int age;



    @EDUCATION(education = "本科")
    public void education() {

    }
    
}
```

3、写一个注解的处理器


```
public class PersonAnnotationUtil {


    public static void getPersonmInfo(Class<Person> clz) {


        try {
            Field ageField = clz.getField("age");
            Method educationMethod = clz.getMethod("education");

            INTRODUCE introduceAnnotation = clz.getAnnotation(INTRODUCE.class);
            String country = introduceAnnotation.from().country;
            String job = introduceAnnotation.job();

            AGE ageFieldAnnotation = ageField.getAnnotation(AGE.class);
            int age = ageFieldAnnotation.value();

            if (age <= 0 || age > 150)
                throw new Error("age'range is 0-150");

            EDUCATION educationAnnotation = educationMethod.getAnnotation(EDUCATION.class);
            String education = educationAnnotation.education();

            System.out.println("from ：" + country + "， job ： " + job + ", age : " + age + " , education : " + education);//from ：中国， job ：程序员, age : 18 , education : 本科

        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }


    }


}
```


#### 7.2 通过自定义注解处理器的方式。
详细看： [26_Java自定义注解处理器](https://github.com/Zeng-Ke/DailyLearning/blob/master/2018/10/26_Java自定义注解处理器.md)
  
  

---

### 完！！！

### 参考：
- http://hannesdorfmann.com/annotation-processing/annotationprocessing101
- https://race604.com/annotation-processing/






