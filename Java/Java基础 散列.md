[toc]
## 1. 引言
散列非常高效，使用散列将耗费O(1)时间来查找、插入以及删除一个元素。
## 2. 散列
散列使用一个散列函数，将一个键映射到一个索引上。先回顾一下映射表(map)又称字典(dictionary)、散列表(hash table)或者关联数组(associate array)，是一种使用散列实现的数据结构，用来存取条目的容器对象。
Java合集框架定义了java.util.Map接口来对映射表建模，三个具体实现类为java.util.HashMap(散列实现)、java.util.LinkedHashMap(使用LinkedList)、java.util.TreeMap(使用红黑书)。
存储了值的数组称为散列表(hash table)，将键映射到散列表中的索引上的函数称为散列函数(hash function)。散列(hashing)是一种无需进行搜素，即可通过从键得到的索引来获取值的技术。
## 3. 散列函数和散列码
典型的散列函数首先将搜索键转换成一个称为散列码的整数值，然后将散列码压缩为散列表中的索引。