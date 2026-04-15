---
name: zod-to-valibot
description: Migrates TypeScript/JavaScript code from Zod to Valibot schema validation. Handles all common patterns: schemas, validators, transformations, error handling, type inference, and object utilities. Use when user mentions "migrate to valibot", "replace zod with valibot", "zod to valibot", or when files import from "zod" and the task involves switching to valibot.
---

# Zod to Valibot Migration

## Quick Start

Before migrating any file:
1. Install Valibot: `npm install valibot` (or pnpm/yarn/bun equivalent)
2. Remove Zod (after full migration): `npm uninstall zod`
3. Use `import * as v from 'valibot'` as the standard import

## Workflow

- [ ] Scan for all files importing from `"zod"` using grep: `grep -r "from 'zod'" --include="*.ts" --include="*.tsx" -l`
- [ ] Check if the official codemod can handle bulk conversion: `npx zod-to-valibot <file>`
- [ ] For files codemod doesn't fully handle, apply manual patterns from [REFERENCE.md](REFERENCE.md)
- [ ] Update all `z.infer<typeof Schema>` to `v.InferOutput<typeof Schema>`
- [ ] Run TypeScript compiler to catch remaining issues: `tsc --noEmit`
- [ ] Run tests to verify behavior is identical

## The One Rule That Changes Everything

Zod chains methods on schemas. Valibot uses `v.pipe()` for everything beyond the base type:

```ts
// Zod
z.string().email().min(5)

// Valibot
v.pipe(v.string(), v.email(), v.minLength(5))
```

Methods like `.parse()` and `.safeParse()` also move — they become standalone functions with schema as first argument:

```ts
// Zod
schema.parse(data)
schema.safeParse(data)

// Valibot
v.parse(schema, data)
v.safeParse(schema, data)
```

## Common Patterns at a Glance

| Zod | Valibot |
|-----|---------|
| `z.string()` | `v.string()` |
| `z.number()` | `v.number()` |
| `z.boolean()` | `v.boolean()` |
| `z.object({})` | `v.object({})` |
| `z.array(s)` | `v.array(s)` |
| `z.optional(s)` | `v.optional(s)` |
| `z.nullable(s)` | `v.nullable(s)` |
| `z.union([...])` | `v.union([...])` |
| `z.literal(x)` | `v.literal(x)` |
| `z.enum(['a','b'])` | `v.picklist(['a', 'b'])` |
| `z.nativeEnum(Enum)` | `v.enum(Enum)` |
| `z.infer<typeof S>` | `v.InferOutput<typeof S>` |

See [REFERENCE.md](REFERENCE.md) for complete patterns and [EXAMPLES.md](EXAMPLES.md) for real-world schema migrations.
