---
title: "排序算法(比较排序)"
categories:
  - Algorithm
tags:
  - Algorithm
  - Java
  - Sort
toc: true
toc_sticky: true
---

# Sorting Algorithm - Comparison Sorts

> 比较排序 ([Comparison Sorts](https://www.wikiwand.com/en/Comparison_sort)) 是排序算法的一种，通过一个抽象的内容比较操作（通常是“小于或等于”操作或者一个 [three-way comparison](https://www.wikiwand.com/en/Three-way_comparison)）来确定两个元素中哪个应该放在序列前面。

## Properties(3个基础属性)

* In-place: It does not require additional memory (except a few units of memory). (基本上不需要额外辅助的数据结构,然而,允许少量额外的辅助变量来转换数据)

* Stable: It preserves the relative order of elements that have identical keys. (原本有相等键值的纪录维持相对次序。也就是如果一个排序算法是**稳定**的，当有两个相等键值的纪录**R**和**S**，且在原本的串列中**R**出现在**S**之前，在排序过的串列中**R**也将会是在**S**之前。)
* Input-insensitive: Its running time is fairly independent of input properties other than size

## Summary(总结)

|                | In-place | Stable |     Worst-case     |    Average-case    |           Best-case            |
| -------------: | :------: | :----: | :----------------: | :----------------: | :----------------------------: |
| Selection Sort |    √     |   ×    | О(*n*<sup>2</sup>) | О(*n*<sup>2</sup>) |       О(*n*<sup>2</sup>)       |
| Insertion Sort |    √     |   √    | О(*n*<sup>2</sup>) | О(*n*<sup>2</sup>) |             О(*n*)             |
|     Shell Sort |    √     |   ×    | О(*n*<sup>2</sup>) |      Depends       |         O(*n* log *n*)         |
|      Heap Sort |    √     |   ×    |   O(*n* log *n*)   |   O(*n* log *n*)   | O(*n* log *n*) (distinct keys) |
|     Quick Sort |    √     |   ×    | О(*n*<sup>2</sup>) |   O(*n* log *n*)   |         O(*n* log *n*)         |
|     Merge Sort |    ×     |   √    |   O(*n* log *n*)   |   O(*n* log *n*)   |         O(*n* log *n*)         |

*注：附Wikipedia中的[Comparison Sorts的表格的链接](https://www.wikiwand.com/en/Sorting_algorithm#/Comparison_sorts) (更加全面详细)*

## Selection Sort(选择排序)

排序数列分为2部分，①已排序的部分和②未排序的部分，从index = 0开始，总是选择(Select) ②未排序部分的最小(大)值放到①已排序部分的末端，直至index = array.length - 1，完成排序。

| In-place | Stable | Worst-case         | Average-case       | Best-case          |
| :------- | ------ | ------------------ | ------------------ | ------------------ |
| √        | ×      | О(*n*<sup>2</sup>) | О(*n*<sup>2</sup>) | О(*n*<sup>2</sup>) |

```java
public static void selectionSort(int[] arr) {    
    int length = arr.length;    
    int min;    
    for (int i = 0; i < length - 1; i++) {        
        min = i;        
        //找到并选择(select)最小的Index        
        for (int k = i + 1; k < length; k++) {            
            if (arr[k] < arr[min]) {                
                min = k;            
            }        
        }        
        //交换 arr[i] 和 arr[min]        
        int temp = arr[i];        
        arr[i] = arr[min];        
        arr[min] = temp;    
    }
}
```

## Insertion Sort(插入排序)

排序数列分为2部分，①已排序的部分和②未排序的部分，从index = 1开始，总是将当前index的item插入(insert)到①已排序部分相对应的位置，直至index = array.length - 1，完成排序。

对**小**的Array较好（比如几百个元素）

| In-place | Stable | Worst-case         | Average-case       | Best-case |
| -------- | ------ | ------------------ | ------------------ | --------- |
| √        | √      | О(*n*<sup>2</sup>) | О(*n*<sup>2</sup>) | О(*n*)    |

*注：最好情况的 О( n )是在输入数组基本就是已排序的状态下，比如说用别的高效率的排序算法先排序一个混乱数组到差不多排序的状态下，让Insertion Sort接手继续排序。*

```java
public static void insertionSort(int[] arr) {
    int length = arr.length;
    for (int i = 1; i < length; i++) {
        int item = arr[i];
        int k = i - 1;
        //把比当前item小的items都后移一位
        while (k >= 0 && item < arr[k]) {
            arr[k + 1] = arr[k];
            k--;
        }
        //插入(Insert)当前item
        arr[k + 1] = item;
    }
}
```

## Shell Sort(希尔排序)

Insertion Sort的改进版本，设置一连串的Gap序列值(最后的Gap为1)，对数组的第 index + k * Gap  (0 <= index < Gap, k*Gap < array.length, k ∈ {0,1,2...}) 项组成的多个相对应的数组进行Insertion Sort，逐渐使Gap至1，对最后Gap为1的数组进行一次Insertion Sort，完成排序。

例子：假设有数组为 [ 13 14 94 33 82 25 59 94 65 23 45 27 73 25 39 10 ]  (引用自[Wikipedia](https://www.wikiwand.com/zh-cn/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F))

①Gap = 5 (每组都做一次Insertion Sort)

第[0,5,10,15] 为一组->即[13,25,45,10]为一组->排序后[10,13,25,45]

第[1,6,11]为一组       ->即[14,59,27]为一组     ->排序后[14,27,59]

第[2,7,12]为一组       ->即[94,94,73]为一组     ->排序后[73,94,94]

第[3,8,13]为一组       ->即[33,65,25]为一组     ->排序后[25,33,65]

第[4,9,14]为一组       ->即[82,23,39]为一组     ->排序后[23,39,82]

->第一轮Gap=5结果为[10,14,73,25,23,13,27,94,33,39,25,59,94,65,82,45]

②Gap = 3

第[0,3,6,9,12,15] 为一组->即[10,25,27,39,94,45]为一组->排序后[10,25,27,39,45,94]

第[1,4,7,10,13]为一组    ->即[14,23,94,25,65]为一组     ->排序后[14,23,25,65,94]

第[2,5,8,11,14]为一组    ->即[73,13,33,59,82]为一组     ->排序后[13,33,59,73,92]

->第二轮Gap=3结果为[10,14,13,25,23,33,27,25,59,39,65,73,45,94,92,94]

③Gap = 1 (等同于Insertion Sort)

->最终结果为[10,13,14,23,25,25,27,33,39,45,59,65,73,92,94,94]

| In-place | Stable | Worst-case         | Average-case | Best-case      |
| -------- | ------ | ------------------ | ------------ | -------------- |
| √        | ×      | О(*n*<sup>2</sup>) | Depends      | O(*n* log *n*) |

*注：Gap序列的选取决定了Shell Sort的排序速度和时间复杂度，一个好的Gap序列有助于提高效率(但选取好的Gap序列很难)*

```java
/**
    * 希尔排序利用 gap sequence 为 length/2, length/4 ... 1
    */
public static void shellSort(int[] arr) {
    int length = arr.length;
    //从数组长度/2的Gap开始, 减小到Gap = 1
    for (int gap = length / 2; gap > 0; gap /= 2) {
        //相当于Insertion Sort从第2个元素开始insert,
        //Shell Sort从第gap个元素开始insert
        for (int i = gap; i < length; i++) {

            int temp = arr[i];

            //相当于Insertion Sort比较当前的item和之前排序过的items
            //小于,则把之前的元素后移gap位(Insertion Sort是1位)
            int j;
            for (j = i; j >= gap && arr[j - gap] > temp; j -= gap) {
                arr[j] = arr[j - gap];
            }
            //把当前item 插入正确的位置
            arr[j] = temp;
        }
    }
}
```

## Heap Sort(堆排序)

堆排序（原地）是运用了堆的性质，基本思想是，给定一个未排序的数组 **H** [1...n], ①把 **H** 转化为一个最大或最小堆(利用sift down操作的时间复杂度是О(*n*))，②执行弹出操作n-1次(每次弹出操作的时间复杂度是O( log *n*) ，具体操作是将堆顶元素和堆尾元素交换位置，堆大小-1，对新的堆顶元素进行sift down操作致相对应的位置，在n-1次后原数组即排序完成)

小顶堆生成降序排列，大顶堆生成升序排序。

- 对①操作时间复杂度О(*n*)的证明:

为简便化，假设当前堆为一颗完全二叉树，则节点总数 *n=2<sup>(h+1)</sup> -1*  

其中h为height高度, root 为 height h, leaves为 height 0.

下列式子是 sift down 操作所需要的上限(worst-case):

$$ 
\sum^{h}_{i=1}\ \sum^{}_{nodes\:at\:hight\:i}{i} = \sum^{h}_{i=1}{i*2^{h-i}} = 2^{h+1}-h-2
$$

对height i 的每一个node它最糟糕的sift down就是被移动到height 0，即对height i 的 nodes来说，worst-case就是移动 i 次，height为h，即产生等式左侧的上限公式,可有数学归纳法简单得出右侧等式。

注意到，*2<sup>h+1</sup> - h - 2 < 2<sup>(h+1)</sup> -1 = n* ,所以，最多是linear的 sift down操作，每一个sift down 操作又是比较了2个key，所以比较操作也是linear的。因此①的建堆操作是О(*n*)的时间复杂度。

| In-place | Stable | Worst-case     | Average-case   | Best-case                     |
| -------- | ------ | -------------- | -------------- | ----------------------------- |
| √        | ×      | O(*n* log *n*) | O(*n* log *n*) | O(*n* log *n*)(distinct keys) |

*注：Java中的PriorityQueue不指定Comparator时默认为最小堆，可学习源码。*

```java
/**
 * 基于最大堆的堆排序
 */
public static void heapSort(int[] arr) {
    int size = arr.length;
    //建立一个最大堆
    for (int i = arr.length / 2 - 1; i >= 0; i--) {
        siftDown(arr, i, size);
    }
    //不断交换heap头和尾的数值，因为是最大堆，最终排序为从小到大
    while (size > 0) {
        int temp = arr[0];
        arr[0] = arr[size - 1];
        arr[size - 1] = temp;
        size--;
        //重新让arr符合heap的性质，让交换上来的第一位元素下沉
        siftDown(arr, 0, size);
    }
}
    
/**
 * 存在一个Parent节点的值小于任一Child节点，就让它Sift Down，直到符合堆的性质（最大堆）
 *
 * @param heap     存放堆的array
 * @param keyIndex 当前要处理的Parent节点的Index
 * @param heapSize Heap的大小
 */
private static void siftDown(int[] heap, int keyIndex, int heapSize) {
    int keyValue = heap[keyIndex];
    boolean isHeap = false;
    while (!isHeap && keyIndex < heapSize / 2) {
        //左孩子的index
        int childIndex = 2 * keyIndex + 1;
        //如果有右孩子的情况
        if (childIndex < heapSize - 1) {
            //比较两个child值的大小，把index值设为大的那个（最大堆中）
            if (heap[childIndex] < heap[childIndex + 1]) {
                childIndex++;
            }
        }
        //递归比较子节点的大小和keyValue的大小来决定是否满足heap的属性
        if (heap[childIndex] <= keyValue) {
            isHeap = true;
        } else {
            //child节点的值上移
            heap[keyIndex] = heap[childIndex];
            keyIndex = childIndex;
        }
    }
    //把原本的KeyValue放到最终的位置，形成一个heap
    heap[keyIndex] = keyValue;
}
```
## Quick Sort(快速排序)

快速排序也利用了分治(Divide and Conquer)的思想，然而和归并排序的方式不同，①它选择一个元素作为基准(pivot)，②并围绕拾取的基准分割给定的数组，将比基准小的元素放在基准前，将比基准大的元素放在基准后。③递归的不断排序子序列。

**快速排序改进方法**

- 选取基准值有数种具体方法，此选取方法对排序的时间性能有决定性影响：如①永远选取第一个元素（在下方代码中的实现），②Median-of-Three方法选取基准（有很大程度提高了速度） ③还有很多别的方法......

- Early Cut-off：在数组快要达到排序状态时，从快速排序切换到插入排序，可以一定程序提高速度。

| In-place | Stable | Worst-case         | Average-case   | Best-case      |
| :------- | ------ | ------------------ | -------------- | -------------- |
| √        | ×      | О(*n*<sup>2</sup>) | O(*n* log *n*) | O(*n* log *n*) |

*注1：快速排序被认为是最好排序方法来处理未排序的数据，即使归并排序有更好的表现保证，但快速排序在平均上来说更快！*

*注2：Java源码中的排序算法采用的DualPivotQuicksort，一种快速排序的方法之一（选取的Pivot方法不同）,且根据数组大小，结合了别的排序方法，可学习源码。*

```java
public static void quickSort(int[] arr) {
    quickSortHelper(arr, 0, arr.length - 1);
}

private static void quickSortHelper(int[] arr, int lo, int hi) {
    if (lo < hi) {
        int partitionIndex = partition(arr, lo, hi);
        quickSortHelper(arr, lo, partitionIndex - 1);
        quickSortHelper(arr, partitionIndex + 1, hi);
    }
}

/**
 * Hoare Partition（https://www.wikiwand.com/en/Quicksort#/Hoare_partition_scheme）
 * 一种简单的Partition方法，选取一个pivot（我在代码中选取了index=lo的元素），
 * 然后用2个指针，从头尾向对方移动，直到检测到一个大于pivot，一个小于或等于pivot，
 * 彼此是相对不正确的顺序，然后交换两者。当2个指针相遇，调整pivot到正确位置，并返回最终index。
 *
 * @param arr 等待被排序的数组
 * @param lo  开始index,
 * @param hi  结束index
 * @return 选取的pivot最终正确的位置的index
 */
private static int partition(int[] arr, int lo, int hi) {
    int pivot = arr[lo], loIndex = lo, hiIndex = hi;
    while (loIndex < hiIndex) {
        while (loIndex < hi && arr[loIndex] <= pivot) loIndex++;
        while (hiIndex > lo && arr[hiIndex] > pivot) hiIndex--;
        //交换逆序元素,如果不设这个判断，最后一次loIndex>hiIndex也会进行交换
        if (loIndex <= hiIndex) {
            int temp = arr[loIndex];
            arr[loIndex] = arr[hiIndex];
            arr[hiIndex] = temp;
        }
    }
    //将pivot放到正确的位置
    int temp = arr[lo];
    arr[lo] = arr[hiIndex];
    arr[hiIndex] = temp;
    //返回pivot最终正确的位置的index
    return hiIndex;
}
```

## Merge Sort(归并排序)

归并排序是分治(Divide and Conquer)的一个典型应用。总的来说，排序分成两部分 ①将未排序的列表分成n个子列表（有递归和迭代两种方式，具体见下方代码），每个子列表包含一个元素（一个元素的列表被视为已排序）。②重复合并子列表以生成新的已排序子列表，直到只剩下一个子列表， 最后剩下的就是排序好的列表。

| In-place | Stable | Worst-case     | Average-case   | Best-case      |
| :------- | ------ | -------------- | -------------- | -------------- |
| ×        | √      | O(*n* log *n*) | O(*n* log *n*) | O(*n* log *n*) |


```java
/**
 * 递归版(Top-down)的 归并排序
 * 处理一个未排序数组，不断递归的分成2半，分别排序，最后合并
 */
public static void mergeSortTopDown(int[] arr) {
    if (arr.length > 1) {
        int mid = arr.length / 2;
        //将原数组分成左右两半，分别拷贝到临时数组中
        //left=[leftIndex, midIndex), right = [midIndex, rightIndex)
        int[] left = Arrays.copyOfRange(arr, 0, mid);
        int[] right = Arrays.copyOfRange(arr, mid, arr.length);
        //对左右两半临时数组进行排序
        mergeSortTopDown(left);
        mergeSortTopDown(right);
        //将左右两半临时数组归并到原数组
        merge(left, right, arr);
    }
}

private static void merge(int[] left, int[] right, int[] arr) {
    //3个数组的初始index都设置为0
    int leftIndex = 0, rightIndex = 0, mergedIndex = 0;
    //不断比较左右两个数组中元素的大小，复制到原数组中，直到一个数组元素完全复制完
    while (leftIndex < left.length && rightIndex < right.length) {
        if (left[leftIndex] < right[rightIndex]) {
            arr[mergedIndex++] = left[leftIndex++];
        } else {
            arr[mergedIndex++] = right[rightIndex++];
        }
    }
    //将剩余一个数组中的剩余元素复制到原数组的末尾
    if (leftIndex == left.length) {
        System.arraycopy(right, rightIndex, arr, mergedIndex, 
                            right.length - rightIndex);
    } else {
        System.arraycopy(left, leftIndex, arr, mergedIndex, 
                            left.length - leftIndex);
    }
}
```

```java
/**
 * 迭代版(Bottom-up)的 归并排序
 * 迭代的对原数组中不断增长的2的倍数(2,4,8...)长度的子数组进行排序
 */
public static void mergeSortBottomUp(int[] arr) {
    int[] temp = new int[arr.length];
    //对从长度为2开始的数组进行排序
    for (int i = 2; i <= arr.length; i *= 2) {
        //对每一次排序而言，对当前数组对半分，进行排序
        for (int j = 0; j <= (arr.length - 1) / i; j++) {
            //数组左中右index，等同于将数组分成了左右两半，
            //对最后一个数组，需要判断是否超过原数组下标
            int left = i * j;
            int mid = Math.min(left + i / 2, arr.length);
            int right = Math.min(left + i, arr.length);
            //对左右数组进行排序操作，等同于Top-down里的merge操作
            int leftIndex = left, rightIndex = mid, mergedIndex = left;
            while (leftIndex < mid && rightIndex < right) {
                if (arr[leftIndex] < arr[rightIndex]) {
                    temp[mergedIndex++] = arr[leftIndex++];
                } else {
                    temp[mergedIndex++] = arr[rightIndex++];
                }
            }
            if (leftIndex == mid) {
                System.arraycopy(arr, rightIndex, temp, mergedIndex, 
                                    right - rightIndex);
            } else {
                System.arraycopy(arr, leftIndex, temp, mergedIndex, 
                                    mid - leftIndex);
            }
            //注释掉这句话可以看具体排序过程
            //System.out.println(Arrays.toString(temp));
            //将排序好的数组拷贝回原数组相对应位置
            System.arraycopy(temp, left, arr, left, right - left);
        }
    }
}
```