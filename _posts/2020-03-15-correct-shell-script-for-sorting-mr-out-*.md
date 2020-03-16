---
layout: post
title:  "[6.824 Lab1] correct shell script for sorting mr-out*"
date:   2020-03-15 now
categories: go
---

When I run the test_mr.sh of lab1, there is a very confusion question.

In MapReduce, if we've got 10 reduce task, then we will got 10 mr-out-x in lab1, each with something like 

```
AS 1
Agnew 1
Alfonso 1
Anything 2
Appreciate 1
Are 6
B 2
Barbi 1
```

and in the 10 mr-out-x, there surely will have been severl "AS 1" things, so when we use 'sort mr-out*', 
```
A 106
A 108
A 3
A 32
A 36
A 47
A 83
A 94
ABOUT 2
ACT 8
```
but what we need in the mr-correct-wc.txt( this is used to cmp in test shell)
```
A 509
ABOUT 2
ACT 8
ACTRESS 1
ACTUAL 8
ADLER 1
ADVENTURE 12
ADVENTURES 7
AFTER 2
AGREE 16
```

I guess the test_mr.sh have got mistake, and we can easily correct the sh like this:

```sh 
#####this is the old script, only sort is used
#sort mr-out* | grep . > mr-wc-all   

#####this is the new script
###s[$1] += $2 will add all the second colomn number 
awk '{s[$1] += $2}END{ for(i in s){  print i, s[i] } }' mr-out* 
                                                   | sort > mr-wc-all  

if cmp mr-wc-all mr-correct-wc.txt
then
  echo '---' wc test: PASS
else
  echo '---' wc output is not the same as mr-correct-wc.txt
  echo '---' wc test: FAIL
  failed_any=1
fi
```

also for index test, change the sh like this:
```sh
#######this is the oldd script, only sort is used
#sort mr-out* | grep . > mr-indexer-all  

#######this is the new script
###firtly, we sort all the lines together, to get the 3rd col in order
###then, awk to add all the 2ed and 3rd cols, left an extra "," to be delete
###lastly, print substr(a[i],2) to get the 3rd col without the begging ","
sort mr-out* | grep . > mr-out-10
awk '{s[$1] += $2; a[$1]=a[$1]","$3 }END{ for(i in s)
         {  print i,s[i],substr(a[i],2) } }' mr-out-10 | sort > mr-indexer-all

if cmp mr-indexer-all mr-correct-indexer.txt
then
  echo '---' indexer test: PASS
else
  echo '---' indexer output is not the same as mr-correct-indexer.txt
  echo '---' indexer test: FAIL
  failed_any=1
```