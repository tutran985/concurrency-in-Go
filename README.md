# Concurrency in Go

## 1. Go's Concurrency Building Blocks
`Goroutines` là 1 trong những đơn vị tổ chức cơ bản nhất trong chương trình Go, vì vậy điều quan trọng chúng ta phải hiểu chúng là gì và các chúng hoạt động như thế nào. Trong thực tế trong mỗi Go program đều có ít nhất 1 goroutine: *the main goroutine*, nó được tạo tự động và bắt đầu khi quá trình bắt đầu. Trong hầu hết mọi chương trình bạn có thể sớm hoặc muộn tìm đến goroutine để hỗ trợ bạn giải quyết vấn đề. Vậy tóm lại chúng là gì?
