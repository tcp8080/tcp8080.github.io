---
layout: post
title:  "[go tips] How to config regexp for html tag"
categories: go
---

Today, I parsed a html page, and did some regexp in golang.

Here is the example:
First, you need to define a regexp, with function MustCompile(\`your regexp string\`), 
Then, you use the defined regexp.findAllStringSubmatch("the string need to parse", -1) to get everything

three useful tips:
1. \`your regexp string\`, ` `means you want to exactly the string which matched your regexp string
    let's take the following string for example:
    >this is the title <h2> good job </h2>. yes. I love it.

    if you want to get "good job" from the string, you need to config your exgexp string as: \`<h2>(.+)</h2>\`

    () means, you want this as the return values
    .+ means, you want everything

2. (?s) means you want to find everything including the wrap

   like, in string:
   > this is example <p> hello my wrap
     world </p>
    
   you need to define the regexp as: \`<p>(?s).+</p>\`
   if you need the "hello my wrap world" as return, you need to define as: \`<p>((?s).+)</p>\`

3. [^<] meas you do not want < in your matching string

   like, in string: 
   > this is example <p> hello world </p>
                     <p> again hello world again</p>
                     <p> third times hello world the third times </p>

   if you want the first " hello world " as the return, not the whold big three linecontext, 
   you need to define the regexp as : \`<p>(.[^<]+)<\p>\`
   also if you need to include the wrap lines, you need to define like: \`<p>((?s).[^<]+)</p>\`


```
func GetArticleContext(page string) string{
    // .+ 任何长度大于0的字符串
    // (?s).+ 包含换行符的任意字符串
    // .[^<]+ 不包含<的任意字符串
    articleRegexp := regexp.MustCompile(`<p>((?s).[^<]+)</p>`) 
    params := articleRegexp.FindAllStringSubmatch(page, -1)
    if len(params) < 2 {
        fmt.Println("could not find article contxt")
        return "No Context"
    } else {
        fmt.Println("LENTGH:", len(params))
        selectidx := 1
        for i, v := range params {
            fmt.Println("====",v[1])
            if len(v[1]) > MAX_LENTH {
                fmt.Println("------", v, i)
                selectidx = i
                break
            }
        }

        ret, err := fileop.DecodeWord(params[selectidx][1], EncodeType_GBK)
        if err != nil {
            fmt.Println("could not decode the article")
        }
        return ret
    }
}
```