# 【架构设计】理解软件设计中的SOLID原则
前言
--

在软件架构设计领域，有一个大名鼎鼎的设计原则——SOLID 原则，它是由由 Robert C. Martin（也称为 Uncle Bob）提出的，指导我们写出可维护、可以测试、高扩展、高内聚、低耦合的代码。是不是很牛，但是你们都理解这个设计原则吗，如果理解不深入的话，更这我通过 JAVA 示例深入浅出的明白这个重要的原则吧。SOLID 实际上是由 5 条原则组成, 我们逐一介绍。

**S**：单一职责原则（SRP）

**O** : 开闭原则 (OSP)

**L** : 里氏替换原则 (LSP)

**I**：接口隔离原则（ISP）

**D**：依赖倒置原则（DIP）

> 欢迎关注个人公众号【JAVA 旭阳】交流学习

单一职责原则（SRP）
-----------

这个原则指出“一个类应该只有一个改变的理由”，这意味着每个类都应该有单一的责任或单一的目的。

举个例子来理解其中的核心思想，假设有一个`BankService`的类需要执行以下操作：

1.  存钱
    
2.  取钱
    
3.  打印通票簿
    
4.  获取贷款信息
    
5.  发送一次性密码
    

```
package com.alvin.solid.srp;


public class BankService {

    // 存钱
    public long deposit(long amount, String accountNo) {
        //deposit amount
        return 0;
    }

    // 取钱
    public long withDraw(long amount, String accountNo) {
        //withdraw amount
        return 0;
    }

    // 打印通票簿
    public void printPassbook() {
        //update transaction info in passbook
    }

    // 获取贷款信息
    public void getLoanInterestInfo(String loanType) {
        if (loanType.equals("homeLoan")) {
            //do some job
        }
        if (loanType.equals("personalLoan")) {
            //do some job
        }
        if (loanType.equals("car")) {
            //do some job
        }
    }

    // 发送一次性密码
    public void sendOTP(String medium) {
        if (medium.equals("email")) {
            //write email related logic
            //use JavaMailSenderAPI
        }
    }

}
```

**现在我们来看看这么写会带来什么问题？**

比如对于获取贷款信息 `getLoanInterestInfo()` 方法，现在银行服务只提供个人贷款、房屋贷款和汽车贷款的信息，假设将来银行的人想要提供一些其他贷款功能，如黄金贷款和学习贷款，那么你需要修改这个类实现对吗？

同样，考虑 `sendOTP()` 方法，假设 `BankService` 支持将 `OTP` 媒体作为电子邮件发送，但将来他们可能希望引入将 `OTP` 媒体通过手机短信发送，这时候需要再次修改`BankService`来实现。

发现没有，它不遵循单一职责原则，因为这个类有许多责任或任务要执行，不仅会让`BankService`这个类很庞大，可维护性差。

为了实现单一职责原则的目标，我们应该实现一个单独的类，它只执行单一的功能。

*   打印相关的工作`PrinterService`  
    

```
public class PrinterService {
   public void printPassbook() {
        //update transaction info in passbook
    }
}
```

*   贷款相关的工作`LoanService`  
    

```
public class LoanService {
  public void getLoanInterestInfo(String loanType) {
        if (loanType.equals("homeLoan")) {
            //do some job
        }
        if (loanType.equals("personalLoan")) {
            //do some job
        }
        if (loanType.equals("car")) {
            //do some job
        }
    }
}
```

*   通知相关的工作`NotificationService`  
    

```
public class NotificationService{
  public void sendOTP(String medium) {
        if (medium.equals("email")) {
            //write email related logic
            //use JavaMailSenderAPI
        }
    }
}
```

现在，如果你观察到每个类都有单一的责任来执行他们的任务。这正是 单一职责 `SRP` 的核心思想。

开闭原则（OSP）
---------

该原则指出“软件实体（类、模块、函数等）应该对扩展开放，但对修改关闭”，这意味着您应该能够扩展类行为，而无需修改它。

让我们通过一个例子来理解这个原则。让我们考虑一下我们刚刚创建的同一个通知服务。

```
public class NotificationService {
  public void sendOTP(String medium) {
        if (medium.equals("email")) {
            //write email related logic
            //use JavaMailSenderAPI
        }
    }
}
```

如前所述，如果您想通过手机号码发送 `OTP`，那么您需要修改 `NotificationService`，对吗？

但是根据 OSP 原则，对扩展开放，对修改关闭, 因此不建议为增加一个通知方式就修改`NotificationService`类，而是要扩展，怎么扩展呢？

*   定义一个通知服务接口
    

```
public interface NotificationService {
  public void sendOTP();
}
```

*   E-mail 方式通知类`EmailNotification`  
    

```
public class EmailNotification implements NotificationService{
  public void sendOTP(){
    // write Logic using JavaEmail api
  }
}
```

*   手机方式通知类`MobileNotification`  
    

```
public class MobileNotification implements NotificationService{
    public void sendOTP(){
    // write Logic using Twilio SMS API
  }
}
```

*   同样可以添加微信通知服务的实现`WechatNotification`  
    

```
public class WechatNotification implements NotificationService{
  public void sendOTP(String medium){
    // write Logic using wechat API
  }
}
```

这样的方式就是遵循开闭原则的，你不用修改核心的业务逻辑，这样可能带来意向不到的后果，而是扩展实现方式，由调用方根据他们的实际情况调用。

里氏替换原则（LSP）
-----------

该原则指出“派生类或子类必须可替代其基类或父类”。换句话说，如果类 A 是类 B 的子类型，那么我们应该能够在不中断程序行为的情况下用 A 替换 B。

这个原理有点棘手和有趣，它是基于继承概念设计的，所以让我们通过一个例子更好地理解它。

让我们考虑一下我有一个名为 `SocialMedia` 的抽象类，它支持所有社交媒体活动供用户娱乐，如下所示：

```
package com.alvin.solid.lsp;

public abstract class SocialMedia {
    
    public abstract  void chatWithFriend();
    
    public abstract void publishPost(Object post);
    
    public abstract  void sendPhotosAndVideos();
    
    public abstract  void groupVideoCall(String... users);
}
```

社交媒体可以有多个实现或可以有多个子类，如 `Facebook`、`Wechat`、`Weibo` 和 `Twitter` 等。

现在让我们假设 `Facebook` 想要使用这个特性或功能。

```
package com.alvin.solid.lsp;

public class Wechat extends SocialMedia {

    public void chatWithFriend() {
        //logic  
    }

    public void publishPost(Object post) {
        //logic  
    }

    public void sendPhotosAndVideos() {
        //logic  
    }

    public void groupVideoCall(String... users) {
        //logic  
    }
}
```

我们都知道`Facebook`都提供了所有上述的功能，所以这里我们可以认为`Facebook`是`SocialMedia`类的完全替代品，两者都可以无中断地替代。

现在让我们讨论 `Weibo` 类

```
package com.alvin.solid.lsp;

public class Weibo extends SocialMedia {
    public void chatWithFriend() {
        //logic
    }

    public void publishPost(Object post) {
      //logic
    }

    public void sendPhotosAndVideos() {
      //logic
    }

    public void groupVideoCall(String... users) {
        //不适用
    }
}
```

我们都知道`Weibo`微博这个产品是没有群视频功能的，所以对于 `groupVideoCall`方法来说 `Weibo` 子类不能替代父类 `SocialMedia`。所以我们认为它是不符合里式替换原则。

**那有什么解决方案吗？**

那就把功能拆开呗。

```
public interface SocialMedia {   
   public void chatWithFriend(); 
   public void sendPhotosAndVideos() 
}
```

```
public interface SocialPostAndMediaManager { 
    public void publishPost(Object post); 
}
```

```
public interface VideoCallManager{ 
   public void groupVideoCall(String... users); 
}
```

现在，如果您观察到我们将特定功能隔离到单独的类以遵循 LSP。

现在由实现类决定支持功能，根据他们所需的功能，他们可以使用各自的接口，例如 `Weibo` 不支持视频通话功能，因此 `Weibo` 实现可以设计成这样：

```
public class Instagram implements SocialMedia,SocialPostAndMediaManager{
  public void chatWithFriend(){
    //logic
    }
    public void sendPhotosAndVideos(){
    //logic
    }
    public void publishPost(Object post){
    //logic
    }
}
```

这样子就是符合里式替换原则 LSP。

接口隔离原则（ISP）
-----------

这个原则是第一个适用于接口而不是 `SOLID` 中类的原则，它类似于单一职责原则。它声明“不要强迫任何客户端实现与他们无关的接口”。

例如，假设您有一个名为 `UPIPayment` 的接口，如下所示

```
public interface UPIPayments {
    
    public void payMoney();
    
    public void getScratchCard();
    
    public void getCashBackAsCreditBalance();
}
```

现在让我们谈谈 `UPIPayments` 的一些实现，比如 `Google Pay` 和 `AliPay`。

`Google Pay` 支持这些功能所以他可以直接实现这个 `UPIPayments` 但 `AliPay` 不支持 `getCashBackAsCreditBalance()` 功能所以这里我们不应该强制客户端 `AliPay` 通过实现 `UPIPayments` 来覆盖这个方法。

我们需要根据客户需要分离接口，所以为了支持这个 ISP，我们可以如下设计：

*   创建一个单独的接口来处理现金返还。
    

```
public interface CashbackManager{ 
  public void getCashBackAsCreditBalance(); 
}
```

现在我们可以从 `UPIPayments` 接口中删除`getCashBackAsCreditBalance` ，`AliPay`也不需要实现`getCashBackAsCreditBalance()`这个它没有的方法了。

依赖倒置原则（DIP）
-----------

该原则指出我们需要使用抽象（抽象类和接口）而不是具体实现，高级模块不应该直接依赖于低级模块，但两者都应该依赖于抽象。

我们直接上例子来理解。

假如你去当地一家商店买东西，并决定使用刷卡付款。因此，当您将卡交给店员进行付款时，店员不会检查你提供的是哪种卡，借记卡还是信用卡，他们只会进行刷卡，这就是店员和你之间传递“卡”这个抽象。

现在让我们用代码替换这个例子，以便更好地理解它。

*   借记卡
    

```
public class DebitCard { 
  public void doTransaction(int amount){ 
        System.out.println("tx done with DebitCard"); 
    } 
}
```

*   信用卡
    

```
public class CreditCard{ 
  public void doTransaction(int amount){ 
        System.out.println("tx done with CreditCard"); 
    } 
}
```

现在用这两张卡你去购物中心购买了一些订单并决定使用信用卡支付

```
public class ShoppingMall {
  private DebitCard debitCard;
  public ShoppingMall(DebitCard debitCard) {
        this.debitCard = debitCard;
     }
  public void doPayment(Object order, int amount){              debitCard.doTransaction(amount); 
   }
  public static void main(String[] args) {
       DebitCard debitCard=new DebitCard();
       ShoppingMall shoppingMall=new ShoppingMall(debitCard);
       shoppingMall.doPayment("some order",5000);
    }
}
```

上面的做法是一个错误的方式，因为 `ShoppingMall` 类与 `DebitCard` 紧密耦合。

现在你的借记卡余额不足，想使用信用卡，那么这是不可能的，因为 `ShoppingMall` 与借记卡紧密结合。

当然你也可以这样做，从构造函数中删除借记卡并注入信用卡。但这不是一个好的方式，它不符合依赖倒置原则。

**那该如何正确设计呢？**

*   定义依赖的抽象接口`BankCard`  
    

```
public interface BankCard { 
  public void doTransaction(int amount); 
}
```

*   现在 `DebitCard` 和 `CreditCard` 都实现`BankCard`  
    

```
public class CreditCard implements BankCard{
  public void doTransaction(int amount){            
        System.out.println("tx done with CreditCard");
    }
}
```

```
public class DebitCard implements BankCard { 
  public void doTransaction(int amount){ 
    System.out.println("tx done with DebitCard"); 
    } 
}
```

*   现在重新设计购物中心这个高级类，他也是去依赖这个抽象，而不是直接低级模块的实现类
    

```
public class ShoppingMall {
  private BankCard bankCard;
  public ShoppingMall(BankCard bankCard) {
        this.bankCard = bankCard;
    }
  public void doPayment(Object order, int amount){
        bankCard.doTransaction(amount);
    }
  public static void main(String[] args) {
        BankCard bankCard=new CreditCard();
        ShoppingMall shoppingMall1=new ShoppingMall(bankCard);
        shoppingMall1.doPayment("do some order", 10000);
    }
}
```

现在，如果您观察购物中心与 `BankCard` 松散耦合，任何类型的卡处理支付都不会产生任何影响，这就是符合依赖倒置原则的。

总结
--

我们再来回顾总结下 SOLID 原则，

单一职责原则：每个类应该负责系统的单个部分或功能。

开闭原则：软件组件应该对扩展开放，而不是对修改开放。

里式替换原则：超类的对象应该可以用其子类的对象替换而不破坏系统。

接口隔离原则不应强迫客户端依赖于它不使用的方法。

依赖倒置原则：高层模块不应该依赖低层模块，两者都应该依赖抽象。

这些原则看起来都很简单，但用起来用的好就比较难了，希望大家在平时的开发的过程中多多思考、多多实践。
