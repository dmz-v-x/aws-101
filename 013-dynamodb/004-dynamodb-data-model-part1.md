## DynamoDB Data Model

### 1. DynamoDB Data Model: Starting From Absolute Zero

Before touching keys or partitions, we must understand **how DynamoDB actually stores data**.

DynamoDB does NOT think in terms of:
- Rows
- Columns
- Tables with fixed schemas

Instead, DynamoDB thinks in terms of:
- Tables
- Items
- Attributes

Everything else builds on this foundation.

---

### 2. What Is a Table in DynamoDB?

A **table** is the top-level container in DynamoDB.

You can think of a table as:
- A collection of related data
- Similar to a table in SQL, but without fixed columns

Important differences from SQL tables:
- No fixed schema
- No required columns except keys
- Items in the same table can look completely different

A table defines:
- Primary key structure (mandatory)
- Optional indexes
- Settings like capacity, backups, streams

---

### 3. What Is an Item? (Very Important)

An **item** is a single record stored inside a table.

Mental model:
- SQL row = DynamoDB item

Key properties of an item:
- Each item is uniquely identified by its **primary key**
- Items are independent of each other
- Each item can have different attributes

Example (conceptual):

    Item 1:
      userId: "user-1"
      name: "Alice"
      age: 30

    Item 2:
      userId: "user-2"
      email: "bob@example.com"
      isPremium: true

Both items can exist in the same table — no problem.

---

### 4. What Is an Attribute?

An **attribute** is a single piece of data inside an item.

Mental model:
- SQL column = DynamoDB attribute (but flexible)

Important facts:
- Attributes are name → value pairs
- Attributes do NOT need to be predefined
- Attributes can be added or removed anytime

Example:

    Attribute name: "age"
    Attribute value: 30

Each item can have:
- Different attribute names
- Different attribute counts
- Different attribute types

This flexibility is core to DynamoDB.

---

### 5. Attribute Value Types: Big Picture

DynamoDB supports **multiple attribute types** to store different kinds of data.

They fall into three major groups:
1. Scalar types (single values)
2. Document types (nested structures)
3. Set types (collections of unique values)

Let’s go through each one carefully.

---

### 6. Scalar Attribute Types

Scalar types store **one atomic value**.

Supported scalar types:
- String
- Number
- Binary
- Boolean
- Null

Examples:

    "name": "Alice"        (String)
    "age": 30              (Number)
    "isActive": true       (Boolean)
    "deletedAt": null      (Null)

Important notes:
- Numbers are stored as variable-precision numbers
- DynamoDB does not differentiate int vs float
- Strings are UTF-8 encoded

Scalars are the building blocks of all items.

---

### 7. Document Attribute Types (Maps and Lists)

Document types allow **nested, JSON-like structures**.

There are two document types:

#### Map (Object-like)

A **map** is a collection of key-value pairs.

Example:

    "address": {
      "city": "Mumbai",
      "country": "India",
      "zip": "400001"
    }

Maps can:
- Contain scalars
- Contain other maps
- Contain lists

Maps allow deep nesting.

---

#### List (Array-like)

A **list** is an ordered collection of values.

Example:

    "hobbies": [
      "reading",
      "gaming",
      "travel"
    ]

Lists:
- Preserve order
- Can contain mixed types
- Can contain maps and lists

Example (nested):

    "orders": [
      {
        "orderId": "o1",
        "amount": 500
      },
      {
        "orderId": "o2",
        "amount": 1200
      }
    ]

---

### 8. Set Attribute Types

Sets are **collections of unique values**.

Supported set types:
- String Set
- Number Set
- Binary Set

Example:

    "skills": ["js", "node", "aws"]

Rules for sets:
- All values must be of the same type
- No duplicate values
- Sets are unordered

Important limitation:
- Sets cannot contain maps or lists

Sets are useful when:
- You want uniqueness
- Order does not matter

---

### 9. Primary Keys: Why They Exist

Every DynamoDB table **must** have a primary key.

Why?
Because DynamoDB:
- Is a distributed system
- Needs a way to uniquely identify items
- Needs to decide where data lives physically

The primary key determines:
- Uniqueness
- Data distribution
- Query patterns
- Performance

There are two primary key types.

---

### 10. Primary Key Type 1: Partition Key Only

This is the **simplest** primary key.

Structure:
- One attribute
- Must be unique per item

Example:

    PartitionKey: "userId"

Items:

    userId = "user-1"
    userId = "user-2"

Rules:
- No two items can have the same partition key
- Each item is completely independent

Use this when:
- Each item represents a standalone entity
- You always access data by exact key

---

### 11. Primary Key Type 2: Composite Key (Partition + Sort)

This is the **most powerful and most common** design.

Structure:
- Partition key
- Sort key

Example:

    PartitionKey: "userId"
    SortKey: "createdAt"

Items:

    userId = "user-1", createdAt = "2024-01-01"
    userId = "user-1", createdAt = "2024-02-01"
    userId = "user-2", createdAt = "2024-01-10"

Rules:
- Partition key groups related items
- Sort key orders items within the same group
- Combination of both must be unique

This enables:
- One-to-many relationships
- Range queries
- Time-based data modeling

---

### 12. How DynamoDB Partitioning Works (High-Level)

DynamoDB is a **distributed database**.

It stores data across many physical machines called **partitions**.

Key idea:
- You do NOT choose partitions
- DynamoDB handles this automatically

What you control:
- The **partition key value**

---

### 13. Partition Key → Physical Partition Mapping

When you write an item:
1. DynamoDB takes the partition key value
2. Applies an internal hashing function
3. Maps it to a physical partition

Important consequences:
- Items with the same partition key go to the same logical partition
- Different partition key values spread across partitions
- Good key design = even distribution

Bad key design can cause:
- Hot partitions
- Throttling
- Performance issues

This is why partition key choice is critical.

---

### 14. What Is a Physical Partition?

A physical partition:
- Is a chunk of storage + throughput
- Has limits on size and request rate
- Is fully managed by AWS

You never:
- See partitions
- Configure partitions
- Manage partitions

But your key design determines:
- How evenly load is spread
- How scalable your table is

---

### 15. Data Types and Constraints (Schema-Less Reality)

DynamoDB does **not enforce a schema**.

That means:
- No fixed columns
- No required attributes (except keys)
- No migrations needed for new fields

Each item can:
- Have different attributes
- Skip attributes entirely
- Add attributes later

This leads to an important concept.

---

### 16. Sparse Attributes

A **sparse attribute** is an attribute that exists only on some items.

Example:

    Item A:
      userId: "user-1"
      email: "a@example.com"

    Item B:
      userId: "user-2"
      phone: "+91-9999999999"

This is completely valid.

Sparse attributes allow:
- Multiple entity types in one table
- Flexible data modeling
- Efficient indexing strategies later

Sparse design is a powerful DynamoDB pattern.

---

### 17. Constraints You Must Know

Even though DynamoDB is flexible, it still has rules:

- Item size limit: 400 KB
- Attribute names are case-sensitive
- Empty strings and empty sets have restrictions
- Primary key attributes must be scalar
- Sort keys must be scalar

These constraints influence real-world design decisions.

---

### 18. Advanced Mental Model Summary

At this point, your mental model should be:

- Table = container
- Item = record
- Attribute = field (flexible)
- Primary key = identity + distribution
- Partition key = scalability lever
- Sort key = grouping and ordering
- Documents = nested JSON-like data
- Schema-less = flexibility + responsibility

---

### 19. What You Fully Understand Now

After completing this module, you understand:
- How DynamoDB stores data internally (conceptually)
- Every attribute type and when to use it
- How primary keys work and why they matter
- How partitioning affects scalability
- Why schema-less design is powerful but dangerous if misused
