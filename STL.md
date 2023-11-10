# STL

**整个STL都是左开右闭！**

根据数据在容器中的排列，容器被分为**序列式容器**和**关联式容器**

![各种容器之间的关系](D:\cppLearning\figure\各种容器之间的关系.png)

##关于**vector**值得记住的点有：

* 它维护的是一个**连续的线性空间**，可以**动态改变**，与c++11引入的array是固定大小空间不同。

* 正因为其是连续空间，因此其**iterator**就是**普通指针**，普通指针所具备的功能它都支持（--，++，+=，->.....）。也就是说**`vector<int>::iterator it`其实就相当于`int* it`.**

* 它所维护的成员变量如下图：

  ![vector变量成员](D:\cppLearning\figure\vector变量成员1.png)

  ![vector变量成员2](D:\cppLearning\figure\vector变量成员2.png)

  start 和 finish分别指向的是**已被使用**的空间开始和结尾，(STL遵循左闭右开)，而end_of_storage指向**整块连续空间**包括备用空间的尾端。这也就是**`size()`与`capacity()`的区别**。

* 当用`push_back()`添加新元素，若还有备用空间，直接在备用空间上**构造**元素，若没有，就需要扩充空间（**重新配置（原来的2倍），移动数据，释放原空间**）。一旦空间被重新配置，指向原来vector的迭代器全部失效了，因为原空间都被释放了。

* vector的`erase()`删除一段范围内的元素或者某个位置上的元素，需要注意的是，此函数只是删除掉所在位置的元素值，并**不释放**掉vector所占空间，因此**对capacity无影响**，`clear()`底层就是调用此函数。erase的实现是将后一段元素copy到移除后的位置，然后destroy后面多余的位置。

* `insert()`可以在插入点上插入一个值或者从插入点开始插入n个相同的值，若备用空间不够，另开辟新空间。若够，插入逻辑分两种情况，一种是插入点剩余元素比n多，一种是比n少。（详情看书）

##关于**list**值得记住的点有：

* list是一个带**有头结点**的环状双向链表，因为是链表，不能任意访问元素，因此它的迭代器是**Bidirection Iterator**. 前边的vector是 Random Access Iterator. 它每一个节点设计如下：

  ![list_node](D:\cppLearning\figure\list_node.png)

  迭代器设计如下：

  ![list迭代器](D:\cppLearning\figure\list迭代器.png)

* `push_back()`实际上调用了内置的`insert(position，x)`后者是在position之前构造一个新节点插入。

* `erase(it)`是移除迭代器为it的节点，就是双向链表移除一个节点的操作，`pop_front(),pop_back()，clear()`都是内调了earse().

* list内部提供了一个`transfer(position,first,last)`操作，将first与last之间的元素移到position之前，可以是两个链表之间的transfer也可以是单个链表之间的，因此`splice(),reverse(),merge()`都是调用了此函数

* STL算法中的sort()只接受RandomAccess Iterator, 而list不是这个的迭代器，因此**它没法使用sort()**, 但是它内部函数实现了sort(),排序机制也是快排，再加上swap与merge

##关于**deque**值得记住的点：

* deque是**双向开口**的**连续线性空间**，它与vector的不同点在于，1. deque可以在常数时间内对**头端**进行插入或移除操作。2. deque没有capacity的概念，它是以**分段连续空间**组合而成。如图是它的数据结构:![deque数据结构1](D:\cppLearning\figure\deque数据结构1.png)

  ![deque数据结构1](D:\cppLearning\figure\deuqe数据结构2.png)

  deque使用的是**一小块连续空间map**(不是容器map)作为**中控器**，map中的每一个**node指向一段较大的连续存储空间**，被称为**缓冲区**。缓冲区才是deque存储元素的主体。

* deque也提供的是RandomAccess Iterator，但因它的结构是靠map联系的分段连续空间，因此有关迭代器的一系列操作需要重新定义。迭代器为了保持与map,真正的存储主体联系起来，它的结构就不在是指针，而是一个结构体。![deuqe迭代器结构](D:\cppLearning\figure\deuqe迭代器结构.png)

  迭代器起作用的示意图是这样的：![deque迭代器示意图](D:\cppLearning\figure\deque迭代器示意图.png)

  分为四个部分，**cur**指向的是当前缓冲区中现行元素（start Iterator中指向的是整个deque的第一个元素，finish Iterator中指向最后一个元素的下一个位置，左开右闭），**first**指向当前缓冲区头，last指向尾，**node**指向在map中的node是哪一个。

  这里需要注意，**map在刚开始分配node节点个数的时候最少是8个**，而且，会**将node尽可能分配在整个map中间位置**，方便之后扩展map这块中控器。

  迭代器中比较重要的操作是`set_node(map_pointer new_node)`跳转缓冲区。

* deuqe定义两个专属空间配置器，一个是**data_allocator**, 一个是**map_allocator**。

  当只有push前端时，map中已无多余的前面node，要重新配置一块map区域，然后将原来的map**拷贝**过去，释放掉原来的，再进行赋值。push后端同理，只不过SGI STL是左开右闭，后端Iterator指向的是最后一个元素的下一个，因此，当末尾空余空间没有的时候，push就得重新分配map.

* 当使用`find()`完成搜索返回迭代器，结构仍然是deque的迭代器结构，具有上述四个部分。

* `clear()`清除整个deque时，会将deque回复到最初状态，**保留一个缓冲区**，有多个缓冲区在，保留头缓冲区，本身就有一个，只清除元素。

* `erase()`清除元素或者区间，都会计算清除元素或区间前后剩余元素多少，前面剩余元素少，则向后移动剩余元素（**覆盖**），最后清除前面的冗余元素。反之，后面剩余元素少，会向前移动。

  `insert()`做得事类似，先计算，后覆盖。

##关于**stack 与 queue**值得记住的点：

* deque是双向开口的容器，即两端都可以进行插入、删除。因此对于stack、queue限制了某端的出入问题，都是**以deque来实现**。stack是deque封闭了前端，queue封闭了前端的入与后端的出。
* 像这样修改了某个容器的接口，一般称为**adapter(配接器)**，不归类为container，而是**container adapter**。
* stack与queue均不支持遍历元素，因此**没有迭代器**。

##关于**priority_queue**值得记住的点：

* priority_queue的底层实现是**binary max heap(大顶堆)**。而首先我们先提及关于binary max heap需要记住的点。

* 大顶堆的底层实现就是由**vector实现的complete binary tree(完全二叉树)**，因此大顶堆的上溯(percolate up)、下溯(percolate down)很容易实现。**当前节点的下标为i，其左孩子下标就为2i，右孩子为2i+1,其父节点为i/2.** 所以实现上下溯进行元素调换位置时很方便。

* `push_heap()`首先将要push的元素**加入到vector的最后一个位置**，**然后进行调整**。`pop_heap()`就是删除根节点，也就是整个vector中最大的节点.删除后，为了维持完全二叉树的性质，需要把最后一层最右边的节点进行调整。具体实现就是，将最后一个节点放到根节点，然后一层层调整下去（比较左右节点，大的往上放）。

  需要注意的是，在`pop_heap()`中，根**节点并没有从vector中删除掉，而是放在了最后一个位置**，如果需要真正移除它就需要使用vector的**pop_back()**。

  也正因此，当一直对一个max-heap使用pop_heap()时，当pop到最后一个元素，整个heap（底层是vector）就会变成**一个升序vector**，也就是`sort_heap()`的底层实现。**当然sort过后，heap就失去了原有的数据之间的关系。**

* 有一段已有数据构建大顶堆，我们要从**最后一个非叶子节点的位置**开始遍历到根节点，也就是从（n/2）-1处，一步步调整至0.使用**下溯调整**。

*  priority_queue是大顶堆的实现，但在队列内部并不是一个严格的降序数组，我们每次输出top后pop，内部将其继续自动调整为大顶堆。且push,pop等操作都是直接调用heap的操作。

* 它也没有迭代器，也因为是heap的实现因此被归类为container adapter

## 关于RB-tree值得记住的点：

* 提起了AVL tree，也就是平衡二叉搜索树。提到了一个之前没有注意到的点。当插入一个节点，为了维护平衡而需要调整时，只需要调整插入点到根节点之间平衡被破坏最深的那个节点。且假设这个最深点是X:

  ![AVLtree](D:\cppLearning\figure\AVLtree.png)

  外侧插入用单旋转即可，内侧插入用双旋转操作。

* RB-tree的调整操作即为复杂，没看明白，但觉得面试应该不会问此。因此直接看了其数据结构。

* RB-tree的节点和迭代器都是两层设计，即：![RB-tree节点与迭代器](D:\cppLearning\figure\RB-tree节点与迭代器.png)

  **\_\_rb_tree_node继承于\_\_rb\_tree\_node\_base, \_\_rb\_tree\_iterator继承于\_\_rb\_tree\_base\_iterator.**

* RB-tree的一个实现技巧，是创造了一个**header节点**，**与根节点互为父节点**。且header节点的左右孩子指向的是整棵树中最小值和最大值。无论怎样设计，都是为了方便在插入或者删除节点时，方便RB-tree继续维护其独有特性。我觉得在学习过程中，要时刻记得，RB-tree是set，map等关联性容器(配接器？)的底层实现，关于这些容器的方法就是调用RB-tree的接口加以改变。

* RB-tree是一种平衡二叉树，AVL-tree也是一种平衡二叉树，那为什么前者可以允许节点出现相同值，其他的包括AVL都不允许呢？

  因为，红黑树的平衡并不是仅仅依赖节点的值来维护，还基于节点的颜色和路径中黑色节点的数量来维护，因此可以使得出现相同值。

## 关于set与map值得记住的点：

* RB_tree的排序就是key_compare，但关于set它的**key就相当于value**。而正因为value就是key，set不允许有重复元素出现。

  因此，set的insert调用的是RB_tree的**insert_unique()**

  也因此，set的元素值遵循一定排序规则，**不能使用迭代器改变其值**(当然不能更改key值，要想实现这样效果，删除掉原来的，插入一个你想要的)，set的Iterator**继承**的是RB_tree的**const_iterator**.

* 面对**关联式容器**，使用其内部的find函数会比算法库中的find更有效率。

* map的底层实现还是RB-tree, 不过与set不同的是，map是以**键值对**作为元素，因此key值进行排序，key值不可以被修改。但**value值可以修改**，不影响其排序，因此**继承的Iterator只是普通的Iterator**。

## 关于multiset与multimap值得记住的点：

* 这两个容器的操作与set和map的操作**一模一样**，只是在**创造函数和插入函数**中，使用的是RB-tree的**insert_equal()**,**允许key值重复**，而且RB-tree一般将重复的值插入到节点的右侧。

## 关于hashtable值得记住的点：

* 映射函数就是散列函数(hash function)，不同的元素映射到相同位置产生碰撞问题(collision)，解决方法是**线性探测、二次探测、开链**。

  线性探测就是若映射位置已有元素，则向下一个位置探索，到末尾了再从头开始。(若此时table真的无位置，如何处理？其实并不会出现这样的情况，hashtable会有一个**负载系数**，当超过某个值后，会重新分配空间，重新计算各个元素位置)

  二次探测就是映射函数发生变化，线性每次是**+1**，而二次是**+$i^2$**.

  开链就是链地址法，每个槽子存储的一个链表，碰撞了直接插在链表里，但此刻**负载系数会大于1**.

* **hashtable槽位是质数并不是硬性要求**，只是设置为质数会减少碰撞，从而提高性能，但不能完全免除。

* hashtable_node的实现如下：

  ```c++
  template <class Vlaue>
  struct __hashtable_node
  {
    __hashtable_node* next;
    Vlaue val;
  }
  ```

  可以看到 ，SGI STL的hashtable实现并没有采用内置的list容器或者slist,而是**自己维护了一个node节点**，而至于槽位(SGI STL中称为**bucket**)则是**采用vector实现**，方便扩容。

* 其迭代器需要注意的一点是重载了++运算符，因为需要判断是否跳到下一个bucket中去。

  其**迭代器没有后退操作**，是一个单向的链表。

* 虽然链地址法不要求表格大小为质数，但是SGI STL中设定了一个含有**28个质数的数组**(每一个相差大约**2倍**)，每当需要申请的时候，会找到**最接近n并大于n的那个数，作为vector的大小**

* **hashtable的元素是键值对，既然有key那么不允许存在相同键值的元素，键的唯一性**。因此，它的默认插入是**insert_unique**, 也可以强行插入重复key值，但是必须采用insert_equal()

* 表格重建，**reseize()**. 比较奇特，SGI STL规定当表格中的**总元素数超过bucket的数量后**，就判断需要**重建**表格(这里是总元素个数，也就是每一个bucket的元素)

* hashtable的hash function映射实现，是执行了**bkt_num()**，里面调用了hash function。是因为有些元素无法直接拿来进行模运算，需要进行**预处理**。

* hashtable的clear()会**清除掉所有bucket里的节点**，但是**bucket所占的空间**并不会被清除。

* stl中定义的哈希函数只能处理**int short long 以及char和char*类型**。处理float double string等需要自定义hash function

## 关于hash_set与hash_map值得记住的点：

* hash_set与hash_map都是以hash_table为底层实现，因此大多数操作都是直接调用的其接口。
* set与hash_set,,map与hash_map的区别就在于前者因为底层是RB-tree是自动排序的，后者实现是hashtable不会排序。

## 关于hash_multiset与hash_multimap值得记住的点：

* 这俩与之前提到的multi_set和multi_map唯一的不同就在于元素的是否有序。含multi的都是insert_equal()插入，允许key值重复。

***

**以上都是容器值得注意的点，接下来记一些关于STL空间分配与迭代器构造的点**

***

## 空间配置器：

* STL标准的空间配置器是std::allocator，但我们研究的版本SGI STL没有使用这个，使用的是**std::alloc**。

* ```cpp
  class Foo {...}
  Foo* foo=new Foo;
  delete foo;
  ```

  这里的new 和delete都包含两段操作，第一段是配置内存，第二段是调用构造函数。第一段是调用析构函数，第二段是释放内存。

* STL 将其也归类为，**内存配置与释放**，**对象构造与析构**。前者由**alloc:allocate(), alloc::deallocate()**. 后者由**::construct(),::destory()**负责。

* STL采用的是**二级配置器**，**第一级处理大于128bytes**时，**第二级处理小于128bytes**的配置请求。

  **第一级直接使用malloc, free,realloc**而**第二级采用内存池管理**，就是维护了**16个free-list**,每一个链表里存储每一个内存碎片的位置，16个节点分别管理着8bytes、16bytes、24bytes....128bytes。每次遇到小额空间配置请求，二级配置都会将其**调整至8的倍数**，然后找一个。

* 所以allocate()与deallocate()在配置和释放时都会先**判断是否超过了128bytes**,**调用不同的配置器**。

## 迭代器：

* 迭代器特性萃取，主要作用是在编译期间获取迭代器所指向元素的类型信息。

  然而，不同的容器和元素类型可能需要不同的处理方式。例如，对于原生类型（如int、char等），我们可以直接使用内存拷贝函数（如memcpy）来进行元素的复制但是，对于一些复杂的类类型，我们可能需要调用其拷贝构造函数来进行元素的复制

  这就是迭代器特性萃取的作用所在。通过迭代器特性萃取，我们可以在编译期间获取迭代器所指向元素的类型信息，然后根据这些信息来选择最合适的处理方式


* 迭代器类型：![迭代器类型](D:\cppLearning\figure\迭代器类型.png)

  迭代器之间的关系：

  ![迭代器之间的关系](D:\cppLearning\figure\迭代器之间的关系.png)

* 迭代器的类型成员：

  ![迭代器类型成员](D:\cppLearning\figure\迭代器类型成员.png)

  1. [`iterator_category`：迭代器的类别，如输入迭代器、输出迭代器、前向迭代器、双向迭代器和随机访问迭代器](https://zhuanlan.zhihu.com/p/352606819)[1](https://zhuanlan.zhihu.com/p/352606819)。
  2. [`value_type`：迭代器所指向元素的类型](https://zhuanlan.zhihu.com/p/352606819)[1](https://zhuanlan.zhihu.com/p/352606819)。
  3. [`difference_type`：两个迭代器之间的距离的类型](https://zhuanlan.zhihu.com/p/352606819)[1](https://zhuanlan.zhihu.com/p/352606819)。
  4. [`pointer`：指向元素的指针类型](https://zhuanlan.zhihu.com/p/352606819)[1](https://zhuanlan.zhihu.com/p/352606819)。
  5. [`reference`：元素的引用类型](https://zhuanlan.zhihu.com/p/352606819)[1](https://zhuanlan.zhihu.com/p/352606819)。

***

## 算法：

* accumulate(iterator a, iterator b, T c)将[a,b)内所有总和加到c上。必须提供初始值c, 可以是0；

* max,min提供两个版本，一个是正常，另一个可以自己写比较规则作为参数(仿函数那样)

* count与count_if都需要将容器顺序遍历一遍。

* find与find_if也是

* reverse只支持迭代器类型是双向或者随机定位的。

* sort要求参数迭代器都是randomaccess类型的。关系式容器都使用它自己的，序列式容器中stack、queue、priority_queue都限制了元素出入口方向，无法排序，list的迭代器是属于bidirectioinal类型，只能使用它自己内置的sort(使用的是快排)。所以整个STL使用算法库里的sort只有vector和deque。

  ​