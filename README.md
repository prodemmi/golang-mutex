# golang-mutex
How to work with go mutex

```go
package main

import (
  "fmt"
  "sync"
)

type Memory[T any] struct {
  Data []T
  Mutex sync.Mutex
}

func NewMemory[T any]() (*Memory[T], error) {
  mem := Memory[T]{
    Data: make([]T, 0),
  }
  return &mem, nil
}

func (mem *Memory[T]) Append(data T) {
  mem.Mutex.Lock()
  mem.Data = append(mem.Data, data)
  mem.Mutex.Unlock()
}
  
func (mem *Memory[T]) Display() {
  for _, item := range mem.Data {
    fmt.Println(item)
  }
}

func main() {
  memory, _ := NewMemory[string]()
  
  terminator := make(chan string, 0)
  
  runner := func(procId string, mem *Memory[string]) {
    defer func() {
    terminator <- ""
  }()

  for i := 0; i < 5; i++ {
    item := fmt.Sprintf("PID[%s]: %d", procId, i)
    mem.Append(item)
  }
}

go runner("1", memory)
go runner("2", memory)
go runner("3", memory)

j := 0

for {
  if j >= 3 {
    break
  }

  select {
    case _ = <-terminator:
    j++
  }
}

memory.Display()
```
