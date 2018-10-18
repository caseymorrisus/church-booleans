> I’ll start this article with a warning, I mention the use of Lambda Calculus, but do not let that scare you! I promise, no knowledge of λ-Calculus is required to understand or use the main concepts in this article! Phew!

In my last [Functional JS article](https://medium.com/dailyjs/functional-js-with-es6-recursive-patterns-b7d0813ef9e3), we went over recursive patterns that allow you to operate/iterate over array values. This time, we’re going to get a bit more abstract to hopefully explain some of the bare fundamentals of functional programming.

### A moment on Currying

You’ll notice that we never define a function with more than 1 argument. We return a new function for every additional argument needed. This is a technique used very commonly in functional programming that has its roots in Lambda Calculus. This has multiple benefits, but is used mainly to simplify partial application of functions.

### **What are we doing?**

Glad you asked! We’re going to re-create: `true`, `false`, `||`(or), `&&`(and), `!`(not), `==`(equals), and `!=`(not equals). As a bonus, we’re going to use some magical Lambda Calculus to make our functions much more efficient!

### Let’s get started!

The first functions we need to define are our booleans: `true` and `false`. How we define these functions makes more sense once they’re used with the conditional function we soon define. The way we represent booleans as functions is a concept known as Church Booleans.

### TRUE (true):

No, I’m not trying to yell, I’m using this naming convention because true in all lowercase is a reserved word and can’t be used to name our function. This function takes two arguments and will always return the first argument. In comparison, the `FALSE` function does the same, but always returns the second argument. These functions are also known as `selectFirst` and `selectSecond`in other languages.

```javascript
const TRUE = (x) => (y) => x

TRUE('foo')('bar') // => 'foo'
TRUE(10)(5) // => 10
```



### FALSE (false):

Almost identical to the `TRUE` function, but returns the second argument rather than the first.

```javascript
const FALSE = (x) => (y) => y

FALSE('foo')('bar') // => 'bar'
FALSE(10)(5) // => 5
```



### Conditional (cond):

The next function we need to define is our function that will handle conditional logic. Normally we’d use an if/else statement or a ternary to do this, but we want everything to be a function. It’s functions all the way down, this comes in handy later on.

**If you break down a conditional in JavaScript, you end up with 3 parts:**

1. Conditional Statement
2. Expression when `true`
3. Expression when `false`

To mimic this, we will create a function that takes 3 arguments and returns the conditional function with both expressions applied. If you notice, I place the conditional last rather than first. This is a functional programming convention that helps when creating partially applied functions.

If you remember how we defined our booleans, they are both functions that when called with 2 arguments, return one of these arguments. This is exactly how our conditional function is going to work. The conditional function (`TRUE` or `FALSE`) passed in is called with 2 arguments, our expressions when true and false.

```javascript
const cond = (isTrue) => (isFalse) => (conditional) =>
  conditional(isTrue)(isFalse)

// use partial application to store the first two arguments and return a function ready to accept the last
const trueOrFalse = cond('This is true.')('This is false.')
trueOrFalse(TRUE) // => 'This is true.'
trueOrFalse(FALSE) // => 'This is false.'

// The follow will not work, we must use the defined TRUE and FALSE functions
trueOrFalse(true) // => TypeError: conditional is not a function
trueOrFalse(false) // => TypeError: conditional is not a function
```



### Not (!)

This is the first logical operator we will define. It takes a single argument: `x`. If `x` is `TRUE` it returns `FALSE`, if `x` is `FALSE` it returns `TRUE`. We make use of the previously defined `cond` function to handle the conditional logic. Within the conditional: if `x` is `TRUE` we return `FALSE` otherwise we return `TRUE` .

```javascript
const not = (x) => cond(FALSE)(TRUE)(x)

not(FALSE) // => TRUE
not(TRUE) // => FALSE
cond('Foo')('Bar')(not(TRUE)) // => 'Bar'
```



### Or (||)

This is the first logical operator we will define. It takes 2 arguments: `x` and `y`. If either `x` or `y` is `TRUE`, we will return `TRUE`, otherwise we return `FALSE`. Again, we use the `cond` function to handle this. Within the conditional: if `x` is `TRUE` we return `TRUE` otherwise we return `y` .

```javascript
const or = (x) => (y) => cond(TRUE)(y)(x)

or(TRUE)(FALSE) // => TRUE
or(FALSE)(FALSE) // => FALSE
or(FALSE)(TRUE) // => TRUE
or(TRUE)(TRUE) // => TRUE
```



### And (&&)

This function takes 2 arguments: `x` and `y`. Both `x` and `y` must be `TRUE` for this to return `TRUE`, otherwise it will return `FALSE`. We make use of the `cond`function again, not much is different. Within the conditional: if `x` is `TRUE` we return `y` otherwise we return `FALSE`.

```javascript
const and = (x) => (y) => cond(y)(FALSE)(x)

and(TRUE)(FALSE) // => FALSE
and(FALSE)(TRUE) // => FALSE
and(FALSE)(FALSE) // => FALSE
and(TRUE)(TRUE) // => TRUE
```



### Equal (==)

This function takes 2 arguments: `x` and `y`. Both `x` and `y` must be the same value for this to return `TRUE` otherwise it returns `FALSE`. We need to use the `cond` function and the `not` function to handle this one! Within the conditional: if `x` is `TRUE` we return `y` otherwise we return `not(y)`to only return `TRUE` when `y` is also `FALSE`.

```javascript
const equal = (x) => (y) => cond(y)(not(y))(x)

equal(TRUE)(FALSE) // => FALSE
equal(FALSE)(TRUE) // => FALSE
equal(FALSE)(FALSE) // => TRUE
equal(TRUE)(TRUE) // => TRUE
```



### Not Equal (!=)

This function takes 2 arguments: `x` and `y`. `x` and `y` must not be the same value for this to return `TRUE`, otherwise `FALSE` is returned.

```javascript
const notEqual = (x) => (y) => cond(not(y))(y)(x)

notEqual(TRUE)(FALSE) // => TRUE
notEqual(FALSE)(TRUE) // => TRUE
notEqual(FALSE)(FALSE) // => FALSE
notEqual(TRUE)(TRUE) // => FALSE
```



### EXTRA: **β (Beta) Reductions** through λ-Calculus

> Disclaimer: do not feel the need to understand this at all as it is not a requirement to understand and use functional concepts. However, I think it’s an interesting topic to learn about and helps demonstrate the power of functional programming through math. I will gloss over topics as I am not the best resource to learn λ-Calculus, it is not the focus of this article, just an added bonus. These optimizations are handled automatically with most statically compiled functional languages.

#### And (&&) Reduction

In the following example, we use β reduction to remove the use of our `cond`function entirely from our `and` function. Our `and` function is now composed entirely with booleans and is mathematically equivalent!

```javascript
// original and function
const _and = (x) => (y) => cond(y)(FALSE)(x)

// this function can also be written as:
// λx.λy.cond y FALSE x

// it uses `cond` internally which can be written as:
// cond = λe1.λe2.λc.c e1 e2

// let's grab the function body and start reducing

// cond y FALSE x
// expand `cond` function
// (λe1.λe2.λc.c e1 e2) y FALSE x
// (λe2.λc.c y e2) FALSE x
// (λc.c y FALSE) x
// x y FALSE

// which results in:
const and = (x) => (y) => x(y)(FALSE)
```



#### Or (||) Reduction

Again, we use reduction to remove the use of our `cond` function entirely from our `or` function. Like before, our `or` function is now composed entirely with booleans and is mathematically equivalent.

```javascript
// original and function
const _or = (x) => (y) => cond(TRUE)(y)(x)

// this function can also be written as:
// λx.λy.cond TRUE y x

// it uses `cond` internally which can be written as:
// cond = λe1.λe2.λc.c e1 e2

// let's grab the function body and start reducing

// cond TRUE y x
// expand `cond` function
// (λe1.λe2.λc.c e1 e2) TRUE y x
// (λe2.λc.c TRUE e2) y x
// (λc.c TRUE y) x
// x TRUE y

// which results in:
const or = (x) => (y) => x(TRUE)(y)
```



#### Equal (==) Reduction

Just like before, we use reduction to remove the use of our `cond` function entirely from our `equal` function. Our `equal` function is now composed entirely with booleans and is mathematically equivalent.

```javascript
// original and function
const _equal = (x) => (y) => cond(y)(not(y))(x)

// this function can also be written as:
// λx.λy.cond y (not y) x

// it uses `cond` internally which can be written as:
// cond = λe1.λe2.λc.c e1 e2

// it uses `not` internally which can be written as:
// not = λx.cond FALSE TRUE x

// let's grab the function body and start reducing

// cond y (not y) x
// expand `cond` function
// λx.λy.(λe1.λe2.λc.c e1 e2) y (not y) x
// (λe1.λe2.λc.c e1 e2) y (not y) x
// (λe2.λc.c y e2)(not y) x
// (λc.c y (not y)) x
// (x y (not y))
// expand `not` function
// (x y (cond FALSE TRUE y))
// expand `cond` function
// (x y ((λe1.λe2.λc.c e1 e2) FALSE TRUE y))
// (x y ((λe2.λc.c FALSE e2) TRUE y))
// (x y ((λc.c FALSE TRUE) y))
// (x y (y FALSE TRUE))

// which results in:
const equal = (x) => (y) => x(y)(y(FALSE)(TRUE))
```



#### Not Equal(!=) Reduction

For the last time, we remove the use of our `cond` function entirely from our `notEqual` function. Our `notEqual` function is now composed entirely with booleans and is mathematically equivalent.

```javascript
// original and function
const _notEqual = (x) => (y) => cond(not(y))(y)(x)

// this function can also be written as:
// λx.λy.cond y (not y) x

// it uses `cond` internally which can be written as:
// cond = λe1.λe2.λc.c e1 e2

// it uses `not` internally which can be written as:
// not = λx.cond FALSE TRUE x

// let's grab the function body and start reducing

// cond (not y) y x
// expand `cond` function
// (λe1.λe2.λc.c e1 e2) (not y) y x
// (λe2.λc.c (not y) e2) y x
// (λc.c (not y) y) x
// x (not y) y
// expand `not` function
// x ((λx.cond FALSE TRUE x) y) y
// x (cond FALSE TRUE y) y
// expand `cond` function
// x ((λe1.λe2.λc.c e1 e2) FALSE TRUE y) y
// x ((λe2.λc.c FALSE e2) TRUE y) y
// x ((λc.c FALSE TRUE) y) y
// x (y FALSE TRUE) y

// which results in:
const notEqual = (x) => (y) => x(y(FALSE)(TRUE))(y)
```



### Wrapping Up

I hope this helped demonstrate the power and flexibility of functions. This logic can be shared and re-written in any language that has first-class functions!

The functions and concepts used in the article will be expanded in a later article where we use Church numerals to represent natural numbers as pure functions. We will even implement arithmetic and comparison operators for these numbers! 🚀