## A24: Oracle CONNECT BY — Hierarchical Queries

### The Problem It Solves

Some data has natural tree/hierarchy structure — parent/child relationships stored in a single table via a self-referencing foreign key.

Examples:
- Employee → Manager → Director → CEO
- Container route: Origin Port → Transshipment 1 → Transshipment 2 → Destination
- Organizational units: Department → Team → Sub-team
- Product categories: Electronics → Phones → Smartphones

Standard SQL can't traverse these easily without recursive queries. Oracle's `CONNECT BY` makes it one-liner.

### Basic Syntax

```sql
SELECT col1, col2, LEVEL
FROM   table_name
START WITH condition_for_root
CONNECT BY PRIOR parent_col = child_col;
```

- **`START WITH`** — defines the root rows (where traversal starts)
- **`CONNECT BY PRIOR`** — defines the parent-child relationship
- **`LEVEL`** — pseudocolumn showing depth (root = 1, its children = 2, etc.)
- **`PRIOR`** — refers to the parent row in the hierarchy

### Logistics Example — Container Transshipment Route

Suppose you have a table tracking every leg a container takes:

```sql
CREATE TABLE shipment_leg (
    leg_id       NUMBER PRIMARY KEY,
    container_no VARCHAR2(20),
    from_port    VARCHAR2(10),
    to_port      VARCHAR2(10),
    prev_leg_id  NUMBER,   -- links to the leg before this one
    vessel       VARCHAR2(50),
    etd          DATE,
    eta          DATE
);
```

A container going HCMC → Singapore → Rotterdam has 2 rows, where leg 2 has `prev_leg_id = leg 1's id`.

**Query to trace the full route for a container:**
```sql
SELECT LEVEL AS leg_number,
       from_port,
       to_port,
       vessel,
       SYS_CONNECT_BY_PATH(from_port || '->' || to_port, ' | ') AS full_path
FROM   shipment_leg
WHERE  container_no = 'MSCU1234567'
START WITH prev_leg_id IS NULL            -- start from the first leg
CONNECT BY PRIOR leg_id = prev_leg_id;    -- follow the chain forward
```

Output:
```
LEG_NUMBER | FROM_PORT | TO_PORT   | VESSEL       | FULL_PATH
-----------+-----------+-----------+--------------+---------------------------------
1          | HCMC      | SINGAPORE | ONE Apus      | HCMC->SINGAPORE
2          | SINGAPORE | JEBEL ALI | MSC Oscar     | HCMC->SINGAPORE | SINGAPORE->JEBEL ALI
3          | JEBEL ALI | ROTTERDAM | Ever Given    | HCMC->SINGAPORE | SINGAPORE->JEBEL ALI | JEBEL ALI->ROTTERDAM
```

`SYS_CONNECT_BY_PATH` builds the accumulated path string — very useful for display.

### Employee Hierarchy Example (Classic)

```sql
SELECT LPAD(' ', 2*(LEVEL-1)) || employee_name AS org_tree,
       manager_id,
       LEVEL
FROM   employees
START WITH manager_id IS NULL                  -- the CEO
CONNECT BY PRIOR employee_id = manager_id;     -- traverse down
```

Output indents each level:
```
CEO
  VP Operations
    Regional Manager North
      Team Lead A
      Team Lead B
    Regional Manager South
  VP Engineering
```

### Direction Matters — PRIOR Position

**Top-down** (root to leaves):
```sql
CONNECT BY PRIOR parent_id = id;  -- PRIOR is on parent side
```

**Bottom-up** (leaf to root):
```sql
CONNECT BY PRIOR id = parent_id;  -- PRIOR is on child side
```

Switching the PRIOR keyword changes the traversal direction.

### Key Features

**1. `LEVEL` — Depth in the hierarchy**
```sql
WHERE LEVEL <= 3  -- only go 3 levels deep
```

**2. `ORDER SIBLINGS BY` — Sort children at each level**
```sql
...
CONNECT BY PRIOR id = parent_id
ORDER SIBLINGS BY name;
```
Regular `ORDER BY` breaks the hierarchy output. `ORDER SIBLINGS BY` preserves it.

**3. `CONNECT_BY_ROOT` — Root value for each row**
```sql
SELECT CONNECT_BY_ROOT employee_name AS root_manager, employee_name, LEVEL
FROM employees
START WITH manager_id IS NULL
CONNECT BY PRIOR employee_id = manager_id;
```
Every row shows which root it descends from.

**4. `CONNECT_BY_ISLEAF` — 1 if leaf node, 0 otherwise**
```sql
SELECT employee_name, CONNECT_BY_ISLEAF AS is_leaf
FROM employees
CONNECT BY PRIOR employee_id = manager_id;
```

**5. `NOCYCLE` — Prevent infinite loops**
If your data has accidental cycles (A → B → A), the query runs forever. `NOCYCLE` stops at cycles:
```sql
CONNECT BY NOCYCLE PRIOR parent_id = id;
```
Combined with `CONNECT_BY_ISCYCLE` to flag cyclic rows.

### Modern Alternative — Recursive CTE

Oracle 11gR2+ also supports SQL standard recursive CTEs:
```sql
WITH route (leg_id, from_port, to_port, lvl) AS (
    SELECT leg_id, from_port, to_port, 1
    FROM   shipment_leg
    WHERE  container_no = 'MSCU1234567' AND prev_leg_id IS NULL
    UNION ALL
    SELECT s.leg_id, s.from_port, s.to_port, r.lvl + 1
    FROM   shipment_leg s
    JOIN   route r ON s.prev_leg_id = r.leg_id
)
SELECT * FROM route;
```

Recursive CTEs are the SQL standard and portable across databases. CONNECT BY is Oracle-specific but more concise.

### Common Pitfalls

- **Infinite loop** — cycles in data + no `NOCYCLE` = query never returns
- **Wrong PRIOR direction** — you get siblings instead of descendants, or nothing at all
- **`WHERE` vs `CONNECT BY` filter** — `WHERE` filters the final result; adding conditions to `CONNECT BY` filters the traversal itself
- **Performance** — on very deep/wide hierarchies, can be slow. Index `parent_id` / `prev_leg_id`.

### Use Cases in Logistics

1. **Container transshipment route** — trace all legs a container takes
2. **Voyage → Port call → Container relationships**
3. **Organizational hierarchy** — who reports to whom
4. **Product categorization** — parent-child categories
5. **Customer/partner hierarchy** — parent company, subsidiaries, branches
6. **Shipping location hierarchy** — Country → Region → City → Port → Terminal

### Interview-Ready Answer

> "Oracle's CONNECT BY is for hierarchical queries — querying tree-structured data stored via self-referencing foreign keys. Syntax: START WITH picks the roots, CONNECT BY PRIOR defines parent-child direction. The LEVEL pseudocolumn tells you the depth.
>
> In logistics, I'd use it to trace a container's transshipment route — each leg has a prev_leg_id pointing to the previous leg. One query with CONNECT BY PRIOR leg_id = prev_leg_id gives me the full route from origin to destination in order.
>
> Recursive CTEs with WITH ... UNION ALL are the SQL standard alternative — portable across databases but more verbose. CONNECT BY is Oracle-only but concise.
>
> Watch out for cycles in data — without NOCYCLE, a bad row like A→B→A loops forever."
