[[Go]]
[[Go canvas.canvas|Go canvas]]

## Пакеты

Каждый `go` файл принадлежит определённому пакету. Поэтому в самом начале файла должен идти либо комментарий и определение пакета, либо пакет:

```go
package main // <<---- 

import "fmt"

func main () {
	fmt.Println("Hello World!")
}
```

`package main` определяет точку входа в программу.

## Импорты

Импортировать можно, либо через `()`, либо подряд через `import`:

```go
import "fmt"
import "os"
import "math"
```

```go
import (
	"fmt"
	"os"
	"math"
)
```

Но `go fmt`
## Экспортирование имен

