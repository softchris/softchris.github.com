# Typescript fundamentals - part 1

This is meant to be a series of blog posts, this is part I. It is supposed to educate you enough so when someone says your next project involves typescript, you are not panicking, you might even feel so confident as to take charge.

Typescript is not its own language it is a superset of javascript.

## Installation

```
npm install typescript -g
```

## Features

### Types

Typescript is about dealing with types. You should use types whenever possible in your code. There are four known types in javascript, **number**, **boolean**, **string** and **any**.

**Any** is the base type that every other type stems from. Let's look at few examples using types.

```
var val:number = 4;

var val2 = 4 // type inference, typescript makes an honest attempt to guess your type.
val = '3'  // illegal according to compiler as '3' is a string
val2 = false; // also illegal as compiler has inferred val2 to be of type number
var val3:any = 'a string';
val3 = false;  // legal as any allows anything to be assigned to it. Avoid
```

#### Types in functions

Typescript compiler allows for your es5 code to exist when compiling but you can make it better by adding types to it.

Old es5 code

```
function add( a, b ) {
  return a + b;
}
```

Adding typescript it looks like so

```
function add(a:number, b:number):number {
  return a + b;
}
```

So when using it I you will get a compile time error if doing so:

```
add(1, 'a string')
```

Or if I you try to change the definition like so:

```
function add(a:number, b:number):number {
  return ' a string ';
}
```

If you don't specifiy a type on your variables or method returns, they will automatically become of type **any**, so try to give them types as much as possible to make your code less error prone.

### Compiler

When installing typescript it comes with a compiler called **tsc**. Let's take it for a spin.

Imagine you have the following code written in typescript.

```
// variables.ts
var a:number = 3;

function add(a:number, b:number){
  return a + b;
}

var result = add(3,3);

console.log(result);
```

When compiling

```
tsc variables.ts
```

it compiles to es5 that looks like this:

```
var a = 3;
function add(a, b) {
  return a + b;
}

var result = add(3, 3);
console.log(result);
```

So it becomes pretty standard code. But the typesafety isn't there for you when having compiled but rather _when compiling_.

So with the first compile we created by default a js file with the same name as the typescript file:

```
tsc variables.ts --out dist.js
```

The above line renames the resulting file into dist.js. So why is this fun? It isn't until you start compiling more than one file. Let's first create the _methods.ts_ and give it some content like so:

```
function subtract(a:number, b:number) {
  return a - b;
}
```

Then run our command mentioning both files.

```
tsc variables.ts methods.ts --out dist.js
```

The resulting compiled file is this:

```
var a = 3;
function add(a, b) {
  return a + b;
}

var result = add(3, 3);
console.log(result);

function subtract(a, b) {
  return a - b;
}
```

As you can see using the —out flag enabled us to concatenate all the files.

> But what if I would like to use something from methods.ts in variables.ts. What then?

You need to use a reference command placed at the top of your file pointing out where to find methods.ts. Like so:

```
/// <reference path="methods.ts"/>
var subtractedResult = subtract(2,2);

var a = 3;
function add(a, b) {
  return a + b;
}

var result = add(3, 3);
console.log(result);
```

And this still compiles but now you can split up your project in many different parts.

> What if I have a large project and don't want to specify all the .ts manually, on the command line. Can't I just give it an input directory?

The short answer is no you can't from the command line. But there are several other options available for you. You could use **grunt** or **gulp** to compile your typescript or you could create **tsconfig.json** file where you setup input directory as well as output directory and other settings. Let's look at a tsconfig file :

```
{
  "compilerOptions": {
    "target": "es5"
  }
}
```

This is as simple as it can get and the whole purpose is to replace everything you type on the command line when compiling so you only have to type

```
tsc
```

Running tsc at this point however compiles each .ts file so it produces a variables.js and methods.js respectively.

> Not exactly what we used to have, how do we get everything compiled into one file again?

We need to add the **outFile** property so our **tsconfig.json** now looks like this:

```
{
  "compilerOptions": {
    "target": "es5",
    "outFile": "dist.js"
  }
}
```

One more thing to cover before presenting other constructs and that is source maps. Source maps is something you really want when you concate all the files into one js file. You want to show in the browser what it used to look like before compilation so that it is easy to debug your code in the browser.

Running a command that looks like this:

```
tsc variables.ts methods.ts -sourcemaps --out dist.js
```

Will give you source maps. Also —sourceMap will work as well.

> But as we started using tsconfig.json how to add source maps?

That's really easy just add sorceMaps property to **tsconfig.json** under _compilerOptions_.

```
"sourceMap": true
```

### Template strings

One thing that has been really annoying is how to concatenate strings. Most languages has some kind of format functions that does this for them. In javascript you usually resort to building your own utility function when you get tired of writing string like this.

```
var topic = 'products';
var id = 1;
var url = baseUrl + '/api/' + topic + '/' + id;
```

In ES2015 and typescript you can use something called template strings. Using the backtick ` we can write the above as:

```
var url = `${baseUrl}/${topic}/${id}`;
```

Much more readable and less error prone.

### Let and Const

First off is let so why let? Using the **var** keyword allowed you to create a variable but it had one great drawback, lack of _block scoping_ and thereby causing this problem

```
var value = 3;
if(true) {
  var value = 5:
}

console.log(value)  // prints 5
```

So for that reason **let** was created and ensures that a variable is block scoped like so:

```
let value = 3;
if(true) {
  let value = 5;
}

console.log( value );  // prints 3
```

When compiling to es5 typescript renames the inner value variable so that it does not collide with the one declared at the top.

The second concept was that of **const**. As expected it allows you to define a constant which is immutable in its nature. It forces you to initialize _and_ you cannot change it. Also a constant can be an object literal.

```
const PI = 3.14
const test // will give error as it is not initialized
PI = 3  // cannot be assigned to

const obj = {  test : 'ABC' }
obj.test = 'DEF' // allowed
obj = { test : 'new obj' } // not allowed, the top level reference is protected
```

### Arrow functions

One of the problems with javascript was that is was NOT easy as a developer to remember what **this** actually pointed to.

```
function Person() {
  this.name = 'test';
  this.data = {};

  this.getData(function(data){
    this.data = data;  // will be assigned to the inner functions `this`
  })
}
```

In this case **this.data** would be assigned to the wrong **this**. To solve this we would create a reference to the outher **this** calling it **me** or **that**, for example.

```
function Person() {
  this.name = 'test';
  this.data = {};
  var me = this;

  this.getData(function(data){
    me.data = data;  // will be assigned to the inner functions `this`
  })
}
```

By using something called **arrow functions** in typescript we no longer need to care as much about this problem.

```
function Person() {
  this.name = 'test';
  this.data = {};
  this.getData( (data) => {
    this.data = data;
  })
}
```

It actually creates a reference to the outer **this** behind the scenes, but we don't need to care anymore.

Hope you had a good read so far, stay tuned for part II.
