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
