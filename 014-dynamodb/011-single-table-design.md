## Single Table Design

### 1. What Is Single-Table Design?

Single-table design means:

> Storing multiple entity types in **one DynamoDB table**,  
> using carefully designed keys to model relationships.

This feels wrong if you come from SQL.

In SQL:
- One table per entity
- Relationships via joins

In DynamoDB:
- One table for a whole application (often)
- Relationships encoded in keys
- No joins, ever

Single-table design is a **modeling strategy**, not a feature.

---

### 2. Why Single-Table Design Exists

Single-table design exists because of how DynamoDB works internally:

- Queries are fast only within a partition
- Joins do not exist
- Network hops are expensive
- Predictability matters more than flexibility

Single-table design ensures:
- Related data lives together
- One Query answers complex questions
- Performance is predictable at scale

This is not an optimization — it’s alignment with DynamoDB’s nature.

---

### 3. Why Single-Table Design Is Powerful

When done correctly, single-table design gives you:

- One Query instead of many
- No joins
- No transactions (often)
- No scans
- Linear scalability
- Simple operational model

Example outcome:
- Fetch user profile + orders + settings in one Query
- Fetch post + comments in one Query
- Fetch leaderboard slice in one Query

This is why it’s powerful.

---

### 4. Why Single-Table Design Is Tricky

Single-table design is tricky because:

- You must think upfront
- You must understand access patterns deeply
- Mistakes are architectural, not cosmetic
- Refactoring later is painful

Common emotional reaction:
> “This feels hacky / messy / unreadable”

That feeling is normal.
Clarity comes with mental models.

---

### 5. Core Principle: Item Collections

An **item collection** is:

> All items that share the same partition key.

This is the most important concept in single-table design.

Mental model:
- Partition key = owner
- Item collection = everything owned by that entity

Example:

	PK = "USER#123"

Item collection for USER#123:
- Profile
- Orders
- Sessions
- Preferences
- Notifications

Everything related to that user lives together.

---

### 6. Why Item Collections Matter

Item collections enable:
- One-to-many relationships
- Efficient queries
- Atomic-looking reads
- Natural grouping

DynamoDB is optimized to:
- Fetch an entire item collection
- Or a slice of it

This is intentional.

---

### 7. Adjacency Lists

An **adjacency list** is a way to represent relationships using keys.

Instead of:
- Foreign keys
- Join tables

We use:
- Partition key for grouping
- Sort key for relationship encoding

Example (user → orders):

	PK = "USER#123"
	SK = "PROFILE"

	PK = "USER#123"
	SK = "ORDER#2024-01-01"

	PK = "USER#123"
	SK = "ORDER#2024-01-10"

This adjacency list:
- Encodes relationship implicitly
- Needs no join
- Scales naturally

---

### 8. Pattern 1: One-to-Many 

One-to-many is the most common pattern.

Examples:
- User → Orders
- Post → Comments
- Account → Transactions

Pattern:

	PK = "PARENT#ID"
	SK = "CHILD#<something sortable>"

Why this works:
- All children grouped
- Sort key orders children
- Query retrieves all or some children

This pattern underpins almost everything else.

---

### 9. Pattern 2: Many-to-Many 

Many-to-many examples:
- Students ↔ Courses
- Users ↔ Groups
- Products ↔ Categories

In DynamoDB:
- You model the relationship explicitly
- Often with **duplicated items**

Example (user ↔ group):

User perspective:

	PK = "USER#123"
	SK = "GROUP#A"

Group perspective:

	PK = "GROUP#A"
	SK = "USER#123"

Yes, data is duplicated.
That is correct.
That is intentional.

This allows:
- Query groups for a user
- Query users for a group
- No scans
- No joins

---

### 10. Pattern 3: Polymorphic Entities

A polymorphic table stores:
- Different entity types
- In the same table

Examples:
- USER, ORDER, PRODUCT, SESSION
- All in one table

How this works:
- Entity type encoded in keys
- Attributes differ per type

Example:

	PK = "USER#123"
	SK = "PROFILE"
	type = "USER"

	PK = "USER#123"
	SK = "ORDER#1"
	type = "ORDER"

This is not messy.
This is structured polymorphism.

---

### 11. Why Polymorphic Design Is Powerful

Polymorphic design enables:
- Fewer tables
- Rich item collections
- Flexible evolution
- Sparse attributes

But it requires:
- Clear item roles
- Strong naming conventions
- Discipline

Without discipline, it becomes chaos.

---

### 12. Pattern 4: Time-Series Data

Time-series examples:
- Logs
- Events
- Metrics
- Activity feeds

Pattern:

	PK = "ENTITY#ID"
	SK = "EVENT#<ISO_TIMESTAMP>"

Benefits:
- Natural ordering
- Range queries
- “Latest N” queries

Advanced variation:
- Time bucketing (daily / monthly)
- To avoid hot partitions

This pattern is used everywhere in DynamoDB.

---

### 13. Pattern 5: Leaderboards

Leaderboards require:
- Sorted access
- Top-N queries

Common pattern:

	PK = "LEADERBOARD#GAME#1"
	SK = "<SCORE>#<USER_ID>"

Example:

	SK = "00009876#USER#123"

Why zero-padding:
- Ensures correct string sorting
- Higher scores appear first (or last, by design)

Leaderboards often combine:
- Sort keys
- GSIs
- Time buckets

They require careful design but are very doable.

---

### 14. Multi-Item Patterns: Why One Entity ≠ One Item

In DynamoDB:
- One real-world entity often maps to multiple items

Example:
- User profile item
- User preferences item
- User settings item

Why split?
- Item size limit
- Independent update frequency
- Cleaner access patterns

Single-table design is about:
**logical grouping, not physical singularity**

---

### 15. Common Single-Table Anti-Patterns

Treating it like a JSON blob store  
Random sort keys  
Mixing unrelated entities in one partition  
No naming conventions  
Using Scan to “figure things out”  

Single-table design rewards clarity and planning.

---

### 16. Mental Model Upgrade

At this stage, your DynamoDB mental model should be:

- Table = application boundary
- Partition key = owner / scope
- Item collection = related data
- Sort key = relationship + order
- Duplicate data = normal
- One Query = success

