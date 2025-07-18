# C语言中的整数类型和数据转换

### **常见C语言类型大小**

|    有符号     |     无符号     | 32位 | 64位 |
| :-----------: | :------------: | :--: | :--: |
| [signed] char | unsigned char  |  1   |  1   |
|     short     | unsigned short |  2   |  2   |
|      int      |  unsigned int  |  4   |  4   |
|     long      | unsigned long  |  4   |  8   |
|    int32_t    |    uint32_t    |  4   |  4   |
|    int64_t    |    uint64_t    |  8   |  8   |
|     char*     |                |  4   |  8   |
|     float     |                |  4   |  4   |
|    double     |                |  8   |  8   |


### **左移和右移的两种方式**


逻辑左移（SHL）和算数左移(SAL)，规则相同，右边统一添0

逻辑右移(SHR)，左边**统一添0**

算数右移(SAR)，左边添加的数和符号有关 **(正数补0，负数补1)**

我们做一个简单的实验 ：

```c++
#include<bits/stdc++.h>
using namespace std;
void showBits(int num) {
	for(int i = 31 ; i >= 0 ; i--)
		cout << (num >> i & 1);
	cout << "\n";
}
signed main()
{
	int num = INT_MIN;
	showBits(num);
	showBits(num >> 1);
	showBits(num >> 10);
	return 0;
}
/*
10000000000000000000000000000000
11000000000000000000000000000000
11111111111000000000000000000000
*/
```

我们发现 C 族系都是默认是算数右移，所以有些时候会产生混乱， 所以像 JAVA 就有明确规定了

`>>` 算数右移 `>>>` 逻辑右移

**扩展：当移动位数大于实际位数时该怎么办？**

假设我们现在有一个 $$32$$ 位整数 $$x$$ ， 我们执行操作 `x >> 1234567` 

系统并不会直接将这个整数 $$x$$ 变成 $$0$$

相反的，计算机会将 $$1234567$$ 转化为 $$1234567 \bmod 32$$，

更一般的来讲： 对于一个 $$w$$ 位的数字， 移动 $$k$$ 位， 这个 $$k$$ 会被计算机隐式的转换为  $$k \bmod w$$



### **数字的拓展和截断**

**拓展**

​		要将一个无符号数转换为一个更大的数据类型，我们只要简单地在表示的开头添加 $$0$$。这种运算被称为**零扩展**（ zero extension）。

​		要将一个补码数字转换为一个更大的数据类型，可以执行一个**符号扩展**（ sign exten sion），即扩展符号位。举个例子 :  `100000101`  $$\Rightarrow$$ `1111111100000101`

​	更普遍的来说， 我们只需要对一个数进行算数右移即可完成拓展

**截断**

对于无符号整数来说， 如果你要从无符号编码的第 $$k$$ 位开始截断， 那么这个数就会变成对 $$2^{k}$$ 取模后的余数

简单来说就是  $$U \to U \bmod 2^k$$

对于补码的截断来说， 我们采用两步， 我们先把补码转化为无符号编码，之后再视作无符号进行截断，最后再转化为补码