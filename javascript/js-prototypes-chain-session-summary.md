# Session Summary: Prototypes, Prototype Chain, Object.create() (closes JS Section 3)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself.*

## The core mechanism

```js
const obj = { name: "Alex" };
obj.toString(); // "[object Object]" — works even though obj never defined this method
```
Every object has an internal, hidden link to another object called its **prototype**. When accessing a property/method not found directly on the object, JavaScript automatically walks up the chain — prototype, then the prototype's prototype, and so on — until found or the chain runs out.

```
obj  →  Object.prototype  →  null
```
`Object.prototype`'s own prototype is `null` — end of the chain, where lookup stops and returns `undefined` if nothing was found.

## Viewing and manipulating the chain

```js
Object.getPrototypeOf(obj); // Object.prototype
obj.__proto__;                // same idea, older/informal — recognize but avoid in real code

const animal = { makeSound() { console.log("..."); } };
const dog = Object.create(animal); // creates a new object with animal as ITS prototype
dog.makeSound(); // "..." — dog doesn't have this method, its prototype does
```
`Object.create(proto)` is the explicit, direct way to create an object with a specific prototype — the clearest way to see the mechanism, wiring the chain manually.

## Property lookup — the actual algorithm

```js
const animal = { type: "generic", makeSound() { console.log("..."); } };
const dog = Object.create(animal);
dog.name = "Rex";

dog.name;         // "Rex" — found directly ON dog, stop immediately
dog.type;          // "generic" — not on dog, check prototype (animal), found there
dog.makeSound();   // not on dog, check prototype, found there, called
dog.nonExistent;   // undefined — not found anywhere, chain ends at null
```
Check the object itself first; if not found, check prototype, then prototype's prototype, until found or chain ends at `null`.

**Critical nuance — lookup only applies to READING, not writing:**
```js
dog.type = "canine";     // creates a NEW property directly ON dog
dog.type;                 // "canine" — dog's own property now
animal.type;               // "generic" — completely unaffected
```
Assigning a property always creates/modifies it directly on the object being assigned to — never walks up the chain to modify a prototype's property. A common point of confusion, since reading and writing behave fundamentally differently here.

## Defining/overriding a method on an object (e.g. custom `toString`) — shadowing, not conflict

```js
const obj = {
  name: "Alex",
  toString() { return `Person: ${this.name}`; }
};
obj.toString(); // "Person: Alex" — YOUR version runs
```
Nothing conflicts. Lookup checks the object itself first — since `toString` is defined directly on `obj`, the lookup finds and stops there, never reaching `Object.prototype`'s version.

**Real practical use — `toString` is auto-invoked for implicit string conversion:**
```js
const person = { name: "Alex", toString() { return `Person(${this.name})`; } };
`User: ${person}`;   // "User: Person(Alex)" — template literal triggers toString automatically
person + "";           // "Person(Alex)" — string concatenation triggers it too
String(person);        // "Person(Alex)" — explicit conversion uses it too
```
Without the custom `toString`, all three would print `[object Object]` (the `Object.prototype` default).

**The original method is untouched, only shadowed — still explicitly accessible:**
```js
Object.getPrototypeOf(obj).toString();  // still "[object Object]" — original untouched
Object.prototype.toString.call(obj);     // explicitly borrowing the original — ties back to call() from Section 2
```

**Overriding on one instance doesn't affect the prototype or other instances:**
```js
const arr = [1, 2, 3];
arr.toString = function() { return "custom array string"; };
arr.toString();       // "custom array string" — shadows Array.prototype's version, only for THIS array
[4, 5].toString();     // "4,5" — a different array, unaffected
```
To change behavior for ALL arrays would require modifying `Array.prototype.toString` itself directly — a real capability but strongly discouraged ("monkey-patching a built-in prototype"), since it causes confusing, hard-to-trace bugs in any other code relying on original behavior.

## Classes — syntax sugar over exactly this mechanism

```js
class Animal {
  constructor(name) { this.name = name; }
  makeSound() { console.log("..."); }
}
const dog = new Animal("Rex");
```
Functionally near-identical to:
```js
function Animal(name) { this.name = name; }
Animal.prototype.makeSound = function() { console.log("..."); };
const dog = new Animal("Rex");
```

**The genuinely important insight:** `makeSound` is not copied onto every instance — it lives once on `Animal.prototype`, and every instance's prototype points to that same shared object. A real memory/performance detail: a million `Animal` instances still share one single copy of `makeSound` via the prototype chain, not a million separate copies.

```js
Object.getPrototypeOf(dog) === Animal.prototype; // true
dog.hasOwnProperty('name');       // true — set directly via this.name in constructor
dog.hasOwnProperty('makeSound');   // false — lives on the prototype, not the instance
```
`hasOwnProperty()` checks only the object itself, ignoring the prototype chain entirely — answers "does this specific object have this property, or is it inherited."

## `extends`/`super` — chaining prototypes together

```js
class Dog extends Animal {
  constructor(name, breed) {
    super(name);       // calls Animal's constructor, sets this.name
    this.breed = breed;
  }
  makeSound() {
    super.makeSound(); // calls Animal's version first
    console.log("Woof!");
  }
}
```
`extends` sets `Dog.prototype`'s own prototype to `Animal.prototype` — extending the chain by one more link. `super()` calls the parent constructor; `super.methodName()` calls the parent's version of that method. Just the prototype chain, one level deeper.

## `instanceof` — what it actually checks

```js
dog instanceof Dog;      // true
dog instanceof Animal;   // true — Animal.prototype is further up the chain
dog instanceof Object;   // true — everything eventually chains up to Object.prototype
```
Walks the actual prototype chain checking whether the given constructor's `.prototype` appears anywhere in it — not "was this literally created by this exact class," but "does this class's prototype exist somewhere in this object's chain." This is why it correctly returns `true` for every ancestor class, not just the immediate one.

## Where we left off

This closes out **JS Section 3: Objects** entirely across two sessions — literals/shorthand/computed properties, the key `Object.*` methods, getters/setters/property descriptors (with the Vue 2 `defineProperty` vs Vue 3 `Proxy` connection), and a full prototype chain deep dive (lookup algorithm, read-vs-write distinction, method shadowing/overriding, classes as prototype sugar, `extends`/`super`, `instanceof`).

Next up per the JavaScript reference doc: **Section 4 — Classes** (class declarations/expressions, constructors, instance/static methods and properties, private fields `#field`, getters/setters in classes, inheritance — largely already covered via the prototype chain session — `instanceof` — also covered — class vs prototype-based patterns).
