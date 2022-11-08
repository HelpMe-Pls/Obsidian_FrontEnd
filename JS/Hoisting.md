- The term "hoisting" is just to simplify the understanding of the "lexical scope" in a way that the majority can get it: think of hoisting as a *visualization* of various actions JS takes in setting up the program **before execution** (compile-time operation of generating runtime instructions). So there's no such thing as "hoisting" officially.

- `let` and `const` does "hoist" but it a different way: when `var` is "hoisted", it actually assigns `undefined` to the variable at compile time (in the lexical scope), where `let` and `const` are "uninitialized" until the execution stack run into that `let` or `const` declaration, which causes the [TDZ error](https://www.freecodecamp.org/news/what-is-the-temporal-dead-zone). That's why it's the best practice to ==always declare your variables before using them==.

- `var` *declarations* can be re-declared (although it's a no-op) where `let` and `const` can't.

- "Hoisting" function ***declarations*** is a good practice because it improves readability.

- Variables are created when their containing lexical environment is instantiated (i.e. when the lexical binding is evaluated, entering the "compile time" phase), that's why *the function expressions are **not** "hoisted"*, because they are evaluated at run time. Therefore, always ==define your function *expressions* **before** you call them.