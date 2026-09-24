# Session Summary: Classes (JS Section 4)

*This is a summary of our conversation — not a replacement for your own notes.md. Use it as a reference while you write that up yourself. Note: constructors, extends/super, instanceof, and prototype-based inheritance were already covered in depth during the prototype chain session (JS Section 3) and are not repeated here.*

## Static methods and properties

```js
class Counter {
  static instances = 0; // static property

  constructor() {
    Counter.instances++;
  }

  static getInstanceCount() { // static method
    return Counter.instances;
  }
}

new Counter();
new Counter();
Counter.getInstanceCount(); // 2
```
**Key distinction:** static members belong to the class itself, not instances — `new Counter().getInstanceCount()` would throw an error, since instances have no direct access to static members. Useful for utility methods conceptually related to the class but not needing specific instance data, or tracking class-wide state (e.g. an instance counter).

## Private fields — `#field`

```js
class BankAccount {
  #balance = 0; // private field, accessible only inside this class

  deposit(amount) { this.#balance += amount; }
  getBalance() { return this.#balance; }
}

const account = new BankAccount();
account.deposit(100);
account.getBalance(); // 100
account.#balance;       // SyntaxError — genuinely inaccessible from outside
```

**Why this is genuinely different from the old convention-based approach (`this._balance`):** the underscore convention was purely a social contract — completely accessible and mutable from anywhere, just a signal "please don't touch this." `#balance` is enforced by the language itself — accessing it from outside is a hard syntax error, not a linting warning or a broken convention. A real, meaningful capability for building genuinely encapsulated APIs, especially relevant for library/component authoring where internal state must be protected from external corruption.

**Private methods work the same way:**
```js
class BankAccount {
  #balance = 0;
  #validateAmount(amount) { if (amount <= 0) throw new Error("Invalid amount"); }
  deposit(amount) { this.#validateAmount(amount); this.#balance += amount; }
}
```

## Getters/setters in classes

```js
class Temperature {
  #celsius = 0;
  get fahrenheit() { return this.#celsius * 9/5 + 32; }
  set fahrenheit(value) { this.#celsius = (value - 32) * 5/9; }
}
```
Same mechanism as Section 3's plain-object getters/setters, written with class syntax — often paired with a private field to enforce that external code can only interact through the computed public interface (`fahrenheit`), never the raw internal value (`#celsius`) directly.

## Class vs prototype-based patterns — the honest, current framing

**The nuance worth having precise for interviews:** class syntax started as pure sugar over the exact prototype mechanism covered in the prior session — `extends`, prototype-based methods, `instanceof` all map directly onto what already exists at the prototype level.

**But private fields are NOT just sugar** — they introduce a genuinely new capability the raw prototype-based approach never had. Before private fields, "private" state was only ever a naming convention (`_balance`) or a closure-based trick (defining the private variable inside a factory function's closure, never directly on the object) — never something the object model itself enforced.

**Honest interview framing:** "classes are syntax sugar over prototypes" is true for the core inheritance mechanism, but it's an oversimplification once private fields enter the picture — that's a genuine new language capability, not just a nicer way to write something already possible before.

## Where we left off

Section 4 (Classes) covered — static members, private fields/methods (with the genuine capability distinction from the old underscore convention), and getters/setters in class syntax. Everything else in this section (constructors, extends/super, instanceof, inheritance) was already thoroughly covered in the prototype chain deep dive during Section 3.

Next up per the JavaScript reference doc: **Section 5 — Arrays** (array literals, `Array.from()`/`of()`/`isArray()`, mutating vs non-mutating methods, the new ES2023 immutable array methods `toSorted`/`toReversed`/`toSpliced`/`with`, destructuring, spread with arrays, array-like objects vs true arrays, `Array.fromAsync`) — flagged as a recent-gap section given the newer immutable methods.
