

byte和string

```go
一、string
1、string的单行表示为"",作为string的空返回值为""。多行用``
2、string的底层是byte数组，所以可以用切片的方式创造子字符串a = a[1:2],这个字符串可以安全的功用数据，子字符串生成的开销低廉，可以没有分配内存
3、字符串可以和字节slice进行转换，所以字符串可以通过slice来修改值，或者切片
s:="a"
b:=[]byte(s)
注意：字符串和byte转换如果字符串太大，就消耗的比较大，因为要复制内容

二、byte和rune
1、byte是uint8，rune是int32
byte只能处理ascii类型，rune能够处理一切字符串
  str := "hello 世界"
  str2 := []rune(str)


```





#### 相关api

```go
strings.go和bytes中都有下面的方法,当然byte中的参数和返回值是不一样的
1、func Contains(s,sep string)bool:判断s中是否包含sep
2、func Count(s,sep string)int：判断字符串sep在s中重叠的个数
3、func Fields(s string)[]string:以连续的空白字符为分隔符，将s切分成多个字串
4、func Index(s ,sep string)int：判断sep在s中下标
5、func Join(a []string,sep string)string :将a中的字串链接成单独的字符串，字串之间用sep分割

```



#### 要注意的坑

```go
一、首先说byte和string之间的坑
package main
import "fmt"
func main() {
s := []byte("")
s1 := append(s, 'a')
s2 := append(s, 'b')
//fmt.Println(s1, "==========", s2)
fmt.Println(string(s1), "==========", string(s2))
}
// 出现个让我理解不了的现象, 注释时候输出是 b ========== b
// 取消注释输出是 [97] ========== [98] a ========== b 

这个坑导致原因
1、这里是让空字符串转化为了byte，默认容量为32
2、如果cap的容量不是0，那么s1进行append的时候，进行逃逸分析，这个变量会一直在栈内存，不会发生逃逸，也就是说s1和s2进行了复制
3、如果空字符串转化了byte，但是发生了变量逃逸，那么，cap的容量就为0，为0的话，那么就会发生呢个slice的扩容，然后值就不一样了
byte和string进行转换注意：
1、变量是否发生逃逸
2、时刻注意cap，cap如果为0，那么slice就会扩容，如果不为0，那么就会产生复制

```

#### 内存布局相关，

```go
1、string的底层是byte数组
2、string的切片的字串和string指向同一个地址，节约开销
```

#### 还有可以用什么表示

待更新····



#### buffer相关

```go
缓冲器  byes.Buffer和strings.Builder
var b bytes.Buffer
b.Reset()   //重置
b.Writestring("")   //往缓冲器中写入数据
b.byte()  //b.string()  获取结果

var c strings.Builder
c.Reset()  //重置
c.Writestring("") //往缓冲器中写入数据
c.string()  //获取结果
```

## 类型转换

```go
类型转换是必须要掌握的
一、断言

t,ok := new(结构体).(接口名字)
二、类型转换
1、有名的类型转换
var f func()

func a(){
  
}

//转换
dd := f(a)
例子：
      type a func()

      type b struct{
          A a
      }

      func f(){
          fmt.Println("f")
      }

      func main(){
          c:=new(b)
          c.A = a(f)
          c.A()
      }
2、匿名的类型转换
type f func()

转化：
f(func(){})

例子：
      type I interface {
          call(interface{})
      }

      type f func()

      func (p f)call(d interface{}){
          fmt.Println("d",d)
        //可以调用自己，也可以不用
          p()
      }

      func main(){
        f(func(){fmt.Println("this is anonymous function")}).call(1)
      }
这种匿名函数的转换是实现了接口，必须要转换才可以调用接口的方法
如果要传参是下面的例子：
      type I interface {
          call(interface{})
      }
	
	  //需要这里传参
      type f func(interface{})

      func (p f)call(d interface{}){
          fmt.Println("d",d)
          p(d)
      }

      func main(){
          f(func(d interface{}){fmt.Println("this is anonymous function",d)}).call(1)
        /*也可以下面的方式实现
        a := f(func(d interface{}){fmt.Println("this is anonymous function",d)})
	a.call(1)
        
        */
      }

```



### 变量和常量语法，应用场景

```
一、变量都是用var来定义的
变量是用来存储用户的数据的，每一种变量都有自己的类型
使用例子：
var a = &sync.Map{}
a.store("","")//这里面传递的就是要存储的变量


二、类型
类型都是用Type来定义的
1、类型别名
type a = int  这个a就是int
2、类型都是type在前面
如结构体
type a struct{}
3、类型声明函数
type Fn func()xx

有了类型系统，才使函数式编程成为可能

```





### 数组

```
一、数组的声明
var a [2]int
a:=[2]int{}
a:=[...]int{1}

遍历用for循环
```

#### 数组的内存布局

```
数组是一段固定长度的连续内存区域，是值类型
```

#### 数组相关的api

```
1、数组是值类型，改变副本的值，不会改变本身
2、两个数组的比较
	1、如果两个数组的类型相同，并且容量相同，那么这个数组就是可以比较的，否则是不可以直接比较的
	var a = [2]int
	var b = [2]int
	if (a == b){
      return ture
	}else{
      return false 
	}
3、如果要对数组进行传参，如果不是指针的话，对数组的修改，只是修改副本，如果传递的是指针，那么就会修改到原来的数组
```

### 切片

```go
动态分配大小的连续空间
slice有三部分组成，指针，容量和长度
1、表示原有的切片
	a[:]
2、清空切片
	a[0:0]
3、声明切片
	var a []int
	var a = []int{}
	var a = b[:]
4、使用make来给slice分配内存，动态的创建切片
make([]int,2,3)
```

#### 切片内存布局

```
1、切片引用的是底层的数组，并且slice的地址是指向数组的第一个元素的地址
2、如果添加的数超过了slice的cap，那么就会根据当前容量进行扩容，扩容是当前容量的二倍，扩容之后的地址和之前的地址是不一样的，会生成一个新的slice
3、字符串和slice的区别。
	字符串通过切片得到的还是字符串
	slice通过切片得到的是slice
4、在64位架构的机器上，一个切片需要24B的内存，指针字段需要8B，长度和容量各8B
```

#### 切片的删除操作骚操作

```
func remove(s []int,l int)[]int{
  s[l] = slice[len(s)-1]
  return s[:len(s)-1]
}

```

#### 切片的函数式编程的应用

```go
一、使用事件系统的方式
其实就是用切片生成一个充满函数名的队列，如下面实现一个限流的队列
用户使用的话，就调用usercall和ipcall的方法就可以了，大概流程：
1、把类型方法存储到sync.Map中，这个map的value是slice类型
2、把struct的方法，存储到slice中，加入到sync.Map中，  1和2都是注册
3、如果使用的话直接从map中去获取，如果获取不到就执行2中的方法，也就是调用。看情况获取返回值


package server

import (
	"ApiGateway/Config"
	"github.com/astaxie/beego/logs"
	"sync"
)

/*
这里如果有id的map记录的话，那么就调用调用函数
如果没有的话，就调用注册函数
然后ip的也是一样，因为有两个map所以要写两个函数

注意：这里要用每个结构体的方法作为函数类型，因为这样的话，可以把结构体的
其他字段当作缓存来使用
1、声明map
2、写结构体，和对应的方法以及对应的方法的类型函数
3、写注册函数，把方法注册到map中填充1声明的map
4、写调用函数
*/

var(

	userMap = &sync.Map{}
	ipMap = &sync.Map{}
	fn  FnSlice
)

type UserLimit struct {
	count int
	currentTime int64
}

type IpLimit struct {
	count int
	currentTime int64
}

type Fn func(nowTime int64)(currentCount int)

//这个用于sync.Map的转化
type FnSlice  []Fn

func (p *UserLimit)Sec(nowTime int64)(currentCount int){
	if p.currentTime != nowTime {
		p.count = 1
		p.currentTime = nowTime
		currentCount = p.count
		return
	}

	p.count++
	currentCount = p.count
	return
}

func (p *UserLimit)Min(nowTime int64)(currentCount int){
	if nowTime-p.currentTime > 60 {
		p.count = 1
		p.currentTime = nowTime
		currentCount = p.count
		return
	}

	p.count++
	currentCount = p.count
	return
}

func (p *IpLimit)Sec(nowTime int64)(currentCount int){
	if p.currentTime != nowTime {
		p.count = 1
		p.currentTime = nowTime
		currentCount = p.count
		return
	}

	p.count++
	currentCount = p.count
	return
}

func (p *IpLimit)Min(nowTime int64)(currentCount int){
	if nowTime-p.currentTime > 60 {
		p.count = 1
		p.currentTime = nowTime
		currentCount = p.count
		return
	}

	p.count++
	currentCount = p.count
	return
}

func UserRegister(id int){
	userlimit := new(UserLimit)
	//首先存储为空，构成map的类型，store里面要跟变量而不是类型，注意类型和变量要分清
	userMap.Store(id,fn)
	//之后获取空的value
	f,_ := userMap.Load(id)
	//类型断言，注意后面要跟类型也就是type声明的
	fnslice := f.(FnSlice)
	fnslice = append(fnslice,userlimit.Sec)
	fnslice = append(fnslice,userlimit.Min)
	userMap.Store(id,fnslice)
}

func IpRegister(ip string){
	iplimit := new(IpLimit)
	ipMap.Store(ip,fn)
	f,_ := ipMap.Load(ip)
	fnslice := f.(FnSlice)
	fnslice = append(fnslice,iplimit.Sec)
	fnslice = append(fnslice,iplimit.Min)
	ipMap.Store(ip,fnslice)
}

//这里如果id没有记录的话就加入，然后还要计算，因为要加1
func UserCall(id int,param int64)bool{
	limit,ok := userMap.Load(id)
	if !ok {
		UserRegister(id)
	}
	secUserCount := limit.(FnSlice)[0](param)
	minUserCount := limit.(FnSlice)[1](param)
	if secUserCount > Config.User_sec_acc{
		logs.Warn("this user%v a second visit %v",id,secUserCount)
		return false
	}
	if minUserCount > Config.User_min_acc{
		logs.Warn("this user%v a min visit %v",id,minUserCount)
		return false
	}
	return true
}

func IpCall(ip string,param int64)bool{
	limit,ok := ipMap.Load(ip)
	if !ok {
		IpRegister(ip)
	}
	secIpCount := limit.(FnSlice)[0](param)
	minIpCount := limit.(FnSlice)[1](param)
	if secIpCount > Config.Ip_sec_acc{
		logs.Warn("this Ip %v a second visit %v",ip,secIpCount)
		return false
	}
	if minIpCount > Config.Ip_min_acc{
		logs.Warn("this Ip %v a Second visit %v",ip,minIpCount)
		return false
	}
	return true
}


二、通过闭包的方式实现生成器

三、字符串的链式操作
需要准备
1、需要处理的字符串列表，slice的元素是字符串
2、需要处理的函数链,slice的元素是函数名字
3、处理函数列，两个参数，一个采纳数是1，一个参数是2。这个函数把1中的元素作为2中的元素的参数，进行操作。然后把2中每个参数处理的结构保存在1中按照index方式保存起来



```

#### 切片的比较

```go
一、slice不能用==直接来比较，但是可以用下面的方式
func equal(x,y []string)bool{
  if len(x) != len(y){
    return false
  }
  for i:=range x{
    if x[i] != y[y]{
      return false
    }
  }
  return true
}
二、slice的返回值是nil也就是说可以和nil做比较
```

#### 切片的反转

```go
func revers(s []int){
  for i,j :=0,len(s)-1;i<j;i,j = i+1,j-1{
    s[i],s[j] = s[j],s[i]
  }
}
```



### Map

```go
一、map
1、这里的map不是线程安全的，然后还需要用make的方式进行创建
2、初始化的两种方式
m := make(map[int]string,初始值，容量)
m := make(map[int]string)

二、sync.Map
这里的map是安全的，
a:= &sync.Map{}
添加
a.store(k,v) //注意这里面的k，v都是变量类型
v,ok := a.Load(k)  //获取

删除
a.Delete(k)

遍历
//遍历sync.Map, 要求输入一个func作为参数
	f := func(k, v interface{}) bool {
		//这个函数的入参、出参的类型都已经固定，不能修改
		//可以在函数体内编写自己的代码，调用map中的k,v
 
			fmt.Println(k,v)
			return true
		}
m.Range(f)

```

#### Map的内存布局

```
1、map在底层是哈希表实现的，map就是一个哈希数组列表，由一个个bucket组成，此列表中的每一个元素都成为bucket的结构体，每个bucket可以保存8个键值对，bucket填满后，将通过一个overflow指针来扩展bucket，形成一个链表，解决哈希的冲突问题。
简单的说，这个map就是一个bucket指针型的一维数组，每个bucket的指针不定长，可能挂着bucket指针lst，也可能只有一个，而是由hash冲突而定
2、map是引用类型，内存是随机的。
3、map的键必须是可以通过操作符==比较的数据类型
```

#### Map的一些骚算法

```go
给一个数组，然后计算里面的两个数相加等于另外一个数的算法：
下面是利用了map的key是唯一性的问题
package main

import (
   "fmt"
)

func main(){
   var a = []int{1,2,5,19,20,7}
   L(a,21)
}

func L(a []int,b int){
   var Cmap = make(map[int]int)
   for k,v :=range a{
      if i,ok:=Cmap[b-v];ok&& k!=i{
         fmt.Println(i,k)
         break
      }
      Cmap[v] = k
   }
}
```

#### Sync.Map的用法以及断言

```go
待更新···
```

#### Map的api功能常用的

```go
1、遍历。这种迭代顺序是不固定的，顺序是随机的
for k,v := range m{
  ...
}
2、查找，做判断这个key是否在这个map中
value,ok := m[k]
```

#### Map的比较

```go
1、map是不可以比较的，但是可以和nil比较
2、为了判断两个map是否拥有享用的键值对，必须写一个循环

func equal(x,y map[string]int)bool{
  if len(x) != len(y){
    return false
  } 
  for k,v := range x{
    if xy,ok :=y[k];!ok || xy != v{
      return false
    }
  }
}
```

#### Map的函数式编程

```
主要还是利用map的注册系统来实现，可以详细看slice中的
```

#### Map的注册系统

```go
主要是三部分组成
1、对外的调用函数
2、注册函数
3、需要一个map，这个map的value如果是接口，那么就是面向对象编程，如果是slice里面的元素是函数类型，那么就是函数式变成
//声明的接口
type Balancer interface {
	DoBalance([]*Instance, ...string) (*Instance, error)
}

type BalanceMgr struct {
	allBalancer map[string]Balancer
}
//初始化的map
var mgr = BalanceMgr{
	allBalancer: make(map[string]Balancer),
}
//内部的注册函数
func (p *BalanceMgr) registerBalancer(name string, b Balancer) {
	p.allBalancer[name] = b
}

//对外的调用函数
func RegisterBalancer(name string, b Balancer) {
	mgr.registerBalancer(name, b)
}
上面的注册系统，是面向对象编程的，主要是进行某种方法的扩展
即把结构体的地址注册到map中
```

### struct

```
1、结构体是值类型
2、结构体的成员如果都是可以比较的，那么就可以比较
4、如果结构体可以比较，那么这个结构体就可以做为map的key
m[struct]int
```

#### struct的内存布局

```
struct是值类型，成员地址是连续的
```

#### struct初始化

```
func a()*struct{
  return &struct{}
}
```

#### struct的比较

```
如果结构体可以比较，那么这个结构体就可以做为map的key
m[struct]intstruct中的成员
```

#### struct与interface的比较

```go
1、struct必须实现interface的所有的方法，才能进行比较
2、结构体可以嵌套，所以如果父和子各实现了interface的方法，那么实例化这个父的结构体，那么就是实现了这个接口
如:
type service interface {
  start()
  log()
}

type log struct{
  
}

func (p *log)start(){
  
}

type gameserver struct{
  log
}

func (p *gameserver)log(){
  
}

func main(){
  a:=new(gameserver)
  a.start()
  a.log()
}
```

#### struct的函数式编程,两种方式

```go
下面都可以结合定时器来使用
一、以结构体的字段为函数类型的函数式编程（主要是对于嵌套结构体的架构）
//父结构体
type Worker struct {
	ticker *time.Ticker
	runner *Runner
}
//子结构体
type Runner struct {
	Controller controlChan
	Error controlChan
	Data dataChan
	dataSize int
	longLived bool
	Dispatcher fn 
	Executor fn
}

const (
	READY_TO_DISPATCH = "d"
	READY_TO_EXECUTE = "e"
	CLOSE = "c"

	VIDEO_PATH = "./videos/"
)

type controlChan chan string

type dataChan chan interface{}

type fn func(dc dataChan) error
//工厂函数，初始化子结构体
func NewRunner(size int, longlived bool, d fn, e fn) *Runner {
	return &Runner {
		Controller: make(chan string, 1),
		Error: make(chan string, 1),
		Data: make(chan interface{}, size),
		longLived: longlived,
		dataSize: size,
		Dispatcher: d,
		Executor: e,
	}
}
//初始化父结构体
func NewWorker(interval time.Duration, r *Runner) *Worker {
	return &Worker {
		ticker: time.NewTicker(interval * time.Second),
		runner: r,
	}
}

//定时器方法
func (w *Worker) startWorker() {
	for {
		select {
		case <- w.ticker.C:
			go w.runner.StartAll()
		}
	}
}
//实现定时器的方法
func (r *Runner) StartAll() {
	r.Controller <- READY_TO_DISPATCH
	r.startDispatch()
}
//去具体的完成逻辑
func (r *Runner) startDispatch() {
	defer func() {
		if !r.longLived {
			close(r.Controller)
			close(r.Data)
			close(r.Error)
		}
	}()

	for {
		select {
		case c :=<- r.Controller:
			if c == READY_TO_DISPATCH {
				err := r.Dispatcher(r.Data)
				if err != nil {
					r.Error <- CLOSE
				} else {
					r.Controller <- READY_TO_EXECUTE
				}
			}

			if c == READY_TO_EXECUTE {
				err := r.Executor(r.Data)
				if err != nil {
					r.Error <- CLOSE
				} else {
					r.Controller <- READY_TO_DISPATCH
				}
			}
		case e :=<- r.Error:
			if e == CLOSE {
				return
			}
		default:

		}
	}
}
//和数据库相关了
func VideoClearDispatcher(dc dataChan) error {
	res, err := dbops.ReadVideoDeletionRecord(3)
	if err != nil {
		log.Printf("Video clear dispatcher error: %v", err)
		return err
	}

	if len(res) == 0 {
		return errors.New("All tasks finished")
	}

	for _, id := range res {
		dc <- id
	}

	return nil
}
//执行数据库相关了
func ReadVideoDeletionRecord(count int) ([]string, error) {
	stmtOut, err := dbConn.Prepare("SELECT video_id FROM video_del_rec LIMIT ?")

	var ids []string

	if err != nil {
		return ids, err
	}

	rows, err := stmtOut.Query(count)
	if err != nil {
		log.Printf("Query VideoDeletionRecord error: %v", err)
		return ids, err
	}

	for rows.Next() {
		var id string
		if err := rows.Scan(&id); err != nil {
			return ids, err
		}

		ids = append(ids, id)
	}

	defer stmtOut.Close()
	return ids, nil
}

func VideoClearExecutor(dc dataChan) error {
	errMap := &sync.Map{}
	var err error

	forloop:
		for {
			select {
			case vid :=<- dc:
				go func(id interface{}) {
					if err := deleteVideo(id.(string)); err != nil {
						errMap.Store(id, err)
						return
					}
					if err := dbops.DelVideoDeletionRecord(id.(string)); err != nil {
						errMap.Store(id, err)
						return 
					}
				}(vid)
			default:
				break forloop
			}
		}

	errMap.Range(func(k, v interface{}) bool {
		err = v.(error)
		if err != nil {
			return false
		}
		return true
	})

	return err
}

func DelVideoDeletionRecord(vid string) error {
	stmtDel, err := dbConn.Prepare("DELETE FROM video_del_rec WHERE video_id=?")
	if err != nil {
		return err
	}

	_, err = stmtDel.Exec(vid)
	if err != nil {
		log.Printf("Deleting VideoDeletionRecord error: %v", err)
		return err
	}

	defer stmtDel.Close()
	return nil
}


func Start() {
	// Start video file cleaning
	r := NewRunner(3, true, VideoClearDispatcher, VideoClearExecutor)
	w := NewWorker(3, r)
	go w.startWorker()
}



二、以结构体的方法为函数类型的函数式编程
可以用结构体的方法来改造上面的例子，会更加简单一点
1、实现结构体的方法
2、实现函数类型，
3、实现map，其中map的value为函数类型的slice
4、实现注册函数，把结构体的方法注册到map中
5、调用函数，查找map中是否有其中的value，如果没有那么就执行注册函数。如果有，就实现有的逻辑
```

#### 

### 函数式编程

```
一、匿名函数
a:= func(){}
二、闭包，闭包可以用于生成器，工厂函数
func a()func(){
  return func(){}()
}
```

#### 函数式编程的套路

```
一、链式操作
1、首先声明一个列表，这个列表是保存要调用函数的参数
2、声明一个列表，这个列表是保存函数名的
3、写一个执行方法，参数为上面的两个参数。然后遍历第一个列表，
```



### 面向对象编程

```
面向对象主要是通过interface来实现的
```

#### 面向对象编程的套路

```
面向对象的套路有两个
1、通过注册调用的方式来实现

```



### 接口

```
接口是实现面向对象的重要工具
判断是否实现了接口，那么可以用接口的断言的方式来判断

空接口可以保存任意的值
var a int =1
var i interface{} = a
var b int = i.(int)
这样就可以传递了，注意interface类型的值如果转化的话，需要做类型断言
```

#### 接口的比较

```
空接口只能比较保存的是固定值
疑问：接口是否比较看是否是一样的方法？
```

#### 接口的断言方式

```go
一、查询是否实现了这个接口
t,ok := struct等.(interface的名字)
例子：
type Flyer interface{
  fly()
}

type bird struct{}

func (p *bird)fly(){
  
}

func main(){
  a:=make(map[string]interface{})
  a["x"] = new(bird)
  
  for k,v := range a{
    f,ok := v.(Flyer)
    if !ok{
      return
    }else{
       f.fly()
    }
  }
}

二、使用类型分支的方式进行断言
switch 接口变量.(type):
case int:
xxx
case string:
xxx
return

```

#### 接口的类型

```
接口的类型是指针
```

#### 类型和接口的关系

```go
类型只要实现了接口的方法，就实现了这个接口
1、结构体实现接口
type a interface{
  call()
}

type b struct{
  
}
func (p *b)call(){
  
}

2、函数实现接口
type funca func(interface{})

func (f funca)call(a interface{}){
  f(a)  //调用函数本体
}
//注意这里是转换，把匿名结构体进行了类型转换
c:=funca(func(a interface{}){
  fmt.Println(a)
})
c.call()
```

#### 父子接口的关系和实现以及作用

```go
待更新···
```

### 函数

```go
待更新
```

#### 函数的闭包，以及应用场景

```
1、用于生成器
2、用于middleware。
```

#### 函数的参数

```
变长参数需要用...
```

#### 函数的各种数据类型的返回值

```
字符串的返回值：""
引用类型的返回值：nil

```

#### 函数的链式操作以及应用场景

```go
上面的链式操作是函数的，还有结构体方法的链式操作

type s struct{}

func (p *s)do1()*s{
  dosomething
  return p
}
func (p *s)do2()*s{
  dosomthing
  return p
}
//调用的话
a:=new(s)
a.do1().do2()

用于事物的方式

```

#### 函数的接口实现

```
思路：
1、写一个函数类型
2、写一个函数类型的方法
3、写一个接口，让函数类型实现接口的方法
4、然后通过类型转换的方式实现类型转换，之后.方法()就可以了
```

#### 函数的延迟操作

```go
defer
1、defer是类似于栈的情况，先进后出
2、defer的参数如果是函数()的方式，会先初始化这个函数
如：
func main(){
  defer func(a()){
   	fmt.Println("this is defer") 
  }()
  fmt.Println("1")
}

func a(){
  fmt.Println("this is a ")
}
结果会先打印出a的print，然后再打印1，之后再打印this is defer

3、defer和return的区别
return并不是原子操作，返回值和return的返回值不一定一致，defer、return和返回值的顺序应该是，return最先给返回值赋值，然后defer执行一些收尾工作，最后return指令携带返回值退出函数
defer会受到外部的参数的影响。

注意：通过第三点，可以明确知道函数的返回值要有名字，如果没有名字的话，那么defer执行之后的值和return的值是不一样的（详细看书籍）


```

#### 匿名函数，以及应用场景

```
1、闭包
2、可以通过下面的方式来命名
 a := func(){}
```

#### 函数的宕机恢复

```
defer func(){if err == recover{xxx}}()
```

#### sync.Once在函数中的应用

```go
启动多个相同goroutine，但是里面的某个操作我只希望被执行一次，这个时候Once就上场了。
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	var once sync.Once
	one := func() {
		fmt.Println("just once")
	}

	for i := 0; i < 10; i++ {
		go func(a int) {
			once.Do(one) // 只是被执行一次
		}(i)

	}
	time.Sleep(time.Millisecond * 200)
}
```

### Cond:

```go
 sync.Cond是用来控制某个条件下，goroutine进入等待时期，等待信号到来，然后重新启动。
 1package main
 2
 3import (
 4    "fmt"
 5    "sync"
 6    "time"
 7)
 8var locker = new(sync.Mutex)
 9var cond = sync.NewCond(locker)
10
11func test(x int) {
12    cond.L.Lock() //获取锁
13    cond.Wait()//等待通知 暂时阻塞
14    fmt.Println(x)
15    time.Sleep(time.Second * 1)
16    cond.L.Unlock()//释放锁
17}
18func main() {
19    for i := 0; i < 40; i++ {
20        go test(i)
21    }
22    fmt.Println("start all")
23    time.Sleep(time.Second * 3)
24    fmt.Println("signal1")
25    cond.Signal()   // 下发一个通知随机给已经获取锁的goroutine
26    time.Sleep(time.Second * 3)
27    fmt.Println("signal2")
28    cond.Signal()// 下发第二个通知随机给已经获取锁的goroutine
29    time.Sleep(time.Second * 1)  // 在广播之前要等一会，让所有线程都在wait状态
30    fmt.Println("broadcast")
31    cond.Broadcast()//下发广播给所有等待的goroutine
32    time.Sleep(time.Second * 60)
33}
```



#### 有名函数的实现方式

```go
一、
func a(){}
二、
a := func(){}
```

### 流程控制的几个关键部分

```
一、break loop的退出
   goto loop 的退出
 这种用于for循环的多层嵌套，或者for循环里面嵌套switch或者

```

#### loop

```go
一、但存的用下面的方式
func main() {
Loop:
	for i := 0; i < 9; i++ {
		if i == 1 {
			break Loop
		}
	}

	fmt.Println("ok")
}
二、在for循环中嵌套switch的话，就不能单独的用break了，因为这个时候只能跳出switch，不能跳出for循环，所以这个时候可以用break Loop，也可以用goto
func main() {
	Loop:
		for i := 0; i < 9; i++ {

			switch  {
			case i==1:
				break Loop
			case i==2:
				fmt.Println("2")
			case i == 3:
				fmt.Println("3")
			default:
				fmt.Println("default")
			}
		}

	fmt.Println("ok")
}
结果是
default
ok

```

#### goto

```go
goto在多重循环的情况下可以使用

import (
	"fmt"
)

func main() {
	for i := 0; i < 9; i++ {
		if i == 1 {
			goto Loop
		}
	}
Loop:
	fmt.Println("ok")
}
例子2
func main() {

		for i := 0; i < 9; i++ {

			switch  {
			case i==1:
				goto Loop
			case i==2:
				fmt.Println("2")
			case i == 3:
				fmt.Println("3")
			default:
				fmt.Println("default")
			}
		}
	Loop:
		fmt.Println("ok")
}
结果：
default
ok

```

#### 怎么跳出select

```
goto//或者break loop的方式
```

#### for循环的省略方式

```
for循环后面有三个值，主要是根据这三个值类确定，这个也叫做for循环的子语句，
```



### 逃逸分析

```go
待更新···
```





### 并发

```
待更新
```

#### 互斥锁

```
待更新
```

#### 读写锁

```
读写锁：可以并发的读取，但是要注意，读和写只能同时执行一个
```

#### channel的声明方式

```go
一、channel的声明
var c chan int
var m map[string]chan bool
初始化
c:=make(chan int)
m:=make(map[string]chan bool)

二、channel可以做生成器，下面是限流的例子

type ConnLimiter struct {
	ConcurrentConn int //定值
	bucket         chan int
}

//构造结构体  运用工厂模式，对外调用
func NewConnLimiter(cc int)*ConnLimiter{
	return &ConnLimiter{
		ConcurrentConn:cc,
		bucket:make(chan int,cc),
	}
}

//进来的时候来调一下tocken
func (cl *ConnLimiter)GetConn()bool{
	if len(cl.bucket)>cl.ConcurrentConn{
		log.Printf("reached the rate limitation.")
		return false
	}
	cl.bucket <- 1
	return true
}

//这个方法是访问结束后进行调用的
func (cl *ConnLimiter)ReleaseConn(){
	_=<- cl.bucket
	log.Printf("new Connction coming:%d",)
}
上面就是通过channel的容量进行生成器来限流的
```

##### channel的特性，单项通信，双向通信，有缓冲和无缓冲，以及关闭了channel的一系列操作的结果,关闭channel的应用场景

```go
一、单向channel：
var ch1 chan<- int：单向写
var ch2 <-chan int :单向读

单向channel的应用，注意旨在传参的时候用

首先初始化双向channel
a:=make(chan int)
A（a）
func A(c chan <-int){}
func Aa (cc <-chan int)

二、有缓冲的channel，在没有读取完毕之前不会阻塞，没有缓冲的channel会阻塞，如果先读后写的话会导致程序永远阻塞

三、往关闭的channel中写数据会发生panic，从关闭了的channel中读取数据会把channel中的数据读取完毕，如果读取完毕会读取零值，可以用ok的方式去判断，应用场景，可以做广播通知，也可以做同步goroutine的方式

四、试图关闭一个关闭的通道会宕机。垃圾回收机制会根据一个通道是否可以访问来回收，而不是关闭

五、通过channel向外部goroutine报告完成来控制子goroutine 或者通过waitgroup可以保证goroutine不被泄露，在主线程结束前结束
```

##### 用channel实现锁

```
https://colobu.com/2018/03/26/channel-patterns/?from=singlemessage&isappinstalled=0#TryLock_By_Channel
```

##### 用channel控制goroutine

```go
1、通过channel向外部goroutine报告完成来控制子goroutine
2、通过控制父goroutine程序的退出方式来关闭
func a(){
  defer n.Done()
  if c{
    return
  }
  go xx()
}
但是上面的性能不好
3、推荐方式用select的方式
func a(){
  select{
    case <ch
    return
    
  }
}

4、可以通过context方式
```

##### context

```
1、退出机制，可以通知传递给整个goroutine调用树上的每个goroutine
2、传递数据，数据可以传递给整个goroutine调用树上的每一个goroutine
```



##### channel的模型机理

```
一、单向通道  （从后往前）
var a chan<-int   //只能发送通道
var a <- chan int //只能接收通道

1、初始化方式一
ch := make(chan int)
var chsend chan <- int = ch
var chrecv <- chan int = ch

2、初始化方式二
ch :=make(<-ch int)

二、channel （引用类型）
1、chan的类型的空值是nil
2、需要经过make创建才可以使用
```

##### channel实现goroutine的非阻塞，也就是上面的同步的例子2

```
待更新
```

##### channel来控制goroutine异步的等待锁

```go
package main

import (
	"fmt"
	"time"
)

var chan1 = make(chan string, 512)

var arr1 = []string{"qq", "ww", "ee", "rr", "tt"}

func chanTest1() {
	for _, v := range arr1 {
		chan1 <- v
	}
	close(chan1) // 关闭channel
}

func chanTest2() {
	for {
		getStr, ok := <-chan1 // 阻塞,直到chan1里面有数据
		if !ok {              // 判断channel是否关闭或者为空
			return
		}
		fmt.Println(getStr) // 按数组顺序内容输出
	}
}

func main() {
	go chanTest1()
	go chanTest2()

	time.Sleep(time.Millisecond * 200)
}

```

##### channel来控制子goroutine的状态

```
待更新
```

##### 如何优雅的关闭channel

```
待更新
```

##### 使用struct和channel来阻塞资源

```go
待更新
```



##### 使用channel来控制锁超时

```go
待更新
```

##### 使用一个channel来控制另外一个channel的输入

```go
这种模式是我们经常使用的一种模式，通过一个信号channel(done)来控制(取消)输入channel的处理。

一旦从done channel中读取到一个信号，或者done channel被关闭， 输入channel的处理则被取消。

这个模式提供一个简便的方法，把done channel 和 输入 channel 融合成一个输出channel。

func orDone(done <-chan struct{}, c <-chan interface{}) <-chan interface{} {
	valStream := make(chan interface{})
	go func() {
		defer close(valStream)
		for {
			select {
			case <-done:
				return
			case v, ok := <-c:
				if ok == false {
					return
				}
				select {
				case valStream <- v:
				case <-done:
				}
			}
		}
	}()
	return valStream
}
```

##### channel的扇入模式

```go
扇入模式(FanIn)是将多个同样类型的输入channel合并成一个同样类型的输出channel，也就是channel的合并

每个channel起一个goroutine。
func fanIn(chans ...<-chan interface{}) <-chan interface{} {
	out := make(chan interface{})
	go func() {
		var wg sync.WaitGroup
		wg.Add(len(chans))
		for _, c := range chans {
			go func(c <-chan interface{}) {
				for v := range c {
					out <- v
				}
				wg.Done()
			}(c)
		}
		wg.Wait()
		close(out)
	}()
	return out
}

```

##### channel的Tee模式

```go
扇出模式(FanOut)是将一个输入channel扇出为多个channel。

扇出行为至少可以分为两种：

从输入channel中读取一个数据，发送给每个输入channel，这种模式称之为Tee模式
从输入channel中读取一个数据，在输出channel中选择一个channel发送
本节只介绍第一种情况，下一节介绍第二种情况

func fanOut(ch <-chan interface{}, out []chan interface{}, async bool) {
	go func() {
		defer func() {
			for i := 0; i < len(out); i++ {
				close(out[i])
			}
		}()
		for v := range ch {
			v := v
			for i := 0; i < len(out); i++ {
				i := i
				if async {
					go func() {
						out[i] <- v
					}()
				} else {
					out[i] <- v
				}
			}
		}
	}()
}
```

##### channel的集合操作

```go
一、map类型，这里map因为是一个关键字，所以这里用mapchan来替代
map和reduce是一组常用的操作。

map将一个channel映射成另外一个channel， channel的类型可以不同。
func mapChan(in <-chan interface{}, fn func(interface{}) interface{}) <-chan interface{} {
	out := make(chan interface{})
	if in == nil {
		close(out)
		return out
	}
	go func() {
		defer close(out)
		for v := range in {
			out <- fn(v)
		}
	}()
	return out
}

二、reduce
func reduce(in <-chan interface{}, fn func(r, v interface{}) interface{}) interface{} {
	if in == nil {
		return nil
	}
	out := <-in
	for v := range in {
		out = fn(out, v)
	}
	return out
}
你可以用`reduce`实现`sum`、`max`、`min`等聚合操作。
三、、还有待补充
```

#### goroutine的机理和调度

```
待更新
```

#### 经典生产者消费者模型

```go
1、要有三个goroutine  （response的body是需要被关闭的）
第一个goroutine，也就是生产者，用go product(chan)的方式生产，然后放入到chan中
第二个goroutine，消费者，不断的读取chan，方式
go func(){
  for {
    r ,ok :=<-chan 
    if ok{
      slice = append(slice,r)
    }else{
      break
    }
  }
}()
第三个goroutine也就是最外边的gouroutine，是做广播通知用的，也就是关闭channel，但是要用waitgroup用来等待生产者完毕，然后close(chan)

代码示例
package main

func ServerHttp(ctx context.Context,w http.ResponseWrite,r *http.Request){
  var ch = make(chan int,10)
  var wg = &sync.WaitGroup{}
  var slice = make([]int,10)
  var sig = make(chan int ,1)
  
  body,err := ioutil.ReadAll(r.body) //这里应该用bufio来读取，可以试验下
  if err != json.Unmarshal(body,xx);err != nil {
      return
  }
  //todo  validate验证
  wg.Add(1)
  go product(ctx,ch,wg)
  go func(){
    r,ok :=<- ch
    if ok{
      slice = append(slicce,r)
    }else{
      sig <- 1  //这里用来阻塞外部的goroutine，防止退出
      break
    }
  }()
  wg.wait()  //注意要在消费者后面阻塞
  close(ch)  //这里做广播通知消费者可以退出了
  <-ch
  fmt.Println(xxx)
  io.WriteString(w,ok)
  return
}

```

#### 使用goroutine需要注意的问题点

```
1、使用框架的话，如果在每个接口的外部声明channel的话，如果在接口内部关闭，会出现问题。会panic，因为调用这个接口的会是多次
2、每个goroutine内部起来的时候都要写goroutine
3、
```



#### goroutine的Context

```

```

#### goroutine的发布订阅

```

```

### Select的使用

```

```

### select的高级使用

```go
for {
  select{
    case <- xx:
    dosomething()
  }
}
上面的是阻塞式的方式使用，这种用于定时器等的需要阻塞的场景，例如会议系统等
```



### 定时器的使用

```go
一次定时器会发生内存泄漏time.After(1*time.Second)
所以要用下面的方式，超时机制
package main

import (
   "net/http"
   "io"
   "time"
   //"fmt"
)

var G = make(chan int)

func main(){
   http.HandleFunc("/",T(Do))
   //if err := http.ListenAndServe(":8088", nil); err != nil {
   //}
   http.ListenAndServe(":8080",nil)

}

func T(handler http.HandlerFunc)http.HandlerFunc{
   return func(w http.ResponseWriter,r *http.Request){

      timer := time.NewTicker(time.Second)
      defer func(){
        //一定要关闭定时器
         timer.Stop()
      }()
      go handler(w,r)
      select {
      case <-G:
         return
      case <- timer.C:
         io.WriteString(w, "222")
         return
      }


   }
}

func Do(w http.ResponseWriter,r *http.Request){
   time.Sleep(3*time.Second)
   io.WriteString(w, "ok")
   G <- 1
   return
}

```

### 进阶话题

```
1、什么时间发生上下文切换
2、函数式编程的在三种数据结构中的使用
切片、map、和结构体（结构体中的字段）
```


