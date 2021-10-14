# 6 Methods

## 6.1 - 6.4

> Target

- `func (*IntSet) Len() int`      // return the number of elements
- `func (*IntSet) Remove(x int)`  // remove x from the set
- `func (*IntSet) Clear()`        // remove all elements from the set
- `func (*IntSet) Copy() *IntSet` // return a copy of the set
- `func addall`, 用`5.7 variadic function`实现
- `交集，差集，并差集`
- `func elems`, 返回集合中数字的切片


> Code
```go
package main

import (
	"bytes"
	"fmt"
)

func main() {
	var x intSet
	x.Add(63)
	x.Add(0)
	x.Add(127)

	fmt.Println("---------------- 6.1 ---------------")
	fmt.Println(x.len())

	x.remove(127)
	fmt.Println(x.has(127))

	y := x.copy()
	fmt.Println(x.string(), y.string())

	x.clear()
	fmt.Println(x)

	fmt.Println("---------------- 6.2 ---------------")

	x.addAll(63, 0, 1)
	fmt.Println("x:", x.string())

	fmt.Println("---------------- 6.3 ---------------")

	var z intSet
	z.addAll(1, 25)
	fmt.Println("x:", x.string(), "z:", z.string())
	x.differenceWith(&z)
	fmt.Println("差集:", x.string())

	x.addAll(25)
	fmt.Println("x:", x.string(), "z:", z.string())
	x.symetricDiff(&z)
	fmt.Println("xor：", x.string())

	fmt.Println("---------------- 6.4 ---------------")
	fmt.Println(x.elems())

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

func (s *intSet) string() string {
	var buf bytes.Buffer
	buf.WriteByte('{')

	for i, v := range s.words {
		if v == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			// 依次找每一位，如果那位不是0,说明此位置的数存在
			if v&(1<<uint(j)) != 0 {
				if buf.Len() > 1 {
					buf.WriteByte(' ') // 特判是否是第一个数字,不是的话，先加空格
				}
				fmt.Fprintf(&buf, "%d", 64*i+j) // 添加此数
			}
		}
	}

	buf.WriteByte('}')
	return buf.String()
}

// Ex6.1
func (s *intSet) len() int {
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

// 6.2
func (s *intSet) addAll(a ...int) {
	for _, v := range a {
		s.Add(v)
	}
}

// 6.3
func (s *intSet) intersectWith(t *intSet) {
	// 交集： 在s中，也在t中[s,t均为1--> 11--> and]
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] &= tword
		}
	}
}

func (s *intSet) differenceWith(t *intSet) {
	// 差集： 在s中，未在t中[s里为1,t里为0--> 10]
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] &= (^tword)
		}
	}
}

func (s *intSet) symetricDiff(t *intSet) {
	// 并差集： 在s中未在t中/ 在t中未在s中[不能全为0,也不能全为1 --> 01/10--> xor]
	for i, tword := range t.words {
		if i < len(s.words) {
			s.words[i] ^= tword
		}

	}
}

//6.4 use bare return & return a Slice
func (s *intSet) elems() (res []int) {
	for i, v := range s.words {
		if v == 0 {
			continue
		}
		for j := 0; j < 64; j++ {
			// 依次找每一位，如果那位不是0,说明此位置的数存在
			if v&(1<<uint(j)) != 0 {
				res = append(res, 64*i+j)
			}
		}
	}
	return
}
```

> Run & Result
```bash
---------------- 6.1 ---------------
3
false
{0 63} {0 63}
{[]}
---------------- 6.2 ---------------
x: {0 1 63}
---------------- 6.3 ---------------
x: {0 1 63} z: {1 25}
差集: {0 63}
x: {0 25 63} z: {1 25}
xor： {0 1 63}
---------------- 6.4 ---------------
[0 1 63]

[Process exited 0]
```


## Summary
1. 不同类型的接收者的方法名可以一样
2. 使用方法的优点：在另一个包调用方法的时候，可以省略包名
3. 如果需要更新接收者变量，就需要声明指针接收者。语言特性帮助我们使代码更简洁。
4. 组合类型可以使用内嵌类型实现的方法,尽管组合类型没有声明那个方法。
