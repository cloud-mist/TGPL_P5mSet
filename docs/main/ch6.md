# 6 Methods
## Summary
1. 不同类型的接收者的方法名可以一样
2. 使用方法的优点：在另一个包调用方法的时候，可以省略包名
3. 如果需要更新接收者变量，就需要声明指针接收者。语言特性帮助我们使代码更简洁。
4. 组合类型可以使用内嵌类型实现的方法,尽管组合类型没有声明那个方法。

## 6.1

> Target

> Code
```go
package main

import "fmt"

func main() {
	var x intSet
	x.Add(63)
	x.Add(0)
	x.Add(127)

	// 测试Ex1
	fmt.Println(x.len())

	x.remove(127)
	fmt.Println(x.has(127))

	y := x.copy()
	fmt.Printf("x:%b\ny:%b\n", x, *y)

	x.clear()
	fmt.Println(x)

}

// 一个word有64位，代表64个数，word[0]={0~63},word[1]={64~127}
type intSet struct {
	words []uint64
}

func (s *intSet) has(x int) bool {
	word, bit := x/64, uint(x%64)
	// word大的话，肯定不在，那么就看所在那位是不是1
	return word < len(s.words) && s.words[word]&(1<<bit) != 0
}

func (s *intSet) Add(x int) {
	word, bit := x/64, uint(x%64) // word：索引位置，bit该索引的那个位置

	// 如果没有位置，添加0,直到有我的位置
	for word >= len(s.words) {
		s.words = append(s.words, 0)
	}
	// 把那个数字应该放的位置置为1
	s.words[word] |= 1 << bit
}

func (s *intSet) unionWith(t *intSet) {
	// 对于t的每一段，如果i小于，说明还在s的范围内，就进行add操作，如果超出，那么直接append来添加这段。
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] |= tword
		} else {
			s.words = append(s.words, tword)
		}
	}
}

// Ex6.1
func (s intSet) len() int {
	// 那就是依次找每段的1的个数相加,就是元素个数
	res := 0
	for _, v := range s.words {
		res += calcOneNumber(v)
	}
	return res
}

func (s *intSet) remove(x int) {
	// 找到x所在位置，将其置为0
	word, bit := x/64, uint(x%64)
	s.words[word] &= (^(1 << bit)) // 注意：`^`是golang的取反操作
}

func (s *intSet) clear() {
	// 移除所有元素
	s.words = s.words[:0]
}

func (s *intSet) copy() *intSet {
	var res intSet
	res.words = s.words[:]
	return &res
}

// 计算数字x二进制下1的位数的个数
func calcOneNumber(x uint64) int {
	res := 0
	for x != 0 {
		x -= x & (-x)
		res++
	}
	return res
}
```

> Run & Result
```
3
false
x:{[1000000000000000000000000000000000000000000000000000000000000001 0]}
y:{[1000000000000000000000000000000000000000000000000000000000000001 0]}
{[]}

```
