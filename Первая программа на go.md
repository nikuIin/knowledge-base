[[Go canvas.canvas|Go canvas]]
[Первая программа на go (Metanit)](https://metanit.com/go/tutorial/1.2.php)

Назовем файл `hello.go`

```go
package main // package main должен быть в файле, являющимся точкой входа

import "fmt"

func main() { // ну, а это уже конкретная точка входа

    fmt.Println("Hello METANIT.COM!")

}
```

- Для выполнения нужно запустить:
```shell
go run hello.go
```

- Для билда проекта:

```shell
go build hello.go

./hello # запускаем бинарник
```

