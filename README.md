# Go

## Primeros pasos

Go requiere de cierta organización, no es como los demás lenguajes que nos dan libertad sobre donde poner nuestro código.
Hay que tener ciertos cuidados a la hora de estructurar nuestra app.

### Instalación

https://golang.org/doc/install

Las guías de instalación para las distintas plataformas nos dicen claramente que debemos tener

- .../bin/go en el PATH
- $HOME/go es el directorio default para todo nuestro código fuente
- Podemos cambiarlo definiendo la variable de entorno GOPATH
- Conviene incluir $GOPATH/bin al path

Para crear un programa "hello world" por ejemplo necesitamos paramos en GOPATH/src
creamos la carpeta hello con el archivo hello.go

```go
// Los programas principales van dentro del paquete main
package main

// usamos import para definir librerías de terceros
import "fmt"

// La función main es la función que determina que este es un ejecutable
func main() {
  // fmt nos permite imprimir texto en pantalla
  fmt.Printf("hello, world\n")
}
```


```bash
go build
./hello
```

```bash
go run hello.go
./hello
```

### Organización de código Go

- Generalmente mantenemos un solo workspace GOPATH
- Dentro de ese workspace se incluyen muchos repositorios (git por ejemplo)
- Dentro de cada repositorio organizamos nuestro código en paquetes.
- Los paquetes son directorios, el nombre del paquete es el nombre del ultimo directorio
- Normalmente no son muchos directorios, es una estructura bastante plana

### El Workspace

El workspace esta donde definimos GOPATH, y contiene los siguientes directorios:

- src con código fuente, donde están nuestros repositorios git descargados
- bin con los binarios generado por go

El comando go busca los fuentes en GOPATH/src.
Solo podemos tener un workspace activo en cada momento, por lo que se suele usar un solo repositorio.

Aunque no queramos publicar en un repositorio nuestros programas de ejemplo, es buena practica generar esta estructura de directorios para organizarnos mejor, por ejemplo el programa hello.go,  debería estar en una carpeta similar a GOPATH/src/github.com/user/test/hello (user es nuestro usuario git)

``` bash
go install github.com/user/test/hello
```

Este comando lo podemos ejecutar desde cualquier directorio, go busca en el workspace el código fuente y lo pone en el directorio bin del workspace

### Imports

Supongamos la siguiente librería en 
GOPATH/src/github.com/user/test/stringutil/reverse.go

``` go
// El paquete es el nombre del directorio
package stringutil

// invierte una cadena
func Reverse(s string) string {
  r := []rune(s)
  for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
    r[i], r[j] = r[j], r[i]
  }
  return string(r)
}
```

Ahora modifiquemos nuestro hello 
$GOPATH/src/github.com/user/test/hello):

``` go
package main

// Import nos permite incluir librerías en nuestro código
// Cuando debemos incluir varias librerías usamos un bloque import
import (
  "fmt"  // Las librerías de go no necesitan repositorio

  "github.com/user/test/stringutil"  // aca estaríamos incluyendo nuestra lib
)

func main() {
  fmt.Println(stringutil.Reverse("!oG ,olleH"))
}
```
#### Format

Format permite imprimir resultados tanto en pantalla como en un stream.
Permite datos con formato, por lo que recomiendo leer la documentación antes de usarlo.

fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))

### Los nombres de los paquetes

- Lo primero que debemos definir en el código es el nombre del paquete
- La convención es que el nombre del paquete tenga el mismo nombre que el ultimo directorio, podría ser diferente
- Dentro de una carpeta todos los archivos deben tener el mismo nombre de paquete
- Los archivos ejecutables (declaran la función main) tienen que estar dentro del paquete main

### Como descargamos librerías de terceros

Todas las librerías las vamos a encontrar en algún repositorio. En general se usa git.

Supongamos que buscamos alguna librería interesante de algún tercero como puede ser github.com/gin-gonic/gin que es un servidor web.

``` bash
go get github.com/golang/example/hello
```

esto nos va a descargar ese repositorio en la carpeta GOPATH/src

### Dep

El problema que tiene el esquema anterior es que es difícil controlar que versiones de librerías queremos usar en nuestro proyecto.
Hay algunos proyectos para mejorar esto, pero por el momento podemos usar la herramienta dep

Go saco un manejador de paquetes go modules

https://blog.golang.org/using-go-modules

mod nos permite definir que versiones de librerías queremos usar, modifica la forma en que se compilan las aplicaciones.

mod utiliza 2 archivos go.mod y go.sum, estos archivos son los que tienen la definición de las librerías usadas y que version.

Las librerías en vez de ser descargadas dentro de GOPATH/src son descargadas en el directorio vendor dentro de nuestro proyecto.


En general go trabaja bastante bien con mod, pero a veces cuando usamos librerias que no esan preparadas o suficientemente maduras conviene usar mod vendoring, para crear copias locales de las libs en la carpeta vendor

### El lenguaje de programación GO

Go es un lenguaje imperativo, no es funcional ni orientado a objetos.
No debemos intentar compararlo con ningún otro lenguaje. Es lo que es.

#### Comentarios

Podemos usar

- bloques  /* */
- lineas //
- Los paquetes deben estar documentados
- Los elementos que se exportan de un paquete deben documentarse
- go proporciona godoc una tool para extraer y documentar. Visual Studio Code la utiliza automáticamente para autocompletar

#### Nombres

- Paquetes
  - se nombran igual que el ultimo directorio
  - todos dentro del directorio deben tener el mismo nombre
  - cuando importamos un paquete _import "bytes"_ podremos usar sus contenidos exportados con el nombre que le dimos _bytes.Buffer_
  - Para el caso de _import "encoding/base64"_ el nombre es _base64_
  - El nombre del paquete debe ser corto
  - Por convención se escriben en minúsculas, una sola palabra
  - Si importamos  2 librerías con el mismo nombre de paquete debemos renombrar una en el import _import base64B "other/base64"_

- Declaraciones

Dentro de un archivo .go las declaraciones que comienzan con mayúsculas significan que son publicas, o sea que son elementos que se exportan desde este paquete y pueden ser leidos fuera del paquete.

Todos los archivos .go dentro de un paquete, go los interpreta como el mismo archivo, por lo tanto podemos acceder a elementos privados dentro del mismo paquete sin problemas.

Los nombres de interfaces y tipos de datos se definen en camel case. Deben ser lo mas corto y preciso posible.

#### Punto y coma

Go podría usar ; para terminar las sentencias, pero no se escriben. Aunque en ciertos escenarios puede que las necesitemos.


#### Variables

Podemos declarar una variable sin asignación de valor, solo con el tipo
``` go
var age int
```

Podemos declararla y asignarle un valor
``` go
var age int = 29
```

O directamente dejar que infiera el tipo de dato
``` go
var age = 29

var width, height int = 100, 50
var width, height = 100, 50
```

Lo siguiente se llama short hand declaration y es lo mas usado
``` go
name := "naveen"
name, age := "naveen", 29
```


Si la variable ya existe no debemos usar :=, debemos usar solamente = para redefinir su valor.
En el ejemplo anterior, la segunda vez vez que usamos := es posible usarlo porque d no existe.

El alcance de las variables es muy intuitivo, se acceden dentro del bloque que se definen, y solo se pueden acceder una vez definidas.
##### Tipos de datos

bool
string
int int8 int16 int32 int64
uint uint8 ...
byte
rune  // Representa un caracter unicode int32
float32 y float64
complex64 complex128

#### Estructuras de control

##### If

``` go
if x > 0 {
    return y
}
```

No necesitamos () en la expresión.
Las llaves siempre se abren al final de la sentencia.

If puede tener un bloque de initialization de variables

``` go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

##### For

For permite los 3 bloques básicos

``` go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

Podemos usar , para separar distintas operaciones dentro de un bloque
``` go
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    ...
}
```

Pero también se puede usar como bloque while
``` go
i := 0
for i<10 {
    i += 1
}
```

Podemos usar para hacer un bucle infinito, debemos salir con break para salir
``` go
for {
    break
}
```

Podemos usarlo para navegar un arreglo o un mapa de valores
``` go
for key, value := range mapaDeDatos {
    otroMapa[key] = value
}
```

En el caso de los mapas, si solo necesitamos el indice, podemos omitir el valor
``` go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

El _ sirve para omitir algún valor que necesitemos definir, pero no usar.
``` go
sum := 0
for _, value := range array {
    sum += value
}
```

##### If con definicion de datos

La variable inicializada solo puede accederse dentro del bloque if, y en la expresión solo luego de definirse.

Es buena practica en go terminar la secuencia de ejecución de una función cuando se detectan errores, como en el ejemplo anterior.

``` go
// En go las funciones pueden retornar mas de un valor, en este caso un puntero al archivo y un código de error en caso de que existan errores
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
```

##### Switch

``` go

//Switch es bastante genérico, podemos analizar un valor
switch c {
case ' ', '?', '&', '=', '#', '+', '%':
    return true
}
return false

// o podemos usar expresiones, la primera que se cumpla se ejecuta
switch {
case '0' <= c && c <= '9':
    return c - '0'
case 'a' <= c && c <= 'f':
    return c - 'a' + 10
case 'A' <= c && c <= 'F':
    return c - 'A' + 10
default:
    return 0
}

// No necesitamos break para salir de una condición, pero si podríamos usarlo para salir
switch {
case n < sizeOne:
    if validateOnly {
      break
    }
    size = 1
    update(src[n])

case n < sizeTwo:
    ...
}
```

### Funciones

``` go
// Las funciones en go pueden retornar muchos valores.
// Es muy común usar esta capacidad para retornar errores.

func (file *File) Write(b []byte) (n int, err error)

// Por ejemplo una función que devuelve un numero +1 y +2
func nextInt(i int) (int, int) {
    return i+1, i+2
}

// Podemos darle un nombre a las variables de retorno, esto hace mas clara su intención y fácil documentación
func nextInt(i int) (plus1 int, plus2 int) {
    return i+1, i+2
}

// Podemos omitir el tipo de datos, si es obvio, pero incluirlo ayuda a comprender mejor la función
func nextInt(i int) (plus1, plus2) {
    return i+1, i+2
}

```

#### Defer

La llamada que definamos con defer hace que se ejecute cuando salimos del bloque de control.

``` go
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close se ejecuta cuando salimos de la función, sin importar cuando salgamos.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err
        }
        result = append(result, buf[:n]...) // append is discussed later.
    }
    return string(result), nil
}
```

``` go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
// son llamadas diferentes que se encolan como LIFO, lo que nos daría la salida 4 3 2 1 0

```

#### Punteros

Go maneja punteros, a diferencia de C los punteros en go son tipados.

``` go
i := 1          // int
p := &i         // puntero a i
fmt.Println(*p) // lee i a partir de su puntero
*p = 21         // establece un valor a i con el puntero
```

#### Estructuras de datos

Go se maneja con estructuras de datos.

``` go
type Vertex struct {
  X int
  Y int
}

// Simple definicion de una variable nos hace una instancia nueva
var datoV Vertex

// podemos proveer los valores de inicialización
v := Vertex{1, 2}
// o algunos, el resto se inicializa con 0
v = Vertex{Y : 2}
v.X = 1

// p es del tipo *Vertex
p := &v
p.X = 2

// Podemos usar el operador __new__ para crear variables de estos tipos de datos, new retorna punteros

// var nos permite definir una variable sin asignarle un valor
var p *Vertex
p = new(Vertex)  // *Vertex
p = &Vertex{X:1}
p = nil

d := new(Vertex)  //  *Vertex

v := *d  // Vertex, pero cuidado porque al ser una estructura la asignacion genera una copia del dato, y no es el dato mismo

c := v

```

La ventaja de usar punteros es la posibilidad de pasar estructuras por referencia en los parámetros y resultados de las funciones.
Go se maneja por valor, si pasamos una estructura nos pasa un copia de la misma, no la estructura original. Si queremos pasar por referencia, debemos usar punteros.
Ademas, con punteros podemos analizar la posibilidad de retornar nil.
Las variables de algún tipo de estructura no pueden ser nil, pero los punteros a esas estructuras si.

#### Arrays

Los arreglos son Valores, no punteros, si se pasan como parámetros, se pasan copias del arreglo, no su dirección.
Si agregamos valores a un array recibimos una copia del array.
El tamaño del arreglo es parte del tipo. tipo de dato de un arreglo de 10 elementos es distinto al del 20.

```go

var a [5]int
a[4] = 100
fmt.Println("len:", len(a))

var twoD [2][3]int

// el tamaño puede evitarse si se definen los elementos
array := [...]float64{7.0, 8.5, 9.1}

// las funciones deben recibir su puntero, para que sea por referencia.
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

x := Sum(&array)  // Para pasar el puntero necesitamos obtener la dirección

```

__make__

Make sirve para inicializar arrays de valores. Los valores dentro del array

```
var v  []int = make([]int, 100) // inicializa un arreglo de 100 int
var p *[]int = new([5]int)       // es un arreglo de punteros a int, los punteros están en nil
```

#### Slices

Los slices nos permiten obtener subarrarys de un array, sin crear un array nuevo

```go
primes := [6]int{2, 3, 5, 7, 11, 13}

// s es un slice de primes que toma los valores de 1 a 4. No es una copia, sino un subconjunto.
var s []int = primes[1:4]
```

Internamente es un arreglo de punteros a arreglos.

Lo que nos permite expandir los slices agregando mas arreglos de datos, sin necesidad de copiar el arreglo completo

El ejemplo mas común seria

```go
// A diferencia de un arreglo un slice se identifica con [] vacío
var s []int

// append nos permite agregar un elemento al arreglo, no es muy eficiente de a 1
s = append(s, 0)

// Pero podemos agregar varios, o un array, en cuyo caso son interpretados como un array, y es donde comienza a tener mas sentido el slice
s = append(s, 2, 3, 4)

```

```go

// Los parametros dinamicos son slices
// func sum(nums []int) {

func sum(nums ...int) {
    fmt.Print(nums, " ")
    total := 0
    for _, num := range nums {
        total += num
    }
    fmt.Println(total)
}
```

#### Maps

``` go
// Un puntero a un mapa se define como
var m map[string]int
// m es un puntero a un mapa de enteros con una clave string, que no apunta a ningun mapa, es un puntero apuntando a nil

// Ahora inicializamos un mapa y lo asignamos a m. Y a partir de aca podemos usarlo
m = make(map[string]int)
m["route"] = 66

// se definen de la siguiente forma [clave]valor
var timeZone = map[string]int{
    "UTC":  0,
    "EST": -5,
    "CST": -6,
    "MST": -7,
    "PST": -8,
}

// podemos agregar nuevos
timeZone["MDZ"] = -3

// se acceden con su indice
offset := timeZone["EST"]

// podemos borrar
delete(timeZone, "PDT")

// Si queremos leer un valor que no se encuentra en el mapa, nos va a retornar el valor '0'
// por eso la función para leer un valor devuelve 2 valores, no uno

if val, ok := timeZone["UNK"]; ok {
    // si ok es true es porque se encontró
}
```

#### Constantes

Las constantes en go se definen con const.

```go
const c = 1
const c2 = c + 1
```

#### Funciones de estructuras

Las estructuras y los tipos (type) en general pueden tener asociadas funciones (métodos)

``` go

type Rectangle struct {
    length, width int
}

func (r Rectangle) Area() int {
    return r.length * r.width
}

r1 := Rectangle{4, 3}
fmt.Println("Rectangle area is: ", r1.Area())

// r en este caso se pasa por valor, si no queremos copias dando vueltas usamos punteros

func (r *Rectangle) Area() int {
    return r.length * r.width
}

r1 := &Rectangle{4, 3}
fmt.Println("Rectangle area is: ", r1.Area())

```

#### Interfaces

Las interfaces nos permiten definir el comportamiento de las estructuras.
Se dice que un tipo o estructura implementa una interfaz si cumple con todos los métodos.

```go

// Tenemos una interfaz con comportamiento
type geometry interface {
    area() float64
    perim() float64
}

// Tenemos una estructura que vamos a hacer que respete la interfaz geometry
type rect struct {
    width, height float64
}
func (r rect) area() float64 {
    return r.width * r.height
}
func (r rect) perim() float64 {
    return 2*r.width + 2*r.height
}

// Go no maneja herencia ni nada de eso, pero si podemos usar interfaces para establecer
// comportamientos comunes a objetos, esta interfaz la podría implementar cualquier tipo
// que queramos

// esta función recibe como parámetro un objeto que implementa la interfaz
func measure(g geometry) {
    fmt.Println(g)
    fmt.Println(g.area())
    fmt.Println(g.perim())
}

// Dado que rect implementa todos los métodos de la interfaz lo podemos usar sin problemas
r := rect{width: 3, height: 4}
measure(r)

```

##### switch para tipos

Switch es bastante usado para identificar el tipo de datos de una interface.
Si nosotros recibimos por ejemplo como parámetro una interfaz, podríamos
identificar el tipo de datos real.

``` go

func measure(g geometry) {
    switch t := g.(type) {
    default:
        fmt.Printf("unexpected type %T\n", t)
    case circle:
        fmt.Printf("circle %d\n", g.area())
    case rect:
        fmt.Printf("rect %d\n", g.area())
    case triangle:
        fmt.Printf("triangle %d\n", g.area())
}
```

Es importante, porque es la única forma que tenemos de generalizar un parámetro, no tenemos herencia, por lo que esta forma nos permite recibir distintos tipos de datos como parámetro

```go

// recibimos una interfaz que no implementa ningún método, todos los tipos en go implementan esta interfaz
func PrintIt(a interface{}) {
    switch t := a.(type) {
    default:
        ...
    case int:
        ...
    ...
}

```

### Goroutines

El proceso de multi-threading de go es bastante avanzado, las goroutina es una función que se ejecuta concurrentemente con otras goroutina en el mismo espacio de memoria.

Utiliza diversas estrategias, dependiendo del hardware y el SO, en general terminan ejecutándose en un thread.

Para llamar una goroutina debemos usar el prefijo __go__. Una goroutina se ejecuta en segundo plano, por lo que el que la llama no se bloquea sino que continua en segundo plano.

```go

// Se define una función con go seguido de una función normal
func waitAndShow(message string, delay time.Duration) {
  time.Sleep(delay)
  fmt.Println(message)
}

func announce(message string, delay time.Duration) {
  // La siguiente es una llamada con corutinas
  go waitAndShow(message, delay)
}

```

Si ejecutamos esto en un bloque main, vemos que nunca se imprime la salida.

```go

func main() {
  // fmt nos permite imprimir texto en pantalla
  announce("hola", 100)
  time.Sleep(time.Second)
}
```


##### Channels

Los canales son una forma de comunicarnos en forma asíncrona desde una goroutina con el programa que la llama o bien con otras goroutinas.
Como los arreglos, los canales se crean con make. Y si resultado hace referencia a una estructura de datos.

```go
ci := make(chan int)            // Por ejemplo creamos un canal de int
```

Los canales podrían hacer que el programa que llama a una goroutina espere a obtener un resultado antes de continuar.

```go
c := make(chan int)  // Creamos un canal
// Iniciamos una goroutina, cuando termina envía el resultado al canal.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
i := <-c   // Aca esperamos a que se obtenga un valor desde el canal.
// Hacemos algo con i
```

Cuando leemos datos desde un canal, bloqueamos la ejecución de código hasta que el canal obtenga un valor.

Los canales pueden ser buffers de datos, cuando se trabaja con buffers, el emisor se bloquea si el buffer se llena, hasta que el receptor vaya liberando.

```go
package main

import (
  "fmt"
  "time"
)

// Import nos permite incluir librerías en nuestro código
// Cuando debemos incluir varias librerías usamos un bloque import

// aca estaríamos incluyendo nuestra lib
var sem = make(chan int, 5)

func process(i string) {
  time.Sleep(2 * time.Second)
  fmt.Println(i)
  <-sem
}

func main() {

  queue := make(chan string, 20)
  for i := 0; i < 20; i++ {
    queue <- fmt.Sprintf("val %d", i)
  }
  close(queue)

  for req := range queue {
    // si sem tiene buffer no bloquea este proceso, pero una vez que
    // pero sem solo permite enviar MaxOutstanding 1's, por lo tanto
    // cuando se llene el buffer de sem, si se va a bloquear, y no permitir
    // que se envíen mas 1's hasta que se libere el buffer, lo que significa que
    // se bloquearía en este punto
    sem <- 1
    go process(req)
  }
}


```

Go utiliza los cores disponibles para ejecutar las Goroutinas en paralelo, por lo que la estrategia anterior puede ser muy util para aprovechar completamente los procesados disponibles.

Podemos utilizar _runtime.GOMAXPROCS_ para conocer la cantidad de cores disponibles en el cpu.

var numCPU = runtime.GOMAXPROCS(0)

#### Errors

Por convención cuando definimos nuestros errores respetamos la interface error de go

```go
type error interface {
    Error() string
}

// PathError registra un error de path en los archivos, por ejemplo.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string
    Err error
}

// Entonces respetamos la interface error de go
func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

En general las funciones que vamos a definir deben retornar errores en caso que queramos que sean manejadas por quien llama a la función.

```go

// El segundo resultado es el error, y tenemos que hacernos cargo
file, err := os.Create(filename)
// Siempre es un puntero a una estructura *error, de esta forma podemos evaluar nil.
// en este caso err es de tipo *os.PathError
if err == nil {
    return
}
```

En caso que no podamos recuperarnos de un error y necesitemos colgar la aplicación, usamos panic

```go

// De esta forma el programa sale con error al sistema operativo
panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
```

Por supuesto que debemos evitar llamar esta función, pero si el problema no tiene solución, es la forma de salir al sistema operativo y terminar el programa.

Podemos recuperarnos de una llamada a panic, esto podría ser util en llamadas a goroutina, pero sin embargo hay que evitarlo, porque no es algo normal en go.

```go
func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}

// si do(work) ejecuta panic, la función defer recupera el estado, para no colgar todo el sistema

```
