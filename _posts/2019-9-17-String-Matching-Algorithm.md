---
title: "字符串匹配算法"
categories:
  - Algorithm
tags:
  - Algorithm
  - Java
  - String
classes: wide
toc: true
toc_sticky: true
---

# String Matching Algorithm(字符串匹配算法)

>在编辑文本程序的过程中，我们经常需要在文本中找到某个模式的所有出现位置。有效地解决这个问题的算法叫做字符串匹配算法。我在此记录了3种常见的字符串匹配算法供自己留作记录参考，分别是从最简单的暴力求解的朴素算法入手，至两个优化过复杂度的算法Hospool和KMP。

## Naïve Pattern Searching Algorithm
>一个简单的 Brute Force 的寻求子字符串的匹配解法。

### 1.算法说明
在Text上不断滑动 Pattern，如果不匹配，继续滑动 Pattern 到下一个 Index，如果匹配，返回Text上第一个匹配到 Pattern 的起始位置。

### 2.算法表现
Text: 长度 N; Pattern: 长度 M; <br>
Worst Case: 总共比较次数为 ((N-M)+1)*M <br>
Average: *Θ(N \* M)* <br>

```java
public static List<Integer> naiveMatching(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    // 一个循环在Text上一个一个Index的进行滑动
    for (int i = 0; i <= text.length() - pattern.length(); i++) {
        // 在当前Index，检查Pattern是否匹配Text
        for (int j = 0; j < pattern.length(); j++) {
            if (text.charAt(i + j) != pattern.charAt(j)) break;
            if (j == pattern.length() - 1) result.add(i);
        }
    }
    return result;
}
```


## Boyer–Moore–Horspool algorithm (Horspool's algorithm)

> 一个用空间换取时间来获取一个 Average-case Complexity O(n) 的子字符串匹配算法。

### 1.算法说明 
1. **首先处理 Pattern，建立一个 Shift Table**

   + Shift Table 理论上应该包含所有字符，Java 用了Unicode 集，一个 char 可以包含65536个不同字符，一般情况，只做字母的匹配话，包含字母即可，我折中选取了 ASCII 表，使 Shift Table 包含128个元素，也方便了在 Java 中 char 直接转换为在 ASCII 中对应的 Index。

   + 对 Shift Table 中的元素（**除最后一位末尾**）移动距离的构建：
   ①非在 Pattern 中出现的 char，则移动位置为 Pattern 长度。
   ②在 Pattern 中出现的 char，根据当前 char 距离最后一个元素的距离决定，如果有重复的元素,则右边的元素覆盖左边的元素，以右边距离最后一个元素的距离为准。

   + 直观上对移动 Pattern 距离的理解，就是以 Pattern 末尾元素对应的 Text 中的元素 TAIL 为基准，
   ①如果 TAIL 没出现在 Pattern 末尾之前的元素里，则把整个 Pattern 往后移动整个 Pattern 的距离
   ②如果 TAIL 出现在 Pattern 末尾之前的元素里，则把 Pattern 移动到使最靠近末尾的和 TAIL 相同的元素和 TAIL 相对应的位置上。

2. **Pattern 的检索过程**
   首先将 Pattern 放在 Text 最开始位置，然后每次从 Pattern 的最右端（即末尾）开始进行匹配，如果不匹配，根据 Pattern 末尾元素对应的 Text 中的元素在 Shift Table 中的 Value 进行移动 Pattern.直到找到匹配位置，或者检索完整个 Text。

   

### 2.算法表现

Text: 长度 N; Pattern: 长度 M; <br>
Worst Case: *O(N \* M)* Best Case: *O(N / M)* <br>
Average Case: *O(N)* <br>

**引用**

[YouTube用户Mike Slade讲解](https://www.youtube.com/watch?v=PHXAOKQk2dw) <br>
[Wikipidea Boyer–Moore–Horspool_algorithm](https://www.wikiwand.com/en/Boyer%E2%80%93Moore%E2%80%93Horspool_algorithm)




```java
public static List<Integer> holspoolMatching(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    //构建移动Pattern距离的一个移位表，只考虑ASCII编码的128位
    //Java用了Unicode集，前128位即采用了ASCII编码
    int[] shiftTable = buildShiftTable(pattern);
    int searchIndex = 0;
    while (searchIndex <= text.length() - pattern.length()) {
        //从pattern右侧开始往左侧匹配
        int patternIndex = pattern.length() - 1;
        while (text.charAt(searchIndex + patternIndex) == pattern.charAt(patternIndex)) {
            //如果匹配到Pattern的头部，说明完全匹配，则返回Text中相对应的Index，
            if (patternIndex == 0) result.add(searchIndex);
            //不断往左匹配
            patternIndex--;
        }
        //根据起始匹配位置（即Pattern最右侧匹配的对应)的那个Text中的char来决定右移的长度
        searchIndex += shiftTable[text.charAt(searchIndex + pattern.length() - 1)];
    }
    return result;
}


/**
 * 构建移动 Pattern 距离的一个移位表，只考虑 ASCII 编码的128位
 * Java 用了 Unicode 集，前128位即采用了 ASCII 编码
 *
 * @param pattern 需要匹配的子字符串
 * @return 根据Pattern来构建的Shift Table
 */
private static int[] buildShiftTable(String pattern) {
    int[] shiftTable = new int[128];
    //非在Pattern中出现的char，则移动位置为Pattern长度
    Arrays.fill(shiftTable, pattern.length());
    //将每个出现在Pattern中的char在数组中的位置里填充上移动距离(除最后一位)
    for (int j = 0; j < pattern.length() - 1; j++) {
        //char自动通过ASCII编码转码到对应的Int值，即shiftTable中对应Index
        //shift的距离根据当前char距离最后一个元素的距离决定，如果有重复的元素，则右边
        //的元素覆盖左边的元素，以右边距离最后一个元素的距离为准
        shiftTable[pattern.charAt(j)] = (pattern.length() - 1) - j;
    }
    return shiftTable;
}
```

## Knuth-Morris-Pratt(KMP) Algorithm
>此算法通过运用对这个词在不匹配时本身就包含足够的信息来确定下一个匹配将在哪里开始的发现，从而避免重新检查先前匹配的字符。

### 1.算法说明
1. 首先 Pattern 和 Pattern 自己进行比较，来建立一个 Prefix Table，其中如何建立在核心解释中说明。

2. 然后 Text 和 Pattern 进行比较，通过刚建立的 Prefix Table，在当前位置不匹配的时候回退 Pattern，继续比较，直到匹配到完整的 Pattern。在 Text 和 Pattern 进行比较时，运用的回退思想和建立 Prefix Table 时的回退是一个原理，都是不断回退到次一级长度的最长前缀。这也是为什么两个函数的代码格式看起来几乎一样，这次相当于把整个 Pattern 当作了一个前缀，去匹配 Text, 在 Text中出现 Pattern 长度的后缀时候，则说明匹配到。

3. **核心解释**<br>
创建最长前缀长度表的核心回退语句的示例分析，
Prefix Table 中的数字代表在字符串[0..i]中相匹配的前缀与后缀中，最长的字符串的长度。<br>
**例子：<br>
下标 Index: &ensp; &ensp;&ensp; &ensp;&ensp; &ensp;&ensp;[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]<br>
子字符串 Pattern: &ensp; &ensp; [a, b, a, b, z, a, b, a, b, a]<br>
最终 Prefix Table: &ensp; &ensp; [0, 0, 1, 2, 0, 1, 2, 3, 4, 3]**

   前面的 Index 较好理解，主要考虑 Index=9 时候的最终结果为什么等于3来理解函数中嵌套在 for 循环中的 while 循环，从 Index=5 开始 maxPrefixLength=1，而后到 Index=8 为止，之间的元素都与前缀匹配，所以 maxPrefixLength=4, 然而 Index=9 时，与 Index=4 的前缀不匹配，则需要进行回退，接下来主要分析**如何回退**.
   
   - 可以很明显的发现，在字符串 ababzabab(0-8) 中，后缀的最长前缀为abab(0-3)，然而当下一  个 Index 不匹配的时候，需要去找一个这样的新后缀，让这个后缀在当前范围内(0-8)有一个前缀，且对应的这个前缀(0-2) + 下一个 Index(3) 构成的新前缀(0-3)可以等于这个后缀(7-8) + Index(9) 构成的新后缀(7-9)。
   
   - 显而易见的，这个后缀越长越好，在例子，这个后缀为 Index7-8 构成的长度为2的后缀，也正好是次最长前缀长度。然而如果这个次最长前缀 + 新Index不符合要求，则再往下找新的更小的前缀，直到退到前缀长度为0，则下一个 Index 的 maxPrefixLength 从0再开始计算。
   
   - 最终，我们要用代码来快速的不断找次一级的最长前缀长度，当前例子中，Index0-8 之间，后缀的最长前缀为"abab"(0-3)，次一级为"ab"(0-2),再次一级为“”(0),可以看出次一级的前缀永远都是上一级的子集，即可以不断在上一级的前缀里找到下一级前缀，同时，在PrefixTable中，已经都存取了每一段子字符串的最长前缀的长度：
   
     比如 ababzabab(0-8), 在 Index=8 的 PrefixTable 中，最长前缀长度为4(abab),
   
     往前找 abab(0-3), 在 Index=3 的 PrefixTable 中，最长前缀长度为2(ab),
   
     往前找 ab(0-1), 在 Index=1 的 PrefixTable 中，最长前缀长度为0,
   
     所以，在代码中，如果字符串的下一个 Index 不符合 Prefix 的下一个 Index，则通过 PrefixTable 中记录的最长前缀长度不断的回退，直到匹配要求。


### 2.算法表现
Text: 长度 N; Pattern: 长度 M; <br>
*O(N+M)* <br>
建立最长前缀长度表*O(M)*, 搜索*O(N)* <br>

**引用**

[知乎用户逍遥行答案](https://www.zhihu.com/question/21923021)<br>
算法导论

```java
public static List<Integer> KMPMatching(String text, String pattern) {
    List<Integer> result = new ArrayList<>();
    int[] prefixTable = buildPrefixTable(pattern);
    int matchedCharacter = 0;
    for (int i = 0; i < text.length(); i++) {
        //不匹配，根据prefixTable回退到相对应的位置，
        while (matchedCharacter > 0 && pattern.charAt(matchedCharacter) != text.charAt(i)) {
            matchedCharacter = prefixTable[matchedCharacter - 1]; //核心回退语句
        }
        //匹配，匹配数量+1
        if (pattern.charAt(matchedCharacter) == text.charAt(i)) {
            matchedCharacter++;
        }
        if (matchedCharacter == pattern.length()) {
            result.add(i - matchedCharacter + 1);
            matchedCharacter = prefixTable[matchedCharacter - 1];
        }
    }
    return result;
}
    
private static int[] buildPrefixTable(String pattern) {
    int[] prefixTable = new int[pattern.length()];
    int maxPrefixLength = 0;
    //遍历Pattern，对每一个Char创建一个对应的最长前缀长度表
    for (int i = 1; i < prefixTable.length; i++) {
        while (maxPrefixLength > 0 && pattern.charAt(maxPrefixLength) != pattern.charAt(i)) {
            //当前index的char和prefix的char对应不上，则不断去找去掉当前位置得char的
            //之前的所有的text的后缀的“次一级最长前缀长度”，直到
            //①Length为0，从头再开始匹配，
            //②匹配到相对应的prefix，往下运行。具体可以看例子来理解这句话
            maxPrefixLength = prefixTable[maxPrefixLength - 1];    //核心回退语句
        }
        if (pattern.charAt(maxPrefixLength) == pattern.charAt(i)) {
            maxPrefixLength++;
        }
        prefixTable[i] = maxPrefixLength;
    }
    return prefixTable;
}    
```