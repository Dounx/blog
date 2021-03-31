---
layout: post
title: Row size too large
categories: mysql
---

`Mysql2::Error: Row size too large (> 8126). Changing some columns to TEXT or BLOB or using ROW_FORMAT=DYNAMIC or ROW_FORMAT=COMPRESSED may help. In current row format, BLOB prefix of 768 bytes is stored inline.`

线上有一个表在使用的时候报了这个错，其表结构类似于：

```
CREATE TABLE `foo_bars` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(256) NOT NULL,
`foo1` TEXT NOT NULL,
`foo2` TEXT NOT NULL,
`foo3` TEXT NOT NULL,
`foo4` TEXT NOT NULL,
`foo5` TEXT NOT NULL,
`foo6` TEXT NOT NULL,
`foo7` TEXT NOT NULL,
`foo8` TEXT NOT NULL,
`created_at` datetime DEFAULT NULL,
`updated_at` datetime DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

其原因就是类似于 foo* 这样的 TEXT / BLOB 字段过多，导致其行大小超过 InnoDB 的限制（旧代码里有看到过使用序列化并压缩减少空间占用，这次是使用了 serialize 而没加类型限制，导致出现大量的 YAML 类型字段字符串（ActiveSupport::HashWithIndifferentAccess，并还有嵌套）。

InnoDB 对最大行大小有一个限制，该限制略小于数据库页的一半（16k -（页头部 + 页尾部）/ 2）。默认的页大小为 16kb，所以默认的行大小限制大约为 8000 字节（实际测试为 8126）。此限制是由于 InnoDB 在页上存储两行，如果使用COMPRESS ，则可以在页上存储一行。

如果行中有可变长度的列，而整个行的大小超过了此限制，则 InnoDB 会选择可变长度的列进行页外存储。

在种情况下，每个可变长度列的前 768 个字节存储在页本地，其余的存储在页面外部。

另外，此限制只适用于值的字节大小，而不适用于字符大小。这意味着，如果将具有所有多字节字符的 500 个字符串插入 VARCHAR(500) 列，则还将选择该值进行页外存储。当本地字符集转换为 utf8 后，可能会遇到此错误，因为它可能会大大增加字节大小。

如果有 10 个以上的可变长度列，每个可变长度列都超过 768 个字节，即使不算其他固定长度的列，也将有 8448 个字节在本地存储。这超出了限制，因此会出现错误。

其解决方式有以下几种：

1. InnoDB 升级到 Barracuda 格式（服务器端使用 Antelope 格式）。

    Barracuda 对于可变长度类型仅使用 20 字节的指针

    `MySQL > show variables like "%innodb_file%";` 可以用于查询当前格式。

    5.6 版本之前要升级到 Barracuda 格式需要重新编译源码。

2. 限制可变长度的列的大小。

3. 使用 COMPRESS / UNCOMPRESS 函数（会导致读写性能急剧下降）。

    数据越随机，压缩效果越差。
    
    COMPRESS 函数返回二进制数据，应当使用 BLOB 存储。

4. 拆分表格，使得每个表最多包含 10 个可变长度列。

    将可变长度列拆分到几个不同的表中去，使用的时候用 join 再连接起来。

5. 将所有可变长度字段组合到单个 TEXT / BLOB 中，然后在应用中进行拆分。

    需要读取所有的数据才能找到所需要的数据，效率降低。

6. 组合使用以上方式。

## 参考：

1. https://www.percona.com/blog/2011/04/07/innodb-row-size-limitation/
2. https://www.cnblogs.com/chenpingzhao/p/6719258.html
