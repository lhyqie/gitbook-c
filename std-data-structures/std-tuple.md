# std::tuple

```cpp
// Problem: https://youtu.be/aPQY__2H3tE?t=606

#include <algorithm>
#include <iostream>
#include <map>
#include <queue>
#include <stack>
#include <string>
#include <vector>
#include <limits.h>
#include <stdio.h>
using namespace std;

using Box = std::tuple<int, int, int>;

void Print(const Box& box) {
    cout << std::get<0>(box) << " " << std::get<1>(box) << " " << std::get<2>(box);
}

void Print(const vector<Box>& boxes) {
    for (const auto& box : boxes) {
        Print(box); cout << endl;
    }
}

bool CanStack(const Box& a, const Box& b) {
    return std::get<0>(a) < std::get<0>(b) && std::get<1>(a) < std::get<1>(b);   
}

void PrintMap(const map<Box, int>& boxMaxHeight) {
    for (const auto& [box, h] : boxMaxHeight) {
        Print(box); cout << " => " << h << endl;
    }
}

int main()
{
    vector<Box> boxes = {
        std::make_tuple(4, 5, 3), 
        std::make_tuple(2, 3, 2), 
        std::make_tuple(3, 6, 2), 
        std::make_tuple(1, 5, 4), 
        std::make_tuple(2, 4, 1),
        std::make_tuple(1, 2, 2),
    };
    
    Print(boxes); cout << endl;
    
    std::sort(boxes.begin(), boxes.end(), [](const Box& a, const Box& b) {
        if (std::get<0>(a) == std::get<0>(b)) return std::get<1>(a) < std::get<1>(b);
        return std::get<0>(a) < std::get<0>(b);
    });
    
    Print(boxes); cout << endl;
    
    map<Box, int> boxMaxHeight;
    for (const auto& box : boxes) {
        boxMaxHeight[box] = std::get<2>(box);
    }
    
    PrintMap(boxMaxHeight); cout << endl;
    
    for (int i = 1; i < boxes.size(); ++i) {
        for (int j = 0; j < i; ++j) {
            if (CanStack(boxes[j], boxes[i])) {
                boxMaxHeight[boxes[i]] = max(boxMaxHeight[boxes[i]], boxMaxHeight[boxes[j]] + std::get<2>(boxes[i]));
            }
        }
    }
    
    PrintMap(boxMaxHeight); cout << endl;
    
    using pair_type = decltype(boxMaxHeight)::value_type;
    
    cout << "the answer is " 
         << std::max_element(boxMaxHeight.begin(), boxMaxHeight.end(), [](const pair_type& a, const pair_type& b) {
                 return a.second < b.second;        
            })->second;
    return 0;
}

// output
/*
4 5 3
2 3 2
3 6 2
1 5 4
2 4 1
1 2 2

1 2 2
1 5 4
2 3 2
2 4 1
3 6 2
4 5 3

1 2 2 => 2
1 5 4 => 4
2 3 2 => 2
2 4 1 => 1
3 6 2 => 2
4 5 3 => 3

1 2 2 => 2
1 5 4 => 4
2 3 2 => 4
2 4 1 => 3
3 6 2 => 6
4 5 3 => 7

the answer is 7
*/
```
