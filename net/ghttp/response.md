# 请求输出

数据输出对象定义如下：
```go
type Response struct {
    http.ResponseWriter
    bufmu   sync.RWMutex // 缓冲区互斥锁
    buffer  []byte       // 每个请求的返回数据缓冲区
    request *Request     // 关联的Request请求对象
}
```
可以看到```ghttp.Response```对象继承了标准库的```http.ResponseWriter```对象，因此完全可以使用标准库的方法来进行输出控制。

当然，建议使用```ghttp.Response```封装的方法来控制输出，```ghttp.Response```的数据输出使用```Write*```相关方法实现，并且数据输出采用了Buffer机制，数据的处理效率比较高，任何时候可以通过```OutputBuffer```方法输出缓冲区数据到客户端，并清空缓冲区数据。

相关方法（API详见： [godoc.org/github.com/johng-cn/gf/g/net/ghttp#Response](https://godoc.org/github.com/johng-cn/gf/g/net/ghttp)）：
```go
func (r *Response) Buffer() []byte
func (r *Response) SetBuffer(buffer []byte)
func (r *Response) ClearBuffer()
func (r *Response) OutputBuffer()

func (r *Response) Write(content...interface{})
func (r *Response) Writeln(content...interface{})
func (r *Response) WriteXml(content interface{}, rootTag ...string) error
func (r *Response) WriteJson(content interface{}) error
func (r *Response) WriteJsonP(content interface{}) error
func (r *Response) WriteStatus(code int, content ...string)

func (r *Response) SetAllowCrossDomainRequest(allowOrigin string, allowMethods string, maxAge ...int)
func (r *Response) RedirectTo(location string)
```
此外，需要提一下，Header的操作可以通过标准库的方法来实现，例如：
```go
r.Header().Set("Content-Type", "text/plain; charset=utf-8")
```