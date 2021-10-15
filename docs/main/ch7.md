# 7 Interfaces

## 7.1

> Target

- `Counters` for words and for lines.
- Hint: ByteCounter, bufio.ScanWords

> Code

```go
package main

import (
	"bufio"
	"bytes"
	"fmt"
	"strings"
)

func main() {
	var c wordLineCounter
	c.Write([]byte("hello world\nthis is second line."))
	fmt.Println(c)

	c = wordLineCounter{0, 0}
	sth := "ni hao\ntest\n1 2 3 4 5"
	fmt.Fprintf(&c, "%s", sth)
	fmt.Println(c)
}

type wordLineCounter struct {
	wordNum, lineNum int
}

// 默认按行分割，直接统计行。然后拿出每一行，再按单词分隔来统计单词
func (w *wordLineCounter) Write(p []byte) (n int, err error) {
	input := bufio.NewScanner(bytes.NewReader(p))
	for input.Scan() {
		(*w).lineNum++
		tmp := input.Text()
		ipt := bufio.NewScanner(strings.NewReader(tmp))
		ipt.Split(bufio.ScanWords)
		for ipt.Scan() {
			(*w).wordNum++
		}
	}
	n = len(p)
	return
}
```

> Run & Result

```bash
➜  wordLineCounter (master) ✗ go run .
{6 2}
{8 3}
```
