# 函数式接口

## 相关概念

- **有且只有一个抽象方法(还可以有多个非抽象方法)** 的**接口**(**不能**是**抽象类**)

- 其注解为**`@FunctionalInterface`**，但是并不是说写了这个注解才算函数式接口，写上这个注解可以让编译器帮你检查
- 函数式接口是从**JDK1.8**引入的，官方的有一部分在**`java.util.function`包**下，我所知道的还有在**`java.util`包**下的......
- 可以自己定义**函数式接口**，但是建议使用**`@FunctionalInterface`**来让编译器检查是否有错误



## 常见的函数式接口

```java
//java.util.function
@FunctionalInterface public interface Function<T,R>		//代表一个接受一个参数T并产生一个结果R的函数R apply(T t)
@FunctionalInterface public interface BiFunction<T,U,R>
@FunctionalInterface public interface IntFunction<R>	//接受一个原始类型int的参数,并产生一个类型为 R 的结果
    													//R  apply(int value)
@FunctionalInterface public interface BinaryOperator<T> extends BiFunction <T,T,T>
@FunctionalInterface public interface UnaryOperator<T> extends Function <T,T>

@FunctionalInterface public interface Consumer<T>		//表示接受单个输入参数且不返回任何结果的操作 accept(T)
@FunctionalInterface public interface BiConsumer<T,U>	//表示接受两个输入参数且不返回任何结果的操作,(key,value)
    
@FunctionalInterface public interface Predicate<T>		//一个条件判断器，专门用来封装返回 boolean 值的逻辑规则
    													//boolean test(T t)
@FunctionalInterface public interface Supplier<T>

@FunctionalInterface public interface BiPredicate<T,U>

```

```java
//java.util
@FunctionalInterface public interface Comparator<T>		//定义两个对象T之间的一种比较方式(或排序规则）
    //int compare(T o1, T o2)
   /*
官方定义标准:
Compares its two arguments for order. Returns a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second.
(中文翻译：比较其两个参数以确定顺序。当第一个参数小于、等于或大于第二个参数时，分别返回负整数、零或正整数)
   返回值含义：
		负整数: 表示 o1 小于 o2(o1 应该排在 o2 前面)
		零: 表示 o1 等于 o2(它们的相对顺序不重要，或者根据具体排序算法，它们的顺序可能保持不变)
		正整数: 表示 o1 大于 o2(o1 应该排在 o2 后面)
     使用场景：
     Collections.sort(list,comparator);
     Arrays.sort(array, comparator);
     Stream中的sorted(comparator);
     TreeSet 和 TreeMap 的构造函数可以接受一个 Comparator 来指定元素的排序方式;
     Collections.max(collection, comparator);
     Collections.min(collection, comparator);
     Stream的max(comparator)、min(comparator);
    */
```



## 注意

- 如果一些抽象方法和**`Object`**会继承给实现类的方法的名字相同，这种方法不强制要求重写，因为实现类从**`Object`**中已经继承下来非抽象的了



# Lambda表达式

## 概念与特点

- 让代码更加简洁，促进了 **函数式接口** 的使用

- **Lambda**只能用于**函数式接口**，你**自己也可以定义**函数式接口，但是要**保证定义正确**，否则无法使用**Lambda**

- **Lambda 表达式是以函数式接口为目标类型的语法简写形式，其底层通过 `invokedynamic` 指令动态生成实现类，相比传统匿名内部类具有更优的性能特性和内存效率**

  > 我自己总结的是下面这个，但是好像不太对：Lambda表达式是  实现了函数式接口的匿名内部类的语法糖形式

## 格式

```java
()->{}
//小括号对应方法形参		如果只有一个形参，小括号可以省略
//大括号对应方法体		 如果方法体中只有一条语句，大括号和这条语句后面的分号都可以省略，但如果要省略，这两者必须同时省略
```

## 可以省略的语法

- 匿名内部类在创建时使用的**`new`关键字**和**接口名**，**接口名后面的大括号**，以及**方法名**,这几个**必须同时省略**，省略后还**必须**要加上**`->`**

> 并不是所有匿名内部类都可以被Lambda表达式代替和使用，有的匿名内部类实现的接口是函数式接口，这种接口可以被Lambda代替和使用，而有的匿名内部类中可能要实现多个抽象方法，这种就不行，因为所实现的这种接口不是函数式接口，必须是函数式接口，就是只有一个抽象方法的那个接口

- 方法中**形参的类型**也可以省略
- 如果**只有一个形参**，**小括号**可以省略
- 如果**方法体中只有一条语句**，**大括号和这条语句后面的分号**,以及**`return`关键字**都可以省略，但**如果要省略，这些必须同时省略**

### 示例

```java
()->sout("Hello")
a->sout("a:"+a)
(a,b)->{sout(a);
        sout(b)}
```



# 方法引用

## 相关概念与知识

- **Lambda表达式的简写形式**， 是一种**语法糖**
- 本质上还是**Lambda表达式**

- 使用**` ::`** 符号(**方法引用操作符 (Method Reference Operator)**)
- 使用**方法引用** **替代** **Lambda 表达式**的前提条件
  - **目标类型**必须是**函数式接口**
  - 被引用的方法**必须已经存在**
  - 引用的方法（或构造器）的**签名**必须与目标**函数式接口的抽象方法的签名** **兼容**
    - **参数数量和类型必须兼容**（考虑自动装箱/拆箱、类型转换、可变参数）
    - **返回类型必须兼容**（函数式接口中的 void 兼容任何被引用方法的返回类型，只是返回类型会被丢弃，当然是指有形参的方法能传入void方法，如上所述，当然如果方法没形参，是传不进去参数的，啊呀写的时候尽量写正确吧；函数式接口中的父类兼容被引用方法的子类返回类型。）
    - **异常必须兼容**（引用的方法抛出的检查异常必须与函数式接口声明的异常兼容）



## 方法引用的几种类型

### 引用静态方法

- 语法： **类名::静态方法名**

- 示例

  ```java
  public static void main(String[] args) {
          ArrayList<String> list = new ArrayList<>();
          Collections.addAll(list,"1","2","3","4","5","6","7","8","9");
          List<Integer> col = list.stream().
              						map(Integer::parseInt).
              						collect(Collectors.toList());
          System.out.println(col);
      }
  ```



### 引用成员方法

- 语法

  - **引用本类中的成员方法**：**this::成员方法名**

  - **只能在成员方法中用**

    - 示例

      ```java
      public class ThisDemo {
          public void print(String c){
              System.out.println(c);
          }
      
          public void test(Consumer<String> c){
              c.accept("This is a sentence");
          }
      
          public void m() {
              test(this::print);  //此处使用方法引用,引用本类中的print方法,因为它和Consumer中的accept方法兼容
          }
      }
      ```

  - **引用父类中的成员方法**：**super::成员方法名**

  - **只能在成员方法中用**

    - 示例

      ```java
      class Father {
          public void fatherMethod(String s) {
              System.out.println(s);
          }
      }
      
      public class Son extends Father {
          public void test(Consumer<String> c){
              c.accept("This is a test");
          }
          public void m(){
              test(super::fatherMethod);//此处使用方法引用，引用父类中的fatherMethod方法,因为它和accept兼容
          }
      }
      ```

  - **引用其他类中的成员方法**：**其他类的对象::成员方法名**

    - 示例

      ```java
      list.stream().filter(new String()::startsWith).forEach(System.out::println);
      					//引用String中的成员方法startsWith以及PrintStream中的成员方法println
      ```




### 引用构造方法

- 语法：**类名::new**

- **构造方法**可以理解成**接收参数(或不接收)，返回对应对象的方法**，所以**满足方法引用的前提要求便可用来进行方法引用**

  - 实例

    ```java
    class Student{
        private String name;
        private int age;
        public Student(String s) {		//可以理解为:接收参数(或不接收),返回对应对象的方法
            this.name = s.split(",")[0];
            this.age = Integer.parseInt(s.split(",")[1]);
        }
    }
    
    public class Main{
        public static void main(String[] args) {
        	ArrayList<String> list = new ArrayList<>();
            Collections.addAll(list, "张无忌,15", "周芷若,14", "赵敏,13", "张强,20", "张三丰,100");
            List<Student> col = list.stream().map(Student::new).collect(Collectors.toList());//方法引用
    		System.out.println(col);
        }
    }
    ```



### 使用类名引用成员方法

- 语法：**类名::成员方法**

- 前提： **被引用的成员方法的返回值要和函数式接口中抽象方法的返回值兼容**；
      **函数式接口中的抽象方法的第一个参数的类型要和引用的方法所在类兼容**；
      **函数式接口中的抽象方法的剩余参数要和被引用的成员方法的参数所匹配，如果没有，说明这个被引用的成员方法无参**；
      **如果被引用的成员方法声明了会抛出检查异常 (Checked Exception)，那么函数式接口的抽象方法也**必须**在它的`throws` 子句中声明这个检查异常，或者是它的父类型异常。否则，编译不通过**

- Java文档中对于**参数**的规定：

  - 函数式接口的抽象方法的**第一个** **形式参数**被用作方法调用的**目标引用 (target reference)** 或**接收者 (receiver)**

    > 这句话的意思是：第一个参数作为被引用方法的调用者
    >
    > 我感觉在Stream流中啊，第一个参数一般都表示流里面的每一个数据

  - 函数式接口的抽象方法的**剩余** **形式参数**作为**参数 (arguments)** 传递给该方法

- 这种**方法引用**的**思想**：

  - **普通成员方法（实例方法）需要对象：** 像 **`length()`**、**`toUpperCase()`**、**`isEmpty()`** 这些方法，它们操作的是对象**内部的数据**（比如字符串里的字符）。所以，你必须先有一个**具体的对象**（比如一个具体的字符串 "hello"），才能调用这些方法。没有对象，这些方法就不知道该对谁操作。正因为这些方法依赖对象内部的状态，所以它们**不能随便变为静态方法** 。 当我们遇到这种情况，想把“**调用某个实例方法**”这个**动作本身**交给别人(比如 Stream API 的 map 操作或者 sort 里的比较器)去用时，我们就可以使用**类名::成员方法名**这种方法去进行方法引用。**“传入谁就调用谁的”** 是它的核心！等真正调用的时候，把传进来的**第一个参数**当作那个**对象实例**，然后**让这个对象实例去执行那个成员方法**。如果还需要其他参数，就用传进来的**第二、第三个参数...**

  - 还可以理解为：**在函数式接口的抽象方法实现中，在抽象方法的方法体中，用这个抽象方法的参数调用这个引用的成员方法**

- 示例

  ```java
  list.stream().map(String::toLowerCase).forEach(System.out::println);
  ```



### 数组的构造器引用

- 语法：**数组中元素类型名[]::new**

- 用处：用在那些抽象方法接受一个整数（代表数组大小 **`size`**） 并 **返回该类型数组** 的函数式接口上(如**`IntFunction<R>`、`Function<Integer, R>`**)(**`Stream`**中的**`toArray(IntFunction<A[]> generator)`等方法**

  **这个语法天生就是为了匹配 “输入一个整数大小，输出对应类型和大小的数组” 这个操作的**

- 示例：

  ```java
  Integer[] array = list.stream().toArray(Integer[]::new);
  ```

  

## 