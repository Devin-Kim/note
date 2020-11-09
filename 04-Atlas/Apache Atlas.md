# Apache Atlas

## 一、Apache Atlas简介

### 1.1 Atlas产生背景

&ensp;&ensp;&ensp;&ensp;谈到数据治理，自然离不开元数据。元数据(Metadata)，用一句话定义就是：描述数据的数据。元数据打通了数据源、数据仓库、数据应用，记录了数据从产生到消费的全过程。因此，数据治理的核心就是`元数据管理`

&ensp;&ensp;&ensp;&ensp;面对海量且持续增加的各式各样的数据对象，你是否有信心知道哪些数据从哪里来以及它如何随时间而变化？采用Hadoop必须考虑数据管理的实际情况，元数据与数据治理成为企业级数据湖的重要部分。

&ensp;&ensp;&ensp;&ensp;为寻求数据治理的开源解决方案，Hortonworks 公司联合其他厂商与用户于2015年发起数据治理倡议，包括数据分类、集中策略引擎、数据血缘、安全和生命周期管理等方面。Apache Atlas 项目就是这个倡议的结果，社区伙伴持续的为该项目提供新的功能和特性。==该项目用于管理共享元数据、数据分级、审计、安全性以及数据保护等方面，努力与Apache Ranger整合，用于数据权限控制策略==。



**Apache Atlas为Hadoop的元数据治理提供了以下特性：**

-  数据分类
  1. 为元数据导入或定义业务导向的分类注释
  2. 定义，注释，以及自动捕获数据集和底层元素之间的关系
  3. 导出元数据到第三方系统

- 集中审计
  1. 捕获与所有应用，过程以及与数据交互的安全访问信息
  2. 捕获执行，步骤，活动等操作的信息

- 搜索与血缘
  1. 预定义的导航路径用来探索数据分类以及审计信息
  2. 基于文本的搜索特性来快速和准确的定位相关联的数据和审计事件
  3. 对数据集血缘关系的可视化浏览使用户可以下钻到操作，安全以及数据起源相关的信息

- 安全与策略引擎
  1. 基于数据分类模式，属性以及角色的运行时合理合规策略
  2. 基于分类-预测的高级策略定义以防止数据推导
  3.  基于cell的属性和值的行/列级别的masking

---



### 1.2 Atlas架构原理

&ensp;&ensp;&ensp;&ensp;Atlas 是一个可扩展和可扩展的核心基础治理服务集 - 使企业能够有效地和高效地满足 Hadoop 中的合规性要求，并允许与整个企业数据生态系统的集成。

![b96e93b2ea7548fbd8bcb15eab93988c](img/b96e93b2ea7548fbd8bcb15eab93988c.jpg)



Atlas 的组件可以分为以下主要类别：

#### 1. Core

##### Type System

&ensp;&ensp;&ensp;&ensp;==Atlas 允许用户为他们想要管理的元数据对象定义一个模型。该模型由称为"类型"的定义组成。"类型" 的 实例被称为 "实体" 表示被管理的实际元数据对象。==类型系统是一个组件，允许用户定义和管理类型和实体。由 Atlas 管理的所有元数据对象（例如Hive表）都使用类型进行建模，并表示为实体。要在 Atlas 中存储新类型的元数据，需要了解类型系统组件的概念。

&ensp;&ensp;&ensp;&ensp;需要注意的一个关键点是，Atlas 中建模的通用性质允许数据管理员和集成者定义技术元数据和业务元数据。也可以使用 Atlas 的==特征==来定义两者之间的丰富关系。



##### Ingest / Export

&ensp;&ensp;&ensp;&ensp;Ingest 【译：摄取】组件允许将元数据添加到 Atlas。

&ensp;&ensp;&ensp;&ensp;Export 组件暴露由 Atlas 检测到的元数据更改，以作为事件引发，消费者可以使用这些更改事件来实时响应元数据更改。



##### Graph Engine

&ensp;&ensp;&ensp;&ensp;在内部，Atlas 通过使用图形模型管理元数据对象。以实现元数据对象之间的巨大灵活性和丰富的关系。==图形引擎是负责在类型系统的类型和实体之间进行转换的组件，以及基础图形模型。除了管理图形对象之外，图形引擎还为元数据对象创建适当的索引，以便有效地搜索它们。==



##### JanusGraph

&ensp;&ensp;&ensp;&ensp;目前，Atlas 使用 JanusGraph 图数据库来存储元数据对象。[JanusGraph](https://blog.csdn.net/ZYC88888/article/details/86605696)

&ensp;&ensp;&ensp;&ensp;JanusGraph 使用两个存储：**默认情况下元数据存储配置为 HBase ，索引存储配置为 Solr**。元数据存储用于存储元数据对象本身，并且索引存储用于存储元数据属性的索引，其允许高效搜索。



---



#### 2. Integration

【Integration  译： 集成】

用户可以使用两种方法管理 Atlas 中的元数据：

##### API

&ensp;&ensp;&ensp;&ensp;Atlas 的所有功能可通过 REST API 提供给最终用户，==允许创建，更新和删除类型和实体==。==它也是查询Atlas 管理的类型和实体的主要方法==。

##### Messaging

&ensp;&ensp;&ensp;&ensp;除了 API 之外，用户还可以选择使用基于 Kafka 的消息接口与 Atlas 集成。来自各个组件的Hook 的元数据通知事件通过写入到名为 ATLAS_HOOK 的 Kafka Topic，Atlas只需订阅这个主题即可获取到元数据对象。Atlas中的元数据更改事件会写入到名为 ATLAS_ENTITIES 的 Kafka Topic，通过订阅这个主题，可以更便利的与其他组件集成（比如：Ranger）。



---



#### 3. 元数据源

Atlas 支持与许多元数据源的集成。将来还会添加更多集成。目前，Atlas 支持从以下来源获取和管理元数据：

- [HBase](http://atlas.apache.org/#/HookHBase)
- [Hive](http://atlas.apache.org/#/HookHive)
- [Sqoop](http://atlas.apache.org/#/HookSqoop)
- [Storm](http://atlas.apache.org/#/HookStorm)
- [Kafka](http://atlas.apache.org/#/HookKafka)

如果要集成其它元数据源意味着要做两件事：

* 要在 Atlas Type System 中定义相关的元数据模型。 
* 通过 Atlas 提供摄取元数据对象的组件Ingest，批量或实时获取元数据。



---



#### 4. Apps

由 Atlas 管理的元数据各种应用程序使用，满足许多治理用例。

##### Atlas Admin UI

&ensp;&ensp;&ensp;&ensp;该组件是一个基于 Web 的应用程序，==允许数据管理员和科学家发现和注释元数据==。这里最重要的是==搜索界面和 SQL 样的查询语言，可以用来查询由 Atlas 管理的元数据类型和对象==。管理 UI 使用 Atlas 的 REST API 来构建其功能。

> 本集群Admin UI：http://flink02.baicdt.com:21000/login.jsp ， 用户名：admin，密码： Wsygzgr_2020



##### Ranger Tag Based Policies

&ensp;&ensp;&ensp;&ensp;Apache Ranger 是针对 Hadoop 生态系统的高级安全管理解决方案，与各种 Hadoop 组件具有广泛的集成。

&ensp;&ensp;&ensp;&ensp;通过与 Atlas 集成，Ranger 允许安全管理员定义元数据驱动的安全策略，以实现有效的治理。 Ranger 是Atlas元数据更改事件的消费者。



##### Business Taxonomy

&ensp;&ensp;&ensp;&ensp;从元数据源获取到 Atlas 的元数据对象主要是一种技术形式的元数据。为了增强可发现性和治理能力，Atlas 提供了一个业务分类界面，允许用户首先定义一组代表其业务域的业务术语，并将其与 Atlas 管理的元数据实体相关联。==业务分类法是一种 Web 应用程序，目前是 Atlas Admin UI 的一部分，并且使用 REST API 与 Atlas 集成。==



---



### 1.3 Atlas Type System

&ensp;&ensp;&ensp;&ensp;类型系统Type System是Atlas最核心的组件之一，用户可以通过类型系统对数据资产进行分类、定义，然后用Ingest/Export组件添加元数据或输出元数据变化。对外，其他系统可以通过REST API或Kafka Message与Atlas进行集成。

&ensp;&ensp;&ensp;&ensp;==Atlas 允许用户为他们想要管理的元数据对象定义一个模型。该模型由称为 "类型" 的定义组成。被称为 "实体" 的 "类型" 实例表示被管理的实际元数据对象。类型系统是一个组件，允许用户定义和管理类型和实体。由 Atlas 管理的所有元数据对象（例如Hive表）都使用类型进行建模，并表示为实体。要在Atlas中存储新类型的元数据，需要了解类型系统组件的概念。==

#### 1. Types

&ensp;&ensp;&ensp;&ensp;Atlas中的 "类型" 定义了如何存储和访问特定类型的元数据对象。类型表示了所定义元数据对象的一个或多个属性集合，如`hdfs_path` Type,`hive_db` Type,`hive_table` Type,`hive_column` Type。。具有开发背景的用户可以将 "类型" 理解成面向对象的编程语言的 "类" 定义的或关系数据库的 "表模式"。

&ensp;&ensp;&ensp;&ensp;例如：hive_table就是一个由Atlas已经定义好了的类，这个类具有的属性如下

```properties
Name:         hive_table
TypeCategory: Entity
SuperTypes:   DataSet
Attributes:
    name:             string
    db:               hive_db
    owner:            string
    createTime:       date
    lastAccessTime:   date
    comment:          string
    retention:        int
    sd:               hive_storagedesc
    partitionKeys:    array<hive_column>
    aliases:          array<string>
    columns:          array<hive_column>
    parameters:       map<string>
    viewOriginalText: string
    viewExpandedText: string
    tableType:        string
    temporary:        boolean
```

从上面的例子可以注意到以下五点：

1. name属性是Type的唯一标识



2. ==类型具有元类型==， 本示例中TypeCategory为Entity，Atlas 有以下几种元类型：
   * 基本元类型： Iboolean, byte, short, int, long, float, double, biginteger, bigdecimal, string, date
   * 枚举元类型
   * 集合元类型：例如Array，Map
   * 复合元类型：Entity, Struct【译：结构体】, Classification【译：分类;归类;分级;类别;等级;门类;】, Relationship【译：关系;联系;】



3. Entity，Classification这两种元类型可以从称为 "supertype" 的父类型 "extend" ，因此，它将包含在 "supertype" 中定义的属性。这允许模型在一组相关类型等之间定义公共属性。类似于面向对象语言中的超类的概念。 Atlas 中的类型也可以从多个超类型扩展，比如：在该示例中，每个 hive 表从预定义的超类型（称为 "DataSet"）扩展。稍后将提供关于此预定义类型的更多细节。



4. Entity，Struct，Classification，Relationship这四种元类型可以具有集合属性。每个属性都有一个名称（例如 "name"）和一些其他关联的属性。可以使用表达式 type_name.attribute_name 来引用属性。【注意，属性本身是使用 Atlas 元类型定义的】。在这个例子中，hive_table.name 是一个字符串，hive_table.aliases 是一个字符串数组，hive_table.db 引用一个类型的实例称为 hive_db 等等。



5. 在属性中引用（如hive_table.db）。使用这样的属性，我们可以在 Atlas 中定义的两种类型之间的任意关系，从而构建丰富的模型。注意，也可以收集一个引用列表作为属性类型（例如 hive_table.cols，它表示从 hive_table 到 hive_column 类型的引用列表）



---



#### 2. Entities

&ensp;&ensp;&ensp;&ensp;Atlas中的Entities是Type的特定值或实例，表示真实世界中的特定元数据对象。 与面向对象编程语言的类比，"实例" 是某个 "类" 的 "对象"。

&ensp;&ensp;&ensp;&ensp;实体的示例将是特定的 Hive 表。 比如："Hive" 在 "默认" 数据库中有一个名为 "customers" 的表。 此表将是Atlas中 Type为hive_table  的一个 "实体"。它将具有作为 Hive 表 "类型" 的一部分的每个属性的值，例如：

```properties
guid:     "9ba387dd-fa76-429c-b791-ffc338d3c91f"
typeName: "hive_table"
status:   "ACTIVE"
values:
    name:             “customers”
    db:               { "guid": "b42c6cfc-c1e7-42fd-a9e6-890e0adf33bc","typeName": "hive_db" }
    owner:            “admin”
    createTime:       1490761686029
    updateTime:       1516298102877
    comment:          null
    retention:        0
    sd:               { "guid": "ff58025f-6854-4195-9f75-3a3058dd8dcf","typeName":"hive_storagedesc" }
    partitionKeys:    null
    aliases:          null
    columns:          [ { "guid": "65e2204f-6a23-4130-934a-9679af6a211f","typeName": "hive_column" },{ "guid": "d726de70-faca-46fb-9c99-cf04f6b579a6","typeName": "hive_column" }, ... ]
    parameters:       { "transient_lastDdlTime": "1466403208" }
    viewOriginalText: null
    viewExpandedText: null
    tableType:        “MANAGED_TABLE”
    temporary:        false
```

从上面的例子可以注意到以下几点：

- ==每个实体都由唯一标识符 GUID 标识。此 GUID 由 Atlas 服务器在定义对象时生成，并在实体的整个生命周期内保持不变。在任何时间点，可以使用其 GUID 来访问该特定实体。==在本示例中， "customers" 表具有GUID为 "9ba387dd-fa76-429c-b791-ffc338d3c91f" 的唯一标识。



- 类型的名称（typeName）与实体定义一起提供。在这个例子中，"customers" 表的typeName是 "hive_table"。



- 此实体的所有属性名称及对应值的类型，与hive_table 类型定义中定义的属性的值一致。官网原文：此实体的值是配置单元表类型定义中定义的所有属性名称及其值的映射。



- 属性值将根据属性的元类型而定。==Entity元类型将具有AtlasObjectId类型的值。==

  在本示例中，表和数据库之间的关系通过 "db" 属性捕获，"db" 属性的值将是一个hive_db 实体且唯一标识 的 GUID【GUID可以表示一个hive_table的实例。hive_table的元类型是Entity】。通过"db"这个属性，我们现在可以看到Entity和Struct 两种元类型之间的区别。==Entity的实例有一个标识（具有GUID值），并且可以从其他实体引用（例如配置单元db实体是从配置单元表实体引用的）。==而Struck元类型的实例没有自己的标识。Struck元类型的值是“嵌入”在实体本身内部的属性集合。



---



#### 3. Attributes

&ensp;&ensp;&ensp;&ensp;我们已经了解了属性在元类型中定义，如：Entity, Struct, Classification, Relationship，所以我们简单地将属性称为具有名称和元类型值。 然而， Atlas 中还有一些属性与类型系统有更紧密的联系。例如：

```properties
  name: string,
  dataTypeName: string,
  isComposite: boolean,
  isIndexable: boolean,
  isUnique: boolean,
  multiplicity: enum,
  reverseAttributeName: string
```

上述属性具有以下含义：

- name - attribute的名称



- dataTypeName - attribute的元类型名称（本机，集合或复合）



- isComposite - attribute是否复合
  - 如果一个attribute被定义为复合，意味着它的生命周期不能独立于它所包含的实体。例如：构成 hive 表一部分的一组列。由于列在 hive 表之外没有意义，它们被定义为组合属性。
  - 必须在 Atlas 中创建复合属性及其所包含的实体。即，hive中的单元列必须与单元表一起创建。



- isIndexable - attribute是否索引
  - 是否应该索引，以便可以使用属性值作为谓词来执行查找，并且可以有效地执行查找。



- isUnique - attribute是否唯一

  - 如果指定为唯一，这意味着为JanusGraph中的此属性创建一个特殊索引，允许基于等式的查找。
  - 具有此标志的真实值的任何属性都被视为主键，用于将此实体与其他实体区分开。例如， hive_table 的 name 属性，具有相同名称的表可以存在于多个数据库中。如果 Atlas 在多个集群中存储 hive 表的元数据，即使一对（数据库名称，表名称）也不是唯一的。只有集群位置，数据库名称和表名称可以在物理世界中被视为唯一。

  

* multiplicity【译：多重性】 - 指示这个attribute是required【必需的】, optional【可选的】, multi-valued【多值的】

  如果实体的attribute的定义与类型定义中的多重性声明不匹配，则这将是一个约束违反，并且实体添加将失败。因此，该字段可以用于定义元数据信息上的一些约束。



&ensp;&ensp;&ensp;&ensp;根据上述内容，让我们来看一下的hive_table的一些attribute的定义。例如："db" 的属性，它表示 hive 表所属的数据库：

```properties
  db:
      "dataTypeName": "hive_db",
      "isComposite": false,
      "isIndexable": true,
      "isUnique": false,
      "multiplicity": "required",
      "name": "db",
      "reverseAttributeName": null
```

&ensp;&ensp;&ensp;&ensp;注意多重性的 "multiplicity" = "required" 约束。 如果没有 db 引用，则不能创建表实体。

```properties
  columns:
      "dataTypeName": "array<hive_column>",
      "isComposite": true,
      "isIndexable": true,
      “isUnique": false,
      "multiplicity": "optional",
      "name": "columns",
      "reverseAttributeName": null
```

&ensp;&ensp;&ensp;&ensp;请注意列的 "isComposite" = true 值。通过这样做，我们指示定义的列实体应该始终绑定到它们定义的表实体。

&ensp;&ensp;&ensp;&ensp;从这个描述和示例中，您将能够意识到attribute的定义可以用于影响 Atlas 系统要执行的特定建模行为（约束，索引等）。

#### 4. 系统特定类型及其意义

Atlas 提供了一些预定义的系统类型。例如1.3-1章节中`SuperTypes:   DataSet`。

* **Referenceable**【译：可参考】

  此类型表示可使用名为 qualifiedName 的唯一attribute搜索的所有实体。

  

* **Asset**【译：资产】

  Asset扩展了Referenceable类型并且加入一些属性，如name，description，owner。name属性是必需的（即，multiplicity = required），其他是可选的。Referenceable 和 Asset的目的是为建模者在定义和查询类型的实体时提供强制一致性的方法。有了这些特定的属性集，应用程序和接口就可以根据约定来假设默认类型的属性。

  

* **Infrastructure**【译：基础设施】

  Infrastructure扩展了Asset类型，典型且常见的使用Infrastructure的超类有：群集，主机等。

  

* **DataSet**

  DataSet扩展了Referenceable。**在概念上，它可以用于表示任何数据存储的类型**。在 Atlas 中，hive表，Sqoop RDBMS表等都是从 DataSet 扩展的类型。扩展 DataSet 的类型应该具有期望模式，它们将具有定义该数据集的属性的属性。例如， hive_table 中的 columns 属性。另外，扩展 DataSet 的实体类型的实体参与数据转换，这种转换可以由 Atlas 通过 lineage（或 provenance）生成图形。

* **Process**

  Process扩展了 Asset。**在概念上，它可以用于表示任何数据变换操作**。例如，<u>将一个带有原始数据的hive表转换为另一个存储某些聚合的hive表的ETL进程</u>可以是扩展了Process类型的特定类型。Process类型有两个特定的属性， inputs 和 outputs。 inputs 和 outputs都是 DataSet 实体的数组。因此，Process 类型的实例可以使用这些 inputs 和 outputs来捕获 DataSet 的 lineage 如何演变。



## 二、部署与测试

基础环境：Ambari-2.7.3 + HDP-3.1.0

```yaml
部署节点: flink02.baicdt.com
Atlas Admin UI: http://flink02.baicdt.com:21000/login.jsp
用户名: admin
密  码: Wsygzgr_2020
```

>Atlas Admin UI: 该组件是一个基于 Web 的应用程序，允许数据管理员和科学家发现和注释元数据。Admin UI提供了搜索界面和 类SQL的查询语言，可以用来查询由 Atlas 管理的元数据类型和对象。Admin UI 使用 Atlas 的 REST API 来构建其功能。



### 2.1 Hive-Hook

http://atlas.apache.org/1.2.0/Hook-Hive.html

#### 1. 导入Hive元数据

```shell
# 切换到atlas用户下操作
su atlas
cd ~
/usr/hdp/3.1.0.0-78/atlas/hook-bin/import-hive.sh
# 输入用户名 admin    密码 Wsygzgr_2020
```

导入成功，日志如下

```java
2020-08-13T09:14:47,963 INFO [main] org.apache.atlas.hive.bridge.HiveMetaStoreBridge - Successfully imported 78 tables from database whdb
Hive Meta Data imported successfully!!!
```

> 库表越多，导入的时间越长

在Atlas Admin UI中查看Hive的元数据

![image-20200813093629118](img/image-20200813093629118.png)



#### 2. 测试

```shell
[root@flink02 ~]# sudo -u hdfs hdfs dfs -mkdir -p /yanxu-jin/atlas/hive
[root@flink02 ~]# sudo -u hdfs hdfs dfs -rmr /warehouse/tablespace/external/hive/atlasdb.db
```

在hive中创建一个atlasdb数据库，用来测试

```sql
drop database if exists atlas;
create database atlasdb;
```

使用sqoop将数据导入到hdfs

```shell
sudo -u hdfs sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
--split-by com_id \
--connect jdbc:mysql://ambari03.baicdt.com:3306/bitestdb?useSSL=false \
--username root \
--password 123456 \
--table t_yyc_ts_info \
--target-dir /yanxu-jin/atlas/hive/suv-com-info \
--delete-target-dir
```

建表并加载数据

```sql
# 建表 `atlasdb`.`ods_suv_com_info`
drop table if exists `atlasdb`.`ods_suv_com_info`;
CREATE external TABLE `atlasdb`.`ods_suv_com_info`(
  `com_id` string COMMENT 'complain投诉编号', 
  `com_agent_name` string COMMENT 'complain经销商名称  周口京彩润，北京鹏元，长沙脸谱，南阳京润, 昆明孝尊 ', 
  `com_type_id` string COMMENT 'complain投诉类型id', 
  `com_type_name` string COMMENT 'complain投诉类型名称 1001 产品质量 1002 售后投诉 1003 配件投诉 1004销售投诉 1005其它类投诉', 
  `com_car_type` string COMMENT 'complain投诉车型 BJ40L,BJ40 PLUS,B80C,B40皮卡,B40柴油版，B40V,B40L PLUS-柴油版，BJ80,BJ80汽油版，BJ40环塔冠军版，BJ40城市猎人版，BJ40PLUS域 ', 
  `com_info` string COMMENT 'complain投诉内容', 
  `com_time` string COMMENT 'complain投诉时间', 
  `com_dt_limit` string COMMENT 'complain最晚投诉处理期限 ', 
  `com_status` string COMMENT 'complain投诉处理状态，10待处理，11已处理', 
  `com_do_time` string COMMENT 'complain投诉处理时间', 
  `com_do` int COMMENT 'complain投诉处理周期  多少天')
 row format delimited fields terminated by ',' STORED AS textfile;
 
# 从hdfs文件系统向表中加载数据(移动操作)
load data inpath '/yanxu-jin/atlas/hive/suv-com-info/' into table `atlasdb`.`ods_suv_com_info`;

select * from `atlasdb`.`ods_suv_com_info` limit 10;
```

查看ods_suv_com_info表的数据血缘

![image-20200813142533370](img/image-20200813142533370.png)



```sql
drop table if exists `atlasdb`.`dim_tbl_car_type`;
CREATE external TABLE `atlasdb`.`dim_tbl_car_type`(
    `car_type` string COMMENT '车型', 
    `trial_produce_time` string COMMENT '量产时间',
    `produce_address` string COMMENT '产地' 
) row format delimited fields terminated by '\u0001' STORED AS orc;

insert into `atlasdb`.`dim_tbl_car_type` values ('B80C','2011','北京'),('B40L PLUS-柴油版','2012','福建'),('BJ80','2013','北京'),('B40柴油版','2014','福建'),('BJ40 PLUS','2015','北京'),('B40V','2016','北京'),('BJ40L','2017','北京'),('BJ40城市猎人版','2018','福建'),('BJ80汽油版','2019','北京'),('BJ40PLUS域','2020','北京');

select * from `atlasdb`.`dim_tbl_car_type`;

-- -----------------------------------------------


drop table if exists `atlasdb`.`dwd_suv_com_info`;

create table `atlasdb`.`dwd_suv_com_info` as 
select b.`car_type`,b.`trial_produce_time`,b.`produce_address`,a.`count_car_type` from 
(select `com_car_type`, count(`com_car_type`) as `count_car_type`
  from `atlasdb`.`ods_suv_com_info`
  group by com_car_type) as a
inner join `atlasdb`.`dim_tbl_car_type` as b
on a.`com_car_type` = b.`car_type`;

select * from `atlasdb`.`dwd_suv_com_info` limit 10;
```



![image-20200813153723085](img/image-20200813153723085.png)



```sql
drop table if exists `atlasdb`.`rs_suv_com_info`;
create table `atlasdb`.`rs_suv_com_info` as select `car_type`, `produce_address` from `atlasdb`.`dwd_suv_com_info`;
select * from `atlasdb`.`rs_suv_com_info`;
```

#### 3. Limitations【局限性】

hive hook 仅支持捕获以下 hive 操作

- create database
- create table/view, create table as select
- load, import, export
- DMLs (insert)
- alter database
- alter table (skewed table information, stored as, protection is not supported)
- alter view



### 2.2 Sqoop-Hook

#### 1. 测试

```shell
sudo -u hdfs sqoop import "-Dorg.apache.sqoop.splitter.allow_text_splitter=true" \
--split-by cif_no \
--connect jdbc:mysql://ambari03.baicdt.com:3306/mysqlsourcedb?useSSL=false \
--username root \
--password 123456 \
--input-fields-terminated-by "\t" \
--table t_user_reg \
--hive-import \
--hive-table atlasdb.import_t_user_reg \
--delete-target-dir
```

sqoop的后几行日志为：

```java
20/08/14 10:35:35 INFO utils.AppInfoParser: Kafka version : 2.0.0.3.1.0.0-78
20/08/14 10:35:35 INFO utils.AppInfoParser: Kafka commitId : 0f47b27cde30d177
20/08/14 10:35:35 INFO kafka.KafkaNotification: <== KafkaNotification.createProducer()
20/08/14 10:35:35 INFO clients.Metadata: Cluster ID: 51_dm1oiQkKFyxmW0lmXIQ
20/08/14 10:35:35 INFO hook.AtlasHook: <== Shutdown of Atlas H
```

在Atlas Admin UI中查看Sqoop元数据

![image-20200814103900405](img/image-20200814103900405.png)



#### 2. Limitations【局限性】

&ensp;&ensp;&ensp;&ensp; atlas sqoop-hook目前只支持捕获hiveImport 这一种 sqoop 操作



### 2.3 HBase-Hook

#### 1. 导入Hbase元数据

```shell
# 切换到atlas用户下操作
su atlas
cd ~
/usr/hdp/3.1.0.0-78/atlas/hook-bin/import-hbase.sh
# 输入用户名 admin    密码 Wsygzgr_2020
```

导入成功，日志如下

```java
Log file for import is /usr/hdp/3.1.0.0-78/atlas/logs/import-hbase.log
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/phoenix/phoenix-5.0.0.3.1.0.0-78-server.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/hdp/3.1.0.0-78/hadoop/lib/slf4j-log4j12-1.7.25.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
Enter username for atlas :- admin
Enter password for atlas :- 
HBase Data Model imported successfully!!!
```

> 库表越多，导入的时间越长



在Atlas Admin UI中查看HBase的元数据

![image-20200814110234507](img/image-20200814110234507.png)



#### 2. 测试

进入phoenix客户端

```shell
[root@flink02 hook-bin]# /usr/hdp/3.1.0.0-78/phoenix/bin/sqlline.py ambari01.baicdt.com,ambari02.baicdt.com,ambari03.baicdt.com:2181
```

在phoenix客户端创建worker表

```sql
-- drop table "worker";
create table "worker"(
 "id" varchar primary key,
 "info"."name" varchar,
 "info"."sex" varchar,
 "data"."age" varchar
);

upsert into "worker" values('1001','jack','male','16');
upsert into "worker" values('1002','rose','female','17');
upsert into "worker" values('1003','devin','male','18');

select * from "worker";
```

创建覆盖索引

```sql
-- drop index "worker_index_name_age" on "worker";
create index "worker_index_name_age" on "worker" ("info"."name") include ("data"."age");

-- 添加多个字段时
-- drop index "worker_index_name_age_sex" on "worker";
-- create index "worker_index_name_age_sex" on "worker" ("info"."name") include ("data"."age","info"."sex");

explain select "id","name","age" from "worker" where "name" = 'jack';
```

在Atlas Admin UI中查看HBase的元数据

![image-20200814111649412](img/image-20200814111649412.png)



**hive on hbase测试**

```sql
drop table if exists `atlasdb`.`user_from_hbase`;

create external table `atlasdb`.`user_from_hbase`(
  id string,
  name string,
  sex string,
  age string
)stored by 'org.apache.hadoop.hive.hbase.HBaseStorageHandler'
with serdeproperties ("hbase.columns.mapping" = ":key,info:name,info:sex,data:age")
tblproperties("hbase.table.name" = "user_to_hive");

insert into `atlasdb`.`user_from_hbase` values ('1001','jack','male','16'),('1002','rose','female','17'),('1003','devin','male','18');


select * from `atlasdb`.`user_from_hbase`;
```

![image-20200814114149397](img/image-20200814114149397.png)



#### 3. Limitations【局限性】

&ensp;&ensp;&ensp;&ensp;atlas hbase-hook只捕获命名空间、表和列族的创建/更新/删除操作以及对列的更改操作。



## 三、Atlas Rest API 入门级

总结部分Rest API，其他API查看Atlas官网[Atlas Rest API](http://atlas.apache.org/api/v2/index.html)

### 3.1 AdminREST

#### 1. `GET` /admin/status 查看Atlas Metadata Server节点状态 

使用shell请求

```shell
curl -s -u admin:Wsygzgr_2020 "http://flink02.baicdt.com:21000/api/atlas/admin/version"
```

使用idea请求

```http
### GET /admin/status   查看Atlas Metadata Server节点状态
GET http://flink02.baicdt.com:21000/api/atlas/admin/status
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```

>ACTIVE:此实例处于活跃状态，可以响应用户请求。
>PASSIVE:此实例处于被动状态。它会将收到的任何用户请求重定向到当前ACTIVE实例。
>BECOMING_ACTIVE:此实例正在转换为ACTIVE实例，在此状态下无法为用户提供请求服务。
>BECOMING_PASSIVE:此实例正在转换为PASSIVE实例，在此状态下无法为用户提供请求服务。
>
>注意：正常情况下，只有一个应该为ACTIVE状态，其他实例均为PASSIVE状态。



#### 2. `GET` /admin/version 查看Atlas版本和描述 

```http
###  GET /admin/version    查看Atlas版本和描述
GET http://flink02.baicdt.com:21000/api/atlas/admin/version
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



### 3.2 DiscoveryREST

#### 1. `GET` /v2/search/basic 基本搜索 

```http
###  GET /v2/search/basic  基本搜索--查询所有Hive表
GET http://flink02.baicdt.com:21000/api/atlas/v2/search/basic?typeName=hive_table
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```

```http
###   GET /v2/search/basic  基本搜索--所有Hive表，且包含某一关键字
GET http://flink02.baicdt.com:21000/api/atlas/v2/search/basic?query=t_result_day_task_status&typeName=hive_table
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



#### 2. `GET` /v2/search/dsl 使用DSL 搜索 

```http
###   GET /v2/search/dsl    DSL 搜索--queryText: `hive_table` where name = \"ods_suv_com_info\" 
GET http://flink02.baicdt.com:21000/api/atlas/v2/search/dsl?typeName=hive_table&query=where%20name%20%3D%20%22ods_suv_com_info%22
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```

> 注意:URL中特殊字符编码，可参考https://www.w3school.com.cn/tags/html_ref_urlencode.html



#### 3. `GET` /v2/search/fulltext 全文检索

```http
###   GET /v2/search/fulltext   全文检索方式查询
GET http://flink02.baicdt.com:21000/api/atlas/v2/search/fulltext?query=where%20name%3D%22dim_tbl_car_type%22
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



### 3.3 TypesREST

#### 1. `GET` /v2/types/typedefs 检索所有Type

```http
###  GET /v2/types/typedefs     检索所有Type
GET http://flink02.baicdt.com:21000/api/atlas/v2/types/typedefs
```

```http
###  GET /v2/types/typedefs/headers    检索所有Type，并返回最少信息
GET http://flink02.baicdt.com:21000/api/atlas/v2/types/typedefs/headers
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



### 3.4 EntityREST

#### 1. `GET` /v2/entity/bulk 批量根据GUID检索Entity 

```http
###  GET /v2/entity/bulk       批量根据GUID检索Entity
GET http://flink02.baicdt.com:21000/api/atlas/v2/entity/bulk?minExtInfo=yes&guid=469df62c-61df-442c-a5a9-b649acb2c6d1
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```

> guid=469df62c-61df-442c-a5a9-b649acb2c6d1是hive_table：atlasdb.import_t_user_reg



#### 2. `GET` /v2/entity/guid/{guid} 获取某个Entity定义 

```http
###  GET /v2/entity/guid/{guid}   获取某个Entity定义
GET http://flink02.baicdt.com:21000/api/atlas/v2/entity/guid/469df62c-61df-442c-a5a9-b649acb2c6d1
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



#### 3. `GET` /v2/entity/guid/{guid}/classifications 获取某个Entity的TAG列表 

```http
###  GET /v2/entity/guid/{guid}/classifications   获取某个Entity的TAG列表
GET http://flink02.baicdt.com:21000/api/atlas/v2/entity/guid/469df62c-61df-442c-a5a9-b649acb2c6d1/classifications
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```



### 3.5 LineageREST

#### 1. `GET` /v2/lineage/{guid} 查询某个Entity的Lineage

```http
### GET /v2/lineage/{guid}        查询某个Entity的Lineage
GET http://flink02.baicdt.com:21000/api/atlas/v2/lineage/a7c15ca6-3c97-4b84-8413-f3c24c1b0a92
Authorization: Basic YWRtaW46V3N5Z3pncl8yMDIw
```

> guid=a7c15ca6-3c97-4b84-8413-f3c24c1b0a92是hive_table：atlasdb.rs_suv_com_info
>
> 数据血缘关系的描述主要返回体在"relations"属性




