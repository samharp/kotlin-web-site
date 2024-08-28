[//]: # (title: Opt-in requirements)

The Kotlin standard library provides a mechanism for requiring and giving explicit consent for using certain API elements.
This mechanism allows library developers to inform users about specific conditions that require opt-in, such as when an API
is in an experimental state and is likely to change in the future. 

To protect API users, the compiler warns about these conditions and requires them to opt in before the API can be used.

## Opt in to API

If a library author marks a declaration from their library's API as _[requiring opt-in](#require-opt-in-for-api)_,
you must give explicit consent to use it in your code.
There are several ways to opt in. We recommend that you choose the way that best suits your situation. 

### Opt in locally

To opt in to a specific API element when you use it in your code, use the `@OptIn` annotation with a reference to the API
element. For example, let's say that you want to use the `DateProvider` class, which requires an opt-in:

```kotlin
// Library code
@RequiresOptIn(message = "This API is experimental. It may be changed in the future without notice.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime

@MyDateTime                            
class DateProvider // A class requiring opt-in
```

In your code, before declaring a function that uses the `DateProvider` class, you must add the `@OptIn` annotation with
a reference to the `MyDateTime` annotation class:

```kotlin
// Client code
@OptIn(MyDateTime::class)
fun getDate(): Date { // Uses DateProvider
    val dateProvider: DateProvider
    // ...
}
```

However, if the `getDate()` function is called elsewhere in your code or used by another developer, no opt-in is required:

```kotlin
// Client code
@OptIn(MyDateTime::class)
fun getDate(): Date { // Uses DateProvider
    val dateProvider: DateProvider
    // ...
}

fun displayDate() {
    println(getDate()) // OK: No opt-in is required
}
```

This is risky because you and others could use experimental API without realizing it.  In the worst case, you could end
up building a supposedly stable API on top of an experimental one. A safer approach is to propagate the opt-in requirements
when you choose to opt in.

#### Propagate opt-in requirements

When you use API in your code that's intended for third-party use, such as a library, you can propagate its opt-in requirement
to your API as well. To do this, annotate your declaration with the _[opt-in requirement annotation](#create-opt-in-requirement-annotations)_ of the API used in its body.

For example, let's use the `DateProvider` class from before, which requires an opt-in:

```kotlin
// Library code
@RequiresOptIn(message = "This API is experimental. It may be changed in the future without notice.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime // Opt-in requirement annotation

@MyDateTime                            
class DateProvider // A class requiring opt-in
```

In your code, before declaring a function that uses the `DateProvider` class, you must add the `@MyDateTime` annotation:

```kotlin
// Client code
@MyDateTime
fun getDate(): Date {  
    val dateProvider: DateProvider // OK: the function requires opt-in as well
    // ...
}

fun displayDate() {
    println(getDate()) // Error: getDate() requires opt-in
}
```

As you can see in this example, the annotated function appears to be a part of the `@MyDateTime` API.
The opt-in propagates the opt-in requirement to users of the `getDate()` function.

If you implicitly use an API element that has an opt-in requirement, you are still required to opt-in. For example, if you
use an API element that doesn't have an opt-in requirement annotation but its signature includes a type that requires
opt-in, using it raises a warning.

```kotlin
// Client code
fun getDate(dateProvider: DateProvider): Date { // Error: DateProvider requires opt-in
    // ...
}

fun displayDate() {
    println(getDate()) // Warning: the signature of getDate() contains DateProvider, which requires opt-in
}
```

To use multiple APIs that require opt-in, mark the declaration with all their opt-in requirement annotations. For example:

```kotlin
@MyDateTime
@AnotherClass
```

Note that if `@OptIn` applies to the declaration whose signature contains a type declared as requiring opt-in,
the opt-in will still propagate:

```kotlin
// Client code
@OptIn(MyDateTime::class)
fun getDate(dateProvider: DateProvider): Date { // Has DateProvider as a part of a signature; propagates the opt-in requirement
    // ...
}

fun displayDate() {
    println(getDate()) // Warning: getDate() requires opt-in
}
```

In modules that don't expose their own API, such as applications, you can opt in to using APIs without propagating
the opt-in requirement to your code. In this case, mark your declaration with [`@OptIn`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-opt-in/)
passing the opt-in requirement annotation as its argument:







### Opt in a file

To use an API that requires opt-in in all functions and classes in a file, add the file-level annotation `@file:OptIn`
to the top of the file before the package specification and imports.

 ```kotlin
 // Client code
 @file:OptIn(MyDateTime::class)
 ```

### Opt in a module

> The `-opt-in` compiler option is available since Kotlin 1.6.0. For earlier Kotlin versions, use `-Xopt-in`.
>
{type="note"}

If you don't want to annotate every usage of APIs that require opt-in, you can opt in to them for your whole module.
To opt in to using an API in a module, compile it with the argument `-opt-in`,
specifying the fully qualified name of the opt-in requirement annotation of the API you use: `-opt-in=org.mylibrary.OptInAnnotation`.
Compiling with this argument has the same effect as if every declaration in the module had the annotation`@OptIn(OptInAnnotation::class)`.

If you build your module with Gradle, you can add arguments like this:

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompilationTask
// ...

tasks.named<KotlinCompilationTask<*>>("compileKotlin").configure {
    compilerOptions.freeCompilerArgs.add("-opt-in=org.mylibrary.OptInAnnotation")
}

```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
import org.jetbrains.kotlin.gradle.tasks.KotlinCompilationTask
// ...

tasks.named('compileKotlin', KotlinCompilationTask) {
    compilerOptions {
        freeCompilerArgs.add("-opt-in=org.mylibrary.OptInAnnotation")
    }
}
```

</tab>
</tabs>

If your Gradle module is a multiplatform module, use the `optIn` method:

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
sourceSets {
    all {
        languageSettings.optIn("org.mylibrary.OptInAnnotation")
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
sourceSets {
    all {
        languageSettings {
            optIn('org.mylibrary.OptInAnnotation')
        }
    }
}
```

</tab>
</tabs>

For Maven, it would be:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-plugin</artifactId>
            <version>${kotlin.version}</version>
            <executions>...</executions>
            <configuration>
                <args>
                    <arg>-opt-in=org.mylibrary.OptInAnnotation</arg>                    
                </args>
            </configuration>
        </plugin>
    </plugins>
</build>
```

To opt in to multiple APIs on the module level, add one of the described arguments for each opt-in requirement marker used in your module.

## Require opt-in for API 

### Create opt-in requirement annotations

If you want to require explicit consent to using your module's API, create an annotation class to use as an _opt-in requirement annotation_.
This class must be annotated with [@RequiresOptIn](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-requires-opt-in/):

```kotlin
@RequiresOptIn
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class MyDateTime
```

Opt-in requirement annotations must meet several requirements:
* `BINARY` or `RUNTIME` [retention](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-retention/)
* No `EXPRESSION`, `FILE`, `TYPE`, or `TYPE_PARAMETER` among [targets](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.annotation/-target/)
* No parameters.

An opt-in requirement can have one of two severity [levels](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-requires-opt-in/-level/):
* `RequiresOptIn.Level.ERROR`. Opt-in is mandatory. Otherwise, the code that uses marked API won't compile. Default level.
* `RequiresOptIn.Level.WARNING`. Opt-in is not mandatory, but advisable. Without it, the compiler raises a warning.

To set the desired level, specify the `level` parameter of the `@RequiresOptIn` annotation.

Additionally, you can provide a `message` to inform API users about special condition of using the API. 
The compiler will show it to users that use the API without opt-in.

```kotlin
@RequiresOptIn(level = RequiresOptIn.Level.WARNING, message = "This API is experimental. It can be incompatibly changed in the future.")
@Retention(AnnotationRetention.BINARY)
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION)
annotation class ExperimentalDateTime
```

If you publish multiple independent features that require opt-in, declare an annotation for each.
This makes the use of API safer for your clients: they can use only the features that they explicitly accept.
This also lets you remove the opt-in requirements from the features independently.

### Mark API elements

To require an opt-in to using an API element, annotate its declaration with an opt-in requirement annotation:

```kotlin
@MyDateTime
class DateProvider

@MyDateTime
fun getTime(): Time {}
```

Note that for some language elements, an opt-in requirement annotation is not applicable:
* You cannot annotate a backing field or a getter of a property, just the property itself.
* You cannot annotate a local variable or a value parameter.

## Opt-in requirements for pre-stable APIs

If you use opt-in requirements for features that are not stable yet, carefully handle the API graduation to avoid 
breaking the client code.

Once your pre-stable API graduates and is released in a stable state, remove its opt-in requirement annotations from declarations.
The clients will be able to use them without restriction. However, you should leave the annotation classes in modules so that 
the existing client code remains compatible.

To let the API users update their modules accordingly (remove the annotations 
from their code and recompile), mark the annotations as [`@Deprecated`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-deprecated/)
and provide the explanation in the deprecation message.

```kotlin
@Deprecated("This opt-in requirement is not used anymore. Remove its usages from your code.")
@RequiresOptIn
annotation class ExperimentalDateTime
```