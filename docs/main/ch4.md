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
