# Basic-Web-Application-using-Java-Spring-Boot-and-Thymeleaf
Basic Web Application using Java, Spring Boot, and Thymeleaf

Alright — let’s go step-by-step and I’ll provide you **a complete Java + Spring Boot web app scaffold** that’s **dynamic, responsive**, and **ready to extend**.

We’ll use:

* **Spring Boot (Java)** for backend (REST + MVC)
* **Thymeleaf** for dynamic server-side HTML rendering
* **Bootstrap 5** for responsiveness
* **H2 / MySQL** for persistence (we’ll start with H2 for simplicity)
* **Spring Data JPA** for ORM

---

## 🏗️ Project Overview

We’ll build a simple **Task Management App**:

* Users can **view**, **add**, **edit**, and **delete** tasks.
* Tasks stored in a database.
* UI fully responsive with Bootstrap.

---

## 1️⃣ Create a Spring Boot Project

Use [Spring Initializr](https://start.spring.io/) or Maven:

**Dependencies:**

* Spring Web
* Thymeleaf
* Spring Data JPA
* H2 Database
* Spring Boot DevTools (optional, for live reload)

**Command (Maven CLI):**

```bash
mvn archetype:generate -DgroupId=com.example -DartifactId=taskapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Or use **Spring Initializr** to generate a project zip and import it into IntelliJ / Eclipse.

---

## 2️⃣ Project Structure

```
taskapp/
├── src/
│   ├── main/
│   │   ├── java/com/example/taskapp/
│   │   │   ├── TaskAppApplication.java
│   │   │   ├── controller/TaskController.java
│   │   │   ├── model/Task.java
│   │   │   └── repository/TaskRepository.java
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── templates/
│   │       │   ├── index.html
│   │       │   ├── add-task.html
│   │       │   └── edit-task.html
│   │       └── static/
│   │           └── css/style.css
└── pom.xml
```

---

## 3️⃣ `pom.xml`

Here’s the base Maven configuration:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>taskapp</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.2</version>
    </parent>

    <dependencies>
        <!-- Web + Thymeleaf -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Database + JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- Optional: Hot Reload -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## 4️⃣ Application Class

`TaskAppApplication.java`

```java
package com.example.taskapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TaskAppApplication {
    public static void main(String[] args) {
        SpringApplication.run(TaskAppApplication.class, args);
    }
}
```

---

## 5️⃣ Model

`Task.java`

```java
package com.example.taskapp.model;

import jakarta.persistence.*;
import java.time.LocalDate;

@Entity
public class Task {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;
    private String description;
    private LocalDate dueDate;
    private boolean completed = false;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getTitle() { return title; }
    public void setTitle(String title) { this.title = title; }

    public String getDescription() { return description; }
    public void setDescription(String description) { this.description = description; }

    public LocalDate getDueDate() { return dueDate; }
    public void setDueDate(LocalDate dueDate) { this.dueDate = dueDate; }

    public boolean isCompleted() { return completed; }
    public void setCompleted(boolean completed) { this.completed = completed; }
}
```

---

## 6️⃣ Repository

`TaskRepository.java`

```java
package com.example.taskapp.repository;

import com.example.taskapp.model.Task;
import org.springframework.data.jpa.repository.JpaRepository;

public interface TaskRepository extends JpaRepository<Task, Long> {
}
```

---

## 7️⃣ Controller

`TaskController.java`

```java
package com.example.taskapp.controller;

import com.example.taskapp.model.Task;
import com.example.taskapp.repository.TaskRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

@Controller
public class TaskController {

    @Autowired
    private TaskRepository taskRepository;

    @GetMapping("/")
    public String index(Model model) {
        model.addAttribute("tasks", taskRepository.findAll());
        return "index";
    }

    @GetMapping("/task/new")
    public String newTask(Model model) {
        model.addAttribute("task", new Task());
        return "add-task";
    }

    @PostMapping("/task/save")
    public String saveTask(@ModelAttribute Task task) {
        taskRepository.save(task);
        return "redirect:/";
    }

    @GetMapping("/task/edit/{id}")
    public String editTask(@PathVariable Long id, Model model) {
        model.addAttribute("task", taskRepository.findById(id).orElseThrow());
        return "edit-task";
    }

    @PostMapping("/task/update/{id}")
    public String updateTask(@PathVariable Long id, @ModelAttribute Task task) {
        task.setId(id);
        taskRepository.save(task);
        return "redirect:/";
    }

    @GetMapping("/task/delete/{id}")
    public String deleteTask(@PathVariable Long id) {
        taskRepository.deleteById(id);
        return "redirect:/";
    }
}
```

---

## 8️⃣ Thymeleaf Templates

### `templates/index.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Task Manager</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="container py-4">
    <h2 class="text-center mb-4">Task Manager</h2>
    <a href="/task/new" class="btn btn-primary mb-3">Add Task</a>
    <table class="table table-striped">
        <thead><tr><th>Title</th><th>Due Date</th><th>Status</th><th>Actions</th></tr></thead>
        <tbody>
        <tr th:each="task : ${tasks}">
            <td th:text="${task.title}"></td>
            <td th:text="${task.dueDate}"></td>
            <td th:text="${task.completed ? 'Completed' : 'Pending'}"></td>
            <td>
                <a th:href="@{'/task/edit/' + ${task.id}}" class="btn btn-warning btn-sm">Edit</a>
                <a th:href="@{'/task/delete/' + ${task.id}}" class="btn btn-danger btn-sm">Delete</a>
            </td>
        </tr>
        </tbody>
    </table>
</body>
</html>
```

### `templates/add-task.html`

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Add Task</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
</head>
<body class="container py-4">
    <h2>Add New Task</h2>
    <form th:action="@{/task/save}" th:object="${task}" method="post">
        <div class="mb-3">
            <label>Title</label>
            <input type="text" class="form-control" th:field="*{title}" required>
        </div>
        <div class="mb-3">
            <label>Description</label>
            <textarea class="form-control" th:field="*{description}"></textarea>
        </div>
        <div class="mb-3">
            <label>Due Date</label>
            <input type="date" class="form-control" th:field="*{dueDate}">
        </div>
        <button type="submit" class="btn btn-success">Save</button>
        <a href="/" class="btn btn-secondary">Cancel</a>
    </form>
</body>
</html>
```

### `templates/edit-task.html`

Same as add, but:

```html
<form th:action="@{'/task/update/' + ${task.id}}" th:object="${task}" method="post">
```

---

## 9️⃣ `application.properties`

```properties
spring.datasource.url=jdbc:h2:mem:tasksdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
spring.thymeleaf.cache=false
```

---

## 🔟 Run the App

```bash
mvn spring-boot:run
```

Visit → [http://localhost:8080](http://localhost:8080)

---

✅ **Result:**
A fully working, dynamic, and responsive **Spring Boot web app** with full CRUD functionality.

---

Would you like me to **extend this app** with:

* ✅ REST API endpoints (for React/Angular frontend)
* ✅ User authentication (Spring Security + Login)
* ✅ MySQL configuration (for production)
  ?
