# Frontend Architecture — Clean Code

Read this before writing or reviewing the code that lives *inside* any layer or component — placing a file in the right folder doesn't make what's written in it readable, testable, or safe to change.

## Rules

Apply these inside every layer, same as in any codebase — no architecture-specific twist:

- **Meaningful names** — a name should say what something is or does, not force the reader to open the file to find out (`maxRetries`, not `x`). A good name removes the need for a comment explaining it.
- **DRY, without over-abstracting** — duplication is a signal to name a shared concept (`isAdult(user)` instead of the same `age >= 18` check copied twice), not a mandate to unify anything that merely looks similar. Two functions with the same three lines but different business meaning should stay separate.
- **KISS** — the minimum complexity the problem actually needs; `if (user.isActive) activateAccount(user)` doesn't need a strategy pattern.
- **Comments explain why, not what** — a comment restating the next line is noise the code should already carry in its name; a comment explaining a non-obvious decision (why 5 minutes, why this retry count) is worth keeping.
- **No magic numbers or strings** — a bare `3` or `'PEND'` forces every reader to already know what it means; a named constant or enum gives the compiler one place to catch a typo.
- **Boy Scout Rule** — leave the code a little cleaner than you found it; fix a bad name or oversized function in passing if a nearby change already surfaces it, no bigger a change than what you were already making.
- **SOLID, where it earns its keep** — reach for these when they remove a real pain, not preemptively: **O**pen/closed (extend, don't modify), **L**iskov substitution (an implementation must stand in for its abstraction), **I**nterface segregation (small specific interfaces over one that forces unrelated methods on every implementer). **S**ingle Responsibility and **D**ependency Inversion are covered below — this skill leans on both architecturally.

## Single Responsibility, Inside a Layer Too

Placing a file in `application/` doesn't satisfy SRP by itself. A use case that validates input, persists an entity, *and* sends a notification has the same smell as unlayered code — it just has better real estate. See `references/layers.md` for the layer-to-layer version; this is the same principle one level down, inside a single file.

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

The same constructor-injection shape is what makes this testable (fake `users`/`notifier` in a test, no concrete class built inline) and is the same DIP that `references/layers.md`'s "invert I/O through ports" already teaches, restated at function scope.

## Low Coupling, High Cohesion

`references/layers.md`'s dependency rule buys this between layers; the same idea applies between two files in the *same* layer — things that change together stay together. A `shared/utils.ts` mixing money formatting, a debounce helper, and an API call is low cohesion even though nothing there crosses a layer boundary.

## One Level of Abstraction Per Function

Don't mix a business rule, a hashing call, a SQL/HTTP call, and an outbound notification in one function body — the inside-the-function version of the layering rule this skill already enforces between files.

<!-- ❌ never — four different levels of abstraction in one function -->
```typescript
function registerUser(input: RegisterUserInput) {
  if (!input.email.includes('@')) throw new Error('bad email'); // business rule
  const hash = hashPassword(input.password, 10);                 // cryptography
  db.query('INSERT INTO users ...', [input.email, hash]);        // persistence
  sendEmail(input.email, 'Welcome!');                             // notification
}
```

Each line belongs to a different layer in `references/layers.md` — extract each concern behind its own named function or port, the same shape as the ✅ `RegisterUserUseCase` above.

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

## No Surprising Side Effects

A function's name is a promise to the caller. If `getUser()` also clears a cache and fires an analytics event, the name broke that promise:

<!-- ❌ never -->
```typescript
function getUser(): User {
  this.cache.clear();
  this.analytics.track('user_fetched');
  return this.user;
}
```

Split the read from the side effects, or rename the function to say what it actually does.
