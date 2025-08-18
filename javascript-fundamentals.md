# JavaScript Fundamentals - Complete Guide

## Table of Contents
1. [Closures](#closures)
2. [Scope and Hoisting](#scope-and-hoisting)
3. [This Keyword](#this-keyword)
4. [Prototypes and Inheritance](#prototypes-and-inheritance)
5. [Promises and Async/Await](#promises-and-asyncawait)
6. [ES6+ Features](#es6-features)
7. [Common Patterns](#common-patterns)

---

## Closures

### What are Closures?

A **closure** is a function that has access to variables in its outer (enclosing) lexical scope even after the outer function has returned. In other words, a closure allows a function to "remember" and access variables from its outer scope even when the function is executed in a different scope.

### Basic Closure Example

```javascript
function outerFunction(x) {
  // This is the outer scope
  const outerVariable = "I'm from outer function";
  
  function innerFunction(y) {
    // This is the inner scope - it's a closure
    console.log(outerVariable); // Can access outerVariable
    console.log(x); // Can access parameter x
    console.log(y); // Can access its own parameter y
  }
  
  return innerFunction;
}

const closure = outerFunction("Hello");
closure("World");
// Output:
// "I'm from outer function"
// "Hello"
// "World"
```

### How Closures Work

1. **Lexical Scoping**: JavaScript uses lexical scoping, meaning inner functions have access to variables in their outer scope
2. **Function Creation**: When a function is created, it captures (remembers) the variables in its outer scope
3. **Execution**: When the function is executed later, it still has access to those captured variables

### Practical Examples

#### 1. Counter with Closure

```javascript
function createCounter() {
  let count = 0; // Private variable
  
  return {
    increment: function() {
      count++;
      return count;
    },
    decrement: function() {
      count--;
      return count;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.getCount()); // 0
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1
console.log(counter.getCount()); // 1

// Each counter has its own private count variable
const counter2 = createCounter();
console.log(counter2.getCount()); // 0 (separate from counter1)
```

#### 2. Data Privacy

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance; // Private variable
  
  return {
    deposit: function(amount) {
      if (amount > 0) {
        balance += amount;
        return `Deposited $${amount}. New balance: $${balance}`;
      }
      return "Invalid deposit amount";
    },
    withdraw: function(amount) {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        return `Withdrew $${amount}. New balance: $${balance}`;
      }
      return "Insufficient funds or invalid amount";
    },
    getBalance: function() {
      return balance;
    }
  };
}

const account = createBankAccount(1000);
console.log(account.getBalance()); // 1000
console.log(account.deposit(500)); // "Deposited $500. New balance: $1500"
console.log(account.withdraw(200)); // "Withdrew $200. New balance: $1300"
// console.log(balance); // Error: balance is not accessible from outside
```

#### 3. Function Factory

```javascript
function multiply(x) {
  return function(y) {
    return x * y;
  };
}

const multiplyByTwo = multiply(2);
const multiplyByTen = multiply(10);

console.log(multiplyByTwo(5)); // 10
console.log(multiplyByTen(5)); // 50
```

#### 4. Event Handlers

```javascript
function createButtonHandler(buttonId) {
  let clickCount = 0;
  
  return function() {
    clickCount++;
    console.log(`Button ${buttonId} clicked ${clickCount} times`);
  };
}

const button1Handler = createButtonHandler("btn1");
const button2Handler = createButtonHandler("btn2");

button1Handler(); // "Button btn1 clicked 1 times"
button1Handler(); // "Button btn1 clicked 2 times"
button2Handler(); // "Button btn2 clicked 1 times"
```

### Closures in React Hooks

Closures are fundamental to how React hooks work. Let's look at some examples:

#### useState Hook (Simplified)

```javascript
function useState(initialValue) {
  let state = initialValue;
  
  const setState = function(newValue) {
    state = newValue;
    // Trigger re-render logic here
  };
  
  return [state, setState];
}

// Usage
const [count, setCount] = useState(0);
```

#### useEffect Hook (Simplified)

```javascript
function useEffect(callback, dependencies) {
  // Store the callback in closure
  const effect = function() {
    // Cleanup previous effect
    if (effect.cleanup) {
      effect.cleanup();
    }
    
    // Run the effect
    const cleanup = callback();
    effect.cleanup = cleanup;
  };
  
  // Run effect when dependencies change
  effect();
}
```

### Common Closure Patterns

#### 1. Module Pattern

```javascript
const calculator = (function() {
  // Private variables and functions
  let result = 0;
  
  function add(a, b) {
    return a + b;
  }
  
  function subtract(a, b) {
    return a - b;
  }
  
  // Public API
  return {
    add: function(a, b) {
      result = add(a, b);
      return result;
    },
    subtract: function(a, b) {
      result = subtract(a, b);
      return result;
    },
    getResult: function() {
      return result;
    }
  };
})();

console.log(calculator.add(5, 3)); // 8
console.log(calculator.subtract(10, 4)); // 6
console.log(calculator.getResult()); // 6
```

#### 2. Partial Application

```javascript
function partial(fn, ...args) {
  return function(...moreArgs) {
    return fn(...args, ...moreArgs);
  };
}

function add(a, b, c) {
  return a + b + c;
}

const addFive = partial(add, 5);
const addFiveAndThree = partial(add, 5, 3);

console.log(addFive(3, 2)); // 10 (5 + 3 + 2)
console.log(addFiveAndThree(2)); // 10 (5 + 3 + 2)
```

### Common Pitfalls

#### 1. Loop Variable Capture

```javascript
// ❌ Problem: All functions capture the same i variable
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // All print 3
  }, 1000);
}

// ✅ Solution 1: Use let (block scope)
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // Prints 0, 1, 2
  }, 1000);
}

// ✅ Solution 2: Use IIFE (Immediately Invoked Function Expression)
for (var i = 0; i < 3; i++) {
  (function(index) {
    setTimeout(function() {
      console.log(index); // Prints 0, 1, 2
    }, 1000);
  })(i);
}
```

#### 2. Memory Leaks

```javascript
// ❌ Potential memory leak
function createHeavyObject() {
  const heavyData = new Array(1000000).fill('data');
  
  return function() {
    console.log(heavyData.length); // heavyData stays in memory
  };
}

// ✅ Better: Clear references when done
function createHeavyObject() {
  const heavyData = new Array(1000000).fill('data');
  
  return function() {
    console.log(heavyData.length);
    // Clear reference when done
    heavyData.length = 0;
  };
}
```

### Benefits of Closures

1. **Data Privacy**: Variables can be kept private
2. **State Preservation**: Functions can maintain state between calls
3. **Function Factory**: Create functions with preset parameters
4. **Module Pattern**: Create private and public APIs
5. **Event Handling**: Maintain context in event handlers

### Performance Considerations

- Closures keep references to outer variables in memory
- Be mindful of memory usage with large objects
- Consider clearing references when no longer needed
- Use `let` instead of `var` in loops to avoid unintended closures

---

## Scope and Hoisting

### Variable Scope

```javascript
// Global scope
const globalVar = "I'm global";

function outerFunction() {
  // Function scope
  const functionVar = "I'm in function scope";
  
  if (true) {
    // Block scope (ES6+)
    let blockVar = "I'm in block scope";
    var functionScopedVar = "I'm function-scoped";
  }
  
  // console.log(blockVar); // Error: blockVar is not accessible
  console.log(functionScopedVar); // Works: var is function-scoped
}

console.log(globalVar); // Works
// console.log(functionVar); // Error: functionVar is not accessible
```

### Hoisting

```javascript
// Function declarations are hoisted
hoistedFunction(); // Works: "Hello from hoisted function"

function hoistedFunction() {
  console.log("Hello from hoisted function");
}

// Variable declarations are hoisted, but not initializations
console.log(hoistedVar); // undefined (not ReferenceError)
var hoistedVar = "I'm hoisted";

// let and const are hoisted but not initialized (Temporal Dead Zone)
// console.log(notHoisted); // ReferenceError
let notHoisted = "I'm not hoisted";
```

---

## This Keyword

### Context Binding

```javascript
const person = {
  name: "John",
  greet: function() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

person.greet(); // "Hello, I'm John"

// Context loss
const greetFunction = person.greet;
greetFunction(); // "Hello, I'm undefined"

// Solutions
greetFunction.call(person); // "Hello, I'm John"
greetFunction.apply(person); // "Hello, I'm John"
const boundGreet = greetFunction.bind(person);
boundGreet(); // "Hello, I'm John"
```

### Arrow Functions and This

```javascript
const obj = {
  name: "Object",
  regularMethod: function() {
    console.log(this.name);
  },
  arrowMethod: () => {
    console.log(this.name); // undefined (this refers to global scope)
  }
};

obj.regularMethod(); // "Object"
obj.arrowMethod(); // undefined
```

---

## Prototypes and Inheritance

### Prototype Chain

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(`${this.name} barks!`);
};

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak(); // "Buddy makes a sound"
dog.bark(); // "Buddy barks!"
```

### ES6 Classes

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  
  bark() {
    console.log(`${this.name} barks!`);
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.speak(); // "Buddy makes a sound"
dog.bark(); // "Buddy barks!"
```

---

## Promises and Async/Await

### Promises

```javascript
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = Math.random() > 0.5;
      if (success) {
        resolve("Data fetched successfully");
      } else {
        reject("Failed to fetch data");
      }
    }, 1000);
  });
}

fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Promise.all
const promises = [
  fetchData(),
  fetchData(),
  fetchData()
];

Promise.all(promises)
  .then(results => console.log("All succeeded:", results))
  .catch(error => console.error("One failed:", error));
```

### Async/Await

```javascript
async function fetchUserData() {
  try {
    const response = await fetch('https://api.example.com/user');
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
}

// Usage
async function displayUser() {
  try {
    const user = await fetchUserData();
    console.log(user);
  } catch (error) {
    console.error('Failed to display user:', error);
  }
}
```

---

## ES6+ Features

### Destructuring

```javascript
// Array destructuring
const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(second); // 2
console.log(rest); // [3, 4, 5]

// Object destructuring
const person = { name: "John", age: 30, city: "NYC" };
const { name, age, country = "USA" } = person;
console.log(name); // "John"
console.log(country); // "USA" (default value)
```

### Template Literals

```javascript
const name = "John";
const age = 30;
const message = `Hello, my name is ${name} and I'm ${age} years old.`;
console.log(message); // "Hello, my name is John and I'm 30 years old."
```

### Spread and Rest Operators

```javascript
// Spread operator
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

const obj1 = { name: "John" };
const obj2 = { ...obj1, age: 30 }; // { name: "John", age: 30 }

// Rest operator
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4, 5)); // 15
```

---

## Common Patterns

### Singleton Pattern

```javascript
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      data: "I'm a singleton",
      getData: function() {
        return this.data;
      }
    };
  }
  
  return {
    getInstance: function() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2); // true
```

### Factory Pattern

```javascript
function createUser(type, name) {
  switch (type) {
    case 'admin':
      return {
        name,
        role: 'admin',
        permissions: ['read', 'write', 'delete']
      };
    case 'user':
      return {
        name,
        role: 'user',
        permissions: ['read']
      };
    default:
      throw new Error('Invalid user type');
  }
}

const admin = createUser('admin', 'John');
const user = createUser('user', 'Jane');
```

### Observer Pattern

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
}

const emitter = new EventEmitter();
emitter.on('userLogin', (user) => {
  console.log(`User ${user.name} logged in`);
});

emitter.emit('userLogin', { name: 'John' }); // "User John logged in"
```

---

## Summary

**Key JavaScript Concepts:**

1. **Closures** - Functions that remember their outer scope
2. **Scope** - Variable accessibility rules
3. **Hoisting** - Declaration lifting behavior
4. **This** - Context binding in functions
5. **Prototypes** - Object inheritance mechanism
6. **Promises** - Asynchronous programming
7. **ES6+ Features** - Modern JavaScript syntax
8. **Design Patterns** - Reusable code solutions

**Best Practices:**
- Use `const` and `let` instead of `var`
- Understand closure memory implications
- Use arrow functions for concise syntax
- Leverage destructuring for cleaner code
- Handle promises properly with try/catch
- Follow consistent naming conventions

**Remember**: JavaScript is a powerful language with many nuances. Understanding these fundamentals will make you a better developer! 