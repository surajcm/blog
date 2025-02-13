---
title: "Refactoring Java 8 Code with Collector: A Practical Approach for Seamless Code Flow"
date: "2022-01-18"
description: "Having served as a lead technical architect with extensive experience in the Java ecosystem, I've had the privilege of witnessing the transformative potential of effective code refactoring. Today, I extend a warm invitation to embark on a journey delving into the art of refactoring Java 8 code. We'll explore a pragmatic and systematic approach that assures you a smoother code flow, ultimately enhancing your development skills."
featured: true
toc: true 
featureImage: "img/2022/01/programming.png"
featureImageAlt: 'Lets refactor'
thumbnail: "img/2022/01/programming.png"
shareImage: "img/2022/01/programming.png"
codeMaxLines: 25
# codeLineNumbers: false # Override global value for showing of line numbers within code block.
categories:
  - Technology
tags:
  - java
  - refactor
---

In our quest for knowledge, let's begin by structuring our narrative with a clear and organized framework. Let's keep our vessel steady as we navigate through the intricate waters of Java 8 code refactoring.

As we progress, we will break down complex ideas into more digestible terms and provide vivid examples to illuminate the path. 

## Understanding the Java Object in Focus
![](/blog/img/2022/01/java8-fun.png)

The code below aims to retrieve specific data from a Company object, which includes a list of employees. I aim to execute the following operations:

* Identify the most frequently occurring name among employees.
* Determine the employee with the highest salary in the company.
* Calculate the total salary of all male employees.
* Ascertain the most prevalent grade or band within the company.

Before the advent of Java 8, refactoring these logics and identifying commonalities was challenging. However, leveraging functional components simplifies the refactoring process significantly.
___

## Pre-Java8 version:

Let's delve into the code to accomplish specific tasks:

### Finding the Most Common Employee Name in the Company
```java
private static String findMostRepeatingNameInCompany(Company company) {
    String repeatingName = null;
    Map<String, Integer> nameAndCount = new HashMap<>();
    for (Department department : company.getDepartments()) {
        for (Employee employee : department.getEmployees()) {
            if (!nameAndCount.containsKey(employee.getName())) {
                nameAndCount.put(employee.getName(), 1);
            } else {
                Integer count = nameAndCount.get(employee.getName());
                count++;
                nameAndCount.put(employee.getName(), count);
            }
        }
    }
    for (Map.Entry<String, Integer> entry : nameAndCount.entrySet()) {
        if (entry.getValue().equals(Collections.max(nameAndCount.values()))) {
            repeatingName = entry.getKey();
        }
    }
    return repeatingName;
}
```
### Locating the Employee with the Highest Salary
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Employee costlyEmployee = null;
    Map<Employee, Long> employeeAndSalary = new HashMap<>();
    for (Department department : company.getDepartments()) {
        for (Employee employee : department.getEmployees()) {
            employeeAndSalary.put(employee, employee.getSalary());
        }
    }
    for (Map.Entry<Employee, Long> entry : employeeAndSalary.entrySet()) {
        if (entry.getValue().equals(Collections.max(employeeAndSalary.values()))) {
            costlyEmployee = entry.getKey();
        }
    }
    return costlyEmployee;
}
```

### Calculating the Sum of Salaries for Male Employees

```java
private static Long findSumOfAllMenSalary(Company company) {
    Long totalSalary = 0L;
    for (Department department : company.getDepartments()) {
        for (Employee employee : department.getEmployees()) {
            if (employee.getGender().equals(Gender.MALE)) {
                totalSalary = totalSalary + employee.getSalary();
            }
        }
    }
    return totalSalary;
}
```

### Identifying the Most Popular Grade/Band in the Company
```java
private static Band findMostPopularBandInCompany(Company company) {
    Band popularBand = null;
    Map<Band, Integer> bandAndCount = new HashMap<>();
    for (Department department : company.getDepartments()) {
        for (Employee employee : department.getEmployees()) {
            if (!bandAndCount.containsKey(employee.getBand())) {
                bandAndCount.put(employee.getBand(), 1);
            } else {
                Integer count = bandAndCount.get(employee.getBand());
                count++;
                bandAndCount.put(employee.getBand(), count);
            }
        }
    }
    for (Map.Entry<Band, Integer> entry : bandAndCount.entrySet()) {
        if (entry.getValue().equals(Collections.max(bandAndCount.values()))) {
            popularBand = entry.getKey();
        }
    }
    return popularBand;
}
```

In each scenario, iterating through different departments and their respective employees is necessary. However, due to varying data formats and conditions, finding commonalities for effective refactoring becomes challenging. 

---

## Let's explore how Java 8 addresses these complexities

To commence, let's employ streams for data processing.

### Finding the Most Common Employee Name in the Company

Let's delve into the logic behind 'nameAndCount'. Considering that the iterations (for loops on departments and employees) yield a specific result represented by 'nameAndCount', we can leverage streams to achieve the same value.

```java
company.getDepartments().stream().flatMap(department -> department.getEmployees().stream())
 .forEach(employee -> addToNameAndCountMap(nameAndCount, employee));
 
...
...
  
private static void addToNameAndCountMap(Map<String, Integer> nameAndCount, Employee employee) {
    if (!nameAndCount.containsKey(employee.getName())) {
        nameAndCount.put(employee.getName(), 1);
    } else {
        Integer count = nameAndCount.get(employee.getName());
        count++;
        nameAndCount.put(employee.getName(), count);
    }
}
```

Why didn't we obtain the result using 'collect'? Our logic involves identifying duplicates, where the map's value represents the count of employee names. Now, let's proceed to examine the final results.

```java
private static String findMostRepeatingNameInCompany(Company company) {
    Map<String, Integer> nameAndCount = new HashMap<>();
    company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .forEach(employee -> addToNameAndCountMap(nameAndCount, employee));
    String repeatingName = null;
    Integer maxCount = Collections.max(nameAndCount.values());
    Optional<String> repeat = nameAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxCount))
            .map(Map.Entry::getKey).findFirst();
    if (repeat.isPresent()) {
        repeatingName = repeat.get();
    }
    return repeatingName;
}
 
private static void addToNameAndCountMap(Map<String, Integer> nameAndCount, Employee employee) {
    if (!nameAndCount.containsKey(employee.getName())) {
        nameAndCount.put(employee.getName(), 1);
    } else {
        Integer count = nameAndCount.get(employee.getName());
        count++;
        nameAndCount.put(employee.getName(), count);
    }
}
```

### Locating the Employee with the Highest Salary

Let's apply the same refactoring approach in this case. However, here, we can utilize 'collect' since the logic is relatively less complex.
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Employee costlyEmployee = null;
    Map<Employee, Long> employeeAndSalary = company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b));
    Long maxSalary = Collections.max(employeeAndSalary.values());
    Optional<Employee> costly = employeeAndSalary.entrySet().stream()
            .filter(e -> e.getValue().equals(maxSalary)).map(Map.Entry::getKey).findFirst();
    if(costly.isPresent()) {
        costlyEmployee = costly.get();
    }
    return costlyEmployee;
}
```

### Calculating the Sum of Salaries for Male Employees

Well, this one seems much simpler.

```java
private static Long findSumOfAllMenSalary(Company company) {
    return company.getDepartments().stream()
            .flatMap(d -> d.getEmployees().stream())
            .filter(e -> e.getGender().equals(Gender.MALE))
            .map(Employee::getSalary).mapToLong(Long::longValue).sum();
}
```

### Find most popular grade/band in the company
We should apply a similar logic to what was implemented in the first method (findMostRepeatingNameInCompany). Let's attempt it here.

```java
private static Band findMostPopularBandInCompany(Company company) {
    Map<Band, Integer> bandAndCount = new HashMap<>();
    company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .forEach(employee -> addToBandAndCoutMap(bandAndCount, employee));
    Band popularBand = null;
    Integer maxBand = Collections.max(bandAndCount.values());
    Optional<Band> popular = bandAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxBand))
            .map(Map.Entry::getKey).findFirst();
    if(popular.isPresent()) {
        popularBand = popular.get();
    }
    return popularBand;
}
 
private static void addToBandAndCoutMap(Map<Band, Integer> bandAndCount, Employee employee) {
    if (!bandAndCount.containsKey(employee.getBand())) {
        bandAndCount.put(employee.getBand(), 1);
    } else {
        Integer count = bandAndCount.get(employee.getBand());
        count++;
        bandAndCount.put(employee.getBand(), count);
    }
}
```

We have identified some common elements, and there might be an opportunity to condense the code by refactoring these shared aspects. Let's proceed in that direction.

---
## Java 8 - iteration 2

This time, let's utilize the stream.collect() method across the board. Consequently, we'll focus on making changes only in two method calls: findMostRepeatingNameInCompany and findMostPopularBandInCompany.

### Finding the Most Common Employee Name in the Company

Let's attempt to employ the collect method here and circumvent the addToNameAndCountMap method call. By leveraging the appropriate collector, we can accumulate data into a map, defining either a custom key or value. We'll utilize 'Collectors.groupingBy(Employee::getName, Collectors.counting())' for this purpose.


```java
private static String findMostRepeatingNameInCompany(Company company) {
    Map<String, Long> nameAndCount = company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(Collectors.groupingBy(Employee::getName,Collectors.counting()));
    Long maxCount = Collections.max(nameAndCount.values());
    Optional<String> repeat = nameAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxCount))
            .map(Map.Entry::getKey).findFirst();
    String repeatingName = null;
    if (repeat.isPresent()) {
        repeatingName = repeat.get();
    }
    return repeatingName;
}
```

### Identifying the Most Popular Grade/Band in the Company

Let's apply the same approach in this case as well.

```java
private static Band findMostPopularBandInCompany(Company company) {
    Map<Band, Long> bandAndCount = company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(Collectors.groupingBy(Employee::getBand, Collectors.counting()));
    Band popularBand = null;
    Long maxBand = Collections.max(bandAndCount.values());
    Optional<Band> popular = bandAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxBand))
            .map(Map.Entry::getKey).findFirst();
    if(popular.isPresent()) {
        popularBand = popular.get();
    }
    return popularBand;
}
```
We've identified a recurring pattern, suggesting potential opportunities to extract functional components. Let's explore the feasibility of this approach.

---
## Java 8 - iteration 3

It seems like the methods findMostRepeatingNameInCompany, findEmployeeWithHighestSalaryInTheCompany, and findMostPopularBandInCompany share a similar pattern. The main difference lies in the type of map used and the collector employed. Let's create a method that can generate a generic map and accepts a collector as an argument.

```java
private static <T> Map<T, Long> processEmployeeToMap(Company company,
                                                     Collector<Employee, ?, Map<T,Long>>
                                                     employeeMapCollector) {
    return company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(employeeMapCollector);
}
```

This method is generic and adaptable to various inputs. If T is a string, it will employ the collector to produce a Map<String, Long>. If we use another object, such as Employee, the return type will be Map<Employee, Long>. Let's implement this method call in the three aforementioned methods.

### Finding the Most Common Employee Name in the Company
```java
private static String findMostRepeatingNameInCompany(Company company) {
    Collector<Employee, ?, Map<String, Long>> repeatingNameCollector =
            Collectors.groupingBy(Employee::getName,Collectors.counting());
    Map<String, Long> nameAndCount = processEmployeeToMap(company,repeatingNameCollector);
    Long maxCount = Collections.max(nameAndCount.values());
    Optional<String> repeat = nameAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxCount))
            .map(Map.Entry::getKey).findFirst();
    String repeatingName = null;
    if (repeat.isPresent()) {
        repeatingName = repeat.get();
    }
    return repeatingName;
}
```
### Locating the Employee with the Highest Salary

Let's apply the same refactoring approach here as well.
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Collector<Employee,?,Map<Employee,Long>> highSalary =
            Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b);
    Map<Employee, Long> employeeAndSalary = processEmployeeToMap(company,highSalary);
    Long maxSalary = Collections.max(employeeAndSalary.values());
    Optional<Employee> costly = employeeAndSalary.entrySet().stream()
            .filter(e -> e.getValue().equals(maxSalary)).map(Map.Entry::getKey).findFirst();
    Employee costlyEmployee = null;
    if(costly.isPresent()) {
        costlyEmployee = costly.get();
    }
    return costlyEmployee;
}
```

### Identifying the Most Popular Grade/Band in the Company

```java
private static Band findMostPopularBandInCompany(Company company) {
    Collector<Employee,?,Map<Band,Long>> popularBandCollector =
            Collectors.groupingBy(Employee::getBand, Collectors.counting());
    Map<Band, Long> bandAndCount = processEmployeeToMap(company,popularBandCollector);
    Band popularBand = null;
    Long maxBand = Collections.max(bandAndCount.values());
    Optional<Band> popular = bandAndCount.entrySet().stream()
            .filter(e -> e.getValue().equals(maxBand))
            .map(Map.Entry::getKey).findFirst();
    if(popular.isPresent()) {
        popularBand = popular.get();
    }
    return popularBand;
}
```

### Calculating the Sum of Salaries for Male Employees

Since this method differs significantly from the previous ones, let's consider a different approach tailored to its distinct nature.

```java
public static Function<Department, Stream<Employee>> 
          allEmployeeInDept = department -> department.getEmployees().stream();
 
private static Long findSumOfAllMenSalary(Company company) {
    return company.getDepartments().stream()
            .flatMap(d -> allEmployeeInDept.apply(d))
            .filter(e -> e.getGender().equals(Gender.MALE))
            .map(Employee::getSalary).mapToLong(Long::longValue).sum();
}
```
While we've successfully utilized functional components for better code reusability, there's room for further refinement and cleanup to enhance its readability and efficiency.

---
## Java 8 - iteration 4

Indeed, all these methods return a single value, not a list. We utilize Optional and certain logic to determine that singular element. Let's extract this commonality and streamline it to improve the code. Let's proceed with that approach now.
```java
private static <T> T fetchParamsFromMap(Map<T, Long> param) {
    return param.entrySet().stream()
            .filter(e -> e.getValue().equals(Collections.max(param.values())))
            .map(Map.Entry::getKey).findFirst().orElse(null);
}
```
This method, using generics, retrieves the key of the largest parameter in a map. It's a versatile function that can be applied across all of our methods.

### Finding the Most Common Employee Name in the Company
```java
private static String findMostRepeatingNameInCompany(Company company) {
    Collector<Employee, ?, Map<String, Long>> repeatingNameCollector =
            Collectors.groupingBy(Employee::getName, Collectors.counting());
    Map<String, Long> nameAndCount = processEmployeeToMap(company, repeatingNameCollector);
    return fetchParamsFromMap(nameAndCount);
}
```
Sometimes, the simplest solutions are the most effective ones.
### Locating the Employee with the Highest Salary
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Collector<Employee, ?, Map<Employee, Long>> highSalary =
            Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b);
    Map<Employee, Long> employeeAndSalary = processEmployeeToMap(company, highSalary);
    return fetchParamsFromMap(employeeAndSalary);
}
```
### Identifying the Most Popular Grade/Band in the Company
```java
private static Band findMostPopularBandInCompany(Company company) {
    Collector<Employee, ?, Map<Band, Long>> popularBandCollector =
            Collectors.groupingBy(Employee::getBand, Collectors.counting());
    Map<Band, Long> bandAndCount = processEmployeeToMap(company, popularBandCollector);
    return fetchParamsFromMap(bandAndCount);
}
```
It seems there's more work left to do. Let's continue refining and enhancing the code to ensure it meets all the required criteria

---
## Java 8 - iteration 5

In the methods `findMostRepeatingNameInCompany` , `findEmployeeWithHighestSalaryInTheCompany`, and `findMostPopularBandInCompany`, there's a recurring pattern where we first utilize `processEmployeeToMap` and immediately follow it with `fetchParamsFromMap`. Let's consolidate these steps into a unified process for improved efficiency.

```java
private static <T> T fetchBestOfMappedEmployees(Company company,
                                                Collector<Employee, ?, Map<T, Long>> employeeMapCollector) {
    Map<T, Long> mappedResults = company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(employeeMapCollector);
    return mappedResults.entrySet().stream()
            .filter(e -> e.getValue().equals(Collections.max(mappedResults.values())))
            .map(Map.Entry::getKey).findFirst().orElse(null);
}
```

Now, let’s see how our methods looks like,

### Finding the Most Common Employee Name in the Company

```java
private static String findMostRepeatingNameInCompany(Company company) {
    Collector<Employee, ?, Map<String, Long>> repeatingNameCollector =
            Collectors.groupingBy(Employee::getName, Collectors.counting());
    return fetchBestOfMappedEmployees(company, repeatingNameCollector);
}
```
### Locating the Employee with the Highest Salary
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Collector<Employee, ?, Map<Employee, Long>> highSalary =
            Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b);
    return fetchBestOfMappedEmployees(company, highSalary);
}
```
### Identifying the Most Popular Grade/Band in the Company
```java
private static Band findMostPopularBandInCompany(Company company) {
    Collector<Employee, ?, Map<Band, Long>> popularBandCollector =
            Collectors.groupingBy(Employee::getBand, Collectors.counting());
    return fetchBestOfMappedEmployees(company, popularBandCollector);
}
```

In our quest for simplification, we've discovered a method to externalize collectors and allow parent calls to pass collect, eliminating the need for method calls themselves. You can explore the step-by-step iterations of this process in separate commits on GitHub. Check out the code at https://github.com/surajcm/java_fun_extraction_01/commits/master for a detailed breakdown.