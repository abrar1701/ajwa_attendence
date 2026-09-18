# Employee CRUD Operations using Spring Boot and Hibernate

## Aim

To create an **Employee entity** and implement **CRUD (Create, Read, Update, Delete) operations** using Spring Boot, Spring Data JPA, Hibernate, and MySQL.

---

## Objectives

* Create a Spring Boot project with required dependencies.
* Configure MySQL database connectivity.
* Create an `Employee` entity using JPA annotations.
* Use Hibernate for Object-Relational Mapping (ORM).
* Implement CRUD operations using `JpaRepository`.
* Create REST APIs for employee management.
* Test the APIs using Postman or a browser.

---

## Technologies Used

| Technology      | Purpose               |
| --------------- | --------------------- |
| Java 17+        | Programming language  |
| Spring Boot     | Application framework |
| Spring Web      | REST API development  |
| Spring Data JPA | Database operations   |
| Hibernate       | ORM framework         |
| MySQL           | Database              |
| Maven           | Dependency management |
| Postman         | API testing           |

---

## Project Structure

```text
src/main/java/com/example/employee
│
├── EmployeeApplication.java
│
├── controller
│   └── EmployeeController.java
│
├── entity
│   └── Employee.java
│
└── repository
    └── EmployeeRepository.java

src/main/resources
└── application.properties
```

---

## Create the Spring Boot Project

Create a project using **Spring Initializr** with the following dependencies:

* Spring Web
* Spring Data JPA
* MySQL Driver

---

## Maven Dependencies

Add the following dependencies to `pom.xml`:

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

</dependencies>
```

---

## Database Configuration

Create a MySQL database:

```sql
CREATE DATABASE employee_db;
```

Configure `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/employee_db
spring.datasource.username=root
spring.datasource.password=your_password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### Configuration Explanation

* `spring.datasource.url` – Specifies the MySQL database URL.
* `spring.datasource.username` – MySQL username.
* `spring.datasource.password` – MySQL password.
* `spring.jpa.hibernate.ddl-auto=update` – Allows Hibernate to create/update tables based on entities.
* `spring.jpa.show-sql=true` – Displays generated SQL queries in the console.
* `hibernate.format_sql=true` – Formats SQL queries for readability.

---

## Create Employee Entity

Create `Employee.java` inside the `entity` package.

```java
package com.example.employee.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;
    private String department;
    private double salary;

    public Employee() {
    }

    public Employee(String name, String email,
                    String department, double salary) {
        this.name = name;
        this.email = email;
        this.department = department;
        this.salary = salary;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
}
```

### Important Annotations

| Annotation        | Purpose                           |
| ----------------- | --------------------------------- |
| `@Entity`         | Marks the class as a JPA entity   |
| `@Table`          | Specifies the database table name |
| `@Id`             | Specifies the primary key         |
| `@GeneratedValue` | Automatically generates the ID    |

Hibernate uses this entity to map the Java `Employee` object to the `employees` database table.

---

## Create Repository

Create `EmployeeRepository.java`:

```java
package com.example.employee.repository;

import com.example.employee.entity.Employee;
import org.springframework.data.jpa.repository.JpaRepository;

public interface EmployeeRepository
        extends JpaRepository<Employee, Long> {
}
```

`JpaRepository` provides predefined methods for database operations.

Some important methods are:

```text
save()
findAll()
findById()
existsById()
deleteById()
```

Therefore, we do not need to write SQL queries for basic CRUD operations.

---

## Create REST Controller

Create `EmployeeController.java`:

```java
package com.example.employee.controller;

import com.example.employee.entity.Employee;
import com.example.employee.repository.EmployeeRepository;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/employees")
public class EmployeeController {

    private final EmployeeRepository repository;

    public EmployeeController(EmployeeRepository repository) {
        this.repository = repository;
    }

    // CREATE
    @PostMapping
    public Employee createEmployee(@RequestBody Employee employee) {
        return repository.save(employee);
    }

    // READ - All employees
    @GetMapping
    public List<Employee> getAllEmployees() {
        return repository.findAll();
    }

    // READ - Employee by ID
    @GetMapping("/{id}")
    public ResponseEntity<Employee> getEmployeeById(
            @PathVariable Long id) {

        return repository.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // UPDATE
    @PutMapping("/{id}")
    public ResponseEntity<Employee> updateEmployee(
            @PathVariable Long id,
            @RequestBody Employee updatedEmployee) {

        return repository.findById(id)
                .map(employee -> {

                    employee.setName(updatedEmployee.getName());
                    employee.setEmail(updatedEmployee.getEmail());
                    employee.setDepartment(
                            updatedEmployee.getDepartment());
                    employee.setSalary(updatedEmployee.getSalary());

                    return ResponseEntity.ok(
                            repository.save(employee));
                })
                .orElse(ResponseEntity.notFound().build());
    }

    // DELETE
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteEmployee(
            @PathVariable Long id) {

        if (!repository.existsById(id)) {
            return ResponseEntity.notFound().build();
        }

        repository.deleteById(id);

        return ResponseEntity.noContent().build();
    }
}
```

---

## CRUD Operations

The application provides the following REST APIs:

| Operation | HTTP Method | Endpoint          | Description        |
| --------- | ----------- | ----------------- | ------------------ |
| Create    | `POST`      | `/employees`      | Add an employee    |
| Read All  | `GET`       | `/employees`      | Get all employees  |
| Read One  | `GET`       | `/employees/{id}` | Get employee by ID |
| Update    | `PUT`       | `/employees/{id}` | Update employee    |
| Delete    | `DELETE`    | `/employees/{id}` | Delete employee    |

---

## Create Employee

### Request

```http
POST http://localhost:8080/employees
```

### JSON Body

```json
{
    "name": "Rahul Kumar",
    "email": "rahul@example.com",
    "department": "IT",
    "salary": 55000
}
```

### Example Response

```json
{
    "id": 1,
    "name": "Rahul Kumar",
    "email": "rahul@example.com",
    "department": "IT",
    "salary": 55000
}
```

---

## Read Employees

### Get All Employees

```http
GET http://localhost:8080/employees
```

Example response:

```json
[
    {
        "id": 1,
        "name": "Rahul Kumar",
        "email": "rahul@example.com",
        "department": "IT",
        "salary": 55000
    }
]
```

### Get Employee by ID

```http
GET http://localhost:8080/employees/1
```

If employee `1` exists, the employee details are returned.

If the employee does not exist:

```text
404 Not Found
```

---

## Update Employee

### Request

```http
PUT http://localhost:8080/employees/1
```

### JSON Body

```json
{
    "name": "Rahul Kumar",
    "email": "rahul@example.com",
    "department": "Development",
    "salary": 60000
}
```

The existing employee with ID `1` is updated.

---

## Delete Employee

### Request

```http
DELETE http://localhost:8080/employees/1
```

The employee with ID `1` is deleted from the database.

Successful deletion returns:

```text
204 No Content
```

If the employee does not exist:

```text
404 Not Found
```

---

## Database Table

Hibernate creates the `employees` table based on the `Employee` entity.

Example:

| id | name        | email                                         | department | salary |
| -: | ----------- | --------------------------------------------- | ---------- | -----: |
|  1 | Rahul Kumar | [rahul@example.com](mailto:rahul@example.com) | IT         |  55000 |

---

## Application Architecture

```text
             Client / Postman
                    |
                    v
          +--------------------+
          | EmployeeController |
          +--------------------+
                    |
                    v
          +--------------------+
          | EmployeeRepository |
          +--------------------+
                    |
                    v
          +--------------------+
          | Spring Data JPA    |
          +--------------------+
                    |
                    v
          +--------------------+
          | Hibernate ORM      |
          +--------------------+
                    |
                    v
          +--------------------+
          | JDBC Driver        |
          +--------------------+
                    |
                    v
          +--------------------+
          | MySQL Database     |
          +--------------------+
```

---

## Execution Flow

For example, when a user sends:

```http
POST /employees
```

the following process takes place:

1. The client sends employee information as JSON.
2. `EmployeeController` receives the request.
3. `@RequestBody` converts JSON into an `Employee` object.
4. The controller calls `repository.save(employee)`.
5. Spring Data JPA processes the repository operation.
6. Hibernate converts the Java object operation into SQL.
7. JDBC sends the SQL query to MySQL.
8. MySQL stores the employee record.
9. The saved employee is returned as a JSON response.

---

## CRUD Method Mapping

```text
CREATE
POST
   ↓
repository.save()

READ
GET
   ↓
repository.findAll()
repository.findById()

UPDATE
PUT
   ↓
repository.findById()
repository.save()

DELETE
DELETE
   ↓
repository.deleteById()
```

---

## How Hibernate Helps

Hibernate acts as an **Object-Relational Mapping (ORM)** framework.

Instead of manually writing SQL such as:

```sql
INSERT INTO employees
(name, email, department, salary)
VALUES (...);
```

we can use:

```java
repository.save(employee);
```

Hibernate generates the appropriate SQL and communicates with the database.

This reduces boilerplate database code and allows developers to work primarily with Java objects.

---

## Testing

The APIs can be tested using **Postman**.

### Test Sequence

```text
1. Start MySQL
       ↓
2. Start Spring Boot application
       ↓
3. POST /employees
       ↓
4. GET /employees
       ↓
5. PUT /employees/1
       ↓
6. GET /employees/1
       ↓
7. DELETE /employees/1
```

---

## Result

The **Employee entity** was successfully created and mapped to a MySQL table using Hibernate.

CRUD operations were implemented using **Spring Data JPA** and exposed through REST APIs.

The application can:

* Create employee records.
* Retrieve employee records.
* Update employee records.
* Delete employee records.

---

## Conclusion

This project demonstrates the integration of **Spring Boot, Spring Data JPA, Hibernate, and MySQL** to develop a database-driven REST application.

Hibernate provides the ORM functionality, while `JpaRepository` simplifies database operations. REST controllers expose the CRUD functionality through HTTP endpoints, making the application simple to test and extend.
