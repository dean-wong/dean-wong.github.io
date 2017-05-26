---
layout: post
title:  "CSVKIT——处理csv文件的瑞士军刀"
date:   2017-04-02
categories: jekyll update
---
> 统计和数据处理都离不开csv文件，一般来说，操作csv都是依赖第三方软件，如Excel、Matlab这样的；也可以自己编程，用Python、Perl等等，直接操作文件比较麻烦。

> 但对简单的处理，用大体量的软件或是写一段代码有点杀鸡用牛刀的感觉，这时，csvkit就有用武之地。

# 开始
## 安装csvkit

安装[csvkit](http://csvkit.readthedocs.io/)很简单：

```sh
pip3 install csvkit
```
我的系统是Python3.6，所以用pip3来安装，如果还在用Python2.7，就用`pip install csvkit`来进行安装。

为了方便后面的测试和解释，建议从官方下载测试样本文件：

```sh
curl -L -O  https://raw.githubusercontent.com/wireservice/csvkit/master/examples/realdata/ne_1033_data.xlsx
```



## in2csv：Excel的杀手

下载的样本文件是一个Excel文件，谁愿意为了仅仅看两行数据而等待Excel漫长的启动界面呢？所以首先要将它转换为csv文件，转换也不用打开Excel，一条命令：

```sh
in2csv ne_1033_data.xlsx > data.csv
```

`>`是shell的重定向符，如果不重定向，in2csv会将转换之后的内容输出到控制台显示，现在我希望能保存为csv文件便于后续的操作，所以就将内容输出定向到data.csv文件中。

接下来看看data.csv的内容吧：

```sh
cat data.csv
```

![cat 的输出原始而粗糙](http://upload-images.jianshu.io/upload_images/1749344-28465bba93cdca4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


csv文件内容有了，如果用`cat`的方式来看，既不方便也不美观，接下来我们用另一个工具查看

## csvlook：数据潜望镜

这个文件有多少行呢，用`cat data.csv | wc -l`看了一下，有**1037**行，所以一屏肯定是看不完的，需要分页查看，而分页功能在类Unix系统上有现成的`less`可用。

```sh
csvlook data.csv | less -S
```

![csvlook](https://ww2.sinaimg.cn/large/006tNc79gy1fe868td2cij30ha0ggdgu.jpg)

这样的显示就好多了，还可以方向键滚动查看，`q`退出，`/`搜索，等一系列`less`专用操作。

## csvcut：数据手术刀

既然名字中带了一个`cut`，聪明的你肯定猜到了它的用途，不过放心，所有的操作都不会修改输入（原始文件），只会影响输出。

先看看这个数据有哪些列

```sh
csvcut -n data.csv
```

![csvcut查看列标题](https://ww1.sinaimg.cn/large/006tNc79gy1fe86ktym89j30ha0ggglw.jpg)

可以看到，总共有14列，在输出信息中，前面的数字是数字索引位置，默认是从**1**开始计数，后面的是列名称，我想只看感兴趣的几列数据(在这里我为了演示，只显示4行数据，所以加了`head`进行输出行数限定)：

```sh
csvcut -c 2,5,6 data.csv | head -n 5
```

![选择列](https://ww1.sinaimg.cn/large/006tNc79gy1fe86sm9wgqj30ha0gg3z0.jpg)

可以看到既可以用数字作为索引，也可以用列名称作为索引，所以上面的指令和下面这个指令是等价的

```sh
csvcut -c county,item_name,quantity data.csv | head -n 5
```

## 用管道进行组合

上面的输出不错，能否再显示得美观一点呢？答案是肯定的，将命令用管道组合起来就行了

```sh
csvcut -c county,item_name,quantity data.csv | csvlook | head
```

![Putting it together with pipes](https://ww4.sinaimg.cn/large/006tNc79gy1fe86ydgk8fj30ha0ggmxh.jpg)

`|`管道是类Unix平台上的强大能力，如果不关心中间结果，甚至可以从头到尾都用管道进行连接，例如上面的命令改成如下方式，也是一样的结果

```sh
in2csv ne_1033_data.xlsx | csvcut -c county,item_name,quantity | csvlook | head
```

这样就省去了生成data.csv文件的中间过程。

# 数据分析

## csvstat：无代码亦统计

使用`csvlook`和`csvcut`查看数据的切片只是探索数据的开始，在实践中，通常还需要一些计算和统计，`csvstat`的设计灵感来自编程语言[ “R”](http://www.r-project.org/)的`summary()`统计，它可以统计汇总一个CSV文件中的数据列。

```sh
csvcut -c county,acquisition_cost,ship_date data.csv | csvstat
```

![命令行的*Summary*](https://ww4.sinaimg.cn/large/006tNc79gy1fe87ad9weoj30ha0hmt99.jpg)

不知道你有没有统计学基础？如果有的话，应该一目了然，唯一值、最小值、最大值、总计、均值、中位值、标准差……，大部分情况下够用了:)

## csvgrep：找到需要的数据

仅仅按照列的方式过滤还是比较弱，还需要按照内容进行过滤，这时`csvgrep`就派上用场，例如，我们只想看兰开斯特的数据。

```sh
csvcut -c county,item_name,total_cost data.csv | csvgrep -c county -m LANCASTER | csvlook
```

![基于列和内容过滤](https://ww3.sinaimg.cn/large/006tNc79gy1fe87nnhn1xj30e40hmgm4.jpg)

`csvgrep`还支持正则表达式，关于正则表达式的讨论有点超出本篇范围，有兴趣可以查看系统工具`grep`的手册。

## csvsort：秩序

前面显示出来的内容都很少，实际中往往都有成千上万行，所以排序功能很重要，我们试一下用`csvsort`对`total_cost`列进行递减顺序：

```sh
csvcut -c county,item_name,total_cost data.csv | csvgrep -c county -m LANCASTER | csvsort -c total_cost -r | csvlook
```

![按照总成本递减排列](https://ww3.sinaimg.cn/large/006tNc79gy1fe87wrjlbuj30e40hmq3g.jpg)

# 宝刀屠龙

## csvjoin：关联数据

在上面的数据中，有各个地区的武器装备信息，接下来想了解一个问题——**装备有武器的地区中，哪里的人口最少？**

原来的数据表不包含人口数量，所以下载一个包含人口信息的数据：

```sh
curl -L -O https://raw.githubusercontent.com/wireservice/csvkit/master/examples/realdata/acs2012_5yr_population.csv
```

![各地区的人口数据](https://ww3.sinaimg.cn/large/006tNc79gy1fe88yxxdvuj30e40hmgm3.jpg)

两个数据表都包含`fips`字段，我们可以用这个字段来连接两个数据：

```sh
csvjoin -c fips data.csv acs2012_5yr_population.csv > joined.csv
csvcut -c county,item_name,total_population joined.csv | csvsort -c total_population | csvlook | head
```

![两个小地区也有5.56毫米突击步枪](https://ww1.sinaimg.cn/large/006tNc79gy1fe893cm9egj30gm0hmaaf.jpg)

## csvstack：合并数据

数据经常会分散在不同地方，这时希望将一堆的csv文件合并成一个数据文件，`csvstack`可以帮助达成这个目标，一般来说`csvstack`需要**同列同名**的数据才可以合并。但是你知道，有时候即使两个csv文件有稍微的不同，也可以通过`csvcut`的列选择来获得同列同名的数据表。

```sh
# 获得堪萨斯州的测试数据
curl -L -O https://raw.githubusercontent.com/wireservice/csvkit/master/examples/realdata/ks_1033_data.csv
# 使用相同的文件名命名规则
in2csv ne_1033_data.xlsx > ne_1033_data.csv
# 合并两个文件到region.csv中
csvstack ne_1033_data.csv ks_1033_data.csv > region.csv
# 查看部分列的统计信息
csvstat -c state,acquisition_cost region.csv
```

![合并两个州的数据](https://ww1.sinaimg.cn/large/006tNc79gy1fe89s0gfz9j30gm0hmmxk.jpg)

## csvsql, sql2csv：必杀技

有时，仅仅命令行是不够的，需要用SQL来进行数据的操作，`csvsql`和`sql2csv`就提供了数据库和csv之间的桥梁。

例如，我想将现在的csv文件转为数据库表

```sh
csvsql -i sqlite joined.csv
```
![Sqlite数据表](https://ww4.sinaimg.cn/large/006tNc79gy1fe8a2nmct9j30gm0hmwex.jpg)

当然，我们可以一步到位，直接创建一个本地数据库：

```sh
csvsql --db sqlite:///leso.db --insert joined.csv
```

![创建本地数据库文件leso.db](https://ww2.sinaimg.cn/large/006tNc79gy1fe8a9q541aj30gm0hmq3a.jpg)

接下来，就可以利用数据库软件对这个数据库进行常规的SQL查询，当然也可以用CSVKIT提供的小工具`sql2csv`来进行查询。

```sh
sql2csv --db sqlite:///leso.db --query "select * from joined where total_population<1000;" | csvcut -c state,county,total_population | csvlook
```

![SQL 查询](https://ww1.sinaimg.cn/large/006tNc79gy1fe8air4g7kj30gm0hm3ys.jpg)

如果SQL查询使用不频繁，何不直接在csv上执行SQL查询呢？

```sh
csvsql --query "select county,item_name,quantity from joined where quantity == 5;" joined.csv 2>/dev/null | csvlook
```

![直接在csv文件上执行SQL查询](https://ww1.sinaimg.cn/large/006tNc79gy1fe8aof4rx1j30gm0hmjro.jpg)

# 总结

由于我经常做数据处理的计算，所以磁盘上总是有大量的csv文件，用普通的文本编辑器打开非常缓慢不说，还不方便查询和检视，有了这套工具确实方便了很多。此外，它对中文的处理也还行，文件设置为UTF-8编码就好。