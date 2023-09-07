﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿﻿# ﻿﻿﻿ACM

[华为机试](https://www.nowcoder.com/exam/oj/ta?page=1&tpId=37&type=37)

> `#include <bits/stdc++.h>` 万能头文件

* cin以空格、换行符、tab为间隔读取数据
* getline可以以指定的结束符结束，默认结束符是换行符

<img src="E:\MarkDown\picture\image-20230401202705924.png" alt="image-20230401202705924" style="zoom:50%;" />





<img src="E:\MarkDown\picture\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aOO6L-Y5aW95Ya3,size_20,color_FFFFFF,t_70,g_se,x_16.png" alt="img" style="zoom:80%;" />

读取'\n'，可以用cin.get()，也可以用getchar()

# 牛客ACM练习

https://ac.nowcoder.com/acm/contest/5657

## 整数和

### 多组空格分隔的两个正整数 

输入描述：

```
输入包括两个正整数a,b(1 <= a, b <= 1000),输入数据包括多组。
```

输出描述：

```
输出 a+b的结果
```

示例1

输入例子：

```
1 5
10 20
```

输出例子：

```
6
30
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int a, b;
    while (cin >> a >> b) { // 注意 while 处理多个 case
        // 64 位输出请用 printf("%lld")
        cout << a + b << endl;
    }
}
```



### 第一行组数接空格分隔的两个正整数

输入描述：

```
输入第一行包括一个数据组数t(1 <= t <= 100)
接下来每行包括两个正整数a,b(1 <= a, b <= 1000)
```

输出描述：

```
输出a+b的结果
```

示例1

输入例子：

```
2
1 5
10 20
```

输出例子：

```
6
30
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int a, b, c;
    cin >> a;
    while (cin >> b >> c) { // 注意 while 处理多个 case
        // 64 位输出请用 printf("%lld")
        cout << b + c << endl;
    }
}
```



### 空格分隔的两个正整数 为0时结束

输入描述：

```
输入包括两个正整数a,b(1 <= a, b <= 10^9),输入数据有多组, 如果输入为0 0则结束输入
```

输出描述：

```
输出a+b的结果
```

示例1

输入例子：

```
1 5
10 20
0 0
```

输出例子：

```
6
30
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int a, b;
    while (cin >> a >> b) { // 注意 while 处理多个 case
        if (a == 0 && b == 0) break;
        cout << a + b << endl;
    }
}
```



### 多组首位为数据个数的组合 为0结束

输入描述：

```
输入数据包括多组。
每组数据一行,每行的第一个整数为整数的个数n(1 <= n <= 100), n为0的时候结束输入。
接下来n个正整数,即需要求和的每个正整数。
```

输出描述：

```
每组数据输出求和的结果
```

示例1

输入例子：

```
4 1 2 3 4
5 1 2 3 4 5
0
```

输出例子：

```
10
15
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int a;
    while (cin >> a) { // 注意 while 处理多个 case
        int sum = 0;
        int b;
        if (a == 0) break;
        while((a--) && cin >> b) {
            sum += b;
        }
        cout << sum << endl;
    }
}
```



### 5.第一行组数接第一个个数接空格分开的整数

输入描述：

```
输入的第一行包括一个正整数t(1 <= t <= 100), 表示数据组数。
接下来t行, 每行一组数据。
每行的第一个整数为整数的个数n(1 <= n <= 100)。
接下来n个正整数, 即需要求和的每个正整数。
```

输出描述：

```
每组数据输出求和的结果
```

示例1

输入例子：

```
2
4 1 2 3 4
5 1 2 3 4 5
```

输出例子：

```
10
15
```

```cpp
#include <iostream>
using namespace std;

int main() {
    int num;
    cin >> num;
    while (num--) {
        int a, b;
        int sum = 0;
        cin >> a;
        while (a-- && cin >> b) {
            sum += b;
        }
        cout << sum << endl;
    }
}
```

### 6.每行第一个为个数后带空格分割整数

输入描述：

```
输入数据有多组, 每行表示一组输入数据。
每行的第一个整数为整数的个数n(1 <= n <= 100)。
接下来n个正整数, 即需要求和的每个正整数。
```

输出描述：

```
每组数据输出求和的结果
```

示例1

输入例子：

```
4 1 2 3 4
5 1 2 3 4 5
```

输出例子：

```
10
15
```



```cpp
#include <iostream>
using namespace std;

int main() {
    int a;
    while (cin >> a) { // 注意 while 处理多个 case
        int b;
        int sum = 0;
        while (a-- && cin >> b) {
            sum += b;
        }
        cout << sum << endl;
    }
}
```

### ==7.多组空格分隔的正整数==

输入描述：

```
输入数据有多组, 每行表示一组输入数据。

每行不定有n个整数，空格隔开。(1 <= n <= 100)。
```

输出描述：

```
每组数据输出求和的结果
```

示例1

输入例子：

```
1 2 3
4 5
0 0 0 0 0
```

输出例子：

```
6
9
0
```



```cpp
#include <iostream>
using namespace std;

int main() {
    int a;
    while (cin >> a) { // 注意 while 处理多个 case
        int sum = a;
        int b;
        // 如果不是换行符的话，读到的是数字后面的空格或者table，被getchar吃掉
        while (getchar() != '\n' && cin >> b) {
            // cin >> b;  // cin也可以放在这
            sum += b;
        }
        // l
        cout << sum << endl;
    }
}

// 写法2
#include <iostream>
using namespace std;

int main() {
    int a;
    int sum = 0;
    while (cin >> a) { // 注意 while 处理多个 case
        sum += a;

        if (cin.get() == '\n') {
            cout << sum << endl;
            sum = 0;
        }
    }
}
```

对于写法1和2，如果输入的一行中最后一个数字后面是空格+换行的话，那么就会导致该行没有输出数据，举例说明，输入1 2 ‘\n’，并没有输出1+2的和

因为cin并不会读取空格，在读取到2时，get()判断下一位并不是\n，所以此时并不会输出数字，但是下一次循环时cin只会跳过空格和结束符，继续读取下一行的数




### 注意数据范围

输入描述：

```
输入有多组测试用例，每组空格隔开两个整数
```

输出描述：

```
对于每组数据输出一行两个整数的和
```

示例1

输入例子：

```
1 1
```

输出例子：

```
2
```

这里虽然a和b都没有越界，但其和越界了！！！

```cpp
#include <iostream>
using namespace std;
int main() {
    // 不要用 int a, b, 因为测试数据会越界，为了效果，所以这个题目故意不在题面说数据范围
    // 不要用 int a, b, 因为测试数据会越界，为了效果，所以这个题目故意不在题面说数据范围
    // 不要用 int a, b, 因为测试数据会越界，为了效果，所以这个题目故意不在题面说数据范围
    // 不要用 int a, b, 因为测试数据会越界，为了效果，所以这个题目故意不在题面说数据范围
    // 你可以试试测试用例 12141483647 12141483647，输出结果是否正确
    long long a,b;
    while(cin >> a >> b)// 注意，如果输入是多个测试用例，请通过while循环处理多个测试用例
        cout << a+b << endl;
}
```



## 字符串排序

### 第一行个数第二行字符串

输入描述：

```
输入有两行，第一行n

第二行是n个字符串，字符串之间用空格隔开
```

输出描述：

```
输出一行排序后的字符串，空格隔开，无结尾空格
```

示例1

输入例子：

```
5
c d a bb e
```

输出例子：

```
a bb c d e
```



```cpp
#include <iostream>
#include<vector>
#include<algorithm>
using namespace std;

int main() {
    int a;
    cin >> a;
    vector<string> vec(a);
    for (int i = 0; i < a; i++) {
		cin >> vec[i];
    }
    sort(vec.begin(), vec.end());
    for (auto i : vec) {
        cout << i << ' ';
    }
}

// 写法2
#include<iostream>
#include<set>
using namespace std;

int main(){
    int n;
    cin >> n;
    set<string> m;
    while(n--){
        string s;
        cin >> s;
        m.insert(s);
    }
    for(auto a:m){
        cout << a << " ";
    }
}

```

### 多行空格分开的字符串

输入描述：

```
多个测试用例，每个测试用例一行。

每行通过空格隔开，有n个字符，n＜100
```

输出描述：

```
对于每组测试用例，输出一行排序过的字符串，每个字符串通过空格隔开
```

示例1

输入例子：

```
a c bb
f dddd
nowcoder
```

输出例子：

```
a bb c
dddd f
nowcoder
```



```cpp
#include <iostream>
#include<set>
using namespace std;

int main() {
    string s;
    set<string> st;  // vector
    while (cin >> s) { // 注意 while 处理多个 case
        st.insert(s);
        // getchar()
        if (cin.get() == '\n') {
            for (auto i : st) {
                cout << i << " ";
            }
            st.clear();
            cout << endl;
        }
    }
}
```

### ==多行逗号分开的字符串==

输入描述：

```
多个测试用例，每个测试用例一行。
每行通过,隔开，有n个字符，n＜100
```

输出描述：

```
对于每组用例输出一行排序后的字符串，用','隔开，无结尾空格
```

示例1

输入例子：

```
a,c,bb
f,dddd
nowcoder
```

输出例子：

```
a,bb,c
dddd,f
nowcoder
```



```cpp
// #include <bits/stdc++.h>  // 也可以直接使用这个万能头文件
#include <iostream>
#include<sstream>
#include<vector>
#include<algorithm>
using namespace std;

int main() {
    string s;
    vector<string> vec;
    while (cin >> s) {  // 注意，这里s读取的是一整行（cin遇空格或换行才停止）
       
        stringstream ss(s);  // 将s变为stringstream对象
        string tmp = "";
        while(getline(ss,tmp,',')) {  // 以
            vec.push_back(tmp);
        }
        sort(vec.begin(), vec.end());
        for (int i = 0; i < vec.size(); i++) {
            cout << vec[i];
            if (i != vec.size() - 1) cout << ",";
        }
        cout << endl;
        vec.clear();
    }
}
```





输入描述：

```
输入有多组测试用例，每组空格隔开两个整数
```

输出描述：

```
对于每组数据输出一行两个整数的和
```

示例1

输入例子：

```
1 1
```

输出例子：

```
2
```



# 二叉树

```cpp
#include <iostream>
#include <vector>
using namespace std;

struct TreeNode {
    int val;
    TreeNode *left;
    TreeNode *right;
    TreeNode(int x) : val(x), left(NULL), right(NULL) {}
};

// 根据数组构造二叉树
TreeNode* construct_binary_tree(const vector<int>& vec) {
    vector<TreeNode*> vecTree (vec.size(), NULL);
    TreeNode* root = NULL;
    // 把输入数值数组，先转化为二叉树节点数组
    for (int i = 0; i < vec.size(); i++) {
        TreeNode* node = NULL;
        if (vec[i] != -1) node = new TreeNode(vec[i]); 
        vecTree[i] = node;
        if (i == 0) root = node;
    }
           
    for (int i = 0; i * 2 + 1 < vec.size(); i++) {
        if (vecTree[i] != NULL) {
            // 线性存储转连式存储关键逻辑
            vecTree[i]->left = vecTree[i * 2 + 1];
            if(i * 2 + 2 < vec.size()) vecTree[i]->right = vecTree[i * 2 + 2];
        }
    }
    return root;
}

int main() {
    // 注意本代码没有考虑输入异常数据的情况
    // 用 -1 来表示null
    int a;
    while (cin >> a) {
	
    }
    vector<int> vec = {4,1,6,0,2,5,7,-1,-1,-1,3,-1,-1,-1,8};
    TreeNode* root = construct_binary_tree(vec);
    
    //  处理逻辑
}

```

# 华为机试

[华为机试](https://www.nowcoder.com/exam/oj/ta?tpId=37)

### **HJ2** **计算某字符出现次数**

**描述**

写出一个程序，接受一个由字母、数字和**空格**组成的字符串，和一个字符，然后输出输入字符串中该字符的出现次数。（不区分大小写字母）

数据范围：1≤*n*≤1000 

**输入描述**：

第一行输入一个由字母、数字和空格组成的字符串，第二行输入一个字符（保证该字符不为空格）。

**输出描述**：

输出输入字符串中含有该字符的个数。（不区分大小写字母）

**示例**1

输入：

```
ABCabc
A
```

复制

输出：

```
2
```

```cpp
#include <iostream>
using namespace std;

int main() {
    string a;
    // getline默认结束符是换行符
    getline(cin, a);
    char b;
    cin >> b;
    int result = 0;
    for (int i = 0; i < a.size(); i++) {
        if (a[i] == b) result++;
        // b为字母
        else if (b >= 65 && abs(a[i] - b) == 32) result++;
    }
    cout << result;
    return 0;
}
```

# 洛谷

https://www.luogu.com.cn/training/list
