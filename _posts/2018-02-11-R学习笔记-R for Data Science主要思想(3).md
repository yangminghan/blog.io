---
layout: post
title: R学习笔记-R for Data Science主要思想（3）
date: 2018-2-11
categories: blog
tags: [R]
description:  
--- 
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# R学习笔记-R for Data Science 主要思想（3）
#### Missing Values
&emsp;&emsp;tidy data时， 我们面临的另一个问题就是如何处理missing value，通常情况下，missing value有两种形式：  
1. 显式：标注为NA；
2. 隐式：没有出现在数据集中；  

&emsp;&emsp;一下面的这个小的数据集就包含这两种数据缺失的形式，2015年第4季度return值为显示的缺失，2016年第1季度为隐式的缺失。

        stocks <- tibble(
          year   = c(2015, 2015, 2015, 2015, 2016, 2016, 2016),
          qtr    = c(   1,    2,    3,    4,    2,    3,    4),
          return = c(1.88, 0.59, 0.35,   NA, 0.92, 0.17, 2.66)
        )

&emsp;&emsp;我们有两种方式，将隐式的缺失变为显式的缺失，一种方法是运用上一篇中提到的speard后，重新gather的方法：

        stocks %>% spread(year, return) %>%
        +            gather(year, return,`2015`:`2016`)
        # A tibble: 8 x 3
            qtr year  return
          <dbl> <chr>  <dbl>
        1    1. 2015   1.88 
        2    2. 2015   0.590
        3    3. 2015   0.350
        4    4. 2015  NA    
        5    1. 2016  NA    
        6    2. 2016   0.920
        7    3. 2016   0.170
        8    4. 2016   2.66 

&emsp;&emsp;另一种方法是使用complete()函数，该函数是找出给定列中所有值的组合：

        stocks %>% complete(year, qtr)
        # A tibble: 8 x 3
           year   qtr return
          <dbl> <dbl>  <dbl>
        1 2015.    1.  1.88 
        2 2015.    2.  0.590
        3 2015.    3.  0.350
        4 2015.    4. NA    
        5 2016.    1. NA    
        6 2016.    2.  0.920
        7 2016.    3.  0.170
        8 2016.    4.  2.66

&emsp;&emsp;当然，我们既然发现了缺失值，就要进行缺失值的填补，tidyr中fill的方式很简单，就是用上面的值向下填补或者用下面的值向上填补。

        treatment <- tribble(
          ~ person,           ~ treatment, ~response,
          "Derrick Whitmore", 1,           7,
          NA,                 2,           10,
          NA,                 3,           9,
          "Katherine Burke",  1,           4
        )



    treatment%>%fill(person, .direction = "up")
        # A tibble: 4 x 3
          person           treatment response
          <chr>                <dbl>    <dbl>
        1 Derrick Whitmore        1.       7.
        2 Katherine Burke         2.      10.
        3 Katherine Burke         3.       9.
        4 Katherine Burke         1.       4.

## 实例
&emsp;&emsp;我们将所有tidy的方法梳理了一遍之后，我们需要将所有的方法运用在一个实例之中，[2014 World Health Organization Global Tuberculosis Report](http://www.who.int/tb/country/data/download/en/)给我们提供了一个真实的数据集。这个数据集中提供了很多流行病学方面的信息，但是处理起来也非常具有挑战性。  

        who
        #> # A tibble: 7,240 x 60
        #>   country     iso2  iso3   year new_sp_m014 new_sp_m1524 new_sp_m2534
        #>   <chr>       <chr> <chr> <int>       <int>        <int>        <int>
        #> 1 Afghanistan AF    AFG    1980          NA           NA           NA
        #> 2 Afghanistan AF    AFG    1981          NA           NA           NA
        #> 3 Afghanistan AF    AFG    1982          NA           NA           NA
        #> 4 Afghanistan AF    AFG    1983          NA           NA           NA
        #> 5 Afghanistan AF    AFG    1984          NA           NA           NA
        #> 6 Afghanistan AF    AFG    1985          NA           NA           NA
        #> # ... with 7,234 more rows, and 53 more variables: new_sp_m3544 <int>,
        #> #   new_sp_m4554 <int>, new_sp_m5564 <int>, new_sp_m65 <int>,
        #> #   new_sp_f014 <int>, new_sp_f1524 <int>, new_sp_f2534 <int>,
        #> #   new_sp_f3544 <int>, new_sp_f4554 <int>, new_sp_f5564 <int>,
        #> #   new_sp_f65 <int>, new_sn_m014 <int>, new_sn_m1524 <int>,...

&emsp;&emsp;这也是一个非常典型的真实生活中的数据集，包含冗余的列，很多奇怪的编码，和很多缺失值。总而言之，是一个messy的数据集，我们下面要做的就是把它变为tidy data。  

- 看起来country, iso2, iso3这三个变量都代表国家，为冗余变量；   
- 我们还不知道其他列的情况，但是从给的列名来看，（new_sp_m5564, new_sp_m65）它们更像是值而不是变量。  

&emsp;&emsp;我们首先先处理 从new_sp_m014 到newrel_f65这些列，我们还不知道这些值代表什么意思，所以给它起一个通用的名称key； 我们知道这些列里面的每个值代表这一列出现的数量，所以使用cases代表出现数量；由于有很多的缺失值，所以我们将缺失值移除，将重点放在现有的值上面。

        who1<- who %>% gather(
        +   new_sp_m014:newrel_f65, key= "key",
        +   value = "cases",
        +   na.rm = TRUE
        + )
        > who1
        # A tibble: 76,046 x 6
           country     iso2  iso3   year key         cases
         * <chr>       <chr> <chr> <int> <chr>       <int>
         1 Afghanistan AF    AFG    1997 new_sp_m014     0
         2 Afghanistan AF    AFG    1998 new_sp_m014    30
         3 Afghanistan AF    AFG    1999 new_sp_m014     8
         4 Afghanistan AF    AFG    2000 new_sp_m014    52
         5 Afghanistan AF    AFG    2001 new_sp_m014   129
         6 Afghanistan AF    AFG    2002 new_sp_m014    90

&emsp;&emsp;新生成的key列包含的数据（new_sp_m014），如果没有解释的话我们完全没有办法理解到底是个什么意思，不过好在WHO给我们提供了数据字典，这样我们就可以解析这一列数据了：  

1. 前三个字母代表的是这一列是否是新的结核（ 结核 tuberculosis TB）病例，在这个数据集中每一列包含的都是新的病例；  
2. 接下来两个字母代表的是结核病例的类型：
    - rel 代表复发病例（relapse）
    - ep  代表肺外结核（extrapulmonary ）
    - sn 代表不可以被涂片诊断的肺结核，涂阴肺结核（smear negative）
    - sp 代表可以被涂片诊断的肺结核，涂阳肺结核(smear positive)
3. 第六个字母代表病人的性别，m代表女性，f代表男性；
4. 剩下的数字代表年龄组：
    - 014 = 0 – 14 years old
    - 1524 = 15 – 24 years old
    - 2534 = 25 – 34 years old
    - 3544 = 35 – 44 years old
    - 4554 = 45 – 54 years old
    - 5564 = 55 – 64 years old
    - 65 = 65 or older

&emsp;&emsp; 我们还需要进行一些小的修改，我们的数据里面有newrel和new_rel，我们需要将他们统一成new\_rel.

        who2 <-who1 %>% mutate(key = stringr::str_replace(key, "newrel","new_rel"))

&emsp;&emsp;接下来，我们就可以将这一列的数据分开，首先用下划线进行分割：  

    who3 <- who2 %>% separate(key,c("new","type","sexage"),sep = "_")

&emsp;&emsp;然后，再丢弃一些我们不需要的列，new这一列没有提供信息，我们不需要，iso2，iso3这两列为代表国家的冗余列，我们同样不需要：

    who4 <-who3%>%select(-new, -iso2, -iso3)

&emsp;&emsp;最后我们将sex和age分割开，我们就完成了整个tidy的步骤：

    who5<-who4%>%separate(sexage,c("sex","age"),sep = 1

最后，将所有的步骤合并在一起，我们得到了最终的tidy data：

        who %>% gather(new_sp_m014:newrel_f65, key= "key",value = "cases",na.rm = TRUE)%>% 
        +         mutate(key = stringr::str_replace(key, "newrel","new_rel")) %>%
        +         separate(key,c("new","type","sexage"),sep = "_") %>%
        +         select(-new, -iso2, -iso3)%>%
        +         separate(sexage,c("sex","age"),sep = 1)
        # A tibble: 76,046 x 6
           country      year type  sex   age   cases
           <chr>       <int> <chr> <chr> <chr> <int>
         1 Afghanistan  1997 sp    m     014       0
         2 Afghanistan  1998 sp    m     014      30
         3 Afghanistan  1999 sp    m     014       8
         4 Afghanistan  2000 sp    m     014      52

