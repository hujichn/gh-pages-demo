---
title: Tips on data analysis by python
date: 2016-03-02 22:42:00
categories: coding
tags: Python
---

利用python进行数据分析时的一些小tips，有的是python自带的数据结构，还有很多是针对pandas当中的dataframe和series，因为是零零碎碎整理的 ，比较发散，平时想到了就加进来，所以没太多逻辑哈。

<!--more-->

author: [@Huji][1]

[toc]

### 1. 建立一个元素全部为0的长度为10的list
```
mylist = [0] * 10
```

类似地还有输出一条分割线
```
print("-" * 40)
```

### 2. 判断list当中不为nan的元素，统计其数值
```
count = sum([str(x) != 'nan' for x in mylist])
```

### 3. 快速得到DataFrame的长度
```
length = len(df.index)
```

### 4. 把Datetime格式的日期转化成字符串
```
dt.strftime('%Y%m%d')
```

### 5. 把DataFrame的某一列转化为datetime格式
```
df['datetime'] = pd.to_datetime(df['datetime'])
```

### 6. 把DataFrame中某一列str数据转换为int
```
df['column_name'] = df['column_name'].astype('int')
```

### 7. 把DataFrame中混合了nan和str的数据转换为整数（int）
```
df['column_name'] = pd.to_numeric(df['column_name'])
```

### 8. 读取csv的时候指定index，列名，以及某一列的数据类型
```
df = pd.read_csv('df.csv', index_col = 'a', header=0, usecols=[0,1], names = {'a','b'}，dtype={'NET_ID':int})
```

### 9. 找出某一列中出现次数最多的数值
（mode()函数的说明：Empty if nothing occurs at least 2 times. Always returns Series even if only one value.）
```
value = mSeries.mode().loc[0]
```

### 10. 判断loc后的结果是Series还是DataFrame
```
isinstance(mReturnValue, pd.Series)
isinstance(mReturnValue, pd.DataFrame)
```

### 11. 从Series建立只有一行的DataFrame
```
df = pd.DataFrame([mSeries])
```

### 12. 得到df当中a列值小于5的部分
```
result = df[df.a < 5]
```

### 13. 写入csv文件，不覆盖之前的内容
```
df.to_csv('a.csv', mode = 'a', header = False)
```

### 14. 写入sql的参数设置
```
df.to_sql('tableName', engine, if_exists = 'append', index = True, index_label = 'indexName')
```
if_exists也可以设置为`replace`

### 15. 找出符合某些条件的区域，赋给一定的数值
```
df.ix[df.stock < 0, 'stock'] = 0
```

### 16. 重采样
```
df = df.resample("30min", closed = 'right', label='right', how='last', fill_method = 'ffill')
```

### 17. apply某一函数
```
df["column_name"] = df['column_name'].apply(lambda x: 2*int(x)-3)
```

### 18. append和concat
功能类似，好像concat在连接空DataFrame的时候效率会很低

### 19. 浮点数取整
- 类型工厂函数,int()，效果：浮点数取整，如int(3.5)就返回3；数字的字符形式转换成数字，如int("35")就返回35
- 内置函数的round(),四舍五入，第二个参数是保留小数点后多少位，默认是0，如round(3.5)返回4.0，round(3.5,1)就返回3.5，不能取整
- math模块的floor(),取小于等于的整数,如floor(3.5)返回3.0,floor(-1.5)返回-2.0，也不能取整

### 20. 提取出DataFrame当中等于某些数值的行
```
df = pd.DataFrame({'vals': [1, 2, 3, 4], 'ids': ['a', 'b', 'f', 'n'], 'ids2': ['a', 'n', 'c', 'n']})
                  
# 对于，dataframe，这里的values还可以写成字典，values = {"ids":['a', 'b'], "vals": [1, 2]}，对于series，list就够了

values = ['a', 'b']

# 对于字典的话，找出同时满足条件的行，则要在后面加个.all(1)
row_mask = df['ids'].isin(values)
df[row_mask]
```


  [1]: https://hujichn.github.io