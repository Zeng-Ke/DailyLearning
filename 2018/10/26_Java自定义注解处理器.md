## Java自定义注解处理器

[TOC]

### 目录：

- 1、基本概念
- 2、虚处理器AbstractProcessor
- 3、注册你的处理器
    - 方法一： 手动注册
    - 方法二： 利用@AutoService(Processor.class)自动注册
- 4、涉及到的类
- 5、例子：
    - 5.1、定义注解：
    - 5.2、 把@Factory添加到需要注解的类上
    - 5.3 FactoryAnnotationClass 这个是 保存每个被@Factory注解的类的信息 ：
    - 5.4 FactoryItemTypeContainer 保存所有 FactoryAnnotationClass 的，并且根据FactoryAnnotationClass用Filer去生成Java代码。
    - 5.4 FactoryProcessor ： 处理@Factory注解的自定义注解处理器。注意这个需通过独立的lib或jar进行引入。


### 1、基本概念
这里讨论的不是运行时通过反射机制处理的注解，而是在编译时处理的注解。  
注解处理器是Javac的一个工具，它用来在编译时扫描和处理注解，你可以自定义注解，并注册相应的注解处理器。   
一个注解的注解处理器，以Java代码作为输入，生成文件（通常是.java）文件。这些生成的java文件，会和其他普通的手写编写的Java源代码一样被javac编译。


### 2、虚处理器AbstractProcessor

每一个自定义的注解处理器都要继承这个AbstrctProcessor。


```
public class MyProcessor extends AbstractProcessor {

    @Override
    public synchronized void init(ProcessingEnvironment env){ }

    @Override
    public boolean process(Set<? extends TypeElement> annoations, RoundEnvironment env) { }

    @Override
    public Set<String> getSupportedAnnotationTypes() { }

    @Override
    public SourceVersion getSupportedSourceVersion() { }

}
```

- init(ProcessingEnvironment env)：  
  ：这个方法会被注解处理工具调用，并提供ProcessingEnvironment参数。ProcessingEnvironment会提供很多有用的工具类Elements，Type和Filer，后面做详细介绍。  

- getSupportedAnnotationTypes() ：  
  这里定义你的主机处理器注册到哪些注解上。它的返回值是一个字符串集合，包含本处理器想要处理的注解类型的合法全称。

- getSupportedSourceVersion()：  
 用来指定你使用的Java版本。一般是SourceVersion.lastestSupported()。
  
- process(Set<? extends TypeElement> annotations, RoundEnvironment env):  
  这相当于每个处理器的主函数main()。 在这里写你的扫描，判断和处理注解的代码，以及生产Java文件。输入参数RoundEnvirment可以让你查询出包含特定注解的被注解元素。




### 3、注册你的处理器
#### 方法一： 手动注册

怎样将处理器MyProcessor注册到javac中。你必须提供一个.jar文件。就像其它的.jar文件一样，你打包你的注解处理器到此文件中。并且在你的jar中，你需要打包一个特定的文件javax.annotation.processing.Processor到META-INF/services路径下。所以你的.jar文件看起来就像下面这样：

- MyProcessor.jar
    - com
        - example
           - MyProcessor.class
    - META-INF        
        - services
           - javax.annotation.processing.Processor


这个javax.annotation.processing.Processor的内容是，自定义注解处理器的合法的全名列表，每一个元素换行分割：

```
com.example.MyProcessor  
com.me.OtherProcessor  
net.blabla.SpecialProcessor 
```


把MyProcessor.jar放到你的buildPath中，javac会自动检查和读取javax.annotation.processing.Processor中的内容，并且注册MyProcessor作为注解处理器。


#### 方法二： 利用@AutoService(Processor.class)自动注册

AutoSerive注解处理器是Google开发的，用来生成META-INF/services/javax.annotation.processing.Processor文件的。 它的使用就是只需要在自定义处理器上方加上@AutoService(Processor.class)即可。

```
@AutoService(Processor.class)
public class MyProcessor extends AbstractProcessor {

   ...
   ...

}
```


### 4、涉及到的类

Element的种类有以下几种：

```
package com.example;    // PackageElement

public class Foo {        // TypeElement

    private int a;      // VariableElement
    private Foo other;  // VariableElement

    public Foo () {}    // ExecuteableElement

    public void setA (  // ExecuteableElement
                     int newA   // VariableElement
                     ) {}
}
```

TypeElement ： 表示一个类或接口程序元素，提供对类型及成员的信息的访问。

方法 | 返回值|说明
---|---|--
getEnclosedElements() | List<? extends Element> | 返回在这个类或接口中直接声明的字段、方法、构造函数和成员类型
getEnclosingElement() | Element | 返回一个所在包的元素
getSimpleName() |Name|返回此类型元素的简单名称。
getSuperclass() | TypeMirror|返回此类型元素的直接父类。
getTypeParameters() | List<? extends TypeParameterElement> | 按照声明顺序返回此类型元素的形式类型参数。






在init()中，我们获得了如下引用:
> - Elements : 一个用来处理Element的工具类  

方法 | 返回值|说明
---|---|--
getAllAnnotationMirrors(Element e)| List<? extends AnnotationMirror> |返回目前元素上所有注释
    getAllMembers(TypeElement type) | List<? extends Element> | 返回一个类型元素的所有成员，无论是否直接继承或声明
getPackageOf(Element type)|PackageElement | 返回元素的包
getTypeElement(CharSequence name) | TypeElement | 返回规定名称的类型元素
...|...|...


> - Types : 一个用来处理TypeMirror的工具类

方法 | 返回值|说明
---|---|--
asElement(TypeMirror t) | Element | 返回对应于类型的元素。
directSupertypes(TypeMirror t)| List<? extends TypeMirror> |返回一个类型的直接超类型。
...|...|...


> - Filer ： 正如这个名字所示，使用Filer可以创建文件。




### 5、例子：

现在有以下三个汽车类。


```
public class Audi{

    public void drive() {
        Log.d("===============","奥迪上路");
    }
}

public class Benz{

    public void drive() {
        Log.d("===============","奔驰上路");
    }
}


public class BMW{

    public void drive() {
        Log.d("===============","宝马上路");
    }
}

```

如果我们不做任何处理的话，就是需要用到哪个类就实例化哪个。

```
Audi audi = new Audi();
audi.drive();
```

而作为Java程序员，我们当然秉承Java的封装思想。把三个类抽象出一个Car接口，并提供一个drive()方法。然后创建一个工厂类，利用枚举或者常量参数去控制实例化。

如：
```
public class CarFactory {

  public Car create(String id) {
    if (id == null ||"".equals(id)) {
       throw  new IllegalArgumentException("id is Null");
    }
    if ("Benz".equals(id)) {
      return new Benz();
    }
    if ("Audi".equals(id)) {
      return new Audi();
    }
    if ("BMW".equals(id)) {
      return new BMW();
    }
    throw new IllegalArgumentException("Unknown id = " + id);
  }
}
```
但是这样我们每次增加一个Car类时，我们又需要改动这个Factory类。这明显也不符合Java的开闭原则。

那再高级点利用反射机制去实例化：

```
public Car create(Class<? extends Car> clz) {
    try {
        return clz.newInstance();
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
    return null;
}
```

但这时又会有人说：反射，影响性能啊。

那能不能有更好的方法呢？

有滴。那就是通过自定义注解处理器去帮我们去写if else的判断代码啦。

先说下思路：
定义一个@Factory注解，注解里有一个id属性。并且把这个注解注解到这三个Car类上。
然后我们自定义注解处理器，去做注解处理：  
获取所有被@Factory注解的类，并且拿到他们@Factory的id，根据这个id去判断实例化对应的类。这样我们就可以通过编译器去生成我们需要的CarFactory，而每当我们继续添加一个Car类时，只需要重新编译即可， 而不需要我们去修改CarFactory类。

那我们把目光再放远点。现在只是汽车Car类型的。那到时假如还有其它的Flower，Tree等等的需求呐？难道我们再去写一个对应的@Factory去实现吗。那我们能不能就用一个@Factory去搞定呐？    

答案是可以的。这是我们在@Factory上再添加一个type()属性即可。

#### 5.1、定义注解：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface Factory {

    String id();  // 通过该id去判断实例化需要的对象。可采用类名，如 Audi

    Class type(); // 通过该class去生成对应的Factory 。如该type()返回Car.class类型，然后生成CarFactory
    
}
```

#### 5.2、 把@Factory添加到需要注解的类上


```
@Factory(
        id = "Audi",
        type = Car.class
)
public class Audi implements Car {

    @Override
    public void drive() {
        Log.d("===============","奥迪上路");
    }
}
-----------------------------------------------
@Factory(
        id = "Benz",
        type = Car.class
)
public class Benz implements Car {

    @Override
    public void drive() {
        Log.d("===============","奔驰上路");
    }
}
------------------------------------------------

@Factory(
        id = "BMW",
        type = Car.class
)
public class BMW implements Car {

    @Override
    public void drive() {
        Log.d("===============","宝马上路");
    }
}

```


#### 5.3 FactoryAnnotationClass 这个是 保存每个被@Factory注解的类的信息  ：


```
public class FactoryAnnotationClass {

    private final TypeElement mAnnotatedClassElement;
    private String mAnnotateTypeName;
    private String mAnnotatedTypeSimpleName;
    private final String mId;

    public FactoryAnnotationClass(TypeElement typeElement) throws IllegalArgumentException {

        mAnnotatedClassElement = typeElement;
        Factory factory = mAnnotatedClassElement.getAnnotation(Factory.class);
        mId = factory.id(); //获取 {@link Factory#id()}中指定的id()
        if (mId == null || "".equals(mId)) {
            throw new IllegalArgumentException(/*String.format("id() in %1$s for class %2$s is Null,that's not allow", Factory.class
                    .getSimpleName(), mAnnotatedClassElement.getQualifiedName().toString())*/"id()  is Null,that's " +
                    "not allow");
        }

        try {
            /**
             * 类已被编译的情况，如果第三方.jar包包含已编译的被@Factory注解的.class文件。这时可以通过try{}中代码块获取
             */
            Class<?> clz = factory.type();
            mAnnotateTypeName = clz.getCanonicalName();// 获取Factory#type()中指定的type()的类型合法全名
            mAnnotatedTypeSimpleName = clz.getSimpleName(); // 获取Factory#type()中指定的type()的类型简单名字

        } catch (MirroredTypeException mte) {

            /**
             * 这种情况是我们尝试编译被@Factory注解的源代码。这种情况会直接抛出MirroredTypeException。幸运的是我们可以通过该Exception去获取所需变量。
             */
            DeclaredType declaredType = (DeclaredType) mte.getTypeMirror(); //获取未编译类
            TypeElement element = (TypeElement) declaredType.asElement();
            mAnnotateTypeName = element.getQualifiedName().toString();
            mAnnotatedTypeSimpleName = element.getSimpleName().toString();
        }

    }


    //获取被@Factory注解的原始元素
    public TypeElement getAnnotatedClassElement() {
        return mAnnotatedClassElement;
    }

    //// 获取Factory#type()中指定的type()的类型合法全名
    public String getAnnotateTypeName() {
        return mAnnotateTypeName;
    }

    // 获取Factory#type()中指定的type()的类型简单名字
    public String getAnnotatedTypeSimpleName() {
        return mAnnotatedTypeSimpleName;
    }

    //获取 {@link Factory#id()}中指定的id()
    public String getId() {
        return mId;
    }
}
```

#### 5.4  FactoryItemTypeContainer 保存所有 FactoryAnnotationClass 的，并且根据FactoryAnnotationClass用Filer去生成Java代码。

这里编写Java代码，利用的是Square的开源框架 javapoet ： https://github.com/square/javapoet


```
public class FactoryItemTypeContainer {


    private String itemTypeName;

    private final String FACTORY = "Factory";

    private Map<String, FactoryAnnotationClass> mAnnotationClassMap = new HashMap<>();


    public FactoryItemTypeContainer(String itemTypeName) {
        this.itemTypeName = itemTypeName;
    }


    //添加
    public void add(FactoryAnnotationClass toInsert) throws ProcessingException {
        FactoryAnnotationClass exist = mAnnotationClassMap.get(toInsert.getId());
        if (exist != null) {
            throw new ProcessingException(toInsert.getAnnotatedClassElement(), "Conflit : The class %1$s is annotated " +
                    "with  %2$s with id = %3$s but %4$s already use the same id", toInsert.getAnnotatedClassElement()
                    .getQualifiedName().toString(), Factory.class.getName(), toInsert.getId(), exist
                    .getAnnotatedClassElement().getQualifiedName().toString());
        }
        mAnnotationClassMap.put(toInsert.getId(), toInsert);
    }


    //拼接字符串编写Java代码 ，使用Square的开源框架 javapoet ： https://github.com/square/javapoet
    /**
     * public class CarFactory {
     * 
     *      public Car create(String id) {
     *          if (id == null) {
     *               throw new IllegalArgumentException("id is Null");
     *          }
     *          if (id.equals("Audi")) {
     *               return new Audi();
     *          }
     * 
     *          if (id.equals("BMW")) {
     *              return new BMW();
     *          }
     * 
     *          if (id.equals("Benz")) {
     *               return new Benz();
     *           }
     *          throw new IllegalArgumentException("unKnow id = " + id);
     *      }
     * 
     *  }
     */
    public void generateCode(Elements elementUtils, Filer filer) throws IOException {

        TypeElement superTypeElement = elementUtils.getTypeElement(itemTypeName);
        String factoryClassName = superTypeElement.getSimpleName() + FACTORY; // 将要生成的工厂类名

        // create() 方法编写
        MethodSpec.Builder methodBuilder = MethodSpec
                .methodBuilder("create")
                .addModifiers(Modifier.PUBLIC)
                .returns(TypeName.get(superTypeElement.asType()))
                .addParameter(String.class, "id")
                .beginControlFlow("if (id == null ||\"\".equals(id))")
                .addStatement(" throw  new IllegalArgumentException(\"id is Null\")")
                .endControlFlow();
        for (FactoryAnnotationClass factoryAnnotationClass : mAnnotationClassMap.values()) {
            methodBuilder.beginControlFlow("if ($S.equals(id))", factoryAnnotationClass.getId())
                    .addStatement("return new $L()", factoryAnnotationClass.getAnnotatedClassElement().getQualifiedName().toString())
                    .endControlFlow();
        }
        methodBuilder.addStatement("throw new IllegalArgumentException($S + id)", "Unknown id = ");

        // 类编写
        TypeSpec typeSpec = TypeSpec.classBuilder(factoryClassName)
                .addModifiers(Modifier.PUBLIC)
                .addMethod(methodBuilder.build())
                .build();
        JavaFile.builder(elementUtils.getPackageOf(superTypeElement)
                .getQualifiedName().toString(), typeSpec)
                .build()
                .writeTo(filer);

    }

}
```

#### 5.4 FactoryProcessor ： 处理@Factory注解的自定义注解处理器。注意这个需通过独立的lib或jar进行引入。


```
@AutoService(Processor.class)
public class FactoryProcessor extends AbstractProcessor {

    private Elements mElementUtils;
    private Types mTypeUtils;
    private Messager mMessager;
    private Filer mFiler;

    private Map<String, FactoryItemTypeContainer> mFactoryItemTypeContainers = new HashMap<>();

    //初始化方法，会被注解处理工具调用，并提供ProcessingEnvironment，可通过其获取各种需要的工具类
    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        mElementUtils = processingEnv.getElementUtils();
        mTypeUtils = processingEnv.getTypeUtils();
        mMessager = processingEnv.getMessager();
        mFiler = processingEnv.getFiler();

    }

    //指定使用的Java版本。一般使用SourceVersion.latestSupported()
    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    //指定这个注解处理器对哪个注解进行处理(此处对@Factory进行处理)
    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> annotations = new LinkedHashSet<>();
        annotations.add(Factory.class.getCanonicalName()); // Factory.class.getCanonicalName() = "com.zk.annotation_lib.Factory"
        return annotations;
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        try {
            for (Element element : roundEnv.getElementsAnnotatedWith(Factory.class)) {
                TypeElement typeElement = (TypeElement) element;
          /*  mMessager.printMessage(Diagnostic.Kind.NOTE, "=============NOTE======", element);
            mMessager.printMessage(Diagnostic.Kind.WARNING, "==========WARNING=========", element);
            mMessager.printMessage(Diagnostic.Kind.ERROR, "==========ERROR=========", element);*/

                FactoryAnnotationClass factoryAnnotationClass = new FactoryAnnotationClass(typeElement);
                if (!idValidClass(factoryAnnotationClass)) {
                    return true; //已经打印了错误信息，退出处理
                }
                FactoryItemTypeContainer factoryItemTypeContainer = mFactoryItemTypeContainers.get(factoryAnnotationClass
                        .getAnnotateTypeName());
                if (factoryItemTypeContainer == null) {
                    factoryItemTypeContainer = new FactoryItemTypeContainer(factoryAnnotationClass.getAnnotateTypeName());
                    mFactoryItemTypeContainers.put(factoryAnnotationClass.getAnnotateTypeName(), factoryItemTypeContainer);
                }
                factoryItemTypeContainer.add(factoryAnnotationClass);

            }
            //创建每个type()的 XXFacroty类
            for (FactoryItemTypeContainer itemTypeContainer : mFactoryItemTypeContainers.values()) {
                itemTypeContainer.generateCode(mElementUtils, mFiler);
            }
            //清除factoryClasses。防止第二轮中
            mFactoryItemTypeContainers.clear();
        } catch (IllegalArgumentException exception) {
            mMessager.printMessage(Diagnostic.Kind.ERROR, exception.getMessage(), null);
            error(null, exception.getMessage());

        } catch (ProcessingException e) {
            error(e.getElement(), e.getMessage());

        } catch (IOException e) {
            e.printStackTrace();
        }
        return true;
    }


    private boolean idValidClass(FactoryAnnotationClass factoryAnnotationClass) throws ProcessingException {

        TypeElement currentClassElement = factoryAnnotationClass.getAnnotatedClassElement();
        //判断当前类是否为public
        if (!currentClassElement.getModifiers().contains(Modifier.PUBLIC)) {
            error(currentClassElement, "the class %$s is not public", currentClassElement.getQualifiedName().toString());
            return false;
        }

        //获取@Factory的type()的TypeElement。也就是被注解元素的父类。如类Car
        TypeElement superClassElement = mElementUtils.getTypeElement(factoryAnnotationClass.getAnnotateTypeName());
        if (superClassElement.getKind() == ElementKind.INTERFACE) {
            if (!currentClassElement.getInterfaces().contains(superClassElement.asType())) {
                throw new ProcessingException(currentClassElement, "The class %1$s annoated with %2$s must implement the interface %3$s",
                        currentClassElement.getQualifiedName().toString(), Factory.class.toString(), factoryAnnotationClass
                        .getAnnotateTypeName());
            }

        } else {
            TypeMirror superclass = superClassElement.getSuperclass();
            while (true) {
                TypeKind superclassKind = superclass.getKind();
                //到达了基本类型（java.lang.Object）,所以退出
                if (superclassKind == TypeKind.NONE) {
                    throw new ProcessingException(currentClassElement, "The class %1$s annotated with %2$s must interit from %3$s",
                            currentClassElement.getQualifiedName(), Factory.class.toString(), factoryAnnotationClass.getAnnotateTypeName());
                }
                if (superclassKind.toString().equals(factoryAnnotationClass.getAnnotateTypeName())) {
                    //找到了要的父类为注解的type()类型
                    break;
                }
                //在继承树上继续查找
                currentClassElement = (TypeElement) mTypeUtils.asElement(superclass);
            }
        }
        //是否提供空参数构造函数
        for (Element enClosed : currentClassElement.getEnclosedElements()) {
            if (enClosed.getKind() == ElementKind.CONSTRUCTOR) {
                ExecutableElement constructorElement = (ExecutableElement) enClosed;
                if (constructorElement.getParameters().size() == 0 && constructorElement.getModifiers().contains(Modifier.PUBLIC)) {
                    //找到了默认函数
                    return true;
                }
            }

        }
        //没有找到默认空参数构造函数
        throw new ProcessingException(currentClassElement, "The class of %1$s must provide a public empty default constructor",
                currentClassElement.getQualifiedName().toString());

    }


    //打印错误信息
    private void error(Element element, String msg, Object... args) {
        mMessager.printMessage(Diagnostic.Kind.ERROR, String.format(msg, args), element);
    }
}
```

注意自定义处理器需要通过独立的lib或jar进行引入。这里我的FactoryProcessor放在 factory module下。那app module需要使用该注解时，需要在app的bugild.gralde下添加依赖：

```
implementation project(':factory')
annotationProcessor project(':factory')
```


然后我们AndroidStudio的顶部栏点击 Build -->  MakeProject按钮  即可生成CarFactory类。
生成的CarFactory类生成目录为：**app --> Build --> generate --> source --> apt --debug** 下。 并且生成的这个类是我们不能修改编辑的。

![CarFactory](1_CarFactory.png)




好了，我们直接就可以调用了：
    
```
CarFactory carFactory = new CarFactory();
carFactory.create("Benz").drive();
```

当我们新增一个Car类，如：

```
@Factory(
        id = "Porsche",
        type = Car.class
)
public class Porsche implements Car {
    @Override
    public void drive() {
        
    }
}
```

此时我们不用管什么if ..else.. ,我们只需重新Build --> MakeProject即可，它会重新生成CarFactory类。

是不是很爽呐,很好地体现了Java的开闭原则，只管添加扩展功能，而不影响原有的代码。


### 参考 ：
- http://hannesdorfmann.com/annotation-processing/annotationprocessing101
- [Java中文开发文档](http://tool.oschina.net/apidocs/apidoc?api=jdk-zh)














































































