[//]: # (title: Intermediate Null safety)

In the beginner's tour, you learned how to handle `null` values in your code but there are some more advanced null safety
features that you can use.

## Smart casts and safe casts

You already know how to explicitly declare the type of a variable or object and that Kotlin sometimes has the ability
to infer the type without explicit declaration. The process of specifying a type is also called **casting**. When a type
is automatically cast, like when it's inferred, it's called **smart casting**.

Before we dive more into how casting works, how can you check if an object has a certain type? For this, you can use the
`is` and `!is` operators with `when` or `if`:
* `is` checks if the object has the type and returns a boolean value.
* `!is` checks if the object **doesn't** have the type and returns a boolean value.

For example:

```kotlin
fun printObjectType(obj: Any) {
    when (obj) {
        is String -> println("It's a String with length ${obj.length}")
        is Int -> println("It's an Integer with value $obj")
        !is Double -> println("It's NOT a Double")
        else -> println("Unknown type")
    }
}

fun main() {
    val myString = "Hello, Kotlin!"
    val myInt = 42
    val myDouble = 3.14
    val myList = listOf(1, 2, 3)

    // The type is String
    printObjectType(myString)   
    // It's a String with length 13
    
    // The type is Int
    printObjectType(myInt)      
    // It's an Integer with value 42

    // The type is List, so it's NOT a Double.
    printObjectType(myList)
    // It's NOT a Double
    
    // The type is Double, so the else branch is triggered.
    printObjectType(myDouble)   
    // Unknown type
}
```
{kotlin-runnable="true"}

You can explicitly cast an object to a non-nullable type with the `as` operator. However, if the cast isn't possible then
the compiler throws an error. This why this operator is called the **unsafe** operator.


To explicitly cast an object to a non-nullable type, but return null instead of throwing an error on failure, use the `as?`
operator. Since the `as?` operator doesn't trigger an error on failure, it os called the **safe** operator.


`as` `as?`

## Filter null values from collections

## Throw exceptions

Note that functions returning Nothing can be used on the right side of the Elvis operator to perform precondition 
checking (See Kotlin in Action). if it fails, an exception is thrown.

This unique type signals that a function will never successfully complete its execution and return a value. It means the
function will always throw an exception and prematurely terminate the control flow.

## Next step