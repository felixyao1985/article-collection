# MySQL分区表：万字详解与实践指南_mysql分区架构-CSDN博客
[MySQL 分区表：万字详解与实践指南\_mysql 分区架构 - CSDN 博客](https://blog.csdn.net/qq_26664043/article/details/138452285?spm=1001.2100.3001.7377&utm_medium=distribute.pc_feed_blog_category.none-task-blog-classify_tag-2-138452285-null-null.nonecase&depth_1-utm_source=distribute.pc_feed_blog_category.none-task-blog-classify_tag-2-138452285-null-null.nonecase) 

### MySQL 分区是一种[数据库优化]

* * *

> MySQL 分区是一种[数据库优化](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E5%BA%93%E4%BC%98%E5%8C%96&spm=1001.2101.3001.7020)的技术，它允许将一个大的表或一个索引分割成多个较小的、更易于管理的片段，称为分区。这种技术可以显著提高查询性能、维护的方便性以及数据管理效率。本文将详细介绍 MySQL 分区的基本概念、工作原理、使用场景以及操作。

#### 目录

-   -   [一、分区的基本概念](#_9)
    -   [二、分区的原理和类型](#_30)
    -   -   -   [InnoDB 逻辑存储结构](#InnoDB_32)
            -   [分区的原理](#_46)
            -   [分区类型](#_62)
    -   [三、分区的优势和使用场景](#_71)
    -   [四、如何实施分区](#_82)
    -   [五、分区表的操作](#_106)
    -   -   [5.1. 创建带有分区的表](#51__110)
        -   -   [RANGE 分区](#RANGE__112)
            -   [LIST 分区](#LIST__127)
            -   [HASH 分区](#HASH__142)
            -   [KEY 分区](#KEY__152)
        -   [5.2. 修改分区表](#52__163)
        -   -   [添加分区](#_165)
            -   [删除分区](#_175)
            -   [合并分区](#_184)
            -   [拆分分区](#_203)
            -   [重建分区](#_231)
            -   [优化分区](#_245)
            -   [分析分区](#_259)
            -   [检查分区](#_273)
            -   [修补分区](#_287)
        -   [5.3. 查看分区信息](#53__300)
    -   [六、复合分区](#_314)
    -   [七、注意事项和限制](#_355)
    -   [八、解释几个问题](#_366)
    -   -   -   [8.1 MySQL 分区处理 NULL 值的方式](#81_MySQLNULL_368)
            -   [8.2 分区列必须主键或唯一键的一部分](#82__375)
            -   [8.3 分区与性能考量](#83__391)

### 一、分区的基本概念

MySQL 分区 是一种数据库优化的技术，它允许将一个大的表、索引或其子集分割成多个较小的、更易于管理的片段，这些片段称为 “分区”。每个分区都可以独立于其他分区进行存储、备份、索引和其他操作。这种技术主要是为了改善大型数据库表的查询性能、维护的方便性以及[数据管理](https://so.csdn.net/so/search?q=%E6%95%B0%E6%8D%AE%E7%AE%A1%E7%90%86&spm=1001.2101.3001.7020)效率。

**物理存储与逻辑分割**

-   物理上，每个分区可以存储在不同的文件或目录中，这取决于分区类型和配置。
-   逻辑上，表数据根据分区键的值被分割到不同的分区里。

**查询性能提升**

-   当执行查询时，MySQL 能够确定哪些分区包含相关数据，并只在这些分区上进行搜索。这减少了需要搜索的数据量，从而提高了查询性能。
-   对于范围查询或特定值的查询，分区可以显著减少扫描的数据量。

**数据管理与维护**

-   分区可以使得数据管理更加灵活。例如，可以独立地备份、恢复或优化某个分区，而无需对整个表进行操作。
-   对于具有时效性的数据，可以通过删除或归档某个分区来快速释放存储空间。

**扩展性与并行处理**

-   分区技术使得数据库表更容易扩展到更大的数据集。当表的大小超过单个存储设备的容量时，可以使用分区将数据分布到多个存储设备上。
-   由于每个分区可以独立处理，因此可以并行执行查询和其他数据库操作，从而进一步提高性能。

### 二、分区的原理和类型

##### InnoDB 逻辑存储结构

InnoDB 存储引擎的逻辑结构是一个层次化的体系，主要由表空间、段、区和页构成。  
![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-5-28%2015-25-17/f7f74c22-6018-4e57-b90f-d1634387f9fd.jpeg?raw=true)

1.  **表空间**：是 InnoDB 数据的最高层容器，所有数据都逻辑地存储在这里。
2.  **段（Segment）**：是表空间的重要组成部分，根据用途可分为数据段、索引段和回滚段等。InnoDB 引擎负责管理这些段，确保数据的完整性和高效访问。
3.  **区（Extent）**：由连续的页组成，每个区默认大小为 1MB，不论页的大小如何变化。为保证页的连续性，InnoDB 会一次性从磁盘申请多个区。每个区包含 64 个连续的页，当默认页大小为 16KB 时。在段开始时，InnoDB 会先使用 32 个碎片页存储数据，以优化小表或特定段的空间利用率。
4.  **页（Page）**：是 InnoDB 磁盘管理的最小单元，也被称为块。其默认大小为 16KB，但可通过配置参数进行调整。页的类型多样，包括数据页、undo 页、系统页等，每种页都有其特定的功能和结构。

##### 分区的原理

分区技术是将表中的记录分散到不同的物理文件中，即每个分区对应一个. idb 文件。这是 MySQL 5.1 及以后版本支持的一项高级功能，旨在提高大数据表的管理效率和查询性能。  
![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-5-28%2015-25-17/ff4ab531-3540-420e-bfb3-8bb228182189.jpeg?raw=true)

1.  **分区类型**：MySQL 支持水平分区，即根据某些条件将表中的行分配到不同的分区中。这些分区在物理上是独立的，可以单独处理，也可以作为整体处理。
2.  **性能和影响**：虽然分区可以提高查询性能和管理效率，但如果不恰当使用，也可能对性能产生负面影响。因此，在使用分区时应谨慎评估其影响。
3.  **索引与分区**：在 MySQL 中，分区是局部的，意味着数据和索引都存储在各自的分区内。目前，MySQL 尚不支持全局分区索引。
4.  **分区键与唯一索引**：当表存在主键或唯一索引时，分区列必须是这些索引的一部分。这是为了确保分区的唯一性和查询效率。

通过合理利用分区技术，可以优化数据库性能、提高管理效率，并更好地适应大规模数据处理的需求。然而，为了充分利用这一功能，数据库管理员和开发者需要深入了解其工作原理和最佳实践。

##### 分区类型

MySQL 支持几种不同类型的分区方式，包括 RANGE、LIST、HASH 和 KEY。下面简要介绍这些分区方式的工作原理：

1.  **RANGE 分区**：基于列的值范围将数据分配到不同的分区。例如，可以根据日期范围将数据分配到不同的月份或年份的分区中。
2.  **LIST 分区**：类似于 RANGE 分区，但 LIST 分区是基于列的离散值集合来分配数据的。可以指定一个枚举列表来定义每个分区的值。
3.  **HASH 分区**：基于用户定义的表达式的哈希值来分配数据到不同的分区。这种分区方式适用于确保数据在各个分区之间均匀分布。
4.  **KEY 分区**：类似于 HASH 分区，但 KEY 分区支持计算一列或多列的哈希值来分配数据。它支持多列作为分区键，并且提供了更好的数据分布和查询性能。

### 三、分区的优势和使用场景

MySQL 分区带来了许多优势，适用于各种使用场景：

1.  **性能提升**：通过将数据分散到多个分区中，可以并行处理查询，从而提高查询性能。同时，对于涉及大量数据的维护操作（如备份和恢复），可以单独处理每个分区，减少了操作的复杂性和时间成本。
2.  **管理简化**：分区可以使得数据管理更加灵活。例如，可以独立地备份、恢复或优化某个分区，而无需对整个表进行操作。这对于大型数据库表来说尤为重要，因为它可以显著减少维护时间和资源消耗。
3.  **数据归档和清理**：对于具有时间属性的数据（如日志、交易记录等），可以使用分区来轻松归档旧数据或删除不再需要的数据。通过简单地删除或归档某个分区，可以快速释放存储空间并提高性能。
4.  **可扩展性**：分区技术使得数据库表更容易扩展到更大的数据集。当表的大小超过单个存储设备的容量时，可以使用分区将数据分布到多个存储设备上，从而实现水平扩展。  
    ![](https://github.com/felixyao1985/article-collection/blob/main/drawing-bed/2024-5-28%2015-25-17/eef2a6a1-37ff-4062-9e89-499bcde075a9.jpeg?raw=true)

### 四、如何实施分区

实施 MySQL 分区需要仔细规划和设计。以下是一些建议的步骤：

1.  **确定分区键**：选择一个合适的列作为分区键，该列的值将用于将数据分配到不同的分区中。通常选择具有连续值或离散值的列作为分区键。
2.  **选择合适的分区类型**：根据数据的特点和查询需求选择合适的分区类型（RANGE、LIST、HASH 或 KEY）。确保所选的分区类型能够均匀地分布数据并提高查询性能。
3.  **创建分区表**：使用`CREATE TABLE`语句创建分区表，并指定分区键和分区类型等参数。例如，使用 RANGE 分区类型创建一个按月分区的销售数据表：


```
`CREATE TABLE sales (
    sale_id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    ...
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p0 VALUES LESS THAN (2022),
    PARTITION p1 VALUES LESS THAN (2023),
    PARTITION p2 VALUES LESS THAN MAXVALUE
);` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10


```

4.  **查询和维护**：一旦创建了分区表，就可以像普通表一样执行查询操作。MySQL 会自动定位到相应的分区上执行查询。同时，可以独立地备份、恢复或优化每个分区。
5.  **监控和调整**：定期监控分区的性能和存储使用情况，并根据需要进行调整。例如，可以添加新的分区来容纳新数据，或者删除旧的分区以释放存储空间。

### 五、分区表的操作

包括创建分区表、修改分区和删除、合并、拆分等。

#### 5.1. 创建带有分区的表

##### RANGE 分区

```
`CREATE TABLE sales_range (
    id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p0 VALUES LESS THAN (2010),
    PARTITION p1 VALUES LESS THAN (2011),
    PARTITION p2 VALUES LESS THAN (2012),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10


```

##### LIST 分区

```
`CREATE TABLE sales_list (
    id INT NOT NULL,
    region ENUM('North', 'South', 'East', 'West') NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
) PARTITION BY LIST COLUMNS(region) (
    PARTITION pNorth VALUES IN('North'),
    PARTITION pSouth VALUES IN('South'),
    PARTITION pEast VALUES IN('East'),
    PARTITION pWest VALUES IN('West')
);` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10


```

##### HASH 分区

```
`CREATE TABLE sales_hash (
    id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL
) PARTITION BY HASH(YEAR(sale_date)) PARTITIONS 4;` 

*   1
*   2
*   3
*   4
*   5


```

##### KEY 分区

```
`CREATE TABLE sales_key (
    id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    PRIMARY KEY (id, sale_date)
) PARTITION BY KEY(id) PARTITIONS 4;` 

*   1
*   2
*   3
*   4
*   5
*   6


```

#### 5.2. 修改分区表

##### 添加分区

对于 RANGE 或 LIST 分区，可以使用 `ALTER TABLE` 语句添加分区：

```
`ALTER TABLE sales_range ADD PARTITION (PARTITION p4 VALUES LESS THAN (2013));` 

*   1


```

对于 HASH 或 KEY 分区，由于它们是基于哈希函数进行分区的，因此不能直接添加分区，但可以通过重新创建表或调整分区数量来间接实现。

##### 删除分区

可以使用 `ALTER TABLE` 语句删除分区：

```
`ALTER TABLE sales_range DROP PARTITION p0;` 

*   1


```

这将删除名为 `p0` 的分区及其包含的所有数据。

##### 合并分区

对于相邻的 RANGE 或 LIST 分区，可以使用 `ALTER TABLE` 语句将它们合并为一个分区：

```
`ALTER TABLE sales_range REORGANIZE PARTITION p1, p2 INTO (
    PARTITION p1_2 VALUES LESS THAN (2012)
);` 

*   1
*   2
*   3


```

把 `p1` 和 `p2` 分区合并为一个名为 `p1_2` 的新分区。

**分区拆分限制：** 

1.  **分区数量限制**：MySQL 对单个表的分区数量有限制，通常最大分区数目不能超过 1024 个。这意味着在进行拆分操作时，需要注意新生成的分区数量是否会超过这个限制。
2.  **分区键和分区类型的限制**：拆分操作通常受到分区键和分区类型的约束。例如，在 RANGE 分区中，拆分点必须基于分区键的连续值。对于 LIST 分区，拆分需要基于离散的枚举值。HASH 和 KEY 分区由于其基于哈希函数的特性，不直接支持拆分操作。
3.  **数据完整性**：拆分分区时，需要确保数据的完整性。如果拆分操作导致数据丢失或损坏，那么这将是一个严重的问题。因此，在执行拆分操作之前，最好进行数据备份。
4.  **性能考虑**：拆分大分区可能会影响数据库性能，因为需要重建索引和移动大量数据。这种操作最好在数据库负载较低的时候进行。

##### 拆分分区

使用 ALTER TABLE 语句来拆分分区。语法，用于 RANGE 分区：

```
`ALTER TABLE table_name REORGANIZE PARTITION partition_name INTO (  
    PARTITION new_partition1 VALUES LESS THAN (value1),  
    PARTITION new_partition2 VALUES LESS THAN (value2)  
);` 

*   1
*   2
*   3
*   4


```

table_name 是你要修改的表名，partition_name 是要拆分的分区名，new_partition1 和 new_partition2 是新分区的名称，而 value1 和 value2 是定义新分区键值范围的值。

```
`ALTER TABLE sales_range REORGANIZE PARTITION p1_2 INTO (  
    PARTITION p1 VALUES LESS THAN (value1),  
    PARTITION p2 VALUES LESS THAN (value2)  
);` 

*   1
*   2
*   3
*   4


```

把一个名为 `p1_2` 的分区拆分为 `p1` 和 `p2` 两个分区。

**分区合并限制：** 

1.  **相邻分区合并**：在 MySQL 中，通常只能合并相邻的分区。这意味着你不能随意选择两个不相邻的分区进行合并。
2.  **分区类型和键的限制**：与拆分操作类似，合并操作也受到分区类型和分区键的约束。不是所有类型的分区都可以轻松合并。
3.  **数据迁移和重建**：合并分区时，可能需要进行数据迁移和索引重建，这可能会影响数据库的性能和可用性。

##### 重建分区

重建分区相当于先清除分区内的所有数据，并随后重新插入，这有助于整理分区内的碎片。

-   **语法**:


```
`ALTER TABLE tbl_name REBUILD PARTITION partition_name_list;` 

*   1


```

-   **示例**:


```
`ALTER TABLE tbl_users REBUILD PARTITION p2, p3;` 

*   1


```

通过这一操作，可以高效地整理`p2`和`p3`这两个分区中的碎片。

##### 优化分区

当从分区中删除了大量数据，或者对包含可变长度字段（如 VARCHAR 或 TEXT 类型列）的分区进行了多次修改后，优化分区可以回收未使用的空间并整理数据碎片。

-   **语法**:


```
`ALTER TABLE tbl_name OPTIMIZE PARTITION partition_name_list;` 

*   1


```

-   **示例**:


```
`ALTER TABLE tbl_users OPTIMIZE PARTITION p2, p3;` 

*   1


```

执行此操作后，`p2`和`p3`分区将会更加紧凑，未使用的空间将被回收。

##### 分析分区

此操作会读取并保存分区的键分布统计信息，有助于查询优化器制定更有效的查询计划。

-   **语法**:


```
`ALTER TABLE tbl_name ANALYZE PARTITION partition_name_list;` 

*   1


```

-   **示例**:


```
`ALTER TABLE tbl_users ANALYZE PARTITION p2, p3;` 

*   1


```

对`p2`和`p3`分区进行分析后，数据库能更准确地为这两个分区上的查询制定执行计划。

##### 检查分区

此操作用于验证分区中的数据或索引是否完整无损。

-   **语法** :


```
`ALTER TABLE tbl_name CHECK PARTITION partition_name_list;` 

*   1


```

-   **示例**:


```
`ALTER TABLE tbl_users CHECK PARTITION p2, p3;` 

*   1


```

执行检查可以确保`p2`和`p3`分区的数据和索引的完整性。

##### 修补分区

如果分区数据或索引受损，可以使用此操作进行修复。

-   **语法**:


```
`ALTER TABLE tbl_name REPAIR PARTITION partition_name_list;` 

*   1


```

-   **示例**:


```
`ALTER TABLE tbl_users REPAIR PARTITION p2, p3;` 

*   1


```

执行修补操作后，`p2`和`p3`分区中的任何损坏都将被修复。

#### 5.3. 查看分区信息

可以使用以下查询来查看表的分区信息：

```
`SELECT * FROM INFORMATION_SCHEMA.PARTITIONS WHERE TABLE_NAME = 'sales_range';` 

*   1


```

或者使用 `SHOW CREATE TABLE` 语句来查看表的创建语句，包括分区定义：

```
`SHOW CREATE TABLE sales_range;` 

*   1


```

### 六、复合分区

复合分区是指在分区表中的每个分区再次进行分割，这种再次分割的子分区既可以使用 HASH 分区，也可以使用 KEY 分区。这种技术也被称为子分区。

**使用场景**

-   **数据量巨大**：当表中的数据量非常大时，单一分区可能无法满足性能需求。复合分区可以将数据更细致地划分，从而提高查询效率。
-   **多维度查询优化**：如果查询经常涉及多个维度（如时间和地区），复合分区可以针对这些维度进行分区，从而优化查询性能。

**在复合分区中，常见的组合是 RANGE 或 LIST 与 HASH 或 KEY 的组合**

创建一个记录用户行为日志的表，首先根据日志日期进行`RANGE`分区，然后在每个日期范围内根据用户 ID 进行`HASH`子分区。

```
`CREATE TABLE user_activity_logs (
    log_id BIGINT NOT NULL AUTO_INCREMENT,
    user_id INT NOT NULL,
    activity_date DATE NOT NULL,
    activity_description VARCHAR(255) NOT NULL,
    PRIMARY KEY (log_id, user_id)
)
PARTITION BY RANGE COLUMNS(activity_date) (
    PARTITION p2022 VALUES LESS THAN ('2023-01-01') (
        SUBPARTITION sp2022a HASH(user_id) PARTITIONS 4
    ),
    PARTITION p2023 VALUES LESS THAN ('2024-01-01') (
        SUBPARTITION sp2023 HASH(user_id) PARTITIONS 4
    ),
    -- 可以根据需要继续添加更多的年份分区和HASH子分区
    PARTITION pfuture VALUES LESS THAN (MAXVALUE) (
        SUBPARTITION spfuture HASH(user_id) PARTITIONS 4
    )
);` 

*   1
*   2
*   3
*   4
*   5
*   6
*   7
*   8
*   9
*   10
*   11
*   12
*   13
*   14
*   15
*   16
*   17
*   18
*   19


```

-   先根据`activity_date`进行范围分区。每个范围分区内部，又根据`user_id`进行了`HASH`子分区。这样做的好处是可以更均匀地分布数据，提高查询性能，特别是当查询条件同时包含日期和用户 ID 时。
-   预留了一个名为`pfuture`的分区，它的范围是小于最大值（`MAXVALUE`），这样可以确保未来的日志也能被正确地插入到表中。
-   `PARTITIONS 4`表示在每个范围分区内创建 4 个哈希子分区。这个数字可以根据数据量的大小和查询模式进行调整。

### 七、注意事项和限制

在实施 MySQL 分区时，需要注意以下事项和限制：

1.  **分区键选择**：选择合适的分区键至关重要。确保分区键能够均匀地分布数据，并且与查询条件相匹配，以提高查询性能。
2.  **分区数量限制**：MySQL 对单个表的分区数量有限制（通常为 1024 个分区）。在设计分区策略时要考虑这个限制。
3.  **全局唯一索引限制**：在分区表上创建全局唯一索引时存在限制。确保了解这些限制，并根据需要进行调整。
4.  **性能和资源消耗**：虽然分区可以提高性能，但在某些情况下，过多的分区可能导致额外的性能和资源消耗。因此，要合理设计分区策略以平衡性能和资源消耗。
5.  **兼容性和迁移**：在迁移现有表到分区表之前，要确保备份原始数据并测试迁移过程的正确性。此外，要了解不同 MySQL 版本之间对分区功能的支持和兼容性差异。

### 八、解释几个问题

##### 8.1 MySQL 分区处理 NULL 值的方式

MySQL 中，当涉及到分区时，系统并不会特别禁止 NULL 值。不论是列的实际值还是用户自定义的表达式结果，MySQL 通常会将 NULL 值视为 0 进行处理。然而，这种行为可能并不总是符合数据完整性和准确性的要求。为了避免这种隐式的 NULL 到 0 的转换，最佳实践是在设计数据库表时，对相关列明确声明为 “NOT NULL”。这样做可以确保数据的准确性和一致性，同时避免由于 NULL 值被错误地解释为 0 而导致的潜在问题。因此，在设计分区表时，应该谨慎考虑 NULL 值的处理方式，并根据需要采取相应的预防措施。

此外，如果确实需要存储 NULL 值，并且不希望 MySQL 将其视为 0，可以考虑使用其他特殊值（如某个不可能在实际业务中出现的标识值）来代替 NULL，或者在设计分区策略时明确考虑 NULL 值的处理逻辑。这样可以在保持数据完整性的同时，更好地满足业务需求。

##### 8.2 分区列必须主键或唯一键的一部分

在 MySQL 中，当表存在主键（primary key）或唯一键（unique key）时，分区的列必须是这些键的一个组成部分的原因主要涉及到数据的完整性和查询性能：

1.  **数据完整性**：

    -   主键和唯一键用于保证表中数据的唯一性。如果分区列不是这些键的一部分，那么在不同分区中可能存在具有相同主键或唯一键值的数据行，这将破坏数据的唯一性约束。
2.  **查询性能**：

    -   分区的主要目的是为了提高查询性能，特别是针对大数据量的表。如果分区列不是主键或唯一键的一部分，那么在进行基于主键或唯一键的查询时，MySQL 可能需要在所有分区中进行搜索，从而降低了查询性能。
3.  **数据一致性**：

    -   当表被分区时，每个分区实际上可以看作是一个独立的 “子表”。如果分区列不是主键或唯一键的一部分，那么在执行更新或删除操作时，MySQL 需要确保跨所有分区的数据一致性，这会增加操作的复杂性和开销。
4.  **分区策略**：

    -   MySQL 的分区策略是基于分区列的值来将数据分配到不同的分区中。如果分区列不是主键或唯一键的一部分，那么分区策略可能会变得复杂且低效，因为系统需要额外处理主键或唯一键的约束。

##### 8.3 分区与性能考量

技术的运用需要恰到好处才能发挥其优势。以显式锁为例，虽然功能强大，但使用不当可能导致性能下降或其他不良后果。同样地，分区技术也并非万能的性能提升工具。

分区确实可以为某些 SQL 查询带来性能上的提升，但其主要价值在于提高数据库的高可用性管理。在应用分区技术时，我们需要根据数据库的使用场景来谨慎选择。

数据库应用大体上可分为 OLTP（在线事务处理）和 OLAP（在线分析处理）两类。对于 OLAP 应用来说，分区能够显著提升查询性能，因为分析类查询往往需要处理大量数据。按时间进行分区，例如按月划分用户行为数据，可以使得查询只需扫描相关分区，从而提高效率。

然而，在 OLTP 应用中，使用分区则需更为谨慎。这类应用通常不会查询大表中超过 10% 的数据，而是通过索引快速检索少量记录。例如，对于包含 1000 万条记录的表，如果查询使用了辅助索引但未涉及分区键，可能导致性能下降。原本在单个 B + 树中 3 次逻辑 IO 就能完成的操作，在 10 个分区的情况下可能需要 (3+3)\*10 次逻辑 IO（分别访问聚集索引和辅助索引）。

因此，在 OLTP 应用中采用分区表时，务必进行充分的性能测试和优化。

为了便于开发者观察 SQL 查询对分区的利用情况，可以使用`EXPLAIN PARTITIONS`语句与`SELECT`查询结合，从而清晰地看到哪些分区被查询涉及。

* * *
