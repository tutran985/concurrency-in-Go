# Concurrency in Go

# 3. Go's Concurrency Building Blocks
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

Điều làm cho các goroutines trở nên độc đáo duy nhất đối với Go là sự tích hợp sâu bên trong của chúng với Go runtime. Goroutines không xác định điểm tạm dừng hay việc thử lại của chính nó; Go's runtime sẽ quan sát hành vi thời gian chạy của goroutines và tự động tạm dừng chúng khi chúng bị khoá và sau đó tiếp tục khi chúng được mở khoá, theo một cách nào đó đã làm chúng được xử lý trước, nhưng chỉ ở những chỗ mà goroutine đã bị khoá. Nó là một mối quan hệ hợp tác thanh lịch giữa runtime và logic của một goroutine. Như vậy, các goroutines có thể được xem xét như là một lớp đặc biệt của coroutine (a special class of coroutine).

Coroutines, và do đó goroutines, là cấu trúc hoàn toàn đồng thời (implicitly concurrent constructs), nhưng concurrency không phải là một thuộc tính của một coroutine: một thứ gì đó phải lưu trữ đồng thời nhiều coroutines và giữ cho mỗi cái một cơ hội để thực hiện nếu không thì, chúng sẽ không phải là concurrent! Lưu ý điều này không có nghĩa là các coroutines thực sự là parallel. Nó chắc chắn là có thể có một số các coroutines thực hiện tuần tự để tạo ra ảo giác về sự song song (the illusion of parallelim) và trong thực tế, điều này xảy ra mọi lúc ở trong Go.

Cơ chế của Go để lưu trữ các goroutines là một việc triển khai cái được gọi là bộ lập lịch M:N. Có nghĩa là nó ánh xạ M green threads to N OS threads. Goroutines sau đó được đặt lịch trên các green threads. Khi chúng ta có nhiều goroutines hơn green threads đang sẵn có, thì bộ lập lịch (the scheduler) sẽ xử lý phân phối các goroutines trên các threads sẵn có và đảm bảo rằng khi các goroutines này bị chặn, thì các goroutines khác sẽ có thể chạy. Chúng ta sẽ thảo luận về tất cả các cái này hoạt động ở chương tiếp theo, nhưng ở đây chúng ta sẽ đề cập đến mô hình Go concurrency (Go models concurrency).

Go theo một mô hình của concurrency có tên là ***fork-join*** model. Từ ***fork*** trên thực tế đề cập đến là tại bất kì thời điểm nào trong chương trình, nó có thể tách ra thành một nhánh con để chạy đồng thời với cha của nó. Từ ***join*** đề cập đến là vào một vài thời điểm nào đó trong tương lai, các nhánh đã tách ra chạy đồng thời với cha của nó sẽ kết hợp lại với nhau. Khi đó những đứa nhánh con sẽ tham gia lại cùng với cha của nó và được gọi là một ***join point***. Dưới đây là một biểu đồ hoạ giúp bạn có thể hình dung nó:

<p align="center">
<img style="display: block;margin: 0 auto;width: 50%;" alt="Screen Shot 2022-07-29 at 00 45 45" src="https://user-images.githubusercontent.com/32538318/181603869-d24d8de2-4a8c-4426-9bad-6b07194d351b.png">
</p>

Câu lệnh Go là cách Go biểu diễn một fork, và các forker threads thực thi là các goroutines. Hãy quay lại về ví dụ goroutine đơn giản của chúng tôi:

```go
    sayHello := func() {
        fmt.Println("hello")
    }
    go sayHello()
    // continue doing other things
```
Ở đây, func sayHello sẽ được chạy trên goroutine của riêng nó, trong khi phần còn lại của trương chình tiếp tục thực hiện. Trong ví dụ này, nó không phải là join point. Goroutine thực thi sayHello sẽ thoát ở một thời điểm không xác định nào đó trong tương lai, và phần còn lại của trương chình sẽ tiếp tục được thực thi.

Tuy nhiên, có một vấn đề với ví dụ này; như đã viết, nó sẽ không xác định được liệu func sayHello có được chạy hay không. Goroutine sẽ được tạo và được lên lịch với Go's runtime được thực thi, nhưng nó có thể không thực sự có cơ hội được thực thi khi quy trình chính thoát ra.

Thật vậy, bởi vì chúng tôi đã bỏ qua phần còn lại của phần còn lại của hàm chính (main function) để đơn giản hoá ví dụ, khi chúng tôi chạy ví dụ nhỏ này, nó gần như chắc chắn rằng hầu hết chương trình sẽ hoàn thành quá trình thực thi trước khi goroutine lưu trữ lệnh gọi hàm sayHello được bắt đầu. Kết quả là bạn sẽ không thấy được chữ "hello" được in ra. Bạn có thể đặt time.Sleep trước khi bạn khởi tạo goroutine, nhưng hãy nhớ rằng điều này không thực sự tạo ra một join point, nó chỉ là một "race condition". Join point là yếu tố đảm bảo tính đúng đắn của chương trình của chúng tôi và loại bỏ các race condition.

Để tạo một join point, bạn có làm cho luồng chính (main goroutine) và sayHello goroutine đồng bộ hoá với nhau. Điều này có thể thực hiện trong một vài cách, nhưng tôi sẽ sử dụng một cách mà chúng tôi sẽ nói về nó trong `"The Sync Package"`: **sync.WaitGroup**. Ngay bây giờ bạn không cần thực sự hiểu cái cách tạo ra một join point, đây là một phiên bản chính xác của ví dụ của chúng tôi:

```go
    var wg sync.WaitGroup 
    sayHello := func() {
        defer wg.Done()
        fmt.Println("hello")
    }  
    wg.Add(1)
    go sayHello()
    wg.Wait() // => đây chính là join point
```
Kết quả trả ra:
`hello`

Ví dụ này sẽ chặn một cách rõ ràng goroutine chính cho đến khi goroutine chứa function sayHello chạy xong. Bạn sẽ tìm hiểu cách sync.WaiGroup hoạt động như thế nào trong `"The Sync Package"`, nhưng để làm cho ví dụ của chúng tôi chính xác. Tôi sẽ bắt đầu sử dụng nó để tạo **join point**.

Chúng tôi đã và đang sử dụng rất nhiều những hàm ẩn danh (anonymous functions) trong ví dụ của chúng tôi để tạo ra những ví dụ đơn giả về goroutine. Ở đây chúng ta có một ví dụ:

```go
    var wg sync.WaitGroup
    salutation := "hello"
    wg.Add(1)
    go func() {
        defer wg.Done()
        salutation = "welcome" // ở đây chúng ta thấy goroutine sửa đổi giá trị của biến salutation
    }()
    wg.Wait()
    fmt.Println(salutation)
```

Bạn nghĩ rằng giá trị của biến salutation sẽ được thay đổi thành gì là "hello" hay là "welcome"? Hãy chạy nó là tìm ra đáp án `welcome`

Thú vị! Nó chỉ ra rằng các goroutines thực thi trong cùng một không gian địa chỉ mà chúng đã được tạo ra, và
chương trình của chúng ta đã in ra từ "welcome". Hãy thử lại với ví dụ khác. Bạn nghĩ rằng kết quả trả ra của chương trình này là gì?
```go 
    var wg sync.WaitGroup
    for _, salutation := range []string{"hello", "greetings", "good day"} {
        wg.Add(1)
        go func() {
            defer wg.Done()
            fmt.Println(salutation)
        }()
    }
    wg.Wait()
```

Đáp án có thể khó hơn mọi người mong đợi, và nó cũng là một trong vài điều đáng ngạc nhiên của Go. Hầu hết mọi người nghĩ rằng kết quả sẽ là "hello", "greetings", "good day" với những vị trí không xác định, nhưng kết quả lại là:
```txt
good day
good day
good day
```

Điều đó thật ngạc nhiên! Hãy tìm hiểu xem điều gì đang xảy ra ở đây. Trong ví dụ này, ***bỏ qua chán chả muốn viết***


## 3.1. The Sync Package
The Sync Package chứa các concurrency primitives hữu ích nhất để đồng bộ hoá truy cập bộ nhớ cấp thấp. Nếu bạn đã từng làm việc bằng các ngôn ngữ chủ yếu xử lý đồng thời thông qua đồng bộ hoá quyền truy cập bộ nhớ, những loại này có thể đã quen thuộc với bạn. Chúng ta hãy đi xem các primitives khác nhau mà gói thư viện sync đưa ra:


###### 3.1.1. WaitGroup
`WaitGroup` là một cách tốt để chờ đợi cho một tập hợp các hoạt động đồng thời cho đến khi hoàn thành khi bạn không quan tâm đến kết quả của hoạt động đồng thời, hay bạn có những phương tiện khác để thu thập kết quá của họ. Nếu cả hai điểu kiện đó đều đúng, tôi đề xuất bạn sử dụng channels và một select để thay thế. WaitGroup rất hữu ích, tôi giới thiệu nó đầu tiên bởi vì tôi có thể sử dụng nó trong tất cả phần tiếp theo. Đây là một ví dụ cơ bản sử dụng waitGroup để chời đợi các goroutines hoàn thành:
```go
    var wg sync.WaitGroup
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println("1st goroutine sleeping...")
        time.Sleep(1)
    }()
    
    wg.Add(1)
    go func() {
        defer wg.Done()
        fmt.Println("2nd goroutine sleeping...")
        time.Sleep(2)
    }()
    wg.Wait()
    fmt.Println("All goroutines complete.")
```
This produces: 
```txt
2nd goroutine sleeping...
1st goroutine sleeping...
All goroutines complete.
```

exp2:
```go
    hello := func(wg *sync.WaitGroup, id int) {
        defer wg.Done()
        fmt.Printf("Hello from %v!\n", id)
    }
    const numGreeters = 5
    var wg sync.WaitGroup
    wg.Add(numGreeters)
    for i := 0; i < numGreeters; i++ {
        go hello(&wg, i+1)
    }
    wg.Wait()
```
This produces:
```txt
Hello from 5!
Hello from 4!
Hello from 3!
Hello from 2!
Hello from 1!
```
###### 3.1.2. Mutex and RWMutex


// cond

Lưu ý rằng call Wait

Như bạn có thể thấy, chương trình đã thêm thành công 10 items vào queue (và thoát trước khi nó thay đổi dequeue the last two items). Nó luôn luôn chờ cho đến khi ít nhất một item là dequeued trước enqueing khác.

Chúng tôi cũng có 1 phương pháp mới trong ví dụ này, Signal. Đó là một trong hai phương thức mà Cond cung cấp để thông báo các goroutines đã được blocked trong Wait rằng condition đã được kích hoạt. Phương thức khác này có tên là Broadcast. Thời gian chạy duy trì một danh sách goroutines đang chờ đợi được báo hiệu; Signal tìm kiếm goroutine chờ đợi lâu nhất và thông báo rằng, trong khi đó Broadcast gửi tín hiệu cho tất cả các goroutines còn lại đang chờ đợi. Broadcast được cho là thú vị hơn nhiều trong hai phương thức vì nó cung cấp một cách để có thể giao tiếp với nhiều goroutines cùng lúc. Chúng tôi có thể tái tạo Signal một cách đáng kinh ngạc với Channel (chúng ta có thể xem trong phần "Channels"), nhưng việc tái tạo hành vi của các cuộc gọi lặp đi lặp lại tới Broadcast sẽ khó hơn nhiều. Ngoài ra, Cond có hiệu suất cao hơn nhiều so với việc sử dụng channels.

Để có cảm nhận về việc sử dụng Broadcast như thế nào, Hãy thử tượng tượng chúng ta đang tạo ra một ứng dụng GUI với một nút button ở trên đó. Chúng tôi muốn đăng ký một vài chức năng bất kì sẽ được chạy khi nút button được bấm. Cond hoàn hảo cho điều đó bởi vì chúng tôi có thể dùng tới Broadcast để thông báo tới tất cả các registed handlers. Nào hãy xem nó trông như thế nào:

```go
    type Button struct { // xác định 1 loại Button có chứa 1 condition, Clicked
		Clicked *sync.Cond
	}
	button := Button{Clicked: sync.NewCond(&sync.Mutex{})}
	subscribe := func(c *sync.Cond, fn func()) { // 2
		var goroutineRunning sync.WaitGroup
		goroutineRunning.Add(1)
		go func() {
			goroutineRunning.Done()
			c.L.Lock()
			defer c.L.Unlock()
			c.Wait()
			fn
            fn()

		}()
		goroutineRunning.Wait()
	}
	var clickRegistered sync.WaitGroup // 3
	clickRegistered.Add(3)
	subscribe(button.Clicked, func() { // 4 
		fmt.Println("Maximizing window.")
		clickRegistered.Done()
	})
	subscribe(button.Clicked, func() { // 5
		fmt.Println("Displaying annoying dialog box!")
		clickRegistered.Done()
	})
	subscribe(button.Clicked, func() { // 6
		fmt.Println("Mouse clicked.")
		clickRegistered.Done()
	})
	button.Clicked.Broadcast() // 7
	clickRegistered.Wait()

    // 2: chúng tôi xác định 1 convenience func nó sẽ cho phép chúng tôi đăng ký func để xử lý tín hiệu từ 1 condition. Mỗi handler sẽ chạy trên goroutine của riêng nó, và subscribe sẽ không thoát cho đến khi gouroutine đó được xác nhận là đang chạy.
    // 3: Chúng tôi tạo ra một WaitGroup. Điều này chỉ được thực hiện để đảm bảo chương trình của chúng tôi không thoát ra trước khi quá trình ghi vào stdout của chúng tôi xảy ra
    // 4: chúng tôi đăng ký 1 handler để mô phỏng tối đa hoá cửa sổ button khi được clicked
    // 5: Chúng tôi đăng ký 1 handler để mô phỏng hiển thị 1 hộp thoại khi clicked
    // 6: Tieesp theo, chúng tôi mô phỏng người dùng di chuyển chuột sau khi clicked vào application's button
    // 7: Gọi Broadcast trên cliecked Cond để cho tất cả các trình xử lý biết rằng đã được nhấp
```
This produces:
```txt
    Mouse clicked.
    Maximizing window.
    Displaying annoying dialog box!
```

Bạn có thể thấy rằng với một lệnh gọi đến Broadcast trên Clicked Cond, tất cả 3 trình xử lý đã được chạy, nó không dành cho clickRegistered WaitGroup, chúng tôi có thể gọi button.Clicked.Broadcast() nhiều lần, và mỗi lần tất cả 3 handlers đều sẽ được gọi. Đây là điều mà các channel không thể làm dễ dàng và do đó là một trong những lý do chính để sử dụng Cond.

Giống như những thứ khác trong sync package, cách sử dụng của Cond hoạt động thực sự tốt nhất khi bị bó buộc trong 1 phạm vi hẹp
