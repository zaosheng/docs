### 安装golong, 编写helloword

- 安装go

- 配置环境变量
  1.  新建目录 保存go项目 D:\go
  2. 添加环境变量 GOPATH：D:\go
  3. 新建三个目录D:\go\src D:\go\bin D:\go\pkg
  4. 添加环境变量D:\go\bin 到 PATH
  
- go env 查看配置的环境变量

- go build 编译 (默认在项目目录下执行go build)

  1. 报错 go.mod file not found in current directory or any parent directory; see 'go help modules'
     go env -w GO111MODULE=auto
  2. 指定文件名： go build -o hello.exe <项目路径从GOPATH/src后写起>

- go run 运行程序（像执行脚本一样执行）

- go install 

  1. 将项目文件编译成***.exe;
  2. 将***.exe拷贝到GOPATH/bin目录下

- 跨平台编译：

  1. 运行以下命令可在windows上编译在linux上运行的程序，其他类似

     ```
     SET CGO_ENABLE=0 //禁用CGO
     SET GOOS=linux  //目标平台
     SET GOARCH=amd64 //目标处理器架构
     ```

- Helloword

  ```
  // 包名，入口程序包名为main
  package main
  
  // 导包
  import "fmt"
  
  //函数外只能放标识符(变量、常量、函数、类型)的声明
  
  // 程序入口函数
  func main() {
  	fmt.Println("hello")
  }
  
  ```

  项目结构截图

  ![](C:\Users\jzs\AppData\Roaming\Typora\typora-user-images\image-20210722131428849.png)

### gin+gorm搭建go语言web框架

- 开发前准备

  ```
  # 配置代理
  # 在win10中，安装Go1.11之后的版本，打开go module：
  go env -w GO111MODULE=on
  #修改指定的代理，在命令行终端中输入：
  go env -w GOPROXY=https://goproxy.io,direct
  
  # 下载gorm包和gin包
  go get -u gorm.io/gorm
  go get -u github.com/gin-gonic/gin
  
  # 更改实时生效
  go get github.com/pilu/fresh
  
  # 运行项目
  go mod init project-name
  fresh
  
  ```

  