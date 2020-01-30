[TOC]



# ER图

ER图全称为实体关系图（也成ERD），主要用于数据库设计的结构图，图中有两个重要的属性，一是实体，二是实体之间的关系。（所以ER图被称为实体关系图）



##  ERD符号指南



* 实体

  ERD中每一个对象（矩形框）即为一个实体。在图中实体使用圆角矩形表示，其名称位于正上方，其属性列在实体形状的主体中。以下为一个实体的示例

  ![image-20191121161854750](/Users/mcj/Library/Application Support/typora-user-images/image-20191121161854750.png)

  

* 实体属性

  也称为**列**，表示实体所拥有的属性。

  一个属性，有一个属性名称和属性类型。例如下图：

  ![image-20191121162011987](/Users/mcj/Library/Application Support/typora-user-images/image-20191121162011987.png)

* 主键

  又称PK（primary key），是一种特殊的实体属性，用于**界定数据库表中的记录的独特性**。是在每个实体中表示唯一键的，例如下图：

  ![image-20191121162239983](/Users/mcj/Library/Application Support/typora-user-images/image-20191121162239983.png)

* 外键（Foreign Key）

  称为外来键和外部键，又称FK，是对主键的引用，用于识别实体之间的关系

  

* 关系

  两个实体以某种方式相互关联。在ERD中使用连接线表示

  

* 基数

  基数定义了**一个实与另一个实体的关系里面，某方可能出现次数**，例如，班级中拥有很多学生，班级和学生，在ERD中则表示为班级一对多学生的关系



  * 一对一的关系

    ![image-20191121163826398](/Users/mcj/Library/Application Support/typora-user-images/image-20191121163826398.png)

  * 一对多的关系

    ![image-20191121163858649](/Users/mcj/Library/Application Support/typora-user-images/image-20191121163858649.png)

  * 多对多的关系

    ![image-20191121163927223](/Users/mcj/Library/Application Support/typora-user-images/image-20191121163927223.png)



# 推荐创建表的顺序

1. 绘制ERD

2. 根据ERD导出create table sql，建表

3. 使用ginbro工具生成对应model文件（此步骤为go语言使用，其他语言可使用对应的工具）















