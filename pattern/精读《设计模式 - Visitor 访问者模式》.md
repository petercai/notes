# 精读《设计模式 - Visitor 访问者模式》 - 知乎
**Visitor（访问者模式）**
------------------

Visitor（访问者模式）属于行为型模式。

**意图：表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。** 

访问者，顾名思义，就是对象访问的一种设计模式，我们可以在不改变要访问对象的前提下，对访问对象的操作做拓展。

**举例子**
-------

由于能应用访问者模式的场景很少，所以这里只举一个例子。

### **建造游戏中的资源设计**

假设你制作一款城市建造游戏，游戏的基础资源只有毛皮、木材、铜矿、铁矿。你需要用这些资源建造各种，比如造楼房、做衣服、制作家具、门、空调、甚至锅、健身房、游泳馆等。记住一个前提，就是你想把游戏设计的非常逼真，所以每种资源的不同使用方法都非常定制，不是简单的消耗 N 个数量就能完成，比如制作家具时，需要用到毛皮和木材，此时毛皮和木材对环境、制作人、资金都有不同的要求。

常见的想法是，我们将资源的所有使用方法都枚举在资源类中，这样资源就在用到不同场景时，调用不同方法即可。但问题是资源本身其实较为固定，我们每增加一种用途就修改一次木材、铁矿的类会显得非常麻烦。

能不能在增加新用途时，不修改原始资源类呢？答案是可以用访问者模式。

**意图解释**
--------

**意图：表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。** 

第一句话指明了 Visitor 的作用，即 “作用于某对象结构中的各元素的操作”，也就是 Visitor 是用于操作对象元素的。“它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作” 也就是说，你可以只修改 Visitor 本身完成新操作的定义，而不需要修改原本对象。

这看上去比较奇怪，给对象定义新的操作，竟然不用修改对象本身，而通过改另外一个对象就可以？这就是 Visitor 设计的奇妙之处，它将对象的操作权移交给了 Visitor。

**结构图**
-------

![](_assets/v2-53208bdb8eb7268666856affb4054892_b.jpg)

*   Visitor：访问者接口。
*   ConcreteVisitor：具体的访问者。
*   Element 可以被访问者使用的元素，它必须定义一个 Accept 属性，接收 visitor 对象。这是实现访问者模式的关键。
*   ObjectStructure：对象结构，存储了多个 Element，利用 Visitor 进行批量操作。

可以看到，要实现操作权转让到 Visitor，核心是元素必须实现一个 Accept 函数，将这个对象抛给 Visitor：

```text
class ConcreteElement implements Element {
  public accept(visitor: Visitor) {
    visitor.visit(this)
  }
}

```

从上面代码可以看出这样一条链路：Element 通过 `accept` 函数接收到 Visitor 对象，并将自己的实例抛给 Visitor 的 `visit` 函数，**这样我们就可以在 Visitor 的 `visit` 方法中拿到对象实例，完成对对象的操作。** 

**代码例子**
--------

下面例子使用 typescript 编写。

```text
class ConcreteVisitorX implements Visitor{
  public visit(element: ELement) {
    element.accept(this);
  }

  public visit(concreteElementA: ConcreteElementA) {
    console.log('X 操作 A')
  }

  public visit(concreteElementB: ConcreteElementB) {
    console.log('X 操作 B')
  }
}

class ConcreteVisitorY implements Visitor{
  public visit(element: ELement) {
    element.accept(this);
  }

  public visit(concreteElementA: ConcreteElementA) {
    console.log('Y 操作 A')
  }

  public visit(concreteElementB: ConcreteElementB) {
    console.log('Y 操作 B')
  }
}

```

配合上面已经写过的 `Element`，可以看到，经历了如下过程：

```text
// 先创建元素
const element = new ConcreteElement()

// 访问者 X
const visitorX = new ConcreteVisitorX()

// 访问者 Y
const visitorY = new ConcreteVisitorY()

// 然后让访问者 visit 观察一下元素
visitorX.visit(element as Element)
visitorY.visit(element as Element)

```

要注意的是，访问者观察的 Element 一定要是通用类型 Element，而不是一个具体类型 ConcreteElement，否则访问者模式抽象性就无法体现了，因为 Visitor 可以访问任何类型的 Element，所以先把接口传进去。

到这里，我们看看下面经历了什么：首先 Visitor 定义的 `visit` 会被调用，由于符合了 Element 这个通用类型，所以会调用 Element 接口定义的 `accept` 函数，这是所有元素都有的方法。

接下来，每个具体元素都重写了 `accept` 方法：

```text
public accept(visitor: Visitor) {
  visitor.visit(this)
}

```

所以又调用了 Visitor 的 `visit` 函数，不同的是，此时的参数是具体 Element 类型，所以可能调用到的是具体对某个元素处理的 `visit` 方法，比如：

```text
public visit(concreteElementA: ConcreteElementA) {
  console.log('X 操作 A')
}

```

最终就输出了 “X 操作 A” 这段话。

我们可以看到这样的程序拓展性有这么些：

1.  Element 元素的所有子类都不用频繁修改，只要修改 Visitor 即可。
2.  一个 Visitor 可以选择性的操作任何类型的 Element 子类，只要申明了处理函数即可处理，不申明就不会命中，比较方便。在城市建造的例子中，可以提现为锅需要用铁制作，但不需要消耗木材，所以不需要定义木材的 `visit` 方法。
3.  可以定义多种 Visitor，对同一种 Element 子类也可以有不同的操作，这在我们城市建造的例子中，可以体现为门和窗户，对铁矿的使用是不同的。

由此一来，我们就能在城市建造的例子中拓展出任意多种使用资源的场景，而无需让资源感知到这些场景的存在。

**弊端**
------

访问者模式使用场景非常有限，请确定你的场景满足以上情况再使用。如果资源并不需要频繁修改和拓展，那么就没必要使用访问者模式。

**总结**
------

访问者模式的精髓，就是在不断拓展的业务场景中，防止基础元素代码不断膨胀。

假设我们这款城市建造游戏有 20 人团队开发，每周发布 2 个版本，每个版本新增了几种资源的组合使用方式，由于资源一共就木材、铁矿、铜矿那么几种，如果你作为团队负责人，任大家随意修改这些资源基础类，过不了半年就会发现，木材类的成员方法突破了 100 种，而且以每天新增 2 种的速度不断增加，你会明显发现自己精心打造的程序即将变成一堆屎山。

更要命的是，你还搞不清楚哪些场景的用法是打包的，当一种使用场景下线时，已存在的成员方法还不敢删除。

假设你用了访问者模式，会发现，每天因为迭代而新增的那几个方法，都会放到一个新 Visitor 文件下，比如一种纳米材料的门板在游戏 V1.5 版本被引进，它对材料的使用会体现在新增一个 Visitor 文件，资源本身的类不会被修改，这既不会引发协同问题，也使功能代码按照场景聚合，不论维护还是删除的心智负担都非常小。

访问者模式背后的思考本质还是，基础的元素数量一般不会随着程序迭代产生太大变化，而对这些基础元素的使用方式或组合使用会随着程序迭代不断更新，我们将变化更快的通过 Visitor 打包提取出来，自然会更利于维护。

> 讨论地址是：**[精读《设计模式 - Visitor 访问者模式》· Issue #306 · dt-fe/weekly](https://link.zhihu.com/?target=https%3A//github.com/dt-fe/weekly/issues/306)**  

**如果你想参与讨论，请 [点击这里](https://link.zhihu.com/?target=https%3A//github.com/dt-fe/weekly)，每周都有新的主题，周末或周一发布。前端精读 - 帮你筛选靠谱的内容。** 

> 关注 **前端精读微信公众号**  

![](_assets/v2-d46b608443dd2afa3dea21b96236fa06_b.jpg)

> 版权声明：自由转载-非商用-非衍生-保持署名（**[创意共享 3.0 许可证](https://link.zhihu.com/?target=https%3A//creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)**）··