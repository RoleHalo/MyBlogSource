---
title: Java基础及集合框架
date: 2023-08-06 10:13:10
tags: Java
toc: true
---

# 集合概述

![Java集合框架的体系解构](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/Java集合框架的体系解构.png)

​																								**图1-1 Java集合框架**

Java集合，也叫做容器，主要由两大类接口派生而来

1. `Collection`: 主要存放单一元素。又派生出：`List`、`Set`、`Queue`
2. `Map`：主要存放键值对

`List` `Set` `Queue` `Map`的区别？

1. `List`(解决顺序) ：存储的元素是有序的、可重复的
1. `Set`(注重独一无二) ：存储的数据不可重复
1. `Queue `(实现排队)：按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的
1. `Map`(键值对)：key-value。类似于y=f(x)；y代表value，是无序的、可重复的；x代表key，是无序的、不可重复的。每个键最多映射到一个值



Java三大特性：
1、封装：一种将抽象性函式接口的实现细节部分包装、隐藏起来的方法。
2、继承：子[类继承](https://so.csdn.net/so/search?q=类继承&spm=1001.2101.3001.7020)父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。
3、多态：同一个行为具有多个不同表现形式或形态的能力。

```Java
多态中：当父类(接口)引用指向子类(实现类)的对象时，父类的引用就能直接调用子类中的方法
==> 可以减少代码开发过程中的修改量
```

# `ArrayList`

### 1. 数据结构

- 动态数组

- 寻址公式：
  - Q1：下标为什么从0开始不从1开始？寻址时要多计算一次 i-1      **[`baseAddress+i*dataTypeSize`]**  
- 时间复杂度
  1. 查询：
     1. **随机查询 : O(1)**
     2. 未知查询：未排序:O(n);  排序后: --> 可二分查找--> **O(`logn`)**
  2. 插入、删除
     1. 内存连续  所以为保证数据的连续性会挪动数据 使得插入和删除效率较低
     2. **最好 O(1)  最差O(n)   平均 O(n)**

### 2. 源码

1. 添加数据的逻辑扩容至1.5倍

   ```java
   public boolean add(E e) {
       //判断是否可以容纳e，若能，则直接添加在末尾；若不能，则进行扩容，然后再把e添加在末尾
       ensureCapacityInternal(size + 1);  // Increments modCount!!
       //将e添加到数组末尾
       elementData[size++] = e;
       return true;
       }
   
   // 每次在add()一个元素时，arraylist都需要对这个list的容量进行一个判断。通过ensureCapacityInternal()方法确保当前ArrayList维护的数组具有存储新元素的能力，经过处理之后将元素存储在数组elementData的尾部
   
   private void ensureCapacityInternal(int minCapacity) {
         ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
   }
   
   private static int calculateCapacity(Object[] elementData, int minCapacity) {
     			
           //如果传入的是个！！空数组！！则最小容量取默认容量与minCapacity之间的最大值
     			//无参构造时 this.elementData=DEFAULTCAPACITY_EMPTY_ELEMENTDATA，所以若是无参构造则第一次添加数据时该条件肯定成立
           if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
               return Math.max(DEFAULT_CAPACITY, minCapacity);//JDK1.8中 DEFAULT_CAPACITY=10  所以第一次添加数据时返回10
           }
     
           return minCapacity;//除了第一次添加数据 都直接return minCapacity
       }
       
   
   private void ensureExplicitCapacity(int minCapacity) {
           modCount++; //记录集合被修改的次数
           // 若ArrayList已有的存储能力满足最低存储要求，则返回add直接添加元素；
           //如果最低要求的存储能力> ArrayList已有的存储能力，这就表示ArrayList的存储能力不足，因此需要调用 grow();方法进行扩容
    			 //第一次添加数据: 10 - 0 > 0  则进行grow()  第2-10次添加数据: 2/3../10 - 10 < 0 不进行grow()
     	   // 第11次 : 11 - 10 > 0  进行grow()
           if (minCapacity - elementData.length > 0)
               grow(minCapacity);
       }
   
   
   private void grow(int minCapacity) {
           // 获取elementData数组的内存空间长度
           int oldCapacity = elementData.length;
           // 扩容至原来的1.5倍  (右移1位-->缩小2倍)
           int newCapacity = oldCapacity + (oldCapacity >> 1);  // 第11次添加数据: new = 10+5 =15
           //校验容量是否够
           if (newCapacity - minCapacity < 0)   // 第一次添加数据:0 - 10 < 0;第11次添加数据: 15 - 11 > 0
               newCapacity = minCapacity;
           //若预设值大于默认的最大值，检查是否溢出   所以，若溢出hugeCapacity()中会抛出异常而无法完成扩容
           if (newCapacity - MAX_ARRAY_SIZE > 0)
               newCapacity = hugeCapacity(minCapacity);
           // 调用Arrays.copyOf方法将elementData数组指向新的内存空间
            //并将elementData的数据复制到新的内存空间
           elementData = Arrays.copyOf(elementData, newCapacity);//第一次添加数据:new=10;第11次:new=15
       }
   ```

2. 构造方法

   ```java
   public class ArrayList<E> extends AbstractList<E>
           implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
       private static final long serialVersionUID = 8683452581122892189L;
   
       /**
        * 默认初始容量大小
        */
       private static final int DEFAULT_CAPACITY = 10;
   
       /**
        * 空数组（用于空实例）。
        */
       private static final Object[] EMPTY_ELEMENTDATA = {};
   
       //用于默认大小空实例的共享空数组实例。
       //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
       private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
   
       /**
        * 保存ArrayList数据的数组
        */
       transient Object[] elementData; // non-private to simplify nested class access
   
       /**
        * ArrayList 所包含的元素个数
        */
       private int size;
   
       /**
        * 带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）
        */
       public ArrayList(int initialCapacity) {
           if (initialCapacity > 0) {
               //如果传入的参数大于0，创建initialCapacity大小的数组
               this.elementData = new Object[initialCapacity];
           } else if (initialCapacity == 0) {
               //如果传入的参数等于0，创建空数组
               this.elementData = EMPTY_ELEMENTDATA;//一对'{}’; EMPTY_ELEMENTDATA = {};
           } else {
               //其他情况，抛出异常
               throw new IllegalArgumentException("Illegal Capacity: " +
                       initialCapacity);
           }
       }
   
       /**
        * 默认无参构造函数
        * DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 
        * 当添加第一个元素的时候数组容量(经过第一次扩容)才变成10
        */
       public ArrayList() {
           this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
       }
   
       
     
     
     	/**
        * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
        */
       public ArrayList(Collection<? extends E> c) {
           //将指定集合转换为数组
           elementData = c.toArray();
           //如果elementData数组的长度不为0
           if ((size = elementData.length) != 0) {
               // 如果elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组所以加上下面的语句用于判断）
               if (elementData.getClass() != Object[].class)
                   //将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组
                   elementData = Arrays.copyOf(elementData, size, Object[].class);
           } else {
               // 其他情况，用空数组代替
               this.elementData = EMPTY_ELEMENTDATA;
           }
       }
   
       /**
        * 修改这个ArrayList实例的容量是列表的当前大小。 应用程序可以使用此操作来最小化ArrayList实例的存储。
        */
       public void trimToSize() {
           modCount++; //记录集合被修改的次数
           if (size < elementData.length) {
               elementData = (size == 0)
                       ? EMPTY_ELEMENTDATA
                       : Arrays.copyOf(elementData, size);
           }
       }
   }
   ```

### 3. Q&A

1. 底层原理是什么？
   - 底层是**动态数组**实现
   - **初始容量为0 第一次添加数据后为10**
   - 进行扩容时是扩容至原来容量的**1.5倍**，且每次扩容都需要**拷贝数组**
   - **扩容机制：**
     1. 先判断容量是否够存新数据， 即已使用的长度`size`+1是否存在
     2. 计算数组容量，如果不够则调用`grow()`方法扩容至1.5倍
     3. 确保新增数据有位置存储后，将新数据插入尾部
     4. 返回添加成功布尔值

2. `ArrayList list =  new ArrayList (10) `中`list`扩容了几次？ 0次 只是创建了一个指定大小的数组并没有扩容

   ```Java
   public ArrayList(int initialCapacity) {
           if (initialCapacity > 0) {
               //如果传入的参数大于0，创建initialCapacity大小的数组
               this.elementData = new Object[initialCapacity];
           } else if ...
       }
   ```

3. 数组和list之间的转换

   1. 数组 --> List: 使用JDK中`java.util.Arrays`工具类的`asList()`方法。但底层只涉及**对象的引用**不涉及对象的创建，两者指向同一个地址，所以**再修改数组内容，list会受影响**
   2. List --> 数组：使用List的`toArray()`方法。无参`toArray()`方法返回数组`Object`数组，传入初始化长度的数组对象，返回该对象数组。而底层是通过`copyOf()`进行**数据拷贝**的，所以**再修改list内容，数组不受影响**

4. `ArrayList`和`LinkList`的区别是什么？※

   1. 链表：

      1. **结点**组成；
      2. **非连续、非顺序**的物理存储结构
      3. 时间复杂度
         1. 单向链表：查找/插入/删除：头O(1)，其他O(n)
         2. **双向链表**：因为有first和last指针，所以查找头尾的时间复杂度是O(1)，平均查找时间O(n)，给定节点找前驱O(1)；插入删除头尾O(1)，其他节点增删O(n)，给定节点增删O(1)

   2. 综上从以下几个方面**Answer**

      1. 底层数据结构？

      2. 操作数据效率？

      3. 内存空间占用？数组、内存连续、节省；双向链表、内存不连续、包括两指针故占用更多

      4. 线程安全 ？ 都不安全。  怎么改善？

         - 在方法内使用。局部变量是线程安全的
         - 使用线程安全的`ArrayList`和`LinkedList`    都使用了`synchronized`包装而成

         



# `二叉搜索树（BST)、红黑树、散列表（HashTable）`

### 1. 二叉搜索树（`BST`)

1. 任意一个结点大于左节点（如果有），小于右节点（如果有）；**没有键值相等的节点**
2. 通常情况下是时间复杂度**O(`logn`)**   当为单枝树时性能最差，回退到O(n)

### 2. 红黑树（数据库索引）

1. 节点只有红、黑
2. 根是黑
3. 叶子节点时黑色的空节点
4. 红的子节点都是黑
5. 任一节点到叶子节点的所有路径都包含相同数目的黑节点
6. 以上5点性质在添加或者删除节点时，若不符合则会发生旋转  --> **保证平衡**
7. 查找、 添加、删除：红黑树本质也是BST，所以也是**O(`logn`)**

### 3. 散列表（`HashTable`）

1. `key-value`  支持随机访问  

2. 最主要的是  解决**散列冲突**（又叫哈希冲突、哈希碰撞，即多个key映射到一个value）问题

   1. **拉链法**：每个下标位置称之为**桶**（或 **槽**），每个桶（或 槽）对应一个**链表**，所有散列值相同的都会放到同一个槽位对应的链表中

      - 插入：O(1)  key相同就覆盖，key不同就**头插法**插入元素

      - 查找、删除：平均O(1)，但当某一个槽数据过多时候会退化为链表O(n)  ==> 改造为其他高效数据结构，如红黑树（同时也可以**防止DDos攻击**）O(`logn`)

   2. 再散列法

   3. 建立公共溢出区

# `HashMap`

### 1. `HashMap`的实现原理？

1. 数据结构，底层是数组+链表/红黑树
   1. **jdk1.8之前**是单纯的拉链法(**数组+链表**)；**jdk1.8之后**（**数组+链表/红黑树**），当**链表长度大于阙值（默认为8）**且**数据总量（数组长度）大于等于64**时转换红黑树；
   2. **小于64**时会先进行**扩容resize()**，而不是转换红黑树，以减少检索时间。
   3. 扩容时。红黑树拆分的树的节点**<=6个**时，会退化为链表

2. 添加数据时，散列函数计算key确定下标。根据key计算出`hashcode`，根据`hashcode`计算出`hash`值【提问：为什么还要额外多算一步`hash`值而不直接使用`hashcode`？先思考一下，下边写寻址的part有回答~】，最后通过`hash&(length-1)`（**取模**运算）计算出存储位置

### 2. 源码

1. 扩容 : resize()

   1. ![HashMap扩容流程](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/HashMap扩容流程.png)

   2. 超过最大值就不再扩容，未超过就容量会扩容到原来数组的**2倍**，  即左移一位

   3. 创建时指定了初始容量或**负载因子(默认0.75)**，就会重新进行阙值初始化；或扩容前的容量小于16，就会重新计算resize上限

   4. 只有一个节点，直接计算新元素的位置

      1. 红黑树拆成2棵树，若子树节点个数小于`UNIREEIFTY_THRESHOLD`(默认是6），则转换为链表，大于`UNIREEIFTY_THRESHOLD`则保持子树的树结构   [注：`UNIREEIFTY_THRESHOLD`的值为 当桶上的节点数小于等于这个数时转换为链表]

   5. 综上**扩容机制**如下

      1. 在**添加元素或初始化**的时候都需要调用resize进行扩容，第一次添加数据初始化数组长度为16，以后每次扩容都是达到了扩容阙值（数组长度*0.75）

      2. 每次扩容的时候，都是扩容到之前的2倍   左移一位

      3. 扩容之后会创建新的数组，需要把老数组中的数据**挪到**新数组中

         1. 没有hash冲突的节点，直接使用`e.hash & (newCap - 1)`【相当于**取模新数组长度newCap**的操作】计算新数组的索引位置

            【拓：**因为 位与 运算比取模运算的效率更高，所以采用按位运算**   前提是数组长度必须是2的n次幂】

            ![&和%运算的对应关系](E:\Git_repositories\blog-image\image\Java-CollectionFramework\&和%运算的对应关系.png)

            ​																			**图3-1 &和%运算的对应关系**

         2. 如果是红黑树，走红黑树的逻辑

         3. 如果是链表，则需要遍历链表，可能需要拆分链表。通过`(e.hash & oldCap)`是否为0分为两类：

            1. **等于0**时，则将该头节点放到**新**数组时的索引位置**等于**其在**旧**数组时的**索引位置**，记为低位区链表lo开头-low;
            2. **不等于0**时,则将该头节点放到**新**数组时的索引位置**等于**其在**旧**数组时的**索引位置再加上旧数组长度****，记为高位区链表hi开头high.

         4. > `(e.hash & oldCap)=0`使用原理的推导过程    `oldCap`: 旧数组的长度

            >因为`oldCap`是2的n次幂的整数,其二进制表达为1个1后面跟n个0：1000…0，若想要`e.hash&oldCap`的结果为0，则`e.hash`的二进制形式中与对应`oldCap`的二进制的1的位置一定为0，其他位置的可以随意，这样即可保证结果为0；
            >假设：
            >`oldCap`= 2 ^ 3 =8 = 1000
            >则`e.hash`可以是 0101
            >`e.hash&oldCap` 0000=0
            >
            >*也就是说*：只有`hash<oldCap`时才会为0,即直接计算新索引后放到新索引位置，反之要放到计算的**新索引+旧数组长度**的索引位置下

   6. ```java
      final Node<K,V>[] resize() {
      	//旧数组
      	Node<K,V>[] oldTab = table;
      	//旧数组的容量
      	int oldCap = (oldTab == null) ? 0 : oldTab.length;
      	//旧数组的扩容阈值，注意看，这里取的是当前对象的 threshold 值，下边的第2种情况会用到。
      	int oldThr = threshold;
      	//初始化新数组的容量和阈值，分三种情况讨论。
      	int newCap, newThr = 0;
      	//1.当旧数组的容量大于0时，说明在这之前肯定调用过 resize扩容过一次，才会导致旧容量不为0。
      	//为什么这样说呢，之前我在 tableSizeFor 卖了个关子，需要注意的是，它返回的值是赋给了 threshold 而不是 capacity。
      	//我们在这之前，压根就没有在任何地方看到过，它给 capacity 赋初始值。
      	if (oldCap > 0) {
      		//容量达到了最大值
      		if (oldCap >= MAXIMUM_CAPACITY) {
      			threshold = Integer.MAX_VALUE;
      			return oldTab;
      		}
      		//新数组的容量和阈值都扩大原来的2倍
      		else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
      				 oldCap >= DEFAULT_INITIAL_CAPACITY)
      			newThr = oldThr << 1; // double threshold
      	}
      	//2.到这里，说明 oldCap <= 0，并且 oldThr(threshold) > 0，这就是 map 初始化的时候，第一次调用 resize的情况
      	//而 oldThr的值等于 threshold，此时的 threshold 是通过 tableSizeFor 方法得到的一个2的n次幂的值(我们以16为例)。
      	//因此，需要把 oldThr 的值，也就是 threshold ，赋值给新数组的容量 newCap，以保证数组的容量是2的n次幂。
      	//所以我们可以得出结论，当map第一次 put 元素的时候，就会走到这个分支，把数组的容量设置为正确的值（2的n次幂)
      	//但是，此时 threshold 的值也是2的n次幂，这不对啊，它应该是数组的容量乘以加载因子才对。别着急，这个会在③处理。
      	else if (oldThr > 0) // initial capacity was placed in threshold
      		newCap = oldThr;
      	//3.到这里，说明 oldCap 和 oldThr 都是小于等于0的。也说明我们的map是通过默认无参构造来创建的，
      	//于是，数组的容量和阈值都取默认值就可以了，即 16 和 12。
      	else {               // zero initial threshold signifies using defaults
      		newCap = DEFAULT_INITIAL_CAPACITY;
      		newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
      	}
      	//③ 这里就是处理第2种情况，因为只有这种情况 newThr 才为0，
      	//因此计算 newThr（用 newCap即16 乘以加载因子 0.75，得到 12） ，并把它赋值给 threshold
      	if (newThr == 0) {
      		float ft = (float)newCap * loadFactor;
      		newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
      				  (int)ft : Integer.MAX_VALUE);
      	}
      	//赋予 threshold 正确的值，表示数组下次需要扩容的阈值（此时就把原来的 16 修正为了 12）。
      	threshold = newThr;
      	@SuppressWarnings({"rawtypes","unchecked"})
      		//我们可以发现，在构造函数时，并没有创建数组，在第一次调用put方法，导致resize的时候，才会把数组创建出来。这是为了延迟加载，提高效率。
      		Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
      	table = newTab;
      	//如果原来的数组不为空，那么我们就需要把原来数组中的元素重新分配到新的数组中
      	//如果是第2种情况，由于是第一次调用resize，此时数组肯定是空的，因此也就不需要重新分配元素。
      	if (oldTab != null) {
      		//遍历旧数组
      		for (int j = 0; j < oldCap; ++j) {
      			Node<K,V> e;
      			//取到当前下标的第一个元素，如果存在，则分三种情况重新分配位置
      			if ((e = oldTab[j]) != null) {
      				oldTab[j] = null;
      				//1.如果当前元素的下一个元素为空，则说明此处只有一个元素
      				//则直接用它的hash()值和新数组的容量取模就可以了，得到新的下标位置。
      				if (e.next == null)
      					newTab[e.hash & (newCap - 1)] = e;
      				//2.如果是红黑树结构，则拆分红黑树，必要时有可能退化为链表
      				else if (e instanceof TreeNode)
      					((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
      				//3.到这里说明，这是一个长度大于 1 的普通链表，则需要计算并
      				//判断当前位置的链表是否需要移动到新的位置
      				else { // preserve order
      					// loHead 和 loTail 分别代表链表旧位置的头尾节点
      					Node<K,V> loHead = null, loTail = null;
      					// hiHead 和 hiTail 分别代表链表移动到新位置的头尾节点
      					Node<K,V> hiHead = null, hiTail = null;
      					Node<K,V> next;
      					do {
      						next = e.next;
      						//如果当前元素的hash值和oldCap做与运算为0，则原位置不变
      						if ((e.hash & oldCap) == 0) {
      							if (loTail == null)
      								loHead = e;
      							else
      								loTail.next = e;
      							loTail = e;
      						}
      						//否则，需要移动到新的位置
      						else {
      							if (hiTail == null)
      								hiHead = e;
      							else
      								hiTail.next = e;
      							hiTail = e;
      						}
      					} while ((e = next) != null);
      					//原位置不变的一条链表，数组下标不变
      					if (loTail != null) {
      						loTail.next = null;
      						newTab[j] = loHead;
      					}
      					//移动到新位置的一条链表，数组下标为原下标加上旧数组的容量
      					if (hiTail != null) {
      						hiTail.next = null;
      						newTab[j + oldCap] = hiHead;
      					}
      				}
      			}
      		}
      	}
      	return newTab;
      }
      
      ```

2. put：put流程（参考）

   ![HashMap-put-JDK1.8](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/HashMapput-JDK1.8.png)

   ​																				**图3-3	HashMap-put-JDK1.8**

   

   ![HashMap的put逻辑示意图](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/HashMap的put逻辑示意图.png)

​																								**图3-4  HashMap的put逻辑示意图**

![HashMap扩容流程](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/HashMap扩容流程.png)

​                                                                                	**图3-5  HashMap的扩容流程示意图**

综上 **put**简要流程如下:

			1. 首先计算hash值，找到该元素在数组中的存储下标
			1. 如果数组时空的，则调用resize进行初始化扩容
			1. 如果没有哈希冲突直接存储对应位置上
			4. 如果有冲突
	     			1. 如果时红黑树，走红黑树的添加逻辑
	     			2. 如果时链表，小于8直接放入，若key已存在，就覆盖value；大于8小于64就进行扩容；大于8且大于64，则将这个节点结构转为红黑树；

### 3. hashMap的寻址算法   

1. 寻址算法
   1. 计算对象的`hashCode()`
   2. 调用hash()进行二次哈希，`hashcode`值右移18位再异或运算，让哈希分布更加均匀
   3. 最后`(capacity-1)&hash`得到索引
2. 回答最早提到的问题：为什么还要通过`hashcode`计算`hash`而不直接使用`hashcode`？
   - 作用：为了使得hash值分布更加均匀，减少hash冲突

3. HashMap数组长度到底为什么必须是2的n次幂？
   1. 计算索引的时候效率更好：2的n次幂可以使用**位与**运算代替取模运算
   2. 扩容时重新计算索引效率更高：即使用`(e.hash & oldCap)=0`将原来的各个数据分为两类









# 多线程/并发编程

### 1. 线程介绍

1. **进程**是系统***分配资源***的单位，一个进程可以包含若干线程；而**线程是CPU*调度和执行*的单位**
1. **线程是独立的执行路径**
1. 程序运行时，即使没有自己创建线程，后台也会有多个线程，如主线程，GC线程
1. main()称之为主线程，是**系统的入口**，用于执行整个程序
1. 在一个进程中，多个线程的运行由调度器安排，**调度器与操作系统密切相关，先后顺序无法人为干预**
1. 对**同一份资源**操作时，会存在资源抢夺问题，需要加入**并发控制**
1. 线程会带来**额外的开销**，如CPU调度时间、并发控制时间等
1. 每个线程在自己的工作内交互，**内存控制不当会导致数据不一致**

### 2. 线程创建※

1. 线程创建的三种方式
   - **继承Thread类（就是实现Runnable接口）※**
     1. 继承Thread类
     2. 重写run()方法：方法内则为线程的执行体
     3. 调用start()方法启动线程
     4. 注意：线程开启不一定立即执行，由CPU调度执行；Java是单继承
   - **实现Runnable接口 ※※**
     1. 实现Runnable接口
     2. 重写run()方法
     3. 创建线程Thread对象（将Runnable接口对象作为参数传入Thread对象中），调用start()方法启动线程 ==>**即Thread类静态代理了Runnable**
     4. 优点：避免单继承局限性，灵活方便，方便同一个对象被多个线程同时使用
   - 实现Callable接口（了解即可）
     1. 实现callable接口，***需要返回值类型***
     2. 重写call方法，***需要抛出异常***
     3. 创建目标对象
     4. 创建执行服务：`ExecutorService executorService = Executors.newFixedThreadPool(1);`
     5. 提交执行：`Future<Boolean> future = executorService.submit(testCallable);`
     6. 获取结果：`boolean res = future.get();`
     7. 关闭服务：`executorService.shutdownNow();`
2. **静态代理：**（如结婚找婚庆公司后就只需要关心自己私事，其他额外的交给婚庆公司布置）
   1. 真实对象和代理对象实现 同一个接口
   2. 代理对象代理真实对象
   3. 优点：
      1. 代理对象可以做很多真实对象做不了的事情
      2. 真实对象专注于做自己的事

3. `Lambda`表达式

   1. 避免匿名类定义过多

   2. 使得代码更加简洁

   3. 去掉无意义的代码，留下核心逻辑

   4. **函数式接口**：任何接口，如果只包含唯一一个抽象方法，那么它就是一个函数时接口   ；函数时接口可以使用该表达式调用

   5. ```java
      //为了方便开发从 第2->第6 形式逐步演变简化
      //Lambda无参简化实现函数式接口
      public class TestLambda1 {
          //3. 静态内部类
          static class Like2 implements ILike{
      
              @Override
              public void lambda() {
                  System.out.println("I like lambda2");
              }
          }
          public static void main(String[] args) {
              ILike like = new Like();
              like.lambda();
      
              like = new Like2();
              like.lambda();
      
              //4. 局部内部类
              class Like3 implements ILike{
                  @Override
                  public void lambda() {
                      System.out.println("I like lambda3");
                  }
              }
              like = new Like3();
              like.lambda();
      
              //5. 匿名类
              like = new ILike() {
                  @Override
                  public void lambda() {
                      System.out.println("I like lambda4");
                  }
              };
              like.lambda();
      
              //6. Lambda简化实现函数式接口
              like = () ->{
                  System.out.println("I like lambda5");
              };
              like.lambda();
      
          }
      }
      //1. 定义函数式接口
      interface ILike{
          void lambda();
      }
      
      //2. 定义外部实现类
      class Like implements ILike{
      
          @Override
          public void lambda() {
              System.out.println("I like lambda1");
          }
      }
      ```

   6. ```java
      
      
      //Lambda有参简化实现函数式接口
      public class TestLambda2 {
          public static void main(String[] args) {
      
              //1.Lambda简化形式1
              //单一参数：
              ILove love = (int a) -> {
                  System.out.println("I love you1-1-->"+a);
              };
              love.Love(2);
              //多参数：
              ILove2 love2 = (int a,int b)->{
                  System.out.println("I love you2-1-->"+a+" "+b);
              };
              love2.Love2(2,1);
      
              //2.Lambda简化形式2
              //单一参数：
              love = (a) -> {
                  System.out.println("I love you1-2-->"+a);
              };
              love.Love(3);
              //多参数：
              love2 = (a,b) -> {
                  System.out.println("I love you2-2-->"+a+" "+b);
              };
              love2.Love2(2,2);
      
              //3.Lambda简化形式3
              //单一参数：
              love = a -> {
                  System.out.println("I love you1-3");
              };
              love.Love(3);
              //多参数:无  多参数必须加括号，不能省略
          }
      
      }
      
      //定义函数式接口   单参数
      interface ILove{
          void Love(int a);
      }
      
      //定义函数式接口   多参数
      interface ILove2{
          void Love2(int a,int b);
      }
      ```

### 3. 线程状态

![线程状态](https://gitee.com/RoleHalo/blog-image/raw/master/image/Java-CollectionFramework/ThreadState.png)

​	                                                                                       **图4-1  线程状态示意图**

1. 线程方法
   1. 更改线程优先级：``
   2. 在指定的毫秒数内让当前正在执行的线程休眠：``
   3. 等待该线程终止：``
   4. 暂停当前正在执行的线程对象。并执行其他线程：``
   5. 中断线程(不建议yo)：``
   6. 测试线程是否处于活动状态：``







### 4. 线程同步※

### 5. 线程通信问题

### 6. 高级主题







































































