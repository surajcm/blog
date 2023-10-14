---
title: "Refactoring java8 code with collector"
description: "Hello world to the blogging universe."
date: "2022-01-18"
featured_image: "img/2022/01/programming.png"
---
I am planning to refactor a code that is based on Java 7 to Java 8 using the functional components, specifically Collector. The plan is simple: learn it by coding it. Let’s get started…

## Overview of the Java Object We Are Dealing With
![](/blog/img/2022/01/java8-fun.png)

The following code is intend to fetch a few specific data from Company object, where company has a list of employees. I would like to perform the following operations,

    Find out the most common name (Employee name) in the company
    Find the highest salaried employee of the company
    Find the sum of salaries for all Men
    Find most popular grade/band in the company

Prior to java 8, it is tough to refactor these logic and find out some common parts. But the functional components make the refactoring easy.

___
## Pre-Java8 version:

Let’s go through the code,
### Find out the most common employee name in the company
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
### Find the highest salaried employee
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

### Find the sum of salaries for all Men

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

### Find most popular grade/band in the company
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

In all the cases, we need to iterate through various departments and then employees in it. But since the conditions and the data we want to collect are in a different format, we wont be able to find something common and refactor it properly. Let’s see how java8 deals with this.

---

As an initial step, lets use streams and do the processing.

### Find out the most common employee name in the company

Let’s go through the logic of nameAndCount, since the iterations (for loop on department and employee) have a defined result(nameAndCount), we can use streams to get that value

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

why didn’t we get the result via collect? We have some logic there, we need to identify the duplicates, the value of the map is count of employee names. Let’s see the final results.

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

### Find the highest salaried employee

Let’s do the same refactoring here as well. But here, we can use collect as the logic is fairly less complex.

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

### Find the sum of salaries for all Men

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
We need to apply the similar logic as we did for the first method (I mean findMostRepeatingNameInCompany). Let’s try it

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

We have some common values, and might be able to save some lines if we find that common things and refactor it properly, lets go with that…

---
This time, lets use stream.collect() method everywhere. So we have changes only on 2 method calls findMostRepeatingNameInCompany and findMostPopularBandInCompany
### Find out the most common employee name in the company

Let’s try to use collect here, and avoid addToNameAndCountMap method call, If we use the right collector, we can collect data to a map, with a custom key or value. We are going to use `Collectors.groupingBy(Employee::getName,Collectors.counting())` for this.

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

### Find most popular grade/band in the company

Let’s do the same thing here as well,
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
Now we have a common pattern, and it seems there are possibilities to extract some functional components out of it. Let’s explore that possibility.

---
We could see that the methods findMostRepeatingNameInCompany, findEmployeeWithHighestSalaryInTheCompany & findMostPopularBandInCompany follows a similar pattern. Only difference is the type of map, and the collector we use. So, lets write a method that returns a generic map, and accept a collector.

```java
private static <T> Map<T, Long> processEmployeeToMap(Company company,
                                                     Collector<Employee, ?, Map<T,Long>>
                                                     employeeMapCollector) {
    return company.getDepartments().stream()
            .flatMap(department -> department.getEmployees().stream())
            .collect(employeeMapCollector);
}
```

This is a generic method, it works for different inputs, if T is string, it will use the collector to get a Map<String, Long> as return. If we use another object, say Employee, then the return type will be Map<Employee, Long>. Let’s try using this method call in these 3 methods.

### Find out the most common employee name in the company
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
### Find the highest salaried employee

Let’s do the same refactoring here as well,
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

### Find most popular grade/band in the company

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

### Find the sum of salaries for all Men

Well, we need a different way here, as this one is completely different from the rest of the methods,

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
Well, we have used the functional components, and achieved a better reusable code, but still this can be cleaned up a bit.

---
You might have noticed that all these methods need a single value in return, not a list. And, we use optional and some logic to identify that single element. Can we extract that too, let’s try that now.
```java
private static <T> T fetchParamsFromMap(Map<T, Long> param) {
    return param.entrySet().stream()
            .filter(e -> e.getValue().equals(Collections.max(param.values())))
            .map(Map.Entry::getKey).findFirst().orElse(null);
}
```
This method returns the key of biggest param in a map, using generics, this one can be applied to all of our methods

### Find out the most common employee name in the company
```java
private static String findMostRepeatingNameInCompany(Company company) {
    Collector<Employee, ?, Map<String, Long>> repeatingNameCollector =
            Collectors.groupingBy(Employee::getName, Collectors.counting());
    Map<String, Long> nameAndCount = processEmployeeToMap(company, repeatingNameCollector);
    return fetchParamsFromMap(nameAndCount);
}
```
that simple ? yes, it is
### Find the highest salaried employee
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Collector<Employee, ?, Map<Employee, Long>> highSalary =
            Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b);
    Map<Employee, Long> employeeAndSalary = processEmployeeToMap(company, highSalary);
    return fetchParamsFromMap(employeeAndSalary);
}
```
### Find most popular grade/band in the company
```java
private static Band findMostPopularBandInCompany(Company company) {
    Collector<Employee, ?, Map<Band, Long>> popularBandCollector =
            Collectors.groupingBy(Employee::getBand, Collectors.counting());
    Map<Band, Long> bandAndCount = processEmployeeToMap(company, popularBandCollector);
    return fetchParamsFromMap(bandAndCount);
}
```
Done ? No, we are not done yet… there is a bit more to do.

---
In the methods `findMostRepeatingNameInCompany`, `findEmployeeWithHighestSalaryInTheCompany` & `findMostPopularBandInCompany` there is still a repeating pattern, we use `processEmployeeToMap` and immediately we call `fetchParamsFromMap`. Let’s combine it.

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

### Find out the most common employee name in the company

```java
private static String findMostRepeatingNameInCompany(Company company) {
    Collector<Employee, ?, Map<String, Long>> repeatingNameCollector =
            Collectors.groupingBy(Employee::getName, Collectors.counting());
    return fetchBestOfMappedEmployees(company, repeatingNameCollector);
}
```
### Find the highest salaried employee
```java
private static Employee findEmployeeWithHighestSalaryInTheCompany(Company company) {
    Collector<Employee, ?, Map<Employee, Long>> highSalary =
            Collectors.toMap(employee -> employee, Employee::getSalary, (a, b) -> b);
    return fetchBestOfMappedEmployees(company, highSalary);
}
```
### Find most popular grade/band in the company
```java
private static Band findMostPopularBandInCompany(Company company) {
    Collector<Employee, ?, Map<Band, Long>> popularBandCollector =
            Collectors.groupingBy(Employee::getBand, Collectors.counting());
    return fetchBestOfMappedEmployees(company, popularBandCollector);
}
```
We can even continue to simplify this, we can put the collectors outside and the parent calls can pass the collect and then we don’t really need this method calls itself.

All code is available on github, and each separate iteration is in separate commit, please check this out at https://github.com/surajcm/java_fun_extraction_01/commits/master


