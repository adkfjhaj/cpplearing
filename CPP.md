# CPP-STL

**整个STL都是左开右闭！**

根据数据在容器中的排列，容器被分为序列式容器和关联式容器

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

* deque是**双向开口**的**连续线性空间**，它与vector的不同点在于，1. deque可以在常数时间内对头端进行插入或移除操作。2. deque没有capacity的概念，它是以**分段连续空间**组合而成。如图是它的数据结构:![deque数据结构1](D:\cppLearning\figure\deque数据结构1.png)

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