# CodeGenerator

Programmatically generate javascript.

## Usage
```
npm install js-code-generator
```


## Features
- Handles code indentation for you while allowing you to set the number of spaces and tabs.
- Provides helper method to generate unique names for iterator variables to make generating nested loops easier
- consistent function arguments and return values

## Example

```javascript
var Generate = require("./CodeGenerator.js");

var itrVar = Generate.variable({
  name: Generate.uniqueIteratorName(),
  value: "0"
});

var code =
  Generate.forLoop({
    startCondition: `${itrVar.data.name} = 0`,
    stopCondition: `${itrVar.data.name} < 10`,
    incrementAction: `${itrVar.data.name} += 1`,
    body: function() {
      return Generate.ifStatement({
        condition: `${itrVar.data.name} % 2 == 0`,
        body: function() {
          return Generate.consoleLog(["'even'"]).code;
        }
      }).code
    }
  }).code
  
console.log(code);

/* Output:
for (var i = 0; i < 10; i += 1) {
  if (i % 2 == 0) {
    console.log("even");
  }
}
*/

```

## Config
**numSpacesPerTab** (defaults to 4)

The number of spaces that make up 1 tab. Used for code indentation.
```javascript
CodeGenerator.numSpacesPerTab = 2;
```

**numTabs** (defaults to 1)

The number of spaces that make up 1 tab. Used for code indentation.
```javascript
CodeGenerator.numTabs = 2;
```

## Methods
**variable**
```javascript
/*
  @param Object data {
    {string} name,
    {string} value (optional)
  }
  
  @returns {
    data: data,
    code: code
  } 
*/

var example1 = Generate.variable({name: "test", value: "0"});
console.log(example1.code)
// var test = 0;

var example2 = Generate.variable({name: "test"});
console.log(example2.code)
// var test;
```

**reassignVariable**
```javascript
/*
  @param Object data {
    {string} name,
    {string} value
  }
  
  @returns {
    data: data,
    code: code
  } 
*/


var code = [];

var testVar = Generate.variable({name: "test"});
code.push(testVar.code);

var example = Generate.reassignVariable({name: testVar.data.name, value: "0"})
code.push(example.code);

console.log(code.join("\n"));

/* Output:
var test;
test = 0;
*/
```

**incrementVariable**
```javascript
/*
@param Object data {
  {string} name,
  {function} value (Optional)
}
*/


var code = [
  Generate.variable(name: "num", value: "0").code, 
  Generate.incrementVariable({name: numVar.data.name, value: "4"}).code,
  Generate.incrementVariable({name: numVar.data.name}).code,
]

console.log(code.join("\n"));

/* Output:
var num = 0;
num += 4;
num++
*/
```

**decrementVariable**
```javascript
/*
@param Object data {
  {string} name,
  {function} value (Optional)
}
*/


var code = [
  Generate.variable(name: "num", value: "0").code, 
  Generate.decrementVariable({name: numVar.data.name, value: "4"}).code,
  Generate.decrementVariable({name: numVar.data.name}).code,
]

console.log(code.join("\n"));

/* Output:
var num = 0;
num -= 4;
num--
*/
```

**firstClassFunction**
```javascript
/*
  @param Object data {
    {string} name,
    {array} args (Optional),
    {function} body
  }
  @returns {
    data: data,
    code: code
  } 
*/

var example = Generate.firstClassFunction({
  name: "func",
  args: ["a", "b", "c"],
  body: function() {
    return Generate.consoleLog("hi").code;
  }
})

console.log(example.code)

/* Output:
var func = function(a, b, c) {
  console.log("hi");
};
*/
```

**newInstance**
```javascript
  /*
    @param Object data {
      {string} type
      {array} args
    }

    @returns Object {
      {string} code
    }
  */

  var instance = Generate.newInstance({type: "Person", args: ["'Bob'", 30]});
  var personVar = Generate.variable({name: "person", value: instance.code});
  console.log(personVar.code);

  /* Output:
    var person = new Person('Bob', 30);
  */

```

**objectPropertyAssignment**
```javascript
/*
  @param Object data {
    {string} objName (Optional. Defaults to "this")
    {string} propName,
    {string} value,
    {boolean} dotNotation (Optional. Defaults to true)
  }
*/


var objVar = Generate.variable({
  name: "obj",
  value: {}
});

var example1 = Generate.objectPropertyAssignment({
  objName: objVar.data.name, 
  propName: "prop",
  value: "hello"
});

var example2 = Generate.objectPropertyAssignment({
  objName: objVar.data.name, 
  propName: "prop",
  value: "hello",
  dotNotation: false
});

var code = [
  objVar.code,
  example1.code,
  example2.code
].join("\n")

console.log(code);

/* Output:
var obj = {};
obj.prop = "hello";
obj["prop"] = "hello";
*/
```

**objectFunctionCall**
```javascript
/*
  @param Object data {
    {string} objName (Optional. Defaults to "this")
    {string} funcName,
    {array} args (Optional)
  }
*/

var example = Generate.objectFunctionCall({
  objName: "Math",
  funcName: "floor",
  args: [3.5]
})

console.log(example.code);

/* Output:
Math.floor(3.5);
*/
```

**objectFunction**
```javascript
/*
  @param Object data {
    {string} objName (Optional. Defaults to "this")
    {string} funcName,
    {array} args (Optional),
    {function} body
  }
*/

var example = Generate.objectFunction({
  objName: "obj",
  funcName: "sumNums",
  args: ["nums"],
  body: function() {
    return Generate.comment({text: "code to sum elements of nums array"}).code
  }
})

console.log(example.code);

/* Output:
obj.sumNums = function(nums) {
  // code to sum elemens of nums array 
}
*/
```

**ifStatement**
```javascript
/*
@param Object data {
  {string} condition,
  {function} body
}
*/

var numVar = Generate.variable(name: "num", value: "0");

var example = Generate.ifStatement({
  condition: `${numVar.data.name} > 2`,
  body: function() {
    let body = [
      Generate.incrementVariable({name: numVar.data.name, value: "4"}).code,
      Generate.incrementVariable({name: numVar.data.name}).code,
    ];

    return body.join("\n");
  }
});

var code = [
  numVar.code,
  example.code
].join("\n");

console.log(code)

/* Output:
var num = 0;
if (num > 2) {
  numVar += 4;
  numVar++
}
*/
```

**elseIfStatement**

**elseStatement**

**tryBlock**

**catchBlock**





**forLoop**
```javascript
/*
  @param Object data {
    {string} startCondition,
    {string} stopCondition,
    {string} incrementAction,
    {function} body
  }
*/

var nestedForLoop = 
  Generate.forLoop({
    startCondition: `${itrVar.data.name} = 0`,
    stopCondition: `${itrVar.data.name} < 10`,
    incrementAction: `${itrVar.data.name} += 1`,
    body: function() {
      let itrVar = Generate.variable({
        name: Generate.uniqueIteratorName(),
        value: "0"
      });

      return (
        Generate.forLoop({
          startCondition: `${itrVar.data.name} = 0`,
          stopCondition: `${itrVar.data.name} < 10`,
          incrementAction: `${itrVar.data.name} += 1`,
          body: function() {
            let itrVar = Generate.variable({
              name: Generate.uniqueIteratorName(),
              value: "0"
            });

            return (
              Generate.forLoop({
                startCondition: `${itrVar.data.name} = 0`,
                stopCondition: `${itrVar.data.name} < 10`,
                incrementAction: `${itrVar.data.name} += 1`,
                body: function() {
                  return "";
                }
              }).code
            )
          }
        }).code
      )
    }
  }).code

console.log(nestedForLoop);

/* Output:
for (var i = 0; i < 10; i += 1) {
  for (var j = 0; j < 10; j += 1) {
    for (var k = 0; k < 10; k += 1) {

    }
  }
}
*/
```