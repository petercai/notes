# PyGWalker：Python中的Tableau，数据可视化

1介绍
---

在[数据分析](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%88%86%E6%9E%90&spm=1001.2101.3001.7020)和可视化的领域，Tableau凭借其强大的功能和直观的界面，一直以来都是专业人士的首选工具。然而，对于许多用户而言，Tableau的封闭性和高昂的成本使其难以触及。就在此刻，PyGWalker应运而生，为Python社区带来了Tableau式的交互式数据可视化体验，且无需额外的授权费用。

PyGWalker不仅继承了Tableau直观的拖拽式操作，更融合了Python编程的灵活性，提供了广泛的数据连接能力和高度可定制的图表类型。用户现在可以在熟悉的Python环境中，利用PyGWalker进行高效的数据探索和分析，无需切换到其他工具，也无需编写冗长的代码。

![](https://i-blog.csdnimg.cn/blog_migrate/5daa553660dd95e651902bfa92437d2d.png)

2 安装
----

使用pip或conda安装pygwalker

```
`pip install pygwalker` 

*   1


```

> 使用 `pip install pygwalker --upgrade` 更新最新版PyGWalker
> 
> 使用 `pip install pygwaler --upgrade --pre` 来尝鲜最新版，获得最新bug修复

```
`conda install -c conda-forge pygwalker` 

*   1


```

3 使用
----

所使用的数据集：[超市数据集](https://download.csdn.net/download/weixin_46043195/89005108)

（1）导入所需要的库：

```
`import pandas as pd
import pygwalker as pyg` 

*   1
*   2


```

（2）利用pandas将数据集读入内存：

```
`import pandas as pd
data = pd.read_excel('数据分析示例超市数据集.xls')
data.head()` 

*   1
*   2
*   3


```

![](https://i-blog.csdnimg.cn/blog_migrate/7d07466a53a7f0df02c15f2f8a924fe5.png)

（3）启动 pygwalker

```
`walker = pyg.walk(
    data,
    spec="./chart_meta_0.json",    
    use_kernel_calc=True,          
)` 

*   1
*   2
*   3
*   4
*   5


```

![](https://i-blog.csdnimg.cn/blog_migrate/c78fd322d9beb8fa51095e7c4298eb52.png)
  
![](https://i-blog.csdnimg.cn/blog_migrate/863bcd328bf55376e0e4942bd24147c5.png)

**接下来用拖拽式的方式构建你的可视化，熟悉tableau的朋友应该非常清楚~**

（4）查看超市商品类别与销售额的关系

![](https://i-blog.csdnimg.cn/blog_migrate/1e0a2b59a9196f3ecbd74c5937fb11da.png)

（5）查看子类别与销售额和利润的关系

![](https://i-blog.csdnimg.cn/blog_migrate/f88982b5b4075e3a132f5555e3741e12.png)

4 将[数据可视化](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%8F%AF%E8%A7%86%E5%8C%96&spm=1001.2101.3001.7020)导出为代码
----------------------------------------------------------------------------------------------------------------------

单击工具栏上的Export to Code 按钮。 该按钮位于“导出为 PNG/SVG”按钮旁边。  
![](https://i-blog.csdnimg.cn/blog_migrate/3e685707fe95f1336dee00052ebbda99.png)
  
![](https://i-blog.csdnimg.cn/blog_migrate/22b38cf3b97f61311106f35002cd24e9.png)

