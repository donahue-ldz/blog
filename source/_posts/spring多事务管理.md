title: "spring多事务管理"
date: 2015-03-15 10:01:36
categories: spring
tags: [spring,事务管理]
---

最近在梳理项目的时候对事务控制进行了梳理，深入了解了spring的事务，结合相关资料整理如下

---
Spring抽象的DAO体系兼容多种数据访问技术，它们各有特色，各有千秋。像Hibernate是非常优秀的ORM实现方案，但对底层SQL的控制不太方便；而iBatis则通过模板化技术让你方便地控制SQL，但没有Hibernate那样高的开发效率；自由度最高的当然是直接使用SpringJDBC了，但它也是底层的，灵活的代价是代码的繁复。很难说哪种数据访问技术是最优秀的，只有在某种特定的场景下才能给出答案。

>所以在一个应用中，往往采用多个数据访问技术：一般是两种，一种采用ORM技术框架，而另一种采用偏JDBC的底层技术，两者珠联璧合，形成联合军种，共同御敌。 

但是，这种联合军种如何应对事务管理的问题呢？我们知道Spring为每种数据访问技术提供了相应的事务管理器，难道需要分别为它们配置对应的事务管理器吗？它们到底是如何协作和工作的呢？这些层出不穷的问题往往压制了开发人员使用联合军种的想法。 
其实，在这个问题上，我们低估了Spring事务管理的能力。如果你采用了一个高端ORM技术（Hibernate、JPA、JDO），同时采用一个JDBC技术（Spring JDBC、iBatis），由于前者的会话（Session）是对后者连接（Connection）的封装，Spring会“足够智能地”在同一个事务线程让前者的会话封装后者的连接。所以，我们只要直接采用前者的事务管理器就可以了。


>组合: Hibernate+ Spring JDBC或iBatis
>事务管理: org.springframework.orm.hibernate3.HibernateTransactionManager


>组合: JPA+Spring JDBC或iBatis
事务管理: org.springframework.orm.jpa.JpaTransactionManager

>组合：JDO+Spring JDBC或iBatis
事务管理：org.springframework.orm.jdo.JdoTransactionManager

---

举例：
`Hibernate+Spring JDBC混合框架的事务管理`                

由于一般不会出现同时使用多个ORM框架的情况（如Hibernate+JPA），我们不拟对此命题展开论述，只重点研究ORM框架+JDBC框架的情况。Hibernate+Spring JDBC可能是被使用得最多的组合，本节我们通过实例观察事务管理的运作情况。 
```
@Service("userService")  
public class UserService extends BaseService {  
    @Autowired  
    private HibernateTemplate hibernateTemplate;  
  
    @Autowired  
    private ScoreService scoreService;  
  
    public void logon(String userName) {  
         
        //①通过Hibernate技术访问数据  
        System.out.println("before updateLastLogonTime()..");  
        updateLastLogonTime(userName);  
        System.out.println("end updateLastLogonTime()..");  
          
        //②通过JDBC技术访问数据  
        System.out.println("before scoreService.addScore()..");  
        scoreService.addScore(userName, 20);  
        System.out.println("end scoreService.addScore()..");  
    }  
      
    public void updateLastLogonTime(String userName) {  
        User user = hibernateTemplate.get(User.class,userName);  
        user.setLastLogonTime(System.currentTimeMillis());  
        hibernateTemplate.update(user);  
  
        //③这句很重要，请看下文的分析  
        hibernateTemplate.flush();  
    }  
}  

```

①处，使用Hibernate操作数据   
②处调用ScoreService#addScore()，该方法内部使用Spring JDBC操作数据。    
③处，我们显式调用了flush()方法，将Session中的缓存同步到数据库中（即马上向数据库发送一条更新记录的SQL语句）。之所以要显式执行flush()方法，原因是在默认情况下，Hibernate对数据的更改只是记录在一级缓存中，要等到事务提交或显式调用flush()方法时才会将一级缓存中的数据同步到数据库中，而提交事务的操作发生在logon()方法返回前。如果所有针对数据库的更改操作都使用Hibernate，这种数据同步的延迟机制并不会产生任何问题。但是，我们在logon()方法中同时采用了Hibernate和SpringJDBC混合数据访问技术，Spring JDBC无法自动感知Hibernate一级缓存，所以如果不及时调用flush()方法将记录数据更改的一级缓存同步到数据库中，则②处通过SpringJDBC进行数据更改的结果将被Hibernate一级缓存中的更改覆盖掉，因为Hibernate一级缓存要等到logon()方法返回前才同步到数据库！ 
ScoreService使用Spring JDBC数据访问技术，其代码如下所示： 
```  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.jdbc.core.JdbcTemplate;  
import org.springframework.stereotype.Service;  
import org.apache.commons.dbcp.BasicDataSource;  
  
@Service("scoreService")  
public class ScoreService extends BaseService{  
  
    @Autowired  
    private JdbcTemplate jdbcTemplate;  
  
    public void addScore(String userName, int toAdd) {  
        String sql = "UPDATE t_user u SET u.score = u.score + ? WHERE user_name =?";  
        jdbcTemplate.update(sql, toAdd, userName);  
        BasicDataSource basicDataSource = (BasicDataSource) jdbcTemplate.getDataSource();  
        //①查看此处数据库激活的连接数量  
        System.out.println("[scoreUserService.addScore]激活连接数量："  
                         +basicDataSource.getNumActive());  
    }  
}  

```
Spring关键的配置文件代码如下所示： 
```
<!--①使用Hibernate事务管理器 -->  
<bean id="hiberManager"  
      class="org.springframework.orm.hibernate3.HibernateTransactionManager"  
      p:sessionFactory-ref="sessionFactory"/>  
  
<!--②使UserService及ScoreService的公用方法都拥有事务 -->  
<aop:config proxy-target-class="true">  
    <aop:pointcut id="serviceJdbcMethod"  
                  expression="within(com.baobaotao.mixdao.BaseService+)"/>  
    <aop:advisor pointcut-ref="serviceJdbcMethod"  
                 advice-ref="hiberAdvice"/>  
</aop:config>  
<tx:advice id="hiberAdvice" transaction-manager="hiberManager">  
    <tx:attributes>  
        <tx:method name="*"/>  
    </tx:attributes>  
</tx:advice>  
  
/beans>  
```
启动Spring容器，执行UserService#logon()方法，可以查看到如下的执行日志： 
```
before userService.logon().. 

①在执行userService.logon()后，Spring开启一个事务 
Creating new transaction with name [com.baobaotao.mixdao.UserService.logon]: PROPAGATION_REQUIRED,ISOLATION_DEFAULT 
opened session at timestamp: 13009379637 
Opened new Session [org.hibernate.impl.SessionImpl@c5f468] for Hibernate transaction 
… 
Exposing Hibernate transaction as JDBC transaction [jdbc:mysql://localhost:3306/sampledb, UserName=root@localhost, MySQL-AB JDBC Driver] 
before userService.updateLastLogonTime().. 

②userService.updateLastLogonTime()执行时自动绑定到①处开启的Session中 
Found thread-bound Session for HibernateTemplate 
loading entity: [com.baobaotao.User#tom] 
about to open PreparedStatement (open PreparedStatements: 0, globally: 0) 
… 
about to close PreparedStatement (open PreparedStatements: 1, globally: 1) 
Not closing pre-bound Hibernate Session after HibernateTemplate 
end updateLastLogonTime().. 

before scoreService.addScore().. 

③scoreService.addScore()执行时绑定到①处开启的Session中，并加入其所对应的事务中 
Found thread-bound Session [org.hibernate.impl.SessionImpl@c5f468] for Hibernate 
transaction 
Participating in existing transaction 
… 
SQL update affected 1 rows 

④此时数据源只打开了一个连接 
[scoreUserService.addScore]激活连接数量：1 
end scoreService.addScore().. 
Initiating transaction commit 

⑤提交Hibernate的事务，它将触发一级缓存到数据库的同步 
Committing Hibernate transaction on Session [org.hibernate.impl.SessionImpl@c5f468] 
commit 
processing flush-time cascades 
dirty checking collections 
Flushed: 0 insertions, 0 updates, 0 deletions to 1 objects 
Flushed: 0 (re)creations, 0 updates, 0 removals to 0 collections 
listing entities: 
com.baobaotao.User{lastLogonTime=1300937963882, score=10, userName=tom, password=123456} 
re-enabling autocommit 

⑥提效Session底层所绑定的JDBC Connection所对应的事务 
committed JDBC Connection 
transaction completed on session with on_close connection release mode; be sure to close the session to release JDBC resources! 
Closing Hibernate Session [org.hibernate.impl.SessionImpl@c5f468] after transaction 
Closing Hibernate Session 
releasing JDBC connection [ (open PreparedStatements: 0, globally: 0) (open ResultSets: 0, globally: 0)] 
transaction completed on session with on_close connection release mode; be sure to close the session to release JDBC resources! 
after userService.logon().. 
```

仔细观察这段输出日志，
①处UserService#logon()开启一个新的事务。
②处的UserService# updateLastLogonTime()绑定到事务上下文的Session中。
③处ScoreService#addScore()方法加入到①处开启的事务上下文中。
④处的输出是ScoreService#addScore()方法内部的输出信息，汇报此时数据源激活的连接数为1，这清楚地告诉我们Hibernate和JDBC这两种数据访问技术在同一事务上下文中“共用”一个连接。
⑤处，提交Hibernate事务，接着在⑥处触发调用底层的Connection提交事务。 

从以上的运行结果，我们可以得出这样的结论：使用Hibernate事务管理器后，可以混合使用Hibernate和SpringJDBC数据访问技术，它们将工作于同一事务上下文中。但是使用SpringJDBC访问数据时，Hibernate的一级或二级缓存得不到同步，此外，一级缓存延迟数据同步机制可能会覆盖SpringJDBC数据更改的结果。 
   
   由于混合数据访问技术方案存在“事务同步而缓存不同步”的情况，所以最好用Hibernate进行读写操作，而只用Spring JDBC进行读操作。如用Spring JDBC进行简要列表的查询，而用Hibernate对查询出的数据进行维护。 
   如果确实要同时使用Hibernate和SpringJDBC读写数据，则必须充分考虑到Hibernate缓存机制引发的问题：必须整体分析数据维护逻辑，根据需要及时调用Hibernate的flush()方法，以免覆盖Spring JDBC的更改，在Spring JDBC更改数据库时，维护Hibernate的缓存。由于方法调用顺序的不同都可能影响数据的同步性，因此很容易发生问题，这会极大提高数据访问程序的复杂性。所以笔者郑重建议不要同时使用Spring JDBC和Hibernate对数据进行写操作。 
>可以将以上结论推广到其他混合数据访问技术的方案中，如Hibernate+iBatis、JPA+Spring JDBC、JDO+Spring JDBC等。 


参考资料：《Spring 3.x企业应用开发实战》


