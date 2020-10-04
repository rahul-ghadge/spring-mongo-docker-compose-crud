# spring-mongo-docker-compose-crud

Docker Compose runs multi-container Docker application. It lets us to run multiple docker container with single docker-compose command by reading configuration file.
Here we are using multiple containers which are depends on each other(*Mongo container with REST APIs using Spring boot application custom container*).  
This project explains dockerized CRUD (**C**reate, **R**ead, **U**pdate, **D**elete) operations with MongoTemplate and MongoRepository using spring boot and mongo DB.
In this app we are using Spring Data JPA for built-in methods to do CRUD operations and Mongo queries using MongoTemplate.     
`@EnableMongoRepositories` annotation is used on main class to Enable Mongo related configuration, which will read properties from `application.yml` file.



## Prerequisites 
- Java
- [Spring Boot](https://spring.io/projects/spring-boot)
- [Maven](https://maven.apache.org/guides/index.html)
- [Mongo DB](https://docs.mongodb.com/guides/)
- [Lombok](https://objectcomputing.com/resources/publications/sett/january-2010-reducing-boilerplate-code-with-project-lombok)
- [Docker Compose](https://docs.docker.com/compose/)


## Tools
- Eclipse or IntelliJ IDEA (or any preferred IDE) with embedded Gradle
- Maven (version >= 3.6.0)
- Postman (or any RESTful API testing tool)


<br/>


###  Build application
**cd /absolute-path-to-directory/spring-mongo-docker-compose-crud**  
and try below command in terminal
> **```mvn clean install```** it will build application and create **jar** file under target directory 


### Docker-compose installation
To download and install `docker-compose` on Ubuntu visit [this link](https://docs.docker.com/compose/install) and check docker-compose version by below command  
> docker-compose --version


Run below docker-compose command to download and up/deploy containers  
> docker-compose up --force-recreate  


List the containers and check the status with this command  
> docker-compose ps


To view the container logs, run this command  
> docker-compose logs


To connect mongo CLI via terminal run below  
> docker exec -it mongo bash



||
|  ---------    |
| **_Note_** : In `SpringBootMongoDBApplication.java` class we have autowired both SuperHero and Employee repositories. <br/>If there is no record present in DB for any one of that module class (Employee or SuperHero), static data is getting inserted in DB from `HelperUtil.java` class when we are starting the app for the first time.| 



### Code Snippets

1. #### Dockerized multi-container Config
    Two containers i.e. **mongo** and **smdc-crud-superhero-service** will get deployed by running `docker-command up` command which are depends on each other.  
    **smdc-crud-superhero-service** container is referring **superhero-service** sub-module to deploy jar file in docker by reading **Dockerfile** placed in same sub-module. 
    
    ```
    version: '3.8'
    
    services:
      mongo:
        image: mongo:4.2.10
        container_name: mongo
        restart: always
        ports:
          - 27017:27017
        network_mode: host
        volumes:
          - $HOME/mongo:/data/db
        healthcheck:
          test: "exit 0"
    
      superhero-service:
        container_name: smdc-crud-superhero-service
        build: superhero-service
        image: superhero-service
        depends_on:
          - mongo
        network_mode: "host"
        hostname: localhost
        restart: always
        ports:
          - 8088:8088
        healthcheck:
          test: "exit 0"
    ```



2. #### Dockerfile to create Spring boot mongo applications jar file in Docker
    This configuration will download openjdk-8 image and deploy jar file. Same image will be created by reading below configuration and this image will be
    passed to docker-compose mentioned above.  
    ```
    FROM openjdk:8-jdk-alpine
    VOLUME /tmp
    ARG JAR_FILE=target/*.jar
    COPY ${JAR_FILE} spring-mongo-app.jar
    ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/spring-mongo-app.jar"]
    ```

3. #### Maven Dependencies
    Need to add below dependencies to enable Mongo related config in **pom.xml**. Lombok's dependency is to get rid of boiler-plate code.   
    ```
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>
   
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    ```
   
   
4. #### Properties file
    Reading Mongo DB related properties from **application.yml** file and configuring Mongo connection factory for mongoDB under **superhero-service** module.  

    **src/main/resources/application.yml**
     ```
    server:
        port: 8088
    spring:
        data:
            mongodb:
                host: localhost
                port: 27017
                database: spring_boot_mongo_app
        application:
            name: superhero-service 
     ```
   
   
5. #### Model class
    Below are the model classes which we will store in MongoDB and perform CRUD operations.  
    **com.spring.mongo.demo.model.SuperHero.java**  
    ```
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    @Document(collection = "super_hero")
    public class SuperHero implements Serializable {
    
        @Id
        private String id;
    
        private String name;
        private String superName;
        private String profession;
        private int age;
        private boolean canFly;
    
        // Constructor, Getter and Setter
    }
    ```
    **com.spring.mongo.demo.model.Employee.java**  
    ```
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @Builder
    public class Employee implements Serializable {
   
        @Id
        private String id;
       
        private int empId;
        private String firstName;
        private String lastName;
        private float salary;
        
        // Constructor, Getter and Setter
    }
   ```
   
   
6. #### CRUD operation for Super Heroes

    In **com.spring.mongo.demo.controller.SuperHeroController.java** class, 
    we have exposed 5 endpoints for basic CRUD operations
    - GET All Super Heroes
    - GET by ID
    - POST to store Super Hero in DB
    - PUT to update Super Hero
    - DELETE by ID
    
    ```
    @RestController
    @RequestMapping("/super-hero")
    public class SuperHeroController {
        
        @GetMapping
        public ResponseEntity<List<?>> findAll();
    
        @GetMapping("/{id}")
        public ResponseEntity<?> findById(@PathVariable String id);
    
        @PostMapping
        public ResponseEntity<?> save(@RequestBody SuperHero superHero);
    
        @PutMapping
        public ResponseEntity<?> update(@RequestBody SuperHero superHero);
    
        @DeleteMapping("/{id}")
        public ResponseEntity<?> delete(@PathVariable String id);
    }
    ```
   
    In **com.spring.mongo.demo.repository.SuperHeroRepository.java**, we are extending `MongoRepository<Class, ID>` interface which enables CRUD related methods.
    ```
    public interface SuperHeroRepository extends MongoRepository<SuperHero, String> {
    }
    ```
   
   In **com.spring.mongo.demo.service.impl.SuperHeroServiceImpl.java**, we are autowiring above interface using `@Autowired` annotation and doing CRUD operation.



7. #### JPA And Query operation for Employee
    In **com.spring.mongo.demo.controller.EmployeeController.java** class JPA related queries API Endpoints are placed.
    In **com.spring.mongo.demo.controller.EmployeeQueryController.java** class Mongo queries API Endpoints are placed.
    
 
 
    
### API Endpoints

- #### Super Hero CRUD Operations
    > **GET Mapping** http://localhost:8088/super-hero  - Get all Super Heroes
    
    > **GET Mapping** http://localhost:8088/super-hero/5f380dece02f053eff29b986  - Get Super Hero by ID
       
    > **POST Mapping** http://localhost:8088/super-hero  - Add new Super Hero in DB  
    
      Request Body  
      ```
        {
            "name": "Tony",
            "superName": "Iron Man",
            "profession": "Business",
            "age": 50,
            "canFly": true
        }
      ```
    
    > **PUT Mapping** http://localhost:8088/super-hero  - Update existing Super Hero for given ID 
                                                       
      Request Body  
      ```
        {
            "id": "5f380dece02f053eff29b986"
            "name": "Tony",
            "superName": "Iron Man",
            "profession": "Business",
            "age": 50,
            "canFly": true
        }
      ```
    
    > **DELETE Mapping** http://localhost:8088/super-hero/5f380dece02f053eff29b986  - Delete Super Hero by ID

- #### Employee Get Operations using JPA
    > **GET Mapping** http://localhost:8088/employee-jpa  - Get all Employees 
    
    > **GET Mapping** http://localhost:8088/employee-jpa/5f380dece02f053eff29b986  - Get Employee by ID
    
    > **GET Mapping** http://localhost:8088/employee-jpa/firstName/Rahul  - Get All Employees with firstname as Rahul 
    
    > **GET Mapping** http://localhost:8088/employee-jpa/one-by-firstName/Rahul  - Get **ONE** Employee with firstname as Rahul 
    
    > **GET Mapping** http://localhost:8088/employee-jpa/firstName-like/Rahul  - Get All Employees which contains Rahul in their firstname 
    
    > **GET Mapping** http://localhost:8088/employee-jpa/one-by-lastName/Ghadage  - Get **ONE** Employee with lastname as Ghadage 
    
    > **GET Mapping** http://localhost:8088/employee-jpa/salary-greater-than/10000  - Get All Employees whose salary is grater than 1000 
    
    > **POST Mapping** http://localhost:8088/employee-jpa/get-by-condition  - Get All Employees with multiple condition 
                                                           
    Request Body  
    ```
    {
        "id": "5f380dece02f053eff29b986"
        "empId": "1",
        "firstName": "Rahul",
        "lastName": "Khan",
        "salary": 5000
    }
    ``` 

- #### Employee Get Operations using Queries
    > **GET Mapping** http://localhost:8088/employee-query  - Get all Employees 
    
    > **GET Mapping** http://localhost:8088/employee-query/firstName/Rahul  - Get All Employees with firstname as Rahul 
    
    > **GET Mapping** http://localhost:8088/employee-query/one-by-firstName/Rahul  - Get **ONE** Employee with firstname as Rahul 
    
    > **GET Mapping** http://localhost:8088/employee-query/firstName-like/Rahul  - Get All Employees which contains Rahul in their firstname 
    
    > **GET Mapping** http://localhost:8088/employee-query/one-by-lastName/Ghadage  - Get **ONE** Employee with lastname as Ghadage 
    
    > **GET Mapping** http://localhost:8088/employee-query/salary-greater-than/10000  - Get All Employees whose salary is grater than 1000 
    
    > **POST Mapping** http://localhost:8088/employee-query/get-by-condition  - Get All Employees with multiple condition 
                                                           
    Request Body  
    ```
    {
        "id": "5f380dece02f053eff29b986"
        "empId": "1",
        "firstName": "Rahul",
        "lastName": "Khan",
        "salary": 5000
    }
    ``` 
