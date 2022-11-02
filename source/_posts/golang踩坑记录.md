---
title: golang踩坑记录
categories:
  - go
tags:
  - go
date: 2020-12-13 18:38:11
updated: 2020-12-17 23:18:16

---

很久没发博客了，这个本来是记在有道云笔记上面的，分享一下吧。

力扣在坚持每天打卡，然后看一些书/教程/文档和网课。偶尔打两把游戏。。感觉时间不够用呐。。

learning is keeping

#### 读取配置文件

conf在项目根目录下，关于go的路径读取还有很多问题，比如`go run`，和`go build`之后运行二进制文件的路径是不一样的。
```golang
data, err := ioutil.ReadFile("conf/app.yaml")
```

#### 读取yaml文件
这里map读出来的都是`interface{}`类型，所以我通过断言转了一下，不知道有没有问题。
```golang
data, err := ioutil.ReadFile("conf/app.yaml")
if err != nil {
	log.Fatalf("read config file app.yaml err: %v", err)
}
m := make(map[interface{}]interface{})
err = yaml.Unmarshal(data, m)
if err != nil {
	log.Fatalf("yaml.Unmarshal read error: %v ", err)
}
// 使用断言，将interface{}转为string
RunMode = m["RUN_MODE"].(string)
```
<!--more-->
#### goland使用filewatcher，goimports下载失败
1. 设置代理，打开设置->搜索proxy->找到Http Proxy->设置Manual proxy configuration
2. 检查连接 `check connection` (golang.org)
3. 在goland的控制台运行，` go get golang.org/x/tools/cmd/goimports
`
4. 再次进入filewatcher界面，添加`goimports`


#### go get命令的参数
- -d 只下载不安装
- -f 只有在你包含了 -u 参数的时候才有效，不让 -u 去验证 import 中的每一个都已经获取了，这对于本地 fork 的包特别有用
- -fix 在获取源码之后先运行 fix，然后再去做其他的事情
- -t 同时也下载需要为运行测试所需要的包
- -u 强制使用网络去更新包和它的依赖包
- -v 显示执行的命令


#### goland无法和vscode一样看提交历史和变更文件

##### 和vscode一样看每行代码的提交信息

下载插件`GitToolBox`即可，默认配置就支持，更多配置功能自己研究。

##### 和vscode一样看当前修改文件之类的。

打开设置->搜索`commit`，把`use non-modal commit interface`勾上->点击应用。你会看到左侧边栏出来了一个`commit`模块，点开就行。


#### golang提前声明返回值
代码中，`(count int)`在函数末端显示声明的返回值变量，可以在函数中直接使用，并且不用显示返回，直接写`return`就可以了
```golang
func GetTagTotal(maps interface{}) (count int) {
	db.Model(&Tag{}).Where(maps).Count(&count)
	return
}
```

#### gorm连接数据库报错
`unknown driver "mysql" (forgotten import?)`,字面意思就是没有引入`mysq`l包，按理说应该会自动引入的，但如果你没有直接使用这个包的方法，`gorm`包不会帮你引入这个包。坑啊
如果直接这样：
```golang
import{
    ...
    "github.com/go-sql-driver/mysql"
    ...
}
```
你使用了filewatcher里的go imports功能的话，一保存就会消失，所以要这样：
```golang
import{
    ...
    _ "github.com/go-sql-driver/mysql"
    ...
}
```

#### time.Duration

一定要记住time.Duration的默认单位是纳秒。
我被这个坑了一两个小时
```golang
func main() {
	router := routers.InitRouter()
	s := http.Server{
		Addr:           fmt.Sprintf(":%d", setting.HttpPort),
		Handler:        router,
		ReadTimeout:    setting.ReadTimeOut,
		WriteTimeout:   setting.WriteTimeOut,
		MaxHeaderBytes: 1 << 20,
	}
	err := s.ListenAndServe()
	if err != nil {
		log.Fatalf("server start err: %v", err)
	}
}
```
上面代码就是正常启动一个服务，我启动后一个调不通。
然后排查了很久发现是`ReadTimeout`和`WriteTimeout`的锅，我配置文件本意是配几千毫秒，但是`timeDuration`类型默认是纳秒，所以这个服务器的超时时间就是几千纳秒。。。


#### golang []int转string
在`js`可以通过`[1,2].join(",")`转换为`"1,2"`

但是`golang`的`strings.Join()`只能把字符串数组转为字符串。。

所以我去`stackoverflow`上面找了一个转换的方法：
```golang
func arrayToString(a []int, delim string) string {
	return strings.Trim(strings.Replace(fmt.Sprint(a), " ", delim, -1), "[]")
	//return strings.Trim(strings.Join(strings.Split(fmt.Sprint(a), " "), delim), "[]")
	//return strings.Trim(strings.Join(strings.Fields(fmt.Sprint(a)), delim), "[]")
}

```

#### gin框架编写全局错误处理中间件
教程里都是微信小程序那种接口返回方式，无论成功失败都返回`200`，然后用自己定义的一个`json`格式里的`code`区分状态。

但我还是喜欢`Restful Api`的方式，所以我就去研究了一下怎么全局捕捉异常，就和`Node.js`一样。

1. 先去掉`gin.Recover`这个中间件
2. 加上这样一个中间件，可能和百度搜到的gin的中间件写法不太一样，没有返回一个
`gin.HandleFunc`,我猜这和中间件的执行顺序有关吧，没有细看，抽时间研究一下。

```golang
func Recover(c *gin.Context) {
	defer func() {
		if r := recover(); r != nil {
			//打印错误堆栈信息
			if data, ok := r.(map[interface{}]interface{}); ok {
				code := data["code"].(int)
				msg := data["msg"].(string)
				log.Printf("panic: %v\n", r)
				//封装通用json返回
				c.JSON(code, gin.H{
					"msg": msg,
				})
			} else {
				log.Printf("panic: %v\n", r)
				// 走到这里说明是未知错误，打印一下堆栈信息
				debug.PrintStack()
				c.JSON(http.StatusInternalServerError, gin.H{
					"msg": "服务器发生未知错误，请通知管理员!",
				})
			}
		}
	}()
	//加载完 defer recover，继续后续接口调用
	c.Next()
}
```

3. 使用`panic(map[interface{}]interface{})`这样抛出错误，就可以被捕获到，当然数据结构你可以自己定义。
4. 然后里面有几个知识点，`r`是一个`map[interface{}]interface{}`,必须要使用断言才能从中读取值，参见[https://stackoverflow.com/questions/25214036/getting-invalid-operation-mymaptitle-type-interface-does-not-support-in](https://stackoverflow.com/questions/25214036/getting-invalid-operation-mymaptitle-type-interface-does-not-support-in)
5. 然后`code := data["code"].(int) msg := data["msg"].(string)` 这种写法和前面读取yaml文件时说的一样，`interface{}`的类型要用断言转一下。参见[https://stackoverflow.com/questions/18041334/convert-interface-to-int](https://stackoverflow.com/questions/18041334/convert-interface-to-int)
6. ps: 反正你用空接口的时候，取值就尝试用断言去转一下类型，不然肯定取不到的。

