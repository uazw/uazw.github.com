---
layout: post
title: 从inserve到双向关联
category: orm
---

##本文目的

* 了解在Hibernate映射配置中inverse的作用
* 如何正确的在实体类中使用集合类
* 双向关联其他

##inserve
###本小结使用的bean和对应的hibernate映射文件

忽略setter & getter

	public class Address {
    private int id;
    private String value;
    private Set<People> peoples = new HashSet<>();
	}
	

	public class People {
    private int id;
    private String name;
    private Set<Address> addresses = new HashSet<>();
	}

Address.hdm.xml:

	<hibernate-mapping package="bidirectional">
    <class name="Address" table="address">
        <id name="id" type="java.lang.Integer">
            <column name="address_id" />
            <generator class="assigned" />
        </id>
        <property name="value" type="java.lang.String" />
        <set name="peoples" table="address_people" inverse="true">
            <key><column name="address_id" /></key>
            <many-to-many class="People" column="people_id" ></many-to-many>
        </set>
    </class>
	</hibernate-mapping>

People.hdm.xml:

	<hibernate-mapping package="bidirectional">
    <class name="bidirectional.People">
        <id name="id">
            <column name="people_id" />
            <generator class="assigned" />
         </id>
        <property name="name" >
            <column name="people_name" />
        </property>
        <set name="addresses" table="address_people">
            <key><column name="people_id" /></key>
            <many-to-many class="bidirectional.Address" column="address_id" />
        </set>
    </class>
	</hibernate-mapping>


###正文

首先需要明确的是inverse是用于双向关联

inverse在[HibernateReference]的解释如下:

>Hibernate, however, does not have enough information to correctly arrange SQL INSERT and UPDATE statements (to avoid constraint violations). Making one side of the association inverse tells Hibernate to consider it a mirror of the other side. That is all that is necessary for Hibernate to resolve any issues that arise when transforming a directional navigation model to a SQL database schema. The rules are straightforward: all bi-directional associations need one side as inverse.

>What this means is that Hibernate should take the other side,hen it needs to find out information about the link between the two

嗯。。。好长的英文，总结如下：

Hibernate为了更新两个实体之间的关系，需要获得一些必要的信息

当双方inverse="false"(默认并且使用注解的情况无法更改)，表示双方共同提供维护关系必备的信息， 

这意味着在上面的例子中如果将Address.hdm.xml:

	<set name="peoples" table="address_people" inverse="true">
改为

	<set name="peoples" table="address_people">

这时候在两边我们的inverse="false", 那么为了更新关联表中的数据我们必须在双边都做操作，例如增加

	address.getPeoples().add(people);
	people.getAddresses().add(address);
	session.update(address);
	session.update(people);

是不是感觉很麻烦?

如果我们将一方设为inverse="true"， 另一方设为inverse="false", 此时将由**inverse="false"**的一边**提供消息**。  
在上面的例子中就是由People对象来管理， 那么对代码的影响是什么呢？

	people.getAddresses().add(address);
	session.update(people);

这样就可以了，注意在inverse="**true**"的一边对关系表进行更新是**没用**的，在本例中也就是
	
	address.getPeoples().add(people);
	session.update(address);

这样就是把双向关联， 简化为单向关联，当然在本例中用这种方式是存在潜在的bug， 那就是address在内存中并没有执行更新，  
只有当提交了更新，并且address再次是persistent状态的时候, 在内存中的address才会是最新，这就存在**不一致**问题。

当然如果我们在双方inverse="true", 那么关系表将不会得到更新， 除非用HQL or SQL直接操作。

##双向关联

那么既然inverse这么好用，为什么注解的方式并不提供呢？我想

[HibernateReference]: http://docs.jboss.org/hibernate/orm/4.3/manual/en-US/html_single
