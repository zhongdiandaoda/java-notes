# Overview

Lambda表达式的作用是代替匿名内部类的烦琐语法。

当使用Lambda表达式代替匿名内部类创建对象时，Lambda表达式的代码块将会代替实现抽象方法的方法体。

Lambda表达式主要由三个部分组成：

- 形参列表。形参列表允许省略形参类型。如果形参列表只有一个参数，那么括号也可以省略。
- 箭头(->)。
- 代码块：如果代码块只有一行代码，可以省略花括号。

Lambda表达式和匿名内部类都可以直接访问外部类的成员变量，但它们会被自动当作final成员来进行处理。

**匿名内部类实现的抽象方法的方法体允许调用接口中定义的default方法，而Lambda表达式不可以。**

# Lambda表达式与函数式接口

Lambda表达式的类型，也被称为“目标类型”，必须是函数式接口，即只包含一个抽象方法的接口。函数时接口可以包含多个default方法、类方法，但只能声明一个抽象方法。

Runnable、ActionListener等接口都是函数式接口。

程序中，可以使用Lambda表达式对对象进行赋值。

```java
Runnable r = () -> {
    for(int i = 0; i < 10; i++)
        System.out.println(i);
}
```

`Java.util.fuction`包下包含了以下4类典型的函数式接口：

- `XxxFunction`：通常包含一个apply方法，该方法对参数进行处理、转换，然后返回一个新的值。通常用作对特定数据进行转换处理。
- `XxxConsumer`：通常包含一个accept方法，负责对参数进行处理，但不会返回结果。
- `XxxPredicate`：通常包含一个test方法，用于对参数进行某种判断，然后返回一个`boolean`值。
- `XxxSupplier`：通常包含一个`getAsXxx()`抽象方法，该方法不需要输入参数，该方法会按照某种算法返回一个数据。

# 方法引用与构造器引用

当Lambda表达式的代码块只有一行代码时，程序可以省略表达式中代码块的花括号，还能在代码块使用方法引用和构造器引用。

Lambda表达式支持的引用方式如下：

| 种类                   | 示例               | 说明                                                         |
| ---------------------- | ------------------ | ------------------------------------------------------------ |
| 引用类方法             | 类名::类方法       | 接口被实现方法的全部参数传给该类方法作为参数                 |
| 引用特定对象的实例方法 | 特定对象::实例方法 | 接口被实现方法的全部参数传给该方法作为参数                   |
| 引用某类对象的实例方法 | 类名::实例方法     | 接口被实现方法的第一个参数作为调用者，后面的参数传给该方法作为参数 |
| 引用构造器             | 类名::new          | 接口被实现方法的全部参数传给该构造器作为参数                 |

## 类的静态方法

例如，定义了以下函数式接口：

```java
@FunctionalInterface
interface Conventer {
    Integer convert(String from);
}
```

该接口的方法负责把字符串转为Integer对象，使用Lambda表达式创建对象的语法：

```java
Converter c = from -> Integer.valueOf(from);
```

上面的代码可以使用以下方法引用进行替换：

```java
//函数式接口的所有参数都传递给该方法
Converter c = Integer::valueOf();
```

## 类的实例方法

定义接口：

```java
@FunctionalInterface
interface MyTest {
    String test(String a, int b, int c);
}
```

创建实例：

```java
MyTest myTest = (a, b, c) -> a.substring(b, c);
```

方法引用：

```java
MyTest myTest = String::substring;
```

**类名::方法名，如果方法为静态方法，则将接口方法的全部参数传递给该静态方法；方法为实例方法是，将接口方法的第一个参数作为调用者，其他参数全部传递给该实例方法。**

## 特定对象的实例方法

使用基本lambda表达式创建一个Converter对象：

```java
String a = "999999994444445";
Conventer c = from -> a.indexOf(from);
```

调用：

```java
Integer v = c.convert();
```

可替换为：

```java
Converter c = a::indexOf;
```

# 构造器

创建函数式接口：

```java
@FunctionalInterface
interface MyTest {
    JFrame win(String a);
}
```

普通lambda表达式：

```java
Mytest myTest = (String a) -> new JFrame(a);
```

构造器引用：

```java
Mytest myTest = JFrame::new;
```

