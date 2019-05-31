[toc]
## 1. 引言
散列非常高效，使用散列将耗费O(1)时间来查找、插入以及删除一个元素。
## 2. 散列
散列使用一个散列函数，将一个键映射到一个索引上。先回顾一下映射表(map)又称字典(dictionary)、散列表(hash table)或者关联数组(associate array)，是一种使用散列实现的数据结构，用来存取条目的容器对象。
Java合集框架定义了java.util.Map接口来对映射表建模，三个具体实现类为java.util.HashMap(散列实现)、java.util.LinkedHashMap(使用LinkedList)、java.util.TreeMap(使用红黑树)。
存储了值的数组称为散列表(hash table)，将键映射到散列表中的索引上的函数称为散列函数(hash function)。散列(hashing)是一种无需进行搜素，即可通过从键得到的索引来获取值的技术。
## 3. 散列函数和散列码
典型的散列函数首先将搜索键转换成一个称为散列码的整数值，然后将散列码压缩为散列表中的索引。
### 3.1 基本数据类型的散列码
- byte,short,int,char将被转换成int
- float将使用Float.floatToIntBits(key)作为散列码，返回一个int值，该值的比特表示与浮点数f的比特表示相同。
- long类型需要拆为前后两个32比特，并执行异或操作将两部分结合（称为折叠folding)
```
int hashCode = (int)(key ^ (key >> 32));  // ^是比特异或操作， >>按位右移操作符，左边补零
```
- double类型首先使用Double.doubleToLongBits方法转化为long，再进行折叠
```
long bits = Double.doubleToLongBits(key);
int hashCode = (int)(bits ^ (bits >> 32));
```
### 3.2 字符串类型的散列码
字符串的散列码会根据字符的unicode和位置得出如下的多项式散列码
$$(...((s_0\times b + s_1)b+s_2)+...+s_{n-2})b+s_{n-1}$$
Si为s.charAt(i)，实验显示，b的最好取值为31，33，37，39和41。String类中的hashCode方法b的值为31。
### 3.3 压缩散列码
假设散列表的索引处于0——(N-1)之间，通常做法为
```
h(hashCode) = hashCode % N;
```
理想状况下N为一个素数，但是选择一个大素数很耗时，在java.util.HashMap中N设置为一个2的幂值，此时
```
h(hashCode) = hashCode % N;
两者结果是一样的
h(hashCode) = hashCode & (N - 1);
```
但是&是位操作要比%操作符快的多。
## 4. 使用开放地址法处理冲突
> 当两个键映射到散列表中的同一个索引上会发生冲突，有两种方法处理冲突：开放地址法和链地址法。

开放地址法(open addressing)是在冲突发生时，在散列表中找到一个开放位置的过程，开放地址法有几个变体：线性探测、二次探测和再哈希法。
### 4.1 线性探测(linear)
如果在hashTable[k % N]发生冲突，那么就检查hashTable[(k + 1) % N]以此类推，散列表中每个单元具有三个可能状态：被占用、标记的或空的。
线性探测的问题就是容易导致散列中连续的单元组（簇cluster）被占用影响CRUD的性能。

![线性探测](https://github.com/little-motor/uml/raw/master/Java/dataStructure/%E9%A1%BA%E5%BA%8F%E6%8E%A2%E6%B5%8B%E6%B3%95.png)
### 4.2 二次探测法(quadratic probing)
可以优化线性探测遇到的问题，他可以尽量避免线性成簇的问题，其索引为(k+j^2)%N，其中j>=0。但是反过来线性探测法可以保证只要表不是满的，一个可用的单元总是可以被找到用于插入新的元素，然而，二次探测法不能保证这个。

![二次探测法](https://raw.githubusercontent.com/little-motor/uml/master/Java/dataStructure/%E4%BA%8C%E6%AC%A1%E6%8E%A2%E6%B5%8B%E6%B3%95.png)

### 4.3 再哈希法
再哈希法在键上应用一个二次散列函数h`(key)来确定增量，从而避免成簇问题，其索引为(k+j*h'(key))%N，其中j>=0，以此类推。
# 5. 使用链地址法处理冲突
链地址法将具有相同的散列索引的条目都放在同一个位置，而不是寻找一个新的位置，链地址法的每一个位置使用一个桶来放置多个条目。我们可以使用list来实现一个桶，或者在桶的长度较大时使用树的结构来实现。

![链地址法处理冲突](https://raw.githubusercontent.com/little-motor/uml/master/Java/dataStructure/%E9%93%BE%E5%9C%B0%E5%9D%80%E6%B3%95.png)
# 6. 装填因子和再散列
装填因子(load factor)衡量一个散列表有多满，如果装填因子溢出，则增加散列表的大小，并重新装载条目到一个新的更大的散列表中，这称为再散列。
装填因子λ是元素数目和散列表大小的比例，如果散列表为空则λ为0，对于开放地址法λ介于0到1之间，对于链地址法，λ可能为任意值。当λ增加时，冲突的可能性增大，研究表明，对于开放地址法而言，需要维持装填因子在0.5以下，而对于链地址法而言，维持在0.9以下。在HashMap中的装填因子为0.75.