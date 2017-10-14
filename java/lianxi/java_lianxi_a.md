> 定义一个Controller注解，用于声明class。一个Action注解，声明方法，Action注解中包含字符串例如，"get:/customer"。扫描有注解的类和方法，保存在Set中。

> 获取当前的class loader，获取类路径下的class和jar文件，打印所有的class，并将所有class加载进内存。

使用Set定义的接口，完成两个集合的求并集、交集、差集。

获取当前资源路径
``` java
((URLClassLoader) MainClass.class.getClassLoader()).getURLs();
```
从资源路径下读取.properties文件，通过Properties的方法打印所有键值。

通过Iterator遍历Collection
```java
List<String> strs = new ArrayList<>();
strs.add("a");
strs.add("b");
strs.add("c");

Iterator<String> iter = strs.iterator();
while (iter.hasNext()) {
    String tmp = iter.next();
    System.out.println(tmp);
}
```

打印一个类中的所有方法
```java

class Param {
    private Map<String , Object> paramMap;

    public Param(Map<String, Object> paramMap) {
        this.paramMap = paramMap;
    }

    public long getLong(String name) {
        return 1;
    }

    public Map<String, Object> getMap() {
        return paramMap;
    }
}
public class TestMain {

    public static void main(String[] args) {
        try {
            //Class<?> cl = Class.forName("Param.class");
            System.out.println(Param.class.getName());
            Class<?> cl = Class.forName(Param.class.getName());
            while (cl != null) {
                for (Method method : cl.getDeclaredMethods()) {
                    System.out.println(Modifier.toString(method.getModifiers()) + " "
                            + method.getReturnType().getCanonicalName() + " "
                            + method.getName() + Arrays.toString(method.getParameters()));
                }

                cl = cl.getSuperclass();
            }

        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }

    }
}
```

打印类中所有的变量，包括类型，名称和值，修改private成员变量的值。
```java
class TestClass {
    private int anInt;
    public String str;
}
public class TestMain {

    public static void main(String[] args) {
        try {
            Class<?> cl = Class.forName("TestClass");
            TestClass tc = (TestClass) cl.newInstance();
            for (Field field : tc.getClass().getDeclaredFields()) {
                field.setAccessible(true);
                System.out.println(Modifier.toString(field.getModifiers()) + " " + field.getType().getSimpleName() + " " + field.getName() + " : " + field.get((Object) tc));
            }

            Field fld = cl.getDeclaredField("anInt");
            fld.setAccessible(true);
            fld.setInt((Object) tc, 123);
            System.out.println(fld.getInt((Object)tc));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}
```

通过反射调用方法
```java
class TestClass {
    private int anInt;
    public String str;

    public void setStr(String str) {
        this.str = str;
    }
}
public class TestMain {

    public static void main(String[] args) {
        try {
            Class<?> cl = Class.forName("TestClass");
            TestClass tc = new TestClass();
            Method method = cl.getDeclaredMethod("setStr", String.class);
            method.invoke(tc, "hello");

            Field field = cl.getDeclaredField("str");
            String str = (String)field.get(tc);
            System.out.println(str);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}
```

构造对象
```java
class TestClass {
    private int anInt;
    public String str;

    public TestClass() {
        anInt = 1;
        str = "a";
    }

    public TestClass(int anInt, String str) {
        this.anInt = anInt;
        this.str = str;
    }

    public void setStr(String str) {
        this.str = str;
    }

    public String toString() {
        return "anInt: " + anInt + "; str : " + str;
    }
}
public class TestMain {

    public static void main(String[] args) {
        try {
            Class<?> cl = Class.forName("TestClass");
            //使用无参构造函数
            TestClass tc = (TestClass) cl.newInstance();
            System.out.println(tc);
            //使用带参数构造函数
            Constructor constr = cl.getConstructor(int.class, String.class);
            Object obj = constr.newInstance(33, "hello");
            System.out.println(obj);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

    }
}
```

实现Arrays类的copyOf方法，用这个方法扩展一个已经满了的数组。
```java
    public static Object[] badCopyOf(Object[] array, int newLength) {
        Object[] newArray = new Object[newLength];
        for (int i = 0; i < Math.min(array.length, newLength); i++) {
            newArray[i] = array[i];
        }
        return newArray;
    }
```
> 这个方法是有问题的，该方法返回的是Object[]，而一个Object[]数组是不能转化为其他类型的数组的。Java数组会记住他的元素类型，也就是创建数组时new表达式中的类型。将其他类型数组临时转化为Object[]，再转回来是可以的。但是一开始就是Object[]类型，则不能转化为其他类型。

正确版本：
```java
    public static Object goodCopyOf(Object array, int newLength) {
        Class<?> cl = array.getClass();
        if (!cl.isArray()) {
            return null;
        }

        Class<?> componentType = cl.getComponentType();
        int length = Array.getLength(array);
        Object newArray = Array.newInstance(componentType, newLength);
        for (int i = 0; i < Math.min(length, newLength); i++) {
            Array.set(newArray, i, Array.get(array, i));
        }
        return newArray;
    }
```
***
