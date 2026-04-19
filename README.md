#  Oracle Key Flexfield (KFF) Structure Report

##  Overview

This SQL script extracts detailed metadata about **Key Flexfields (KFF)** in Oracle Applications (EBS/Fusion).

It provides a complete breakdown of:
- Flexfield definitions
- Structures
- Segments
- Segment properties
- Associated value sets

---

##  What This Report Does

The query generates a structured view of Key Flexfields by combining data from Oracle metadata tables.

Each row in the output represents a **single segment within a flexfield structure**, along with its configuration and properties.

---

##  Key Components

### 1. Flexfield Details
- Flexfield Name
- Flexfield Code

### 2. Structure Information
- Structure Name
- Structure Code
- Enabled Flag
- Segment Delimiter

### 3. Segment Information
- Sequence Number
- Segment Name & Code
- Column Name
- Prompt / Short Prompt
- Enabled Flag

### 4. Segment Behavior
- Required Flag
- Displayed Flag

### 5. Value Set
- Default Value Set Name

---

##  Tables Used

| Table Name | Description |
|-----------|------------|
| FND_KF_FLEXFIELDS_VL | Flexfield definitions |
| FND_KF_STRUCTURES_VL | Flexfield structures |
| FND_KF_SEGMENTS_VL | Segment definitions |
| FND_KF_SEGMENT_INSTANCES | Segment properties |
| FND_FLEX_VALUE_SETS | Value set details |

---

##  Parameters

| Parameter | Description |
|----------|------------|
| `:p_kff_code` | Key Flexfield Code to filter results |

---

## Output

The report returns:

- Flexfield → Structure → Segment hierarchy
- Segment ordering via sequence number
- Validation via value sets
- Flags indicating segment behavior

---

##  Use Cases

-  Functional documentation of KFF
-  Troubleshooting flexfield issues
-  Data migration and mapping
-  Auditing configurations
-  Developer reference

---

## How to Use

1. Provide the flexfield code:
   ```sql
   :p_kff_code
Alright—let’s go deeper than a surface explanation and actually **walk through this query like a developer/debugger would**. By the end, you should be able to explain *every column, join, and design decision* in an interview or code review.

---

#  1. High-Level Purpose (Technical View)

This query builds a **hierarchical metadata model** of a Key Flexfield:

```
Flexfield → Structure → Segment → Segment Properties → Value Set
```

Each row = **one segment instance inside a specific structure of a flexfield**

---

# 2. FROM Clause (Core Data Sources)

```sql
FROM FND_KF_FLEXFIELDS_VL FIF,
     FND_KF_STRUCTURES_VL FKFS,
     FND_KF_SEGMENTS_VL FKSV,
     FND_KF_SEGMENT_INSTANCES FKSI,
     FND_FLEX_VALUE_SETS FFVV
```

### What each alias represents:

| Alias  | Table             | Role                        |
| ------ | ----------------- | --------------------------- |
| `FIF`  | Flexfields        | Top-level definition        |
| `FKFS` | Structures        | Configurations of flexfield |
| `FKSV` | Segments          | Fields inside structure     |
| `FKSI` | Segment Instances | Behavior per segment        |
| `FFVV` | Value Sets        | Validation rules            |

 Think of it like:

```
FIF (header)
 └── FKFS (versions/configs)
      └── FKSV (columns/segments)
           └── FKSI (rules)
           └── FFVV (validation)
```

---

# 🔗 3. JOIN CONDITIONS (Critical Logic)

##  3.1 Flexfield → Structure

```sql
FIF.APPLICATION_ID = FKFS.APPLICATION_ID
AND FIF.KEY_FLEXFIELD_CODE = FKFS.KEY_FLEXFIELD_CODE
```

### Why this matters:

* Flexfields are identified by:

  * `APPLICATION_ID`
  * `KEY_FLEXFIELD_CODE`

 This ensures:

> “Give me structures that belong to THIS flexfield”

---

##  3.2 Filtering Flexfield

```sql
FIF.KEY_FLEXFIELD_CODE IN (:p_kff_code)
```

* Parameterized input
* Allows dynamic filtering

 Example:

```sql
:p_kff_code = 'GL#'
```

---

##  3.3 Structure → Segments

```sql
FKFS.STRUCTURE_ID = FKSV.STRUCTURE_ID
```

### Meaning:

Each structure has multiple segments

 This creates:

```
Structure A → Segment 1, 2, 3
Structure B → Segment 1, 2
```

---

##  3.4 Segment → Segment Instance

```sql
FKSV.segment_code = FKSI.segment_code
```

###  Important Observation:

This is a **potential weak join**

 Why?

* `SEGMENT_CODE` might not be unique across structures
* Ideally, should also include:

  ```sql
  AND FKSV.STRUCTURE_ID = FKSI.STRUCTURE_ID
  ```

 Otherwise:

> You may get incorrect mapping of segment properties

---

##  3.5 Segment → Value Set

```sql
FKSV.DEFAULT_VALUE_SET_ID = FFVV.flex_value_set_id
```

### Meaning:

Each segment is validated using a **Value Set**

 Example:

* Department → List of valid departments
* Account → Chart of accounts values

---

#  4. SELECT Clause (Column-by-Column Deep Dive)

---

##  4.1 Flexfield Level

```sql
FIF.NAME KEY_FLEXFIELD_NAME
FIF.KEY_FLEXFIELD_CODE
```

* Business name (e.g., "Accounting Flexfield")
* Technical identifier (e.g., `GL#`)

---

##  4.2 Structure Level

```sql
FKFS.NAME STRUCTURE_NAME
FKFS.STRUCTURE_CODE
FKFS.ENABLED_FLAG STRUCTURE_ENABLED_FLAG
FKFS.DELIMITER CONCATENATED_SEGMENT_DELIMITER
```

### Explanation:

| Column         | Meaning                      |
| -------------- | ---------------------------- |
| NAME           | Structure display name       |
| STRUCTURE_CODE | Unique identifier            |
| ENABLED_FLAG   | Active or not                |
| DELIMITER      | Separator (e.g., `.` or `-`) |

 Example:

```
01.100.2000
```

---

##  4.3 Segment Level

```sql
FKSV.SEQUENCE_NUMBER SEGMENT_SEQUENCE_NUM
FKSV.NAME SEGMENT_NAME
FKSV.SEGMENT_CODE
FKSV.COLUMN_NAME SEGMENT_COLUMN_NAME
```

### Key Concepts:

| Column          | Meaning          |
| --------------- | ---------------- |
| SEQUENCE_NUMBER | Order of segment |
| NAME            | Segment label    |
| SEGMENT_CODE    | Internal ID      |
| COLUMN_NAME     | DB column        |

 Example:

```
Segment 1 → COMPANY → SEGMENT1
Segment 2 → DEPARTMENT → SEGMENT2
```

---

##  4.4 UI Labels

```sql
FKSV.PROMPT SEGMENT_PROMPT
FKSV.SHORT_PROMPT SEGMENT_SHORT_PROMPT
```

* Used in UI screens
* Helps users understand input fields

---

##  4.5 Segment Status

```sql
FKSV.ENABLED_FLAG SEGMENT_ENABLED_FLAG
```

* Whether segment is active

---

##  4.6 Value Set

```sql
FFVV.FLEX_VALUE_SET_NAME DEFAULT_VALUE_SET_CODE
```

### Purpose:

Defines **validation logic**

 Example:

* Independent values
* Table-based values
* Hierarchical values

---

##  4.7 Segment Behavior

```sql
FKSI.REQUIRED_FLAG
FKSI.DISPLAY_FLAG DISPLAYED_FLAG
```

### Meaning:

| Flag          | Purpose         |
| ------------- | --------------- |
| REQUIRED_FLAG | Mandatory input |
| DISPLAY_FLAG  | Visible in UI   |

---

##  4.8 Audit Columns

```sql
FKSV.CREATION_DATE
FKSV.CREATED_BY
FKSV.LAST_UPDATE_DATE
FKSV.LAST_UPDATED_BY
```

* Useful for tracking changes
* Helps in debugging and auditing

---

#  5. ORDER BY Clause

```sql
ORDER BY 
  FIF.KEY_FLEXFIELD_CODE,
  FKFS.STRUCTURE_CODE,
  FKSV.SEQUENCE_NUMBER
```

### Why this is important:

Ensures logical hierarchy:

```
Flexfield
 └── Structure
      └── Segment (in correct order)
```

---


---


