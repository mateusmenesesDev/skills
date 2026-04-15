# Valibot to Zod: Complete Migration Reference

## Import

```ts
// Remove
import * as v from 'valibot';

// Add
import { z } from 'zod';
```

---

## Parsing & Validation

```ts
// Parse (throws on error)
v.parse(v.string(), data)                → z.string().parse(data)

// Safe parse (returns result object)
v.safeParse(v.string(), data)            → z.string().safeParse(data)
result.success / result.output           → result.success / result.data
result.issues                            → result.error.issues

// Async
v.parseAsync(schema, data)               → schema.parseAsync(data)
v.safeParseAsync(schema, data)           → schema.safeParseAsync(data)

// Error type
error instanceof v.ValiError             → error instanceof ZodError
error.issues[0].message                  → error.issues[0].message  (same shape)
```

---

## Primitive Schemas

```ts
v.string()        → z.string()
v.number()        → z.number()
v.boolean()       → z.boolean()
v.bigint()        → z.bigint()
v.date()          → z.date()
v.symbol()        → z.symbol()
v.null()          → z.null()
v.undefined()     → z.undefined()
v.void()          → z.void()
v.any()           → z.any()
v.unknown()       → z.unknown()
v.never()         → z.never()
v.nan()           → z.nan()
```

---

## String Validations (pipe → chaining)

```ts
v.pipe(v.string(), v.email())                   → z.string().email()
v.pipe(v.string(), v.url())                     → z.string().url()
v.pipe(v.string(), v.uuid())                    → z.string().uuid()
v.pipe(v.string(), v.cuid2())                   → z.string().cuid()
v.pipe(v.string(), v.regex(/pattern/))          → z.string().regex(/pattern/)
v.pipe(v.string(), v.minLength(n))              → z.string().min(n)
v.pipe(v.string(), v.maxLength(n))              → z.string().max(n)
v.pipe(v.string(), v.length(n))                 → z.string().length(n)
v.pipe(v.string(), v.startsWith('x'))           → z.string().startsWith('x')
v.pipe(v.string(), v.endsWith('x'))             → z.string().endsWith('x')
v.pipe(v.string(), v.includes('x'))             → z.string().includes('x')
v.pipe(v.string(), v.trim())                    → z.string().trim()
v.pipe(v.string(), v.toLowerCase())             → z.string().toLowerCase()
v.pipe(v.string(), v.toUpperCase())             → z.string().toUpperCase()
v.pipe(v.string(), v.ip())                      → z.string().ip()
v.pipe(v.string(), v.isoDateTime())             → z.string().datetime()
v.pipe(v.string(), v.isoDate())                 → z.string().date()
v.pipe(v.string(), v.isoTime())                 → z.string().time()
```

Multiple validations in one pipe become chained methods:

```ts
v.pipe(v.string(), v.email(), v.minLength(5), v.maxLength(100))
→ z.string().email().min(5).max(100)
```

---

## Number Validations

```ts
v.pipe(v.number(), v.minValue(n))          → z.number().min(n)
v.pipe(v.number(), v.maxValue(n))          → z.number().max(n)
v.pipe(v.number(), v.integer())            → z.number().int()
v.pipe(v.number(), v.minValue(1))          → z.number().positive()  // if semantically "positive"
v.pipe(v.number(), v.minValue(0))          → z.number().nonnegative()
v.pipe(v.number(), v.maxValue(-1))         → z.number().negative()  // if semantically "negative"
v.pipe(v.number(), v.maxValue(0))          → z.number().nonpositive()
v.pipe(v.number(), v.multipleOf(n))        → z.number().multipleOf(n)
v.pipe(v.number(), v.finite())             → z.number().finite()
v.pipe(v.number(), v.safeInteger())        → z.number().safe()
```

---

## Objects

```ts
// Basic object
v.object({ name: v.string() })
→ z.object({ name: z.string() })

// Strict (reject unknown keys)
v.strictObject({})
→ z.object({}).strict()

// Passthrough unknown keys
v.looseObject({})
→ z.object({}).passthrough()

// Pick
v.pick(schema, ['a'])
→ schema.pick({ a: true })

// Omit
v.omit(schema, ['b'])
→ schema.omit({ b: true })

// Partial
v.partial(schema)
→ schema.partial()

// Required
v.required(schema)
→ schema.required()

// Extend (Valibot spread pattern)
v.object({ ...baseSchema.entries, b: v.number() })
→ baseSchema.extend({ b: z.number() })

// Keyof
v.keyof(schema)
→ schema.keyof()
```

---

## Arrays & Tuples

```ts
v.array(v.string())                                  → z.array(z.string())
v.pipe(v.array(v.string()), v.minLength(n))          → z.array(z.string()).min(n)
v.pipe(v.array(v.string()), v.maxLength(n))          → z.array(z.string()).max(n)
v.pipe(v.array(v.string()), v.length(n))             → z.array(z.string()).length(n)
v.pipe(v.array(v.string()), v.nonEmpty())            → z.array(z.string()).nonempty()

v.tuple([v.string(), v.number()])
→ z.tuple([z.string(), z.number()])

v.tupleWithRest([v.string()], v.number())
→ z.tuple([z.string()]).rest(z.number())
```

---

## Optional, Nullable, Nullish

```ts
v.optional(schema)     → schema.optional()       // or z.optional(schema)
v.nullable(schema)     → schema.nullable()        // or z.nullable(schema)
v.nullish(schema)      → schema.nullish()         // or z.nullish(schema)

// On object keys
{ key: v.optional(v.string()) }  →  { key: z.string().optional() }
```

---

## Union & Intersection & Discriminated Union

```ts
// Union
v.union([v.string(), v.number()])
→ z.union([z.string(), z.number()])

// Intersection
v.intersect([SchemaA, SchemaB])
→ z.intersection(SchemaA, SchemaB)

// Variant (IMPORTANT: called "discriminatedUnion" in Zod)
v.variant('type', [
  v.object({ type: v.literal('a'), foo: v.string() }),
  v.object({ type: v.literal('b'), bar: v.number() }),
])
→ z.discriminatedUnion('type', [
  z.object({ type: z.literal('a'), foo: z.string() }),
  z.object({ type: z.literal('b'), bar: z.number() }),
])
```

---

## Enums & Literals

```ts
// Picklist (string literal union)
v.picklist(['a', 'b', 'c'])
→ z.enum(['a', 'b', 'c'])

// TypeScript native enum
enum Direction { Up, Down }
v.enum(Direction)
→ z.nativeEnum(Direction)

// Literal
v.literal('foo')    → z.literal('foo')
v.literal(42)       → z.literal(42)
v.literal(true)     → z.literal(true)
```

---

## Records & Maps & Sets

```ts
v.record(v.string(), v.number())
→ z.record(z.string(), z.number())

v.map(v.string(), v.number())
→ z.map(z.string(), z.number())

v.set(v.string())
→ z.set(z.string())
```

---

## Transform & Coerce

```ts
// Simple transform (inside pipe)
v.pipe(v.string(), v.transform(s => s.length))
→ z.string().transform(s => s.length)

// Coerce patterns
v.pipe(v.unknown(), v.transform(String))
→ z.coerce.string()

v.pipe(v.unknown(), v.transform(Number))
→ z.coerce.number()

v.pipe(v.string(), v.decimal(), v.transform(Number))
→ z.coerce.number()

v.pipe(v.unknown(), v.transform(Boolean))
→ z.coerce.boolean()

v.pipe(v.unknown(), v.transform(v => new Date(v as string)))
→ z.coerce.date()
```

---

## Refinements (Custom Validation)

```ts
// check → refine
v.pipe(v.string(), v.check(s => s.length > 5, 'Too short'))
→ z.string().refine(s => s.length > 5, 'Too short')

// rawCheck → superRefine
v.pipe(v.string(), v.rawCheck(({ dataset, addIssue }) => {
  if (!dataset.value.includes('@')) addIssue({ message: 'bad' })
}))
→ z.string().superRefine((val, ctx) => {
  if (!val.includes('@')) ctx.addIssue({ code: 'custom', message: 'bad' })
})

// checkAsync → async refine
v.pipe(v.string(), v.checkAsync(async fn))
→ z.string().refine(async fn)
```

---

## Default & Fallback

```ts
// Default value (Valibot uses optional with default)
v.optional(v.string(), 'hello')
→ z.string().default('hello')

// Fallback on parse error
v.fallback(v.string(), 'fallback')
→ z.string().catch('fallback')
```

---

## Branding

```ts
v.pipe(v.string(), v.brand('UserId'))
→ z.string().brand<'UserId'>()

// Type
type UserId = v.InferOutput<typeof UserIdSchema>  →  type UserId = z.infer<typeof UserIdSchema>
```

---

## Lazy / Recursive Schemas

```ts
// Valibot (explicit type annotation)
type Node = { children: Node[] }
const Schema: v.GenericSchema<Node> = v.lazy(() =>
  v.object({ children: v.array(Schema) })
)

// Zod
const Schema: z.ZodType<Node> = z.lazy(() =>
  z.object({ children: z.array(Schema) })
)
```

---

## Type Inference

```ts
type MyType = v.InferOutput<typeof MySchema>
→
type MyType = z.infer<typeof MySchema>

// Input type (before transforms)
type MyInput = v.InferInput<typeof MySchema>
→
type MyInput = z.input<typeof MySchema>
```

---

## Custom Error Messages

Valibot passes error messages as the last argument; Zod uses options objects:

```ts
v.string('Must be text')
→ z.string({ invalid_type_error: 'Must be text' })

v.pipe(v.string(), v.minLength(5, 'Too short'))
→ z.string().min(5, { message: 'Too short' })
// or shorthand:
→ z.string().min(5, 'Too short')

v.object({}, 'Not an object')
→ z.object({}, { invalid_type_error: 'Not an object' })
```

---

## instanceof & Custom Schemas

```ts
v.instance(Date)
→ z.instanceof(Date)

v.custom<MyType>(val => isMyType(val))
→ z.custom<MyType>(val => isMyType(val))
```
