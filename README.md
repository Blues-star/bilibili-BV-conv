# bilibili-BV-conv

> 转载自知乎[https://www.zhihu.com/question/381784377/answer/1099438784]


互相转换脚本，如果算法没猜错，可以保证在 av 号 ![[公式]](https://www.zhihu.com/equation?tex=%3C2%5E%7B27%7D) 时正确，同时应该在 ![[公式]](https://www.zhihu.com/equation?tex=%3C2%5E%7B30%7D) 时也是正确的。此代码以 WTFPL 开源。

UPD：之前的代码中，所有数位都被用到是乱凑的，实际上并不需要，目前只要低 6 位就足够了。（更大的 av 号需要 64 位整数存储，但是 b 站现在使用的应该还是 32 位整数，所以应该还要很久）



发现的方法：

首先从各种渠道的信息来看，应该是 base58 编码的。设 x 是一个钦定的 av 号，查询 ![[公式]](https://www.zhihu.com/equation?tex=58k%2Bx%2C58%5E2k%2Bx%2C58%5E3k%2Bx%2C58%5E4k%2Bx%5Cleft%28k%5Cin+Z%5Cright%29) 这些 av 号对应的 bv 号，发现 bv 号的第 12、11、4、9、5 位分别会变化。所以猜测这些是 58 进制下的相应位。

但是直接 base58 是不行的，所以猜测异或了一个大数，并且 base58 的字符表可能打乱了。经过实验，bv 号最低位相同的数，av 号的奇偶性相同，这一定程度上印证了之前的猜想。

接下来找了一些 av 号 x，满足 x 和 x+1 对应 bv 号的第 11 位不同。设异或的数为 X，那么 ![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%5Clfloor%5Cfrac%7BX%5Coplus+x%7D%7B58%7D%5Cright%5Crfloor%5Cneq+%5Cleft%5Clfloor%5Cfrac%7BX%5Coplus+%28x%2B1%29%7D%7B58%7D%5Cright%5Crfloor) （ ![[公式]](https://www.zhihu.com/equation?tex=%5Coplus)  表示异或）。

由于 av 号（除了最新的少量视频）最多只有 27 bits，所以可以设 ![[公式]](https://www.zhihu.com/equation?tex=X%3D2%5E%7B27%7Da%2Bb%280%5Cle+b%3C2%5E%7B27%7D%29) 。然后可以发现 ![[公式]](https://www.zhihu.com/equation?tex=X) 只和 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7Da%5Cbmod+58) 和 ![[公式]](https://www.zhihu.com/equation?tex=b) 有关，那么可以枚举这两个值（一共 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7D%5Ccdot+58%3D7784628224) 种情况）然后使用上面的式子检查，就能得到若干可能的 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7Da%5Cbmod+58) 和 ![[公式]](https://www.zhihu.com/equation?tex=b) 。

这里我得到的可能值如下：（左边是 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7Da%5Cbmod+58) ，右边是 ![[公式]](https://www.zhihu.com/equation?tex=b) ）

```text
22 90983642
22 90983643
50 43234084
50 43234085
```

有奇有偶是因为异或 1 之后也能找到轮换表。而 ![[公式]](https://www.zhihu.com/equation?tex=90983642%2B43234085%3D2%5E%7B27%7D-1) 则使得模 58 的余数刚好变成 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7D-1) 减它。

我取了 b=43234084，然后处理最低位，可以得到一个字符表，即 fZodR9XQDSUm21yCkr6zBqiveYah8bt4xsWpHnJE7jL5VG3guMTKNPAwcF)。

对于更高位，实际上还需要知道 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7Da%5Cbmod+58%5E2%2C2%5E%7B27%7Da%5Cbmod+58%5E3%2C%5Cdots) ，这些值也可以 枚举 58 次得到，最后我得到的值是 ![[公式]](https://www.zhihu.com/equation?tex=2%5E%7B27%7Da%5Cbmod+58%5E4%3D1749968) 。

这时我发现，每一位的字符表是相同的（实际上只对 b=43234084 是这样的），然后再微调一下参数（上面代码中的两个 magic number 就相当于这里的 ![[公式]](https://www.zhihu.com/equation?tex=a%2Cb) ），最后处理了一下 ![[公式]](https://www.zhihu.com/equation?tex=%5Cge+2%5E%7B27%7D) 的情况就得到了这份代码。