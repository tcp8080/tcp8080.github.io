---
layout: post
title:  "[go tips] How to call translate api from google freely"
date:   2020-09-25 now
categories: go
---

like I said in [this artcle](https://tcp8080.github.io/idea/2020/09/15/one-idea-about-english-learning.html)

I could like to create a way to checkout the new words of the News quickly, instead of google it word by word.

Here is what I did:

1. get the txt file fo the News context.
2. translate the file sentence by sentence, so that I could see the bilingual senetence both in Chinese and in English at the same time, and in that way I could easily get the new words meaning.

For step 1, I was considering download it from EasyFM app automatically, but it seems very hard to get the download url, so I decided to do step 1 manually first, by copy the context from app and send it to my mac though email. I then created the txt file from the email's context.

For step2, it is much easy, there are really very much good repostory in github to use.

>github.com/bregydoc/gtranslate //calls the google tranlate page
github.com/robertkrimen/otto //a JavaScript parser and interpreter written natively in Go.


Here is what I do :

```go
import (
    "fmt"
    "os"
    "bufio"
    "strings"
    "bytes"
    "github.com/bregydoc/gtranslate"
)

func transSentence( sentence string) string{
    if len(sentence) == 0 {
      return "\n----------------\n"
    }

    translated, err := gtranslate.TranslateWithParams(
                                sentence,
                                gtranslate.TranslationParams{
                                                From: "en",
                                                To:   "zh",
                                },

                )
    if err != nil {
        panic(err)
    }

    return translated
}


func translateArticle(filename string, translatedfile string) {
    
    file, err := os.Open(filename)
    if err != nil {
      fmt.Println(err)
      return
    }
    defer file.Close()

    scanner := bufio.NewScanner(file)
    scanner.Split(bufio.ScanLines)

    var lines []string

    for scanner.Scan() {
      lines = append(lines, scanner.Text())
    }

    f, err := os.Create(translatedfile)
    check(err)
    defer f.Close()
    
    var buffer bytes.Buffer
    for _, line := range lines {
      linestr := strings.TrimSpace(line)
      chstr := transSentence(linestr)

      buffer.WriteString(strings.Join([]string{linestr}, "\n\n"))
      buffer.WriteString(strings.Join([]string{chstr}, "\n\n"))
        
    }

    _, err = f.WriteString(buffer.String())
}
```

with the txt I got from the email. and call the translateArticle function, I can got a bilingual file. 

Input file is like 
```
    China says it will continue to share its experiences in fighting against COVID-19 with the rest of the world...
     Most of the new COVID cases in Hong Kong recently have been imported --- not transmitted locally...
     "It has changed the situation completely since early July -- the time of the third wave of the COVID-19 outbreak -- when most of the cases were tending to be local cases...
     The woman accused of sending a poison-laced letter to the U.S. president has made her first court appearance...
```

Output file is like:
```
China says it will continue to share its experiences in fighting against COVID-19 with the rest of the world...
中国表示，它将继续与世界其他地区分享其对抗COVID-19的经验。

Most of the new COVID cases in Hong Kong recently have been imported --- not transmitted locally...
香港最近大多数新的COVID病例都是进口的---不在本地传播...

"It has changed the situation completely since early July -- the time of the third wave of the COVID-19 outbreak -- when most of the cases were tending to be local cases...
“自7月初以来（第三次COVID-19疫情爆发之时），情况已经完全改变了，当时大多数病例倾向于本地病例...

The woman accused of sending a poison-laced letter to the U.S. president has made her first court appearance...
这位被指控向美国总统寄上了毒药的信的妇女已首次出庭...
```

For the Chinese Mainland people, you should make sure you can ping google.com first.

首先，确保你跑应用的机器网络和google是通的。

Q&A
1. 翻译出来的中文乱码？

理论上google 翻译出来的中文都是符合utf8格式的，所以不应该出现乱码的问题。
我今天出现这个问题，最后调查发现是因为输入的纯英文文件本身有问题，之后重新保存成utf8一次，再调用翻译接口，问题解决。