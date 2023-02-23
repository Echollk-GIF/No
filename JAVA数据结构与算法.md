[TOC]

# 数据结构分类

数据结构包括：线性结构和非线性结构。

**线性结构** 

（1）线性结构作为最常用的数据结构，其特点是**数据元素之间存在一对一**的线性关系 

（2）线性结构有两种不同的存储结构，即顺序存储结构(数组)和链式存储结构(链表)。顺序存储的线性表称为顺序表，顺序表中的**存储元素是连续**的 

（3） 链式存储的线性表称为链表，链表中的**存储元素不一定是连续的**，元素节点中存放数据元素以及相邻元素的地址信息 

（4） 线性结构常见的有：**数组、队列、链表和栈**.

**非线性结构**

非线性结构包括：二维数组，多维数组，广义表，**树结构，图结构**

# 稀疏数组

当一个数组中大部分元素为０，或者为同一个值的数组时，可以使用稀疏数组来保存该数组。

稀疏数组的处理方法是: 

（1）记录数组**一共有几行几列，有多少个不同**的值 

（2）把具有不同值的元素的行列及值记录在一个小规模的数组中，从而**缩小程序**的规模

![截屏2023-02-20 17.38.57](JAVA%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95/%E6%88%AA%E5%B1%8F2023-02-20%2017.38.57.png)

**二维数组转稀疏数组的思路**

（1）遍历原始的二维数组，得到有效数据的个数 sum

（2）根据sum 就可以创建 稀疏数组 sparseArr   int[sum + 1] [3]3. 将二维数组的有效数据数据存入到 稀疏数组

**稀疏数组转原始的二维数组的思路**

（1）先读取稀疏数组的第一行，根据第一行的数据，创建原始的二维数组，再读取稀疏数组后几行的数据，并赋给原始的二维数组即可.

```java
package com.SparseArray;

public class SparceArray {
  public static void main(String[] args) {
    //创建一个原始的二维数组棋盘11*11
    //0表示没有子 1表示黑子 2表示白子
    int[][] chessArr1 = new int[11][11];
    chessArr1[1][2] = 1;
    chessArr1[2][3] = 2;
    //输出原始的二维数组
    System.out.println("原始的二维数组~~~");
    for (int i = 0; i < chessArr1.length; i++) {
      for (int j = 0; j < chessArr1[i].length; j++) {
        System.out.printf(String.valueOf(chessArr1[i][j]));
      }
      System.out.println();
    }
    //将二维数组转化为稀疏数组
    //1.先遍历二维数组得到非0数据的个数
    int sum = 0;
    for (int i = 0; i < chessArr1.length; i++) {
      for (int j = 0; j < chessArr1[i].length; j++) {
        if (chessArr1[i][j] != 0) {
          sum++;
        }
      }
    }
    //2.创建对应的稀疏数组
    int[][] sparseArr = new int[sum + 1][3];
    //给稀疏数组赋值
    sparseArr[0][0] = chessArr1.length;
    sparseArr[0][1] = chessArr1[0].length;
    sparseArr[0][2] = sum;

    //遍历二维数组，将非0的值存放到稀疏数组中
    int count = 0;//count用于记录是第几个非0数据
    for (int i = 0; i < chessArr1.length; i++) {
      for (int j = 0; j < chessArr1[i].length; j++) {
        if (chessArr1[i][j] != 0) {
          count++;
          sparseArr[count][0] = i;
          sparseArr[count][1] = j;
          sparseArr[count][2] = chessArr1[i][j];
        }
      }
    }
    //输出稀疏数组的形式
    System.out.println("得到的稀疏数组为如下形式");
    for (int i = 0; i < sparseArr.length; i++) {
      System.out.printf("%d\t%d\t%d", sparseArr[i][0], sparseArr[i][1], sparseArr[i][2]);
      System.out.println();
    }

    //将稀疏数组->恢复成原始的二维数组
    //1.先读取稀疏数组的第一行，根据第一行的数据，创建原始的二维数组
    int[][] chessArr2 = new int[sparseArr[0][0]][sparseArr[0][1]];
    //2.读取稀疏数组后几行的数据，并赋给原始的二维数组即可.
    for (int i = 1; i < sparseArr.length; i++) {
      chessArr2[sparseArr[i][0]][sparseArr[i][1]] = sparseArr[i][2];
    }
    //输出恢复后的二维数组
    System.out.println("恢复后的二维数组");
    for (int i = 0; i < chessArr2.length; i++) {
      for (int j = 0; j < chessArr2[i].length; j++) {
        System.out.printf(String.valueOf(chessArr2[i][j]));
      }
      System.out.println();
    }
  }
}

```

# 队列

队列是一个**有序列表**，可以用**数组**或是**链表**来实现。 

遵循**先入先出**的原则。即：**先存入队列的数据，要先取出。后存入的要后取**出 

## 数组模拟队列

队列本身是有序列表，若使用数组的结构来存储队列的数据，maxSize 是该队列的最大容量。 

因为队列的输出、输入是分别从前后端来处理，因此需要两个变量 front 及 rear 分别记录队列前后端的下标， front 会随着数据输出而改变，而 rear 则是随着数据输入而改变，

当我们将数据存入队列时称为”addQueue”，addQueue 的处理需要有两个步骤：思路分析 

（2）将尾指针往后移：rear+1 , 当 front == rear 【空】 

（2）若尾指针 rear 小于队列的最大下标 maxSize-1，则将数据存入 rear 所指的数组元素中，否则无法存入数据。 rear == maxSize - 1[队列满]

```java
package com.queue;

import java.util.Scanner;

public class ArrayQueueDemo {
  public static void main(String[] args) {
  }
}

//使用数组模拟队列-编写一个ArrayQueue类
class ArrayQueue{
  private int maxSize;//表示数组的最大容量
  private int front;//指向队列头
  private int rear;//指向队列尾
  private int[] arr;//用于存放数据

  //创建队列的构造器
  public ArrayQueue(int arrMaxSize){
    this.maxSize = arrMaxSize;
    this.arr = new int[this.maxSize];
    this.front = -1;//指向队列头部第一个数据的前一个位置
    this.rear = -1;//指向队列尾部（即就是队列最后一个数据）
  }

  //判断队列是否满
  public boolean isFull(){
    return this.rear == this.maxSize-1;
  }
  //判断队列是否为空
  public boolean isEmpty(){
    return this.front == this.rear;
  }
  //添加数据到队列
  public void addQueue(int n){
    //判断队列是否满
    if(isFull()){
      System.out.println("队列满，不能加入数据");
      return;
    }
    this.rear++;
    this.arr[rear] = n;
  }
  //获取队列的数据，出队列
  public int getQueue(){
    if(isEmpty()){
      System.out.println("队列已空");
      throw new RuntimeException("队列为空,不能取数据");
    }
    this.front++;
    return this.arr[this.front];
  }
  //输出所有数据
  public void showQueue(){
    if(isEmpty()){
      System.out.println("队列为空，没有数据");
    }
    for(int i= 0;i<this.arr.length;i++){
      System.out.printf("arr[%d]=%d",i,this.arr[i]);
    }
  }
  //显示队列头数据（注意不是取出数据）
  public int headQueue(){
    if(isEmpty()){
      throw new RuntimeException("队列空，无数据");
    }
    return this.arr[this.front+1];
  }
}

```

问题分析并优化 

（1）目前数组使用一次就不能用， 没有达到复用的效果 

（2）将这个数组使用算法，改进成一个**环形的队列** 取模：%

## 数组模拟环形队列

对前面的数组模拟队列的优化，充分利用数组. 因此将数组看做是一个环形的。(通过**取模的方式来实现**即可)

思路如下:

（1）front 变量的含义做一个调整： front 就指向队列的第一个元素, 也就是说 arr[front] 就是队列的第一个元素  front 的初始值 = 0

（2）rear 变量的含义做一个调整：rear 指向队列的最后一个元素的后一个位置. 因为希望空出一个空间做为约定. rear 的初始值 = 0

（3）当队列满时，条件是  (rear  + 1) % maxSize == front 【满】

（4）对队列为空的条件， rear == front 空

（5）当我们这样分析， 队列中有效的数据的个数   (rear + maxSize - front) % maxSize   // rear = 1 front = 0 

（6）我们就可以在原来的队列上修改得到，一个环形队列

```java
package com.queue;

public class CircleArrayQueueDemo {
  public static void main(String[] args) {

  }
}

class CircleArray {
  private int maxSize;
  private int front;
  private int rear;
  private int[] arr;

  public CircleArray(int maxSize) {
    this.maxSize = maxSize;
    this.arr = new int[maxSize];
  }

  public boolean isFull() {
    return (rear + 1) % maxSize == front;
  }

  public boolean isEmpty() {
    return rear == front;
  }

  public void addQueue(int n) {
    if (isFull()) {
      return;
    }
    arr[rear] = n;
    rear = (rear + 1) % maxSize;
  }

  public int getQueue() {
    if (isEmpty()) {
      throw new RuntimeException("队列为空");
    }
    int value = arr[front];
    front = (front + 1) % maxSize;
    return value;
  }

  public void showQueue() {
    if (isEmpty()) {
      return;
    }
    //从front开始遍历
    for (int i = front; i < front + size(); i++) {
      System.out.printf("arr[%d]=%d\n", i % maxSize, arr[i % maxSize]);
    }
  }

  //求出当前队列有效数据的个数
  public int size() {
    return (rear - front + maxSize) % maxSize;
  }
  //显示头元素
  public int headQueue(){
    if(isEmpty()){
      throw new RuntimeException("空队列");
    }
    return arr[front];
  }
}

```

# 链表

（1）链表是以节点的方式来存储,**是链式存储** 

（2）每个节点包含 data 域， next 域：指向下一个节点. 

（3）**链表的各个节点不一定是连续存储**. 

（4）链表分**带头节点的链表**和**没有头节点的链表**，根据实际的需求来确定

## 单向链表

直接插入到链表尾部

```java
package com.linkedlist;

public class SingleLinkedListDemo {
  public static void main(String[] args) {
    //创建节点
    HeroNode hero1 = new HeroNode(1,"宋江");
    HeroNode hero2 = new HeroNode(2,"卢俊义");
    HeroNode hero3 = new HeroNode(3,"吴用");
    //创建单链表
    SingleLinkedList singleLinkedList = new SingleLinkedList();
    singleLinkedList.add(hero1);
    singleLinkedList.add(hero2);
    singleLinkedList.add(hero3);
    //显示单链表
    singleLinkedList.list();
  }
}

//定义SingleLinkedList
class SingleLinkedList{
  //先初始化一个头节点
  private HeroNode head = new HeroNode(0,"");

  //添加节点到单向列表
  //思路：当不考虑编号顺序时
  //1.找到当前链表的最后节点
  //2.将最后这个节点的next指向新的节点
  public void add(HeroNode heroNode){
    //因为head节点不能动，所以我们需要一个辅助遍历temp
    HeroNode temp = head;
    while(temp.next!=null){
      temp = temp.next;
    }
    temp.next = heroNode;
  }
  //显示链表（遍历
  public void list(){
    //判断链表是否为空
    if(head.next == null){
      System.out.println("链表为空");
      return;
    }
    //因为head节点不能动，所以我们需要一个辅助遍历temp
    HeroNode temp = head.next;
    while(temp != null){
      System.out.println(temp);
      temp = temp.next;
    }
  }
}

//定义HeroNode，每个HeroNode对象就是一个节点
class HeroNode{
  public int no;
  public String name;
  public HeroNode next;//指向下一个节点

  //构造器
  public HeroNode(int no,String name){
    this.no = no;
    this.name = name;
  }

  //为了显示方便，重写toString
  @Override
  public String toString() {
    return "HeroNode{" +
      "no=" + no +
      ", name='" + name + '\'' +
      '}';
  }
}

```

根据排名(id)插入到指定位置，如果有这个排名则添加失败并提示

- 首先找到新添加的节点的位置, 是通过辅助变量(指针), 通过遍历来搞定
- 新的节点.next = temp.next
- 将temp.next = 新的节点