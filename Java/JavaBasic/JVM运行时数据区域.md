## 运行时数据区域

<img src="D:/OneDrive/_mine/docsify/_img/fpI4PaPeoDLx.png" alt="mark" style="zoom:50%;" />

1. 程序计数器（PC）：指向下一条**字节码指令**的地址，线程私有，如果正在执行Native方法，则 PC 值为空；


2. 虚拟机栈：存放描述方法调用的数据结构**栈帧**，包含局部变量表、操作数栈、动态链接、返回地址等信息，一个方法的执行与结束对应着栈帧出入虚拟机栈；
3. 本地方法栈：作用类似于虚拟机栈，为本地方法服务；HotSpot VM将二者合二为一了；


> 上面仨都是随着线程的创建而创建；


4. 堆：随着虚拟机的启动而创建，是虚拟机管理的最大的一块内存，也是GC主要回收的内存区域，所有线程共享；目的就是存放对象实例，但不是所有的对象都存在堆中；
   - 逃逸分析，一种 JIT 优化技术，可以允许直接在栈上分配对象，减少堆的压力；
   
5. 方法区：用于存储已被虚拟机加载的类信息、常量池、静态变量、JIT编译后的代码缓存等数据；
   - 以前也称为“永久代”，是因为HotSpot将GC扩展到方法区，目的是为了让GC覆盖到方法区的回收；
   - JDK 7 中将常量池和静态变量移至堆中；
   - JDK 8 中将剩余内容(类结构信息等)移至**元空间**中；
6. 直接内存：非 JVM 管理的操作系统内存。

<img src="D:/OneDrive/_mine/docsify/_img/xygINplYKcFU.png" alt="mark" style="zoom: 67%;" />

- 运行时常量池：包含了编译期间各种**字面量**和**符号引用**；
  - 字面量：文本字符串，常量值等；
  - 符号引用：包含类和接口的全限定类名，属性和方法的名称及描述符；

- 类加载后形成的 Class 对象位于堆中，FGC 会清理元空间。

---

## 还原OOM异常

　　除了 PC 以外的上述所有区域都会抛出 OutOfMemoryError ，其中栈比较特殊，当线程申请栈空间失败时抛出 OOM ，当线程请求的栈深度大于虚拟机所允许的最大深度时抛出 StackOverflowError；

> 测试环境 JDK1.8

- 栈溢出
  - 思路：要么撑大局部变量表，要么撑大栈深度；

```java
/**
 * -Xss128k
 */
public class VMSOF {
    private int stackLength = 1;

    public void stackLeak() {
        stackLength++;
        stackLeak();
    }

    public static void main(String[] args) throws Throwable {
        VMSOF oom = new VMSOF();
        try {
            oom.stackLeak();
        } catch (Throwable e) {
            System.out.println("stack length:" + oom.stackLength);
            throw e;
        }
    }
}

```

![mark](D:/OneDrive/_mine/docsify/_img/QYIRTj94zyGe.png)

- 常量池溢出
  - 思路：JDK7之后 常量池移到了堆中，所以要对堆大小进行设置；

```java
/**
 * -Xmx2m
 */
public class MethodAreaOOM {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        int i = 0;
        while (true) {
            list.add(String.valueOf(i++).intern());
        }
    }
}
```

![mark](D:/OneDrive/_mine/docsify/_img/NisOBtISSPoY.png)

- 元空间溢出
  - 思路：元空间里主要存放类信息，可以不断创建Class对象；

```java
/**
 *-XX:MetaspaceSize=10m
 *-XX:MaxMetaspaceSize=10m
 */
public class MetaSpaceOOMTest {

    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    return methodProxy.invokeSuper(o, args);
                }
            });
            enhancer.create();
        }
    }
    
    static class OOMObject {
    }
}
```

- 堆溢出
  - 思路：一直new对象且保持其引用；

```java
/**
 *-Xms10m -Xmx10m
 */
public class HeapOOM {
    static class OOMObject {
    }
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<OOMObject>();
        while (true) {
            list.add(new OOMObject());
        }
    }
}
```

![mark](D:/OneDrive/_mine/docsify/_img/2Vba35K8EFK1.png)

