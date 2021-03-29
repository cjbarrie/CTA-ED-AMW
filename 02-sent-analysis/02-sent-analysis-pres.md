Sentiment Analysis
========================================================
author: Christopher Barrie 
date: University of Edinburgh
width: 2500
height: 900
transition: none  
  website: https://cjbarrie.xyz     
  github: https://github.com/cjbarrie       
  Twitter: https://www.twitter.com/cbarrie
  
Sentiment analysis
========================================================

- Tokenizing
- Counting words with dictionary
- Denominating by totals

========================================================

<center>
<img src="images/subjwellnhb.png" width=1450 height=900>
</center>

========================================================

<center>
<img src="images/prosocpnas.png" width=1450 height=900>
</center>



Sentiment analysis
========================================================

- A sample of words from Pride and Prejudice


```
 [1] "heightened"    "intimate"      "blue"          "suspicion"    
 [5] "gaped"         "finest"        "creatures"     "pack"         
 [9] "contribute"    "tradesman's"   "peculiarly"    "concealed"    
[13] "cottagers"     "expostulation" "unjustly"      "founded"      
[17] "adventure"     "conciliate"    "animation"     "abuse"        
[21] "requited"      "guilty"        "repined"       "friend's"     
[25] "accede"        "readily"       "appointment"   "sooner"       
[29] "occasion's"    "lucases"       "beauteous"     "yawn"         
[33] "travelling"    "counterpart"   "prudential"    "hertfordshire"
[37] "temptations"   "jenkinson"     "levelled"      "clement's"    
[41] "change"        "headstrong"    "importance"    "meryton"      
[45] "quarreling"    "commencement"  "feet"          "longed"       
[49] "resisting"     "bosoms"       
```

Sentiment analysis
========================================================


```r
bingsent <- get_sentiments("bing")
bingsent
```

```
# A tibble: 6,786 x 2
   word        sentiment
   <chr>       <chr>    
 1 2-faces     negative 
 2 abnormal    negative 
 3 abolish     negative 
 4 abominable  negative 
 5 abominably  negative 
 6 abominate   negative 
 7 abomination negative 
 8 abort       negative 
 9 aborted     negative 
10 aborts      negative 
# … with 6,776 more rows
```

Sentiment analysis
========================================================

- Pride and Prejudice words tagged with sentiment 


```
# A tibble: 11 x 2
   word       sentiment
   <chr>      <chr>    
 1 abuse      negative 
 2 beauteous  positive 
 3 conciliate positive 
 4 finest     positive 
 5 guilty     negative 
 6 intimate   positive 
 7 peculiarly negative 
 8 readily    positive 
 9 suspicion  negative 
10 unjustly   negative 
11 yawn       negative 
```

========================================================

<center>
<img src="images/subjwellmeth.png" width=1500 height=600>
</center>
- Source: Hills et al. 2021. "Historical analysis of national subjective wellbeing using millions of digitized books," *Nature Human Behaviour*, 3:1271–1275.

========================================================

<center>
<img src="images/prosocmeth.png" width=1500 height=600>
</center>
- Source: Martins, Mauricio de Jesus Dias and Nicolas Baumard. 2021. "The rise of prosociality in fiction preceded democratic revolutions in Early Modern Europe," *Proceedings of the National Academy of Sciences*, 117(46) 28684–28691.

Running Your Own Analysis: Tahrir Documents
========================================================





```r
library(tidyverse) # loads dplyr, ggplot2, and others
library(readr) # more informative and easy way to import data
library(stringr) # to handle text elements
library(tidytext) # includes set of functions useful for manipulating text

pamphdata <- read_csv("https://raw.githubusercontent.com/cjbarrie/RDL-Ed/main/03-screenscrape-apis/data/pamphlets_formatted_gsheets.csv")
```

Sentiment dictionaries
========================================================
* `AFINN` from [Finn Årup Nielsen](http://www2.imm.dtu.dk/pubdb/views/publication_details.php?id=6010),
* `bing` from [Bing Liu and collaborators](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html), and
* `nrc` from [Saif Mohammad and Peter Turney](http://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm)

Getting sentiment dictionaries
========================================================

```r
get_sentiments("afinn")
```

```
# A tibble: 2,477 x 2
   word       value
   <chr>      <dbl>
 1 abandon       -2
 2 abandoned     -2
 3 abandons      -2
 4 abducted      -2
 5 abduction     -2
 6 abductions    -2
 7 abhor         -3
 8 abhorred      -3
 9 abhorrent     -3
10 abhors        -3
# … with 2,467 more rows
```

Getting sentiment dictionaries
========================================================

```r
get_sentiments("nrc")
```

```
# A tibble: 13,901 x 2
   word        sentiment
   <chr>       <chr>    
 1 abacus      trust    
 2 abandon     fear     
 3 abandon     negative 
 4 abandon     sadness  
 5 abandoned   anger    
 6 abandoned   fear     
 7 abandoned   negative 
 8 abandoned   sadness  
 9 abandonment anger    
10 abandonment fear     
# … with 13,891 more rows
```

Getting sentiment dictionaries
========================================================

```r
get_sentiments("bing")
```

```
# A tibble: 6,786 x 2
   word        sentiment
   <chr>       <chr>    
 1 2-faces     negative 
 2 abnormal    negative 
 3 abolish     negative 
 4 abominable  negative 
 5 abominably  negative 
 6 abominate   negative 
 7 abomination negative 
 8 abort       negative 
 9 aborted     negative 
10 aborts      negative 
# … with 6,776 more rows
```


Preprocess Tahrir documents
========================================================


```r
tidy_pamph <- pamphdata %>% 
  mutate(desc = tolower(text)) %>%
  unnest_tokens(word, desc) %>%
  filter(str_detect(word, "[a-z]"))

tidy_pamph <- tidy_pamph %>%
    filter(!word %in% stop_words$word)
```

Inspect
========================================================


```r
tidy_pamph %>%
  count(word, sort = TRUE)
```

```
# A tibble: 12,133 x 2
   word             n
   <chr>        <int>
 1 revolution    2051
 2 people        1564
 3 egypt         1073
 4 egyptian      1049
 5 al             905
 6 regime         804
 7 political      646
 8 party          619
 9 constitution   577
10 demands        575
# … with 12,123 more rows
```

Order by date and index
========================================================


```r
#order and format date
tidy_pamph<- tidy_pamph %>%
  arrange(date)

tidy_pamph$order <- 1:nrow(tidy_pamph)
```

Get sentiments in tidy format
========================================================


```r
#join pamphlets with nrc sentiment data
pamph_nrc_sentiment <- tidy_pamph %>%
  inner_join(get_sentiments("nrc"))
```

Get sentiments in tidy format
========================================================


```r
#calculate sentiment over per 1000 words per date
pamph_nrc_sentiment <- pamph_nrc_sentiment %>%
  count(date, index = order %/% 1000, sentiment)

head(pamph_nrc_sentiment)
```

```
# A tibble: 6 x 4
  date       index sentiment        n
  <date>     <dbl> <chr>        <int>
1 2011-03-11     0 anger           18
2 2011-03-11     0 anticipation    13
3 2011-03-11     0 disgust         16
4 2011-03-11     0 fear            25
5 2011-03-11     0 joy             12
6 2011-03-11     0 negative        32
```

Get sentiments in tidy format
========================================================


```r
#separate into 
pamph_nrc_sentiment <- pamph_nrc_sentiment %>%
  spread(sentiment, n, fill = 0)

head(pamph_nrc_sentiment)
```

```
# A tibble: 6 x 12
  date       index anger anticipation disgust  fear   joy negative positive sadness surprise trust
  <date>     <dbl> <dbl>        <dbl>   <dbl> <dbl> <dbl>    <dbl>    <dbl>   <dbl>    <dbl> <dbl>
1 2011-03-11     0    18           13      16    25    12       32       36      14        9    29
2 2011-03-13     0    26           27      11    35    18       38       89      24       11    52
3 2011-03-14     0     2            2       1     3     0        4        6       2        3     2
4 2011-03-15     0    34           25       6    31    10       51       47      27       17    27
5 2011-03-15     1    51           79      22    80    62       88      168      53       42   109
6 2011-03-15     2     3            2       1     3     4        4       18       2        1    19
```

Get sentiments in tidy format
========================================================


```r
#separate into 
pamph_nrc_sentiment <- pamph_nrc_sentiment %>%
  mutate(sentiment = positive - negative)

head(pamph_nrc_sentiment)
```

```
# A tibble: 6 x 13
  date       index anger anticipation disgust  fear   joy negative positive sadness surprise trust sentiment
  <date>     <dbl> <dbl>        <dbl>   <dbl> <dbl> <dbl>    <dbl>    <dbl>   <dbl>    <dbl> <dbl>     <dbl>
1 2011-03-11     0    18           13      16    25    12       32       36      14        9    29         4
2 2011-03-13     0    26           27      11    35    18       38       89      24       11    52        51
3 2011-03-14     0     2            2       1     3     0        4        6       2        3     2         2
4 2011-03-15     0    34           25       6    31    10       51       47      27       17    27        -4
5 2011-03-15     1    51           79      22    80    62       88      168      53       42   109        80
6 2011-03-15     2     3            2       1     3     4        4       18       2        1    19        14
```

Plot
========================================================


```r
pamph_nrc_sentiment %>%
  ggplot(aes(date, sentiment)) +
  geom_point(alpha=0.5) +
  geom_smooth(method= loess, alpha=0.25)
```

![plot of chunk unnamed-chunk-17](02-sent-analysis-pres-figure/unnamed-chunk-17-1.png)
