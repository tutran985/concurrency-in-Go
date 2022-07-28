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

Vì vậy hãy xem xét xem điều gì đang xảy ra ở phía sau tại đây: Làm thế nào để goroutines thực sự hoạt động? Có phải là OS threads không? Green threads? Và chúng ta có thể tạo ra bao nhiêu?

Goroutines là duy nhất của Go mặc dù có một số ngôn ngữ khác cũng có concurrency primitive tương tự như vậy. Chúng không phải là OS threads, và cũng không chính xác là green threads - được quản lý bởi thời gian chạy của một ngôn ngữ (language’s runtime), chúng là một cấp độ trừu tượng cao hơn được biết đến là ***coroutines***. Coroutines chỉ đơn giản là các chương trình con chạy đồng thời với nhau không mang tính ưu tiên (concurrent subroutines), điều đó có nghĩa là chúng không thể bị gián đoạn.

Điều làm cho các goroutines trở nên độc đáo duy nhất đối với Go là sự tích hợp sâu bên trong của chúng với Go runtime. Goroutines không xác địng điểm tạm dừng hay việc thử lại của chính nó; Go's runtime sẽ quan sát hành vi thời gian chạy của goroutines và tự động tạm dừng chúng khi chúng bị khoá và sau đó tiếp tục khi chúng được mở khoá, theo một cách nào đó đã làm chúng được xử lý trước, nhưng chỉ ở những chỗ mà goroutine đã bị khoá. Nó là một mối quan hệ hợp tác thanh lịch giữa runtime và logic của một goroutine. Như vậy, các goroutines có thể được xem xét như là một lớp đặc biệt của coroutine (a special class of coroutine).

Coroutines, và do đó goroutines, là cấu trúc hoàn toàn đồng thời (implicitly concurrent constructs), nhưng concurrency không phải là một thuộc tính của một coroutine: một thứ gì đó phải lưu trữ đồng thời nhiều coroutines và giữ cho mỗi cái một cơ hội để thực hiện nếu không thì, chúng sẽ không phải là concurrent! Lưu ý điều này không có nghĩa là các coroutines thực sự là parallel. Nó chắc chắn là có thể có một số các coroutines thực hiện tuần tự để tạo ra ảo giác về sự song song (the illusion of parallelim) và trong thực tế, điều này xảy ra mọi lúc ở trong Go.

Cơ chế của Go để lưu trữ các goroutines là một việc triển khai cái được gọi là bộ lập lịch M:N. Có nghĩa là nó ánh xạ M green threads to N OS threads. Goroutines sau đó được đặt lịch trên các green threads. Khi chúng ta có nhiều goroutines hơn green threads đang sẵn có, thì bộ lập lịch (the scheduler) sẽ xử lý phân phối các goroutines trên các threads sẵn có và đảm bảo rằng khi các goroutines này bị chặn, thì các goroutines khác sẽ có thể chạy. Chúng ta sẽ thảo luận về tất cả các cái này hoạt động ở chương tiếp theo, nhưng ở đây chúng ta sẽ đề cập đến mô hình Go concurrency (Go models concurrency).

Go theo một mô hình của concurrency có tên là ***fork-join*** model. Từ ***fork*** trên thực tế đề cập đến là tại bất kì thời điểm nào trong chương trình, nó có thể tách ra thành một nhánh con để chạy đồng thời với cha của nó. Từ ***join*** đề cập đến là vào một vài thời điểm nào đó trong tương lai, các nhánh đã tách ra chạy đồng thời với cha của nó sẽ kết hợp lại với nhau. Khi đó những đứa nhánh con sẽ tham gia lại cùng với cha của nó và được gọi là một ***join point***. Dưới đây là một biểu đồ hoạ giúp bạn có thể hình dung nó:
Screen Shot 2022-07-29 at 00.25.12.png