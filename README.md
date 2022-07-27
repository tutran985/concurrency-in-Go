# Concurrency in Go

## 1. Go's Concurrency Building Blocks
`Goroutines` là một trong những đơn vị tổ chức cơ bản nhất trong chương trình Go, vì vậy điều quan trọng chúng ta phải hiểu chúng là gì và các chúng hoạt động như thế nào. Trong thực tế trong mỗi Go program đều có ít nhất một goroutine: *the main goroutine*, nó được tạo tự động và bắt đầu khi quá trình bắt đầu. Trong hầu hết mọi chương trình bạn có thể sớm hoặc muộn tìm đến goroutine để hỗ trợ bạn giải quyết vấn đề. Vậy tóm lại chúng là gì?

Nói một cách đơn giản, một goroutine là một function đang chạy đồng thời cùng với code khác (hãy nhớ rằng: *not necessarily in parallesl!*). Bạn có thể bắt đầu một cách đơn giản bằng cách đặt từ khoá **go** trước function:
```go
  func main() {
    go sayHello()
    // continue doing other things
  }
  func sayHello() {
    fmt.Println("hello")
  }
```
Function ẩn danh cũng sẽ hoạt động! đây là một ví dụ thực hiện điều tương tự như ở ví dụ trên, tuy nhiên, thay vì tạo một goroutine từ một function, chúng tôi sẽ tạo từ một function ẩn danh:
```go
  go function() {
    fmt.Println("hello")
  }() // lưu ý rằng ở đây chúng ta phải gọi func ngay lập tức để sử dụng được từ khoá go
  // continue doing other things
```
Ngoài ra, bạn cũng có thể gán một func vào một biến và gọi chúng như thế này:
```go
  sayHello := func() {
    fmt.Println("hello")
  }
  go sayHello()
  // continue doing other things
```
Trông có vẻ tuyệt! Chúng ta có thể tạo ra một khối logic concurrent (concurrent block of logic) với một function và một từ khoá đơn giản. Tin nổi không, nó là tất cả những gì bạn cần biết để có thể tạo ra và bắt đầu với goroutines. Có rất nhiều điều để nói về cách sử dụng chúng đúng cách, đồng bộ và sắp xếp chúng, nhưng đây là tất cả những gì bạn cần biết để bắt đầu sử dụng chúng. Tiếp theo đây chúng ta sẽ đi sâu hơn về goroutines và tìm hiểu về cách chúng hoạt động. Nếu bạn chỉ quan tâm đến cách viết code để nó hoạt động bình thường với goroutines, bạn có thể cân nhắc đến việc dừng lại.


