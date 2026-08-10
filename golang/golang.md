# Golang学习

**Golang 学习**

官方中文文档：https://go-lang.org.cn/learn/
其他学习文档：https://golangstar.cn/
                           https://liwenzhou.com/

```go
【Go 指针传参完整精简总结】

一、核心铁律：什么时候必须传指针 *T
1. 自定义结构体 struct
想在函数内修改外部结构体字段 → 传指针
结构体很大，不想整体拷贝耗内存 → 传指针
小结构体、只读不修改，可值传递（工程上一般统一结构体传指针更规范）
2. sync.Mutex / RWMutex / WaitGroup / Once / Cond 同步结构体
只要作为函数参数传入，必须传 * 指针
原因：值传递会复制锁，变成两把独立锁，并发互斥失效、计数错乱
如果函数不接收锁参数，直接读写全局锁变量，不用指针，直接使用即可
3. 定长数组 [N]int
数组是完整值拷贝，要修改原数组内容，需要传指针
4. 基础类型 int/string/bool/float
默认值传递不加指针；只有不想 return、要就地修改外部原值时，才用指针。
5. slice /map/chan 特殊场景
正常读写内部元素，不用指针；
只有要整体替换这个变量本身（赋值全新切片 / 字典 / 通道），才需要 *[]int、*map[...]、*chan int

二、绝对不要加 * 的类型（高频踩坑）
所有接口类型
context.Context、error、io.Reader、io.Writer
接口自带间接引用，套 * 多余、不符合 Go 规范
✅ func f(ctx context.Context)
❌ func f(ctx *context.Context)
slice、map、chan（常规读写内部数据）
map、chan 变量本身就是底层结构指针，传参天然共享底层数据
slice 是头部结构体，内含底层数组指针，传参共用底层数组
三者修改内部内容外部都生效，无需额外加星
函数类型 func(...)...，直接传，不用指针

三、一句话快速判断口诀
要改外面原变量内容 / 避免大拷贝 → 传指针
sync 同步结构体传参必指针，直接用全局不用指针
interface 永远不加星
slice/map/chan 改内部不用星，整体替换才要星
int/string 默认不传指针，就地修改才用指针
```

```go
【Go 字符串拼接方案总结】
性能排序（从快 → 慢）
strings.Builder ≈ strings.Join > bytes.Buffer > []byte append 拼接 > + 加号拼接 > fmt.Sprintf

*零星几个字符串拼接 → + 最简单够用
*循环大批量拼接 → strings.Builder 首选
*字符串数组整体合并 → strings.Join
*字节与字符串混用 → bytes.Buffer
*格式化少量文本 → fmt.Sprintf
```

