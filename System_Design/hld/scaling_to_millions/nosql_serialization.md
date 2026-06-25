# Why NoSQL is Preferred for Serialize/Deserialize

NoSQL (document stores like MongoDB) stores data in JSON/BSON — which maps directly to how code represents objects.

## The Core Reason

With SQL, data is **transformed twice**:
- **Write:** Object → break into rows/columns across tables → store
- **Read:** fetch rows → JOIN tables → reassemble into object

With NoSQL:
- **Write:** Object → serialize to JSON → store as-is
- **Read:** fetch document → deserialize to object → done

No transformation, no JOIN overhead.

## Example

User object in code:
```json
{
  "id": 1,
  "name": "Alice",
  "address": { "city": "Delhi", "zip": "110001" },
  "hobbies": ["coding", "chess"]
}
```

**SQL** needs 3 tables: `users`, `addresses`, `hobbies` + JOINs to rebuild it.  
**MongoDB** stores it exactly as shown — one read, one write.

## Why This Matters at Scale

- Fewer DB calls (no JOINs)
- Horizontal scaling is easier (documents are self-contained, easy to shard)
- Schema flexibility — no ALTER TABLE when object shape changes

## When SQL is Still Better

If data has **complex relationships** (orders → products → users → payments), SQL's relational model + JOINs are more powerful and consistent.

**TL;DR:** NoSQL wins when data is already object-shaped and you just need to store/retrieve it — no relational logic needed.
