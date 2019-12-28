# Query Understanding



## 0. Introduction

1. 什么是搜索引擎:  接受文字输入(query)返回一系列与之相关的结果.
2. 为什么要 Query Understanding: 是用户与搜索引擎沟通的通道, 用户通过 query 表达自己的意图, 系统通过 query understanding 理解用户的意图, 从而更好搜索相关的结果
3. Query Understanding 将 query 视为一等公民, 在搜索引擎检索网页之前发生
4. Query understanding 在搜索引擎的交互界面起着至关重要的作用
   1. 自动补全
   2. 拼写检查
   3. query 细化(refinement)
5. Query understanding 是搜索建议的核心, 通过搜索 query 空间, 提示适合的搜索建议
6. 关注于 query understanding 能够给开发者不同的视角, 将注意力从 ranking 算法转移到 query 解读机制上.

## 1.  Language Identification -- 语言识别

1. 语言识别是 query understanding 的第一步
2. 历史:
   1. 1970s: 基于常用的字符组合以及声音片段进行识别
   2. 基于统计的方法
3. query 通常是短文本, 基于统计的方法在长文本效果很好, 在短文本上效果差
4. 使用其他的信号进行语言识别
   1. 搜索引擎的**点击日志**: 通过分析召回的结果来确定 query 的语言
   2. 引入使用者所在的**地理位置**来解决 query 与想要得到的 result 语言不同的情况.  难点在于:  在某个国家上网不代表就是使用该国的语言
   3. 将语言识别建模成以 query, doc, searcher's country 的机器学习问题. 准确率能够达到94.5%(across 10 European languages)
5. 当语言识别的置信度不高时: **询问**用户想要的语言!
6. 延伸阅读: “[Survey of Language Identification Techniques and Applications](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.689.5252&rep=rep1&type=pdf)” by Archana Garg



## 2.  Character Filtering -- 字符过滤

1. unicode正规化根据每一个字的具体的含义，把一些相同字的不同变形都对应到了同一个字符上，这样就可以方便一些本来应该是同一个字的不同字符的比较

2. Unicode Normalization: 统一将 query 和 document 使用 unicode 表示

   1. Normalization Form Canonical Decomposition (NFD): 将含有和音符号的字符进行分解表示

   2. Normalization Form Canonical Composition (NFC): 将含有和音字符的字符进行组合表示

   3. 搜索引擎的使用中推荐使用 NFD 方式, **目的是**后序移除 accent 方便

   4. ```python
      import unicodedata
      s1 = 'Spicy Jalape\u00f1o'
      s2 = 'Spicy Jalapen\u0303o'
      t1 = unicodedata.normalize('NFC', s1)
      t2 = unicodedata.normalize('NFC', s2)
      ```

3. 移除 accents: `''.join(c for c in t1 if not unicodedata.combining(c))`

4. 忽略大小写, 将文本统一转成小写(这个过程称为: case folding), 要注意指定的语言

## 3. Tokenization

1. tokenization 将字符序列转成 token(words) 序列
2. 将标点符号当作分隔符
3. 连字符和" ' ": 不同情况不同考虑
   1. 连字符: 考虑是否是真的连字符, twenty-five vs California-based
   2. ' :  缩写 vs 所有格, 缩写需要拆分还原; 所有格将 's 拆分为单独的 token
4. multiple tokenizations, 即同时保留多种 tokenize 的情况: e.g. women's -- womens 和 women's, 提高召回率
5. tokens 不是 words 的情况: 
   1. token 是单独的数字: 作为的单独的 token
   2. token 是混合情况, e.g. EC2
   3. token 是电话号码, 邮编等需要额外的算法进行处理

## 4. Spelling Correction

