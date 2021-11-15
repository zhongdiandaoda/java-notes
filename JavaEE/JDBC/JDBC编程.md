# overview

JDBC 是 Java DataBase Connectivity 的缩写，它是 Java 程序访问数据库的标准接口。

使用 Java 程序访问数据库时，Java 代码并不是直接通过 TCP 连接去访问数据库，而是通过 JDBC 接口来访问，而 JDBC 接口则通过 JDBC 驱动来实现真正对数据库的访问。

例如，我们在 Java 代码中如果要访问 MySQL，那么必须编写代码操作 JDBC 接口。注意到 JDBC 接口是 Java 标准库自带的，所以可以直接编译。而具体的 JDBC驱动是由数据库厂商提供的，例如，MySQL 的 JDBC 驱动由 Oracle 提供。因此，访问某个具体的数据库，我们只需要引入该厂商提供的JDBC驱动，就可以通过JDBC 接口来访问，这样保证了 Java 程序编写的是一套数据库访问代码，却可以访问各种不同的数据库，因为他们都提供了标准的 JDBC 驱动：

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

│  ┌───────────────┐  │
   │   Java App    │
│  └───────────────┘  │
           │
│          ▼          │
   ┌───────────────┐
│  │JDBC Interface │<─┼─── JDK
   └───────────────┘
│          │          │
           ▼
│  ┌───────────────┐  │
   │  JDBC Driver  │<───── Vendor
│  └───────────────┘  │
           │
└ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ┘
           ▼
   ┌───────────────┐
   │   Database    │
   └───────────────┘
```

从代码来看，Java标 准库自带的 JDBC 接口其实就是定义了一组接口，而某个具体的 JDBC 驱动其实就是实现了这些接口的类：

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐

│  ┌───────────────┐  │
   │   Java App    │
│  └───────────────┘  │
           │
│          ▼          │
   ┌───────────────┐
│  │JDBC Interface │<─┼─── JDK
   └───────────────┘
│          │          │
           ▼
│  ┌───────────────┐  │
   │ MySQL Driver  │<───── Oracle
│  └───────────────┘  │
           │
└ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ┘
           ▼
   ┌───────────────┐
   │     MySQL     │
   └───────────────┘
```

实际上，一个 MySQL 的 JDBC 的驱动就是一个 jar 包，它本身也是纯 Java 编写的。我们自己编写的代码只需要引用 Java 标准库提供的 java.sql 包下面的相关接口，由此再间接地通过 MySQL 驱动的 jar 包通过网络访问 MySQL 服务器，所有复杂的网络通讯都被封装到 JDBC 驱动中，因此，Java 程序本身只需要引入一个MySQL 驱动的 jar 包就可以正常访问 MySQL 服务器：

```ascii
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐
   ┌───────────────┐
│  │   App.class   │  │
   └───────────────┘
│          │          │
           ▼
│  ┌───────────────┐  │
   │  java.sql.*   │
│  └───────────────┘  │
           │
│          ▼          │
   ┌───────────────┐     TCP    ┌───────────────┐
│  │ mysql-xxx.jar │──┼────────>│     MySQL     │
   └───────────────┘            └───────────────┘
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘
          JVM
```

使用 JDBC 的好处是：

- 各数据库厂商使用相同的接口，Java 代码不需要针对不同数据库分别开发；
- Java程序编译期仅依赖 java.sql 包，不依赖具体数据库的jar包；
- 可随时替换底层数据库，访问数据库的 Java 代码基本不变。

# JDBC 查询

添加 Maven 依赖：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.47</version>
    <scope>runtime</scope>
</dependency>
```

注意到这里添加依赖的`scope`是`runtime`，因为编译 Java 程序并不需要 MySQL 的这个 jar 包，只有在运行期才需要使用。如果把`runtime`改成`compile`，虽然也能正常编译，但是在 IDE 里写程序的时候，会多出来一大堆类似`com.mysql.jdbc.Connection`这样的类，非常容易与 Java 标准库的 JDBC 接口混淆，所以坚决不要设置为`compile`。

有了驱动，我们还要确保 MySQL 在本机正常运行，并且还需要准备一点数据。这里我们用一个脚本创建数据库和表，然后插入一些数据：

```sql
-- 创建数据库learjdbc:
DROP DATABASE IF EXISTS learnjdbc;
CREATE DATABASE learnjdbc;

-- 创建登录用户learn/口令learnpassword
CREATE USER IF NOT EXISTS learn@'%' IDENTIFIED BY 'learnpassword';
GRANT ALL PRIVILEGES ON learnjdbc.* TO learn@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

-- 创建表students:
USE learnjdbc;
CREATE TABLE students (
  id BIGINT AUTO_INCREMENT NOT NULL,
  name VARCHAR(50) NOT NULL,
  gender TINYINT(1) NOT NULL,
  grade INT NOT NULL,
  score INT NOT NULL,
  PRIMARY KEY(id)
) Engine=INNODB DEFAULT CHARSET=UTF8;

-- 插入初始数据:
INSERT INTO students (name, gender, grade, score) VALUES ('小明', 1, 1, 88);
INSERT INTO students (name, gender, grade, score) VALUES ('小红', 1, 1, 95);
INSERT INTO students (name, gender, grade, score) VALUES ('小军', 0, 1, 93);
INSERT INTO students (name, gender, grade, score) VALUES ('小白', 0, 1, 100);
INSERT INTO students (name, gender, grade, score) VALUES ('小牛', 1, 2, 96);
INSERT INTO students (name, gender, grade, score) VALUES ('小兵', 1, 2, 99);
INSERT INTO students (name, gender, grade, score) VALUES ('小强', 0, 2, 86);
INSERT INTO students (name, gender, grade, score) VALUES ('小乔', 0, 2, 79);
INSERT INTO students (name, gender, grade, score) VALUES ('小青', 1, 3, 85);
INSERT INTO students (name, gender, grade, score) VALUES ('小王', 1, 3, 90);
INSERT INTO students (name, gender, grade, score) VALUES ('小林', 0, 3, 91);
INSERT INTO students (name, gender, grade, score) VALUES ('小贝', 0, 3, 77);
```

加载驱动：

```java
/**
  * 加载驱动有两种方式
  *
  * 1：会导致驱动会注册两次，过度依赖于mysql的api，脱离的mysql的开发包，程序则无法编译
  * 2：驱动只会加载一次，不需要依赖具体的驱动，灵活性高
  *
  * 一般使用第二种方式
  **/

//1.
DriverManager.registerDriver(new com.mysql.jdbc.Driver());

//2.
Class.forName("com.mysql.jdbc.Driver");
```



## JDBC 连接

Connection代表一个JDBC连接，它相当于Java程序到数据库的连接（通常是TCP连接）。打开一个Connection时，需要准备URL、用户名和口令，才能成功连接到数据库。

URL是由数据库厂商指定的格式，例如，MySQL的URL是：

```
jdbc:mysql://<hostname>:<port>/<db>?key1=value1&key2=value2
```

假设数据库运行在本机`localhost`，端口使用标准的`3306`，数据库名称是`learnjdbc`，那么URL如下：

```
jdbc:mysql://localhost:3306/learnjdbc?useSSL=false&characterEncoding=utf8
```

后面的两个参数表示不使用SSL加密，使用UTF-8作为字符编码（注意MySQL的UTF-8是`utf8`）。

要获取数据库连接，使用如下代码：

```java
// JDBC连接的URL, 不同数据库有不同的格式:
String JDBC_URL = "jdbc:mysql://localhost:3306/test";
String JDBC_USER = "root";
String JDBC_PASSWORD = "password";
// 获取连接:
Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD);
// TODO: 访问数据库...
// 关闭连接:
conn.close();
```

核心代码是`DriverManager`提供的静态方法`getConnection()`。`DriverManager`会自动扫描 classpath，找到所有的 JDBC 驱动，然后根据我们传入的 URL 自动挑选一个合适的驱动。

因为 JDBC 连接是一种昂贵的资源，所以使用后要及时释放。使用`try (resource)`来自动释放JDBC连接是一个好方法：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    ...
}
```

## JDBC查询

获取到JDBC连接后，下一步我们就可以查询数据库了。查询数据库分以下几步：

第一步，通过`Connection`提供的`createStatement()`方法创建一个`Statement`对象，用于执行一个查询；

第二步，执行`Statement`对象提供的`executeQuery("SELECT * FROM students")`并传入SQL语句，执行查询并获得返回的结果集，使用`ResultSet`来引用这个结果集；

第三步，反复调用`ResultSet`的`next()`方法并读取每一行结果。

完整查询代码如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (Statement stmt = conn.createStatement()) {
        try (ResultSet rs = stmt.executeQuery("SELECT id, grade, name, gender FROM students WHERE gender=1")) {
            while (rs.next()) {
                long id = rs.getLong(1); // 注意：索引从1开始
                long grade = rs.getLong(2);
                String name = rs.getString(3);
                int gender = rs.getInt(4);
            }
        }
    }
}
```

注意要点：

`Statment`和`ResultSet`都是需要关闭的资源，因此嵌套使用`try (resource)`确保及时关闭；

`rs.next()`用于判断是否有下一行记录，如果有，将自动把当前行移动到下一行（一开始获得`ResultSet`时当前行不是第一行）；

`ResultSet`获取列时，索引从`1`开始而不是`0`；

必须根据`SELECT`的列的对应位置来调用`getLong(1)`，`getString(2)`这些方法，否则对应位置的数据类型不对，将报错。

## SQL注入

使用`Statement`拼字符串非常容易引发 SQL 注入的问题，这是因为 SQL 参数往往是从方法参数传入的。

我们来看一个例子：假设用户登录的验证方法如下：

```java
User login(String name, String pass) {
    ...
    stmt.executeQuery("SELECT * FROM user WHERE login='" + name + "' AND pass='" + pass + "'");
    ...
}
```

其中，参数`name`和`pass`通常都是 Web 页面输入后由程序接收到的。

如果用户的输入是程序期待的值，就可以拼出正确的 SQL。例如：name = `"bob"`，pass = `"1234"`：

```sql
SELECT * FROM user WHERE login='bob' AND pass='1234'
```

但是，如果用户的输入是一个精心构造的字符串，就可以拼出意想不到的SQL，这个 SQL 也是正确的，但它查询的条件不是程序设计的意图。例如：name = `"bob' OR pass="`, pass = `" OR pass='"`：

```sql
SELECT * FROM user WHERE login='bob' OR pass=' AND pass=' OR pass=''
```

这个 SQL 语句执行的时候，根本不用判断口令是否正确，这样一来，登录就形同虚设。

要避免 SQL 注入攻击，一个办法是针对所有字符串参数进行转义，但是转义很麻烦，而且需要在任何使用 SQL 的地方增加转义代码。

### PreparedStatement

还有一个办法就是使用`PreparedStatement`。使用`PreparedStatement`可以*避免SQL注入*的问题，它使用`?`作为占位符，并对参数中的特殊字符进行转义。`PreparedStatement`每次传给数据库的 SQL 语句是相同的，只是占位符的数据不同，能高效利用数据库本身对查询的缓存。上述登录SQL如果用`PreparedStatement`可以改写如下：

```java
User login(String name, String pass) {
    ...
    String sql = "SELECT * FROM user WHERE login=? AND pass=?";
    PreparedStatement ps = conn.prepareStatement(sql);
    ps.setObject(1, name);
    ps.setObject(2, pass);
    ...
}
```

所以，`PreparedStatement`比`Statement`更安全，而且更快。

> PreparedStatement不是将参数简单拼凑成sql，而是做了一些预处理，将参数转换为string，两端加单引号，将参数内的一些特殊字符（换行，单双引号，斜杠等）做转义处理，这样就很大限度的避免了sql注入。

```java
/**
     * Set a parameter to a Java String value. The driver converts this to a SQL
     * VARCHAR or LONGVARCHAR value (depending on the arguments size relative to
     * the driver's limits on VARCHARs) when it sends it to the database.
     * 
     * @param parameterIndex
     *            the first parameter is 1...
     * @param x
     *            the parameter value
     * 
     * @exception SQLException
     *                if a database access error occurs
     */
    public void setString(int parameterIndex, String x) throws SQLException {
        synchronized (checkClosed().getConnectionMutex()) {
            // if the passed string is null, then set this column to null
            if (x == null) {
                setNull(parameterIndex, Types.CHAR);
            } else {
                checkClosed();

                int stringLength = x.length();

                if (this.connection.isNoBackslashEscapesSet()) {
                    // Scan for any nasty chars

                    boolean needsHexEscape = isEscapeNeededForString(x, stringLength);

                    if (!needsHexEscape) {
                        byte[] parameterAsBytes = null;

                        StringBuilder quotedString = new StringBuilder(x.length() + 2);
                        quotedString.append('\'');
                        quotedString.append(x);
                        quotedString.append('\'');

                        if (!this.isLoadDataQuery) {
                            parameterAsBytes = StringUtils.getBytes(quotedString.toString(), this.charConverter, this.charEncoding,
                                    this.connection.getServerCharset(), this.connection.parserKnowsUnicode(), getExceptionInterceptor());
                        } else {
                            // Send with platform character encoding
                            parameterAsBytes = StringUtils.getBytes(quotedString.toString());
                        }

                        setInternal(parameterIndex, parameterAsBytes);
                    } else {
                        byte[] parameterAsBytes = null;

                        if (!this.isLoadDataQuery) {
                            parameterAsBytes = StringUtils.getBytes(x, this.charConverter, this.charEncoding, this.connection.getServerCharset(),
                                    this.connection.parserKnowsUnicode(), getExceptionInterceptor());
                        } else {
                            // Send with platform character encoding
                            parameterAsBytes = StringUtils.getBytes(x);
                        }

                        setBytes(parameterIndex, parameterAsBytes);
                    }

                    return;
                }

                String parameterAsString = x;
                boolean needsQuoted = true;

                if (this.isLoadDataQuery || isEscapeNeededForString(x, stringLength)) {
                    needsQuoted = false; // saves an allocation later

                    StringBuilder buf = new StringBuilder((int) (x.length() * 1.1));

                    buf.append('\'');

                    //
                    // Note: buf.append(char) is _faster_ than appending in blocks, because the block append requires a System.arraycopy().... go figure...
                    //

                    for (int i = 0; i < stringLength; ++i) {
                        char c = x.charAt(i);

                        switch (c) {
                            case 0: /* Must be escaped for 'mysql' */
                                buf.append('\\');
                                buf.append('0');

                                break;

                            case '\n': /* Must be escaped for logs */
                                buf.append('\\');
                                buf.append('n');

                                break;

                            case '\r':
                                buf.append('\\');
                                buf.append('r');

                                break;

                            case '\\':
                                buf.append('\\');
                                buf.append('\\');

                                break;

                            case '\'':
                                buf.append('\\');
                                buf.append('\'');

                                break;

                            case '"': /* Better safe than sorry */
                                if (this.usingAnsiMode) {
                                    buf.append('\\');
                                }

                                buf.append('"');

                                break;

                            case '\032': /* This gives problems on Win32 */
                                buf.append('\\');
                                buf.append('Z');

                                break;

                            case '\u00a5':
                            case '\u20a9':
                                // escape characters interpreted as backslash by mysql
                                if (this.charsetEncoder != null) {
                                    CharBuffer cbuf = CharBuffer.allocate(1);
                                    ByteBuffer bbuf = ByteBuffer.allocate(1);
                                    cbuf.put(c);
                                    cbuf.position(0);
                                    this.charsetEncoder.encode(cbuf, bbuf, true);
                                    if (bbuf.get(0) == '\\') {
                                        buf.append('\\');
                                    }
                                }
                                buf.append(c);
                                break;

                            default:
                                buf.append(c);
                        }
                    }

                    buf.append('\'');

                    parameterAsString = buf.toString();
                }

                byte[] parameterAsBytes = null;

                if (!this.isLoadDataQuery) {
                    if (needsQuoted) {
                        parameterAsBytes = StringUtils.getBytesWrapped(parameterAsString, '\'', '\'', this.charConverter, this.charEncoding,
                                this.connection.getServerCharset(), this.connection.parserKnowsUnicode(), getExceptionInterceptor());
                    } else {
                        parameterAsBytes = StringUtils.getBytes(parameterAsString, this.charConverter, this.charEncoding, this.connection.getServerCharset(),
                                this.connection.parserKnowsUnicode(), getExceptionInterceptor());
                    }
                } else {
                    // Send with platform character encoding
                    parameterAsBytes = StringUtils.getBytes(parameterAsString);
                }

                setInternal(parameterIndex, parameterAsBytes);

                this.parameterTypes[parameterIndex - 1 + getParameterIndexOffset()] = Types.VARCHAR;
            }
        }
    }
```



 使用Java对数据库进行操作时，必须使用`PreparedStatement`，严禁任何通过参数拼字符串的代码！

我们把上面使用`Statement`的代码改为使用`PreparedStatement`：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("SELECT id, grade, name, gender FROM students WHERE gender=? AND grade=?")) {
        ps.setObject(1, "M"); // 注意：索引从1开始
        ps.setObject(2, 3);
        try (ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                long id = rs.getLong("id");
                long grade = rs.getLong("grade");
                String name = rs.getString("name");
                String gender = rs.getString("gender");
            }
        }
    }
}
```

使用`PreparedStatement`和`Statement`稍有不同，必须首先调用`setObject()`设置每个占位符`?`的值，最后获取的仍然是`ResultSet`对象。

另外注意到从结果集读取列时，使用`String`类型的列名比索引要易读，而且不易出错。

注意到 JDBC 查询的返回值总是`ResultSet`，即使我们写这样的聚合查询`SELECT SUM(score) FROM ...`，也需要按结果集读取：

```java
ResultSet rs = ...
if (rs.next()) {
    double sum = rs.getDouble(1);
}
```

## 数据类型

使用JDBC的时候，我们需要在Java数据类型和SQL数据类型之间进行转换。JDBC在`java.sql.Types`定义了一组常量来表示如何映射SQL数据类型，但是平时我们使用的类型通常也就以下几种：

| SQL数据类型   | Java数据类型             |
| :------------ | :----------------------- |
| BIT, BOOL     | boolean                  |
| INTEGER       | int                      |
| BIGINT        | long                     |
| REAL          | float                    |
| FLOAT, DOUBLE | double                   |
| CHAR, VARCHAR | String                   |
| DECIMAL       | BigDecimal               |
| DATE          | java.sql.Date, LocalDate |
| TIME          | java.sql.Time, LocalTime |

注意：只有最新的JDBC驱动才支持`LocalDate`和`LocalTime`。

## 小结

JDBC接口的`Connection`代表一个JDBC连接；

使用JDBC查询时，总是使用`PreparedStatement`进行查询而不是`Statement`；

查询结果总是`ResultSet`，即使使用聚合查询也不例外。

# JDBC 更新

## 插入

插入操作是`INSERT`，即插入一条新记录。通过JDBC进行插入，本质上也是用`PreparedStatement`执行一条SQL语句，不过最后执行的不是`executeQuery()`，而是`executeUpdate()`。示例代码如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement(
            "INSERT INTO students (id, grade, name, gender) VALUES (?,?,?,?)")) {
        ps.setObject(1, 999); // 注意：索引从1开始
        ps.setObject(2, 1); // grade
        ps.setObject(3, "Bob"); // name
        ps.setObject(4, "M"); // gender
        int n = ps.executeUpdate(); // 1
    }
}
```

设置参数与查询是一样的，有几个`?`占位符就必须设置对应的参数。虽然`Statement`也可以执行插入操作，但我们仍然要严格遵循**绝不能手动拼 SQL字符串**的原则，以避免安全漏洞。

当成功执行`executeUpdate()`后，返回值是`int`，表示插入的记录数量。此处总是`1`，因为只插入了一条记录。

## 插入并获取主键

如果数据库的表设置了自增主键，那么在执行`INSERT`语句时，并不需要指定主键，数据库会自动分配主键。对于使用自增主键的程序，有个额外的步骤，就是如何获取插入后的自增主键的值。

要获取自增主键，不能先插入，再查询。因为两条SQL执行期间可能有别的程序也插入了同一个表。获取自增主键的正确写法是在创建`PreparedStatement`的时候，指定一个`RETURN_GENERATED_KEYS`标志位，表示JDBC驱动必须返回插入的自增主键。示例代码如下：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement(
            "INSERT INTO students (grade, name, gender) VALUES (?,?,?)",
            Statement.RETURN_GENERATED_KEYS)) {
        ps.setObject(1, 1); // grade
        ps.setObject(2, "Bob"); // name
        ps.setObject(3, "M"); // gender
        int n = ps.executeUpdate(); // 1
        try (ResultSet rs = ps.getGeneratedKeys()) {
            if (rs.next()) {
                long id = rs.getLong(1); // 注意：索引从1开始
            }
        }
    }
}
```

观察上述代码，有两点注意事项：

一是调用`prepareStatement()`时，第二个参数必须传入常量`Statement.RETURN_GENERATED_KEYS`，否则JDBC驱动不会返回自增主键；

二是执行`executeUpdate()`方法后，必须调用`getGeneratedKeys()`获取一个`ResultSet`对象，这个对象包含了数据库自动生成的主键的值，读取该对象的每一行来获取自增主键的值。如果一次插入多条记录，那么这个`ResultSet`对象就会有多行返回值。如果插入时有多列自增，那么`ResultSet`对象的每一行都会对应多个自增值（自增列不一定必须是主键）。

## 更新

更新操作是`UPDATE`语句，它可以一次更新若干列的记录。更新操作和插入操作在 JDBC 代码的层面上实际上没有区别，除了 SQL 语句不同：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("UPDATE students SET name=? WHERE id=?")) {
        ps.setObject(1, "Bob"); // 注意：索引从1开始
        ps.setObject(2, 999);
        int n = ps.executeUpdate(); // 返回更新的行数
    }
}
```

`executeUpdate()`返回数据库实际更新的行数。返回结果可能是正数，也可能是0（表示没有任何记录更新）。

## 删除

删除操作是`DELETE`语句，它可以一次删除若干列。和更新一样，除了SQL语句不同外，JDBC代码都是相同的：

```java
try (Connection conn = DriverManager.getConnection(JDBC_URL, JDBC_USER, JDBC_PASSWORD)) {
    try (PreparedStatement ps = conn.prepareStatement("DELETE FROM students WHERE id=?")) {
        ps.setObject(1, 999); // 注意：索引从1开始
        int n = ps.executeUpdate(); // 删除的行数
    }
}
```

# JDBC 事务

数据库事务（Transaction）是由若干个 SQL 语句构成的一个操作序列，有点类似于 Java 的`synchronized`同步。数据库系统保证在一个事务中的所有 SQL 要么全部执行成功，要么全部不执行，即数据库事务具有 ACID 特性：

- Atomicity：原子性
- Consistency：一致性
- Isolation：隔离性
- Durability：持久性

数据库事务可以并发执行，而数据库系统从效率考虑，对事务定义了不同的隔离级别。SQL标准定义了4种隔离级别，分别对应可能出现的数据不一致的情况：

| Isolation Level  | 脏读（Dirty Read） | 不可重复读（Non Repeatable Read） | 幻读（Phantom Read） |
| :--------------- | :----------------- | :-------------------------------- | :------------------- |
| Read Uncommitted | Yes                | Yes                               | Yes                  |
| Read Committed   | -                  | Yes                               | Yes                  |
| Repeatable Read  | -                  | -                                 | Yes                  |
| Serializable     | -                  | -                                 | -                    |

对应用程序来说，数据库事务非常重要，很多运行着关键任务的应用程序，都必须依赖数据库事务保证程序的结果正常。

要在JDBC中执行事务，本质上就是如何把多条SQL包裹在一个数据库事务中执行。我们来看JDBC的事务代码：

```java
Connection conn = openConnection();
try {
    // 关闭自动提交:
    conn.setAutoCommit(false);
    // 执行多条SQL语句:
    insert(); update(); delete();
    // 提交事务:
    conn.commit();
} catch (SQLException e) {
    // 回滚事务:
    conn.rollback();
} finally {
    conn.setAutoCommit(true);
    conn.close();
}
```

其中，开启事务的关键代码是`conn.setAutoCommit(false)`，表示关闭自动提交。提交事务的代码在执行完指定的若干条SQL语句后，调用`conn.commit()`。要注意事务不是总能成功，如果事务提交失败，会抛出 SQL 异常（也可能在执行SQL语句的时候就抛出了），此时我们必须捕获并调用`conn.rollback()`回滚事务。最后，在`finally`中通过`conn.setAutoCommit(true)`把`Connection`对象的状态恢复到初始值。

实际上，默认情况下，我们获取到`Connection`连接后，总是处于“自动提交”模式，也就是每执行一条SQL都是作为事务自动执行的，这也是为什么前面几节我们的更新操作总能成功的原因：因为默认有这种“隐式事务”。只要关闭了`Connection`的`autoCommit`，那么就可以在一个事务中执行多条语句，事务以`commit()`方法结束。

如果要设定事务的隔离级别，可以使用如下代码：

```java
// 设定隔离级别为READ COMMITTED:
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
```

如果没有调用上述方法，那么会使用数据库的默认隔离级别。MySQL的默认隔离级别是`REPEATABLE READ`。

# JDBC Batch

使用 JDBC 操作数据库的时候，经常会执行一些批量操作。

例如，一次性给会员增加可用优惠券若干，我们可以执行以下 SQL 代码：

```sql
INSERT INTO coupons (user_id, type, expires) VALUES (123, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (234, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (345, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (456, 'DISCOUNT', '2030-12-31');
...
```

实际上执行 JDBC 时，因为只有占位符参数不同，所以 SQL 实际上是一样的：

```java
for (var params : paramsList) {
    PreparedStatement ps = conn.preparedStatement("INSERT INTO coupons (user_id, type, expires) VALUES (?,?,?)");
    ps.setLong(params.get(0));
    ps.setString(params.get(1));
    ps.setString(params.get(2));
    ps.executeUpdate();
}
```

类似的还有，给每个员工薪水增加 10%～30%：

```sql
UPDATE employees SET salary = salary * ? WHERE id = ?
```

通过一个循环来执行每个`PreparedStatement`虽然可行，但是性能很低。SQL 数据库对 SQL 语句相同，但只有参数不同的若干语句可以作为 batch 执行，即批量执行，这种操作有特别优化，速度远远快于循环执行每个 SQL。

在 JDBC 代码中，我们可以利用 SQL 数据库的这一特性，把同一个 SQL 但参数不同的若干次操作合并为一个 batch 执行。我们以批量插入为例，示例代码如下：

```java
try (PreparedStatement ps = conn.prepareStatement("INSERT INTO students (name, gender, grade, score) VALUES (?, ?, ?, ?)")) {
    // 对同一个PreparedStatement反复设置参数并调用addBatch():
    for (Student s : students) {
        ps.setString(1, s.name);
        ps.setBoolean(2, s.gender);
        ps.setInt(3, s.grade);
        ps.setInt(4, s.score);
        ps.addBatch(); // 添加到batch
    }
    // 执行batch:
    int[] ns = ps.executeBatch();
    for (int n : ns) {
        System.out.println(n + " inserted."); // batch中每个SQL执行的结果数量
    }
}
```

执行 batch 和执行一个 SQL 不同点在于，需要对同一个`PreparedStatement`反复设置参数并调用`addBatch()`，这样就相当于给一个 SQL 加上了多组参数，相当于变成了“多行” SQL。

第二个不同点是调用的不是`executeUpdate()`，而是`executeBatch()`，因为我们设置了多组参数，相应地，返回结果也是多个`int`值，因此返回类型是`int[]`，循环`int[]`数组即可获取每组参数执行后影响的结果数量。

# JDBC 连接池

创建线程是一个昂贵的操作，如果有大量的小任务需要执行，并且频繁地创建和销毁线程，实际上会消耗大量的系统资源，往往创建和消耗线程所耗费的时间比执行任务的时间还长，所以，为了提高效率，可以用线程池。

类似的，在执行 JDBC 的增删改查的操作时，如果每一次操作都来一次打开连接，操作，关闭连接，那么创建和销毁 JDBC 连接的开销就太大了。为了避免频繁地创建和销毁 JDBC 连接，我们可以通过连接池（Connection Pool）复用已经创建好的连接。

JDBC 连接池有一个标准的接口`javax.sql.DataSource`，注意这个类位于 Java 标准库中，但仅仅是接口。要使用 JDBC 连接池，我们必须选择一个 JDBC 连接池的实现。常用的 JDBC 连接池有：

- HikariCP
- C3P0
- BoneCP
- Druid

以 HikariCP 为例，要使用JDBC连接池，先添加 HikariCP 的依赖如下：

```xml
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>2.7.1</version>
</dependency>
```

紧接着，我们需要创建一个`DataSource`实例，这个实例就是连接池：

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/test");
config.setUsername("root");
config.setPassword("password");
config.addDataSourceProperty("connectionTimeout", "1000"); // 连接超时：1秒
config.addDataSourceProperty("idleTimeout", "60000"); // 空闲超时：60秒
config.addDataSourceProperty("maximumPoolSize", "10"); // 最大连接数：10
DataSource ds = new HikariDataSource(config);
```

注意创建`DataSource`也是一个非常昂贵的操作，所以通常`DataSource`实例总是作为一个全局变量存储，并贯穿整个应用程序的生命周期。

有了连接池以后，我们如何使用它呢？和前面的代码类似，只是获取`Connection`时，把`DriverManage.getConnection()`改为`ds.getConnection()`：

```java
try (Connection conn = ds.getConnection()) { // 在此获取连接
    ...
} // 在此“关闭”连接
```

通过连接池获取连接时，并不需要指定JDBC的相关URL、用户名、口令等信息，因为这些信息已经存储在连接池内部了（创建`HikariDataSource`时传入的`HikariConfig`持有这些信息）。一开始，连接池内部并没有连接，所以，第一次调用`ds.getConnection()`，会迫使连接池内部先创建一个`Connection`，再返回给客户端使用。当我们调用`conn.close()`方法时（`在try(resource){...}`结束处），不是真正“关闭”连接，而是释放到连接池中，以便下次获取连接时能直接返回。

因此，连接池内部维护了若干个`Connection`实例，如果调用`ds.getConnection()`，就选择一个空闲连接，并标记它为“正在使用”然后返回，如果对`Connection`调用`close()`，那么就把连接再次标记为“空闲”从而等待下次调用。这样一来，我们就通过连接池维护了少量连接，但可以频繁地执行大量的SQL语句。

通常连接池提供了大量的参数可以配置，例如，维护的最小、最大活动连接数，指定一个连接在空闲一段时间后自动关闭等，需要根据应用程序的负载合理地配置这些参数。此外，大多数连接池都提供了详细的实时状态以便进行监控。

**小结**

数据库连接池是一种复用`Connection`的组件，它可以避免反复创建新连接，提高JDBC代码的运行效率；

可以配置连接池的详细参数并监控连接池。

# 常见面试题

**1、JDBC操作数据库的步骤？**

1. 注册数据库驱动。
2. 建立数据库连接。
3. 创建一个Statement。
4. 执行SQL语句。
5. 处理结果集。
6. 关闭数据库连接

**2、JDBC中的Statement 和PreparedStatement，CallableStatement的区别？**

区别：

- PreparedStatement是预编译的SQL语句，效率高于Statement。
- PreparedStatement支持?操作符，相对于Statement更加灵活。
- PreparedStatement可以防止SQL注入，安全性高于Statement。
- CallableStatement适用于执行存储过程。

