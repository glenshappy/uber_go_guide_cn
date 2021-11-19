## 1.函数式选项模式（Functional Options）

需求分析：
假设现在一个对象的一些特性在初始化的时候，一般是有个默认值的。比如在我国的公安部门在为一些人员登记信息的时候，这些人员的默认国籍大部分都会是中国，所以公安工作人员在录入信息的时候系统自动默认录入人员是中国人，除非必要否则并不需要修改，这样减少了公安工作人员的工作量。再比如学校里的学生的犯罪前科都是无的，这一栏默认无就行。

但是Go语言并没有提供默认参数，一旦一个对象有很多属性（这些属性都可以有默认值）的时候，生成一个对象会变得及其麻烦。

比如我们要生成一个叫张三的人，当然张三还有其他的特性，比如身高，国籍，性别，年龄等，但是我们暂时除了他的名字之外啥都不知道，那么我们可以将其他值设置为一个默认的值。

type Person struct {
	Name string
	Age int
	Country string
	Gender string
	Height int
	Address string
}
func main()  {
	person := Person{
		Name:    "张三",    
		Age:     -1, 		// 不知道默认-1, 不想默认值为0
		Country: "China",	// 不知道，默认中国
		Gender:  "Male",	// 不知道，默认男性
		Height:  -1,		// 不知道，默认-1
		Address: "unknown",	// 不知道，直接填写“unknown”
	}
｝


可以看到这样写的话初试化一个对象很长，特别是当一个对象有更多的特性的时候，上面的初始化会显得冗长。

解决方案：函数式选项（Functional Options）
巧妙的利用了闭包和可变参数来做到给一个对象设置默认参数。具体解析在代码注释中。

package functionOptions

// 对象人
type Person struct {
	Name string
	Age int
	Country string
	Gender string
	Height int
	Address string
}

// 将func(*Person)这种类型的函数简化下命名
type per func(*Person)

// 使用了闭包
func Country(country string) per {
	return func(person *Person) {
		person.Country = country
	}
}

func Gender(gender string) per {
	return func(person *Person) {
		person.Gender = gender
	}
}

func Address(address string) per {
	return func(person *Person) {
		person.Address = address
	}
}

// 新建一个对象人，这个人必须有名字，其他的特性设置交给settings这可变参数中的闭包函数处理
func NewPerson(name string, settings ...per) *Person  {
	// 除了名字，都有默认值
	person := &Person{
		Name: name,
		Age: -1,
		Country: "China",
		Gender: "Male",
		Height: 0,
		Address: "unknown",
	}
	
	// 遍历可能存在所要设置的函数
	for _, setting := range settings {
		setting(person)
	}
	return person
}

用法：

package main

import (
	"fmt"
	op "studygo/pattern/functionOptions"
)

func main()  {
	person1 := op.NewPerson("张三")
	fmt.Println(person1)
	person2 := op.NewPerson("Mary", op.Gender("Female"), op.Country("Japan"))
	fmt.Println(person2)
}


输出结果对比：

&{张三 -1 China Male 0 unknown}
&{Mary -1 Japan Female 0 unknown}






