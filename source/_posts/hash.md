---
title: 哈希算法究竟是个什么鬼
date: 2018-10-19 13:28:01
categories: 算法与数据结构
tags: [哈希算法, 算法]
---
> 一直以来写博客都是抱着给别人看的心态写的，其实，别人看了又怎样，重要的是自己写的过程。这几天终于想明白了这个道理，但是博客该有的套路还是得一个不少的加上。最近感觉写Java方面的知识有些厌倦了，于是干脆写点自己想写的。

<!-- more -->

曾经面试被问到过哈希算法，而曾经的我什么都不懂，就在那儿乱扯一通，现在看来那个时候的自己真的好笑。于是准备补上这个坑，搜集汇总哈希算法的相关只是于此。

# 一、前言

我们如何比较两个文件是否一致呢？

将文件读成二进制流，然后比较两个文件的二进制流？如果二进制流一开始就不一样还好说，或者是文件小都好说。但是如果是大文件呢？如果文件最后几位二进制不一样，这怎么办？耗时太久。

这个时候我们就需要一个高效而可靠办法，给每个文件一个唯一的ID，然后直接比较两个ID，这貌似是一个不错的方法。嗯，想一想很不错。但是如何确定两个文件的ID不同呢？这时候就可以用到HASH方法了。

# 二、简介

**散列算法（Hash Algorithm）**，又称哈希算法，杂凑算法，是一种从任意文件中创造小的数字「指纹」的方法。与指纹一样，散列算法就是一种以较短的信息来保证文件唯一性的标志，这种标志与文件的每一个字节都相关，而且难以找到逆向规律。因此，当原有文件发生改变时，其标志值也会发生改变，从而告诉文件使用者当前的文件已经不是你所需求的文件。


# 三、应用

哈希算法，目前在信息安全领域主要用到以下几个方面：

* 文件校验

我们常见的很多文件下载的时候，除了能够下载具体的文件外，还有很多会给出文件的MD5码。这就是用于文件校验使用的。因为在传统的加密渠道中，我们只能对文件传输过程中的信道进行容错处理，但是不能对抗恶意的文件篡改问题。所以如果有了MD5码，我们就能够通过MD5码，校验我们收到的问题件是否是未经篡改的文件。
  
* 数字签名
  
Hash 算法也是现代password体系中的一个重要组成部分。因为非对称算法的运算速度较慢，所以在数字签名协议中，单向散列函数扮演了一个重要的角色。 对 Hash 值，又称"数字摘要"进行数字签名，在统计上能够觉得与对文件本身进行数字签名是等效的。
  
* 鉴权协议
  
当数据在传输信道是可被侦听，但不可被篡改的情况下，这是一种简单而安全的方法。

# 四、实现

Hash算法最主要的目的就是为了将一个大范围映射到一个小范围。将大范围映射到小范围是为了节省空间。另外，还要使Hash值足够唯一，这样ID才具有唯一性。除此之外，Hash算法还得具有单向性。

具体说来，Hash算法应该具有以下几个限制点：

* Hash的主要原理就是把大范围映射到小范围；所以，你输入的实际值的个数必须和小范围相当或者比它更小。不然冲突就会非常多。
* Hash逼近单向函数；所以，你能够用它来对数据进行加密。
* 不同的应用对Hash函数有着不同的要求；比方，用于加密的Hash函数主要考虑它和单项函数的差距，而用于查找的Hash函数主要考虑它映射到小范围的冲突率。

明白了这几点，我们就能够对Hash函数的实现做一些了解了。总的说来，目前主流的Hash算法有以下几种实现方法：

## 1. 加法Hash

所谓的加法Hash就是把输入元素一个一个的加起来构成最后的结果。标准的加法Hash的构造例如以下：

```
static int additiveHash(String key, int prime){
    int hash, i;
    for (hash = key.length(), i = 0; i < key.length(); i++)
    hash += key.charAt(i);
    return (hash % prime);
}

```

这里的prime是随意的质数，看得出，结果的值域为[0,prime-1]。

## 2. 位运算Hash

这类型Hash函数通过利用各种位运算（常见的是移位和异或）来充分的混合输入元素。比方，标准的旋转Hash的构造例如以下：

```
static int rotatingHash(String key, int prime) {
    int hash, i;
    for (hash=key.length(), i=0; i<key.length(); ++i)
        hash = (hash<<4)^(hash>>28)^key.charAt(i);
    return (hash % prime);
 }
```

先移位，然后再进行各种位运算是这样的类型Hash函数的主要特点。比方，以上的那段计算hash的代码还能够有例如以下几种变形：

* 
```
hash = (hash<<5)^(hash>>27)^key.charAt(i);
```
* 
```
hash += key.charAt(i);
hash += (hash << 10);
hash ^= (hash >> 6);
```
* 
```
if((i&1) == 0){
    hash ^= (hash<<7) ^ key.charAt(i) ^ (hash>>3);
} else {
    hash ^= ~((hash<<11) ^ key.charAt(i) ^ (hash >>5));
}
```
*    
```
hash += (hash<<5) + key.charAt(i);
```
*       
```
hash = key.charAt(i) + (hash<<6) + (hash>>16) – hash;
```
*     
```
hash ^= ((hash<<5) + key.charAt(i) + (hash>>2));
```

## 3. 乘法Hash

这种类型的Hash函数利用了乘法的不相关性（乘法的这种性质，最有名的莫过于平方取头尾的随机数生成算法，虽然这种算法效果并不好）。比如，

```
static int bernstein(String key) {
    int hash = 0;
    int i;
    for (i=0; i<key.length(); ++i) 
        hash = 33*hash + key.charAt(i);
    return hash;
}
```

jdk5.0里面的String类的hashCode()方法也使用乘法Hash。不过，它使用的乘数是31。推荐的乘数还有：131, 1313, 13131, 131313等等。

使用这种方式的著名Hash函数还有：

* 32位FNV算法
 
```
int M_SHIFT = 0;
public int FNVHash(byte[] data) {
    int hash = (int)2166136261L;
    for(byte b : data)
        hash = (hash * 16777619) ^ b;
    if (M_SHIFT == 0)
        return hash;
    return (hash ^ (hash >> M_SHIFT)) & M_MASK;
}
```

* 以及改进的FNV算法：

```
public static int FNVHash1(String data) {
    final int p = 16777619;
    int hash = (int)2166136261L;
    for(int i=0;i<data.length();i++)
        hash = (hash ^ data.charAt(i)) * p;
    hash += hash << 13;
    hash ^= hash >> 7;
    hash += hash << 3;
    hash ^= hash >> 17;
    hash += hash << 5;
    return hash;
}
```
    
除了乘以一个固定的数，常见的还有乘以一个不断改变的数，比如：

```
static int RSHash(String str){
    int b = 378551;
    int a = 63689;
    int hash = 0;
    for(int i = 0; i < str.length(); i++){
        hash = hash * a + str.charAt(i);
        a = a * b;
    }
    return (hash & 0x7FFFFFFF);
}
```

虽然Adler32算法的应用没有CRC32广泛，不过，它可能是乘法Hash里面最有名的一个了。关于它的介绍，大家可以去看RFC 1950规范。

## 4. 除法Hash

除法和乘法一样，同样具有表面上看起来的不相关性。不过，因为除法太慢，这种方式几乎找不到真正的应用。需要注意的是，我们在前面看到的hash的 结果除以一个prime的目的只是为了保证结果的范围。如果你不需要它限制一个范围的话，可以使用如下的代码替代”hash%prime”： 

```
hash = hash ^ (hash>>10) ^ (hash>>20)
```

## 5. 查表Hash

查表Hash最有名的样例莫过于CRC系列算法。尽管CRC系列算法本身并非查表，可是，查表是它的一种最快的实现方式。以下是CRC32的实现：

```
static int crctab[256] = {
0x00000000, 0x77073096, 0xee0e612c, 0x990951ba, 0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3, 
0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988, 0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91, 
0x1db71064, 0x6ab020f2, 0xf3b97148, 0x84be41de, 0x1adad47d, 0x6ddde4eb, 0xf4d4b551, 0x83d385c7, 
0x136c9856, 0x646ba8c0, 0xfd62f97a, 0x8a65c9ec, 0x14015c4f, 0x63066cd9, 0xfa0f3d63, 0x8d080df5, 
0x3b6e20c8, 0x4c69105e, 0xd56041e4, 0xa2677172, 0x3c03e4d1, 0x4b04d447, 0xd20d85fd, 0xa50ab56b, 
0x35b5a8fa, 0x42b2986c, 0xdbbbc9d6, 0xacbcf940, 0x32d86ce3, 0x45df5c75, 0xdcd60dcf, 0xabd13d59, 
0x26d930ac, 0x51de003a, 0xc8d75180, 0xbfd06116, 0x21b4f4b5, 0x56b3c423, 0xcfba9599, 0xb8bda50f, 
0x2802b89e, 0x5f058808, 0xc60cd9b2, 0xb10be924, 0x2f6f7c87, 0x58684c11, 0xc1611dab, 0xb6662d3d, 
0x76dc4190, 0x01db7106, 0x98d220bc, 0xefd5102a, 0x71b18589, 0x06b6b51f, 0x9fbfe4a5, 0xe8b8d433, 
0x7807c9a2, 0x0f00f934, 0x9609a88e, 0xe10e9818, 0x7f6a0dbb, 0x086d3d2d, 0x91646c97, 0xe6635c01, 
0x6b6b51f4, 0x1c6c6162, 0x856530d8, 0xf262004e, 0x6c0695ed, 0x1b01a57b, 0x8208f4c1, 0xf50fc457, 
0x65b0d9c6, 0x12b7e950, 0x8bbeb8ea, 0xfcb9887c, 0x62dd1ddf, 0x15da2d49, 0x8cd37cf3, 0xfbd44c65, 
0x4db26158, 0x3ab551ce, 0xa3bc0074, 0xd4bb30e2, 0x4adfa541, 0x3dd895d7, 0xa4d1c46d, 0xd3d6f4fb, 
0x4369e96a, 0x346ed9fc, 0xad678846, 0xda60b8d0, 0x44042d73, 0x33031de5, 0xaa0a4c5f, 0xdd0d7cc9, 
0x5005713c, 0x270241aa, 0xbe0b1010, 0xc90c2086, 0x5768b525, 0x206f85b3, 0xb966d409, 0xce61e49f, 
0x5edef90e, 0x29d9c998, 0xb0d09822, 0xc7d7a8b4, 0x59b33d17, 0x2eb40d81, 0xb7bd5c3b, 0xc0ba6cad, 
0xedb88320, 0x9abfb3b6, 0x03b6e20c, 0x74b1d29a, 0xead54739, 0x9dd277af, 0x04db2615, 0x73dc1683, 
0xe3630b12, 0x94643b84, 0x0d6d6a3e, 0x7a6a5aa8, 0xe40ecf0b, 0x9309ff9d, 0x0a00ae27, 0x7d079eb1, 
0xf00f9344, 0x8708a3d2, 0x1e01f268, 0x6906c2fe, 0xf762575d, 0x806567cb, 0x196c3671, 0x6e6b06e7, 
0xfed41b76, 0x89d32be0, 0x10da7a5a, 0x67dd4acc, 0xf9b9df6f, 0x8ebeeff9, 0x17b7be43, 0x60b08ed5, 
0xd6d6a3e8, 0xa1d1937e, 0x38d8c2c4, 0x4fdff252, 0xd1bb67f1, 0xa6bc5767, 0x3fb506dd, 0x48b2364b, 
0xd80d2bda, 0xaf0a1b4c, 0x36034af6, 0x41047a60, 0xdf60efc3, 0xa867df55, 0x316e8eef, 0x4669be79, 
0xcb61b38c, 0xbc66831a, 0x256fd2a0, 0x5268e236, 0xcc0c7795, 0xbb0b4703, 0x220216b9, 0x5505262f, 
0xc5ba3bbe, 0xb2bd0b28, 0x2bb45a92, 0x5cb36a04, 0xc2d7ffa7, 0xb5d0cf31, 0x2cd99e8b, 0x5bdeae1d, 
0x9b64c2b0, 0xec63f226, 0x756aa39c, 0x026d930a, 0x9c0906a9, 0xeb0e363f, 0x72076785, 0x05005713, 
0x95bf4a82, 0xe2b87a14, 0x7bb12bae, 0x0cb61b38, 0x92d28e9b, 0xe5d5be0d, 0x7cdcefb7, 0x0bdbdf21, 
0x86d3d2d4, 0xf1d4e242, 0x68ddb3f8, 0x1fda836e, 0x81be16cd, 0xf6b9265b, 0x6fb077e1, 0x18b74777, 
0x88085ae6, 0xff0f6a70, 0x66063bca, 0x11010b5c, 0x8f659eff, 0xf862ae69, 0x616bffd3, 0x166ccf45, 
0xa00ae278, 0xd70dd2ee, 0x4e048354, 0x3903b3c2, 0xa7672661, 0xd06016f7, 0x4969474d, 0x3e6e77db, 
0xaed16a4a, 0xd9d65adc, 0x40df0b66, 0x37d83bf0, 0xa9bcae53, 0xdebb9ec5, 0x47b2cf7f, 0x30b5ffe9, 
0xbdbdf21c, 0xcabac28a, 0x53b39330, 0x24b4a3a6, 0xbad03605, 0xcdd70693, 0x54de5729, 0x23d967bf, 
0xb3667a2e, 0xc4614ab8, 0x5d681b02, 0x2a6f2b94, 0xb40bbe37, 0xc30c8ea1, 0x5a05df1b, 0x2d02ef8d
};
int crc32(String key, int hash) {
    int i;
    for (hash=key.length(), i=0; i<key.length(); ++i)
        hash = (hash >> 8) ^ crctab[(hash & 0xff) ^ k.charAt(i)];
    return hash;
}
```

查表Hash中有名的样例有：Universal Hashing和Zobrist Hashing。他们的表格都是随机生成的。

## 6. 混合Hash

混合Hash算法利用了以上各种方式。各种常见的Hash算法，比如MD5、Tiger都属于这个范围。它们一般很少在面向查找的Hash函数里面使用。

## 7. 数组Hash

```
inline int hashcode(const int *v) {
    int s = 0;
    for(int i=0; i<k; i++)
        s=((s<<2)+(v[i]>>4))^(v[i]<<10);
    s = s % M;
    s = s < 0 ? s + M : s;
    return s;
}
```

# 五、碰撞

通过前面说到的几种Hash算法的实现，我们可以发现，Hash算法虽然很多好，但是无法保证两个片段产生的Hash值不能重复。Hash算法产生冲突的情况，称之为Hash冲突，也称之为Hash碰撞。

通过构造性能良好的Hash算法，可以减少冲突，但一般不可能完全避免冲突，因此解决冲突是Hash算法的另一个关键问题。目前解决Hash冲突的主流方法大概有以下几种：

## 1. 开放寻址法(再散列法)

开放寻址法的基本思想是：当关键字key的哈希地址p=H（key）出现冲突时，以p为基础，产生另一个哈希地址p1，如果p1仍然冲突，再以p为基础，产生另一个哈希地址p2，…，直到找出一个不冲突的哈希地址pi ，将相应元素存入其中。这种方法有一个通用的再散列函数形式：
            
Hi=（H（key）+di）% m   i=1，2，…，n
            
其中H（key）为哈希函数，m 为表长，di称为增量序列。

## 2. 再哈希法

这种方法的基本思想是：同时构造多个不同的哈希函数：

Hi=RH1（key）  i=1，2，…，k

当哈希地址Hi=RH1（key）发生冲突时，再计算Hi=RH2（key）……，直到冲突不再产生。这种方法不易产生聚集，但增加了计算时间。

## 3. 链地址法（拉链法）

这种方法的基本思想是：将所有哈希地址为i的元素构成一个称为同义词链的单链表，并将单链表的头指针存在哈希表的第i个单元中，因而查找、插入和删除主要在同义词链中进行。链地址法适用于经常进行插入和删除的情况。

## 4. 建立公共溢出区 

这种方法的基本思想是：将哈希表分为基本表和溢出表两部分，凡是和基本表发生冲突的元素，一律填入溢出表。

目前用的比较多的就是 **开放寻址法** 和 **拉链法** ,针对 开放寻址法 和 拉链法 其具体的优缺点如下：

### 拉链法

* 优点：

1.避免了动态调整的开销

2.由于底层是链表结构，所以尤其适合那种记录本身尺寸（size）很大的情况，因为此时指针的开销可以忽略不计了

3.同样也是因为链表结构，删除记录时，比较方便，直接通过指针操作即可

* 缺点：

1.由于链表结构，所以查询比较耗时

2.由于链表结构，不利于序列化操作

### 开放寻址法

* 优点

1.记录更容易进行序列化（serialize）操作

* 缺点

1.存储记录的数目不能超过桶数组的长度，如果超过就需要扩容，而扩容会导致某次操作的时间成本飙升

2.使用探测序列，有可能其计算的时间成本过高，导致哈希表的处理性能降低 

3.由于记录是存放在桶数组中的，而桶数组必然存在空槽，所以当记录本身尺寸（size）很大并且记录总数规模很大时，空槽占用的空间会导致明显的内存浪费 

4.删除记录时，比较麻烦。比如需要删除记录a，记录b是在a之后插入桶数组的，但是和记录a有冲突，是通过探测序列再次跳转找到的地址，所以如果直接删除a，a的位置变为空槽，而空槽是查询记录失败的终止条件，这样会导致记录b在a的位置重新插入数据前不可见，所以不能直接删除a，而是设置删除标记。这就需要额外的空间和操作。

**总结：拉链法适合数据量不可预知，且写入多，查询少的情况。开放寻址法适合数据量可预知，查询多，写入少的情况。**

在传统的JDK中，采用的都是拉链法，自JDK1.8起，对拉链法做了改造。当链表长度超过预定值后将采用红黑树结构替代链表结构，这样对查询操作做了更好的优化。具体可参考我之前写的关于HashMap的文章 [Java集合-0——HashMap类](/2018/07/30/Java集合-0——HashMap类/)


**PS: 文中对Hash算法实现的代码均源于网络，若有侵权，请联系删除 [原文链接](https://blog.csdn.net/tanga842428/article/details/79850832)**