# 百度工程师教你玩转设计模式（适配器模式）_Java_百度Geek说_InfoQ写作社区
![](assets/139497c2e4fb5a697e250a4b8cab711f.png)

作者 | 北极星小组

在现实生活中，经常会遇到两个“对象” 因为接口不兼容而不能一起工作的场景，这时需要第三者进行适配，如：国内的充电线插头不一定适用国外的插座需要借助转接头、SD 卡无法直接链接电脑需要借助读卡器、用直流电的笔记本电脑接交流电源时需要一个电源适配器等。

在软件设计中，需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但它们与当前系统的接口规范不兼容，如果重新开发这些组件成本又很高，这时用适配器模式能很好地解决这些问题。

适配器模式(Adapter Pattern)：是指将某个类的接口转化成客户端期望的另一个接口，主要目的是兼容性，让原本因接口不匹配不能工作的两个类可以协同工作。

![](assets/a0aef6d734e63eb19cfe5ad394ac0052.png)

****△适配器示意图****

适配器模式包含如下角色：

*   Target：目标抽象类， 定义客户要使用的接口
    
*   Adapter：适配器类，将 Adaptee 的接口进行适配转换
    
*   Adaptee：适配者类，需要被转换接口的对象
    
*   Client：客户类，通过适配器接口 Target 去使用 Adaptee 的功能
    

![](assets/47332696804bafcb6124223b86ed4c19.png)

**△对象适配器**

![](assets/c5fd2e0d8d0019e16f3316698aaf7aff.png)

**△类适配器**

微软 office 文档有两种数据格式，即 office2007(OOXML 格式)和 Office2003(二进制格式)。在新业务场景中，因现有系统已经具备 Office2007 文档处理功能组件（数据、阅读器等），现在需要在此基础上扩充支持 office2003 格式的文档。即复用现有的 office2007 组件类，但接口与复用环境要求不一致，此时可以使用适配器模式。

示例使用对象适配器方式，忽略业务处理细节，仅做流程上的抽象来表明适配器模式的使用，根据模式结构抽象出各个角色类：

*   Doc2003 目标抽象类
    
*   Doc2007 适配者类
    
*   DocAdapter 适配器类
    
*   Document 客户类
    

```
// Document客户类使用方
public class Document {
  public void View(Doc2003 doc) {
    doc.show()
  }
}

// Doc2003接口抽象
public interface IDoc2003 {
    void show();
}

// Doc2007适配者类
public class Doc2007 {
    public void show() {
        System.out.println("office2007标准处理流程");
    }
}

// DocAdapter适配器类
public class DocAdapter implements IDoc2003 {
    private Doc2007 doc2007;
    public DocAdapter(Doc2007 doc2007){
        this.doc2007 = doc2007;
    }
    
    @Override
    public void show() {
        System.out.println("适配器：这里省略了适配到2007的一堆适配逻辑...");
        doc2007.show();
    }
}

// 测试类
public class Test {
  public static void main(String[] args) {
    Document document = new Document();
    Doc2003  doc2003 = new DocAdapter(new Doc2007()) ; 
    document.view(doc2003);
  }
}
```

在实际的研发过程中，我们经常会需要对依赖组件进行更新迭代。在替换时，相关调用代码，往往分布在非常多的地方，并且由于调用方法与参数不一致，逐个修改工作量大，且存在遗漏风险，这时候可以使用类适配器，将接口统一，减少相关代码的改动。

示例使用类适配器方式，忽略业务处理细节，仅做流程上的抽象来表明适配器模式的使用，根据模式结构抽象出各个角色类：

*   AHandler 目标抽象类
    
*   BHandler 适配者类
    
*   BAdapter 适配器类
    
*   Client 客户类
    

```
//原依赖库-目标抽象类
public class AHandler {
    public void operation() {}
}

//新依赖库-适配者类
public class BHandler {
    public void action() {}
}

//适配器类
public class BAdapter extends BHandler {
    public void operation() {
        ...
        super.action();
        ...
    }
}

//客户类
public class Client {
    public static void main(String[] args) {
        //原调用
        AHandler aHandler = new AHandler();
        aHandler.operation();
        
        //新调用
        BAdapter bAdapter = new BAdapter();
        bAdapter.operation();
    }
}
```

在实际的开发过程中，一个接口有大量的方法，但是对应的不同类只需要关注部分方法，其他无关的方法全都实现过于繁琐，尤其是涉及的实现类过多的情况。

例如，现有一个需要的目标接口对象 Target，定义了大量相关的方法。但是在实际使用过程只需分别关注其中部分方法，而不是全部实现。在此场景中：

被依赖的目标对象：TargetObj

适配器：Adapter

客户端：Client

```
// 目标对象：定义了大量的相关方法
public interface TargetObj {
    void operation1();
    void operation2();
    void operation3();
    void operation4();
    void operation5();
}

// 适配器：将目标接口定义的方法全部做默认实现
public abstract class Adapter implements TargetObj {
    void operation1(){}
    void operation2(){}
    void operation3(){}
    void operation4(){}
    void operation5(){}
}

// 客户端：采用匿名内部类的方式实现需要的接口即可完成适配
public class Client {

    public static void main(String[] args) {
        Adapter adapter1 = new Adapter() {
            @Override
            public void operation3() {
            // 仅仅实现需要关注的方法即可
            System.out.println("operation3")
            }
        }
        
        Adapter adapter2 = new Adapter() {
            @Override
            public void operation5() {
            // 仅仅实现需要关注的方法即可
            System.out.println("operation5")
            }
        }
        
        adapter1.operation3();
        adapter2.operation5();

    }

}
```

以上介绍了三种使用适配器模式的场景，适配器模式本质上是将现有的不兼容的接口转换为需要的接口，不需要改变原来的代码结构即可实现新的功能，通常实现方式上有：

1.  对象适配器：通过对象的组合来达到适配目的；
    
2.  类适配器：通过类的继承或者接口的实现来达到适配目的；
    
3.  接口适配器：通过实现接口的方式转换来达到适配的目的；
    

适配器模式是在现有的类和系统都不易修改的情况下使用，在系统设计之初慎用适配器模式，这样会导致代码可读性变差，不易维护。

\---------- END ----------

推荐阅读【技术加油站】系列：

[揭秘百度智能测试在测试评估领域实践](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247534777%26idx%3D1%26sn%3Dfa007c2649e48d8ab6d43d54fd062f04%26chksm%3Dc03e78c5f749f1d380eca98e67a97d60211c21bdcab4d2e5f3937bb7a41a335b05d8797ec773%26scene%3D21%23wechat_redirect)  

[百度工程师带你探秘C++内存管理（理论篇）](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247532289%26idx%3D1%26sn%3D5c5ab524a3047cf92b7faf6969b7dfbf%26chksm%3Dc03e4f7df749c66b0738ebd3234d1d0460d63a3aa52d55ca33a9dbba6fdf50c6a356b982114b%26scene%3D21%23wechat_redirect)  

[从零到一了解APP速度测评](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247531838%26idx%3D1%26sn%3D7e37cf3f6262b87597bacc1d0adc8157%26chksm%3Dc03e4d42f749c454c94ea49bc1cb170e28d7fcd6a36cd2c42d668e67a66e19555ebe38e0691f%26scene%3D21%23wechat_redirect)  

[百度工程师教你玩转设计模式（工厂模式）](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247528403%26idx%3D1%26sn%3D0e2263062ab6c9d87822406b35f675d2%26chksm%3Dc03e5faff749d6b9bc719ad14d71e05d99c9aee464a5ab400ff4bd92170b994ceaa886301cf2%26scene%3D21%23wechat_redirect)  

[揭秘百度智能测试在测试分析领域实践](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247527603%26idx%3D1%26sn%3D63ab93ac854dea7ee8661e475528183f%26chksm%3Dc03e5ccff749d5d9097869215413ead68c802f262bcd59ec9ee2ae9bef7f94a38dc8f444305c%26scene%3D21%23wechat_redirect)  

[百度用户产品流批一体的实时数仓实践](https://xie.infoq.cn/link?target=http%3A%2F%2Fmp.weixin.qq.com%2Fs%3F__biz%3DMzg5MjU0NTI5OQ%3D%3D%26mid%3D2247526320%26idx%3D1%26sn%3D661ba569e8fb39aeead346e7ab5f8ae8%26chksm%3Dc03e57ccf749dedae885a7204f3b686f8dd3c0ac0e79793e2996ab89766697a14b08e2c032d3%26scene%3D21%23wechat_redirect)  
