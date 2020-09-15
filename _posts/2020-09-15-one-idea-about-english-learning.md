---
layout: post
title:  "[idea] a rough idea about english learning tool"
date:   2020-09-15 now
categories: idea
---

I usually listen to the BeijingHour at 7:00AM in ShangHai on my way office.

The tough thing is there is always new words to me, I had to translate them manually by google.

What If it will translated automatically so that I could just open the github and read the english-chinese version.

Here is what I trying to do, to dev a tool to translate this automatically

1. get the beijing hour txt automatically
>try to use wireshark/fiddler to get the txt url. (maybe had, left to the weekend)

2. get the txt translated automatically
>try to create a function which can call google api, input english sentence and output chinese.

3. get the english-chinese txt output into an md file
>try to parse the english text from step1 and cut them into sentence, then translate them one by one from the function in step2

4. put the 2 language md file into github every
>try to do the push automatically so I can just open the browser and read.
