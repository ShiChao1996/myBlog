# bytes.go 源码简析

> 学习bytes的时候，网上找了很多文章，但基本上是用法，很少对代码的分析。这篇文章对bytes.go 做了简要的分析。

首先从 `import` 的包来看，只引入了`unicode` 包的部分函数，所以这个文件代码主要是对字节切片做字符处理。从主要的功能来分，bytes.go 分为几个部分：
* index/contain 系列函数
* splite 拆分函数
* Map 遍历函数
* trim 修剪函数

### prepare
开始对每个部分分析前，先来看一下 `unicode` 包里用的比较多的内容。
##### 常量
```go
const (
	RuneError = '\uFFFD'     // 这是一个特殊的字符，用于表示“非法”的unicode
	RuneSelf  = 0x80         // 十六进制的80，等于十进制的127，0~127刚好表示128个ASCII码。一个字符的字节码大于127，表示不是ASCII，可能是unicode
	MaxRune   = '\U0010FFFF' // 最大的unicode编码
	UTFMax    = 4            // 一个unicode码最大的字节长度
)
```
##### 函数
utf8.DecodeRune(p []byte)(r rune, size int):从字节切片中解析出第一个完整的unicode码，
返回一个`rune`类型的`r`（第一个unicode），和`r`的真实长度size，从代码中可看出，size的值是0~4。简单看一下返回值就好，这不是本文的重点：
```go
func DecodeRune(p []byte) (r rune, size int) {
	n := len(p)
	if n < 1 {
		return RuneError, 0
	}
	p0 := p[0]
	x := first[p0]
	if x >= as {
		// The following code simulates an additional check for x == xx and
		// handling the ASCII and invalid cases accordingly. This mask-and-or
		// approach prevents an additional branch.
		mask := rune(x) << 31 >> 31 // Create 0x0000 or 0xFFFF.
		return rune(p[0])&^mask | RuneError&mask, 1
	}
	sz := x & 7
	accept := acceptRanges[x>>4]
	if n < int(sz) {
		return RuneError, 1
	}
	b1 := p[1]
	if b1 < accept.lo || accept.hi < b1 {
		return RuneError, 1
	}
	if sz == 2 {
		return rune(p0&mask2)<<6 | rune(b1&maskx), 2
	}
	b2 := p[2]
	if b2 < locb || hicb < b2 {
		return RuneError, 1
	}
	if sz == 3 {
		return rune(p0&mask3)<<12 | rune(b1&maskx)<<6 | rune(b2&maskx), 3
	}
	b3 := p[3]
	if b3 < locb || hicb < b3 {
		return RuneError, 1
	}
	return rune(p0&mask4)<<18 | rune(b1&maskx)<<12 | rune(b2&maskx)<<6 | rune(b3&maskx), 4
}
```

### index
这里有一系列的index的函数:
* func Contains(b, subslice []byte) bool
* func Count(s, sep []byte) int
* func Index(s, sep []byte) int
* func IndexByte(s []byte, c byte) int
* func IndexRune(s []byte, r rune) int
* func IndexAny(s []byte, chars string) int
* func IndexFunc(s []byte, f func(r rune) bool) int
* func LastIndex(s, sep []byte) int
* func LastIndexAny(s []byte, chars string) int
* func LastIndexFunc(s []byte, f func(r rune) bool) int
* ...

但是基本上是在调用同一个函数： `func Index(s, sep []byte) int `

```go
func Index(s, sep []byte) int {
	n := len(sep)
	switch {
	case n == 0:
		return 0
	case n == 1:
		return IndexByte(s, sep[0])
	case n == len(s):
		if Equal(sep, s) {
			return 0
		}
		return -1
	case n > len(s):
		return -1
	case n <= shortStringLen:
		// Use brute force when s and sep both are small
		if len(s) <= 64 {
			return indexShortStr(s, sep)
		}
		c := sep[0]
		i := 0
		t := s[:len(s)-n+1]
		fails := 0
		for i < len(t) {
			if t[i] != c {
				// IndexByte skips 16/32 bytes per iteration,
				// so it's faster than indexShortStr.
				o := IndexByte(t[i:], c)
				if o < 0 {
					return -1
				}
				i += o
			}
			if Equal(s[i:i+n], sep) {
				return i
			}
			fails++
			i++
			// Switch to indexShortStr when IndexByte produces too many false positives.
			// Too many means more that 1 error per 8 characters.
			// Allow some errors in the beginning.
			if fails > (i+16)/8 {
				r := indexShortStr(s[i:], sep)
				if r >= 0 {
					return r + i
				}
				return -1
			}
		}
		return -1
	}
	return indexRabinKarp(s, sep)
}
```
可以看到，是几个 case，主要是排除掉一些简单的情况。从`s`中找到第一次出现`sep`的位置，当sep长度为1，sep就是一个字节，
直接调用`IndexByte（s, sep[0]）`,就是把`s`里的字节挨个取出和sep[0]对比。
`case n <= shortStringLen:`表示最普遍的情况，这里使用了相关匹配算法，就不深入谈论了。


再简单看下`func IndexAny(s []byte, chars string) int`函数，返回`s`中第一次出现`chars`中任一字符的位置。
注意chars是string类型，可能包含ASCII和unicode，所以要先对`chars`做处理：
```go
func IndexAny(s []byte, chars string) int {
	if chars == "" {
		// Avoid scanning all of s.
		return -1
	}
	if len(s) > 8 {
		if as, isASCII := makeASCIISet(chars); isASCII {
			for i, c := range s {
				if as.contains(c) {
					return i
				}
			}
			return -1
		}
	}
	var width int
	for i := 0; i < len(s); i += width {
		r := rune(s[i])
		if r < utf8.RuneSelf {
			width = 1
		} else {
			r, width = utf8.DecodeRune(s[i:])
		}
		for _, ch := range chars {
			if r == ch {
				return i
			}
		}
	}
	return -1
}
```