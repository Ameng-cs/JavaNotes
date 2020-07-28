#	HashMap

## 数据结构

![image](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-8-22/67233764.jpg)
- 数组的表示 :Node[] table
- 单链表的表示方式
```java
class Node{
    hash   记录hash位置
    key     存键
    value   存值
    Node.next 存链表下一结点
}
```
### 重要参数
```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;    
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;   
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30; 
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8; 
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小大小
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table; 
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;   
    // 临界值 当实际大小(容量*填充因子)超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
}
```

### 结点的插入
- 插入点计算出来为int值 
  
    - key,value  ----> 通过key->Object->hashcode
- 插入点的范围不能超过数组的0-15  数组大小的范围
  
    - Hash %16  
- 插入点要尽可能的充分利用数组的每一个位置
    
    - ##### hash算法	
```java
static final int hash(Object key){
    int h;
    return (key==null)?0:(h=key.hashCode()^(h>>>16);
}
//用key.hashCode()的高16位和低16位进行异或运算,这样结果才尽可能不同
```


- #### 插入过程
    ![image](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/put%E6%96%B9%E6%B3%95.png)
    - 判断成员变量table数组是否为空,即是否进行了初始化
    - 若成员table为空或长度为0(未初始化或者因为remove操作使得数组长度为0),则调用resize()方法进行初始化:
        - resize() 初始化:
        ```java
        //此为部分代码
        //设置初始容量为默认值16
        newCap = DEFAULT_INITIAL_CAPACITY;
        //设置初始扩容阈值为 容量*负载因子
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        //设置指定了initialCapacity情况下的新的 threshold
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
        }
        //设置扩容阈值
        threshold = newThr;
        //  创建一个新的初始化好的数组
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        //返回该新数组,初始化完成
        return newTap;
        ```
        - 判断 成员table[] 是否为空,判断是否初始化
        - 若成员table为空则进行初始化 
            - 规定 table大小初始值  newCap = 16
            - 规定 table 进行扩容的阀值 newThr =  16*0.75=12
            - 设置table扩容阈值 threshold  = newThr = 12
            - 新建Node数组 newTab[newCap]
            - 返回该 newTab 初始化完成
    - 若成员table不为空(完成了初始化),插入分四种情况:
        1. 数组原本的位置为空 **直接插入**

        ```java
        if((p=tab[i=(n-1)&hash])==null)
            tab[i]=newNode(hash,key,value,null);
        //判断插入位置为空后直接将新节点插入指定位置
        //n-1&hash 等价于 hash%n  且计算更快
        ```
        2. 数组原来的位置不为空,新插入元素的hash值与key值相等 **直接覆盖**

        ```java
        if(p.hash==hash&&((k=p.key)==key)||(key!=null&&key.equals(k))))
            e=p;
        //若key值和hash值相同,则直接覆盖
        ```
        3. 数组原本的位置不为空,且下面是红黑树结构 **红黑树插入**

        ```java
         else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        //直接进行红黑树的插入操作
        ```
        4. 数组原本的位置不为空,且下面是链表结构 **遍历链表插入尾部**

        ```java
        else {
            //循环用来计算链表的长度
            for (int binCount = 0; ; ++binCount) {
                //如果当前位置没有next结点则直接插入
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    //插入后如果链表长度大于阀值
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        //将链表转为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                //如果hash值和key值相同则直接覆盖
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        ```
    - 插入完成后检查数组容量是否超标,超标则调用resize()进行扩容操作
        - resize() 扩容
            - 检查数据table是否进行初始化
            - 若进行了初始化,则将数组进行扩大,扩为原来的两倍
            ```java
            //判断是否进行了初始化
            if (oldCap > 0) {
                //若数组大小大于等于最大容量
                if (oldCap >= MAXIMUM_CAPACITY) {
                    threshold = Integer.MAX_VALUE;
                    //不进行分扩容
                    return oldTab;
                }
                //若扩大两倍后容量不大于最大容量
                else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                         oldCap >= DEFAULT_INITIAL_CAPACITY)
                    //将数组容量扩大为原来的两倍
                    newThr = oldThr << 1; // double threshold
            }
            ```
            - rehash,扩容后,将原来数组中的元素的hash值进行重新计算,在新数组中重新分配这些元素的位置
                - rehash 红黑树结点
                - rehash 链表结点
                    - 新的结点的hash为原hash值或者原hash值+原容量
                ```java
                 // 把每个bucket都移动到新的buckets中
                 for (int j = 0; j < oldCap; ++j) {
                    Node<K,V> e;
                    if ((e = oldTab[j]) != null) {
                        oldTab[j] = null;
                        //如果是next为null的链表结点
                        if (e.next == null)
                            // rehash后分配新位置
                            newTab[e.hash & (newCap - 1)] = e;
                        // 如果是红黑树结点
                        else if (e instanceof TreeNode)
                           //通过红黑树的split方法分配新的位置 ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                        //如果是next不为null的链表结点   
                        else { // preserve order
                            Node<K,V> loHead = null, loTail = null;
                            Node<K,V> hiHead = null, hiTail = null;
                            Node<K,V> next;
                            do {
                                next = e.next;
                                //原索引
                                if ((e.hash & oldCap) == 0) {
                                    if (loTail == null)
                                        loHead = e;
                                    else
                                        loTail.next = e;
                                    loTail = e;
                                }
                                //原索引+oldCap
                                else {
                                    if (hiTail == null)
                                        hiHead = e;
                                    else
                                        hiTail.next = e;
                                    hiTail = e;
                                }
                            } while ((e = next) != null);
                            // 原索引放到bucket里
                            if (loTail != null) {
                                loTail.next = null;
                                //新hash值为原hash值
                                newTab[j] = loHead;
                            }
                            // 原索引+oldCap放到bucket里
                            if (hiTail != null) {
                                hiTail.next = null;
                                //新hash值为原hash值+原容量
                                newTab[j + oldCap] = hiHead;
                            }
                        }
                    }
                } 
                ```

                - ### 这里讲一下对链表的rehash方

                  ```java
                  HashMap.Node<K,V> loHead = null, loTail = null;
                  HashMap.Node<K,V> hiHead = null, hiTail = null;
                  HashMap.Node<K,V> next;
                  //遍历该桶
                  do {
                      next = e.next;
                      //找出拆分后仍处在同一个桶中的节点
                      if ((e.hash & oldCap) == 0) {
                          if (loTail == null)
                              loHead = e;
                          else
                              loTail.next = e;
                          loTail = e;
                      }
                      else {
                          if (hiTail == null)
                              hiHead = e;
                          else
                              hiTail.next = e;
                          hiTail = e;
                      }
                  } while ((e = next) != null);
                  if (loTail != null) {
                      loTail.next = null;
                      newTab[j] = loHead;
                  }
                  if (hiTail != null) {
                      hiTail.next = null;
                      newTab[j + oldCap] = hiHead;
                  }
                  
                  ```

                  这里定义了4个变量：loHead, loTail ,hiHead , hiTail，这四个变量从字面意思可以看出应该是两个头节点，两个尾节点。那么为什么需要两个链表的头尾节点呢？看一张图就明白了：

                  ![[image]](https://img-blog.csdnimg.cn/20190621145827456.png)

                  这张图中index=2的桶中有四个节点，在未扩容之前，它们的 hash& cap 都等于2。在扩容之后，它们之中2、18还在一起，10、26却换了一个桶。这就是这句代码的含义：选择出扩容后在同一个桶中的节点。

                  ```java
                   if ((e.hash & oldCap) == 0)
                  ```

                  我们这时候的oldCap = 8，2的二进制为：0010，8的二进制为：1000，0010 & 1000 =0000
                  10的二进制为：1010，1010 & 1000 = 1000，
                  18的二进制为：10010, 10010 & 1000 = 0000，
                  26的二进制为：11010，11010 & 1000 = 1000，
                  从与操作后的结果可以看出来，2和18应该在同一个桶中，10和26应该在同一个桶中。

                  所以lo和hi这两个链表的作用就是保存原链表拆分成的两个链表。

                  ```java
                  if ((e.hash & oldCap) == 0) {
                  	//尾节点为空，说明lo链表是空的
                      if (loTail == null)
                          loHead = e;
                      else
                          loTail.next = e;
                      loTail = e;
                  }
                  else {
                      if (hiTail == null)
                          hiHead = e;
                      else
                          hiTail.next = e;
                      hiTail = e;
                  }
                  现在再来看这段代码是不是好理解多了？找到拆分后仍处于同一个桶的节点，将这些节点重新连接起来。
                  ```
                  下面这段代码是将拆分完的链表放进桶里的操作，比较简单，只需要将头节点放进桶里就ok了，newTab[j]和newTab[j + oldCap]分别代表了扩容之后原位置与新位置，就相当于之前那张图中的2和10.

                  ```java
                   if (loTail != null) {
                      loTail.next = null;
                      newTab[j] = loHead;
                   }
                    if (hiTail != null) {
                      hiTail.next = null;
                      newTab[j + oldCap] = hiHead;
                   }
                  ```
                  

#### 参考博客

[https://blog.csdn.net/weixin_41565013/article/details/93190786](https://blog.csdn.net/weixin_41565013/article/details/93190786)

