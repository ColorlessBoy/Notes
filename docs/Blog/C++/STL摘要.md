# STL 摘要

## 删除 `erase`
从 C++11 起，删除某个迭代器，函数返回被删除迭代器的下一个迭代器，
时间复杂度是 $O(\ln n + k)$。
这里借用一个[示例](https://zh.cppreference.com/w/cpp/container/set/erase)来演示。

```c++
#include <set>
#include <iostream>

int main() {
    std::set<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    // 删除所有奇数
    for(auto it = nums.begin(); it != nums.end();) {
        if(*it % 2 == 1) {
            it = nums.erase(it);
        }
        else {
            ++it;
        }
    }
    for(auto n: nums) {
        std::cout << n << ' ';
    }
    return 0;
}
```

## priority_queue 默认大顶堆

priority_queue 默认是大顶堆，即默认使用 `less<T>`，这里容易记混了。

```cpp
#include <iostream>
#include <vector>
#include <queue>
#include <functional>

int main() {
    std::vector<int> nums = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    std::priority_queue<int, std::vector<int>, std::greater<int>> 
        pq(nums.begin(), nums.end());
    while(!pq.empty()) {
        // 优先队列的遍历很难受
        std::cout << pq.top() << ' ';
        pq.pop();
    }
    return 0;
}
```

## 交换容器的时间复杂度

在 C++11 后，`std::swap()` 交换容器的时间复杂度为 $O(1)$，
在 C++11 前，`std::swap()` 交换 `queue`, `stack`, `priority_queue` 的时间复杂度是 $O(1)$。 