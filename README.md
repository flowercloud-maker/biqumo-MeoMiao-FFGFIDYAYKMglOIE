
## 思想：非比较而是划分值域


基数排序（Radix Sort）是一种非比较排序算法，它通过逐位对数据进行处理，依次按位从**最低有效位（Least Significant Digit, LSD）**到**最高有效位（Most Significant Digit, MSD）**或者反过来，对数据进行排序。


与常见的比较排序算法（如快速排序、归并排序）存在本质不同，基数排序不基于元素间的直接比较，而是依赖于元素的位权信息来排序。这使得基数排序能够在某些情况下实现线性时间复杂度，理论上达到 O(kn)O(kn)O(kn)，其中 kkk 是位数，nnn 是元素个数。即复杂度取决于值域规模。


基数排序的核心思想是分桶和合并：通过多次分桶操作，将元素按照某个位的值放入对应的桶中，然后再按照桶的顺序合并，逐步将数组排序。


## 简单的十进制基数排序


以下是 LSD 基数排序的简单流程，基于十进制：


1. 找到数组中最大的数，确定需要处理的最大位数 ddd。
2. 从最低有效位开始，依次对每一位执行以下步骤：
	* 使用计数排序（Counting Sort）等稳定排序算法，根据当前位的值对数据进行分桶。
	* 按分桶的顺序重新排列数组。


这种方式之所以有效，是因为每次分桶时，数据局部是有序的，每一位的排序是稳定的，因此较高位的排序不会改变低位已排序数字的相对顺序。通过逐位排序，逐渐将数字按整体大小排列。想象整理一叠卡片，先按卡片右侧的颜色分组，再按中间的图案分组，最后按左侧的形状分组。由于每一步都保留了之前分组的顺序，最终整理好的卡片是完全有序的。



```


|  | #include |
| --- | --- |
|  | #include |
|  | #include |
|  | using namespace std; |
|  |  |
|  | void countingSort(vector<int>& arr, int exp) { |
|  | int n = arr.size(); |
|  | vector<int> output(n); |
|  | vector<int> count(10, 0); // count[i]: 第 exp 位为 i 的数有多少？ |
|  |  |
|  | for (int i = 0; i < n; i++) |
|  | count[(arr[i] / exp) % 10]++; |
|  |  |
|  | for (int i = 1; i < 10; i++) |
|  | count[i] += count[i - 1]; |
|  |  |
|  | for (int i = n - 1; i >= 0; i--) { |
|  | // 根据第 exp 位决定应该占用的位置， |
|  | output[count[(arr[i] / exp) % 10] - 1] = arr[i]; |
|  | count[(arr[i] / exp) % 10]--; |
|  | } |
|  |  |
|  | for (int i = 0; i < n; i++) |
|  | arr[i] = output[i]; |
|  | } |
|  |  |
|  | void radixSort(vector<int>& arr) { |
|  | int maxVal = *max_element(arr.begin(), arr.end()); |
|  |  |
|  | for (int exp = 1; maxVal / exp > 0; exp *= 10) |
|  | countingSort(arr, exp); |
|  | } |
|  |  |
|  | int main() { |
|  | vector<int> arr = {170, 45, 75, 90, 802, 24, 2, 66}; |
|  | radixSort(arr); |
|  |  |
|  | for (int num : arr) |
|  | cout << num << " "; |
|  | return 0; |
|  | } |


```

可以把百，十，个位依次认为是第一，第二，第三关键字，按关键字权重低到高多趟排序，而排到某一位时，其低位已经排序好，只要按顺序取出就可以保持顺序。


从中可以看出，基数排序通常需要 O(n\+k)O(n\+k)O(n\+k) 的辅助空间。且在对每一位（如个位、十位）排序时，基数排序是稳定的，相同键值的元素在排序后相对顺序保持不变。而高位的排序不会改变低位已排好元素的顺序。因此，基数排序天然稳定。


## 高位优先（MSD）和低位优先（LSD）


上述是一个 LSD 的例子，其实从高位到低位也是可行的，且理解起来更加简单。MSD 基数排序从数字的最高位开始排序，将数字分组到不同的桶中（如按千位分组）。对每个桶分别递归排序，逐步向低位处理。每轮排序后，桶的内容会被按顺序合并。像按照城市、省份、街道逐级分类邮件。先按城市分，再在每个城市中按省份分，最后在每个省份中按街道分。高层次分类先决定大范围，递归细分确保每个细节都正确。



```


|  | void msdRadixSortUtil(vector<int>& arr, int left, int right, int exp) { |
| --- | --- |
|  | if (left >= right || exp == 0) return; |
|  |  |
|  | vectorint>> buckets(10); |
|  |  |
|  | // Place elements into corresponding buckets based on the current significant digit |
|  | for (int i = left; i <= right; i++) { |
|  | int digit = (arr[i] / exp) % 10; |
|  | buckets[digit].push_back(arr[i]); |
|  | } |
|  |  |
|  | // Merge buckets back into the array |
|  | int index = left; |
|  | for (int i = 0; i < 10; i++) { |
|  | for (int num : buckets[i]) { |
|  | arr[index++] = num; |
|  | } |
|  | } |
|  |  |
|  | // Recursively sort each non-empty bucket |
|  | index = left; |
|  | for (int i = 0; i < 10; i++) { |
|  | if (!buckets[i].empty()) { |
|  | int bucketSize = buckets[i].size(); |
|  | msdRadixSortUtil(arr, index, index + bucketSize - 1, exp / 10); |
|  | index += bucketSize; |
|  | } |
|  | } |
|  | } |
|  |  |
|  | void msdRadixSort(vector<int>& arr) { |
|  | if (arr.empty()) return; |
|  |  |
|  | // Find the maximum value to determine the number of digits |
|  | int maxVal = *max_element(arr.begin(), arr.end()); |
|  | int maxExp = pow(10, static_cast<int>(log10(maxVal))); |
|  |  |
|  | // Start MSD radix sort from the highest significant digit |
|  | msdRadixSortUtil(arr, 0, arr.size() - 1, maxExp); |
|  | } |


```

**高位优先（MSD）排序**从最高有效位开始排序，递归地对子数组进行分桶，逐步细化到最后的排序结果，通常需要递归。MSD方法常用于字符串排序，因为它可以提前确定不同类别。


**低位优先（LSD）排序**从最低有效位开始排序，逐步提升到最高有效位。它每次操作的排序范围是全数组，且每次排序不破坏之前的顺序（稳定性）。因此，对于整数排序，LSD方法更为常用。


两种方法都可行，但低位优先排序实现简单，且可以直接应用于数字，故在实践中更受欢迎。


## 二进制的基数排序


计算机中的数据都使用二进制（或者说十六进制）存储，十进制会导致每位信息利用不充分，且需要低效的模 10 运算，非常低效。


假设仅有正数，对于无符号32位整数，可以按二进制位分为多组。例如，每次处理8位（共分4组）。这样的处理方式仍然保持基数排序的思想，但使用更接近硬件位操作的方式，比十进制效率高得多，处理效率高。



```


|  | void radixSortBinary(vector<uint32_t>& arr) { |
| --- | --- |
|  | const int BITS = 32; |
|  | const int RADIX = 256; // 每次处理8位 |
|  | const int MASK = RADIX - 1; |
|  |  |
|  | vector<uint32_t> buffer(arr.size()); |
|  |  |
|  | // 四轮循环，分别处理0 - 7, 8 - 15, 16 - 23, 24 - 32 位。count 大小也增加到 256 |
|  | for (int shift = 0; shift < BITS; shift += 8) { |
|  | array<int, RADIX> count = {0}; |
|  |  |
|  | for (uint32_t num : arr) |
|  | count[(num >> shift) & MASK]++; |
|  |  |
|  | for (int i = 1; i < RADIX; i++) |
|  | count[i] += count[i - 1]; |
|  |  |
|  | for (int i = arr.size() - 1; i >= 0; i--) { |
|  | uint32_t bucket = (arr[i] >> shift) & MASK; |
|  | buffer[--count[bucket]] = arr[i]; |
|  | } |
|  |  |
|  | arr.swap(buffer); |
|  | } |
|  | } |
|  |  |
|  | int main() { |
|  | vector<uint32_t> arr = {170, 45, 75, 90, 802, 24, 2, 66}; |
|  | radixSortBinary(arr); |
|  |  |
|  | for (uint32_t num : arr) |
|  | cout << num << " "; |
|  | return 0; |
|  | } |


```

### 每次处理位数和


上例处理 32 位整数，分四次排序，一次八位。称呼其位宽 8。实际上也可以选择一次排序 16 位，排序两次，可以减少一半的轮次，但创建 65536 个桶可能会导致内存压力，且桶分布不均时效率下降：如果数据的分布高度集中，某些桶可能会很大，导致操作不均衡。若位宽仅 4，则分桶的范围较小，分桶和合并过程相对快速，但排序的趟数太多，适合小规模数组或内存受限的场景。


## 拓展知识


### 基数排序与快速排序


基数排序和快速排序是两种经典的排序算法，适用于不同场景。**基数排序**是一种非比较排序算法，依赖数字的位数特性，通过按位分组排序实现有序，适合处理数字或固定长度的字符串，具有线性时间复杂度 O(n⋅d)O(n \\cdot d)O(n⋅d)（其中 ddd 是位数）。它对数据规模较大且值域较小的数据表现出色，但需要额外的空间来存储桶。相比之下，**快速排序**是最经典的基于比较的分治算法，通过选择一个基准值（pivot）将数组划分为两部分递归排序，平均时间复杂度为 O(nlog⁡n)O(n \\log n)O(nlogn)。快速排序在大多数情况下效率极高，适用于通用数据类型，且原地排序所需额外空间较少，但其性能可能因基准选择不当而退化。简而言之，基数排序适合特定结构数据（如整数或字符串），而快速排序更通用，适合各种类型和规模的输入数据。


### 基数排序和桶排序


基数排序和桶排序虽然都是基于分组的非比较排序算法，但它们的目标和实现方式有所不同，且可以认为**基数排序是桶排序的延伸**。桶排序通过将数据分布到有限数量的桶中，每个桶内部再进行排序（通常使用插入排序或其他算法），最终将桶内容按顺序合并以获得排序结果；它主要依赖数据的分布特性，适用于数据均匀分布的场景，时间复杂度接近 O(n)O(n)O(n)。而基数排序本质上可以看作是多轮的桶排序：在值域很大时，它通过按位（如个位、十位等）多次分桶并排序，逐步实现最终的全局有序性。基数排序的核心思想是通过多次分桶来解决单次分桶无法处理多位数据的问题。因此，可以理解为基数排序在设计上对桶排序的扩展，用于处理数字、固定长度字符串等多位特征的数据。


### 基数排序应用于非整数


某些场合下，**基数排序可以扩展应用于非整数（如浮点数）和结构体**，但需要对数据进行适当的预处理，使其特性适合基数排序的机制。以下是实现这些扩展的关键思路：




---


#### **1\. 处理浮点数**


浮点数位码有一个特殊性质：IEEE 754 格式保证了从小到大的正数，其位模式从小到大单调递增。因此，可以直接将浮点数的位模式解释为无符号整数，然后按整数排序。也就是说，若不考虑符号可以直接视为整数排序。



```


|  | void radixSortFloat(vector<float>& arr) { |
| --- | --- |
|  | vector<uint32_t> bitPattern(arr.size()); |
|  |  |
|  | // 将浮点数解释为无符号整数，假设浮点数均是正数。 |
|  | for (size_t i = 0; i < arr.size(); ++i) { |
|  | memcpy(&bitPattern[i], &arr[i], sizeof(float)); |
|  | } |
|  |  |
|  | // 对无符号整数排序 |
|  | radixSort(bitPattern.begin(), bitPattern.end()); |
|  |  |
|  | // 排序完成后还原为浮点数 |
|  | for (size_t i = 0; i < arr.size(); ++i) { |
|  | memcpy(&arr[i], &bitPattern[i], sizeof(float)); |
|  | } |
|  | } |


```



---


#### **2\. 处理结构体**


基数排序天然可以划分关键字，对于结构体，可以通过选择一个或多个键值（字段）作为排序依据，将结构体排序问题转化为对这些键的排序。


比如对于包含字段 `age` 和 `salary` 的结构体数组：



```


|  | struct Employee { |
| --- | --- |
|  | int age; |
|  | double salary; |
|  | }; |


```

若 `age` 为第一关键字，且两属性均为正数，则可以直接将整个结构体划分位宽进行基数排序。


 本博客参考[PodHub豆荚加速器官方网站](https://rikeduke.com)。转载请注明出处！
