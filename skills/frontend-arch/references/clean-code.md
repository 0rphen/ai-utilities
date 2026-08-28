# Frontend Architecture — Clean Code

Read this before writing or reviewing the code that lives *inside* any layer or component — placing a file in the right folder doesn't make what's written in it readable, testable, or safe to change.

## Meaningful Names

A name should say what something is or does, not force the reader to open the file to find out.

<!-- ✅ -->
```typescript
const maxRetries = 10;
const activeUsers = getActiveUsers();
```

<!-- ❌ never — a name that says nothing -->
```typescript
const d = 10;
const x = getData();
```

A good name removes the need for a comment explaining it.

---

## Small, Single-Purpose Functions

A function does one thing. Orchestration delegates to named steps instead of inlining every detail:

```typescript
function registerUser(input: RegisterUserInput): Promise<User> {
  validateRegistration(input);
  const user = createUser(input);
  return userRepository.save(user);
}
```

If `registerUser` grows a fourth and fifth responsibility (send a welcome email, log an analytics event), that's a signal to extract another named step, not to keep inlining.

---

## Single Responsibility, Inside a Layer Too

Placing a file in `application/` doesn't satisfy SRP by itself. A use case that validates input, persists an entity, *and* sends a notification has the same smell as code with no layers at all — it just has better real estate. See `references/layers.md` for the layer-to-layer version of this rule; this is the same principle one level down, inside a single file.

<!-- ❌ never — one use case doing three jobs -->
```typescript
class RegisterUserUseCase {
  async execute(input: RegisterUserInput) {
    if (!input.email.includes('@')) throw new Error('bad email');
    const user = await this.users.save(new User(input));
    await this.notifier.send(user.email, 'Welcome!');
    return user;
  }
}
```

<!-- ✅ -->
```typescript
class RegisterUserUseCase {
  constructor(
    private readonly users: UserRepository,
    private readonly notifier: NotificationGateway,
  ) {}

  async execute(input: RegisterUserInput): Promise<User> {
    const user = new User(input); // validation lives in the constructor/factory
    await this.users.save(user);
    await this.notifier.sendWelcome(user);
    return user;
  }
}
```

---

## Low Coupling, High Cohesion

`references/layers.md`'s dependency rule buys this between layers. The same idea applies between two files in the *same* layer: things that change together stay together; things that don't share a reason to change don't get bundled into one module just because they're both loosely "utils." A `shared/utils.ts` mixing money formatting, a debounce helper, and an API call is low cohesion even though nothing there crosses a layer boundary.

---

## DRY, Without Over-Abstracting

<!-- ❌ never — the same rule copied at two call sites -->
```typescript
if (user.age >= 18) { /* ... */ }
// ...
if (user.age >= 18) { /* ... */ }
```

<!-- ✅ -->
```typescript
function isAdult(user: User): boolean {
  return user.age >= 18;
}
```

Duplication is a signal to name a shared concept — it isn't a mandate to unify anything that merely looks similar today. Two functions that happen to have the same three lines but represent different business concepts should stay separate; forcing them into one shared function couples two things that should be free to change independently.

---

## KISS

Clean code isn't more classes, interfaces, and patterns — it's the minimum complexity the problem actually needs.

```typescript
if (user.isActive) {
  activateAccount(user);
}
```

This doesn't need a strategy pattern or an abstract factory. The skill's own "scale the architecture to the project" instruction makes this call at the folder level; KISS is the same judgment one level down, inside a single function or class.

---

## Comments Explain Why, Not What

<!-- ❌ never — a comment that restates the next line -->
```typescript
// Check if user is active
if (user.status === 'ACTIVE') { /* ... */ }
```

<!-- ✅ -->
```typescript
if (user.isActive()) { /* ... */ }

// Keep the previous token for 5 minutes: mobile clients retry
// the request without giving the server time to blacklist it.
```

A comment that explains what the next line does is noise the code should already carry in its own name. A comment that explains why a non-obvious decision was made is one of the few comments worth keeping.

---

## No Magic Numbers or Strings

<!-- ❌ never — an unexplained literal that means something specific -->
```typescript
if (user.role === 3) { /* ... */ }
if (order.statusCode === 'PEND') { /* ... */ }
```

<!-- ✅ -->
```typescript
const ADMIN_ROLE = 3;
if (user.role === ADMIN_ROLE) { /* ... */ }

enum OrderStatusCode {
  Pending = 'PEND',
  Confirmed = 'CONF',
}
if (order.statusCode === OrderStatusCode.Pending) { /* ... */ }
```

A bare `3` or `'PEND'` forces every reader to already know what it means; a named constant or enum makes the comparison self-explanatory and gives the compiler one place to catch a typo.

---

## Handle Errors Explicitly

<!-- ❌ never — a swallowed error, or an ambiguous null standing in for three different failures -->
```typescript
try {
  await save(order);
} catch {
  // ignore
}

function findUser(id: string): User | null {
  // null might mean "not found", "not authorized", or "deleted" — the caller can't tell which
}
```

<!-- ✅ -->
```typescript
try {
  await save(order);
} catch (err) {
  throw new OrderPersistenceError(order.id, { cause: err });
}

// or, in an architecture that models results explicitly:
function findUser(id: string): Result<User, UserNotFoundError> { /* ... */ }
```

An error is part of the design, not an afterthought — name the failure instead of letting `null` or a swallowed `catch` carry an ambiguous meaning.

---

## No Surprising Side Effects

A function's name is a promise to the caller. If `getUser()` also clears a cache and fires an analytics event, the name broke that promise.

<!-- ❌ never -->
```typescript
function getUser(): User {
  this.cache.clear();
  this.analytics.track('user_fetched');
  return this.user;
}
```

Split the read from the side effects, or rename the function to say what it actually does.

---

## One Level of Abstraction Per Function

Don't mix a business rule, a hashing call, a SQL/HTTP call, and an outbound notification in one function body — that's the inside-the-function version of the layering rule the skill already enforces between files.

<!-- ❌ never — four different levels of abstraction in one function -->
```typescript
function registerUser(input: RegisterUserInput) {
  if (!input.email.includes('@')) throw new Error('bad email'); // business rule
  const hash = hashPassword(input.password, 10);                 // cryptography
  db.query('INSERT INTO users ...', [input.email, hash]);        // persistence
  sendEmail(input.email, 'Welcome!');                             // notification
}
```

Each of those lines belongs to a different layer in `references/layers.md` — the fix is the same one: extract each concern behind its own named function or port, and let `registerUser` (or a use case) read as one level of abstraction: orchestration.

---

## Design for Testability

Constructor-injected dependencies over reaching for a concrete implementation inline — the same DI/ports idea `references/layers.md` already teaches for repositories, restated at function scope.

<!-- ✅ -->
```typescript
class RegisterUserUseCase {
  constructor(
    private readonly users: UserRepository,
    private readonly hasher: PasswordHasher,
  ) {}
}
```

<!-- ❌ never — dependencies constructed inline, impossible to fake in a test -->
```typescript
class RegisterUserUseCaseBad {
  register() {
    const db = new ConcreteDatabase();
    const hasher = new ConcretePasswordHasher();
    // ...
  }
}
```

## SOLID, Where It Earns Its Keep

SOLID isn't identical to Clean Code, but the two are complementary — and two of the five are ones this skill already leans on architecturally (SRP shows up throughout `references/layers.md`; DIP is exactly what "invert I/O through ports" means):

| Principle | Idea |
| --- | --- |
| **S** — Single Responsibility | One reason to change, per module/class/function. |
| **O** — Open/Closed | Open to extension, closed to modification. |
| **L** — Liskov Substitution | An implementation must be substitutable for the abstraction it satisfies. |
| **I** — Interface Segregation | Small, specific interfaces over one that forces unrelated methods on every implementer. |
| **D** — Dependency Inversion | Depend on abstractions (ports), not concrete implementations. |

Reach for these when they remove a real pain, not preemptively — the same judgment call as everywhere else in this skill.

## Boy Scout Rule

> Leave the code a little cleaner than you found it.

If a nearby change surfaces a bad name or an oversized function, fix it in passing — it doesn't need to wait for a dedicated refactor.

```typescript
// before, in code you're already touching for an unrelated fix
function calculate() {
  const x = /* ... */;
  const y = /* ... */;
}

// after — no bigger a change than what you were already making
function calculateTotal() {
  const subtotal = /* ... */;
  const tax = /* ... */;
}
```
