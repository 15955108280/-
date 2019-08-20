### Lambda表达式

->代替匿名内部类，简化匿名内部类的写法（接口中只能有一个抽象方法）

::可以引用方法类，静态方法，构造函数

~~~java
public class Lambda {
	public static  void main(String args[]){
        //实现了conparator接口并重写了compute方法
        //若是就一个参数（a,b）可以连括号也省略
		Comparator comparator = (a,b)->a*b;
        //
		Comparator comparator1 = Something::startWith;
		System.out.println(comparator1.compute(1,2));
		System.out.println(comparator.compute(1,2));
	}
public interface Comparator{
	int compute(int a, int b);
}
public class Something {
	static Integer startWith(Integer a,Integer b){
		return  a+b;
	}
}
~~~

lambda可以访问表达式外部的final局部变量，若是未声明为final局部变量，则会在编译过程中自动翻译为final局部变量

~~~java
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
//由于num已经编译为final所以下一步执行报错
num = 3;
~~~

lambda表达式的内部能获取到对成员变量或静态变量的读写权

### 数据流

~~~java
List<String> myList =
    Arrays.asList("a1", "a2", "b1", "c2", "c1");

myList
    .stream()
    .filter(s -> s.startsWith("c"))
    .map(String::toUpperCase)
    .sorted()
    .forEach(System.out::println);
//运行结果
// C1
// C2
~~~

数据流又衔接操作和终止操作衔接操作无需分号隔离有返回值。

~~~java
Stream.of("d2", "a2", "b1", "b3", "c")
    .filter(s -> {
        System.out.println("filter: " + s);
        return true;
    })
    .forEach(s -> System.out.println("forEach: " + s));

运行结果：
filter:  d2
forEach: d2
filter:  a2
forEach: a2
filter:  b1
forEach: b1
filter:  b3
forEach: b3
filter:  c
forEach: c
~~~

啊啊啊啊
