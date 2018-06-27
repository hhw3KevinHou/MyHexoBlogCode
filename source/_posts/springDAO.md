---
title: Spring Data access 
date: 2018-06-08 12:52:08
tags: Spring DAO, Spring JDBC, Spring ORM, Spring DATA
categories: Maven
---

# Spring-DAO

Spring-DAO is not stricto senso a spring module, but rather conventions that should dictate you to write DAO, and to write them well. As such, it does neither provide interfaces nor implementations nor templates to access your data. When writing a DAO, you should annotate them with @Repository so that exceptions linked to the underlying technology (JDBC, Hibernate, JPA, etc.) are consistently translated into the proper DataAccessException subclass.

As an example, suppose you're now using Hibernate, and your service layer catches HibernateException in order to react to it. If you change to JPA, your DAOs interfaces should not change, and the service layer will still compile with blocks that catches HibernateException, but you will never enter these blocks as your DAOs are now throwing JPA PersistenceException. By using @Repository on your DAO, the exceptions linked to the underlying technology are translated to Spring DataAccessException; your service layer catches these exceptions and if you decide to change the persistence technology, the same Spring DataAccessExceptions will still be thrown as spring have translated native exceptions.

Note however that this has limited usage for the following reasons:

    
1. Your should usually not catch persistence exceptions, as the provider may have rolled back the transaction (depending on the exact exception subtype), and thus you should not continue the execution with an alternative path.
    

2. The hierarchy of exceptions is usually richer in your provider than what Spring provides, and there's no definitive mapping from one provider to the other. Relying on this is hazardous. This is however a good idea to annotate your DAOs with @Repository, as the beans will be automatically added by the scan procedure. Further, Spring may add other useful features to the annotation.

# Spring-JDBC

Spring-JDBC provides the JdbcTemplate class, that removes plumbing code and helps you concentrate on the SQL query and parameters. You just need to configure it with a DataSource, and you can then write code like this:

		int nbRows = jdbcTemplate.queryForObject("select count(1) from person", Integer.class);
		
		Person p = jdbcTemplate.queryForObject("select first, last from person where id=?", 
		 rs -new Person(rs.getString(1), rs.getString(2)), 
		 134561351656L);

Spring-JDBC also provides a JdbcDaoSupport, that you can extend to develop your DAO. It basically defines 2 properties: a DataSource and a JdbcTemplate that both can be used to implement the DAO methods. It also provides an exceptions translator from SQL exceptions to spring DataAccessExceptions.

If you plan to use plain jdbc, this is the module you will need to use.

# Spring-ORM

Spring-ORM is an umbrella module that covers many persistence technologies, namely JPA, JDO, Hibernate and iBatis. For each of these technologies, Spring provides integration classes so that each technology can be used following Spring principles of configuration, and smoothly integrates with Spring transaction management.

For each technology, the configuration basically consists in injecting a DataSource bean into some kind of SessionFactory or EntityManagerFactory etc. bean. For pure JDBC, there's no need for such integration classes (apart from JdbcTemplate), as JDBC only relies on a DataSource.

If you plan to use an ORM like JPA or Hibernate, you will not need spring-jdbc, but only this module.

# Spring-Data

Spring-Data is an umbrella project that provides a common API to define how to access data (DAO + annotations) in a more generic way, covering both SQL and NOSQL data sources.

The initial idea is to provide a technology so that the developer writes the interface for a DAO (finder methods) and the entity classes in a technology-agnostic way and, based on configuration only (annotations on DAOs & entities + spring configuration, be it xml- or java-based), decides the implementation technology, be it JPA (SQL) or redis, hadoop, etc. (NOSQL).

If you follow the naming conventions defined by spring for the finder method names, you don't even need to provide the query strings corresponding to finder methods for the most simple cases. For other situations, you have to provide the query string inside annotations on the finder methods.

When the application context is loaded, spring provides proxies for the DAO interfaces, that contain all the boilerplate code related to the data access technology, and invokes the configured queries.

Spring-Data concentrates on non-SQL technologies, but still provides a module for JPA (the only SQL technology).


### Need  Hibernate...?
You can use a core Spring module like Spring JDBC for directly working with MySQL (or any other RDBMS) but if you wish to use Spring Data JPA, you will need an underlying JPA implementation (since Spring Data JPA is a wrapper on top of JPA and not a JPA implementation in itself) like Hibernate or EclipseLink.

The structure is as follows:


    Spring Data JPA
       |
      JPA
       |
    Hibernate


You need Hibernate as an JPA implementation, but from your perspective you should only see Spring Data JPA.

When designing your entities if you make sure that you use only annotations from the javax.persistence package you will not depend on one concrete JPA implementation (in this case Hibernate) but theoretically you could swap Hibernate for EclipseLink or something else.


# What's next

Knowing all this, you have now to decide what to pick. The good news here is that you don't need to make a definitive final choice for the technology. This is actually where Spring power resides : as a developer, you concentrate on the business when you write code, and if you do it well, changing the underlying technology is an implementation or configuration detail.

    

1. Define a data model with POJO classes for the entities, and get/set methods to represent the entity attributes and the relationships to other entities. You will certainly need to annotate the entity classes and fields based on the technology, but for now, POJOs are enough to start with. Just concentrate on the business requirements for now.
    

2. Define interfaces for your DAOs. 1 DAO covers exactly 1 entity, but you will certainly not need a DAO for each of them, as you should be able to load additional entities by navigating the relationships. Define the finder methods following strict naming conventions.
    

3. Based on this, someone else can start working on the services layer, with mocks for your DAOs.
    

4. You learn the different persistence technologies (sql, no-sql) to find the best fit for your needs, and choose one of them. Based on this, you annotate the entities and implement the DAOs (or let spring implement them for you if you choose to use spring-data).
    

5. If the business requirements evolve and your data access technology is not sufficient to support it (say, you started with JDBC and a few entities, but now need a richer data model and JPA is a better choice), you will have to change the implementation of your DAOs, add a few annotations on your entities and change the spring configuration (add an EntityManagerFactory definition). The rest of your business code should not see other impacts from your change.

# Note : Transaction Management

Spring provides an API for transaction management. If you plan to use spring for the data access, you should also use spring for transaction management, as they integrate together really well. For each data access technology supported by spring, there is a matching transaction manager for local transactions, or you can choose JTA if you need distributed transactions. All of them implement the same API, so that (once again) the technology choice is just a matter a configuration that can be changed without further impact on the business code.

