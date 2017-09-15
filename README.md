# Kotlin Style Guide
A style guide for Android developers writing in Kotlin. These guidelines are based on the official Kotlin [coding conventions](https://kotlinlang.org/docs/reference/coding-conventions.html).

### Naming Style

If in doubt, default to the Java Coding Conventions such as:

* use of camelCase for names (and avoid underscore in names*)
* types start with upper case
* methods and properties start with lower case
* use 4 space indentation
* public functions should have documentation such that it appears in [Kotlin Doc](https://kotlinlang.org/docs/reference/kotlin-doc.html)

*The exception to this is synthetic properties generated by the `kotlin-android-extensions` plugin. This creates snake\_case property names from the snake\_case View IDs defined in a layout file. To fix this you would either need to go against Android's layout ID naming conventions or create new objects which reference the synthetic properties. Hopefully in the future, JetBrains will release an updated plugin which strips out underscores from the generated properties.

`const` properties or `val`s defined in a `companion object` should be SCREAMING\_SNAKE_CASE as if you were defining a `static final` variable in Java.

### Colons

There should be no space before a colon when separating a property/parameter name and it's type. There should however be a space before a colon which separates a type and supertype or interface:

```kotlin
interface Foo<out T : Any> : Bar {
    fun foo(a: Int): T
}
```

### Lambdas

Follow the coding conventions:

>In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters from the body. Whenever possible, a lambda should be passed outside of parentheses.
>
>```kotlin
>list.filter { it > 10 }.map { element -> element * 2 }
>```
>In lambdas which are short and not nested, it's recommended to use the it convention instead of declaring the parameter explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.

To add to this, lambda parameters (or those in destructured declarations) which are unused should be replaced with an underscore, unless leaving the parameter significantly improves readability. Android Studio will nag you about this anyway:

```kotlin
 .subscribe(
	{ bitmap -> view.displayImage(bitmap) },
	{ _ -> view.setUiState(UiState.FAILURE) })
```

### Lambda Returns
Adding a `return` to a Lambda in Kotlin is optional, and multiline Lambdas will just return the last line. In complex multi-line Lambdas, strongly consider adding a `return@map/flatmap/whatever` statement for the sake of clarity, as the return may not be particularly obvious at first glance.

# Classes

### Constructors

Initialize the properties of a class via primary constructor parameters instead of using an init block.

#### Do:

```kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
    // ...
}
```
#### Don't:

```kotlin
class Person(firstName: String, lastName: String, age: Int) {
  val firstName: String
  val lastName: String
  var age: Int

  init {
    this.firstName = firstName
    this.lastName = lastName
    this.age = age
  }

  // ...
}
```

For formatting these primary constructors, follow the Kotlin coding conventions:

>Classes with longer headers should be formatted so that each primary constructor argument is in a separate line with indentation. Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces should be located on the same line as the parenthesis.
>
>For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

```kotlin
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name),
	KotlinMaker {
    // ...
}
```
### Companion Objects
Companion objects, such as those for `Fragment` `newInstance` methods, should be defined at the bottom of a class declaration;

```kotlin
class MyFragment : Fragment() {

    init {
        Injector.INSTANCE.inject(this)
    }

    @Inject lateinit var dataManager: DataManager

    fun doSomething() {
        // ...
    }

    companion object {
        const val BUNDLE_VALUE_ONE = "bundle_value_one"

        fun newInstance(value1: String, value2: String): MyFragment {
            // ...
        }
    }
}
```
As a sidenote regarding `val` properties in companion objects; accessing them from Java requires accessing the companion object too, which is ugly and not idiomatic:

```java
String key = MyFragment.Companion.BUNDLE_VALUE_ONE;
```

Delcaring the property as a `const val` allows you to access a property as if it was static from Java code:

```java
String key = MyFragment.BUNDLE_VALUE_ONE;
```
This also inlines any access to the `val`. There is a caveat though: this only works with 
 For more information, check out [this](https://blog.egorand.me/where-do-i-put-my-constants-in-kotlin/) excellent article regarding constants in Kotlin.

# Functions

### Unit (void) Functions

If a function returns Unit (eg `void` in Java), the return type should be omitted:

```kotlin
fun foo() { // ": Unit" is omitted here
	// ...
}
```

The exception is an empty or stub method, in which case convert the method to an expression body:

```kotlin
fun foo() = Unit
```
With that being said, if stubbing functions prefer using Kotlin's inline `TODO()` function instead to make it obvious that a function is yet to be implemented:

```kotlin
fun foo() {
	TODO("This is yet to be implemented")
}
```
### Expression Bodies

Functions whose bodies are single line should be converted to expression bodies where possible, unless that function returns `Unit`. One exception is passthrough methods, where the caller function is delegating to another class' method. 

Whilst expression bodies allow the omission of the return type, strongly consider keeping it for the sake of readability unless the function quite obviously returns a primitive type such as a `Boolean` or `String`. Whilst it's trivial to work out the return type using an IDE, omitting these types make code review painful and more error prone.

### Functions vs Properties

You'll often find that the `Convert Java file to Kotlin` intention in Android Studio/IntelliJ will convert some methods into properties. This might be a bit jarring coming from a Java background. However, properties are the idiomatic way of doing many things in Kotlin (no getters/setters, for instance). Follow JetBrain's simple guidelines to see whether a property is appropriate:

>In some cases functions with no arguments might be interchangeable with read-only properties. Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.
>
>Prefer a property over a function when the underlying algorithm:
>
>* does not throw
>* has a O(1) complexity
>* is cheap to calculate (or caсhed on the first run)
>* returns the same result over invocations

### Extension Functions

As you would with creating Java util classes, create an extension package and make separate files for each type:

- extensions
  - ContextExtensions.kt
  - ViewExtensions.kt
  - ...

Extension functions are fun, but don't go overboard. Try not to hide mountains of complexity behind clever extensions.

# Flow Control

### When Statements

To keep `when` statements clean, do not call more than one function from a condition. Prefer to move these functions into a single enclosing function:

#### Do
```kotlin
when (aValue) {
  1 -> doSomethingForCaseOne()
  2 -> doSomethingForCaseTwo()
  3 -> doSomethingForCaseThree()
}

fun doSomethingForCaseTwo() {
	foo()
	bar()
}
```

#### Don't

```kotlin
when (aValue) {
    1 -> doSomethingForCaseOne()
    2 -> {
        foo()
        bar()
    }
    3 -> doSomethingForCaseThree()
}
```
Similarly, because `when` doesn't fall through, separate cases using commas if you wish for multiple cases to be handled the same way:

#### Do
```kotlin
when (aValue) {
  1, 3 -> doSomethingForCaseOneAndThree()
  2 -> doSomethingForCaseTwo()
}
```
#### Don't

```kotlin
when (aValue) {
  1 -> doSomethingForCaseOneAndThree()
  2 -> doSomethingForCaseTwo()
  3 -> doSomethingForCaseOneAndThree()
}
```