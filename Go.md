##Go语言

### 第一章 类型 ###

**1.1 变量**

- 定义变量使用关键词var，自动化初始化为0，提供初始值，可省略类型，系统会自行推断

			var x , y, z int  //这里可同时一次定义多个变量
			s="abc"

- 函数内部，可使用“：=”定义变量

		func main(){
			x:=12
			y,z := 123, "235"  //这里可同时一次定义多个变量
		}
- 还可以通过内建(built-in)函数new来创建变量 

		p := new(int)   // p,类型*int,指向一个没有命名的int变量
		fmt.Println(*p) // "0"
		*p = 2          
		fmt.Println(*p) // "2"

- 特殊只写变量"_" ,可用于忽略值占位

- 对于定义的没有使用的变量，编译器会报错

		func main(){
			i:=0  //erro,可使用"_=i"规避
		}

- 注意的是短声明左边的变量未必都是新声明的

		s := "avc"
		fmt.Println(&s)
	
		s, y := "hello", 23
		fmt.Println(&s, y)
		{
			s, z := 100, 234
			fmt.Println(&s, z)
		}

	输出：

	0xc042044210

	0xc042044210 23

	0xc04204e140 234

- 可见性（在Go语言中，变量、函数等的导出只取决于一个因素：名字首字母的大小写。 ）

	1. 声明在函数内部，是函数的本地值，类似private
	
	1. 声明在函数外部，是对当前包可见(包内所有.go文件都可见)的全局值，类似protect
	 
	1. 声明在函数外部且首字母大写是所有包可见的全局值,类似public  

- 包内可见的变量在 main 函数开始执行之前初始化，本地变量在函数执行到对应的声明语句时初始化。 

- 建议采用驼峰命名法

- 如果需要为本地变量显式的指定类型，或者先声明一个变量后面再赋值，那么应该使用var

- 可以利用并行赋值的特性来进行值交换: 

		i, j = j, i // swap values of i and j 

- 变量的声明周期

	1. 包内可见的变量的生命期是固定的 
	
	1. 	本地变量的生命期是动态的 ，每次声明语句执行时，都会创建一个新的变量实例，生命期结束后变量可能会被回收。

	1. 	函数的参数和本地变量都是动态生命期，在每次函数调用和执行的时候，这些变量会被创建。  

- 短声明变量的作用域是要特别注意的

		var cwd string
		
		func init() {
		    cwd, err := os.Getwd() // compile error: unused: cwd
		    if err != nil {
		        log.Fatalf("os.Getwd failed: %v", err)
		    }
		}
	这里cwd和err在init的词法块中都没有声明过，因此 := 语句会将它们两声明为本地变量。init内部的cwd声明会隐藏外部的,因此这个程序没有达到更新包级变量cwd的目的。有一些办法可以处理这种潜在的错误，最直接的就是避免使用:=，通过var来声明err变量： 



**1.2 常量**

- 常量值必须是可编译可确定的数字、字符串、布尔值

		const x,y int=1,2  //多常量初始化
		const z="hello" //自动推断
		const(
			a,b=12,3
			c bool =false
		)
		
		func main(){
			const s="world"
		}	//未使用局部常量不会引发编译错误

- 常量组中，不提供类型和初始值，视为和上一个常量相同

		const(
		  s="123"
		  x
		  z=len(s) //可以是len、cap、unsafe.Sizeof等编译结果
		)
- 枚举

	关键字iota 定义常量组从0开始计数

		const (
				a1 = iota //0
				a2        //1
				a3        //2
				a4 = "ha" //独立值，iota += 1
				a5        //"ha"   iota += 1
				a6 = 100  //iota +=1
				a7        //100  iota +=1
				a8 = iota //7,恢复计数,此处由于上面被打断，所以要手动显示恢复
				a9        //8
				_
				a10  //10
			)
	
	同一个常量组中，可提供多个iota，它们各自增长

		const(
			A,B= iota ,iota //0 ,0
			C,D 			//1,1
		)

	可通过自定义类型实现枚举类型的限制

		type Color int
		
		const (
			Black Color = iota
			Red
		)
		
		func test(c Color) {
			fmt.Println(c)
		}
	
		func main(){
			test(1) //1,常量会自动转换
		}

**1.3 基本类型**

![](https://i.imgur.com/A1zFNit.png)

空指针为nil

**1.4 引用类型**

- 引用类型包括slice、map、channel

- 内置函数new 计算类型大小、分配零值内存，返回指针；make被编译器成具体创建函数，分配内
存和初始化成员结构，返回对象

		a:=[]int
		a[1]=10
	
		b:=make([]int,3) //make slice
		b[1]=10

**1.5 类型转换**

- 显示转换

		var b byte=100
		var n int=int(b)

- 使用括号避免优先级错误

		 *Point(p) //相当于 *(Ponit(p))
		（*Ponit)(p)
	
- 不能将int类型当bool使用，即1不等于true，反之同

**1.6 字符串**

- 字符串是不可变值类型，无法修改字节数组,因此就地修改字符串是不允许的： 

		s[0] = 'L' // compile error: cannot assign to s[0]

- 不能用序号获取字节元素指针，&s[i]非法

- 支持跨行

		s:=`a
		c`
		fmt.Println(s)

	输出

		a
		c

- 连接跨行字符串，"+" 必须在上一行末尾

- 支持索引返回子串

		s:="Hello World!"
		s1:=s[:5]  //Hello
		s2:=s[7:]	//World!	
		s3:=s[1:5]  //ello

- 要修改字符串，要先将其转为[] rune 或者 []byte,完成后再转为string,无论哪种转换都会重新分配内存，复制字节数组(rune是int32的另一种表示法)

		func main(){
			s:="abc"
			bs:=[]byte(s)

			bs[1] ="B"
			fmt.Println(string(bs)) //aBc
		}

- 内置的len函数会返回字符串的所有字节(byte)数 

**1.7指针**

- 支持指针类型`*T`，指针的指针 `**T`

- 支持用`.`访问目标成员

		func main(){
			type data struct{a int}
			var d =data{123}
			var p *data
			p=&d
			fmt.Printf("%p,&v\n",p,p.a)
		}

	输出

		0x2101ef018,123

- 返回局部变量指针是安全的

		func test() *int{
			x:=199
			return &x
		}

- 指针值是一个变量的存储地址。注意：不是所有的值都有地址，但是变量肯定是有地址的！这个概念一定要搞清楚！ 通过指针，我们可以间接的去访问一个变量，甚至不需要知道变量名。

		x := 1
		p := &x         // p类型:*int,指向x
		fmt.Println(*p) // "1"
		*p = 2          // 等价于x = 2
		fmt.Println(x)  // "2"

- 聚合类型struct或者array中的元素也是变量，因此是可以通过寻址(&)获取指针的。 

- 指针的零值是nil ，指针是可以比较的，两个指针相等意味着两个指针都指向同一个变量或者两个指针都为nil。 （go的所有类型在没有初始值时都默认会初始化为该类型的零值 ）

- 每次调用f都会返回不同的指针，因为f会创建新的本地变量并返回指针: 

		var p = f()
		fmt.Println(*p) //output:1
		func f() *int {
		    v := 1
		    return &v
		}
		fmt.Println(f() == f()) // "false"

- 把变量的指针传递给函数，即可以在函数内部修改该变量 
		
		func incr(p *int) int {
		    	*p++ // increments what p points to; does not change p
		    	return *p
		}
		
		v := 1
		incr(&v)              // v现在是2
		fmt.Println(incr(&v)) // "3" (and v is 3)

**1.9 Package**

- 每个.go文件的第一行都是package tempconv的声明，表示该文件属于哪个包。当包被导入后，可以这样调用它的成员：tempconv.CToF等等 

		//温度转换
		package tempconv
		
		// 将Celsius温度转换为Fahrenheit
		func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }
		
		// 将Fahrenheit温度转换为Celsius.
		func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }

- 包导入

	在import声明中，每个包都有自己的导入包名，按照惯例，这个包名是导入路径的最后一个词，例如github.com/sunface/tempconv的导入包名是temconv。因此可以直接调用tempconv.CToF。还可以使用别名机制避免包名冲突： 

		import (
			"github.com/sunface/tempconv"
			temp "github.com/sunfei/tempconv"
		) 
	如果导入一个包后不去使用，那么就会报编译错误，编译器的这个检查可以帮助消除不需要的包引用， 

- 包的初始化

	init函数都会在程序刚启动的时候自动运行，文件内每个init函数会按照声明的顺序依次执行。 
		
		func init() { /* ... */ }

### 第二章 表达式 ###

**2.1 保留字**

![](https://i.imgur.com/ud5oUV7.png)

**2.1 运算符**

- 不支持运算符重载

		n:=0
		p:=n++//erro

**2.2 初始化**

- 初始化复合对象，必须使用类型标签，且左大括号必须在类型尾部

		var a=struct{ x int}{100}
		var b=[]int{1,3,4}
		var c=[]int{
			1,
			2,  //ok
		}

		var d=[]int{
			3,
			4}  //ok

		var e=[]int{
			5,
			6  //erro,可以分多行，但最后一行必须以","或者"}"结尾
		}

**2.3 if**

- 可省略条件表达式括号

- 支持初始化语句，可定义代码块局部变量

- 代码块左大括号必须在表达式尾部

		x:=0
		if x > 10
		{
		}  //Error

		if n:="avc";x > 0{
			...
		}else if x < 0{
			...
		}

	**注意：不支持三目操作符**

- 第一个if语句里声明的变量对第二个if语句是可见的 

**2.4 For**

- 支持以下循环方式

		s:="abc"

		for i,n:=0,len(s)l; i < n; i++{
			fmt.Println(s[i])
		}

		n:=len(s)
		for n > 0{
			fmt.Println(s[n])
			n--
		} 

		for{   //替代 while (true){}
			fmt.Println(s)  //for(;;){}
		}//结果不停输出

		
		for _,c:=range s{  //忽略index ,第二个是值
			fmt.Println(c)
		}

- range会复制对象，故建议使用引用类型（slice,map,channel），不要用数组

**2.5 Switch**

- 基本使用

	x:=[]int{1,2,3}
	i:=2
	switch i{
		case x[1]:
			fmt.Println("a")
		case 1,3:
			fmt.Println("b")
		default:
			fmt.Println("c")
	}
输出
	
	a

- 如果要继续下个分支，可使用fallthrough,但不再判断条件

	x:=10
	switch x{
		case 10:
			fmt.Println("a")
			fallthrough
		case 0：
   			fmt.Println("b")
	}

输出

	a
	
	b

- 如果省略条件，可以当if..else用

- 可带初始化语句

	switch i:=x[2];{
		case i >0:
			...
		case i<0
			...
		default:
			...
	}

**2.6 Goto,Break,Continue**

- 支持在函数内goto跳转。标签名区分大小写，未使用标签引发错误
		
		var  int
		for {
			fmt.Println(i)
			i++
			if i > 2 {
				goto BREAK
			}
		}
		 BREAK:
			fmt.Println("break")

- continue仅能用于for循环,break可用于for、switch、select

### 第三章 函数 ###

**3.1 函数定义**

- func定义，左大括号依旧不能另起一行
	
		func test(x,y int,s string)(int,string){  //类型相同的相邻参数可合并，多返回值必须用括号

			...
		} 

- 函数可作为参数传递，建议将复杂签名定义为函数类型

	 	func test(fn func() int) int{ 
			return fn()
		}
	
		type FormatFunc func(s string,x,y int) string  //定义函数类型
	
		func format(fn FormatFunc, s string ,x,y int) string{
			return fn(s,x,y)
		}
	
		func main(){
			s1:=test(func() int{return 100})  //直接将匿名函数当参数
			s2:= format(func(s string, x,y int) string{
	    		return fmt.Println(s,x,y)
			},"%d,%d",10,20)
	 		fmt.Println(s1,s2)
		}

- 进行函数调用时，传递给函数参数的值实际上是变量的拷贝，所以函数参数接收的是一个副本，并不是原始的变量，这种就是值传递，Go语言的函数调用默认就是值传递。这种机制在传递较大的数组值时，效率是很低的(数组中的所有元素都会重新拷贝一次)，并且对数组的修改实际上是修改拷贝值，而不是原始数组。Go语言的这种机制和其它很多语言是不同的，其它编程语言在函数调用时，可能会隐式地将数组作为引用或者指针进行参数传递(C语言)。当然，我们可以显示传递一个数组指针，这样函数对数组的修改就会直接修改原始数组。下面的函数将[32]byte类型的数组进行清零： 

		func zero(ptr *[32]byte) {
		    *ptr = [32]byte{}
		}

**3.2 不定参数**

	func main(){
	    fmt.Println(test("sum: %d", 1, 2, 3))  //sum:6
		t := []int{1, 2, 3}
		fmt.Println(test("sum: %d", t...))//sum:6
	}
	func test(s string, n ...int) string {
	
		var x int
		for _, i := range n {
			x += i
		}
		return fmt.Sprintf(s, x)
	}

**3.3 返回值**

- 多返回值可以直接作为其他函数调用实参

		func main(){
			fmt.Println(add(example())) //3
		}
		
		func example() (int, int) {
			return 1, 2
		}
		func add(x, y int) int {
			return x + y
		}
- 裸返回

		func add(x,y int) (z int){
			z=x+y
			return
		}
	
		func main(){
	      fmt.Println(add(1,2))
		}

	**注意如果局部变量和返回变量同名，这时需要显式返回**
	
		func add(x,y int) (z int){
			var z=x+y
			return //error
			return z
		}

- 命名返回参数允许defer延迟调用通过闭包读取和修改

		func add(x, y int) (z int) {
			defer func(){   //return 返回前，会先修改命名返回参数
			   z+=100
			}()
			z =x + y
			return 
		}

		func main(){
   			fmt.Println(add(1,2))  //输出：103
		}

**3.4 延迟调用**

- 延迟调用一般用于释放资源或者错误处理

		func test() error{
			f,err:=os.Create("test.txt")
			if err!=nil{ return err}
			
			defer f.Close()
			
			f.WriteString("Hello,World!")
			return nil
		}

- 多个defer注册，延迟调用发生错误，其他调用依旧会执行
	
		func test(x int){
			defer fmt.Println("a")
			defer fmt.Println("b")
	
			defer func(){
				fmt.Println(100 / x)
			}()
	
			defer fmt.Println("c")
		}
	
		func main(){
			test(0)
		}

	输出：

		c
		b
		a
		panic:...



###第四章 数据

**4.1 Array**

- 指针数组[n]*T,数组指针*[n]T

- 复合语句初始化

		d：=[...]struct{
			name string
			age uint8
		}{
			{"user1",10},
			{"user2",20},   //**别忘了最后一行的逗号**
		}

- 支持多维数组

- 值拷贝行为会造成性能问题，建议使用slice,或者数组指针

- 内置函数len和cap都能返回数组的长度（元素数量）

**4.2 slice**

- slice不是数组或者数组指针，它是通过内部指针和相关属性引用数组片段，以实现变长方案。slice之间不能通过==操作符比较，标准库中提供了高度优化的bytes.Equal函数，可以用来判断两个[]byte是否相等，不过对于其它类型的slice,我们必须实现自己的比较函数： 

		func equal(x, y []string) bool {
		    if len(x) != len(y) {
		        return false
		    }
		    for i := range x {
		        if x[i] != y[i] {
		            return false
		        }
		    }
		    return true
		}
唯一例外：slice可以和nil进行比较 
- 测试一个slice是否为空，要使用len(s) == 0 

- 引用类型

- 初始化

		data:=[...]int{1,2,3,4,5}
		slice:=data[1:4:5]  //[low:high:max]

	![](https://i.imgur.com/jiWnwPg.png)

- 使用make创建

		s1:=make([]int,6,8) //指定len和cap,如果省略cap，相当于len=cap

- 获取底层数组元素指针

		s1:=[]int{1,3,4,56}
		p:=&s1[2]
		*p+=100
		fmt.Println(s)

		输出：
		[1,3,104,56]

- [][]T,是指元素类型为[]T

	data:=[][]int{
		[]int{1,2,3}
		[]int{100,5}
		[]int{11,333,44}
	}

- append，向slice尾部添加数据，返回新的Slice对象

		data:=[...]int{0,1,2,3,4,5}
		s:=data[:3]
		s2：=append(s,100,200)  //支持同时添加多个数据
	
		fmt.Println(data)  //[0,1,2,3,4,5]
		fmt.Println(s)  //[0,1,2]
		fmt.Println(s2)  //[0,1,2,100,200]

- 若超出Slice.cap，就会重新分配底层数组，即使原数组并未填满
		
		data:=[...]int{0,1,2,5:0}   //[0,1,2,0,0,0]
		s:=data[:2:3]
		
		s=append(s,100,200)  //超出范围
		
		fmt.Println(s,data)
		fmt.Println(&s[0] ,&data[0])  //比对起始针地址

		输出：
		[0,1,2] [0,1,2,0,0,0]
		0xc042008300 0xc042008330

- 通常以2倍容量重新分配底层数组，批量添加数据时，建议一次性分配足够大的空间，以减少内存分配和数据复制开销。及时释放不再使用的对象，避免持有过期数组，造成GC无法回收

- copy在两个slice间复制数据，复制长度以len小的为准,允许区间重叠

		data:=[...]int{0,1,2,3,4,5,6,7,8,9}
	
		s:=data[8:]
		s2:=data[:5]
	
		copy(s2,2)
	
		fmt.Println(s2)  //[8 9 2 3 4]
		fmt.Println(data) //[8 9 2 3 4 5 6 7 8 9]

- 要删除slice某个元素i并保存原有的元素顺序，可以通过copy将i后面的元素依次向前移动一位：

		func remove(slice []int, i int) []int {
		    copy(slice[i:], slice[i+1:])
		    return slice[:len(slice)-1]
		}
		
		func main() {
		    s := []int{5, 6, 7, 8, 9}
		    fmt.Println(remove(s, 2)) // "[5 6 8 9]"
		} 

**4.3 Map**

- 键必须是能比较的类型，值可以是任意类型

- 预先给make函数一个合理元素数量参数，有助于提升性能，事先申请一大块内存，可避免后续频繁扩张

		m:=make(map[int]string,1000)

- 以下是常见操作

		m:=map[string]int{   
			"a":1
		}

		m["a"]++  //x += y和 x++ 等短赋值语法也可以用在map上： 
		
		ages:=make(map[int]int)

		if v,ok:=m["a"]; ok{   //判断key 是否存在
         	fmt.Println(v)
		}
		

		fmt.Println(m["c"])   //对于不存在的key，直接返回\0，不会报错

		m["b"]=2		//新增或者修改

		delete(m,"c")		//删除

		fmt.Println(len(m))    //获取键值对数量，cap无效

		for k,v :=range m{   //迭代，顺序随机
 			fmt.Println(k,v)
		}

- 从map中直接取值，取回的只是value的临时复制品，修改不了其成员

		type user struct{ name string}

		m:=map[int]user{
		  1:{"user1"},
		}

		m[1].name="Tom"  //error

		正确做法如下：

		u:=m[1]
		u.name="Tom"
		m[1]=u  //替换value

		或者
		
		m2：=map[int]*user{
			1:&user{"user1"}
		}

		m2[1].name="Tom"  //返回的是指针复制品，通过指针修改原对象是允许的

- map中的元素不是变量，因此不能寻址！！ 

**4.4 struct**

- 可以直接对成员赋值 

		type Employee struct {
		    ID        int
		    Name      string
		    Address   string
		    DoB       time.Time
		    Position  string
		    Salary    int
		    ManagerID int
		}
		
		var dilbert Employee

		dilbert.Salary -= 5000 // demoted, for writing too few lines of code

- 也可以对成员进行取址，然后通过指针访问 

		position := &dilbert.Position
		*position = "Senior " + *position // promoted, for outsourcing to Elbonia

- 点操作符还可以应用在struct指针上 

		var employeeOfTheMonth *Employee = &dilbert
		employeeOfTheMonth.Position += " (proactive team player)"



- 可用_补位，支持自身类型的指针成员

		type Node struct{
			_ int
			id int
			data *byte
			next *Node
		}

		func main(){
			n1:=Node{
		      id:1,
			  data：nil
			  next:&n1	
			}
		}

- 顺序初始化必须包含全部字段

		type User struct{
			name string
			age int
		}
		u1:=User{"Tom",20}
		u2:=User{"Tom"}  //Error

- 匿名字段,被匿名嵌入的可以是任何类型，当然也包括指针

		type User struct{
			name string
			age int
		}

		type Manager struct{
			User
			title string
		}

		m:=Manager{
			User:User{"Tom"}  //匿名字段的显式字段名，和类型名相同
			title:"aaaa"
		}

		m.age=49  //可以像普通字段那样访问匿名字段成员
		
		注意：若出现外层同名字段时，显式字段名

		fmt.Println(m.User.age)

- 如果函数需要修改传入的struct参数，那么就必须使用指针 

		func AwardAnnualRaise(e *Employee) {
		    e.Salary = e.Salary * 105 / 100
		}

- 通常来说struct都是作为指针来使用,使用短声明的方法来初始化一个struct并获取它的地址： 

		pp := &Point{ 1 , 2 }
		//等价于下面
		pp := new(Point)
		*pp = Point{1, 2}



**4.5 Json**

- 将Go语言中的数据结构转为JSON叫做marshal,可以通过json.Marshal函数来完成：(Struct转JSON)

	考虑一个收集电影信息并提供电影推荐的应用程序，它声明了一个Movie类型，然后初始化了一个Movie列表：

		type Movie struct {
		    Title  string
		    Year   int  `json:"released"`
		    Color  bool `json:"color,omitempty"`
		    Actors []string
		}
		
		var movies = []Movie{
		    {Title: "Casablanca", Year: 1942, Color: false,
		        Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
		    {Title: "Cool Hand Luke", Year: 1967, Color: true,
		        Actors: []string{"Paul Newman"}},
		    {Title: "Bullitt", Year: 1968, Color: true,
		        Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
		    // ...
		}

		data, err := json.Marshal(movies)
		if err != nil {
		    log.Fatalf("JSON marshaling failed: %s", err)
		}
		fmt.Printf("%s\n", data)

	Marshal函数返回的JSON字符串是没有空白字符和缩进的，下面为了方便对JSON字符串进行折行显示： 

	[{"Title":"Casablanca","released":1942,"Actors":["Humphrey Bogart","Ingr
	id Bergman"]},{"Title":"Cool Hand Luke","released":1967,"color":true,"Ac
	tors":["Paul Newman"]},{"Title":"Bullitt","released":1968,"color":true,"
	Actors":["Steve McQueen","Jacqueline Bisset"]}]

- 如果需要为前端生成便于阅读的格式，可以调json.MarshaIndent，该函数有两个参数表示每一行的前缀和缩进方式： 

		data, err := json.MarshalIndent(movies, "", "    ")
		if err != nil {
		    log.Fatalf("JSON marshaling failed: %s", err)
		}
		fmt.Printf("%s\n", data)

	JSON的表示和Go语言有一个很大的区别，在最后一个成员后面没有逗号！！ 
		
		[
		    {
		        "Title": "Casablanca",
		        "released": 1942,
		        "Actors": [
		            "Humphrey Bogart",
		            "Ingrid Bergman"
		        ]
		    },
		    {
		        "Title": "Cool Hand Luke",
		        "released": 1967,
		        "color": true,
		        "Actors": [
		            "Paul Newman"
		        ]
		    },
		    {
		        "Title": "Bullitt",
		        "released": 1968,
		        "color": true,
		        "Actors": [
		            "Steve McQueen",
		            "Jacqueline Bisset"
		        ]
		    }
		]

	在JSON编码中，默认使用struct的字段名做为JSON的对象(通过reflect技术)，只有导出的字段才会被编码，这也是我们为什么使用大写字母开头的字段。 

	Year字段在编码后变为released,Color变为color，这个就是因为之前提到的Tag。一个struct字段的Tag是该字段的元数据： 

		Year  int  `json:"released"`
		Color bool `json:"color,omitempty"`  //omitempty选项，表示当Color为空或者零值时不生成JSON对象 

- JSON解析为Struct,JSON格式的电影数据解码为一个struct组成的slice,其中struct中只含有Title字段。通过定义合适的数据结构，我们可以选择性的解码JSON数据中需要的字段。当Unmarsha函数返回时，slice中包含的struct将只有Title字段，其它的JSON成员将被忽略。

		var titles []struct{ Title string }
		if err := json.Unmarshal(data, &titles); err != nil {
		    log.Fatalf("JSON unmarshaling failed: %s", err)
		}
		fmt.Println(titles) // "[{Casablanca} {Cool Hand Luke} {Bullitt}]"

		

		




		
		









	
	
 






- 







		











