
# 🚀 Ultimate JavaScript Methods Cheat Sheet
*Your complete, battle-tested reference for Array, Object, String, Number, Math, Date & Global methods. Save this note — you'll use it daily.*

> [!TIP] Pro Tip
> **Bookmark this note** in Obsidian. Use `Ctrl/Cmd+P` → "Open Quick Switcher" to jump here instantly during coding sessions. Pair with `#javascript #cheatsheet #reference` tags!

---

## 🔁 1. ARRAY METHODS (Most Important & Frequently Used)

### 🔄 Mutating Methods (modify original array)
```js
// 1. push() - add to end
let fruits = ['apple', 'banana'];
fruits.push('orange', 'mango');
console.log(fruits); // ['apple', 'banana', 'orange', 'mango']

// 2. pop() - remove from end
let last = fruits.pop();
console.log(last);    // 'mango'
console.log(fruits);  // ['apple', 'banana', 'orange']

// 3. unshift() - add to beginning
fruits.unshift('grape');
console.log(fruits); // ['grape', 'apple', 'banana', 'orange']

// 4. shift() - remove from beginning
let first = fruits.shift();
console.log(first);   // 'grape'

// 5. splice(start, deleteCount, ...items) - surgical edits
let removed = fruits.splice(1, 2, 'kiwi', 'pineapple');
console.log(fruits);   // ['apple', 'kiwi', 'pineapple'] 
console.log(removed);  // ['banana', 'orange']

// 6. sort() - IN-PLACE sort (⚠️ strings by default!)
let nums = [10, 5, 100, 2];
nums.sort((a, b) => a - b); // Numeric ascending
console.log(nums); // [2, 5, 10, 100]

// 7. reverse()
fruits.reverse();
console.log(fruits); // ['pineapple', 'kiwi', 'apple']

// 8. fill(value, start, end)
let arr = [1, 2, 3, 4, 5];
arr.fill(0, 2, 4);
console.log(arr); // [1, 2, 0, 0, 5]
```

### 📦 Non-Mutating Methods (return new array)
```js
// 9. concat()
let arr1 = [1, 2];
let arr2 = [3, 4];
let combined = arr1.concat(arr2, [5, 6]);
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 10. slice(start, end) - shallow copy
let part = fruits.slice(1, 3);
console.log(part); // ['kiwi', 'apple']

// 11. map() - transform every element
let numbers = [1, 2, 3, 4];
let doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8]

// 12. filter() - keep matching elements
let evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4]

// 13. reduce() - accumulate values (MOST POWERFUL)
let sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum); // 10

// Advanced reduce patterns:
let max = numbers.reduce((a, b) => Math.max(a, b)); // 4
let grouped = ['a','b','a','c','b'].reduce((obj, char) => {
  obj[char] = (obj[char] || 0) + 1;
  return obj;
}, {}); // { a:2, b:2, c:1 }

// 14. flat(depth) & flatMap()
let nested = [1, [2, [3, 4]]];
console.log(nested.flat(2)); // [1, 2, 3, 4]

let sentences = ['Hello world', 'JS is fun'];
let words = sentences.flatMap(s => s.split(' '));
console.log(words); // ['Hello', 'world', 'JS', 'is', 'fun']
```

### 🔍 Search & Check Methods
```js
// 15. find() / findIndex()
let users = [{id:1, name:'John'}, {id:2, name:'Jane'}];
let user = users.find(u => u.id === 2); // {id:2, name:'Jane'}
let idx = users.findIndex(u => u.name === 'John'); // 0

// 16. includes() / indexOf() / lastIndexOf()
console.log(fruits.includes('apple')); // true
console.log(fruits.indexOf('kiwi'));   // 1

// 17. some() / every()
let hasEven = numbers.some(n => n % 2 === 0); // true
let allPositive = numbers.every(n => n > 0);  // true
```

### 🧰 Other Essential Methods
```js
// 18. forEach() - iterate (no return)
fruits.forEach((fruit, i) => console.log(`${i}: ${fruit}`));

// 19. entries() / keys() / values()
for (let [i, val] of fruits.entries()) {
  console.log(i, val);
}

// 20. copyWithin(target, start, end)
[1,2,3,4,5].copyWithin(0, 3); // [4,5,3,4,5]

// 21. join() / toString()
console.log(fruits.join(' | ')); // "pineapple | kiwi | apple"
```

---

## 📦 2. OBJECT METHODS (ES6+)

```js
const person = { name: 'Alice', age: 25, city: 'Paris' };

// 1. Keys/Values/Entries (LOOPING ESSENTIALS)
Object.keys(person);    // ['name', 'age', 'city']
Object.values(person);  // ['Alice', 25, 'Paris']
Object.entries(person); // [['name','Alice'], ...]

// Loop pattern (BEST PRACTICE):
for (let [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}

// 2. Cloning & Merging
const clone = Object.assign({}, person);       // Legacy shallow clone
const clone2 = { ...person };                  // ✅ Modern preferred
const updated = { ...person, age: 26, job: 'Dev' };

// 3. Immutability Helpers
Object.freeze(person);   // 🔒 No modifications allowed
Object.seal(person);     // 🔒 No add/delete, but existing props mutable
Object.isFrozen(person); // true

// 4. Property Checks
Object.hasOwn(person, 'name'); // ✅ Better than .hasOwnProperty()
Object.getOwnPropertyNames(person); // Includes non-enumerable props

// 5. Entries ↔ Object Conversion
const entries = [['a', 1], ['b', 2]];
const obj = Object.fromEntries(entries); // { a:1, b:2 }

// REAL-WORLD: URLSearchParams → Object
const params = new URLSearchParams('name=john&age=25');
const userObj = Object.fromEntries(params); 
// { name: 'john', age: '25' }
```

---

## 🔤 3. STRING METHODS

```js
let str = "   Hello JavaScript World!   ";

// ✂️ Trimming & Case
str.trim();        // "Hello JavaScript World!"
str.trimStart();   // "Hello JavaScript World!   "
str.toLowerCase(); // "   hello javascript world!   "
str.toUpperCase();

// 🔎 Search
str.includes('Java');    // true
str.startsWith('Hello'); // false (has spaces!)
str.endsWith('!');       // false (has spaces!)
str.indexOf('o');        // 7 (first 'o' in "Hello")
str.match(/Java\w+/);    // ['JavaScript']

// ✂️ Slicing
str.slice(6, 16);      // "JavaScript" (after trim!)
"Mozilla".substring(2,5); // "zil" (start/end order insensitive)
"Mozilla".substr(2, 3);   // "zil" (⚠️ Deprecated - avoid)

// 🔄 Replace & Split
"foo foo".replace('foo', 'bar');    // "bar foo" (first only)
"foo foo".replaceAll('foo', 'bar'); // "bar bar" ✅
str.split(' '); // ['','Hello','JavaScript','World!',''] (trim first!)

// 📏 Padding & Repeat
"5".padStart(3, '0'); // "005"
"5".padEnd(3, '0');   // "500"
"Ha".repeat(3);       // "HaHaHa"

// 🌐 Encoding (CRITICAL FOR URLS)
encodeURIComponent("john@example.com"); // "john%40example.com"
```

---

## 🔢 4. NUMBER & MATH METHODS

### Number Static Methods
```js
Number.isInteger(42);      // true
Number.isNaN(NaN);         // true (✅ safe vs global isNaN)
Number.parseInt("42px");   // 42
Number.parseFloat("3.14"); // 3.14
```

### Number Instance Methods
```js
let num = 123.456;
num.toFixed(2);      // "123.46" (string, rounds)
num.toPrecision(4);  // "123.5" (significant digits)
```

### Math Utilities
```js
Math.max(1,5,3);     // 5
Math.min(1,5,3);     // 1
Math.abs(-5);        // 5
Math.pow(2, 10);     // 1024
Math.sqrt(16);       // 4
Math.round(4.6);     // 5
Math.ceil(4.1);      // 5
Math.floor(4.9);     // 4
Math.trunc(4.9);     // 4 (removes decimal)

// 🎲 Random integer (1-10)
Math.floor(Math.random() * 10) + 1;

// Constants
Math.PI, Math.E, Math.LN2, Math.LOG10E
```

---

## 📅 5. DATE METHODS

```js
const now = new Date();

// 📅 Creation
new Date();                     // Current date/time
new Date('2025-12-25');         // ISO string
new Date(2025, 11, 25);         // ⚠️ Month is 0-indexed! (Dec = 11)
new Date(2025, 11, 25, 14, 30); // With time

// 🔍 Getters
now.getFullYear(); // 2026
now.getMonth();    // 0-11 (⚠️ Jan=0)
now.getDate();     // Day of month (1-31)
now.getDay();      // 0-6 (Sun=0)
now.getHours();    // 0-23
now.getTime();     // Milliseconds since Unix epoch

// 🛠️ Setters
now.setFullYear(2030);
now.setDate(now.getDate() + 7); // Next week

// 🌍 Formatting (CRITICAL FOR UX)
now.toISOString(); // "2026-02-13T15:30:45.123Z"
now.toLocaleDateString('en-US'); // "2/13/2026"
now.toLocaleDateString('en-GB'); // "13/02/2026"
now.toLocaleString('fr-FR', { 
  dateStyle: 'full', 
  timeStyle: 'short' 
}); // "jeudi 13 février 2026 à 15:30"
```

---

## 🌐 6. GLOBAL / UTILITY METHODS

```js
// 🔢 Parsing
parseInt("42px");    // 42
parseFloat("3.14");  // 3.14

// ✅ Validation
isNaN("hello");      // true (⚠️ coerces! Prefer Number.isNaN)
isFinite(123);       // true
Number.isFinite(Infinity); // false ✅

// 🔗 URL Encoding (ESSENTIAL FOR APIs)
encodeURI("https://example.com?name=John Doe"); 
// "https://example.com?name=John%20Doe"
encodeURIComponent("John Doe"); // "John%20Doe" ✅ Use for query params

// ⏱️ Timing
const id = setTimeout(() => console.log("Run once"), 1000);
clearTimeout(id);
setInterval(() => console.log("Tick"), 2000);

// 📦 JSON
const jsonStr = JSON.stringify({ name: "Alice" });
const obj = JSON.parse(jsonStr);

// 🖨️ Console (Pro Tips)
console.log("Debug");
console.table([{a:1}, {a:2}]); // Beautiful table view
console.warn("Warning!");
console.error("Error!");
console.time("op"); /* ... */ console.timeEnd("op"); // Performance
```

---

> [!SUCCESS] You're Now Equipped!
> This covers **99% of real-world JavaScript method usage**.  
> 💡 **Pro Workflow**:  
> 1. Tag this note `#javascript #reference`  
> 2. Create a Dataview query to link related notes:  
>    ````markdown
>    ```dataview
>    list from #javascript and !"Cheat Sheet"
>    ```
>    ````  
> 3. Use Obsidian's **Command Palette** → "Insert Link to Current Note" when documenting project-specific patterns!  
>   
> *Master these, and you'll debug faster, write cleaner code, and stand out in interviews. You've got this!* 🌟  
> *Last verified: {{date:YYYY-MM-DD}} | JS Engine: ECMAScript 2025 (ES16)*
