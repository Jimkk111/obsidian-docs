---
title: package
source: https://www.yuque.com/mook-mvqcp/viz8gx/my9f7cuugxfez9bq
created: 2026-08-16
updated: 2026-08-16
tags:
  - Java SE
---

包是**相关类型（类、接口、枚举和注解类型）的分组**，旨在**提供访问保护和命名空间管理**。

#### 声明与使用

1.包的层次结构与文件系统的目录结构相对应。例如，包`oliver.twist`是包`oliver`的子包。但需要注意，**父子包之间没有特殊的访问权限关系**。

2.在源文件的第一行使用`package`语句声明包。

3.要使用其他包中的`public`类型，可以通过**使用全限定名**、**导入（**`import`**）特定类型**或**导入整个包**三种方式。

#### classpath

classpath表示存放类的位置，在这个路径下面来声明包，是包的根目录。

classpath可以设置多个。

classpath的作用是让jvm能找到类定义在哪里。

#### 全限定名

包名+类名
