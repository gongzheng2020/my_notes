
## 冒泡排序

![[Bubble_sort_animation.gif]]

重复地走访过要排序的数列，**一次比较两个元素**，如果他们的**顺序错误就把他们交换过来**，O(n^2)。

``` c
#include <stdio.h>
// 函数声明
void bubble_sort(int arr[], int len);
int main() {
    int arr[] = { 22, 34, 3, 32, 82, 55, 89, 50, 37, 5, 64, 35, 9, 70 };
    int len = sizeof(arr) / sizeof(arr[0]);  // 计算数组长度
    bubble_sort(arr, len);  // 调用冒泡排序函数
    // 打印排序后的数组
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
    return 0;
}
// 冒泡排序函数
void bubble_sort(int arr[], int len) {
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - 1 - i; j++) {
            // 交换元素位置
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

---

## 选择排序

![[Selection_sort_animation.gif]]

首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾，0(n^2)。

``` c
#include <stdio.h>
// 函数声明
void bubble_sort(int arr[], int len);
int main() {
    int arr[] = { 22, 34, 3, 32, 82, 55, 89, 50, 37, 5, 64, 35, 9, 70 };
    int len = sizeof(arr) / sizeof(arr[0]);  // 计算数组长度
    bubble_sort(arr, len);  // 调用冒泡排序函数
    // 打印排序后的数组
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
    return 0;
}
// 冒泡排序函数
void bubble_sort(int arr[], int len) {
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - 1 - i; j++) {
            // 交换元素位置
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

---

## 插入排序

![[Insertion_sort_animation.gif]]

它的工作原理是通过构建有序序列，**对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入**，0(n^2)

``` c
#include <stdio.h>
// 函数声明
void insertion_sort(int arr[], int len);
int main() {
    int arr[] = { 22, 34, 3, 32, 82, 55, 89, 50, 37, 5, 64, 35, 9, 70 };
    int len = sizeof(arr) / sizeof(arr[0]);  // 计算数组长度
 
    insertion_sort(arr, len);  // 调用插入排序函数
 
    // 打印排序后的数组
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
 
    return 0;
}
// 插入排序函数
void insertion_sort(int arr[], int len) {
    for (int i = 1; i < len; i++) {
        int temp = arr[i];  // 当前待插入的元素
        int j = i;
        // 向右移动大于temp的元素
        while (j > 0 && arr[j - 1] > temp) {
            arr[j] = arr[j - 1];
            j--;
        }
        arr[j] = temp;  // 插入元素到正确位置
    }
}
```

---

## 希尔排序

![[Sorting_shellsort_anim.gif]]

算法步骤如下：
1. 设初始增量 `gap = ⌊n/2⌋`；
2. 将数组中下标相差 `gap` 的元素分为一组，共 `gap` 组；
3. 对每组独立执行插入排序；
4. 更新 `gap = ⌊gap/2⌋`，重复步骤 2~3；
5. 当 `gap == 1` 时，执行最后一次插入排序，排序结束。

>**直观理解：**先让相距较远的元素**“大跨步”移动到大致正确的位置**，再逐步缩小步长进行**局部微调**，最后用**步长为1的插入排序收尾**，0(n^2)。

``` c
#include <stdio.h>
// 函数声明
void shell_sort(int arr[], int len);
int main() {
    int arr[] = { 22, 34, 3, 32, 82, 55, 89, 50, 37, 5, 64, 35, 9, 70 };
    int len = sizeof(arr) / sizeof(arr[0]);  // 计算数组长度
    shell_sort(arr, len);  // 调用希尔排序函数
    // 打印排序后的数组
    for (int i = 0; i < len; i++) {
        printf("%d ", arr[i]);
    }
    return 0;
}
// 希尔排序函数
void shell_sort(int arr[], int len) {
    // 计算初始间隔
    for (int gap = len / 2; gap > 0; gap /= 2) {
        // 对每个间隔进行插入排序
        for (int i = gap; i < len; i++) {
            int temp = arr[i];  // 当前待插入的元素
            int j = i;
            // 移动大于temp的元素
            while (j >= gap && arr[j - gap] > temp) {
                arr[j] = arr[j - gap];
                j -= gap;
            }
            arr[j] = temp;  // 插入元素到正确位置
        }
    }
}
```

---

## 归并排序

把数据分为两段，从**两段中逐个选最小的元素移入新数据段的末尾**，0(nlogn)。

![[Merge_sort_animation2.gif]]
![[Merge-sort-example-300px.gif]]

---

## 快速排序

在区间中随机挑选一个元素作基准，将小于基准的元素放在基准之前，大于基准的元素放在基准之后，再分别对小数区与大数区进行排序，0(n^2)。

![[Sorting_quicksort_anim.gif]]

1. **选基准（Pivot）**：从数组中选择一个元素作为基准（可取首、尾、中间或随机元素）。
2. **分区（Partition）**：重新排列数组，使所有**小于**基准的元素放在其左侧，所有**大于**基准的元素放在右侧。分区完成后，基准元素就处于最终排序后的正确位置。
3. **递归排序**：对基准左右两个子数组重复上述过程，直到子数组长度为 `0` 或 `1`（自然有序）。