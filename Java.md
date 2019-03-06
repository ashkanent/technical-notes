# Java Basics

* Getting user input:
```Java
Scanner userInput = new Scanner(System.in);
if (userInput.hasNextLine()) {
	this.name = userInput.nextLine();
}
```

* Printing in Java:
```Java
System.out.println("Hello World!");
```

* defining a class:
```Java
public class Animal {
	public static final double FAV_NUMBER = 1.61;
	private Boolean hasTail = true;
	protected static int numberOfAnimals = 0;

	public Animal() {
		numberOfAnimals++;
	}
}
```

* Inheritance:  
```Java
public class Cat extends Animal {
}
```

* to make our class executable:
```java
public class ExecutableClass {

	public static void main(String[] args) {
		Animal theAnimal = new Animal();
	}
}
```

* writing switch/case:
```Java
switch(input) {
	case 8:
		System.out.println("eight");
		break;
	case 9:
		System.out.println("nine");
		break;
	default:
		System.out.println("seven");
		break;
}
```

# JDBC
* It stands for Java Database Connectivity
* Simply put, when you want to interact with databases in your Java application, you use JDBC.
	* to _connect_ "java" to your _database_
* steps in establishing the connection:
	1. import the package  -----> `java.sql.*`
	2. load and register the driver  
    2.a. load ---> from `mysql-connector`  
		2.b. register ---> `Class.forName("com.mysql.jdbc.Driver")`
	3. establish the connection --> we have a 'Connection' interface that we need to instantiate
	4. create the statement ---> we have 1) statement, 2) preparedStatement and 3) callableStatement
	5. execute the query
	6. process result
	7. close

* Sample implementation in _Java_:
```java
import java.sql.*;
public class DemoClass {
	public static void main(String[] args) throws Exception {
		String url = "jdbc:mysql://localhost:3307/ashkan"
		String userName = "root";
		String pass = "";
		String query = "select username from student where userId=3";

		Class.forName("com.mysql.jdbc.Driver");
		Connection con = DriverManager.getConnection(url, userName, pass);
		Statement st = con.createStatement();
		ResultSet rs = st.executeQuery(query);

		rs.next();
		String name = rs.getString("username");

		st.close();
		con.close();
	}
}
```

## DAO and DTO
* _DAO_: Data Access Object
* _DTO_: Data Transfer Object
* in a proper architecture in which we want to access data, we create a DAO for the object and whenever we need to access data, we use the DAO class:
  * if we have **_Student_** object and we store it in our database, we create the **StudentDAO** class
	* this class can have a method like **getStudent()**.
	* body of this method should be similar to JDBC example in last section
	* this method returns **StudentDTO** which is another class we create that contains Student information
	* anywhere in our code that we want to get access to student, we simply call **getStudent()**.

# Hibernate
* It is meant to make it easier to work with objects and storing/retrieving from DB
* So instead of the seven steps in JDBC, we can say something like _save(theStudent)_ and it will store the _Student_ object in the database.
* We need to Understand **ORM** or _Object Relational Mapping_.
* Hibernate is an ORM tool that helps us to create these mappings (like between _Student_ class and _Student_ table in our database).

## Implementation
* for libraries/dependencies, we need 'Core Hibernate' and 'mysql connector'
* simple example:
 ```Java
 import org.hibernate.*;

 public class App {
	 public static void main(String[] args) {
		 Student theStudent = new Student(); // Student is a POJO, simple class that we define
		 theStudent.setStudentId(101);
		 theStudent.setStudentName("Ashkan");
		 theStudent.setMark(80);

		 Configuration conf = new Configuration().configure()
		 													.addAnnotatedClass(Student.class);
		 ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(conf.getProperties())
		 													.buildServiceRegistry();
		 SessionFactory sessionFactory = conf.buildSessionFactory(serviceRegistry);
		 Session session = sessionFactory.openSession();
		 Transaction transaction = session.beginTransaction();
		 session.save(theStudent);
		 transaction.commit();
	 }
 }
 ```
 also we need to define _Student_ as well:
 ```java
 import javax.persistense.Entity;
 import javax.persistense.Id;

 @Entity
 public class Student {
	 @Id // to specify a primary key
	 private int studentId;
	 private String studentName;
	 private int mark;

	 // also implement setter/getters
 }
 ```
* if you want your table name to be different from class name:
```java
@Entity(name="student_table")
public class Student {
	// ...
}
```
  * this is actually entity name that we are changing (that will be used for table)
  * you can also just change table name (`@Table(name="whatever")`)
* some other options that we have:
  ```java
  import javax.persistense.Entity;
  import javax.persistense.Id;

  @Entity
	@Table(name="student_table")
  public class Student {
 	 @Id // to specify a primary key
 	 private int studentId;
	 @Transient // it won't be stored in db
 	 private String studentName;
	 @Column(name="student_mark") // specify column name
 	 private int mark;

 	 // also implement setter/getters

	 // we use this method for fetching:
	 @Override
	 public String toString() {
		 return "Student [studentId=" + studentId + ", studentName=" + studentName + ", mark=" + mark + "]";
	 }
  }
  ```
* to fetch, you do this:
  ```java
	Student theStudent;
	// hibernate conf...
	theStudent = (Student)session.get(Student.class, 101); // to fetch from db
	```

	# JPA
	- It stands for _Java Persistence API_
	- We use it to persist data into our database. We use **ORM** (Object Relational Mapping) which maps class name to table name and class properties to table columns.
	- to use ORM in different languages, there are different tools. In Java we have Hibernate, iBatis, TopLink, etc.
	- JPA is only a specification. to use that we need a tool like Hibernate.

	# Spring Boot
	- Spring Framework is a framework that lets you write enterprise java applications.
	- Spring Boot is a tool that lets you easily start a Spring Framework application
	  - Spring Boot makes it easy to create stand-alone, production-grade Spring based applications that you can just run!

	```Java
	@SpringBootApplication // to specify the starting point for spring boot app
	public class MainApp {
		public static void main(String[] args) {
			SpringApplication.run(MainApp.class, args); // this will start the application
		}
	}
	```
	creating a REST controller:

	```Java
	@RestController
	public class HelloController {

		@RequestMapping("/hello") // whenever there is http request to this app in this url, execute this
		public String sayHi() {
			return "Hi";
		}
	}
	```
	- Note: `@RequestMapping` maps by default to all HTTP methods (get, post, ...)
	- if we return a list of object (instead of String), Spring will automatically convert it to JSON

## Simple Web Application
- This is simple web app. We have exposed 4 end points.
- One endpoint, gets all the topics. One adds a new one. One deletes and the other updates an existing topic.
- In this example we don't use JPA, so everything is stored in memory!

```Java
@Service
public class TopicService {
	private List<Topic> topics = new ArrayList<>(Arrays.asList(
		new Topic("spring", "Spring Framework", "descr"),
		new Topic("java", "Core Java", "descr")
	));

	public List<Topic> getAllTopics() {
		return topics;
	}

	public Topic getTopic(String id) {
		topics.stream().filter(t -> t.getId().equals(id)).findFirst().get();
	}

	public void addTopic(Topic topic) {
		topics.add(topic);
	}

	public void updateTopic(String id, Topic topic) {
		for (int i = 0; i < topics.size(); i++) {
			topic t = topics.get(i);
			if (t.getId().equals(id)) {
				topics.set(i, topic);
				return;
			}
		}
	}

	public void deleteTopic(String id) {
		topics.removeIf(t -> t.getId().equals(id));
	}
}
```
```Java
@RestController
public class TopicController {
	@Autowired // to tell spring to inject this dependency
	private TopicService topicService;

	@RequestMapping("/topics")
	public List<Topic> getAllTopics() {
		topicService.getAllTopics();
	}

	@RequestMapping("/topics/{id}") // {id} will tell Spring framework that this is a param
	public Topic getTopic(@PathVariable String id) {
		return topicService.getTopic(id);
	}

	@RequestMapping(method=RequestMethod.POST, value="/topics")
	public void addTopic(@RequestBody Topic topic) {
		topicService.addTopic(topic);
	}

	@RequestMapping(method=RequestMethod.PUT, value="/topics/{id}")
	public void updateTopic(@RequestBody Topic topic, @PathVariable String id) {
		topicService.updateTopic(id, topic);
	}

	@RequestMapping(method=RequestMethod.DELETE, value="/topics/{id}")
	public void deleteTopic(@PathVariable String id) {
		topicService.deleteTopic(id);
	}
}
```
**Notes**:
- `@PathVariable` is to tell spring that the method argument should be mapped to the path variable passed through the URI. If the method argument name is exactly the same as path variable, it will work, if it's something different we should specify it like this: `@PathVariable("foo") String id` if the variable in URI is called `foo`



## Simple Spring JPA App
- We use the above example and modify it to use DB to store data

```Java
@Entity
public class Topic {
	@Id // to tell the DB this is the primary key
	private String id;
	private String name;
	private String description;

	public Topic() {}

	public Topic(String id, String name, String description) {
		super();
		this.id = id;
		this.name = name;
		this.description = description;
	}

	// implement setters and getters...
}
```

```Java
public interface TopicRepository extends CrudRepository<Topic, String> {

}
```

```Java
@Service
public class TopicService {

	@Autowired
	private TopicRepository topicRepository;

	public List<Topic> getAllTopics() {
		List<Topic> topics = new ArrayList<>();
		topicRepository.findAll().forEach(topics::add);
		return topics;
	}

	public Topic getTopic(String id) {
		return topicRepository.findOne(id);
	}

	public void addTopic(Topic topic) {
		topicRepository.save(topic);
	}

	public void updateTopic(String id, Topic topic) {
		topicRepository.save(topic);
	}

	public void deleteTopic(String id) {
		topicRepository.delete(id);
	}
}
```
```Java
@RestController
public class TopicController {
	@Autowired // to tell spring to inject this dependency
	private TopicService topicService;

	@RequestMapping("/topics")
	public List<Topic> getAllTopics() {
		topicService.getAllTopics();
	}

	@RequestMapping("/topics/{id}") // {id} will tell Spring framework that this is a param
	public Topic getTopic(@PathVariable String id) {
		return topicService.getTopic(id);
	}

	@RequestMapping(method=RequestMethod.POST, value="/topics")
	public void addTopic(@RequestBody Topic topic) {
		topicService.addTopic(topic);
	}

	@RequestMapping(method=RequestMethod.PUT, value="/topics/{id}")
	public void updateTopic(@RequestBody Topic topic, @PathVariable String id) {
		topicService.updateTopic(id, topic);
	}

	@RequestMapping(method=RequestMethod.DELETE, value="/topics/{id}")
	public void deleteTopic(@PathVariable String id) {
		topicService.deleteTopic(id);
	}
}
```
