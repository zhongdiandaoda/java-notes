# Jconsole

**Jconsole(Java Monitoring and Management Console)是一种基于JMX的可视化监视，管理工具。**

**位于JDK/bin目录下，通过 “jconsole.exe”启动JConsole后，可自动检索处本机的虚拟机进程。**

测试程序为：

```java
package monitorTest;

import java.util.ArrayList;
import java.util.List;

public class JconsoleDemo {

    static class OOMObject
    {
        public byte[] placeholder=new byte[64*1024];
    }
    public static void fillheap(int num)throws InterruptedException
    {
        List<OOMObject> list=new ArrayList<OOMObject>();
        for(int i=0;i<num;i++)
        {
            Thread.sleep(50);//稍作延时
            list.add(new OOMObject());
        }
        System.gc();
    }


    public static void main(String[] args) throws Exception{
        // TODO 自动生成的方法存根
        fillheap(1000);
    }

}
```

代码的含义是64KB/50毫秒的速度向Java堆中填充数据，一共执行1000次。

设置虚拟机启动参数： -Xms100m -Xmx100m -XX:+UseSerialGC。表示初始堆大小和最大堆大小均为100MB，GC算法为Serial垃圾收集算法，即新生代采用标记复制算法，老生代采用标记整理算法。



