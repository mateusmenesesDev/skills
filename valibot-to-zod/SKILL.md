---
name: valibot-to-zod
description: Migrates TypeScript/JavaScript code from Valibot to Zod schema validation. Handles all common patterns: schemas, validators, transformations, error handling, type inference, and object utilities. Use when user mentions "migrate to zod", "replace valibot with zod", "valibot to zod", or when files import from "valibot" and the task involves switching to zod.
---

# Valibot to Zod Migration

## Quick Start

Before migrating any file:
1. Install Zod: `npm install zod` (or pnpm/yarn/bun equivalent)
2. Remove Valibot (after full migration): `npm uninstall valibot`
3. Use `import { z } from 'zod'` as the standard import

## Workflow

- [ ] Scan for all files importing from `"valibot"` using grep: `grep -r "from 'valibot'" --include="*.ts" --include="*.tsx" -l`
- [ ] Migrate each file using patterns from [REFERENCE.md](REFERENCE.md)
- [ ] Update all `v.InferOutput<typeof Schema>` to `z.infer<typeof Schema>`
- [ ] Run TypeScript compiler to catch remaining issues: `tsc --noEmit`
- [ ] Run tests to verify behavior is identical

## The One Rule That Changes Everything

Valibot uses `v.pipe()` for everything beyond the base type. Zod chains methods on schemas:

```ts
// Valibot
v.pipe(v.string(), v.email(), v.minLength(5))

// Zod
z.string().email().min(5)
```

Standalone functions become methods on the schema:

```ts
// Valibot
v.parse(schema, data)
v.safeParse(schema, data)

// Zod
schema.parse(data)
schema.safeParse(data)
```

## Common Patterns at a Glance

| Valibot | Zod |
|---------|-----|
| `v.string()` | `z.string()` |
| `v.number()` | `z.number()` |
| `v.boolean()` | `z.boolean()` |
| `v.object({})` | `z.object({})` |
| `v.array(s)` | `z.array(s)` |
| `v.optional(s)` | `z.optional(s)` or `s.optional()` |
| `v.nullable(s)` | `z.nullable(s)` or `s.nullable()` |
| `v.union([...])` | `z.union([...])` |
| `v.literal(x)` | `z.literal(x)` |
| `v.picklist(['a','b'])` | `z.enum(['a', 'b'])` |
| `v.enum(Enum)` | `z.nativeEnum(Enum)` |
| `v.InferOutput<typeof S>` | `z.infer<typeof S>` |

See [REFERENCE.md](REFERENCE.md) for complete patterns and [EXAMPLES.md](EXAMPLES.md) for real-world schema migrations.
