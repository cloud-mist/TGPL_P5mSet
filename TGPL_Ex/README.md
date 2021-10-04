# 1 Tutorial
## 1.1
> Target
- Also print os.Args[0].

> Code
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for _, v := range os.Args[:] {
		fmt.Println(v)
		// Args[0]:代表着执行命令本身的名字
	}
}
```
> Run & Result
```bash
➜  E1 (master) ✗ go run . c1 c2 c3
/tmp/go-build4220552769/b001/exe/E1
c1
c2
c3
```
## 1.2
> Target
- Print the `index & value` of each of its arguments.

> Code
```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for i, arg := range os.Args[1:] {
		fmt.Println(i+1, arg)
	}
}
```

> Run & Result
```bash
➜  E2 (master) ✗ go run . c1 c2 c3
1 c1
2 c2
3 c3
```

# 5 Func




