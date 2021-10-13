# 4 Composite Types

## 4.1

> Target
- Count the num of bits that are differ in two SHA256.
- Hint: `PopCount` func.

> Code
```go
// 两个sha256的不同bit位的数量
package main

import (
	"crypto/sha256"
	"fmt"
)

func main() {

	c1 := sha256.Sum256([]byte("x"))
	c2 := sha256.Sum256([]byte("X"))

	// 先取出1byte,然后这两个异或--> 因为不同为1-->所以就可以直接加上（每一位&1）的结果即可
	res := 0
	for i := 0; i < 32; i++ {
		tmp := c1[i] ^ c2[i]
		for j := 0; j < 8; j++ {
			res += (int(tmp) >> j) & 1
		}
	}

	/*
		// m1
		// 32*8,所以每次取出1byte,然后再对这1byte的8bit进行逐位比较
		for i := 0; i < 32; i++ {
			for j := 0; j < 8; j++ {
				t1 := (c1[i] >> j) & 1
				t2 := (c2[i] >> j) & 1
				if t1 != t2 {
					res++
				}
			}
		}
	*/

	fmt.Println(res)
}
```

> Run & Result
```
125
```

## 4.2

> Target

- 默认：读取标准输入,解析SHA256
- 特殊：指定SHA384/SHA512

> Code

```go
package main

import (
	"crypto/sha256"
	"crypto/sha512"
	"flag"
	"fmt"
	"log"
	"os"
)

var fl1 = flag.String("sha512", " ", "usage for 512 code")
var fl2 = flag.String("sha384", " ", "usage for 384 code")
var help = flag.String("h", " ", "Help for this program")

func main() {
	flag.Parse()

	// 如果长度只为1,说明只有运行命令本身，读取标准输入,然后返回sha256的结果
	if len(os.Args) == 1 {
		var s []byte
		fmt.Scan(&s)
		res := sha256.Sum256(s)
		fmt.Printf("%x\n", res)
	} else {
		// 如果长度大于1,，就看是哪种然后解析输出
		if os.Args[1] == "-sha512" {
			res := sha512.Sum512([]byte(os.Args[2]))
			fmt.Printf("%x\n", res)
		} else if os.Args[1] == "-sha384" {
			res := sha512.Sum384([]byte(os.Args[2]))
			fmt.Printf("%x\n", res)
		} else {
			log.Fatalf("\n参数错误\nPlease run `go run . -h` for help")
		}
	}
}
```

> Run & Result

```bash
➜  E2 (master) ✗ go run . -sha384 x
d752c2c51fba0e29aa190570a9d4253e44077a058d3297fa3a5630d5bd012622f97c28acaed313b5c83bb990caa7da85
➜  E2 (master) ✗ go run . x
2021/10/06 21:18:16
参数错误
Please run `go run . -h` for help
exit status 1
➜  E2 (master) ✗ go run .
x
2d711642b726b04401627ca9fbac32f5c8530fb1903cc4db02258717921a4881
➜  E2 (master) ✗ go run . -a x
flag provided but not defined: -a
Usage of /tmp/go-build3248078377/b001/exe/E2:
  -h string
        Help for this program (default " ")
  -sha384 string
        usage for 384 code (default " ")
  -sha512 string
        usage for 512 code (default " ")
exit status 2
```

## 4.3

> Target

- Rewrite `reverse` to use an array pointer instead of a slice.

> Code

```go
package main

import "fmt"

const N = 6

func reverse(s *[N]int) {
	for i, j := 0, N-1; i < j; i, j = i+1, j-1 {
		s[i], s[j] = s[j], s[i]
	}
}

func main() {
	a := [N]int{1, 2, 3, 4, 5, 0}
	reverse(&a)
	fmt.Print(a)
}
```



> Run & Result

```bash
[0 5 4 3 2 1]
```

## 4.4

> Target

- Write `rotate` that operates in a single pass.

> Code

```go
package main

import "fmt"

var s []int

func main() {
	s = []int{0, 1, 2, 3, 4, 5}
	rotate(2)
	fmt.Println(s)
}

// 先将k个数加到slice后面,再取切片
func rotate(k int) {
	for i := 0; i < k; i++ {
		s = append(s, s[i])
	}
	s = s[k:]
}
```

> Run & Result
```bash
[2 3 4 5 0 1]

[Process exited 0]
```

## 4.5

> Target

- Write an `in-place` func to eliminate adjacent duplicates in a [  ]string slice.

> Code

```go
package main

import "fmt"

func main() {
	data := []string{"a", "b", "b", "a", "c", "e", "e", "f"}
	fmt.Printf("%q\n", eaDup(data))
}

func eaDup(strings []string) []string {
	out := strings[:0]
	for i := 0; i < len(strings); i++ {
		if i == 0 || strings[i] != out[len(out)-1] {
			out = append(out, strings[i])
		}
	}
	return out
}
```

> Run & Result

```bash
["a" "b" "a" "c" "e" "f"]

[Process exited 0]
```

## 4.8 

> Target

- Modify `charcount` to count letters, digits, and so on.

> Code

```go
package main

import (
	"bufio"
	"fmt"
	"io"
	"os"
	"unicode"
	"unicode/utf8"
)

func main() {
	counts := make(map[rune]int)    // unicode字符数
	invalid := 0                    // 不合法的utf8字符
	var utflen [utf8.UTFMax + 1]int // utf8编码长度

	in := bufio.NewReader(os.Stdin)

	var letter, number, other int
	for {
		r, n, err := in.ReadRune() // 返回3个值。解码的rune字符的值，字符UTF-8编码后的长度，和一个错误值
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Fprintf(os.Stderr, "charcount: %v/n", err)
			os.Exit(1)
		}

		// 不合法字符的统计
		if r == unicode.ReplacementChar && n == 1 {
			invalid++
			continue
		}
		counts[r]++
		utflen[n]++

		if unicode.IsDigit(r) {
			number++
		} else if unicode.IsLetter(r) {
			letter++
		} else {
			other++
		}
	}

	// 输出不同字符的个数
	fmt.Printf("rune\tcount\n")
	for c, n := range counts {
		fmt.Printf("%q\t%d\n", c, n)
	}

	// 输出各个长度的有多少个
	fmt.Print("\nlen\tcount\n")
	for i, n := range utflen {
		if i > 0 {
			fmt.Printf("%d\t%d\n", i, n)
		}
	}

	// 输出invalid
	if invalid > 0 {
		fmt.Printf("\n%d invalid UTF-8 characters\n", invalid)
	}

	// E4.8 输出不同类型的
	fmt.Printf("\nletter\tdigit\tother\n")
	fmt.Printf("%d\t%d\t%d\n", letter, number, other)
}
```

> Run & Result

```bash
➜  charcount (master) ✗ go run .
world
123
rune    count
'w'     1
'd'     1
'1'     1
'2'     1
'o'     1
'r'     1
'l'     1
'\n'    2
'3'     1

len     count
1       10
2       0
3       0
4       0

letter  digit   other
5       3       2
```

## 4.9

> Target

- Report the frequency of each word in an input text file.

> Code

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	counts := make(map[string]int)

	// input
	input := bufio.NewScanner(os.Stdin)
	input.Split(bufio.ScanWords)

	// map
	for input.Scan() {
		counts[input.Text()]++
	}

	// output
	fmt.Printf("string\tcounts\n")
	for k, v := range counts {
		fmt.Printf("%s\t%d\n", k, v)
	}
}
```

> Run & Result

```bash
➜  E9 (master) ✗ go run .
aaa bbb ccc 123 213321 tnes a ntes zz 43
a zz aaa
string  counts
tnes    1
zz      2
ccc     1
123     1
213321  1
a       2
ntes    1
43      1
aaa     2
bbb     1
```

## 4.13 

> Target

> Code
```go
package main

import (
	"encoding/json"
	"fmt"
	"io"
	"log"
	"net/http"
	"os"
)

type movieRes struct {
	Title, Year, Released, Runtime, Genre, Director, Language, Country, Awards, Poster, Metascore string
}

func main() {
	// 注意，这里要换成你的apikey,并删除`尖括号`
	preUrl := "http://www.omdbapi.com/?apikey=<your apikey>&t="
	if len(os.Args) < 3 {
		log.Fatalf("\nError!\nExample:go run . -t/-d <film name>")
	}
	url := preUrl + os.Args[2]

	mr := seach(url)

	if os.Args[1] == "-s" {
		if mr.Title != "" {
			fmt.Printf("%-10s  %s\n", "TAGS", "RES")
			fmt.Printf("-----------------------\n")
			fmt.Printf("%-10s: %s\n", "Title", mr.Title)
			fmt.Printf("%-10s: %s\n", "Year", mr.Year)
			fmt.Printf("%-10s: %s\n", "Runtime", mr.Runtime)
			fmt.Printf("%-10s: %s\n", "Genre", mr.Genre)
			fmt.Printf("%-10s: %s\n", "Director", mr.Director)
			fmt.Printf("%-10s: %s\n", "Awards", mr.Awards)
			fmt.Printf("%-10s: %s\n", "Metascore", mr.Metascore)
		} else {
			fmt.Printf("抱歉，没有找到您要的电影🥴\n")
		}
	} else if os.Args[1] == "-d" {
		if mr.Poster != "" {
			curl(mr.Poster, os.Args[2]+".jpg")
			fmt.Printf("Download Successd!\nFile Name is %s\n", os.Args[2]+".jpg")
		} else {
			fmt.Printf("抱歉，没有找到您要的电影🥴\n")
		}
	} else {
		log.Fatalf("\nError!\nExample:go run . -t/-d <film name>")
	}

}

// parse
func seach(url string) movieRes {
	resp, err := http.Get(url)
	if err != nil {
		log.Fatalln(err)
	}
	defer resp.Body.Close()

	// 将json响应解码到结构类型
	var mr movieRes
	err = json.NewDecoder(resp.Body).Decode(&mr)
	if err != nil {
		log.Fatalln(err)
	}

	return mr
}

// download
func curl(url string, fileName string) {
	r, err := http.Get(url)
	if err != nil {
		log.Fatalln(err)
	}
	defer r.Body.Close()

	f, err := os.Create(fileName)
	if err != nil {
		log.Fatalln(err)
	}
	defer f.Close()

	io.Copy(f, r.Body)
}
```

> Run & Result

[Result Video](../_files/ch4/e4-13.mp4 ':include  :type=iframe width=100% height=540px loading:lazy sandbox')
