# Code smells

Signs that code may be harder to change than it needs to be. A smell is a reason to look closer, not proof of a fault: report each one as "possible <smell>", never as a blocker, and drop it when a documented standard of the repo says otherwise.

- **Mysterious name**: a function, variable or module whose name doesn't say what it holds or does, so the reader has to open the body to find out → rename it after what it means to the caller; if no good name comes, it probably does too much.
- **Duplicated code**: the same logic, or nearly the same, written out in more than one place → keep one copy in a shared function and call it from each place.
- **Feature envy**: a function that works mostly with another module's or object's data rather than its own → move it, or the part that reaches over, next to the data it uses.
- **Data clumps**: the same few values travel together through many signatures or structures, such as `xmin, ymin, xmax, ymax` → group them into one type or record and pass that.
- **Primitive obsession**: plain strings, numbers or dicts standing in for an idea with rules of its own, such as a CRS, a date range or a band name → give the idea a small type that checks its rules in one place.
- **Repeated switches**: the same `if`/`elif` or `match` on the same value in several places, so every new case means editing all of them → keep the cases in one place: a lookup table, or one class per case.
- **Shotgun surgery**: one small change in behaviour needs edits scattered over many files → gather the code that changes together into one module.
- **Divergent change**: one module gets edited for many unrelated reasons, such as file format, storage and business rules → split it so each part changes for one reason.
- **Speculative generality**: hooks, parameters, options or abstract base classes added for needs nobody has yet → remove them until a real caller needs them.
- **Message chains**: code that walks a long chain through other objects, like `a.b().c().d`, and so depends on the shape of each → ask the first object for what you need, or add one method that hides the walk.
- **Middle man**: a class or function that only passes calls on to another one and adds nothing → call the real one directly and remove the go-between.
- **Refused bequest**: a subclass that inherits methods or data it doesn't use, or overrides them to do nothing → use composition instead of inheritance, or split the base class.
- **Dead or unused code**: functions, branches, parameters, imports or commented-out code that nothing reaches → delete it; version control keeps the history.
