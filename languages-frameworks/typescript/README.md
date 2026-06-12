# TypeScript — Deep Dive

> Tags: #dx #scalability

Production-focused notes on TypeScript: type system mastery, patterns, performance, and Node.js/backend-specific topics.

---

## Contents

- [Type System Essentials](#type-system-essentials)
- [Advanced Types](#advanced-types)
- [Patterns & Idioms](#patterns--idioms)
- [Runtime Safety](#runtime-safety)
- [Performance Tips](#performance-tips)
- [Node.js Backend Patterns](#nodejs-backend-patterns)
- [Common Pitfalls](#common-pitfalls)

---

## Type System Essentials

### Structural Typing

TypeScript uses structural (duck) typing — types are compatible if their shapes match:

```ts
interface Point { x: number; y: number }
interface Named { name: string; x: number; y: number }

function printPoint(p: Point) { /* ... */ }

const named: Named = { name: "origin", x: 0, y: 0 };
printPoint(named); // ✅ — Named has all fields of Point
```

### `unknown` vs `any`

```ts
// any — disables type checking (avoid)
const val: any = JSON.parse(input);
val.foo.bar; // no error, but crashes at runtime

// unknown — forces you to narrow before use (prefer)
const val: unknown = JSON.parse(input);
if (typeof val === "object" && val !== null && "foo" in val) {
    // now val is narrowed
}
```

### Type Narrowing

```ts
function handle(value: string | number) {
    if (typeof value === "string") {
        return value.toUpperCase(); // string
    }
    return value.toFixed(2); // number
}

// Discriminated Union — best pattern for union narrowing
type Result<T> =
    | { success: true; data: T }
    | { success: false; error: string };

function handleResult(result: Result<User>) {
    if (result.success) {
        console.log(result.data.name); // data available here
    } else {
        console.error(result.error); // error available here
    }
}
```

---

## Advanced Types

### Mapped Types

```ts
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Partial<T>  = { [K in keyof T]?: T[K] };
type Required<T> = { [K in keyof T]-?: T[K] };

// Practical example — API update payload
type UpdatePayload<T> = { [K in keyof T]?: T[K] };
```

### Conditional Types

```ts
type IsArray<T> = T extends any[] ? true : false;
type ElementType<T> = T extends (infer U)[] ? U : never;

type A = ElementType<string[]>; // string
type B = ElementType<number>;   // never
```

### Template Literal Types

```ts
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`; // "onClick" | "onFocus" | "onBlur"

type Getter<T extends string> = `get${Capitalize<T>}`;
type Setter<T extends string> = `set${Capitalize<T>}`;
```

### Utility Types Cheat Sheet

| Utility | Description |
|---------|-------------|
| `Partial<T>` | All properties optional |
| `Required<T>` | All properties required |
| `Readonly<T>` | All properties readonly |
| `Pick<T, K>` | Keep only keys K |
| `Omit<T, K>` | Remove keys K |
| `Record<K, V>` | Object with keys K and values V |
| `Exclude<T, U>` | Remove U from union T |
| `Extract<T, U>` | Keep only U from union T |
| `NonNullable<T>` | Remove null and undefined |
| `ReturnType<F>` | Return type of function F |
| `Parameters<F>` | Parameter tuple of function F |
| `Awaited<T>` | Unwrap Promise type |

---

## Patterns & Idioms

### Builder Pattern with Method Chaining

```ts
class QueryBuilder {
    private filters: string[] = [];
    private _limit = 100;

    where(condition: string): this {
        this.filters.push(condition);
        return this;
    }

    limit(n: number): this {
        this._limit = n;
        return this;
    }

    build(): string {
        const where = this.filters.length ? `WHERE ${this.filters.join(" AND ")}` : "";
        return `SELECT * FROM table ${where} LIMIT ${this._limit}`;
    }
}

const query = new QueryBuilder().where("age > 18").where("active = true").limit(50).build();
```

### Option / Result Pattern (without libraries)

```ts
type Option<T> = T | null;
type Result<T, E = Error> = { ok: true; value: T } | { ok: false; error: E };

function divide(a: number, b: number): Result<number, string> {
    if (b === 0) return { ok: false, error: "Division by zero" };
    return { ok: true, value: a / b };
}
```

### Branded Types (Nominal Types)

Prevent mixing semantically different values of the same primitive type:

```ts
type UserId   = string & { readonly _brand: "UserId" };
type OrderId  = string & { readonly _brand: "OrderId" };

function createUserId(id: string): UserId   { return id as UserId; }
function createOrderId(id: string): OrderId { return id as OrderId; }

function getUser(id: UserId) { /* ... */ }

const userId  = createUserId("u-123");
const orderId = createOrderId("o-456");

getUser(userId);  // ✅
getUser(orderId); // ❌ TypeScript error
```

---

## Runtime Safety

### Zod — Runtime Schema Validation

```ts
import { z } from "zod";

const UserSchema = z.object({
    id:    z.string().uuid(),
    name:  z.string().min(1),
    email: z.string().email(),
    age:   z.number().int().positive().optional(),
});

type User = z.infer<typeof UserSchema>; // type is derived from schema

// Validate at runtime (e.g., API request body)
const result = UserSchema.safeParse(req.body);
if (!result.success) {
    return res.status(400).json({ errors: result.error.flatten() });
}
const user = result.data; // fully typed
```

### Type Guards

```ts
function isUser(value: unknown): value is User {
    return (
        typeof value === "object" &&
        value !== null &&
        "id" in value &&
        typeof (value as any).id === "string"
    );
}
```

---

## Performance Tips

- **`const` assertions** — `as const` freezes literal types, avoids widening
- **Lazy imports** — `import()` for code splitting in bundlers
- **Avoid deep nesting** — deeply nested generics slow down the compiler
- **Use `interface` over `type` for objects** — interfaces are slightly faster to check
- **`skipLibCheck: true`** in `tsconfig.json` — skip type-checking `.d.ts` files for faster builds
- **`isolatedModules: true`** — enables `tsc --transpileOnly` and compatible with esbuild/SWC

### tsconfig.json Best Practices

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "outDir": "dist",
    "rootDir": "src"
  }
}
```

---

## Node.js Backend Patterns

### Async Error Handling in Express

```ts
// Wrap async route handlers to catch errors
const asyncHandler = (fn: RequestHandler): RequestHandler =>
    (req, res, next) => Promise.resolve(fn(req, res, next)).catch(next);

app.get("/users/:id", asyncHandler(async (req, res) => {
    const user = await userService.findById(req.params.id);
    res.json(user);
}));

// Global error handler
app.use((err: Error, req: Request, res: Response, _next: NextFunction) => {
    res.status(500).json({ error: err.message });
});
```

### Environment Variable Validation

```ts
import { z } from "zod";

const EnvSchema = z.object({
    DATABASE_URL: z.string().url(),
    PORT:         z.coerce.number().default(3000),
    NODE_ENV:     z.enum(["development", "test", "production"]),
    JWT_SECRET:   z.string().min(32),
});

export const env = EnvSchema.parse(process.env);
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| `as` overuse | Bypasses type safety | Use type guards and narrowing |
| `any` leaking from JSON.parse | Loses type info | Use `unknown` + Zod/validation |
| Mutation of function params | Side effects, hard to trace | Use `Readonly<T>` params |
| Missing `strictNullChecks` | `null`/`undefined` go undetected | Enable `strict: true` |
| Enum vs `as const` | Enums have runtime overhead | Prefer `as const` + union type |
| Forgetting `await` on async | Returns `Promise` silently | Enable `no-floating-promises` ESLint rule |

---

## References

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- [TypeScript Deep Dive (free book)](https://basarat.gitbook.io/typescript/)
- [Matt Pocock's Total TypeScript](https://www.totaltypescript.com/)
- [Zod Documentation](https://zod.dev/)
- Book: *Programming TypeScript* — Boris Cherny
