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

> Run&Result
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



# 5.4

> Target
- Extend the `visit` func so that it extracts other kinds of links from the document, such as images, scripts, and style sheets.

> Code
```go
package main

import (
	"fmt"
	"log"
	"os"

	"golang.org/x/net/html"
)

func main() {
	tag = map[string]bool{"a": true, "img": true, "script": true, "href": true, "src": true}

	doc, err := html.Parse(os.Stdin)
	if err != nil {
		log.Fatalf("E4:%s", err)
	}

	for _, link := range Vt(nil, doc) {
		fmt.Println(link)
	}

}

var tag map[string]bool

func Vt(links []string, n *html.Node) []string {
	if n.Type == html.ElementNode && tag[n.Data] {
		for _, a := range n.Attr {
			if tag[a.Key] {
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

> Run&Result
```bash
➜  E4 (master) ✗ go run . < ../findlinks1/html_input
/lib/godoc/jquery.js
/lib/godoc/playground.js
/lib/godoc/godocs.js
https://support.eji.org/give/153413/#!/donation/checkout
/
/lib/godoc/images/go-logo-blue.svg
/doc/
/pkg/
/project/
/help/
/blog/
https://play.golang.org/
/dl/
/lib/godoc/images/cloud-download.svg
https://tour.golang.org/
https://blog.golang.org/
/lib/godoc/images/footer-gopher.jpg
/doc/copyright.html
/doc/tos.html
http://www.google.com/intl/en/policies/privacy/
http://golang.org/issues/new?title=x/website:
https://google.com
```
- _备注_: 未添加`style sheets`, 因为不清楚这个。

# 5.5

> Target
- Implement `countWordsAndImgs` (See Ex4.9 for word-splitting)

> Code
```go
package main

import (
	"bufio"
	"fmt"
	"net/http"
	"strings"

	"golang.org/x/net/html"
)

func main() {
	words, images, _ := count("https://www.zhihu.com")
	fmt.Printf("words: %d\timages: %d\n", words, images)
}

func count(url string) (words, images int, err error) {
	resp, err := http.Get(url)
	if err != nil {
		return // bare return
	}

	doc, err := html.Parse(resp.Body)
	if err != nil {
		err = fmt.Errorf("parsing HTML: %s", err)
	}
	defer resp.Body.Close()

	words, images = countWordsAndImgs(doc)
	return
}

func countWordsAndImgs(n *html.Node) (int, int) {
	textNode(n) // 利用E3的函数,得到所有文本元素
	calcImg(n)  // 利用E2的函数，得到img的个数

	// 对文本元素计数
	words := 0
	input := bufio.NewScanner(strings.NewReader(resWords))
	input.Split(bufio.ScanWords)
	for input.Scan() {
		words++
	}
	return words, resImgs
}

var resWords string

func textNode(n *html.Node) {
	if n.Type == html.TextNode {
		resWords = resWords + " " + n.Data
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		if c.Data != "script" && c.Data != "style" {
			textNode(c)
		}
	}
}

var resImgs int

func calcImg(n *html.Node) {
	if n.Type == html.ElementNode {
		resImgs++
	}

	for c := n.FirstChild; c != nil; c = c.NextSibling {
		calcImg(c)
	}
}
```

> Run&Result
```bash
➜  E5 (master) ✗ go run .
words: 60       images: 152
```

