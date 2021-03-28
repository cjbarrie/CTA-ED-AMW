Topic models
========================================================
author: Christopher Barrie 
date: University of Edinburgh
width: 2500
height: 900
transition: none  
  website: https://cjbarrie.xyz     
  github: https://github.com/cjbarrie       
  Twitter: https://www.twitter.com/cbarrie

Topic models
========================================================


Getting the data
========================================================




```r
library(tidyverse) # loads dplyr, ggplot2, and others
library(stringr) # to handle text elements
library(tidytext) # includes set of functions useful for manipulating text
library(topicmodels) # to estimate topic models
library(gutenbergr) # to get text data
library(scales)
library(tm)
library(ggthemes) # to make your plots look nice
```


```r
tocq <- gutenberg_download(c(815, 816), 
                            meta_fields = "author")
```


