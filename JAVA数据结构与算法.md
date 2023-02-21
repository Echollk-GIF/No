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