---
title: 排序算法的总结 (c++实现)
date: 2017-03-20 11:16:27
categories: coding
tags: [sort, c++]
---

周末复习了数据结构中排序的知识，用c++实现了一遍（完整的项目代码见文末Github链接），总结如下。

<!--more-->

author: [@Huji][1]

## 冒泡排序 BubbleSort

【**原理**】 

遍历要排序的序列，比较两个相邻的元素，如果顺序错误就进行交换，直到序列有序。

【**步骤**】

- 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
- 对第0个到第n-1个数据做同样的工作。这时，最大的数就“浮”到了数组最后的位置上。
- 针对所有的元素重复以上的步骤，除了最后一个。
- 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

【**代码实现**】

```
vector<int> bubble_sort(vector<int> arr)
{
    int n = arr.size();
    for(int i=0; i<n; ++i){
        for(int j=1; j<(n-i);++j){
            if(arr[j-1] > arr[j]){
                swap(arr[j-1], arr[j]);
            }
        }
    }
    return arr;
}
```

上述代码可以进行优化。

- 优化1：如果某一次遍历没有数据交换了，那么就做一个标记，跳出循环。
- 优化2：记录某次遍历的时候最后发生数据交换的位置，后面的数据已经排好序，那么循环的范围就可以缩小了。

详细的优化方案1和2的代码见文末。

## 选择排序 SelectionSort

【**原理**】

每次手动选择最小的元素放到已排序序列的末尾。（和冒泡相比少了两两比较交换的过程）

【**步骤**】

- 在未排序序列中找到最小（大）元素，存放到排序序列的起始位置。
- 再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。
- 以此类推，直到所有元素均排序完毕。

【**代码实现**】

```
vector<int> selection_sort(vector<int> arr)
{
    int n = arr.size();
    for(int i=0; i<n; ++i)
    {
        int minPos = i;
        for(int j=i+1;j<n;++j)
        {
            if(arr[j]<arr[minPos])
                minPos = j;
        }
        swap(arr[i], arr[minPos]);
    }
    return arr;
}
```

## 插入排序 InsertionSort

【**原理**】

对每一个未排序元素，从后往前扫描已排序序列，插入到合适的位置。

【**步骤**】

- 从第一个元素开始，该元素可以认为已经被排序
- 取出下一个元素，在已经排序的元素序列中从后向前扫描
- 如果被扫描的元素（已排序）大于新元素，将该元素后移一位
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
- 将新元素插入到该位置后
- 重复步骤2~5

【**代码实现**】

```
vector<int> insertion_sort(vector<int> arr)
{
    int n = arr.size();
    int i,j,tmp;

    for(i=1; i<n; ++i)
    {
        if(arr[i]<arr[i-1])
        {
            tmp = arr[i];
            for(j=i; arr[j-1]>tmp && j>0; --j)
            {
                arr[j] = arr[j-1];
            }
            arr[j] = tmp;
        }
    }
    return arr;
}
```

## 希尔排序 ShellSort

【**原理**】

实际上是分组的插入排序。把记录按步长分组，对每组记录采用直接插入排序方法进行排序。随着步长逐渐减小，所分成的组包含的记录越来越多，当步长的值减小到 1 时，整个数据合成为一组，构成一组有序记录，则完成排序。

【**步骤**】

例如，假设有这样一组数[ 13 14 94 33 82 25 59 94 65 23 45 27 73 25 39 10 ]，如果我们以步长为5开始进行排序，我们可以通过将这列表放在有5列的表中来更好地描述算法，这样他们就应该看起来是这样：

```
13 14 94 33 82
25 59 94 65 23
45 27 73 25 39
10
```
然后我们对每列进行排序：
```
10 14 73 25 23
13 27 94 33 39
25 59 94 65 82
45
```
将上述四行数字，依序接在一起时我们得到：[ 10 14 73 25 23 13 27 94 33 39 25 59 94 65 82 45 ]。这时10已经移至正确位置了，然后再以3为步长进行排序：
```
10 14 73
25 23 13
27 94 33
39 25 59
94 65 82
45
```
排序之后变为：
```
10 14 13
25 23 33
27 25 59
39 65 73
45 94 82
94
```
最后以1步长进行排序（此时就是简单的插入排序了）。

【**代码实现**】

```
vector<int> shell_sort(vector<int> arr)
{
    int n = arr.size();
    int i,j,tmp;
    int step = round(n/2);
    while(step > 0)
    {
        for(i=step; i<n; ++i)
        {
            if(arr[i] < arr[i-step])
            {
                tmp = arr[i];
                for(j=i; arr[j-step]>tmp && j>=step; j=j-step){
                    arr[j] = arr[j-step];
                }
                arr[j] = tmp;
            }
        }
        step = round(step/2);
    }
    return arr;
}
```

## 归并排序 MergeSort

【**思想**】

归并排序就是不断地划分待排序的序列，直到序列长度为1就是有序序列，然后再逐步将两个已经排序的序列合并成一个序列。

![mergesort][2]

【**步骤**】

- Divide: 把长度为n的输入序列分成两个长度为n/2的子序列。

- Conquer: 对这两个子序列分别采用归并排序。

- Combine: 将两个排序好的子序列合并成一个最终的排序序列。

【**代码实现**】
```
void merge_array(vector<int> &arr, int b, int m, int e)
{
    int i=b, j=m+1, k=b;
    vector<int> tmp = arr;
    while(i<=m && j<=e) arr[k++] = tmp[i]<tmp[j]? tmp[i++]:tmp[j++];
    while(i<=m) arr[k++] = tmp[i++];
    while(j<=e) arr[k++] = tmp[j++];
}

void mergesort(vector<int> &arr, int b, int e)
{
    if(b<e)
    {
        int m = (b+e)/2;
        mergesort(arr, b, m);
        mergesort(arr, m+1, e);
        merge_array(arr, b, m, e);
    }
}
vector<int> merge_sort(vector<int> arr)
{
    int n = arr.size();
    vector<int> result = arr;
    mergesort(result, 0, n-1);
    return result;
}

```

## 快速排序 QuickSort

【**原理**[^quicksort]】

通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

![quicksort][3]

【**步骤**】

- 在数据集之中，选择一个元素作为”基准”（pivot）。
- 所有小于”基准”的元素，都移到”基准”的左边；所有大于”基准”的元素，都移到”基准”的右边。这个操作称为分区 (partition) 操作，分区操作结束后，基准元素所处的位置就是最终排序后它的位置。
- 对”基准”左边和右边的两个子集，不断重复第一步和第二步，直到所有子集只剩下一个元素为止。

【**代码实现**】

```
vector<int> quick_sort(vector<int> arr)
{
    int n = arr.size();
    vector<int> result = arr;
    quicksort(result, 0, n-1);
    return result;
}
void quicksort(vector<int> &arr, int low, int high)
{
    int pivot;
    if(low<high)
    {
        pivot = Partition(arr, low, high);
        quicksort(arr, low, pivot-1);
        quicksort(arr, pivot+1, high);
    }
}
int Partition(vector<int> &arr,int low,int high)
{
    int pivotKey;
    pivotKey = arr[low];
    while(low<high)
    {
        while(low<high&&arr[high]>=pivotKey)
        {
            high--;
        }
        swap(arr[low],arr[high]);
        while(low<high&&arr[low]<=pivotKey)
        {
            low++;
        }
        swap(arr[low],arr[high]);
    }
    return low;
}
```

## [堆排序 HeapSort][4]

【**原理**[^heapsort]】

堆排序就是把最大堆堆顶的最大数取出，将剩余的堆继续调整为最大堆，再次将堆顶的最大数取出，这个过程持续到剩余数只有一个时结束。

![heapsort][5]

【**步骤**】

- 构造最大堆（Build_Max_Heap）：若数组下标范围为0~n，考虑到单独一个元素是大根堆，则从下标n/2开始的元素均为大根堆。于是只要从n/2-1开始，向前依次构造大根堆，这样就能保证，构造到某个节点时，它的左右子树都已经是大根堆。

- 堆排序（HeapSort）：由于堆是用数组模拟的。得到一个大根堆后，数组内部并不是有序的。因此需要将堆化数组有序化。思想是移除根节点，并做最大堆调整的递归运算。第一次将heap[0]与heap[n-1]交换，再对heap[0...n-2]做最大堆调整。第二次将heap[0]与heap[n-2]交换，再对heap[0...n-3]做最大堆调整。重复该操作直至heap[0]和heap[1]交换。由于每次都是将最大的数并入到后面的有序区间，故操作完后整个数组就是有序的了。

- 最大堆调整（Max_Heapify）：该方法是提供给上述两个过程调用的。目的是将堆的末端子节点作调整，使得子节点永远小于父节点 。

【**代码实现**】

```
void Heap_Adjust(vector<int> &arr,int s,int m)
{
    int temp = arr[s];
    for(int j=2*s+1;j<=m;j = 2*j+1)
    {
        if(arr[j]<arr[j+1]&&j<m)
        {
            j++;
        }
        if(temp>arr[j])
            break;
        arr[s] = arr[j];
        s = j;
    }
    arr[s] = temp;
}

//堆排序
void heapsort(vector<int> &arr, int n)
{
    //创建一个大顶堆
    for(int s = n/2-1;s>=0;s--)
    {
        Heap_Adjust(arr,s,n-1);
    }

    //排序
    for(int i = n-1;i >= 1;i--)
    {
        swap(arr[0],arr[i]);
        Heap_Adjust(arr,0,i-1);
    }

}

vector<int> heap_sort(vector<int> arr)
{
    int n = arr.size();
    vector<int> result = arr;
    heapsort(result, n);
    return result;
}
```

## 排序算法的对比

对比如下表所示

![compare][6]

**项目的源代码见[这里][7]，输出如下图所示**

![output][8]


[^heapsort]: http://bubkoo.com/2014/01/14/sort-algorithm/heap-sort/

[^quicksort]: http://developer.51cto.com/art/201403/430986.htm


  [1]: https://hujichn.github.io/
  [2]: http://7xsoqo.com1.z0.glb.clouddn.com/sort/Merge-sort.gif
  [3]: http://7xsoqo.com1.z0.glb.clouddn.com/sort/Quicksort.gif
  [4]: http://blog.csdn.net/morewindows/article/details/6709644
  [5]: http://7xsoqo.com1.z0.glb.clouddn.com/sort/Heapsort.gif
  [6]: http://7xsoqo.com1.z0.glb.clouddn.com/sort/compare.jpg
  [7]: https://github.com/hujichn/sort
  [8]: http://7xsoqo.com1.z0.glb.clouddn.com/sort/output.png