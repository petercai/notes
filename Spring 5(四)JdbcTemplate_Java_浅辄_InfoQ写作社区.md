# Spring 5(四)JdbcTemplate_Java_浅辄_InfoQ写作社区
![](https://static001.geekbang.org/infoq/48/48d929be07260b9ceaa71e26cc6064af.jpeg)

四.JdbcTemplate
--------------

#### 1.基本概念

*   什么是 JdbcTemplate?
    
*   Spring 框架对\]DBC 进行封装，使用 JdbcTemplate 方便实现对数据库操作
    
*   准备工作
    
*   引入相关 jar 包
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221112150002220.png)
    
      
    
*   在 spring 配置文件配置数据库连接池
    

```
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
          destroy-method="close">
        <property name="url" value="jdbc:mysql:///db02?serverTimezone=UTC"/>
        <property name="username" value="root"/>
        <property name="password" value="123456"/>
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    </bean>
```

*   配置 JdbcTemplate 对象，注入 DataSource
    

```
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        
        <property name="dataSource" ref="dataSource"/>
    </bean>
```

*   创建 service 类，创建 dao 类，在 dao 注入 jdbcTemplate 对象
    
*   配置文件
    

```
<context:component-scan base-package="com.gbx"/>
```

```
@Service
      public class BookService {
      
          
          @Autowired
          private BooKDao booKDao;
      }
```

```
@Repository
      public class BookImpl implements BooKDao{
      
          
          @Autowired
          private JdbcTemplate jdbcTemplate;
      }
```

#### 2.操作数据库

##### 2.1 添加

*   对应数据库创建实体类
    

```
public class User {
      private String userId;
      private String username;
      private String ustatus;
  
      public void setUserId(String userId) {
          this.userId = userId;
      }
  
      public void setUsername(String username) {
          this.username = username;
      }
  
      public void setUstatus(String ustatus) {
          this.ustatus = ustatus;
      }
  }
```

*   编写 service 和 dao
    
*   在 dao 进行数据库添加操作
    
*   调用 JdbcTemplate 对象里面 update 方法实现添加操作
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221112215328169.png)
    
      
    
*   有两个参数
    
*   第一个参数:sql 语句
    
*   第二个参数:可变参数，设置 sql 语句值
    

```
@Repository
    public class BookImpl implements BooKDao{
    
        
        @Autowired
        private JdbcTemplate jdbcTemplate;
    
        
        @Override
        public void add(Book book) {
            
            String sql = "insert into t_book values (?,?,?)";
            
            Object[] args = {book.getUserId(),book.getUsername(),book.getUstatus()};
            int update = jdbcTemplate.update(sql,args);
            System.out.println(update);
        }
    }
```

*   测试类
    

```
@Test
      public void testJdbcTemplate(){
          ApplicationContext context =
                  new ClassPathXmlApplicationContext("bean1.xml");
          BookService bookService = context.getBean("bookService", BookService.class);
          Book book = new Book();
          book.setUserId("1");
          book.setUsername("三国");
          book.setUstatus("售罄");
          bookService.addBook(book);
      }
```

![](https://static001.geekbang.org/infoq/f6/f67995e648afca1b17ed1648dabb7d0d.png)

##### 2.2 修改和删除

Service 层

```
public void update(Book book){
    booKDao.updateBook(book);
}


public void delete(String id){
    booKDao.deleteBook(id);
}
```

Dao 层

```
void updateBook(Book book);

    void deleteBook(String id);
```

实现类

```
@Override
public void updateBook(Book book) {
    
    String sql = "update t_book set username=?,ustatus=?,where user_id=?";
    
    Object[] args = {book.getUsername(),book.getUstatus(),book.getUserId()};
    int update = jdbcTemplate.update(sql,args);
    System.out.println(update);
}


@Override
public void deleteBook(String id) {
    
    String sql = "delete from t_book where user_id=?";
    
    int update = jdbcTemplate.update(sql,id);
    System.out.println(update);
}
```

Test 类

```
Book book = new Book();
book.setUserId("1");
book.setUsername("三国演义");
book.setUstatus("在售");
bookService.update(book);


bookService.delete("1");
```

##### 2.3 查询

2.3.1 查询返回某个值

*   查询表里面有多少条记录，返回是某个值
    
*   使用 JdbcTemplate 实现查询返回某个值代码
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221113121425285.png)
    
      
    
*   有两个参数
    
*   第一个参数:sql 语句
    
*   第二个参数:返回类型 Class
    

```
@Override
      public int selectCount() {
          String sql =  "select count(*) from t_book";
          Integer count = jdbcTemplate.queryForObject(sql, Integer.class);
          return count;
      }
```

2.3.2 查询返回对象

*   场景：查询图书详情
    
*   JdbcTemplate 实现
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221113140928132.png)
    
      
    
*   有三个参数
    
*   第一个参数：sql 语句
    
*   第二个参数：RowMapper,是接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
    
*   第三个参数：sql 语句值
    

```
@Override
  public Book findBookInfo(String id) {
      String sql = "select * from t_book where userid=?";
      Book book = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<Book>(Book.class), id);
      return book;
  }
```

2.3.3 查询返回集合

*   场景：查询图书列表分页...
    
*   调用 JdbcTemplate 方法实现查询返回集合
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221113142125180.png)
    
      
    
*   有三个参数
    
*   第一个参数：sql 语句
    
*   第二个参数：RowMapper,是接口，返回不同类型数据，使用这个接口里面实现类完成数据封装
    
*   第三个参数：sql 语句值
    

```
@Override
    public List<Book> findAllBook() {
        String sql = "select * from t_book";
        List<Book> bookList = jdbcTemplate.query(sql, new BeanPropertyRowMapper<Book>(Book.class));
        return bookList;
    }
```

##### 2.4 批量操作

2.4.1 批量添加

*   批量操作：操作表里面多条记录
    
*   \]dbcTemplate 实现批量添加操作
    
    ![](https://xingqiu-tuchuang-1256524210.cos.ap-shanghai.myqcloud.com/00325/image-20221113211719867.png)
    
      
    
*   有两个参数
    
*   第一个参数：sql 语句
    
*   第二个参数：List 集合，添加多条记录数据
    

```
@Override
  public void batchAddBook(List<Object[]> batchArgs) {
      String sql = "insert into t_book values (?,?,?)";
      int[] ints = jdbcTemplate.batchUpdate(sql,batchArgs);
      System.out.println(Arrays.toString(ints));
  }
  
  
  
  Test类
      
          List<Object[]> batchArgs=new ArrayList<>();
          Object[]o1 = {"3","java","a"};
          Object[]o2 = {"2","py","b"};
          Object[]o3 = {"4","C++","c"};
          batchArgs.add(o1);
          batchArgs.add(o2);
          batchArgs.add(o3);
          
          bookService.batchAdd(batchArgs);
```

2.4.2 批量修改

```
@Override
public void batchUpdateBook(List<Object[]> batchArgs) {
    String sql = "update t_book set username=?,ustatus=?where user_id=?";
    int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
    System.out.println(ints);
}

        
        List<Object[]> batchArgs=new ArrayList<>();
        Object[]o1 = {"java","在售","4"};
        Object[]o2 = {"py","售罄","9"};
        Object[]o3 = {"C++","在售","6"};
        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
        bookService.batchUpdate(batchArgs);
```

2.4.3 批量删除

```
@Override
public void batchDeleteBook(List<Object[]> batchArgs) {
    String sql = "delete from t_book where user_id=?";
    int[] ints = jdbcTemplate.batchUpdate(sql, batchArgs);
    System.out.println(Arrays.toString(ints));
}

        
        List<Object[]> batchArgs=new ArrayList<>();
        Object[]o1 = {"4"};
        Object[]o2 = {"9"};
        Object[]o3 = {"6"};
        batchArgs.add(o1);
        batchArgs.add(o2);
        batchArgs.add(o3);
        bookService.batchDelete(batchArgs);
```