[//]: # (title: Intermediate Functions)

## Extension functions

In software development, you often need to modify the behavior of a program without altering the original source code. 
For example, in your project, you might want to add additional functionality to a class from a third-party library.

Extension functions allow you to extend a class with additional functionality. You call extension functions as if they 
are member functions of a class.

Before introducing the syntax for extension functions, you need to understand the terms **receiver type** and 
**receiver object**.

Say we have a function that shares some information. This function is called by a sender. The receiver is what the function
is called on. In other words, the receiver is where or with whom the information is shared.

In this example, the `.first()` function is called by `main()` as the sender. The `.first()` function is called on the 
`readOnlyShapes` variable so the `readOnlyShapes` variable is the receiver:

![An example of sender and receiver](sender-receiver.png){width="500"}

The receiver has a **type** so that the compiler understands when the function can be used. The **receiver object** is
the instance of that type.

To declare an extension function, write the name of the class that you want to extend followed by a `.` and the name of
your function. For example: `fun String.bold()`

In the following example:
* `String` is the class that is extended, also known as the receiver type.
* The `.bold()` extension function's return type is `String`.
* An instance of `String` is the receiver object.
* The receiver object is accessed by the [keyword](keyword-reference.md): `this`.
* A string template (`$`) is used to access the value of `this`.
* The `.bold()` extension function takes a string and returns it in a `<b>` HTML element for bold text.

```kotlin
fun String.bold(): String = "<b>$this</b>"

fun main() {
    // "hello" is the receiver object
    println("hello".bold())
    // <b>hello</b>
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-extension-function"}

You can find many examples of extension functions in Kotlin's [standard library](https://kotlinlang.org/api/latest/jvm/stdlib/).
For example, the `String` class has many [extension functions](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/#extension-functions)
to help you work with strings.

### Extension-oriented design

Extension functions can be defined anywhere, enabling you to create extension-oriented designs. Such designs separate 
core functionality from useful but non-essential features, making your code easier to read and maintain.

For example, consider the `Rectangle` class:

```kotlin
class Rectangle(val width: Double, val height: Double) {

    fun area(): Double {
        return width * height
    }

    fun perimeter(): Double {
        return 2 * (width + height)
    }

    fun isSquare(): Boolean {
        return width == height
    }
    fun scale(factor: Double): Rectangle {
        return Rectangle(width * factor, height * factor)
    }
}
```

Let's say that the `area()` and `perimeter()` functions are essential to the `Rectangle` class, while the `isSquare()` 
and `scale()` functions are additional utility functions. To adopt an extension-oriented design, you can move the `isSquare()`
and `scale()` member functions out of the class and convert them into extension functions:

```kotlin
class Rectangle(val width: Double, val height: Double) {

    fun area(): Double {
        return width * height
    }

    fun perimeter(): Double {
        return 2 * (width + height)
    }
}

// Extension functions
fun Rectangle.isSquare(): Boolean {
    return width == height
}

fun Rectangle.scale(factor: Double): Rectangle {
    return Rectangle(width * factor, height * factor)
}
```

Now it is immediately clear which functions are considered essential to the class and which functions are useful but not critical.

This extension-oriented approach is widely used in Kotlin's [standard library](https://kotlinlang.org/api/latest/jvm/stdlib/)
and other libraries. For example, the `String` class has many [extension functions](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-string/#extension-functions)
to help you work with strings.

For more information about extension functions, see [Extensions](extensions.md).

## Extension functions practice

## Scope functions

In programming, a scope is the area in which your variable or object is recognized. The most commonly referred to scopes
are global scope and local scope. In Kotlin, there are scope functions that allow you to create a temporary scope around
an object and execute some code.

Scope functions also make your code more concise because you don't have to refer to the name of your object within the temporary
scope. Depending on the scope function, you can access the object either by referencing it via the keyword `this` or using it as an
argument via the keyword `it`.

Kotlin has five scope functions in total: `.let()`, `.apply()`, `.run()`, `.also()`,  and `with()`.

Each scope function takes a lambda expression and returns either the object or the result of the lambda expression. In 
this tour, we explain each scope function with a recommendation for how to use it.

> If you prefer, the information in this section is also presented in a YouTube video by our developer advocate: Sebastian Aigner.
> To learn more, watch [Back to the Stdlib: Making the Most of Kotlin's Standard Library](https://youtu.be/DdvgvSHrN9g?feature=shared&t=1511).
> 
{type ="tip"}

### Let

Use the `.let()` scope function when you want to perform null checks in your code and later perform further actions
with the returned object.

Consider the example:

```kotlin
fun sendNotification(recipientAddress: String): String {
    println("Yo $recipientAddress!")
    return "Notification sent!"
}

fun getNextAddress(): String {
    return "sebastian@jetbrains.com"
}

fun main() {
    val addr: String? = getNextAddress()
    sendNotification(addr)
}
```
{validate = "false"}

The example has two functions:
* `sendNotification()`, which has a function parameter `recipientAddress` and returns a string.
* `getNextAddress()`, which has no function parameters and returns a string.

The example creates a variable `addr` that has nullable `String` type. But this becomes a problem when you call
the `sendNotification()` function because the `sendNotification()` function doesn't expect that `addr` could be a `null` value.
The compiler reports an error as a result: 

```text
Type mismatch: inferred type is String? but String was expected
```

From the beginner tour, you already know that you can perform a null check with an if condition or use the [Elvis operator](kotlin-tour-null-safety.md#use-elvis-operator). 
But what if you want to use the returned object later on in your code? You could achieve this with an if condition **and** an 
else branch:

```kotlin
fun sendNotification(recipientAddress: String): String {
    println("Yo $recipientAddress!")
    return "Notification sent!"
}

fun getNextAddress(): String {
    return "sebastian@jetbrains.com"
}

fun main() { 
    //sampleStart
    val addr: String? = getNextAddress()
    val confirm = if(addr != null) {
        sendNotification(addr)
    } else { null }
    //sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-let-non-null-if"}

However, a more concise approach is to use the `.let()` scope function:

```kotlin
fun sendNotification(recipientAddress: String): String {
    println("Yo $recipientAddress!")
    return "Notification sent!"
}

fun getNextAddress(): String {
    return "sebastian@jetbrains.com"
}

fun main() {
    //sampleStart
    val addr: String? = getNextAddress()
    val confirm = addr?.let {
        sendNotification(it)
    }
    //sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-let-non-null"}

The example:
* Creates a variable called `confirm`.
* Uses a safe call to call the `.let()` scope function on the `addr` variable.
* Creates a temporary scope within the `.let()` scope function so that you don't have to explicitly refer to the `addr` variable.
* Passes the `sendNotification()` function as a lambda expression into the `.let()` scope function.
* Uses the temporary scope to refer to the `addr` variable via `it`.
* Assigns the result to the `confirm` variable.

With this approach, your code can handle the `addr` variable potentially being a `null` value, and you can use the 
`confirm` variable later on in your code.

### Apply

Use the `.apply()` scope function to initialize objects, like a class instance, at the time of creation rather than later
on in your code. This approach makes your code easier to read and manage.

Consider the example:

```kotlin
class Client() {
    var token: String? = null
    fun connect() = println("connected!")
    fun authenticate() = println("authenticated!")
    fun getData(): String = "Mock data"
}

val client = Client()

fun main() {
    client.token = "asdf"
    client.connect()
    // connected!
    client.authenticate()
    // authenticated!
    client.getData()
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-apply-before"}

The example has a `Client` class that contains one property called `token` and three member functions: `connect()`,
`authenticate()`, and `getData()`.

The example creates `client` as an instance of the `Client` class before initializing its `token` property and calling its
member functions in the `main()` function.

Although this example is compact, in the real world it can be a while after the creation of your class instance before you
configure it and use its member functions. However, if you use the `.apply()` scope function you can create, configure and
use member functions on your class instance in the same place in your code:

```kotlin
class Client() {
  var token: String? = null
  fun connect() = println("connected!")
  fun authenticate() = println("authenticated!")
  fun getData(): String = "Mock data"
}
//sampleStart
val client = Client().apply {
  token = "asdf"
  connect()
  authenticate()
}

fun main() {
  client.getData()
  // connected!
  // authenticated!
}
//sampleEnd
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-apply-after"}

The example:
* Creates `client` as an instance of the `Client` class.
* Uses the `.apply()` scope function on the `client` instance.
* Creates a temporary scope within the `.apply()` scope function so that you don't have to explicitly refer to the `client` instance when accessing its properties or functions.
* Passes a lambda expression to the `.apply()` scope function that updates the `token` property and calls the `connect()` and `authenticate()` functions.
* Calls the `getData()` member function on the `client` instance in the `main()` function.

As you can see, this strategy is convenient when you are working with large pieces of code.

### Run

Similar to `.apply()` you can use the `.run()` scope function to initialize an object, but it's better to use `.run()` 
to initialize an object at a specific moment in your code **and** immediately compute a result.

Let's continue the previous example for the `.apply()` function but this time let's say that you want the `connect()` and
`authenticate()` functions to be grouped so that they are called on every request.

For example:

```kotlin
class Client() {
    var token: String? = null
    fun connect() = println("connected!")
    fun authenticate() = println("authenticated!")
    fun getData(): String = "Mock data"
}

//sampleStart
val client: Client = Client().apply {
    token = "asdf"
}

fun main() {
    val result: String = client.run {
        connect()
        // connected!
        authenticate()
        // authenticated!
        getData()
    }
}
//sampleEnd
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-run"}

The example:
* Creates `client` as an instance of the `Client` class.
* Uses the `.apply()` scope function on the `client` instance.
* Creates a temporary scope within the `.apply()` scope function so that you don't have to explicitly refer to the `client` instance when accessing its properties or functions.
* Passes a lambda expression to the `.apply()` scope function that updates the `token` property.

|--|--|

* Creates a `result` variable with type `String`.
* Uses the `.run()` scope function on the `client` instance.
* Creates a temporary scope within the `.run()` scope function so that you don't have to explicitly refer to the `client` instance when accessing its properties or functions.
* Passes a lambda expression to the `.run()` scope function that calls the `connect()`, `authenticate()`, and `getData()` functions.
* Assigns the result to the `result` variable.

Now you can use the returned result further in your code.

### Also

Use the `.also()` scope function to complete an additional action with an object and then return the object to continue 
using it in your code, like writing a log.

Consider the example:

```kotlin
fun main() {
    val medals: List<String> = listOf("Gold", "Silver", "Bronze")
    val reversedLongUppercaseMedals: List<String> =
        medals
            .map { it.uppercase() }
            .filter { it.length > 4 }
            .reversed()
    println(reversedLongUppercaseMedals)
    // [BRONZE, SILVER]
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-also-before"}

The example:
* Creates the `medals` variable that contains a list of strings.
* Creates the `reversedLongUpperCaseMedals` variable that has `List<String>` type.
* Uses the `.map()` extension function on the `medals` variable.
* Passes a lambda expression to the `.map()` function that refers to `medals` via the `it` keyword and calls the `upperCase()` extension function on it.
* Uses the `.filter()` extension function on the `medals` variable.
* Passes a lambda expression as a predicate to the `.filter()` function that refers to `medals` via the `it` keyword and checks if the length of the list contained in the `medals` variable is longer than 4 items.
* Uses the `.reversed()` extension function on the `medals` variable.
* Assigns the result to the `reversedLongUpperCaseMedals` variable.
* Prints the list contained in the `reversedLongUpperCaseMedals` variable.

It would be useful to add some logging in between the function calls to see what is happening to the `medals` variable.
The `.also()` function helps with that:

```kotlin
fun main() {
    val medals: List<String> = listOf("Gold", "Silver", "Bronze")
    val reversedLongUppercaseMedals: List<String> =
        medals
            .map { it.uppercase() }
            .also { println(it) }
            // [GOLD, SILVER, BRONZE]
            .filter { it.length > 4 }
            .also { println(it) }
            // [SILVER, BRONZE]
            .reversed()
    println(reversedLongUppercaseMedals)
    // [BRONZE, SILVER]
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-also-after"}

Now this example:
* Uses the `.also()` scope function on the `medals` variable.
* Creates a temporary scope within the `.also()` scope function so that you don't have to explicitly refer to the `medals` variable when using it as a function parameter.
* Passes a lambda expression to the `.also()` scope function that calls the `println()` function using the `medals` variable as a function parameter via the `it` keyword.

Since the `.also()` function returns the object, it is useful for not only logging but debugging, chaining
multiple operations and performing other side-effect operations that don't affect the main flow of your code.

### With

Unlike the other scope functions, `with()` is not an extension function, so the syntax is different. You pass the receiver
object to `with()` as an argument. 

Use the `with()` scope function when you want to call multiple functions on an object.

Consider this example:

```kotlin
class Canvas {
    fun rect(x: Int, y: Int, w: Int, h: Int): Unit = print("$x, $y, $w, $h")
    fun circ(x: Int, y: Int, rad: Int): Unit = println("$x, $y, $rad")
    fun text(x: Int, y: Int, str: String): Unit = println("$x, $y, $str")
}

fun main() {
    val mainMonitorPrimaryBufferBackedCanvas = Canvas()

    mainMonitorPrimaryBufferBackedCanvas.text(10, 10, "Foo")
    mainMonitorPrimaryBufferBackedCanvas.rect(20, 30, 100, 50)
    mainMonitorPrimaryBufferBackedCanvas.circ(40, 60, 25)
    mainMonitorPrimaryBufferBackedCanvas.text(15, 45, "Hello")
    mainMonitorPrimaryBufferBackedCanvas.rect(70, 80, 150, 100)
    mainMonitorPrimaryBufferBackedCanvas.circ(90, 110, 40)
    mainMonitorPrimaryBufferBackedCanvas.text(35, 55, "World")
    mainMonitorPrimaryBufferBackedCanvas.rect(120, 140, 200, 75)
    mainMonitorPrimaryBufferBackedCanvas.circ(160, 180, 55)
    mainMonitorPrimaryBufferBackedCanvas.text(50, 70, "Kotlin")
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-with-before"}

The example creates a `Canvas` class that has three member functions: `rect()`, `circ()`, and `text()`. Each of these member
functions prints a statement constructed from the function parameters that you provide.

The example creates `mainMonitorPrimaryBufferBackedCanvas` as an instance of the `Canvas` class before calling a sequence
of member functions on the instance with different function parameters.

You can see that this code is hard to read. If you use the `with()` function, the code becomes much easier to read:

```kotlin
class Canvas {
    fun rect(x: Int, y: Int, w: Int, h: Int): Unit = print("$x, $y, $w, $h")
    fun circ(x: Int, y: Int, rad: Int): Unit = println("$x, $y, $rad")
    fun text(x: Int, y: Int, str: String): Unit = println("$x, $y, $str")
}

fun main() {
    //sampleStart
    val mainMonitorSecondaryBufferBackedCanvas = Canvas()
    with(mainMonitorSecondaryBufferBackedCanvas) {
        text(10, 10, "Foo")
        rect(20, 30, 100, 50)
        circ(40, 60, 25)
        text(15, 45, "Hello")
        rect(70, 80, 150, 100)
        circ(90, 110, 40)
        text(35, 55, "World")
        rect(120, 140, 200, 75)
        circ(160, 180, 55)
        text(50, 70, "Kotlin")
    }
    //sampleEnd
}
```
{kotlin-runnable="true" id="kotlin-tour-scope-function-with-after"}

This example:
* Uses the `with()` scope function with the `mainMonitorSecondaryBufferBackedCanvas` instance as the receiver object.
* Creates a temporary scope within the `with()` scope function so that you don't have to explicitly refer to the `mainMonitorSecondaryBufferBackedCanvas` instance when calling its member functions.
* Passes a lambda expression to the `.also()` scope function that calls the `println()` function using the `medals` variable as a function parameter via the `it` keyword.
* Calls a sequence of member functions with different function parameters.

Now that this code is much easier to read, you are less likely to make mistakes.

## Scope functions practice

## Function literals with receiver

In the beginner's tour, you learned how to use [lambda expressions](kotlin-tour-functions.md#lambda-expressions) to make
your code more concise. A lambda expression is an example of a **function literal** in Kotlin. 

A function literal is a function that is declared:
* Without a name.
* As an expression.

For example, consider the first lambda expression that you saw in the beginner's tour:

```kotlin
fun main() {
    val upperCaseString = { text: String -> text.uppercase() }
    println(upperCaseString("hello"))
    // HELLO
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-intermediate-tour-function-literal"}

The lambda expression has no name and the function is defined within curly braces `{}`: `text: String -> text.uppercase()`.

Function literals are useful not only for making your code more concise but also because they make it easy to pass behavior
as function parameters. For example:

```kotlin
fun performOperation(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

fun main() {
  val sum = performOperation(5, 3) { x, y -> x + y }
  println(sum) // Output: 8
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-intermediate-tour-function-literal-behavior"}

In the example, the `performOperation()` function has a parameter `operation` that accepts any function with function type:
`(Int, Int) -> Int)`. The `performOperation()` function is called with a function literal in the form of a lambda expression: 
`{ x, y -> x + y }`. The result of the lambda expression and integers is assigned to a variable: `sum`.

In addition, you can use function literals where the function is passed as a function parameter to be executed at a certain point
in time, like for callbacks or event listeners. To see examples of these use cases, try [our exercises](#function-literals-with-receiver-practice).

Now that you understand what a function literal is, you're ready to learn about **function literals with receiver**.
As the name implies, a function literal **with receiver** is a function literal that can be called on a **receiver object**.
This means that you can access the receiver's properties and member functions without explicitly referring to the receiver
object. 

Like extension functions, function literals with receiver are a convenient way to extend a class without modifying the class
directly.

The syntax for a function literal with receiver is different when you define the function type: write the receiver object
that you want to extend followed by a `.` and the rest of your function type definition. For example: `MutableList<Int>.() -> Unit`.

This function type has:
* `MutableList<Int>` as the receiver object.
* no function parameters: `()`.
* no return value: `Unit`.

Let's consider an example that extends the `StringBuilder` class:

```kotlin
fun main() {
    // Function literal with receiver definition
    val appendText: StringBuilder.() -> Unit = {
        append("Hello!")
    }

    // Use the function literal with receiver
    val stringBuilder = StringBuilder()
    stringBuilder.appendText()
    println(stringBuilder.toString())  
    // Hello!
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-intermediate-tour-function-literal-with-receiver"}

In this example:
* `StringBuilder` is the class that is extended, also known as the receiver type.
* The function type of the function literal has no function parameters `()` and has no return value `Unit`.
* The function literal is a lambda expression.
* The lambda expression calls the `append()` member function from the `StringBuilder` class and uses the string "Hello!" as the function parameter.
* The function literal is assigned to the `appendText` variable.

|---|---|

* An instance of the `StringBuilder` class is created.
* The lambda expression assigned to `appendText` is called on the `stringBuilder` instance.
* The `stringBuilder` instance is converted to string with the `.toString()` function and printed via the `println()` function.

> Compared to extension functions, function literals with receiver are more flexible as they can be defined locally, passed
> to another function as a parameter, or assigned to a variable.
>
{type="note"}

Function literals with receiver are helpful when you want to create a domain-specific language (DSL). Since you have
access to the receiver object's member functions and properties without explicitly referencing the receiver, your code 
becomes more concise and easier to read.

Let's consider an example that configures items in a menu. Let's begin with a `MenuItem` class, and a `Menu` class that contains
a function to add items to the menu called `item()` as well as a list of all menu items `items`:

```kotlin
class MenuItem(val name: String, val action: () -> Unit)

class Menu(val name: String) {
  val items = mutableListOf<MenuItem>()

  fun item(name: String, action: () -> Unit) {
    items.add(MenuItem(name, action))
  }
}
```

Let's use a function literal with receiver passed as a function parameter (`init`) to the `menu()` function that builds 
a menu as a starting point. You'll notice that the code follows a similar approach to the previous example with the 
`StringBuilder` class:

```kotlin
fun menu(name: String, init: Menu.() -> Unit): Menu {
    // Creates an instance of the Menu class
    val menu = Menu(name)
    // Calls the function literal with receiver init() on the class instance
    menu.init()
    return menu
}
```

Now let's use the DSL to configure a menu and create a `printMenu()` function to print the menu structure to console:

```kotlin
class MenuItem(val name: String, val action: () -> Unit)

class Menu(val name: String) {
  val items = mutableListOf<MenuItem>()

  fun item(name: String, action: () -> Unit) {
    items.add(MenuItem(name, action))
  }
}

fun menu(name: String, init: Menu.() -> Unit): Menu {
  val menu = Menu(name)
  menu.init()
  return menu
}

//sampleStart
fun printMenu(menu: Menu) {
  println("Menu: ${menu.name}")
  menu.items.forEach { println("  Item: ${it.name}") }
}

// Use the DSL
fun main() {
    // Create the menu
    val mainMenu = menu("Main Menu") {
        // Add items to the menu
        item("Home") { println("Home selected") }
        item("Settings") { println("Settings selected") }
        item("New File") { println("New file created") }
        item("Open File") { println("File opened") }
        item("Exit") { println("Exiting program") }
    }
    
    // Print the menu
    printMenu(mainMenu)
    // Menu: Main Menu
      // Item: Home
      // Item: Settings
      // Item: New File
      // Item: Open File
      // Item: Exit
}
//sampleEnd
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-intermediate-tour-function-literal-with-receiver-dsl"}

> Function literals with receivers can be combined with **type-safe builders** in Kotlin to make DSLs that detect any problems
> with types at compile time rather than at runtime. To learn more, see [Type-safe builders](type-safe-builders.md).
>
{type="note"}

## Function literals with receiver practice

### Exercise 1 {initial-collapse-state="collapsed" id="function-literals-exercise-1"}

Exercise for function literals as callbacks

```kotlin
fun fetchData(callback: (String) -> Unit) {
    // Simulate network request
    callback("Data received")
}

fetchData { data -> println(data) } // Output: Data received
```

### Exercise 2 {initial-collapse-state="collapsed" id="function-literals-exercise-2"}

Exercise for function literals as event listeners

```kotlin
button.setOnClickListener {
  println("Button clicked!")
}
```

### Exercise 3 {initial-collapse-state="collapsed" id="function-literals-exercise-3"}

Exercise for function literals with receiver with `buildString()` and `buildList()` functions.

## Infix notation

Another way that you can make the code for your member or extension functions more concise and readable in Kotlin is to 
use infix notation. 

When you add the `infix` keyword to your function declaration, it means that you don't need to use 
the `.` or parentheses when you call your function. This turns your function into an infix function. For example:

```kotlin
class Point(val x: Int, val y: Int) {
  // Add the infix keyword to the moveBy() function
  infix fun moveBy(delta: Point): Point {
    return Point(x + delta.x, y + delta.y)
  }
}

fun main() {
  val p1 = Point(2, 3)
  val p2 = Point(4, 5)

  // Use infix notation
  val p3 = p1 moveBy p2
  println("New Point: (${p3.x}, ${p3.y})")
  // New Point: (6, 8)

  // Alternatively, use the full function call
  val p4 = p1.moveBy(p2)
  println("New Point: (${p4.x}, ${p4.y})")
  // New Point: (6, 8)
}
```
{kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-intermediate-tour-function-infix"}

Similar to function literals with receiver, infix notation is a handy feature to use when you want to create expressive 
DSLs. Infix notation helps your code read more like a natural language.

There are some rules to keep in mind when using infix notation:
* The function must be a member or extension function.
* The function must have only a **single** function parameter.
* The function parameter must not accept a variable number of arguments.
* The function parameter can't have a default value.
* When calling an infix function on a receiver object, you must refer to the receiver object by using keyword `this`.

Now that you've learned new ways to **extend** classes, it's time to learn even more about classes themselves, their special types,
alternatives, and how class inheritance works.

### Exercise {initial-collapse-state="collapsed" id="infix-notation-exercise"}

Convert the `append()` extension function into an infix function and use it in a Kotlin program.

|---|---|
```kotlin
// Change the code here
fun MutableList<Int>.append(item: Int) {
  this.add(item)
}

fun main() {
  val list = mutableListOf(1, 2, 3)
  // Change the code here
  list.append(4)
  println(list)
}
```
{validate="false" kotlin-runnable="true" kotlin-min-compiler-version="1.3" id="kotlin-tour-infix-notation-exercise"}

|---|---|
```kotlin
infix fun MutableList<Int>.append(item: Int) {
  this.add(item)
}

fun main() {
  val list = mutableListOf(1, 2, 3)
  list append 4
  println(list)  // Output: [1, 2, 3, 4]
}
```
{initial-collapse-state="collapsed" collapsed-title="Example solution" id="kotlin-tour-infix-notation-solution"}

## Next step

[Intermediate classes](kotlin-tour-intermediate-classes.md)