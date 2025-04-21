## **STL 算法**‌

以下是 ‌**STL 算法**‌的详细分类介绍，涵盖其功能、用法示例及注意事项：

------

### ‌**1. 遍历算法**‌

对容器元素进行遍历或统计。

#### ‌**(1) `std::for_each`**‌

- ‌**功能**‌：遍历容器，对每个元素执行指定操作。

- ‌**参数**‌：`起始迭代器`, `结束迭代器`, `函数对象`。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 3};
  std::for_each(vec.begin(), vec.end(), [](int x) {
      std::cout << x << " ";  // 输出：1 2 3
  });
  ```

#### ‌**(2) `std::count` / `std::count_if`**‌

- ‌**功能**‌：统计等于特定值或满足条件的元素数量。

- ‌**参数**‌：`起始迭代器`, `结束迭代器`, `值` 或 `谓词`。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::array<int, 5> arr{3, 2, 3, 4, 3};
  int cnt = std::count(arr.begin(), arr.end(), 3);          // 3 出现的次数 → 3
  int even_cnt = std::count_if(arr.begin(), arr.end(), [](int x) { return x % 2 == 0; }); // 偶数数量 → 2
  ```

#### ‌**(3) `std::all_of` / `std::any_of` / `std::none_of`**‌

- ‌**功能**‌：检查是否所有/任意/没有元素满足条件。

- ‌**参数**‌：`起始迭代器`, `结束迭代器`, `谓词`。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{2, 4, 6, 8};
  bool all_even = std::all_of(vec.begin(), vec.end(), [](int x) { return x % 2 == 0; }); // true
  bool has_5 = std::any_of(vec.begin(), vec.end(), [](int x) { return x == 5; });        // false
  ```

------

### ‌**2. 查找算法**‌

在容器中查找特定元素或极值。

#### ‌**(1) `std::find` / `std::find_if`**‌

- ‌**功能**‌：查找第一个等于指定值或满足条件的元素。

- ‌**参数**‌：`起始迭代器`, `结束迭代器`, `值` 或 `谓词`。

- ‌**返回值**‌：指向该元素的迭代器，未找到则返回 `end()`。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{5, 3, 7, 9, 2};
  auto it = std::find(vec.begin(), vec.end(), 7);  // 找到 7
  auto even_it = std::find_if(vec.begin(), vec.end(), [](int x) { return x % 2 == 0; }); // 找到第一个偶数 2
  ```

#### ‌**(2) `std::search`**‌

- ‌**功能**‌：在容器中查找子序列。

- ‌**参数**‌：`主序列起始`, `主序列结束`, `子序列起始`, `子序列结束`。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::string str = "hello world";
  std::string sub = "wo";
  auto pos = std::search(str.begin(), str.end(), sub.begin(), sub.end());  // 找到 "wo"
  ```

#### ‌**(3) `std::max_element` / `std::min_element`**‌

- ‌**功能**‌：查找最大/最小元素的迭代器。

- ‌**参数**‌：`起始迭代器`, `结束迭代器`（可选比较函数）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{3, 1, 4, 2, 5};
  auto max_it = std::max_element(vec.begin(), vec.end()); // 指向 5
  auto min_it = std::min_element(vec.begin(), vec.end()); // 指向 1
  ```

------

### ‌**3. 比较算法**‌

比较两个容器或元素范围。

#### ‌**(1) `std::equal`**‌

- ‌**功能**‌：判断两个范围是否相等。

- ‌**参数**‌：`范围1的起始`, `范围1的结束`, `范围2的起始`（可选自定义比较）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> a{1, 2, 3}, b{1, 2, 3};
  bool is_equal = std::equal(a.begin(), a.end(), b.begin()); // true
  ```

#### ‌**(2) `std::mismatch`**‌

- ‌**功能**‌：查找两个范围中第一个不匹配的元素对。

- ‌**返回值**‌：一对迭代器，指向不匹配的位置。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> a{1, 2, 4}, b{1, 2, 3};
  auto [it1, it2] = std::mismatch(a.begin(), a.end(), b.begin()); // it1指向4，it2指向3
  ```

------

### ‌**4. 填充/替换算法**‌

修改容器元素的值。

#### ‌**(1) `std::fill`**‌

- ‌**功能**‌：将指定范围填充为固定值。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec(5);
  std::fill(vec.begin(), vec.end(), 10); // vec → {10, 10, 10, 10, 10}
  ```

#### ‌**(2) `std::replace` / `std::replace_if`**‌

- ‌**功能**‌：替换满足条件的元素。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 3, 2, 5};
  std::replace(vec.begin(), vec.end(), 2, 20);  // 替换所有2 → {1, 20, 3, 20, 5}
  std::replace_if(vec.begin(), vec.end(), [](int x) { return x > 3; }, 0); // 替换>3的元素 → {1, 20, 3, 0, 0}
  ```

#### ‌**(3) `std::generate`**‌

- ‌**功能**‌：通过生成器函数填充范围。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec(5);
  int val = 0;
  std::generate(vec.begin(), vec.end(), ‌:ml-search[&val] { return val++; }); // vec → {0, 1, 2, 3, 4}
  ```

------

### ‌**5. 拷贝/移动算法**‌

复制或移动元素到其他位置。

#### ‌**(1) `std::copy` / `std::copy_if`**‌

- ‌**功能**‌：复制元素到目标容器。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> src{1, 2, 3}, dst(3);
  std::copy(src.begin(), src.end(), dst.begin()); // dst → {1, 2, 3}
  std::copy_if(src.begin(), src.end(), dst.begin(), [](int x) { return x % 2 == 1; }); // dst → {1, 3}
  ```

#### ‌**(2) `std::move`**‌

- ‌**功能**‌：移动元素（适用可移动类型，如 `std::string`）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<std::string> src{"A", "B", "C"}, dst(3);
  std::move(src.begin(), src.end(), dst.begin()); // src 元素变为空，dst 获得原值
  ```

------

### ‌**6. 变换算法**‌

对元素进行转换操作。

#### ‌**(1) `std::transform`**‌

- ‌**功能**‌：对元素进行转换（如平方、类型转换）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 3}, result(3);
  std::transform(vec.begin(), vec.end(), result.begin(), [](int x) { return x * x; }); // result → {1, 4, 9}
  ```

------

### ‌**7. 排序算法**‌

对元素进行排序或部分排序。

#### ‌**(1) `std::sort` / `std::stable_sort`**‌

- ‌**功能**‌：升序排序（默认）或稳定排序（保持相等元素顺序）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{5, 3, 1, 4, 2};
  std::sort(vec.begin(), vec.end());       // vec → {1, 2, 3, 4, 5}
  std::stable_sort(vec.begin(), vec.end(), std::greater<>()); // 稳定降序排序
  ```

#### ‌**(2) `std::partial_sort`**‌

- ‌**功能**‌：部分排序（如前 N 个元素有序）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{5, 3, 1, 4, 2};
  std::partial_sort(vec.begin(), vec.begin() + 3, vec.end()); // 前3个有序 → {1, 2, 3, 5, 4}
  ```

------

### ‌**8. 划分算法**‌

将元素按条件分为两组。

#### ‌**(1) `std::partition`**‌

- ‌**功能**‌：将满足条件的元素移动到前面。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{3, 5, 2, 1, 4, 6};
  auto it = std::partition(vec.begin(), vec.end(), [](int x) { return x % 2 == 0; }); // 偶数在前
  ```

#### ‌**(2) `std::nth_element`**‌

- ‌**功能**‌：确保第 N 个元素在排序后的正确位置，且左边≤它，右边≥它。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{5, 3, 1, 4, 2};
  std::nth_element(vec.begin(), vec.begin() + 2, vec.end()); // 第3小元素是3 → vec可能为 {2, 1, 3, 4, 5}
  ```

------

### ‌**9. 堆操作算法**‌

构建或维护堆结构。

#### ‌**(1) `std::make_heap` / `std::push_heap`**‌

- ‌**功能**‌：将范围转换为堆，或在堆末尾添加元素后调整堆。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{3, 1, 4, 1, 5};
  std::make_heap(vec.begin(), vec.end()); // 构建最大堆 → 5 3 4 1 1
  vec.push_back(6);
  std::push_heap(vec.begin(), vec.end()); // 调整堆 → 6 3 5 1 1 4
  ```

------

### ‌**10. 数值算法**‌

数学计算相关操作。

#### ‌**(1) `std::accumulate`**‌

- ‌**功能**‌：累加（或自定义操作，如累乘）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 3, 4};
  int sum = std::accumulate(vec.begin(), vec.end(), 0);                   // 1+2+3+4=10
  int product = std::accumulate(vec.begin(), vec.end(), 1, [](int a, int b) { return a * b; }); // 24
  ```

#### ‌**(2) `std::inner_product`**‌

- ‌**功能**‌：计算两个范围的内积（点积）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> a{1, 2}, b{3, 4};
  int dot = std::inner_product(a.begin(), a.end(), b.begin(), 0); // (1*3) + (2*4) = 11
  ```

#### ‌**(3) `std::partial_sum`**‌

- ‌**功能**‌：计算前缀和。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> src{1, 2, 3}, dst(3);
  std::partial_sum(src.begin(), src.end(), dst.begin()); // dst → {1, 3, 6}
  ```

------

### ‌**11. 移除算法**‌

逻辑移除元素（需后续处理）。

#### ‌**(1) `std::remove` / `std::remove_if`**‌

- ‌**功能**‌：将不满足条件的元素移动到前面，返回新逻辑结尾的迭代器。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 3, 2, 5};
  auto new_end = std::remove(vec.begin(), vec.end(), 2); // vec → {1, 3, 5, 2, 5}，new_end 指向第3个位置
  vec.erase(new_end, vec.end()); // 物理删除 → {1, 3, 5}
  ```

------

### ‌**12. 去重算法**‌

去除连续重复元素。

#### ‌**(1) `std::unique`**‌

- ‌**功能**‌：去除连续重复元素（需先排序）。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> vec{1, 2, 2, 3, 3, 3};
  auto last = std::unique(vec.begin(), vec.end()); // vec → {1, 2, 3, 3, 3}, last 指向第3个元素
  vec.erase(last, vec.end()); // → {1, 2, 3}
  ```

------

### ‌**13. 交换算法**‌

交换两个范围的元素。

#### ‌**(1) `std::swap_ranges`**‌

- ‌**功能**‌：交换两个范围中的元素。

- ‌

  示例

  ‌：

  ```
  cppCopy Codestd::vector<int> a{1, 2, 3}, b{4, 5, 6};
  std::swap_ranges(a.begin(), a.end(), b.begin()); // a → {4,5,6}, b → {1,2,3}
  ```

------

### ‌**总结**‌

- ‌**通用性**‌：所有算法通过迭代器操作，与容器类型解耦。
- ‌**性能**‌：大多数算法时间复杂度为线性或对数级（如排序）。
- ‌**适用性**‌：`std::array` 支持所有不依赖动态大小的算法，移除操作需手动处理后续元素。