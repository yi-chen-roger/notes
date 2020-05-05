# A Tour of GO
https://tour.golang.org/welcome/1

## 1. Welcome
formatted with [gofmt](https://golang.org/cmd/gofmt/)

## 2. Basics
### 2.1  Packages, variables, and functions. 

#### Packages
- Programs start running in package `main`=
-  Every Go program is made up of packages
- 首字母大写的名称是被导出的。 


``` go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	fmt.Println("My favorite number is", rand.Intn(10))
}

```


#### Functions
- 类型在变量名之后
- 当两个或多个连续的函数命名参数是同一类型，可以只写最后一个
-  函数可以返回任意数量的返回值

``` go
func swap(x, y string) (string, string) {
	return y, x
}
```
- Go 的返回值可以被命名，并且像变量那样使用。 
``` go

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
```

#### Variables
 - `var` 语句定义了一个变量的列表，类型在后面。 
 - `var` 语句可以定义在包或函数级别。 
 -  变量定义可以包含初始值，每个变量对应一个。 
``` go
var i, j int = 1, 2
```

Short variable declarations
- 在函数中，`:=` 简洁赋值语句在明确类型的地方，可以用于替代 var 定义。
- `:=` 结构不能使用在函数外。 
``` go
package main

import "fmt"

func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"

	fmt.Println(i, j, k, c, python, java)
}

```

#### Types
类型
```
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

零值
- 数值类型为 `0`，
- 布尔类型为 `false`，
- 字符串为 `""`.

类型转换
- 需要显式转换

``` go
var x, y int = 3, 4
var f float64 = math.Sqrt(x*x + y*y)
var z int = int(f)
```

类型推导
-  在定义一个变量但不指定其类型时，变量的类型由右值推导得出。 
    - 当右值定义了类型时，新变量的类型与其相同： 
    - 当右边包含了未指名类型的数字常量时，新的变量就可能是`int` 、 `float64` 或 `complex128`。 这取决于常量的精度
``` go
func main() {
	v := 42 // change me!
	fmt.Printf("v is of type %T\n", v)
}
```
#### Constant Variable
- 使用 `const` 关键字
- 常量不能使用 := 语法定义。 
``` go
const World = "世界"
```
-  数值常量是高精度的值 
-  一个未指定类型的常量由上下文来决定其类型。 
``` go
package main

import "fmt"

const (
	// Create a huge number by shifting a 1 bit left 100 places.
	// In other words, the binary number that is 1 followed by 100 zeroes.
	Big = 1 << 100
	// Shift it right again 99 places, so we end up with 1<<1, or 2.
	Small = Big >> 99
)

func needInt(x int) int { return x*10 + 1 }
func needFloat(x float64) float64 {
	return x * 0.1
}

func main() {
	fmt.Println(needInt(Small))
	fmt.Println(needFloat(Small))
	fmt.Println(needFloat(Big))
}
```

### 2.2 Flow control statements: for, if, else, switch and defer
#### for 
- Go 只有一种循环结构——`for` 循环
- 没有`（）`，但是强制`{}`
    ``` go
    for i := 0; i < 10; i++ {
        sum += i
    }
    ```
- 省略分号和c语言的`while`一样
    ``` go
    for sum < 1000 {
        sum += sum
    }
    ```
- 死循环
    ``` go
    for {
	}
    ```
#### if
- 没有`（）`，但是强制`{}`
    ``` go
    if x < 0 {
        return sqrt(-x) + "i"
    }
    ```
-  跟 for 一样，`if` 语句可以在条件之前执行一个简单的语句。 
    ``` go
    func pow(x, n, lim float64) float64 {
        if v := math.Pow(x, n); v < lim {
            return v
        }
        return lim
    }
    ```
#### switch
-  除非以 `fallthrough` 语句结束，否则分支会自动终止。 
    ``` go
    switch os := runtime.GOOS; os {
    case "darwin":
        fmt.Println("OS X.")
    case "linux":
        fmt.Println("Linux.")
    default:
        // freebsd, openbsd,
        // plan9, windows...
        fmt.Printf("%s.", os)
    }
    ```
- switch的条件从上倒下执行
    ``` go
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        fmt.Println("When's Saturday?")
        today := time.Now().Weekday()
        switch time.Saturday {
        case today + 0:
            fmt.Println("Today.")
        case today + 1:
            fmt.Println("Tomorrow.")
        case today + 2:
            fmt.Println("In two days.")
        default:
            fmt.Println("Too far away.")
        }
    }
    ```
-  没有条件的 switch 同很长的if-else一样
    ``` go
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        t := time.Now()
        switch {
        case t.Hour() < 12:
            fmt.Println("Good morning!")
        case t.Hour() < 17:
            fmt.Println("Good afternoon.")
        default:
            fmt.Println("Good evening.")
        }
    }
    ```

#### defer
- defer 语句会延迟函数的执行直到上层函数返回。
- 延迟调用的参数会立刻生成，但是在上层函数返回前函数都不会被调用。
-  延迟的函数调用被压入一个栈中。当函数返回时， 会按照后进先出的顺序调用被延迟的函数调用。     [博文](https://blog.golang.org/defer-panic-and-recover) 
    ``` go
    package main

    import "fmt"

    func main() {
        fmt.Println("counting")

        for i := 0; i < 10; i++ {
            defer fmt.Println(i)
        }

        fmt.Println("done")
    }
    ```


### 2.3 More types: structs, slices, and maps.
#### Pointer
- Go 具有指针
    ``` go
    var p *int
    i := 42
    p = &i
    ```
#### struct
- 结构体字段可以通过结构体指针来访问
    ``` go
    package main

    import "fmt"

    type Vertex struct {
        X int
        Y int
    }

    func main() {
        v := Vertex{1, 2}
        p := &v
        p.X = 1e9
        fmt.Println(v)
        fmt.Println(v.X)

    }
    ```
- 结构体语法
    ``` go
    package main

    import "fmt"

    type Vertex struct {
        X, Y int
    }

    var (
        v1 = Vertex{1, 2}  // 类型为 Vertex
        v2 = Vertex{X: 1}  // Y:0 被省略
        v3 = Vertex{}      // X:0 和 Y:0
        p  = &Vertex{1, 2} // 类型为 *Vertex
    )

    func main() {
        fmt.Println(v1, p, v2, v3)
    }

    ```
#### array
-  类型 [n]T 是一个有 n 个类型为 T 的值的数组， 数组不能改变大小
```go
package main

import "fmt"

func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```
#### slice
- 改slice的值，会影响本来的值
``` go
package main

import "fmt"

func main() {
    names := [4]string{
        "John",
        "Paul",
        "George",
        "Ringo",
    }
    fmt.Println(names) //[John Paul George Ringo]

    a := names[0:2]
    b := names[1:3]
    fmt.Println(a, b) //[John Paul] [Paul George]

    b[0] = "XXX"
    fmt.Println(a, b) //[John XXX] [XXX George]
    fmt.Println(names) //[John XXX George Ringo]
    // 省略上下标
    fmt.Println("names[:1] ==", names[:1])
    fmt.Println("names[2:] ==", names[2:])
    fmt.Println("names[:] ==", names[:])

}
```
- `[3]bool{true, true, false}` array literal 
- `[]bool{true, true, false}` creates array, then builds a slice that references it:
- slice的`length`是slice的元素数
- slice的`capacity`是从slice第一个元素开始数的背后的数组的元素数
``` go
import "fmt"

func main() {
    s := []int{2, 3, 5, 7, 11, 13}
    printSlice(s)

    // Slice the slice to give it zero length.
    s = s[:0]
    printSlice(s)

    // Extend its length.
    s = s[:4]
    printSlice(s)

    // Drop its first two values.
    s = s[2:]
    printSlice(s)
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```
```
len=6 cap=6 [2 3 5 7 11 13]
len=0 cap=6 []
len=4 cap=6 [2 3 5 7]
len=2 cap=4 [5 7]
```
- slice的零值是`nil`， `length`和`capacity`都是0
```
var s []int
fmt.Println(s, len(s), cap(s))
if s == nil {
    fmt.Println("nil!")
}
```
- `make`可以创建零值数组，并返回slice，一般用来创建可变数组
``` go
a := make([]int, 5)  // len(a)=5
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```
``` go
package main

import (
    "fmt"
    "strings"
)

func main() {
    // Create a tic-tac-toe board.
    board := [][]string{
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
        []string{"_", "_", "_"},
    }

    // The players take turns.
    board[0][0] = "X"
    board[2][2] = "O"
    board[1][2] = "X"
    board[1][0] = "O"
    board[0][2] = "X"

    for i := 0; i < len(board); i++ {
        fmt.Printf("%s\n", strings.Join(board[i], " "))
    }
}

```
-  `append` 的结果是一个包含原 slice 所有元素加上新添加的元素的 slice。 
- 如果 s 的底层数组太小，而不能容纳所有值时，会分配一个更大的数组。 返回的 slice 会指向这个新分配的数组。 
``` go
package main

import "fmt"

func main() {
    var s []int
    printSlice(s)

    // append works on nil slices.
    s = append(s, 0)
    printSlice(s)

    // The slice grows as needed.
    s = append(s, 1)
    printSlice(s)

    // We can add more than one element at a time.
    s = append(s, 2, 3, 4)
    printSlice(s)
}

func printSlice(s []int) {
    fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

```
len=0 cap=0 []
len=1 cap=1 [0]
len=2 cap=2 [0 1]
len=5 cap=6 [0 1 2 3 4]
```

#### Range
- for 循环的 range 格式可以对 slice 或者 map 进行迭代循环。 
``` go
for i, v := range pow {
    fmt.Printf("2**%d = %d\n", i, v)
}
```

- 忽略序号和值

``` go
for i, _ := range pow
for _, value := range pow
for i := range pow
```


#### maps
- k v pair
- 零值是nil，nil 的 map 是空的，并且不能赋值。 
- 必须用 make 而不是 new 来创建
``` go
package main

import "fmt"

type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}

```
``` go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}

```

-  如果顶级的类型只有类型名的话，可以在文法的元素中省略键名。 
``` go
var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```
- 修改 map
    - 插入或修改 `m[key] = elem`
    - 读取 `elem = m[key]`
    - 删除 `delete(m, key)`
    - 双赋值检测某个键存在 `elem, ok = m[key]`

#### Function values
函数也是值
高阶函数(high order function)
``` go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}

```
函数的闭包
``` go
package main

import "fmt"

func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

## 3. Method and Interface
#### Method
- Go 没有类。然而，仍然可以在结构体类型上定义方法。 
``` go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```
- a method is just a function with a receiver argument. 
``` go

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```
- 也可以non-struct定义方法，类型定义和method要在一个包
``` go
func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}

func main() {
	f := MyFloat(-math.Sqrt2)
	fmt.Println(f.Abs())
}
```
- 指针receiver
    - 因为方法一般需要修改receiver，所以指针receiver更常见
    - 或者为了效率
- 一般来说，一个type上所有的methods，要么都是value型，要么都是pointer型receiver，很少混用。

``` go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}

```

``` go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK

var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK 《==  v.Scale(5) as (&v).Scale(5) 
```

``` go
var v Vertex
fmt.Println(AbsFunc(v))  // OK
fmt.Println(AbsFunc(&v)) // Compile error!

var v Vertex
fmt.Println(v.Abs()) // OK
p := &v
fmt.Println(p.Abs()) // OK 《== p.Abs() = (*p).Abs()
``` 

#### Interface
- 接口类型是由一组方法定义的集合。
- 类型通过实现那些方法来实现接口。 没有显式声明的必要；所以也就没有关键字“implements“。 
    - 列子代码的 22 行存在一个错误。 由于 Abs 只定义在 *Vertex（指针类型） 上， 所以 Vertex（值类型） 不满足 `Abser`。
    ``` go
    package main

    import (
        "fmt"
        "math"
    )

    type Abser interface {
        Abs() float64
    }

    func main() {
        var a Abser
        f := MyFloat(-math.Sqrt2)
        v := Vertex{3, 4}

        a = f  // a MyFloat 实现了 Abser
        a = &v // a *Vertex 实现了 Abser

        // 下面一行，v 是一个 Vertex（而不是 *Vertex）
        // 所以没有实现 Abser。
        a = v

        fmt.Println(a.Abs())
    }

    type MyFloat float64

    func (f MyFloat) Abs() float64 {
        if f < 0 {
            return float64(-f)
        }
        return float64(f)
    }

    type Vertex struct {
        X, Y float64
    }

    func (v *Vertex) Abs() float64 {
        return math.Sqrt(v.X*v.X + v.Y*v.Y)
    }

    ```

接口值
-  接口也是值。它们可以像其它值一样传递。
-  在内部，接口值可以看做包含值和具体类型的元组： `(value, type)`

底层值为 nil 的接口值
-  即便接口内的具体值为 nil，方法仍然会被 nil 接收者调用
    - 并不会像其他语言一样触发NPE
    - 保存了 nil 具体值的接口其自身并不为 nil。 
    ``` go
    package main

    import "fmt"

    type I interface {
        M()
    }

    type T struct {
        S string
    }

    func (t *T) M() {
        if t == nil {
            fmt.Println("<nil>")
            return
        }
        fmt.Println(t.S)
    }

    func main() {
        var i I

        var t *T
        i = t
        describe(i)
        i.M()

        i = &T{"hello"}
        describe(i)
        i.M()
        
        var s T
        i = &s
            describe(nil)

        i.M()
    }

    func describe(i I) {
        fmt.Printf("(%v, %T)\n", i, i)
    }

    ```
- Nil 接口值
    -  nil 接口值既不保存值也不保存具体类型。 
    -  为 nil 接口调用方法会产生运行时错误
- 空接口
    - 指定了零个方法的接口值被称为 *空接口：* 
    - 空接口可保存任何类型的值，被用来处理未知类型的值
    ``` go
    package main

    import "fmt"

    func main() {
        var i interface{}
        describe(i)

        i = 42
        describe(i)

        i = "hello"
        describe(i)
    }

    func describe(i interface{}) {
        fmt.Printf("(%v, %T)\n", i, i)
    }
    ```
- 类型断言
    -  类型断言 提供了访问接口值底层具体值的方式。 
    - `t := i.(T)` 该语句断言接口值 i 保存了具体类型 T，并将其底层类型为 T 的值赋予变量 t。 
    - 语法类似map的检测
    ``` go
    package main

    import "fmt"

    func main() {
        var i interface{} = "hello"

        s := i.(string)
        fmt.Println(s)

        s, ok := i.(string)
        fmt.Println(s, ok)

        f, ok := i.(float64)
        fmt.Println(f, ok)

        f = i.(float64) // panic
        fmt.Println(f)
    }

    ```
- 类型选择
    ```
    package main

    import "fmt"

    func do(i interface{}) {
        switch v := i.(type) {
        case int:
            fmt.Printf("Twice %v is %v\n", v, v*2)
        case string:
            fmt.Printf("%q is %v bytes long\n", v, len(v))
        default:
            fmt.Printf("I don't know about type %T!\n", v)
        }
    }

    func main() {
        do(21)
        do("hello")
        do(true)
    }

    ```
### Stringer
Stringer 是最普遍的接口之一，有点像java toString
- fmt 包（还有很多包）都通过此接口来打印值。 

``` go
package main

import "fmt"

type Person struct {
	Name string
	Age  int
}

func (p Person) String() string {
	return fmt.Sprintf("%v (%v years)", p.Name, p.Age)
}

func main() {
	a := Person{"Arthur Dent", 42}
	z := Person{"Zaphod Beeblebrox", 9001}
	fmt.Println(a, z)
}

```

### Error
- 与 fmt.Stringer 类似，error 类型是一个内建接口： 
- 与 fmt.Stringer 类似，fmt 包在打印值时也查找error接口
- 函数可以返回error
    -  error 为 nil 时表示成功；
    - 非 nil 的 error 表示失败。 
``` go
package main

import (
	"fmt"
	"time"
)

type MyError struct {
	When time.Time
	What string
}

func (e *MyError) Error() string {
	return fmt.Sprintf("at %v, %s",
		e.When, e.What)
}

func run() error {
	return &MyError{
		time.Now(),
		"it didn't work",
	}
}

func main() {
	if err := run(); err != nil {
		fmt.Println(err)
	}
}

```

### Reader
 io 包指定了 io.Reader 接口，它表示从数据流的读取端。 

``` go
package main

import (
	"fmt"
	"io"
	"strings"
)

func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

### 图像
 image 包定义了 Image 接口： 
``` go
 package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```
``` go
package main

import (
	"fmt"
	"image"
)

func main() {
	m := image.NewRGBA(image.Rect(0, 0, 100, 100))
	fmt.Println(m.Bounds())
	fmt.Println(m.At(0, 0).RGBA())
}

```

## 4. 并发
### Goroutines
-  Go 程（goroutine）是由 Go 运行时管理的轻量级线程。 
    - `go f(x, y, z)`
- Go 程在相同的地址空间中运行，因此在访问共享的内存时必须进行同步。
    - `sync` 包提供了这种能力，不过在 Go 中并不经常用到，因为还有其它的办法

### Channels & Buffered Channels
- 信道是带有类型的管道，你可以通过它用信道操作符 `<-` 来发送或者接收值。 
- 使用前必须创建 `ch := make(chan int)` 
- 发送和接收操作在另一端准备好之前都会阻塞。
    - 这使得 Go 程可以在没有显式的锁或竞态变量的情况下进行同步。 
- 信道可以是 带缓冲的。将缓冲长度作为第二个参数提供给 make 来初始化一个带缓冲的信道 
    - 我的理解就是想java的blocking queue
    - `ch := make(chan int, 100)` 


``` go
package main

import "fmt"

func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // 将和送入 c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // 从 c 中接收

	fmt.Println(x, y, x+y)
}

```

- 发送者可通过 close 关闭一个信道来表示没有需要发送的值了。
- 接收者可以通过为接收表达式分配第二个参数来测试信道是否被关闭
    - v, ok := <-ch
-  循环 for i := range c 会不断从信道接收值，直到它被关闭。 
- 只有发送者才能关闭信道，而接收者不能。
- 信道与文件不同，通常情况下无需关闭它们。
    - 只有在必须告诉接收者不再有需要发送的值时才有必要关闭，例如终止一个 range 循环。 
``` go
package main

import (
	"fmt"
)

func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```

### select 语句
- select 语句使一个 Go 程可以等待多个通信操作。 
    - select 会阻塞到某个分支可以继续执行为止，这时就会执行该分支。当多个分支都准备好时会随机选择一个执行。
    ``` go
    package main

    import "fmt"

    func fibonacci(c, quit chan int) {
        x, y := 0, 1
        for {
            select {
            case c <- x:
                x, y = y, x+y
            case <-quit:
                fmt.Println("quit")
                return
            }
        }
    }

    func main() {
        c := make(chan int)
        quit := make(chan int)
        go func() {
            for i := 0; i < 10; i++ {
                fmt.Println(<-c)
            }
            quit <- 0
        }()
        fibonacci(c, quit)
    }

    ```
 
    -  当 select 中的其它分支都没有准备好时，default 分支就会执行。
        - 为了在尝试发送或者接收时不发生阻塞，可使用 default 分支：  
    ``` go
    package main

    import (
        "fmt"
        "time"
    )

    func main() {
        tick := time.Tick(100 * time.Millisecond)
        boom := time.After(500 * time.Millisecond)
        for {
            select {
            case <-tick:
                fmt.Println("tick.")
            case <-boom:
                fmt.Println("BOOM!")
                return
            default:
                fmt.Println("    .")
                time.Sleep(50 * time.Millisecond)
            }
        }
    }

    ```

### sync.Mutex
-  Go 标准库中提供了 sync.Mutex 互斥锁类型及其两个方法：
    Lock
    Unlock
-  我们也可以用 defer 语句来保证互斥锁一定会被解锁。参见 Value 方法。 
``` go
package main

import (
	"fmt"
	"sync"
	"time"
)

// SafeCounter 的并发使用是安全的。
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}

// Inc 增加给定 key 的计数器的值。
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	c.v[key]++
	c.mux.Unlock()
}

// Value 返回给定 key 的计数器的当前值。
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock 之后同一时刻只有一个 goroutine 能访问 c.v
	defer c.mux.Unlock()
	return c.v[key]
}

```