---
layout: post
title: R学习笔记-R for Data Science主要思想（2）
date: 2018-2-7
categories: blog
tags: [R]
description:  
--- 
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# R学习笔记-R for Data Science 主要思想（2）
&emsp;&emsp;接着我们上一篇博客继续探索R for Data Science这本书，上一篇博客我们讲到如何才能实现从untidy data 到tidy data的转变，Hadley在书中给出了常用的数据处理的方法分别是，Spreading, Gatering, Separating, Pull, 下面，我们一项一项来讨论一下。
## Tidy
#### Spreading and Gathering
##### Gathering
&emsp;&emsp;数据经常存在的一个问题就是：一个数据集中的某些变量的名称并不是变量的名称，而是一个变量的值，那table4a为例，1999和2000两个变量代表的是年份变量的值，这样的话每行数据代表两个observation，而不是一个：

    library(tidyverse)  
    table4a  
    # A tibble: 3 x 3  
      country     `1999` `2000`  
    * <chr>        <int>  <int>  
    1 Afghanistan    745   2666  
    2 Brazil       37737  80488  
    3 China       212258 213766  

&emsp;&emsp;要想tidy这样一个数据集，我们需要将这些列**收集**成为一个变量, 书中原文使用的是gathering这个词，我认为非常形象，就像我们把其他的变量，收集到一起一样。  
&emsp;&emsp;为了完成这样一个操作，我们需要两个参数：  
1. 我们需要收集到一起的变量，也就是那些代表的是值而不是变量的列，在这里，是1999和2000两列。  
2. 收集来的变量的名称。  


    table4a %>% gather(`1999`, `2000`,key = 'year', value = "cases")    
    # A tibble: 6 x 3  
      country     year   cases  
      <chr>       <chr>  <int>  
    1 Afghanistan 1999     745  
    2 Brazil      1999   37737  
    3 China       1999  212258  
    4 Afghanistan 2000    2666  
    5 Brazil      2000   80488  
    6 China       2000  213766  

![image](http://r4ds.had.co.nz/images/tidy-9.png)

&emsp;&emsp;其实我的感觉非常像reshape2的melt和cast功能，将一个宽的数据变成一个长的数据，但是，reshape2也是Hadley大神的杰作，他在tidyverse里面没有包括reshape2，估计是做了一次自我革命，当然，plyr这个包当时也是非常惊艳，但是现在其中的很多功能被dplyr所代替。
##### Spreading
&emsp;&emsp;**Spreading**和**Gathering**正好是两个相反的操作，当一个观测（observation）分散在不同的行中时，我们就要使用**Spreading**。例如table2:  

    table2
    # A tibble: 12 x 4
       country      year type            count
       <chr>       <int> <chr>           <int>
     1 Afghanistan  1999 cases             745
     2 Afghanistan  1999 population   19987071
     3 Afghanistan  2000 cases            2666

&emsp;&emsp;一个观测（observation）应该是一个国家一年的情况，但是却分散在两行中，这里使用spread，需要传入包括变量名称的列和包含值的列：  

    spread(table2, key = type, value = count)
        # A tibble: 6 x 4
          country      year  cases population
          <chr>       <int>  <int>      <int>
        1 Afghanistan  1999    745   19987071
        2 Afghanistan  2000   2666   20595360
        3 Brazil       1999  37737  172006362
        4 Brazil       2000  80488  174504898
![image](http://r4ds.had.co.nz/images/tidy-8.png)

&emsp;&emsp;spread和gather是互补的两个函数，gather可以使数据变得“窄”并且“长”，spreade则是数据变得“宽”并且“短”。

#### Separating and Pull
&emsp;&emsp;当一列数据包含两个变量的时候，如table3这种情况：  

        table3
        # A tibble: 6 x 3
          country      year rate             
        * <chr>       <int> <chr>            
        1 Afghanistan  1999 745/19987071     
        2 Afghanistan  2000 2666/20595360    
        3 Brazil       1999 37737/172006362  
        4 Brazil       2000 80488/174504898    
        
&emsp;&emsp;rate这一列包含了两个变量cases和population，这样，我们需要使用separate将他重新分割成两个变量。
##### Separate 

    table3 %>%separate(rate, into = c('cases', 'population'))
            # A tibble: 6 x 4
          country      year cases  population
        * <chr>       <int> <chr>  <chr>     
        1 Afghanistan  1999 745    19987071  
        2 Afghanistan  2000 2666   20595360  
        3 Brazil       1999 37737  172006362 
        4 Brazil       2000 80488  174504898 

![image](http://r4ds.had.co.nz/images/tidy-17.png)
&emsp;&emsp;separate函数非常方便，它默认使用非数字也非字母的符号进行分割，当然我们也可以指定分割符号，这里的分割符号为正则表达式：    

    table3 %>%separate(rate, into = c('cases', 'population'), sep = "/")

&emsp;&emsp;新产生的两列数据类型为character，我们也可以通过指定convert参数进行转换：  

        table3 %>%separate(rate, into = c('cases', 'population'), convert = TRUE)
        # A tibble: 6 x 4
          country      year  cases population
        * <chr>       <int>  <int>      <int>
        1 Afghanistan  1999    745   19987071
        2 Afghanistan  2000   2666   20595360
        3 Brazil       1999  37737  172006362

##### Unite
&emsp;&emsp;unite函数正好和separate函数相反，它是将两个不同的列合并为同一个列，我们先使用separate函数将year分成两列，century和year：  

        table3 %>%separate(year, into = c('century', 'year'), sep = 2)->table5
        # A tibble: 6 x 4
          country     century year  rate             
        * <chr>       <chr>   <chr> <chr>            
        1 Afghanistan 19      99    745/19987071     
        2 Afghanistan 20      00    2666/20595360    
        3 Brazil      19      99    37737/172006362  
        4 Brazil      20      00    80488/174504898  
        5 China       19      99    212258/127291527

&emsp;&emsp;然后再使用unite合并两列,unite默认的间隔符号为下划线(_),如果不想使用任何的间隔符号，我们使用“”：   

        table5 %>% unite(new,century, year)
        # A tibble: 6 x 3
          country     new   rate             
          <chr>       <chr> <chr>            
        1 Afghanistan 19_99 745/19987071     
        2 Afghanistan 20_00 2666/20595360    
        3 Brazil      19_99 37737/172006362  
        4 Brazil      20_00 80488/174504898   
        table5 %>% unite(new,century, year,sep = '')




