泛型
=

oracle官方定义：
----
https://docs.oracle.com/javase/tutorial/java/generics/index.html

大意（个人翻译可能有出入）：

在编程中错误是普遍存在的，但是有些错误可以在编译时就检测到，那么可以立刻找到然后修复。但是运行时错误可能会带来更多的问题，通常不能在编译时就检测到，通常会在运行时才出现的错误。

泛型的使用就是通过在编译时检测到更多的错误来增加代码的稳定性。

为什么要使用泛型
----
+ 编译时进行更强类型的检查
+ 消除类型转换
    
    Example:
           
           //不使用泛型
           List list = new ArrayList();
           list.add("hello");
           String s = (String) list.get(0);
           
           //使用泛型
           List<String> list = new ArrayList<String>();
           list.add("hello");
           String s = list.get(0);   // no cast


如何使用泛型
----
+ 定义泛型类

        public class Box<T> {
            public void set(T t) { /* ... */ }
            // ...
        }
        
+ 定义泛型方法

        //只能用 > 比较基本类型，而不能比较 Objects
        public static <T> int countGreaterThan(T[] anArray, T elem) {
            int count = 0;
            for (T e : anArray)
                if (e > elem)  // compiler error
                    ++count;
            return count;
        }
        
        // 或者这样定义，与编译有关
        public class Solution3<T> {
        
            public int countGreaterThan(T[] anArray, T elem) {
                int count = 0;
                for (T e : anArray)
                    if (e > elem)  // compiler error
                        ++count;
                return count;
            }
            
        }
        
        //改进版
        public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
            int count = 0;
            for (T e : anArray)
                if (e.compareTo(elem) > 0)
                    ++count;
            return count;
        }
        
如何实现？泛型擦除
----
看这段代码

        public static void main(String[] args) {
            Pair<String> stringPair = new Pair<>();
            Pair<Integer> integerPair = new Pair<>();
            System.out.println(stringPair.getClass() == integerPair.getClass());  //true
        }

类信息一样？我不是一个 String 一个 Integer 吗？ 既然这样看看 class 文件吧

        javap -c xxx.class //反编译 class 文件
        
原代码:

        class Pair<T> {  
        
            private T value;  
        
            public T getValue() {  
                return value;  
            }  
        
            public void setValue(T value) {  
                this.value = value;  
            }  
        }
    
反编译后        
        
        class com.x.test.Pair<T> {
          com.x.test.Pair();
            Code:
               0: aload_0
               1: invokespecial #1                  // Method java/lang/Object."<init>":()V
               4: return
        
          public T getValue();
            Code:
               0: aload_0
               1: getfield      #2                  // Field value:Ljava/lang/Object;
               4: areturn
        
          public void setValue(T);
            Code:
               0: aload_0
               1: aload_1
               2: putfield      #2                  // Field value:Ljava/lang/Object;
               5: return
        }

发现泛型方法为 Object 类型，类型被擦除。

既然这样，那么再引入一个问题，看代码
        
原代码:
        
        class DateInter extends Pair<Date> {  
        
            @Override  
            public void setValue(Date value) {  
                super.setValue(value);  
            }  
        
            @Override  
            public Date getValue() {  
                return super.getValue();  
            }  
        }
        
在这个子类中，我们设定父类的泛型类型为Pair<Date>，在子类中，我们覆盖了父类的两个方法，我们的原意是这样的：将父类的泛型类型限定为Date，那么父类里面的两个方法的参数都为Date类型。

如果按之前的推断，编译后都是 Object 类型，为什么这个子类还是 @Override 的？ 这难道不应该是重载么？
如果是重载那么我应该两个方法都能调用的，code 如下

        DateInter dateInter = new DateInter();  
        dateInter.setValue(new Date());                  
        dateInter.setValue(new Object());   // error
        
既然这样那么一定是重写了，那这是为什么？ （Java真难，手动滑稽）
不知道怎么办，只能看看反编译之后 class 文件了

反编译之后
        
        class com.x.test.DateInter extends com.x.test.Pair<java.util.Date> {  
          com.x.test.DateInter();  
            Code:  
               0: aload_0  
               1: invokespecial #8                  // Method com/x/test/Pair."<init>":()V  
               4: return  
        
          public void setValue(java.util.Date);  //我们重写的setValue方法  
            Code:  
               0: aload_0  
               1: aload_1  
               2: invokespecial #16                 // Method com/x/test/Pair.setValue:(Ljava/lang/Object;)V  
               5: return  
        
          public java.util.Date getValue();    //我们重写的getValue方法  
            Code:  
               0: aload_0  
               1: invokespecial #23                 // Method com/x/test/Pair.getValue:()Ljava/lang/Object;  
               4: checkcast     #26                 // class java/util/Date  
               7: areturn  
        
          public java.lang.Object getValue();     //编译时由编译器生成的桥方法  
            Code:  
               0: aload_0  
               1: invokevirtual #28                 // Method getValue:()Ljava/util/Date 去调用我们重写的getValue方法;  
               4: areturn  
        
          public void setValue(java.lang.Object);   //编译时由编译器生成的桥方法  
            Code:  
               0: aload_0  
               1: aload_1  
               2: checkcast     #26                 // class java/util/Date  
               5: invokevirtual #30                 // Method setValue:(Ljava/util/Date; 去调用我们重写的setValue方法)V  
               8: return  
        }

发现多了一套 set/get 方法来帮我们做了类型转换。这就是 桥方法 ，来解决类型擦除和多态的冲突。

上述只是类型擦除的个别问题，还有如下
+ 泛型类型变量不能是基本数据类型
   
        擦除之后是 Object 但是 Object 不能存储具体的值
+ 编译时集合的 instanceof

        擦除之后是 Object 不存在 instanceof
+ 不能声明类型为类型参数的静态字段

        public class MobileDevice<T> {
            private static T os;
        
            // ...
        }
        
        //这样声明会有问题，静态变量在类加载阶段就会进行定义
        MobileDevice<Smartphone> phone = new MobileDevice<>();
        MobileDevice<Pager> pager = new MobileDevice<>();
        MobileDevice<TabletPC> pc = new MobileDevice<>();
        
+ 无法创建参数化类型的数组

        // 一样的问题擦除之后不能识别 List 中的具体类型
        List<Integer>[] arrayOfLists = new List<Integer>[2]; // compile-time error

+ 无法将每个重载的形式参数类型擦除为相同原始类型的方法重载

        // 擦除完之后编译器不能定位具体的方法了
        public class Example {
            public void print(Set<String> strSet) { }
            public void print(Set<Integer> intSet) { }
        }
泛型上界、下界
----
上界通配符 < ? extends E>

+ 如果调用方传入的不是 E 或者 E 的子类，那么编译不成功
+ 泛型中可以使用 E 的方法，要不然还得强转成 E 才能使用

使用示例:

        //todo 之后分析一个示例

下界通配符 < ? super E>

+ 传入 E 或者 E 的父类，否则编译不成功

