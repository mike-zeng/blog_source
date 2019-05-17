---
title: 《改善java代码的151个建议》笔记(一)

date: 2019-02-19 20:00

categories:
- 读书笔记
- 《改善java代码的151个建议》

tags:
- 笔记
- 代码质量

---

1. **不要再常量和变量中使用易混淆的字母**

   ```java
   public class Test1 {
       public static void main(String[] args) {
           long i=1l;
           System.out.println(i*2);
       }
   }
   ```

   字母l和字母o尽量不要和数组混用，以免使阅读者的理解与程序意图产生偏差。如果字母和数字必须使用，字母"l"务必大写，字母"O"则增加注释。

2. **不要让常量蜕变成变量**

   ```java
   public class Test2 {
       public static void main(String[] args) {
           System.out.println(Const.RAND_CONST);
       }
   }
   interface Const{ 
       int RAND_CONST=new Random().nextInt();
   }
   ```

   常量就是常量，在编译期就必须确定其值，不应在运行期更改，否则程序的可读性会非常差。

3. **三元操作符的类型务必保持一致**

   ```java
   public class Test3 {
       public static void main(String[] args) {
           int i=80;
           String s1=String.valueOf(i<100?90:100);
           String s2=String.valueOf(i<100?90:100.0);
           System.out.println("s1==s2:"+s1.equals(s2));//false
       }
   }
   ```

   三元操作符类型的转换规则

   - 两个操作数不可转换则不做转换，返回值为Object类型
   - 两个操作数是明确类型的表达式，则按照正常的二进制数字来转换，int类型转为long，long类型转换为float类型。
   - 两个操作数中有一个是数字S，另一个是表达式，且类型表示为T，那么，若数字S在T的范围内，则转换T类型；若S超出了T类型的范围，则T转换为S类型。
   - 若两个操作上都是直接量数字，则返回值类型范围较大者。

4. **避免带有变长参数的方法**

   ```java
   public class Test4 {
       public void calPrice(int price,int discount){
           float k=price*discount/100.0F;
           System.out.println("打折后的价格是:"+formateCurrency(k));
       }
   
       public void calPrice(int price,int... discounts){
           float k=price;
           for (int discount:discounts){
               k*=(discount/100);
           }
           System.out.println("打折后的价格是:"+formateCurrency(k));
       }
   
       private String formateCurrency(float price){
           return NumberFormat.getCurrencyInstance().format(price/100);
       }
       public static void main(String[] args) {
           Test4 test4=new Test4();
           test4.calPrice(49000,75);//打折后的价格是:￥367.50
       }
   }
   ```

   上面的代码执行的是第一个方法。

5. **别让null值和控制威胁到变长方法**

   ```java
   public class Test5 {
       public void methodA(String str,Integer... is){
           
       }
       public void methodA(String str,String... is){
   
       }
       public static void main(String[] args) {
           Test5 test5=new Test5();
           String[] sarr=null;
           test5.methodA("China",0);
           test5.methodA("China","People");
           test5.methodA("China",sarr);//没问题
           test5.methodA("China",null);//无法编译通过
       }
   }
   ```

   前两个方法没问题，第三个执行的方法也没问题，第四个方法无法编译通过。第三个和第四个的区别是虽然`sarr`是null值，但是类型确定为`String[]`，但是第四个方法null是不确定类型的，所以会报错。

6. **覆写变长方法也循规蹈矩**

   ```java
   //代码无法编译通过
   public class Test6 {
       public static void main(String[] args) {
           Base base=new Sub();
           base.func(100,50);
           Sub sub=new Sub();
           sub.func(100,50);//找不到方法
       }
   }
   class Base{
       void func(int a,int... b){
           System.out.println("Base:func");
       }
   }
   
   class Sub extends Base{
       @Override
       void func(int a,int[] b){
           System.out.println("Sub:func");
       }
   }
   ```

7. **警惕自增陷阱**

   ```java
   public class Test7 {
       public static void main(String[] args) {
           int a=0;
           for (int i=0;i<10;i++){
               a=a++;
           }
           System.out.println(a);//输出0而不是10
       }
   }
   ```

   `a=a++`这行代码在有的语言中会完成自增加操作，但是在java不会，原因是`a=a++`执行时，现将a压入操作数栈，然后a本身自增，随后用从操作数栈中将栈顶元素赋值给a，因此a相当于自增了一次，但是又被原来的值覆盖了，一定要记住的是`++`操作是直接对变量操作的。

8. **不要让旧语法困扰你**

   ```java
   public class Test8 {
       public static void main(String[] args) {
           lable:
           for (int i=0;i<5;i++){
               for (int j=0;j<5;j++){
                   if (i*j==9){
                       break lable;
                   }
               }
           }
       }
   }
   ```

   goto是java中的一个保留字，没有直接实现，但是java支持类似goto功能的标签跳转功能，这种方式会使得代码结构混乱调试困难，因此尽量不要使用。

9. **少用静态导入**

   ```java
   import java.text.NumberFormat;
   
   import static java.lang.Math.PI;
   import static java.lang.Double.*;
   import static java.lang.Integer.*;
   import static java.text.NumberFormat.*;
   public class Test9 {
       public static void main(String[] args) {
           double s=PI*parseDouble(args[0]);
           NumberFormat nf=getInstance();
           nf.setMaximumFractionDigits(parseInt(args[1]));
           formatMessage(nf.format(s));
       }
       public static void formatMessage(String s){
           System.out.println("圆的面积是: "+s);
       }
   }
   ```

   静态导入在一定程度上会使得代码更加简介，但是频繁使用会降低代码的可读性。建议对于没有明显字面含义的变量和方法不要使用静态导入。

10. **不要在本类中覆盖静态导入的变量和方法**

    ```java
    import static java.lang.Math.PI;
    import static java.lang.Math.abs;
    public class Test10 {
        //会覆盖静态导入的变量和方法
        public final static String PI="祖冲之";
        public static int abs(int abs){
            return 0;
        }
        public static void main(String[] args) {
            System.out.println("PI="+PI);
            System.out.println("abs(-100)="+abs(-100));
        }
    }
    ```
    覆盖掉静态导入的变量和方法不会造成编译错误，并且在类中将会使用覆盖后的值，但是这样的代码可读性差。(难道不是照顾不懂的人？)

11. **养成良好习惯，显示声明UID**

    在编写实现`Serializable`接口的类时，要显示显示声明` serialVersionUID`。

    当没有显示声明` serialVersionUID`时候，jvm进行序列化的时候，会根据类进行一个复杂的计算得到该值。当反序列化时，jvm首先会判断两者是否相等，如果不相等，jvm会拒绝反序列化，对于jvm来说，` serialVersionUID`就是类的版本号，默认是根据类的修改而变化。但是更多时候我们更需要手动控制版本，所以需要显示声明如下语句。

    ```java
    private static final long  serialVersionUID=xxxL;
    ```

    

12. **避免用序列化类在构造函数中为不变量赋值**

    反序列化的一条规则是:反序列化的过程不会执行构造函数。

    final常量是不变的量，在序列化中应该保持一致，final常量有两种方式初始化，一种是直接方式，一种是勇敢构造函数，由于反序列化的时候不会执行构造函数，因此为了避免错误，应当不要在构造函数中为不变量赋值。

13. **避免为final变量复杂赋值**

    ```java
    public class Test13 implements Serializable {
        private static final long serialVersionUid=1L;
        public final String str=func();
        public String func(){
            return "哈哈哈";
        }
        public static void main(String[] args) throws Exception{
            Test13 test13=new Test13();
            FileOutputStream outputStream=new FileOutputStream("tets");
            ObjectOutputStream objectOutputStream=new ObjectOutputStream(outputStream);
            objectOutputStream.writeObject(new Test13());
            objectOutputStream.close();
            System.out.println(test13.str);
        }
    }
    ```

    ```java
    public class Test13 implements Serializable {
        private static final long serialVersionUid=1L;
        public final String str=func();
        public String func(){
            return "呵呵呵";
        }
        public static void main(String[] args) throws Exception{
            FileInputStream inputStream=new FileInputStream("test");
            ObjectInputStream objectInputStream=new ObjectInputStream(inputStream);
            Test13 test13=(Test13)objectInputStream.readObject();
    
            System.out.println(test13.str);
        }
    }
    //输出呵呵呵
    ```

    上面两端代码分别是序列化和反序列化的过程，str是final修饰的不变量，但是初始化是通过方法的，在反序列化时候，该方法发生了变化。反序列化正常，但是值依然是序列化之前的。

    

14. **使用序列化类的私有方法巧妙解决部分属性持久化问题（定制序列化）**

    [可以参考这篇博客](https://bluepopopo.iteye.com/blog/486548)

15. **break万万不可忘**

    在case语句后面应当随手加上break，千万不能忘记，否则会带来意想不到的或重大的事故。

16. **易变业务使用脚本语言编写**

    在java中可以执行其他脚本语言，如javascript等

    ```java
    public class Test16 {
        public static void main(String[] args) throws ScriptException {
            ScriptEngineManager sem=new ScriptEngineManager();
            ScriptEngine engine=sem.getEngineByName("javascript");
            engine.put("msg", "I am a good man!");
    
            String str = "var user = {name:'Matrix42',age:18,schools:['清华大学','北京大学']};";
            str+= "print(user.age);";
            engine.eval(str);
        }
    }
    ```

    动态语言具有灵活、便捷、简单的特征。脚本语言的这些特性是java没有的，如果这些特性可以引入java中，将会使得java更加强大。在java6之后，java正式支持执行脚本语言，凡是符合`JSR223`规范的脚本语言都可以在java环境中运行。使用脚本语言对系统设计最有利的地方是可以随时发布而不用重复部署。

17. **慎用动态编译**

    从java6开始java支持动态编译，可以在运行时编译.java代码并执行.class。

    使用动态编译需要注意以下几点

    - 在框架中谨慎使用
    - 不要在要求高性能的项目使用
    - 动态编译要考虑安全问题
    - 记录动态编译过程

18. **避免instanceof非预期结果**

    ```java
    public class Test18 {
        public static void main(String[] args) {
            //1.String 是否是Object的实例 true
            System.out.println("String" instanceof Object);
            //2.String 是否是String的实例 true
            System.out.println(new String() instanceof Object);
            //3.Object是否是String的实例 false
            System.out.println(new Object() instanceof String);
            //4.拆箱类型是否是装箱类型的实例 编译不通过
            System.out.println('A' instanceof Character);
            //5.空对象是否是String的实例 false
            System.out.println(null instanceof String);
            //6.类型转换后的空对象是不是String的实例 编译无法通过
            System.out.println((String)null instanceof String);
            //7.Date对象是否是String的实例，编译无法通过（无法比较没有继承关系的实例）
            System.out.println(new Date() instanceof String);
            //8.在泛型类中判断String是否是Date的实例 编译无法通过
            System.out.println(new GenericClass<String>().isDateInstance());
        }
        class GenericClass<T>{
            public boolean isDateInstance(T t){
                return t instanceof Date;
            }
        }
    }
    ```

    `instanceof`的判断规则

    - 只能判断在同一条继承链上的实例

19. **断言决定不是鸡肋**

    [什么时候用断言，什么时候用异常](https://www.zhihu.com/question/24461924)

20. **不要只替换一个类**

    对于final修饰的基本数据类型和String类型，编译器认为他是稳定态，在编译阶段就将值编译到了字节码中，如果该final常量来自于常量类，只是替换常量类的字节码文件，将不会改变原来字节码里面的常量值。

    但是对于现在的智能ide都会整体编译。

    在发布web项目时一定要整体发布war包，而不是替换其中的类。

***

**最后**

- 本篇笔记参考《改善java代码的151个建议》第一章 *Java开发中通用的方法和准则*
- 更多内容请查看原书。