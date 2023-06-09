Arrays
To specify the type of an array like [1, 2, 3], you can use the syntax number[]; this syntax works for any type (e.g. string[] is an array of strings, and so on). You may also see this written as Array<number>, which means the same thing. We’ll learn more about the syntax T<U> when we cover generics.

Note that [number] is a different thing; refer to the section on Tuples.

any
TypeScript also has a special type, any, that you can use whenever you don’t want a particular value to cause typechecking errors.

When a value is of type any, you can access any properties of it (which will in turn be of type any), call it like a function, assign it to (or from) a value of any type, or pretty much anything else that’s syntactically legal:

let obj: any = { x: 0 };
// None of the following lines of code will throw compiler errors.
// Using `any` disables all further type checking, and it is assumed 
// you know the environment better than TypeScript.
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
Try
The any type is useful when you don’t want to write out a long type just to convince TypeScript that a particular line of code is okay.

noImplicitAny
When you don’t specify a type, and TypeScript can’t infer it from context, the compiler will typically default to any.

You usually want to avoid this, though, because any isn’t type-checked. Use the compiler flag noImplicitAny to flag any implicit any as an error.

Type Annotations on Variables
When you declare a variable using const, var, or let, you can optionally add a type annotation to explicitly specify the type of the variable:

let myName: string = "Alice";
Try
TypeScript doesn’t use “types on the left”-style declarations like int x = 0; Type annotations will always go after the thing being typed.

In most cases, though, this isn’t needed. Wherever possible, TypeScript tries to automatically infer the types in your code. For example, the type of a variable is inferred based on the type of its initializer:

// No type annotation needed -- 'myName' inferred as type 'string'
let myName = "Alice";
Try
For the most part you don’t need to explicitly learn the rules of inference. If you’re starting out, try using fewer type annotations than you think - you might be surprised how few you need for TypeScript to fully understand what’s going on.

Functions
Functions are the primary means of passing data around in JavaScript. TypeScript allows you to specify the types of both the input and output values of functions.

Parameter Type Annotations
When you declare a function, you can add type annotations after each parameter to declare what types of parameters the function accepts. Parameter type annotations go after the parameter name:

// Parameter type annotation
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
Try
When a parameter has a type annotation, arguments to that function will be checked:

// Would be a runtime error if executed!
greet(42);
Argument of type 'number' is not assignable to parameter of type 'string'.
Try
Even if you don’t have type annotations on your parameters, TypeScript will still check that you passed the right number of arguments.

Return Type Annotations
You can also add return type annotations. Return type annotations appear after the parameter list:

function getFavoriteNumber(): number {
  return 26;
}
Try
Much like variable type annotations, you usually don’t need a return type annotation because TypeScript will infer the function’s return type based on its return statements. The type annotation in the above example doesn’t change anything. Some codebases will explicitly specify a return type for documentation purposes, to prevent accidental changes, or just for personal preference.

Anonymous Functions
Anonymous functions are a little bit different from function declarations. When a function appears in a place where TypeScript can determine how it’s going to be called, the parameters of that function are automatically given types.

Here’s an example:

// No type annotations here, but TypeScript can spot the bug
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function
names.forEach(function (s) {
  console.log(s.toUppercase());
Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUppercase());
Property 'toUppercase' does not exist on type 'string'. Did you mean 'toUpperCase'?
});
Try
Even though the parameter s didn’t have a type annotation, TypeScript used the types of the forEach function, along with the inferred type of the array, to determine the type s will have.

This process is called contextual typing because the context that the function occurred within informs what type it should have.

Similar to the inference rules, you don’t need to explicitly learn how this happens, but understanding that it does happen can help you notice when type annotations aren’t needed. Later, we’ll see more examples of how the context that a value occurs in can affect its type.

Object Types
Apart from primitives, the most common sort of type you’ll encounter is an object type. This refers to any JavaScript value with properties, which is almost all of them! To define an object type, we simply list its properties and their types.

For example, here’s a function that takes a point-like object:

// The parameter's type annotation is an object type
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
Try
Here, we annotated the parameter with a type with two properties - x and y - which are both of type number. You can use , or ; to separate the properties, and the last separator is optional either way.

The type part of each property is also optional. If you don’t specify a type, it will be assumed to be any.

Optional Properties
Object types can also specify that some or all of their properties are optional. To do this, add a ? after the property name:

function printName(obj: { first: string; last?: string }) {
  // ...
}
// Both OK
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
Try
In JavaScript, if you access a property that doesn’t exist, you’ll get the value undefined rather than a runtime error. Because of this, when you read from an optional property, you’ll have to check for undefined before using it.

function printName(obj: { first: string; last?: string }) {
  // Error - might crash if 'obj.last' wasn't provided!
  console.log(obj.last.toUpperCase());
'obj.last' is possibly 'undefined'.
  if (obj.last !== undefined) {
    // OK
    console.log(obj.last.toUpperCase());
  }
 
  // A safe alternative using modern JavaScript syntax:
  console.log(obj.last?.toUpperCase());
}
Try
Union Types
TypeScript’s type system allows you to build new types out of existing ones using a large variety of operators. Now that we know how to write a few types, it’s time to start combining them in interesting ways.

Defining a Union Type
The first way to combine types you might see is a union type. A union type is a type formed from two or more other types, representing values that may be any one of those types. We refer to each of these types as the union’s members.

Let’s write a function that can operate on strings or numbers:

function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
Try
Working with Union Types
It’s easy to provide a value matching a union type - simply provide a type matching any of the union’s members. If you have a value of a union type, how do you work with it?

TypeScript will only allow an operation if it is valid for every member of the union. For example, if you have the union string | number, you can’t use methods that are only available on string:

function printId(id: number | string) {
  console.log(id.toUpperCase());
Property 'toUpperCase' does not exist on type 'string | number'.
  Property 'toUpperCase' does not exist on type 'number'.
}
Try
The solution is to narrow the union with code, the same as you would in JavaScript without type annotations. Narrowing occurs when TypeScript can deduce a more specific type for a value based on the structure of the code.

For example, TypeScript knows that only a string value will have a typeof value "string":

function printId(id: number | string) {
  if (typeof id === "string") {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase());
  } else {
    // Here, id is of type 'number'
    console.log(id);
  }
}
Try
Another example is to use a function like Array.isArray:

function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // Here: 'x' is 'string[]'
    console.log("Hello, " + x.join(" and "));
  } else {
    // Here: 'x' is 'string'
    console.log("Welcome lone traveler " + x);
  }
}
Try
Notice that in the else branch, we don’t need to do anything special - if x wasn’t a string[], then it must have been a string.

Sometimes you’ll have a union where all the members have something in common. For example, both arrays and strings have a slice method. If every member in a union has a property in common, you can use that property without narrowing:

// Return type is inferred as number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
Try
It might be confusing that a union of types appears to have the intersection of those types’ properties. This is not an accident - the name union comes from type theory. The union number | string is composed by taking the union of the values from each type. Notice that given two sets with corresponding facts about each set, only the intersection of those facts applies to the union of the sets themselves. For example, if we had a room of tall people wearing hats, and another room of Spanish speakers wearing hats, after combining those rooms, the only thing we know about every person is that they must be wearing a hat.

Type Aliases
We’ve been using object types and union types by writing them directly in type annotations. This is convenient, but it’s common to want to use the same type more than once and refer to it by a single name.

A type alias is exactly that - a name for any type. The syntax for a type alias is:

type Point = {
  x: number;
  y: number;
};
 
// Exactly the same as the earlier example
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
Try
You can actually use a type alias to give a name to any type at all, not just an object type. For example, a type alias can name a union type:

type ID = number | string;
Try
Note that aliases are only aliases - you cannot use type aliases to create different/distinct “versions” of the same type. When you use the alias, it’s exactly as if you had written the aliased type. In other words, this code might look illegal, but is OK according to TypeScript because both types are aliases for the same type:

type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// Create a sanitized input
let userInput = sanitizeInput(getInput());
 
// Can still be re-assigned with a string though
userInput = "new input";
Try
Interfaces
An interface declaration is another way to name an object type:

interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
Try
Just like when we used a type alias above, the example works just as if we had used an anonymous object type. TypeScript is only concerned with the structure of the value we passed to printCoord - it only cares that it has the expected properties. Being concerned only with the structure and capabilities of types is why we call TypeScript a structurally typed type system.

Differences Between Type Aliases and Interfaces
Type aliases and interfaces are very similar, and in many cases you can choose between them freely. Almost all features of an interface are available in type, the key distinction is that a type cannot be re-opened to add new properties vs an interface which is always extendable.

Interface	Type
Extending an interface

interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
        
Extending a type via intersections

type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: boolean 
}

const bear = getBear();
bear.name;
bear.honey;
        
Adding new fields to an existing interface

interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
        
A type cannot be changed after being created

type Window = {
  title: string
}

type Window = {
  ts: TypeScriptAPI
}

 // Error: Duplicate identifier 'Window'.

        
You’ll learn more about these concepts in later chapters, so don’t worry if you don’t understand all of these right away.

Prior to TypeScript version 4.2, type alias names may appear in error messages, sometimes in place of the equivalent anonymous type (which may or may not be desirable). Interfaces will always be named in error messages.
Type aliases may not participate in declaration merging, but interfaces can.
Interfaces may only be used to declare the shapes of objects, not rename primitives.
Interface names will always appear in their original form in error messages, but only when they are used by name.
For the most part, you can choose based on personal preference, and TypeScript will tell you if it needs something to be the other kind of declaration. If you would like a heuristic, use interface until you need to use features from type.

Type Assertions
Sometimes you will have information about the type of a value that TypeScript can’t know about.

For example, if you’re using document.getElementById, TypeScript only knows that this will return some kind of HTMLElement, but you might know that your page will always have an HTMLCanvasElement with a given ID.

In this situation, you can use a type assertion to specify a more specific type:

const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
Try
Like a type annotation, type assertions are removed by the compiler and won’t affect the runtime behavior of your code.

You can also use the angle-bracket syntax (except if the code is in a .tsx file), which is equivalent:

const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
Try
Reminder: Because type assertions are removed at compile-time, there is no runtime checking associated with a type assertion. There won’t be an exception or null generated if the type assertion is wrong.

TypeScript only allows type assertions which convert to a more specific or less specific version of a type. This rule prevents “impossible” coercions like:

const x = "hello" as number;
Conversion of type 'string' to type 'number' may be a mistake because neither type sufficiently overlaps with the other. If this was intentional, convert the expression to 'unknown' first.
Try
Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid. If this happens, you can use two assertions, first to any (or unknown, which we’ll introduce later), then to the desired type:

const a = (expr as any) as T;
Try
Literal Types
In addition to the general types string and number, we can refer to specific strings and numbers in type positions.

One way to think about this is to consider how JavaScript comes with different ways to declare a variable. Both var and let allow for changing what is held inside the variable, and const does not. This is reflected in how TypeScript creates types for literals.

let changingString = "Hello World";
changingString = "Olá Mundo";
// Because `changingString` can represent any possible string, that
// is how TypeScript describes it in the type system
changingString;
      
let changingString: string
 
const constantString = "Hello World";
// Because `constantString` can only represent 1 possible string, it
// has a literal type representation
constantString;
      
const constantString: "Hello World"
Try
By themselves, literal types aren’t very valuable:

let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
Type '"howdy"' is not assignable to type '"hello"'.
Try
It’s not much use to have a variable that can only have one value!

But by combining literals into unions, you can express a much more useful concept - for example, functions that only accept a certain set of known values:

function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}
printText("Hello, world", "left");
printText("G'day, mate", "centre");
Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
Try
Numeric literal types work the same way:

function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
Try
Of course, you can combine these with non-literal types:

interface Options {
  width: number;
}
function configure(x: Options | "auto") {
  // ...
}
configure({ width: 100 });
configure("auto");
configure("automatic");
Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
Try
There’s one more kind of literal type: boolean literals. There are only two boolean literal types, and as you might guess, they are the types true and false. The type boolean itself is actually just an alias for the union true | false.

Literal Inference
When you initialize a variable with an object, TypeScript assumes that the properties of that object might change values later. For example, if you wrote code like this:

const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
Try
TypeScript doesn’t assume the assignment of 1 to a field which previously had 0 is an error. Another way of saying this is that obj.counter must have the type number, not 0, because types are used to determine both reading and writing behavior.

The same applies to strings:

const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
Argument of type 'string' is not assignable to parameter of type '"GET" | "POST"'.
Try
In the above example req.method is inferred to be string, not "GET". Because code can be evaluated between the creation of req and the call of handleRequest which could assign a new string like "GUESS" to req.method, TypeScript considers this code to have an error.

There are two ways to work around this.

You can change the inference by adding a type assertion in either location:

// Change 1:
const req = { url: "https://example.com", method: "GET" as "GET" };
// Change 2
handleRequest(req.url, req.method as "GET");
Try
Change 1 means “I intend for req.method to always have the literal type "GET"”, preventing the possible assignment of "GUESS" to that field after. Change 2 means “I know for other reasons that req.method has the value "GET"“.

You can use as const to convert the entire object to be type literals:

const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
Try
The as const suffix acts like const but for the type system, ensuring that all properties are assigned the literal type instead of a more general version like string or number.

null
and
undefined
JavaScript has two primitive values used to signal absent or uninitialized value: null and undefined.

TypeScript has two corresponding types by the same names. How these types behave depends on whether you have the strictNullChecks option on.

strictNullChecks
off
With strictNullChecks off, values that might be null or undefined can still be accessed normally, and the values null and undefined can be assigned to a property of any type. This is similar to how languages without null checks (e.g. C#, Java) behave. The lack of checking for these values tends to be a major source of bugs; we always recommend people turn strictNullChecks on if it’s practical to do so in their codebase.

strictNullChecks
on
With strictNullChecks on, when a value is null or undefined, you will need to test for those values before using methods or properties on that value. Just like checking for undefined before using an optional property, we can use narrowing to check for values that might be null:

function doSomething(x: string | null) {
  if (x === null) {
    // do nothing
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
Try
Non-null Assertion Operator (Postfix
!
)
TypeScript also has a special syntax for removing null and undefined from a type without doing any explicit checking. Writing ! after any expression is effectively a type assertion that the value isn’t null or undefined:

function liveDangerously(x?: number | null) {
  // No error
  console.log(x!.toFixed());
}
Try
Just like other type assertions, this doesn’t change the runtime behavior of your code, so it’s important to only use ! when you know that the value can’t be null or undefined.

Enums
Enums are a feature added to JavaScript by TypeScript which allows for describing a value which could be one of a set of possible named constants. Unlike most TypeScript features, this is not a type-level addition to JavaScript but something added to the language and runtime. Because of this, it’s a feature which you should know exists, but maybe hold off on using unless you are sure. You can read more about enums in the Enum reference page.

Less Common Primitives
It’s worth mentioning the rest of the primitives in JavaScript which are represented in the type system. Though we will not go into depth here.

bigint
From ES2020 onwards, there is a primitive in JavaScript used for very large integers, BigInt:

// Creating a bigint via the BigInt function
const oneHundred: bigint = BigInt(100);
 
// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
Try
You can learn more about BigInt in the TypeScript 3.2 release notes.

symbol
There is a primitive in JavaScript used to create a globally unique reference via the function Symbol():

const firstName = Symbol("name");
const secondName = Symbol("name");
 
if (firstName === secondName) {
This comparison appears to be unintentional because the types 'typeof firstName' and 'typeof secondName' have no overlap.
  // Can't ever happen
}
link: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#arrays
20 Array methods in Typescript you need to know with examples
1. indexOf()
Every array element has an index. This method returns the index of an element in an array.

a. If specific element doesn’t exist within array, indexOf() returns -1.

Hence, this method can be used to check whether an element exists in an array or not.

syntax:
array.indexOf(element)


2. lastIndexOf()
This method returns an array’s last element’s index.

a. If an array is empty then, it returns -1 as of the indexOf() function.
b. If an array has one more same element, then it returns the maximum index of duplicate items.

syntax:
array.lastIndexOf(element)


3. concat()
As the name suggests, this method simply merges two arrays and returns a combined result.

syntax:
array1.concat(array2)


4. join()
According to the name, this method joins all elements of the array into a string with a given operator.

a. If an operator is not given, it joins elements with a comma(,).

syntax:
array.join(operator)


5. push()
This method pushes/adds one or more elements to the array at the last of an array.

syntax:
array.push(element)


6. pop()
This method pops/removes the last element from an array.

syntax:
array.pop()


7. reverse()
As per the name, this method reverts the order of an array.

syntax:
array.reverse()


8. shift()
This method removes starting(first) element from an array and returns the removed element.
We can say that it’s the exact opposite of pop() method, which removes the last element and returns the result.

syntax:
array.shift()


9. unshift()
It has the exact opposite behavior to the shift() method. It adds an element at starting of an array and returns a new array.

syntax:
array.unshift(element)


10. slice()
This method cuts an array, in whichever manner we want and returns the trimmed array.
a. It excludes the last index from an argument.

syntax:
array.slice(start_index, end_index)


11. splice()
This method can be used for multiple purposes. For,
1. Add an element to an array
2. Replace specific elements within an array
3. Remove specific elements from an array

syntax:
array.splice(index, number of elements to be removed, element1,..,elementN)


12. toString()
This method converts an array to a comma-separated string.

syntax:
array.toString()


13. filter()
This method can also be used in multiple use cases. Like, such as finding even numbers from an array, finding common items from two arrays, or getting a distinct array.

Basically, it checks the conditions which are provided and returns a filtered array.

syntax:
array.filter(callback)


14. map()
This method creates a new array with the results of calling a provided function on every element in this array.
In the example, we’ve invoked map() with Math.ceil which returns the lowest maximum number.

syntax:
array.map(callback)


15. every()
This method tests whether all the elements in an array pass the test implemented by the provided function.

In the example, we have checked for even numbers.

syntax:
array.every(callback)


16. reduceRight()
This method applies a function simultaneously against two values of the array (from right to left) to reduce it to a single value.

In the example, the array is reduced with the addition of an element to the previous one(right to left).

syntax:
array.reduceRight(callback)


17. reduce()
This method behaves the exact opposite of the reduceRight() method.
It applies a function simultaneously against two values of the array (from left to right) to reduce it to a single value.

In the example, an array is reduced with the subtraction of an element from the previous one(left to right).

syntax:
array.reduce(callback)


18. some()
This method is generally used for testing purposes.
i.e. To know whether at least a single item from an array is fulfilling a given condition or not.

In the example, again we’ve checked for at least a single even number present in an array.

syntax:
array.some(callback)


19. sort()
As the name suggests, this method arranges array elements in sorting orders.

In the example, we’ve sorted an array in ascending order. It will sort in descending order, with the condition b-a instead of a-b . similarly, we do in js.

syntax:
array.sort(callback)


20. fill()
This method changes all elements in an array to a static value, from a start index (default 0) to an end index (default array.length) and returns the modified array.

a. It can add new elements to specific(multiple) positions

syntax:
array.fill(value, start_index, end_index)


