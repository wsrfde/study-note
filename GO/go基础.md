## 基础学习



### 语言结构

打印程序：

* ` fmt.Print()`   可以将字符串输出到控制台
* `fmt.Println()`  将字符串输出到控制台，并在最后自动增加换行字符 `\n`

格式化字符串

* `fmt.Sprintf()`  格式化字符串并赋值给新串

  ```go
  package main
  
  import "fmt"
  
  func main() {
  	var name = "vicer"
  	var age = 18
  	var jointStr = fmt.Sprintf("我的名字：%s,年龄：%d", name, age)
  	fmt.Println(jointStr)
  }
  ```

  

### 数据类型

