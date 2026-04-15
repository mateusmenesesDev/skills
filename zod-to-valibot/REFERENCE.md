# Zod to Valibot: Complete Migration Reference

## Import

```ts
// Remove
import { z } from 'zod';
import * as z from 'zod';

// Add
import * as v from 'valibot';
```

---

## Parsing & Validation

```ts
// Parse (throws on error)
z.string().parse(data)                    → v.parse(v.string(), data)

// Safe parse (returns result object)
z.string().safeParse(data)                → v.safeParse(v.string(), data)
result.success / result.data              → result.success / result.output
result.error.issues                       → result.issues

// Async
schema.parseAsync(data)                   → v.parseAsync(schema, data)
schema.safeParseAsync(data)               → v.safeParseAsync(schema, data)

// Error type
error instanceof ZodError                 → error instanceof v.ValiError
error.issues[0].message                   → error.issues[0].message  (same shape)
```

---

## Primitive Schemas

```ts
z.string()        → v.string()
z.number()        → v.number()
z.boolean()       → v.boolean()
z.bigint()        → v.bigint()
z.date()          → v.date()
z.symbol()        → v.symbol()
z.null()          → v.null()
z.undefined()     → v.undefined()
z.void()          → v.void()
z.any()           → v.any()
z.unknown()       → v.unknown()
z.never()         → v.never()
z.nan()           → v.nan()
```

---

## String Validations (pipe-based)

```ts
z.string().email()                    → v.pipe(v.string(), v.email())
z.string().url()                      → v.pipe(v.string(), v.url())
z.string().uuid()                     → v.pipe(v.string(), v.uuid())
z.string().cuid()                     → v.pipe(v.string(), v.cuid2())
z.string().regex(/pattern/)           → v.pipe(v.string(), v.regex(/pattern/))
z.string().min(n)                     → v.pipe(v.string(), v.minLength(n))
z.string().max(n)                     → v.pipe(v.string(), v.maxLength(n))
z.string().length(n)                  → v.pipe(v.string(), v.length(n))
z.string().startsWith('x')            → v.pipe(v.string(), v.startsWith('x'))
z.string().endsWith('x')              → v.pipe(v.string(), v.endsWith('x'))
z.string().includes('x')              → v.pipe(v.string(), v.includes('x'))
z.string().trim()                     → v.pipe(v.string(), v.trim())
z.string().toLowerCase()              → v.pipe(v.string(), v.toLowerCase())
z.string().toUpperCase()              → v.pipe(v.string(), v.toUpperCase())
z.string().ip()                       → v.pipe(v.string(), v.ip())
z.string().datetime()                 → v.pipe(v.string(), v.isoDateTime())
z.string().date()                     → v.pipe(v.string(), v.isoDate())
z.string().time()                     → v.pipe(v.string(), v.isoTime())
```

---

## Number Validations

```ts
z.number().min(n)           → v.pipe(v.number(), v.minValue(n))
z.number().max(n)           → v.pipe(v.number(), v.maxValue(n))
z.number().int()            → v.pipe(v.number(), v.integer())
z.number().positive()       → v.pipe(v.number(), v.minValue(0 + epsilon))
z.number().nonnegative()    → v.pipe(v.number(), v.minValue(0))
z.number().negative()       → v.pipe(v.number(), v.maxValue(0 - epsilon))
z.number().nonpositive()    → v.pipe(v.number(), v.maxValue(0))
z.number().multipleOf(n)    → v.pipe(v.number(), v.multipleOf(n))
z.number().finite()         → v.pipe(v.number(), v.finite())
z.number().safe()           → v.pipe(v.number(), v.safeInteger())
```

---

## Objects

```ts
// Basic object
z.object({ name: z.string() })
→ v.object({ name: v.string() })

// Strict (reject unknown keys)
z.object({}).strict()
→ v.strictObject({})

// Strip unknown keys (Zod default behavior)
z.object({})
→ v.object({})  // Valibot strips by default too

// Passthrough unknown keys
z.object({}).passthrough()
→ v.looseObject({})

// Pick
z.object({a: z.string(), b: z.number()}).pick({a: true})
→ v.pick(schema, ['a'])

// Omit
z.object({a: z.string(), b: z.number()}).omit({b: true})
→ v.omit(schema, ['b'])

// Partial
z.object({}).partial()
→ v.partial(schema)

// Required
z.object({}).required()
→ v.required(schema)

// Extend / merge
z.object({a: z.string()}).extend({b: z.number()})
→ v.object({ ...baseSchema.entries, b: v.number() })

// Keyof
z.object({}).keyof()
→ v.keyof(schema)
```

---

## Arrays & Tuples

```ts
z.array(z.string())                   → v.array(v.string())
z.array(z.string()).min(n)            → v.pipe(v.array(v.string()), v.minLength(n))
z.array(z.string()).max(n)            → v.pipe(v.array(v.string()), v.maxLength(n))
z.array(z.string()).length(n)         → v.pipe(v.array(v.string()), v.length(n))
z.array(z.string()).nonempty()        → v.pipe(v.array(v.string()), v.nonEmpty())

z.tuple([z.string(), z.number()])
→ v.tuple([v.string(), v.number()])

z.tuple([z.string()]).rest(z.number())
→ v.tupleWithRest([v.string()], v.number())
```

---

## Optional, Nullable, Nullish

```ts
z.optional(schema)     → v.optional(schema)     // string | undefined
z.nullable(schema)     → v.nullable(schema)      // string | null
z.nullish(schema)      → v.nullish(schema)       // string | null | undefined

// On object keys, Valibot optional works the same way
{ key: z.string().optional() }  →  { key: v.optional(v.string()) }
```

---

## Union & Intersection & Discriminated Union

```ts
// Union
z.union([z.string(), z.number()])
→ v.union([v.string(), v.number()])

// Intersection
z.intersection(SchemaA, SchemaB)
→ v.intersect([SchemaA, SchemaB])

// Discriminated union (IMPORTANT: called "variant" in Valibot)
z.discriminatedUnion('type', [
  z.object({ type: z.literal('a'), foo: z.string() }),
  z.object({ type: z.literal('b'), bar: z.number() }),
])
→ v.variant('type', [
  v.object({ type: v.literal('a'), foo: v.string() }),
  v.object({ type: v.literal('b'), bar: v.number() }),
])
```

---

## Enums & Literals

```ts
// String literal union
z.enum(['a', 'b', 'c'])
→ v.picklist(['a', 'b', 'c'])

// TypeScript native enum
enum Direction { Up, Down }
z.nativeEnum(Direction)
→ v.enum(Direction)

// Literal
z.literal('foo')    → v.literal('foo')
z.literal(42)       → v.literal(42)
z.literal(true)     → v.literal(true)
```

---

## Records & Maps & Sets

```ts
z.record(z.string(), z.number())
→ v.record(v.string(), v.number())

z.map(z.string(), z.number())
→ v.map(v.string(), v.number())

z.set(z.string())
→ v.set(v.string())
```

---

## Transform & Coerce

```ts
// Simple transform
z.string().transform(s => s.length)
→ v.pipe(v.string(), v.transform(s => s.length))

// Coerce (Zod shorthand)
z.coerce.string()
→ v.pipe(v.unknown(), v.transform(String))

z.coerce.number()
→ v.pipe(v.unknown(), v.transform(Number))
// OR (safer - validates format first)
→ v.pipe(v.string(), v.decimal(), v.transform(Number))

z.coerce.boolean()
→ v.pipe(v.unknown(), v.transform(Boolean))

z.coerce.date()
→ v.pipe(v.unknown(), v.transform(v => new Date(v as string)))
```

---

## Refinements (Custom Validation)

```ts
// refine
z.string().refine(s => s.length > 5, 'Too short')
→ v.pipe(v.string(), v.check(s => s.length > 5, 'Too short'))

// superRefine
z.string().superRefine((val, ctx) => {
  if (!val.includes('@')) ctx.addIssue({ code: 'custom', message: 'bad' })
})
→ v.pipe(v.string(), v.rawCheck(({ dataset, addIssue }) => {
  if (!dataset.value.includes('@')) addIssue({ message: 'bad' })
}))
```

---

## Default & Fallback

```ts
// Default value
z.string().default('hello')
→ v.optional(v.string(), 'hello')
// OR
→ v.pipe(v.optional(v.string()), v.transform(v => v ?? 'hello'))

// Catch (fallback on parse error)
z.string().catch('fallback')
→ v.fallback(v.string(), 'fallback')
```

---

## Branding

```ts
z.string().brand<'UserId'>()
→ v.pipe(v.string(), v.brand('UserId'))

// Type
type UserId = z.infer<typeof UserIdSchema>  →  type UserId = v.InferOutput<typeof UserIdSchema>
```

---

## Lazy / Recursive Schemas

```ts
// Zod
const Schema: z.ZodTypeAny = z.lazy(() => z.object({ children: z.array(Schema) }))

// Valibot (requires explicit type annotation)
type Node = { children: Node[] }
const Schema: v.GenericSchema<Node> = v.lazy(() =>
  v.object({ children: v.array(Schema) })
)
```

---

## Type Inference

```ts
type MyType = z.infer<typeof MySchema>
→
type MyType = v.InferOutput<typeof MySchema>

// Input type (before transforms)
type MyInput = z.input<typeof MySchema>
→
type MyInput = v.InferInput<typeof MySchema>
```

---

## Custom Error Messages

Valibot passes error messages as the **last argument** to schema/action functions:

```ts
z.string({ invalid_type_error: 'Must be text' })
→ v.string('Must be text')

z.string().min(5, { message: 'Too short' })
→ v.pipe(v.string(), v.minLength(5, 'Too short'))

z.object({}, { invalid_type_error: 'Not an object' })
→ v.object({}, 'Not an object')
```

---

## instanceof & Custom Schemas

```ts
z.instanceof(Date)
→ v.instance(Date)

z.custom<MyType>(val => isMyType(val))
→ v.custom<MyType>(val => isMyType(val))
```

---

## Codemod Tool

Valibot ships an official codemod for bulk conversion:

```bash
npx zod-to-valibot path/to/file.ts
# or for a whole directory
npx zod-to-valibot src/
```

The codemod handles primitives, objects, arrays, unions, optionals, and basic validations. Manual migration is needed for: discriminated unions, recursive schemas, superRefine, coerce, and complex transforms.
