## Amazon S3 Glacier Retrieval Types and Costs: Expedited, Standard, and Bulk Retrieval

### 1. Introduction

Amazon S3 Glacier storage classes are designed for **long-term archival storage** where data is rarely accessed but must still be preserved for extended periods. These storage tiers offer extremely low storage costs compared to standard storage classes.

However, because archival storage prioritizes cost efficiency rather than immediate access, retrieving objects from Glacier storage is slower and involves additional retrieval processes.

To support different access needs, Amazon S3 Glacier provides multiple **retrieval options**. These retrieval types determine:

- how quickly the data becomes available
- how much the retrieval operation costs
- how suitable the retrieval method is for different workloads

The three primary Glacier retrieval types are:

- Expedited Retrieval
- Standard Retrieval
- Bulk Retrieval

Each option represents a different balance between retrieval speed and cost.

---

### 2. Why Glacier Retrieval Options Exist

Archival storage systems are optimized to store large volumes of data at extremely low cost. Achieving this low cost involves using storage infrastructure that prioritizes capacity and durability rather than immediate access.

Because of this architecture, retrieving archived data may require preparing the data internally before it becomes accessible.

Different retrieval types exist so that users can choose between:

- faster retrieval when data is urgently needed
- slower retrieval when cost savings are more important

This allows organizations to retrieve data efficiently depending on the situation.

---

### 3. Retrieval Workflow in Glacier

When an object is stored in Glacier storage classes such as:

- Glacier Flexible Retrieval
- Glacier Deep Archive

the object cannot be downloaded immediately using a normal request.

Instead, a **restore request** must be initiated.

The restore process works as follows:

Object stored in Glacier storage

A restore request is initiated

AWS prepares the object for retrieval

The object becomes temporarily accessible in S3

The speed of this process depends on the retrieval type chosen.

---

### 4. Expedited Retrieval

Expedited retrieval is designed for situations where archived data must be retrieved quickly.

This retrieval type provides the fastest access to Glacier objects.

Typical characteristics include:

Retrieval time of a few minutes

Higher cost compared to other retrieval options

Limited availability during extremely high demand

Expedited retrieval is intended for urgent scenarios where waiting hours is not acceptable.

---

### 5. Example Use Case for Expedited Retrieval

Consider a scenario where an organization stores historical financial records in Glacier for compliance.

If an unexpected regulatory audit occurs and records must be retrieved immediately, expedited retrieval allows the organization to access those records within minutes.

Typical scenarios include:

- emergency data recovery
- urgent compliance investigations
- legal requests for archived records
- disaster recovery operations

In these cases, speed is more important than cost.

---

### 6. Standard Retrieval

Standard retrieval provides a balance between cost and retrieval speed.

Typical characteristics include:

Retrieval time of several hours

Moderate retrieval cost

Suitable for most archival retrieval scenarios

Standard retrieval is the most commonly used Glacier retrieval option.

It is appropriate when archived data is needed but not urgently.

---

### 7. Example Use Case for Standard Retrieval

Consider a company performing periodic audits on historical system logs stored in Glacier.

These logs may be several months or years old and only accessed occasionally.

In such cases, waiting several hours for the data to become available is acceptable.

Typical scenarios include:

- internal audits
- historical data analysis
- periodic reporting
- compliance checks

Standard retrieval balances cost efficiency with reasonable retrieval speed.

---

### 8. Bulk Retrieval

Bulk retrieval is the lowest-cost retrieval option available for Glacier.

However, it also provides the slowest retrieval time.

Typical characteristics include:

Retrieval time ranging from several hours to over half a day

Very low retrieval cost

Designed for retrieving large volumes of archived data

Bulk retrieval is ideal when large datasets need to be retrieved but time is not critical.

---

### 9. Example Use Case for Bulk Retrieval

Consider a research organization storing scientific datasets in Glacier.

Once per year, the organization may retrieve large historical datasets for analysis.

Because the retrieval is planned in advance, waiting several hours for the data is acceptable.

Bulk retrieval allows the organization to retrieve large datasets at minimal cost.

Typical scenarios include:

- large historical data analysis
- archive migrations
- long-term research datasets
- periodic system reviews

---

### 10. Conceptual Comparison of Retrieval Types

The three retrieval options can be visualized along a spectrum of speed versus cost.

Fastest retrieval

```
Expedited Retrieval
```

Balanced option

```
Standard Retrieval
```

Lowest cost retrieval

```
Bulk Retrieval
```

As retrieval speed increases, the cost per retrieval request also increases.

---

### 11. Retrieval Type Selection Strategy

Choosing the correct retrieval type depends on several factors.

Urgency of access

If the data must be retrieved immediately, expedited retrieval is appropriate.

Tolerance for waiting

If several hours of delay is acceptable, standard retrieval is usually sufficient.

Volume of data

If retrieving very large datasets, bulk retrieval provides the lowest cost.

Planning retrievals based on these factors ensures efficient archival storage usage.

---

### 12. Relationship to Glacier Storage Classes

Glacier retrieval options are primarily used with the following storage classes:

S3 Glacier Flexible Retrieval

Objects stored in this class require restore operations before access.

S3 Glacier Deep Archive

Objects stored here have even longer retrieval times.

Some Glacier storage classes such as **Glacier Instant Retrieval** provide immediate access and therefore do not require retrieval operations.

---

### 13. Temporary Restore Period

When a Glacier object is restored, it becomes temporarily available in S3 for a specified duration.

Example restore duration:

```
7 days
```

During this period, the object behaves like a normal S3 object and can be downloaded.

After the restore period ends, the object returns to its archived state.

---

### 14. Cost Considerations

Retrieval costs depend on several factors.

These include:

Retrieval speed

Expedited retrieval costs more than standard or bulk.

Volume of data retrieved

Larger retrievals increase costs.

Number of requests

Frequent retrieval requests may increase operational expenses.

Because Glacier storage is designed for rarely accessed data, retrieval costs must be considered when designing archival strategies.

---

### 15. Avoiding Frequent Glacier Retrieval

Glacier storage classes should be used only for data that is rarely accessed.

If data must be accessed frequently, it should remain in storage classes such as:

- S3 Standard
- S3 Intelligent-Tiering
- S3 Standard-IA

Frequent retrieval from Glacier may result in higher operational costs.

---

### 16. Glacier Retrieval Mental Model

A useful way to think about Glacier retrieval is as a **request to temporarily thaw archived data**.

The data is not immediately available because it is stored in a cost-optimized archival system.

Once a retrieval request is made, AWS prepares the object for temporary access.

The retrieval type determines how quickly this preparation occurs.

---

### 17. Retrieval Workflow Visualization

Conceptual retrieval workflow:

```
Object stored in Glacier
        ↓
Restore request initiated
        ↓
Retrieval type selected
        ↓
AWS prepares archived data
        ↓
Object becomes temporarily available
        ↓
Object downloaded
```

The retrieval type determines how long the preparation stage takes.

---

### 18. Planning Glacier Retrieval Strategies

Organizations using archival storage should plan retrieval strategies carefully.

Important considerations include:

How quickly archived data may need to be accessed.

Whether retrievals are planned or emergency operations.

How large archived datasets are.

Choosing the correct retrieval option helps balance cost and operational efficiency.

---

### 19. Real-World Archival Example

Consider a financial institution storing transaction history for regulatory compliance.

Data older than one year is archived in Glacier.

Most of the time the data remains untouched.

If auditors request historical records, the institution may use standard retrieval.

If an urgent investigation occurs, expedited retrieval may be used.

For large-scale annual audits, bulk retrieval may be used.

This flexible retrieval system allows organizations to adapt to different scenarios.

---

### 20. Summary

Amazon S3 Glacier storage classes provide extremely low-cost archival storage, but retrieving archived data requires initiating a restore operation.

Three main retrieval types are available:

Expedited retrieval

Provides the fastest access, typically within minutes, but at higher cost.

Standard retrieval

Provides moderate retrieval speed, usually within several hours, at a balanced cost.

Bulk retrieval

Provides the lowest-cost retrieval option but may take many hours.

These retrieval options allow organizations to balance retrieval speed and cost depending on the urgency and scale of data access.
