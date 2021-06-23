# `String`

String是不可变类，一旦一个String对象被创建，包含在这个对象中的字符序列就不能再改变，直至这个对象被销毁。

调用API修改字符串内容时，实际返回的是一个新的对象。

String类的构造方法：

- `String()`
- `String(byte[] bytes, Charset charset)`
- `String(byte[] bytes, String charsetName)`
- `String(byte[] bytes, int offset, int length)`
- `String(byte[] bytes, int offset, int length, String charsetName)`
- `String(char[] value)`
- `String(char[] value, int offset, int count)`
- `String(StringBuffer buffer)`
- `String(StringBuilder builder)`

常用方法：

- `char charAt(int index)`
- `int compareTo(String anotherString)`：根据字典序比较两个字符串。
- `String concat(String str)`：与字符串拼接符'+'相同。
- `boolean contentEquals(StringBuffer sb)`：比较`String`与`StringBuffer`。
- `static String copyValueOf(char[] data)`：将字符数组转换成字符串并返回。
- `static String copyValueOf(char[] data, int offset, int count)`
- `boolean endsWith(String suffix)`：判断该`String`对象是否以`suffix`结尾。
- `boolean equals(Object anObject)`：比较两个字符串，字符序列相同时返回true。
- `boolean equalsIgnoreCase(String str)`：忽略大小写比较字符串。
- `byte[] getBytes()`：将String转为byte数组。
- `void getChars(int srcBegin, int srcEnd, char[] dst, int dstBegin)`
- `int indexOf(int ch)`
- `int indexOf(int ch, int fromIndex)`：找出`fromIndex`后`ch`的第一次出现位置。
- `int indexOf(String str)`
- `int indexOf(String str, int fromIndex)`
- `int lastIndexOf(int ch)`
- `int lastIndexOf(int ch, int fromIndex)`
- `int lastIndexOf(String str)`
- `int lastIndexOf(String str, int fromIndex)`
- `int length()`
- `String replace(char oldChar, char newChar)`：将字符串第一个`oldChar`替换为`newChar`。
- `String replaceFirst(String regex, String replacement)`：替换匹配的第一个子串。
- `String replaceAll(String  regex, String replacement)`
- `boolean startsWith(String prefix)`
- `boolean startsWith(String prefix, int offset)`：offset位置开始是否以prefix开头。（左闭）
- `String substring(int beginIndex)`
- `String substring(int beginIndex, int endIndex)`
- `char[] toCharArray()`
- `String toLowerCase()`
- `String toUpperCase()`
- `static String valueOf(X x)`：X为基本类型。
- `String[] split(String regex, int limit)`

String类是不可变的，因此会产生很多临时变量。

# `StringBuffer`

`StringBuffer`对象代表一个字符序列可变的字符串。`StringBuffer`对象创建后可通过`append`、`insert`、`reverse`、`setCharAt`、`setLength`等方法改变字符序列，最后调用`toString`方法转换回字符串。

`StringBuffer`的方法：

- `StringBuffer append(char[]/String/char/int/boolean/long/...)`
- `StringBuffer delete(int start, int end)`
- `StringBuffer deleteCharAt(int index)`
- `StringBuffer replace(int start, int end, String str)` 
- `StringBuffer reverse()`

# `StringBuilder`

`StringBuilder`时Jdk1.5新增的类，类似`StringBuffer`。但`StringBuffer`是线程安全的，而`StringBuilder`没有实现线程安全功能，因此性能略强。通常情况下，优先考虑使用`StringBuilder`类。

# **创建字符串分析**

1 直接使用双引号创建字符串

判断这个常量是否存在于常量池
  如果存在
       判断这个常量是堆中对象的引用还是常量
            如果是引用，返回引用地址指向的堆空间对象，
            如果是常量，则直接返回常量池常量，
  如果不存在
            在常量池中创建该常量，并返回此常量

2 使用new String创建字符串

在堆上创建对象(无论堆上是否存在相同字面量的对象)
判断常量池中是否存在该字符串
  如果不存在
       在常量池上创建常量
  如果存在
       不做任何操作