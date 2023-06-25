# Java进阶(二十二)使用FileOutputStream写入文件_9月月更_No Silver Bullet_InfoQ写作社区
在 Java 中，文件输出流是一种用于处理原始二进制数据的字节流类。为了将数据写入到文件中，必须将数据转换为字节，并保存到文件。请参阅下面的完整的例子。

```
public class WriteFileExample {
public static void main(String[] args) {
 
FileOutputStream fop = null;
File file;
String content = "This is the text content";
 
try {
 
	file = new File("c:/newfile.txt");
	fop = new FileOutputStream(file);
 
	
	if (!file.exists()) {
		file.createNewFile();
	}
 
	
	byte[] contentInBytes = content.getBytes();
 
	fop.write(contentInBytes);
	fop.flush();
	fop.close();
 
	System.out.println("Done");
 
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
  	try {
 	   if (fop != null) {
 	     fop.close();
 	   }
  	} catch (IOException e) {
  	  e.printStackTrace();
 	 }
	}
}
}
```

      更新的 JDK7 例如，使用新的“尝试资源关闭”的方法来轻松处理文件。

```
public class WriteFileExample {
public static void main(String[] args) {
 
  File file = new File("c:/newfile.txt");
  String content = "This is the text content";

  try (FileOutputStream fop = new FileOutputStream(file)) {

    
    if (!file.exists()) {
    	file.createNewFile();
    }

    
    byte[] contentInBytes = content.getBytes();

    fop.write(contentInBytes);
    fop.flush();
    fop.close();

    System.out.println("Done");

  } catch (IOException e) {
  	e.printStackTrace();
  }
  }
}
```

      在一次编码过程中，有一个现象一直困扰着自己，经过后台的不断调试，才发现原来有时候字符串的空非空。测试代码如下：

```
String medname  =  request.getString("medname").trim();
logger.info("medname.length():" + medname.length());
logger.info("mednameisNULL:" + (medname == null));
```

      当自己在前台什么都不输入的时候，输出结果如下:

![](_assets/de84014051c616606e01d9c0669624ba.jpeg.jpg)

      这样自己就感到费解了,命名自己什么都没有输入啊,并且经过`trim()`【trim():去掉字符串首尾的空格。】方法的操作，按理说应该为空才对。

      然后自己想，是不是因为没输入其实代表的是输入的空字符串，而空字符串不同于 null。于是自己就写了如下测试语句:

```
logger.info("mednam:" + (medname == ""));
```

      测试结果如下:

![](_assets/28d97d91e859a690f433c8391c5a1fc3.jpeg.jpg)

      果然印证了自己的想法。其果然是一个空字符串!

      以后得注意一下这个问题了，否则会很容易留下 BUG 的。

    由于项目需求，自己需要将带有链接的标签去除，例如

<a href="/zhaoyao/17-66.html">头晕</a>，转换后的文档为头晕。

    由于说明书数量太大(100,569)自己需要采用批处理的方式进行操作。以后用户访问的就是批处理后的文档。故采用正则表达式的形式进行文档处理。

要读取文档内 10w 多条的数据，可按照 3 步走战略:

1.  外层循环利用文件过滤器读取文件夹内所有符合条件的文件。
    
2.  读取每一个筛选到的文件，利用正则表达式去除超链接符号。
    
3.  将每一个处理过的文件重写回源文件。
    

​

​

​