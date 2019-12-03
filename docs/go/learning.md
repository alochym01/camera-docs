# Go Learning Path
## Go Fundamental
## 1. "Hello World" Basic type Variables, byte and rune and something funny in programing

### Basic type
bool

string

int int8 int16 int32 int64
Type 	Size 	Range
int8 	8 bits 	-128 to 127
int16 	16 bits 	-215 to 215 -1
int32 	32 bits 	-231 to 231 -1
int64 	64 bits 	-263 to 263 -1
int 	Platform dependent 	Platform dependent

uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune //alias for int32

float32 float64

complext64 complex128
### First go program

```go
package main
import "fmt"
func main() {
    fmt.Println("Hello, world!")
}
```
### Stack and Heap explain

```cmd
    Stack is used for static memory allocation and Heap for dynamic memory allocation, both stored in the computer's RAM.
    The stack is always reserved in a LIFO order, the most recently reserved block is always the next block to be freed. This makes it really simple to keep track of the stack, freeing a block from the stack is nothing more than adjusting one pointer.

    Variables allocated on the heap have their memory allocated at run time and accessing this memory is a bit slower, but the heap size is only limited by the size of virtual memory . Element of the heap have no dependencies with each other and can always be accessed randomly at any time. You can allocate a block at any time and free it at any time. This makes it much more complex to keep track of which parts of the heap are allocated or free at any given time.

```

### Variable declaretion defined

```go

package main

import "fmt"

func main() {
    //full declare and define
    var x string
    x = "Hello World"

    //declare and define in 1 line
    var y string = "Hello World"

    // combine 2 variable
    var i, j int = 1,2

    //Short variable declaration
    k := 3
    c, python, java := true, false, "no!"
    // However, outside a function, every statement begins with a keyword (var, func, and so on) and so the := construct is not available, and we'll get an error:

    fmt.Println(x, y, i, j, k, c, python, java)
    fmt.Printf("%T %T %T %T %T %T\n", i, j, k, c, python, java) //<-------------- %T type
}

```
### Global varibale 
First, one should discuss storage in terms of storage duration, the way C++ standard does: "stack" refers to automatic storage duration, while "heap" refers to dynamic storage duration. Both "stack" and "heap" are allocation strategies, commonly used to implement objects with their respective storage durations.
Global variables have static storage duration. They are stored in an area that is separate from both "heap" and "stack". Global constant objects are usually stored in "code" segment, while non-constant global objects are stored in the "data" segment. 

## 2. TDD vs BDD (Test Driven vs Behavior Driven)
Inline-style: 
![alt text](./tddvsbdd.png "Test Driven vs Behavior")

### Basic things enough to code meo cao chuot chay
* Function
```go
func add(x, y int) int {}
func (receiver) func_name(parameters) return_type{code}
```
* If else


3.. "Hello World with Test, Packages, Go Doc"

- Basic code structurec
export GOPATH= **Current directory**
export PATH=$PATH:$GOPATH/bin

```cmd
.
├── bin # which	holds compiled	binary	executable	programs.
├── pkg # which	holds compiled	binary	package	files.
└── src
```
go get -u github.com/go-delve/delve/cmd/dlv

go get -u github.com/golangci/golangci-lint/cmd/golangci-lint

* Go has an opinioned formatter called go fmt. Your editor should be running this on every file save.
```cmd
go fmt
```

- Convention packages name:
Why package?
```go
Different programs same function
Sharing	code between programs using packages
```

$GOPATH/src/github.com{source location(internal, github, gityeahspace)}/chivt{username, user-id}/hello{package name}

Package name file
```cmd
Go tools use the name in the import path as the name of the directory to load the package source code from. If they don't match, the code won't load.
```

### Writing basic test
```
Writing a test is just like writing a function, with a few rules
It needs to be in a file with a name like xxx_test.go
The test function must start with the word Test
The test function takes one argument only t *testing.T
For now it's enough to know that your t of type *testing.T is your "hook" into the testing framework so you can do things like t.Fail() when you want to fail.
```
### Go doc


4. "Variable, If else condition, Function"

3. "Arrays, Slices, Maps"

4. "Pointer, Struct, Types "

5. "Encapsulation, Method, Interfaces"

6. "Errors"

7. "Dependency Injection"

8. "Mocking"

9. "Concurrency"

10. "Select"

11. "Reflection"

12. "Sync"

13. "Context"

14. "Property based tests, Refactorism"

15. "Maths"

## Go Advanced
1. "HTTP server"
2. "JSON, routing, embedding"
3. "IO and sorting"
4. "Comandline & package structure"
5. "Time"
6. "WebSockets"
7. "Ginko" Framework
