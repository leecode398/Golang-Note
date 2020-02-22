# go note

#### 匿名函数
匿名函数：顾名思义就是没有名字的函数。匿名函数最大的用途是来模拟块级作用域,避免数据污染的。
###### 实例:
1.
```
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
```
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

```
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

```
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

###### 代码实例

```
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

