# java文件流_文件-InfoQ写作社区
一、文件
----

#### 1、文件的含义

文件是保存数据的地方，比如 word 文档，txt 文本文件，视频，图片等都是文件。

#### 2、文件流

文件在程序中是以流的形式来操作的。

![](https://static001.geekbang.org/infoq/81/811dbd6e32311fe8e1a5a0ac3a20a9c7.png)

输入流：数据从文件到程序（内存）的路径。（在程序中读取文件数据）

输出流：数据从程序（内存）到文件的路径。（在程序中将数据写入文件）

二、常用的文件操作
---------

1、创建文件对象相关构造器和方法

相关构造器：

new File(String filePath) 根据路径创建 File 对象

new File(File parent,String child) 根据父目录文件+子路径创建 File 对象

new File(tring parent,String child) 根据父目录+子路径创建 File 对象

方法：

createNewFile 创建新文件

2、创建文件案例演示（三种创建方法）

```
import org.junit.jupiter.api.Test;

import java.io.File;
import java.io.IOException;

public class FileCreate {
    public static void main(String[] args) {

    }

    @Test
    public void create01() {
        String filePath = "e:\\news1.txt";
        File file = new File(filePath);
        try {
            file.createNewFile();
            System.out.println("文件创建成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void create02() {
        File parentFile = new File("e:\\");
        String fileName = "news2.txt";
        File file = new File(parentFile, fileName);
        try {
            file.createNewFile();
            System.out.println("文件创建成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Test
    public void create03() {
        String parentPath = "e:\\";
        String fileName = "news3.txt";
        File file = new File(parentPath, fileName);
        try {
            file.createNewFile();
            System.out.println("文件创建成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

3、获取文件相关信息的方法

“文件名字” — getName()

“文件绝对路径” — getAbsolutePath()

“文件父级目录” — getParent()

“文件大小(字节)” — length()

“文件是否存在” — exists()

“是不是一个文件” — isFile()

“是不是一个目录” — isDirectory()

4、获取文件相关信息方法案例演示

```
import org.junit.jupiter.api.Test;

import java.io.File;

public class FileInformation {
    public static void main(String[] args) {

    }

    @Test
    public void info(){
        File file = new File("e:\\news1.txt");
        System.out.println("文件名字" + file.getName());
        System.out.println("文件绝对路径" + file.getAbsolutePath());
        System.out.println("文件父级目录" + file.getParent());
        System.out.println("文件大小(字节)" + file.length());
        System.out.println("文件是否存在" + file.exists());
        System.out.println("是不是一个文件" + file.isFile());
        System.out.println("是不是一个目录" + file.isDirectory());
    }
}
```

5、目录的操作与删除

mkdir()：创建一级目录

mkdirs()：创建多级目录

delete()：删除目录或文件

6、应用案例演示

(1)判断 e:\\news1.txt 是否存在，如果存在就删除

(2)判断 d:\\demo02 是否存在，如果存在就删除，否则提示不存在

(3)判断 d:\\demo\\a\\b\\c 是否存在，如果存在就提示存在，否则创建

```
import org.junit.jupiter.api.Test;

import java.io.File;

public class Directory_ {
    public static void main(String[] args) {

    }

    @Test
    public void m1() {
        String pathName = "e:\\news1.txt";
        File file = new File(pathName);
        if (file.exists()) {
            if (file.delete()) {
                System.out.println("删除成功");
            } else {
                System.out.println("删除失败");
            }
        } else {
            System.out.println("该文件不存在");
        }
    }

    @Test
    public void m2() {
        String pathName = "d:\\demo02";
        File file = new File(pathName);
        if (file.exists()) {
            if (file.delete()) {
                System.out.println("删除成功");
            } else {
                System.out.println("删除失败");
            }
        } else {
            System.out.println("该目录不存在");
        }
    }

    @Test
    public void m3() {
        String directoryPath = "d:\\demo\\a\\b\\c";
        File file = new File(directoryPath);
        if (file.exists()) {
            System.out.println(directoryPath + "已经存在");
        } else {
            if (file.mkdirs()) {
                System.out.println(directoryPath + "创建成功");
            } else {
                System.out.println(directoryPath + "创建失败");
            }
        }
    }
}
```

三、IO 流原理及流的分类
-------------

1、Java IO 流原理

(1) I／O 是 Input／Output 的缩写，l／O 技术是非常实用的技术，用于处理数据传输。如读／写文件，网络通讯等。

(2) Java 程序中，对于数据的输入／输出操作以”流（stream）”的方式进行。

(3) java.io 包下提供了各种“流”类和接口，用以获取不同种类的数据，并通过方法输入或输出数据。

(4) 输入 input：读取外部数据（磁盘、光盘等存储设备的数据）到程序（内存）中。

(5) 输出 output：将程序（内存）数据输出到磁盘、光盘等存储设备中。

2、流的分类

按操作数据单位不同分为：字节流（8 bit）二进制文件，字符流（按字符）文本文件

按数据流的流向不同分为：输入流，输出流

按流的角色的不同分为：节点流，处理流／包装流

![](https://static001.geekbang.org/infoq/50/502bca37d563b3f02e9283c60b1fa9be.png)

1） Java 的 IO 流共涉及 40 多个类，实际上非常规则，都是从如上 4 个抽象基类派生的。2）由这四个类派生出来的子类名称都是以其父类名作为子类名后缀。

四、IO 流程图
--------

![](https://static001.geekbang.org/infoq/cc/ccc75a80470bc705eda9586b97c46589.png)

五、FileInputStream 字节输入流
-----------------------

可以用单个字节来读取文件，也可以用字节数组来读取文件。

以字节数组为例：

1、确定文件路径

String filePath = “e:\\hello.txt”;

2、引入 FileInputStream

FileInputStream fileInputStream = new FileInputStream(filePath);

3、定义字节数组

byte\[\] buf = new byte\[8\];

4、定义读取字节数

int data = 0;

5、读取数据

```
while ((data = fileInputStream.read(buf)) != -1) {
	System.out.println(new String(buf, 0, data));
}
```

用 while 循环读取，条件 = -1 时说明读取完毕

6、关闭流

fileInputStream.close();

7、完整版

```
public void readFile02() {
        String filePath = "e:\\hello.txt";
        FileInputStream fileInputStream = null;
        byte[] buf = new byte[8];
        int data = 0;
        try {
            fileInputStream = new FileInputStream(filePath);
            while ((data = fileInputStream.read(buf)) != -1) {
                System.out.println(new String(buf, 0, data));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

六、FileOutputStream 字节输出流
------------------------

可以用单个字节来写入文件，也可以用字节数组来写入文件。

以字节数组为例：

1、确定文件路径

String filePath = “e:\\a.txt”;

2、引入 FileOutputStream

FileOutputStream fileOutputStream = new FileOutputStream(filePath);//运行时会覆盖上一次的结果

FileOutputStream fileOutputStream = new FileOutputStream(filePath, true); //运行时不会覆盖上一次的结果

3、写入数据

String str = “hello,world!”;

fileOutputStream.write(str.getBytes());

4、关闭流

fileOutputStream.close();

5、完整版

```
public void writeFile() {
        String filePath = "e:\\a.txt";
        FileOutputStream fileOutputStream = null;
        try {
            
            fileOutputStream = new FileOutputStream(filePath);
            
            fileOutputStream = new FileOutputStream(filePath, true);
            
            fileOutputStream.write('H');
            
            String str = "hello,world!";
            
            fileOutputStream.write(str.getBytes());
            
            fileOutputStream.write(str.getBytes(), 0, str.length());
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileOutputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```

七、用字节流拷贝文件
----------

1、确定原文件与新文件路径

String filePath = “e:\\1.jpg”;

String newFilePath = “d:\\1.jpg”;

2、引入 FileInputStream 与 FileOutputStream

FileInputStream fileInputStream = new FileInputStream(filePath);

fileOutputStream fileOutputStream = new FileOutputStream(newFilePath);

3、定义字节数组

byte\[\] buf = new byte\[1024\];

4、定义读取字节数

int data = 0;

5、拷贝数据

```
while ((data = fileInputStream.read(buf)) != -1) {
	fileOutputStream.write(buf, 0, data);
}
```

6、关闭流

fileInputStream.close();fileOutputStream.close();

7、完整版

```
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;

public class FileCopy {
    public static void main(String[] args) {
        String filePath = "e:\\1.jpg";
        String newFilePath = "d:\\1.jpg";
        FileInputStream fileInputStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            fileInputStream = new FileInputStream(filePath);
            fileOutputStream = new FileOutputStream(newFilePath);
            byte[] buf = new byte[1024];
            int data = 0;
            while ((data = fileInputStream.read(buf)) != -1) {
                fileOutputStream.write(buf, 0, data);
            }
            System.out.println("拷贝成功");
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fileInputStream != null){
                    fileInputStream.close();
                }
                if (fileOutputStream != null){
                    fileOutputStream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

八、FileReader 字符输入流
------------------

可以用单个字符来读取文件，也可以用字符数组来读取文件。

1、FileReader 相关方法：

（1） new FileReader(File/String)

（2） read：每次读取单个字符，返回该字符，如果到文件末尾返回-1

（3） read（char［］）：批量读取多个字符到数组，返回读取到的字符数，如果到文件末尾返回-1

相关 API：

（1） new String（char［］）：将 char［］转换成 String

（2） new String（char［］，off，len）：将 char［］的指定部分转换成 String

2、以字符数组来读取文件

```
import java.io.FileReader;
import java.io.IOException;

public class FileReader_ {
    public static void main(String[] args) {
        String filePath = "e:\\story.txt";
        FileReader fileReader = null;
        char[] chars = new char[100];
        int data = 0;
        try {
            fileReader = new FileReader(filePath);
            while ((data = fileReader.read(chars)) != -1) {
                System.out.print(new String(chars, 0, data));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

九、FileWriter 字符输入流
------------------

1、 FileWriter 常用方法

1） new FileWriter（File／String）：覆盖模式，相当于流的指针在首端

2） new FileWriter（File／String，true）：追加模式，相当于流的指针在尾端

3） write（int）：写入单个字符

4） write（char［］）：写入指定数组

5） write（char［］，off，len）：写入指定数组的指定部分

6） write（string）：写入整个字符串

7） write（string，off，len）：写入字符串的指定部分

相关 API： String 类： toCharArray：将 String 转换成 char［］

＞注意：

FileWriter 使用后，必须要关闭（close）或刷新（flush），否则写入不到指定的文件！

2、写入文件案例

```
import java.io.FileWriter;
import java.io.IOException;

public class FileWriter_ {
    public static void main(String[] args) {
        String filePath = "e:\\1.txt";
        FileWriter fileWriter = null;
        try {
            fileWriter = new FileWriter(filePath);
            fileWriter.write('H');

            char[] chars = {'a', 'b', 'c'};
            fileWriter.write(chars);

            fileWriter.write("大事发生".toCharArray(), 0, 2);

            fileWriter.write("巅峰时代");

            fileWriter.write("法国电视", 0, 3);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                fileWriter.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```