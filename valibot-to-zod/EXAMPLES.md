# Valibot to Zod: Real-World Examples

## Example 1: User Schema

```ts
// BEFORE (Valibot)
import * as v from 'valibot';

const AddressSchema = v.object({
  street: v.pipe(v.string(), v.minLength(1)),
  city: v.pipe(v.string(), v.minLength(1)),
  zip: v.pipe(v.string(), v.regex(/^\d{5}$/)),
});

const UserSchema = v.object({
  id: v.pipe(v.string(), v.uuid()),
  name: v.pipe(v.string(), v.minLength(2), v.maxLength(100)),
  email: v.pipe(v.string(), v.email()),
  age: v.optional(v.pipe(v.number(), v.integer(), v.minValue(0), v.maxValue(150))),
  role: v.picklist(['admin', 'user', 'guest']),
  address: v.optional(AddressSchema),
  tags: v.optional(v.array(v.string()), []),
});

type User = v.InferOutput<typeof UserSchema>;

const result = v.safeParse(UserSchema, input);
if (result.success) {
  console.log(result.output);
}
```

```ts
// AFTER (Zod)
import { z } from 'zod';

const AddressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  zip: z.string().regex(/^\d{5}$/),
});

const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(['admin', 'user', 'guest']),
  address: AddressSchema.optional(),
  tags: z.array(z.string()).default([]),
});

type User = z.infer<typeof UserSchema>;

const result = UserSchema.safeParse(input);
if (result.success) {
  console.log(result.data);  // note: .data not .output
}
```

---

## Example 2: API Response with Discriminated Union

```ts
// BEFORE (Valibot)
const SuccessSchema = v.object({
  status: v.literal('ok'),
  data: v.object({ userId: v.string() }),
});

const ErrorSchema = v.object({
  status: v.literal('error'),
  code: v.number(),
  message: v.string(),
});

// variant → discriminatedUnion
const ResponseSchema = v.variant('status', [SuccessSchema, ErrorSchema]);
```

```ts
// AFTER (Zod)
const SuccessSchema = z.object({
  status: z.literal('ok'),
  data: z.object({ userId: z.string() }),
});

const ErrorSchema = z.object({
  status: z.literal('error'),
  code: z.number(),
  message: z.string(),
});

const ResponseSchema = z.discriminatedUnion('status', [SuccessSchema, ErrorSchema]);
```

---

## Example 3: Form Validation with Transform

```ts
// BEFORE (Valibot)
const FormSchema = v.pipe(
  v.object({
    username: v.pipe(v.string(), v.minLength(3), v.transform(s => s.toLowerCase())),
    password: v.pipe(v.string(), v.minLength(8)),
    confirmPassword: v.string(),
    birthYear: v.pipe(v.unknown(), v.transform(Number), v.integer(), v.minValue(1900), v.maxValue(2024)),
  }),
  v.check(
    data => data.password === data.confirmPassword,
    'Passwords must match'
  )
);
```

```ts
// AFTER (Zod)
const FormSchema = z.object({
  username: z.string().min(3).transform(s => s.toLowerCase()),
  password: z.string().min(8),
  confirmPassword: z.string(),
  birthYear: z.coerce.number().int().min(1900).max(2024),
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords must match',
  path: ['confirmPassword'],
});
```

---

## Example 4: Recursive Schema

```ts
// BEFORE (Valibot)
interface Category {
  name: string;
  subcategories: Category[];
}

const CategorySchema: v.GenericSchema<Category> = v.lazy(() =>
  v.object({
    name: v.string(),
    subcategories: v.array(CategorySchema),
  })
);
```

```ts
// AFTER (Zod)
interface Category {
  name: string;
  subcategories: Category[];
}

const CategorySchema: z.ZodType<Category> = z.lazy(() =>
  z.object({
    name: z.string(),
    subcategories: z.array(CategorySchema),
  })
);
```

---

## Example 5: Error Handling

```ts
// BEFORE (Valibot)
try {
  const data = v.parse(schema, input);
} catch (err) {
  if (err instanceof v.ValiError) {
    err.issues.forEach(issue => {
      console.log(issue.path?.map(p => p.key).join('.'), issue.message);
    });
  }
}
```

```ts
// AFTER (Zod)
import { ZodError } from 'zod';

try {
  const data = schema.parse(input);
} catch (err) {
  if (err instanceof ZodError) {
    err.issues.forEach(issue => {
      console.log(issue.path.join('.'), issue.message);
    });
  }
}
```

---

## Example 6: Branded Types

```ts
// BEFORE (Valibot)
const UserIdSchema = v.pipe(v.string(), v.uuid(), v.brand('UserId'));
type UserId = v.InferOutput<typeof UserIdSchema>;
```

```ts
// AFTER (Zod)
const UserIdSchema = z.string().uuid().brand<'UserId'>();
type UserId = z.infer<typeof UserIdSchema>;
```

---

## Gotchas & Differences to Watch For

1. **`.safeParse` result shape**: Valibot uses `.output`, Zod uses `.data`
2. **Variant → Discriminated union**: `v.variant()` → `z.discriminatedUnion()`
3. **Enums**: `v.picklist(['a','b'])` → `z.enum(['a','b'])`; `v.enum(Enum)` → `z.nativeEnum(Enum)`
4. **Object extension**: Valibot spreads `.entries`; Zod has `.extend()` method
5. **Async refinements**: `v.checkAsync(async fn)` → `z.string().refine(async fn)`
6. **Union shorthand**: no `.or()` / `.and()` equivalents in Valibot, so just swap to `z.union()` / `z.intersection()`
7. **Pipe → chaining**: All `v.pipe(v.type(), v.action1(), v.action2())` become `z.type().action1().action2()`
8. **Error path**: Valibot path items have `.key` property; Zod path is a flat array of strings/numbers
