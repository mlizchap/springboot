# springboot



## Steps
1. **initialize a new project with the Spring initializr**
    - this can be done here: https://start.spring.io/
    - make sure project is set to Maven
    - change the project name
    - choose the following dependencies:
      - Spring Data JPA
      - Spring Web
      - Spring Boot DevTools
      - PostgresSQL Driver
      - Rest Repositories
      
   - click the generate button to download the file
2. **Open up the initialized project in Springboot**
    - the zip file should now be in your `downloads` folder or wherever you downloaded it to, once found extract the files
    - open in by one of the following ways
      - open up the Spring application and select `import from existing sources` 
      - if you're already in Springboot go to file -> new -> project from existing sources
    - you can basically use all the defaults given from the prompts (make sure the Maven project is selected)
    - at this 
3. **Set up the application properties**
    - go to src -> main -> resources and open `application.properties`
    - type the following code into this file:
      ```
      # application properties
      spring.application.name=friday
      server.port=8080

      # postgres properties
      spring.datasource.url=jdbc:postgresql://localhost:5432/<DB NAME>
      spring.datasource.username=<USER>
      spring.datasource.password=<PASSWORD>

      #jpa properties
      spring.jpa.generate-ddl=true
      spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
      spring.jpa.show-sql=true
      ```
4.  **create the database**
    - open up your terminal and type `psql` to enter the Postgres terminal
    - I like to create the db here, while Spring can do this, I've had some bugs in the past
    `CREATE DATABASE <database name>
    - create a new user with a password and grant privilages to that user
      ```     
      # CREATE USER tom WITH PASSWORD 'tom123';
      # GRANT ALL PRIVILEGES ON DATABASE springexample to tom;    
      ```
 5. **Set up the data source and drivers**
    - on the righthand sidebar you should see a `Database` tab, click on it to open it
    - in the lefthand corner of this tab you will see a plus sign, click on this and go to Data Source -> Postgres
    - fill in the appropriate user and password and then click on `Test Connection`, you may need to download the drivers
    - hit apply and close out
    - **CHECK**: run the project, 
      - if you see the project has stopped running on your own, look for the error and try to debug
      - go to `http://localhost:8080/` you should see an object like this:
        ```
         { "_links": {  "profile": { "href": "http://localhost:8080/profile" } )  }
        ```
6. **Set up the database models**
    - in in your project directory (src -> main -> java -> your project file) right click and create a new package
    - add `model` to the end of the package name
    - in the `model`folder you just created, right click to create a new class.  
      - Name it the name of your model. It is best practice to start it with an uppercase and to make it singular rather than plural. ex: `User`
    - configure the model as a **JPA entity** with `@Entity`
      ```
      import javax.persistence.Entity;

      @Entity
      public class User {

      }

      ```
    - annotate the class with a **table name**
      ```
      import javax.persistence.Table;

      @Table(name="user")
      public class User {

      }
      ```
    - add an **id property** for the model 
      - use the `@Id` and `@GeneratedValue` annotations to give it automatically generated ids
      - give it the `@Column` annotation to make the property a column in the table
      ```
      @Id
      @Column
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      private int id;
      ```
   - add the rest of the **properties**, make sure they at least have a `@Column` annotation
      ```
      @Entity
      @Table(name = "course")
      public class Course {
        @Column
        private String code;

        @Column
        private String name;
      ```
      }
   -  create the **getters** and **setters**, as well as the **constructor**
         ```
         public class Course {
          @Id
          @Column
          @GeneratedValue(strategy = GenerationType.IDENTITY)
          private int id;

          @Column
          private String code;

          @Column
          private String name;

          public Course() { }

          public int getId() { return id; }

          public void setId(int id) {  this.id = id; }     

          public String getName() { return name; }

          public void setName(String name) {  this.name = name; }         

         }
         ```
    - **CHECK**: go into the psql terminal. Go into your db (# \connect <DB NAME> and type `# \dt`.  You should now see a table corresponding to the model you just created.
    - repeat these steps for the rest of your models.
  - **Create the CRUD repository**
    - create another package in your project directory (same folder as the model) and call it 
    
**7.  
    
    
    
    
    
    
    
    
   ## Join tables
   ### One to Many
   - the **owning** side
        - usually the "one" side
        - use the `@JoinColumn` to declare the join column name (usually the foreign key in the parent
            ```
            joinColumns = {@JoinColumn(name = "user_id")}
            ```
        - can also create an inverse join column (usually the foreign key in the child), 
            - ` inverseJoinColumns = @JoinColumn(name = "course_id"))`
    
 - the **inverse/referencing**
      - usually the "many" side with the foreign key
      - use `mappedBy`, the value is the name of the entity on the owning side
        ```
        @OneToMany(fetch = FetchType.LAZY, mappedBy = "employee")
        private List<Email> emails;
        ```
 ### Many to Many
 -  https://www.baeldung.com/jpa-many-to-many
 - have each side declare the many to many relations, with the corresponding entity as a property
   ```
    @ManyToMany
    Set<Student> likes;
    
    @ManyToMany
    Set<Student> likes;
     ```
 - configure the relationship
    - in the **owning side** only
        - include the table name 
        - specify the foreing keys for the join columns, and inverse join columns
               - *note*: note this is optional, but recommended if you want  control over the nameing conventions.
            ```
            @ManyToMany
            @JoinTable(
              name = "course_like", 
              joinColumns = @JoinColumn(name = "student_id"), 
              inverseJoinColumns = @JoinColumn(name = "course_id"))
            Set<Course> likedCourses;
            ```
    - in the **target side** map the relationship with `mapped by`
        ```
        @ManyToMany(mappedBy = "likedCourses")
        Set<Student> likes;

        ```




       - specify the column property that will be joined 
       - delcare the **join type**, with the fetch and cascade types specified as arguements
       ```
       private List<Course> courses;
       ```
   
   ### Types
   - **FetchType**
        - .LAZY: (default) Lazy loading - loads only the relationships specified via the getter, can produce NULL pointers
        - .EAGER: loads ALL relationships related to the entity
        
   - **cascadeTypes**: when we perform some action on the target entity, the same action will be applied to the associated entity
        - ALL: propogates ALL operations from parent to child
        - PERSIST: propogates the PERSIST operation from parent to child, when we save the parent, the address will get saved
        - MERGE: propogates the MERGE operatin from the parent to the child
        - REMOVE: propogates the REMOVE operation from parent to child, when we delete the parent, the child will also be   deleted
        - REFRESH: whenever the parent gets reloaded from the DB, so does the child
        - DETACH: when the parent gets removed from the persistent context, so does the child


          
      


      
         

    
      

    
    


    
    
    
  
