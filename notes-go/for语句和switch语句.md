### 18、for语句和switch语句


#### for...range语句需要注意的问题
```go

numbers1 := []int{1, 2, 3, 4, 5, 6}
for i := range numbers1 {
	if i == 3 {
		numbers1[i] |= i
	}
}
fmt.Println(numbers1)

说明：
	[1 2 3 7 5 6]
原因：
	切片是引用类型，i是索引
	

循序渐进，下一个列子

numbers2 := [...]int{1, 2, 3, 4, 5, 6}
maxIndex2 := len(numbers2) - 1
for i, e := range numbers2 {
	if i == maxIndex2 {
		numbers2[0] += e
	} else {
		numbers2[i+1] += e
	}
}
fmt.Println(numbers2)

说明：
	[7 3 5 7 9 11]
原因：
	numbers2是数组，range表达式是进行一次计算，迭代中修改number2的值并不会影响后面迭代中number2的值
	
最后一个列子

numbers3 := []int{1, 2, 3, 4, 5, 6}
maxIndex3 := len(numbers3) - 1
for i, e := range numbers3 {
	if i == maxIndex3 {
		numbers3[0] += e
	} else {
		numbers3[i+1] += e
	}
}
fmt.Println(numbers3)	

说明：
	[22 3 6 10 15 21]
原因：
	range表示试的值虽然是进行一次，但是number3是引用类型的切片，所有在迭代中修改了number3的值是会影响后面迭代使用到的值的。	

```

**总结一下**  
1、range表达式的值只会计算一次  
2、range的表达式求值结果会被复制，被迭代的对象是range表达式结果  
3、数组是值类型，切片是引用类型。数组循环的时候是复制了一份，所以在迭代的过程中值是不会改变的，切片是引用类型，虽然复制了一份，但是底层都是对应的同一个数组结构，所以在迭代中修改值，会印象后面迭代用到的值。  




#### switch语句中的switch表达式和case表达式之间有着怎样的联系？

```go

package main

import "fmt"

func main() {
	value1 := [...]int8{0, 1, 2, 3, 4, 5, 6}
	switch 1 + 3 {
	case value1[0], value1[1]:
		fmt.Println("0 or 1")
	case value1[2], value1[3]:
		fmt.Println("2 or 3")
	case value1[4], value1[5], value1[6]:
		fmt.Println("4 or 5 or 6")
	}
}


说明：
	这段代码是编译不通过的
原因：
	switch表达式是1+3，这个求值的结果是无类型的常量4，那么这个常量会被自动的转换为此种常量的默认值类型，
整数4的默认值就是int类型，每一个case后面的表达式的类型是int8，所以类型不匹配是编译不通过的。

```

-

```go

把代码修改成以下：

value2 := [...]int8{0, 1, 2, 3, 4, 5, 6}
switch value2[4] {
case 0, 1:
	fmt.Println("0 or 1")
case 2, 3:
	fmt.Println("2 or 3")
case 4, 5, 6:
	fmt.Println("4 or 5 or 6")
}

说明：
	这段代码是可以编译成功，可以正确的运行的
原因：
	switch语句会进行自动类型转换，转换的标准是switch表达式。在这个地方switch表达式的类型是int8，
	case表达式都是可以进行自动类型转换的，所以是可以编译通过的。
注意：
	如果自动类型转换没有成功（也就是没有匹配），那么也是不会通过编译的

```

**总结一下**  
从上面的例子可以推断出来，switch表达式和case表达式之间的联系了，因为要进行判等的操作，
	所以两者的类型和值都相同才可以，而且switch会进行自动类型转换，转换的标准是switch表达式


#### switch语句对它的case表达式有哪些约束？

```go

value3 := [...]int8{0, 1, 2, 3, 4, 5, 6}
switch value3[4] {
case 0, 1, 2:
	fmt.Println("0 or 1 or 2")
case 2, 3, 4:
	fmt.Println("2 or 3 or 4")
case 4, 5, 6:
	fmt.Println("4 or 5 or 6")
}

说明：
	代码不会通过编译
原因：
	switch语句在case子句的选择上具有唯一行，不过这个约束只针对常量的子表达式	

```


```go

value5 := [...]int8{0, 1, 2, 3, 4, 5, 6}
switch value5[4] {
case value5[0], value5[1], value5[2]:
	fmt.Println("0 or 1 or 2")
case value5[2], value5[3], value5[4]:
	fmt.Println("2 or 3 or 4")
case value5[4], value5[5], value5[6]:
	fmt.Println("4 or 5 or 6")
}

说明：
	这段代码可以编译通过
原因：
	switch语句虽然对case的语句选择有唯一性，不过是针对常量的表达式	

```

#### 万事都有列外，这里就有一个列外

```go

value6 := interface{}(byte(127))
switch t := value6.(type) {
case uint8, uint16:
	fmt.Println("uint8 or uint16")
case byte:
	fmt.Printf("byte")
default:
	fmt.Printf("unsupported type: %T", t)
}

说明：
	编译不通过
原因：
	byte类型是uint8类型的别名	

```
