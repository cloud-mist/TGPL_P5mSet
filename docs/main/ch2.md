# 2 Program Struct

## 2.1
> Target
- 添加`Kelvin`的转换

> Code

- _code tree_

```bash
➜  ch2 (master) ✗ tree Ex1
Ex1
├── main.go
└── tempconv
    └── tmpconv.go

1 directory, 2 files
```

- _main.go_

```go
package main

import "Golang_learning/TGPL/ch2/Ex1/tempconv"

func main() {
	a := tempconv.F2C(212.0)
	tempconv.Format(a)

	b := tempconv.C2K(100)
	tempconv.Format(b)
}
```

- _tmpconv.go_

```go
package tempconv

import (
	"fmt"
)

// 虽然有相同底层类型，但是是不同的数据类型
type Cel float64 // 摄氏
type Fah float64 // 华氏
type Kel float64 // kelvin

const (
	Absolute0C Cel = -273.15 // 绝对零度
	FreezingC  Cel = 0       // 结冰温度
	BoilingC   Cel = 100     // 沸水温度
)

// 定义一个接口
type change interface {
	String()
}

// 多态函数，将接口作为参数
func Format(c change) {
	c.String()
}

func C2F(c Cel) Fah {
	return Fah(c*9/5 + 32)
}
func F2C(f Fah) Cel {
	return Cel((f - 32) * 5 / 9)
}

// kelvin <--> Cel
func K2C(k Kel) Cel {
	return Cel(k) - Absolute0C
}
func C2K(c Cel) Kel {
	return Kel(c + Absolute0C)
}

// kelvin <--> Fah
func K2F(k Kel) Fah {
	return C2F(K2C(k))
}
func F2K(f Fah) Kel {
	return C2K(F2C(f))
}

//3种自定义类型的实现了change接口,所以可以使用定义的接口的函数
func (c Cel) String() {
	fmt.Printf("%.2f°C\n", c)
}
func (f Fah) String() {
	fmt.Printf("%.2f°F\n", f)
}
func (k Kel) String() {
	fmt.Printf("%.2fK\n", k)
}
```


