---
layout: post
title: Blogging Like a Hacker
---

# Customizing Spring Data MongoDB
Spring Data has simplified the way we write data access layer. In just one steps we can get a full blown CRUD  implementation ready.

###### *Write an interface which extands specific Repository Interface in this case we will use MongoRepository.*

```java
@RepositoryRestResource
public interface PersonRepository extends MongoRepository<String,Person>  {
}
```

> Spring under the hood will provide the implementation of repository in runtime.    
> This removes lot of boiler plate code and makes maintenance a lot easier.  


The authors of Spring Data took it a step ahead, they have allowed us to leverage this feature of Spring and write our own methods specific to the business use cases.

Spring Data provides a nomenclature for naming our custom abstract methods. If we do so it will provide the implementation of those methods in runtime.

#### In this post we will learn two ways to write custom methods in Spring Data.
1. Writing custom abstract methods _(whose implementation will be provided by Spring in runtime)_-.
2. Writing abstract methods with custom query using `@Query` annotation.

## Basic Setup
We have Person Collection with usual attributes and an embedded
Address object.
```java

import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.data.annotation.Id;

@Document
public class Person{

    @Id
    private String id;

    private String firstname;

    private String lastname;

    private String emailId;

    private int age;

    private Address address;

    //getters and setters for the above attributes
}

 class Address{

  private String streetNum;

  private String locality;

  private String state;

  private String country;

  private int pincode;

  //getters and setters for the above attributes
}

```

We also have to write a standard SpringDataRest repository interface  called `PersonRepository`. Spring will provide implementation of this interface in runtime exposing basic CRUD operation as a REST API.
```java
@RepositoryRestResource
public interface PersonRepository extends MongoRepository<String,Person>  {
}
```
- - - -
## Writing Custom Query in Spring Data

#### We want to find a Person by age and locality.
To do so we will first write an interface called `PersonRepositoryCustom`
having a method `fetchPersonByAgeAndLocality`
```java
import com.test.model.Person;

public interface PersonRepositoryCustom
{
  public List<Person> fetchPersonByAgeAndLocality(int age, String locality);
}
```

Then we will create an implementation Class of the `PersonRepositoryCustom` interface  called `PersonRepositoryImpl`

Spring has a standard naming convention for the implementation class, it should start with the Repository name and should have `Impl`  appended to it.

Example : `<RepositoryName>Impl` so in this case it would be `PersonRepositoryImpl`  

> Note: Spring tries to auto detect custom implementations by scanning for classes under the package where repository interface is present.  

```java
import com.test.model.Person;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoOperations;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;

public class PersonRepositoryImpl implements PersonRepositoryCustom
{
  private final MongoOperations operations;

	  @Autowired
	  public PersonRepositoryImpl(MongoOperations operations) {
	    this.operations = operations;
	  }

    @Override
    	public List<Person> fetchPersonByAgeAndLocality(int age, String locality) {
 Query searchQuery = new Query();               searchQuery.addCriteria(Criteria.where("age").is(age));
searchQuery.addCriteria(Criteria.where("address.locality").is(locality));

List<Person> people = operations.find(searchQuery, Person.class);
return people;
}
}
```


Finally we will integrate our `PersonRepository` with custom implementation that we wrote.
We will do so by making `PersonRepository` extend `PersonRepositoryCustom`.

```java
import com.test.model.Person;
import org.springframework.data.mongodb.repository.MongoRepository;
import org.springframework.data.rest.core.annotation.RepositoryRestResource;

@RepositoryRestResource
public interface PersonRepository extends MongoRepository<String,Person> , PersonRepositoryCustom {

}

```
#####  That is all we need to write custom query in Spring Data. 


 
> We may choose not to follow this standard and having our own keyword instead of `Impl`. In such case we have to define that keyword *(in this case let’s say `Implementation ` )* as namespace element’s attribute **repository-impl-postfix**  as below

```xml
<repositories base-package="com.test.repository" />

<repositories base-package="com.test.repository" repository-impl-postfix="Implementation" />

```

Spring will then try to look for class having name ending with `Implementation`
Example : `PersonRepositoryImplementation`

Try it out and let me know if you have any comments.

Happy Coding :)
