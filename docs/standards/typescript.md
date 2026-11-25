# TypeScript

## Types of values

- Give a value the specific type that you know it has. Take a value from outside whose type you don't know, such as the result of `JSON.parse` or the answer of a service, as `unknown`, and narrow it with a check or a zod schema before you use it.

## Object types

- Add fields to an existing object type, or narrow it, with `interface X extends Base`, rather than with an intersection `Base & { ... }`, also when `Base` is a `Pick` or an `Omit`. When the two declare a key with types that don't fit together, such as `number` and `string`, the interface fails where it is declared, while the intersection makes the key `never` and fails only where a value is built, if anywhere.

## Closed sets of values

- Declare a closed set of values, such as states, roles or kinds, as an `as const` array, its type as `(typeof X)[number]`, and its zod schema as `z.enum(X)`, rather than as a union of literal types or as values listed in a schema.
- Declare each closed set once, in the module of the domain that owns it, and import it wherever else it is needed.

## Deriving types

- Give the fields that mean the same and live as long within one bounded context one owner type. Derive the type that another layer or feature needs from the owner, rather than declaring its fields again. When a set of values or a field has no owner yet, decide its owner before you copy it into the type that uses it.
- Derive a type from its owner as below.

  | To | Write |
  | --- | --- |
  | Keep some fields | `Pick<Owner, K>` |
  | Leave some fields out | `Omit<Owner, K>` |
  | Make every field optional, required or read-only | `Partial<Owner>`, `Required<Owner>`, `Readonly<Owner>` |
  | Map keys to values | `Record<K, T>` |
  | Add a field | `interface Derived extends Owner { newField: T }` |
  | Express a narrower state or an invariant | An interface that extends the owner, with the invariant in its comment |
  | Replace a field on purpose | `interface Derived extends Omit<Owner, 'field'> { field: New }`, with a comment that says why and whether the owner changes |

- Derive only a type that is a part of the owner's shape, made of fields that the owner holds, such as `Pick<User, 'email'>`. A type that holds one ID to name its target, such as the input of a lookup, isn't a part of the target's shape: give that ID the type that the owner has for its ID, when the owner has one.
- Keep the type of the owner's field, rather than declaring the field again with a wider type such as `string` or `boolean`.

## Type assertions

- Put an assertion that can't be avoided in a type helper whose only job is that conversion, and give the reason after `--` on the `eslint-disable-next-line` comment above it.
