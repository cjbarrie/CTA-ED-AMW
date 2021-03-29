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

- Form of unsupervised learning
    - Identifies *k* topics from bag of words across documents
        - Assumes each document is generated from a set of *k* topics
        - Assumes each word has probability *p* of belonging to a certain topic
    - We observe only words and documents
    - We specify k of topics
    
Topic models
========================================================

- Why use it?
    - Classification (when k of topics "known")
    - Exploratory analysis
        - "Reverse engineering" the topic generation process
        - N.B. Tea leaves problem
        
========================================================

<center>
<img src="images/nelson.png" width=600 height=900>
</center>

========================================================

<center>
<img src="images/nelsontop.png" width=600 height=900>
</center>
- Source: Nelson, Laura K. 2020. "Computational Grounded Theory: A Methodological Framework," *Sociological Methods & Research*, 49(1) 3-42.

========================================================

<center>
<img src="images/pnasinfact.png" width=1450 height=900>
</center>

========================================================

<center>
<img src="images/pnasinfactop.png" width=1450 height=900>
</center>
- Source: Smith et al. 2021. "Automatic detection of influential actors in disinformation networks," *Proceedings of the National Academy of Sciences*, 118(4) 1-10.

    
Topic models
========================================================



- Data structure:
    - `DocumentTermMatrix`
    

```
<<DocumentTermMatrix (documents: 2, terms: 12092)>>
Non-/sparse entries: 17581/6603
Sparsity           : 27%
Maximal term length: 18
Weighting          : term frequency (tf)
Sample             :
      Terms
Docs   country democratic government laws nations people power society time
  DiA1     357        212        556  397     233    516   543     290  311
  DiA2     167        561        162  133     313    360   263     241  309
      Terms
Docs   united
  DiA1    554
  DiA2    227
```


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



Converting to DocumentTermMatrix
========================================================


```r
tocq_words <- tocq %>%
  mutate(booknumber = ifelse(gutenberg_id==815, "DiA1", "DiA2")) %>%
  unnest_tokens(word, text) %>%
  count(booknumber, word, sort = TRUE) %>%
  ungroup() %>%
  anti_join(stop_words)

tocq_dtm <- tocq_words %>%
  cast_dtm(booknumber, word, n)

tm::inspect(tocq_dtm)
```

```
<<DocumentTermMatrix (documents: 2, terms: 12092)>>
Non-/sparse entries: 17581/6603
Sparsity           : 27%
Maximal term length: 18
Weighting          : term frequency (tf)
Sample             :
      Terms
Docs   country democratic government laws nations people power society time united
  DiA1     357        212        556  397     233    516   543     290  311    554
  DiA2     167        561        162  133     313    360   263     241  309    227
```

Estimating a topic model
========================================================


```r
tocq_lda <- LDA(tocq_dtm, k = 10, control = list(seed = 1234))
```
