# synchronized还能用形参作为锁的对象？
最近整理项目的时候，发现一段代码很有意思，正好拿来研究一下synchronized对象的选择。 众所周知，synchronized是使用在方法或者代码块上，锁的对象分为类锁和对象锁。我们一般常用的都是代码块，获取的是对象锁。项目中的代码也是如此，但是选择的对象有点神奇。模拟线上的示例代码如下:

```java
    public Member getWxUserPhoneId(String username, Integer lock) {
        Member member;
            synchronized (username){
                 member = memberService.findMember(username, null);
                if (member != null) {
                    return member;
                }
                
                System.out.println(Thread.currentThread().getName());
                member = new Member();
                member.setUsername(username);
                member.setPassword("password");
                try {
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                memberService.save(member);
            }

        return member;
    }

```

```java
    @GetMapping("/findMember")
    public ResultMessage<Object> findMember(@NotNull(message = "用户名不能为空") @RequestParam String username){
        
        return ResultUtil.data(memberService.getWxUserPhoneId(username));
    }

```

想必大家也都看出来了，他选择用形参作为锁的对象，是不能保证每个线程获取的对象锁是相同的，事实也是如此。模拟四个线程同时请求

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c88ea7641cb64e7aa52b325f8bbdd315~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1587&h=489&s=123635&e=png&b=2b2b2b)
 一般来说，我们使用对象锁的时候，会用静态类作为锁的对象，这样才能保证多线程下获取的对象锁是同一个。但是用形参是否也可以保证对象唯一呢，当然总有旁门左道去实现。java形参的传递方式分为值传递和引用传递，除了基本类型，都是将对象的引用地址作为副本。而synchronized判断锁对象是否相同，简单理解也是判断对象的地址。所以只要保证传入的对象是地址是唯一的，就可以保证线程安全。将代码的锁对象修改成lock

```java
    public Member getWxUserPhoneId(String username, Integer lock) {
            Member member;
            synchronized (lock){
                 member = memberService.findMember(username, null);
                if (member != null) {
                    return member;
                }
                
                System.out.println(Thread.currentThread().getName());
                member = new Member();
                member.setUsername(username);
                member.setPassword("password");
                try {
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                memberService.save(member);
            }

        return member;
    }

```

```java
    @GetMapping("/findMember")
    public ResultMessage<Object> findMember(@NotNull(message = "用户名不能为空") @RequestParam String username){
        
        return ResultUtil.data(memberService.getWxUserPhoneId(username,1));
    }

```

重新执行一遍，成功了，不存在的情况下只有一个线程进入到代码中（Integer有缓存，所以1的内存地址都是相同的）

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13c08df539f24e8cb9dc892325740d01~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1700&h=461&s=84852&e=png&b=2b2b2b)
 这里就有点奇怪了，按道理String也有缓冲区进行缓存，为什么还是会发生这个问题，具体可能跟springboot有关系，测试下来会发现传入的username的内存地址都不相同，可能是因为它每次都会构建新的String对象导致的，将代码修改一下

```java
    @GetMapping("/findMember")
    public ResultMessage<Object> findMember(@NotNull(message = "用户名不能为空") @RequestParam String username){
        
        
        return ResultUtil.data(memberService.getWxUserPhoneId(username.intern(),null));
    }

```

bingo，只有一条线程进入

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c128a38375b40488672f7ece98d160b~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1846&h=415&s=98241&e=png&b=2b2b2b)
 总结来说，synchronized使用对象锁是，只要对象内存地址一致，无论是形参还是静态对象都可以保证，线程同步，但是还是不建议使用形参作为锁对象，本质跟静态对象是相同的，但是用形参会多出额外的理解成本和不可控的风险（例如锁影响的范围）。