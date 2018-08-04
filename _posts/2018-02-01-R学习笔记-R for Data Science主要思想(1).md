---
layout: post
title: R学习笔记-R for Data Science主要思想（1）
date: 2018-2-1
categories: blog
tags: [R]
description:  
--- 
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# R学习笔记-R for Data Science 主要思想（1）
&emsp;&emsp;Hadley Wickham是R语言界当之无愧的大神，最近他与Garrett 合著的新书R for Data Science更是证明了这一点。在这本书中，Hadley介绍了Data Science的主要流程以及如何用R来实现流程中的每一个步骤。  
&emsp;&emsp;经典的书总是用一种简单明快的方式回答了它想回答的问题，R for Data Science 这本书就是这样，这本书详细阐述了Hadley对于数据科学流程的思考和流程中具体每一个模块应该如何使用R进行实现，但是对于这些R包的细节部分，Hadly没有进行太多的阐述，毕竟，书中介绍的每一个R包包含的内容都十分的丰富。  
&emsp;&emsp;如果有人问我该如何学习R语言，那么现在我的推荐就是这本书，这本书可以快速的构建起来整个Data Science的思维框架，使得在接触实际问题的时候不至于茫然不知所措。   
&emsp;&emsp;目前这本书没有中文版，不过Hadly直接明了的写作风格使得这本书读起来没有任何的负担。而且，他还提供了免费的[网页版本](http://r4ds.had.co.nz/introduction.html)供大家阅读。  
&emsp;&emsp;这本书中，Hadley提出了提出了一种数据科学的一种模式，即：  

![image](http://r4ds.had.co.nz/diagrams/data-science.png)


&emsp;&emsp;其中，Transform,Visualize,Model部分为不断循环的一个过程，直到输出满意的结果。Program是贯穿整个数据科学任务中的一个模块。  
&emsp;&emsp;不管是数据是图像、音频还是基因数据，不管模型是机器学习算法，普通的统计模型还是深度学习模型，所有的数据科学研究基本上就是这样一种模式。因此，只有我们明确这种模式，在接触到一个复杂的数据科学问题的时候，才可以理出头绪，不至于茫然不知所措。  
&emsp;&emsp;那么在这种模式下，R怎么实现这一模式中的各个模块呢，Hadley也在书中给出了很好的解答。他以**Tidyverse**为核心，给出了解决每一个模块中非常优秀的解决方案。这里，我说是非常优秀，但是，是不是最好的解决方法呢，我觉得这是一个开放性的问题，每一个数据科学任务都有自己的特殊性，有些情况下，用其他的包可能是更好的选择，但是，使用书中给出的方案，可以解决80%以上的问题。
## **Import**  
&emsp;&emsp;将数据导入是所有数据科学任务的第一步。Tidyverse中集成的Import包是readr。readr这个包的语法和基础包中的import基本一样。  

    library(tidyverse)  
    height <- read_csv("data/height.csv")  
    write_csv(challenge,"challenge.csv")  

&emsp;&emsp;readr数据导入的速度很快，而且提供了读条的功能，这对于一些比较大的数据包是非常方便的。  
&emsp;&emsp;readr一般是从我们的文件中读入数据，和它一样，而且功能可能更强大，适用于更大数据量的包是data.table,这个包的功能我也是见识过的，一般处理的数据量的大小在几个G的规模，使用data.table就十分方便了。当我们的数据为其他格式的文件的时候：  
&emsp;&emsp;**haven** 可以读取SPSS，Stata, SAS数据；  
&emsp;&emsp;**readxl** 可以读取Excel文件；  
&emsp;&emsp;**DBI（RMySQL,RSQOLite...）** 可以从数据库中读取数据；  
&emsp;&emsp;**jsonliyr** 可以读取JSON；  
&emsp;&emsp;**xml2** 可以读取XML文件。

## **Tidy**  
&emsp;&emsp;Hadly在这一章的开头引用了托尔斯泰的一句话，幸福家庭总是很相似，但是不幸福的家庭总是各有各的原因。这句话Hadly是这样说的Tidy dataset are all alike, but every messy dataset is mess in its own way.  
&emsp;&emsp;其实原来，我也没有认识到Tidy这个步骤的作用，在某种程度上说，我认为Tidy和Transform应该是在一起的，都是所谓的wrangle data.但是随着阅读的深入和实践的增加，我认为两者绝对不应该混为一谈：到达Transform这一步的数据应该已经是Tidy Data也就是我们所说的干净的数据。  
&emsp;&emsp;还有一点区别就是，在整个数据分析过程中，Transform会根据我们不同的模型，不同的可视化的目标，我们会将数据转换，生成出不同的形式，但是Tidy每一次任务我们只需要做一次就可以了。如果想要了解更多关于Tidy Data的内容，可以看一下[Hadley的论文](http://vita.had.co.nz/papers/tidy-data.html)。Hadley给出Tidy Data的定义：  
1. **Each variable must have its own column.**
2. **Each ovservation must have its own row.**
3. **Each value must have its own cell.**   

&emsp;&emsp;所谓大道至简，这里体现的淋漓尽致。Hadley在书中说:Tidy Data的定义如此的简单，有时候，你会认为是不是所有的数据都是Tidy Data，但是实在是很不幸大多数数据都不是Tidy Data，其中最重要的两个原因是：
1. 绝大多数的人并不熟悉Tidy Data的原则。  
2. 数据通常会被组织成方便其他用途的形式，例如数据会被组织成方便填写的情况。 

&emsp;&emsp;特别要提出的是，在平时的工作中，我们最常使用的数据工具是Excle，如果大家希望自己的工作效率提升，最好将自己经常会使用的数据的数据存储和数据展示分成两个表格，在数据存储的位置按照Tidy Data的原则进行组织，不要使用合并单元格等等操作，在数据展示的表格中，为了清晰和美观，可以使用合并单元格等等格式变化。  
&emsp;&emsp;那么，我们知道了Tidy Data的定义，如何才能实现Tidy Data呢？Hadley给出了几个步骤：Spreading ,Gathering ,Separating , Pull, and Deal with Missing Values。关于这几个步骤，我们将在下一篇blog中详细阐述。



