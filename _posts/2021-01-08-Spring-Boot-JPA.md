---
title: Spring Boot JPA 操作
author: 郑闯闯
date: 2020-01-08 14:31:00 +0800
categories: [Java, spring-jpa]
tags: [Java]
toc: false
---

## 初始配置

### 添加依赖

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```

### 项目配置

```yml
# 数据源配置
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/stock?characterEncoding=utf-8
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
```

### 创建entity和repository

```java
package zhengchuang.stock.model.entity;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.io.Serializable;

@Entity
public class Record implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    private String code;

    @Column(columnDefinition = "Decimal(10, 2)")
    private Double price;

    public void setCode(String code) {
        this.code = code;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setPrice(Double price) {
        this.price = price;
    }

    public Double getPrice() {
        return price;
    }

    public String getCode() {
        return code;
    }

    public String getName() {
        return name;
    }

    public Long getId() {
        return id;
    }
}

```

```java
package zhengchuang.stock.model.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import zhengchuang.stock.model.entity.Record;

public interface RecordRepository extends JpaRepository<Record, Long> {
}
```

### 使用

```java
package zhengchuang.stock.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;
import zhengchuang.stock.dto.RecordDTO;
import zhengchuang.stock.model.entity.Record;
import zhengchuang.stock.model.repository.RecordRepository;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
public class RecordController {

    @Autowired
    private RecordRepository recordRepository;

    @PostMapping("/record")
    public Record create(@RequestBody @Validated RecordDTO recordDTO) {
        Record record = new Record();
        record.setCode(recordDTO.getCode());
        record.setName(recordDTO.getName());
        record.setPrice(recordDTO.getPrice());
        return recordRepository.save(record);

    }

    @GetMapping("/record")
    public List<Record> findAll() {
        return recordRepository.findAll();
    }

    @DeleteMapping("/record/{id}")
    public Map<String, Object> delete(@PathVariable Long id) {
        Record record = recordRepository.findById(id).orElse(new Record());
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("message", "数据不存在");

        if (record.getId() == null) {
            return map;
        }
        recordRepository.deleteById(id);
        return map;
    }
}
```