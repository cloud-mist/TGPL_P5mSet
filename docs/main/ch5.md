# 5 Func

## 5.1

> Target

- Using recursive calls to `visit` instead of a loop.

> Code

```go
func Vt(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && n.Data == "a" {
		for _, a := range n.Attr {
			if a.Key == "href" {
				links = append(links, a.Val)
			}
		}
	}
	
	if n.FirstChild != nil {
		links = Vt(links, n.FirstChild)
	}
	if n.NextSibling != nil {
		links = Vt(links, n.NextSibling)
	}

	return links
}
```

> Res
```bash
➜  findlinks1 (master) ✗ go run . < html_input
https://support.eji.org/give/153413/#!/donation/checkout
/
/doc/
/pkg/
/project/
/help/
/blog/
https://play.golang.org/
/dl/
https://tour.golang.org/
https://blog.golang.org/
/doc/copyright.html
/doc/tos.html
http://www.google.com/intl/en/policies/privacy/
http://golang.org/issues/new?title=x/website:
https://google.com
```

# 5.2

> Target
- Write a func to populate a mapping from elem names to the num of elems with that name in an HTML documents tree.

> Code
```go
package main

import (
	"fmt"
	"log"
	"os"

	"golang.org/x/net/html"
)

var res map[string]int

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		log.Fatalf("E2:%s", err)
	}

	// Init&Calc
	res = make(map[string]int)
	calc(doc)

	// Output
	cnt := 0
	for i, v := range res {
		fmt.Printf("%-10s: %-8d", i, v)
		cnt++
		if cnt == 3 {
			fmt.Println()
			cnt = 0
		}
	}
}

// 用map储存结果
func calc(n *html.Node) {
	if n.Type == html.ElementNode {
		res[n.Data]++
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		calc(c)
	}
}
```

> Run&Result
```bash
➜  E2 (master) ✗ go run . < ../findlinks1/html_input
div       : 14      noscript  : 1       html      : 1
title     : 1       header    : 1       iframe    : 1
link      : 3       script    : 8       body      : 1
img       : 3       li        : 10      meta      : 4
a         : 17      nav       : 1       br        : 1
section   : 4       p         : 1       textarea  : 1
pre       : 1       footer    : 1       head      : 1
main      : 1       strong    : 3       ul        : 2
i         : 1       h2        : 3       button    : 3
h1        : 1       select    : 1       option    : 8
```

# 5.3

> Target
- Write a func to print the contents of all `text nodes` in an HTML document tree.
- Do not descent into \<script> or \<style> elems.

> Code
```go
package main

import (
	"fmt"
	"log"
	"os"

	"golang.org/x/net/html"
)

var res []string

func main() {
	doc, err := html.Parse(os.Stdin)
	if err != nil {
		log.Fatalf("E3:%s", err)
	}

	res = make([]string, 0)
	textNode(doc)

	fmt.Println(res)
}


func textNode(n *html.Node) {
	if n.Type == html.TextNode {
		res = append(res, n.Data)
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		if c.Data != "script" && c.Data != "style" {
			textNode(c)
		}
	}
}
```

> Run&Result
```bash
➜  E3 (master) ✗ go run . < ../findlinks1/html_input > e3Result.txt
```
- Download: [e3Result.txt](../_files/ch5/e3Result.txt ':ignore')



