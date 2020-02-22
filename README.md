# go note

#### 匿名函数
匿名函数：顾名思义就是没有名字的函数。匿名函数最大的用途是来模拟块级作用域,避免数据污染的。
###### 实例:
1.
```go
package main

import (
   "fmt"
)
func main() {
   f:=func(){
      fmt.Println("hello world")
   }
   f()//hello world
   fmt.Printf("%T\n", f) //打印 func()
}
```
2.带参数
```go
package main

import (
   "fmt"
)
func main() {
   f:=func(args string){
      fmt.Println(args)
   }
   f("hello world")//hello world
   //或
   (func(args string){
        fmt.Println(args)
    })("hello world")//hello world
    //或
    func(args string) {
        fmt.Println(args)
```
3.带返回值

```go
package main

import "fmt"

func main() {
   f:=func()string{
      return "hello world"
   }
   a:=f()
   fmt.Println(a)//hello world
}
```
4.多个匿名函数

```go
package main

import "fmt"

func main() {
   f1,f2:=F(1,2)
   fmt.Println(f1(4))//6
   fmt.Println(f2())//6
}
func F(x, y int)(func(int)int,func()int) {
   f1 := func(z int) int {
      return (x + y) * z / 2
   }

   f2 := func() int {
      return 2 * (x + y)
   }
   return f1,f2
}
```
#### 闭包
闭包：闭包是由函数和与其相关的引用环境组合而成的实体。

```go
func incr() func() int {
	var x int
	return func() int {
		x++
		return x
	}
}

```
调用这个函数会返回一个函数变量。

i := incr()：通过把这个函数变量赋值给 i，i 就成为了一个闭包。

所以 i 保存着对 x 的引用，可以想象 i 中有着一个指针指向 x 或 i 中有 x 的地址。

由于 i 有着指向 x 的指针，所以可以修改 x，且保持着状态：

```go
println(i()) // 1
println(i()) // 2
println(i()) // 3
```
也就是说，x 逃逸了，它的生命周期没有随着它的作用域结束而结束。

但是这段代码却不会递增：

```go
println(incr()()) // 1
println(incr()()) // 1
println(incr()()) // 1
```
这是因为这里调用了三次incr()，返回了三个闭包，这三个闭包引用着三个不同的 x，它们的状态是各自独立的。
######示例
1.
```go
package main

import "fmt"

func main() {
    a := Fun()
    b:=a("hello ")
    c:=a("hello ")
    fmt.Println(b)//worldhello
    fmt.Println(c)//worldhello hello 
}
func Fun() func(string) string {
    a := "world"
    return func(args string) string {
        a += args
        return  a
    }
}
```
2.
```go
package main

import "fmt"

func main() {
   a := Fun()
   d := Fun()
   b:=a("hello ")
   c:=a("hello ")
   e:=d("hello ")
   f:=d("hello ")
   fmt.Println(b)//worldhellod
   fmt.Println(c)//worldhello hello
   fmt.Println(e)//worldhello
   fmt.Println(f)//worldhello hello
}
func Fun() func(string) string {
   a := "world"
   return func(args string) string {
      a += args
      return  a
   }
}
```
注意两次调用F()，维护的不是同一个a变量。
3.
```go
package main

import "fmt"

func main() {
   a := F()
   a[0]()//0xc00004c080 3
   a[1]()//0xc00004c080 3
   a[2]()//0xc00004c080 3
}
func F() []func() {
   b := make([]func(), 3, 3)
   for i := 0; i < 3; i++ {
      b[i] = func() {
         fmt.Println(&i,i)
      }
   }
}
```
闭包通过引用的方式使用外部函数的变量。例中只调用了一次函数F,构成一个闭包，i 在外部函数B中定义，所以闭包维护该变量 i ，a[0]、a[1]、a[2]中的 i 都是闭包中 i 的引用。因此执行,i 的值已经变为3，故再调用a[0]()时的输出是3而不是0。

4.如何避免上面的BUG ，用下面的方法，注意和上面示例对比。

```go
package main

import "fmt"

func main() {
    a := F()
    a[0]() //0xc00000a0a8 0
    a[1]() //0xc00000a0c0 1
    a[2]() //0xc00000a0c8 2
}
func F() []func() {
    b := make([]func(), 3, 3)
    for i := 0; i < 3; i++ {
        b[i] = (func(j int) func() {
            return func() {
                fmt.Println(&j, j)
            }
        })(i)
    }
    return b
}
或者
package main

import "fmt"

func main() {
    a := F()
    a[0]() //0xc00004c080 0
    a[1]() //0xc00004c088 1
    a[2]() //0xc00004c090 2
}
func F() []func() {
    b := make([]func(), 3, 3)
    for i := 0; i < 3; i++ {
        j := i
        b[i] = func() {
            fmt.Println(&j, j)
        }
    }
    return b
}
```
每次 操作仅将匿名函数放入到数组中，但并未执行，并且引用的变量都是 i，随着 i 的改变匿名函数中的 i 也在改变，所以当执行这些函数时，他们读取的都是环境变量 i 最后一次的值。解决的方法就是每次复制变量 i 然后传到匿名函数中，让闭包的环境变量不相同。
5.

```go
package main

import "fmt"

func main() {
   fmt.Println(F())//2
}
func F() (r int) {
   defer func() {
      r++
   }()
   return 1
}
```
输出结果为2，即先执行r=1 ,再执行r++。

6.递归函数

```go
package main

import "fmt"

func F(i int) int {
   if i <= 1 {
      return 1
   }
   return i * F(i-1)
}

func main() {
   var i int = 3
   fmt.Println(i, F(i))// 3 6
}
```
7.斐波那契数列(Fibonacci)

```go
package main

import "fmt"

func fibonaci(i int) int {
    if i == 0 {
        return 0
    }
    if i == 1 {
        return 1
    }
    return fibonaci(i-1) + fibonaci(i-2)
}

func main() {
    var i int
    for i = 0; i < 10; i++ {
        fmt.Printf("%d\n", fibonaci(i))
    }
}
```
