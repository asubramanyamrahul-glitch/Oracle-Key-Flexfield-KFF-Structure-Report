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

   ```sql
SELECT 
FIF.NAME KEY_FLEXFIELD_NAME                   
,FIF.KEY_FLEXFIELD_CODE                 
,FKFS.NAME STRUCTURE_NAME                       
,FKFS.STRUCTURE_CODE STRUCTURE_CODE                       
,FKFS.ENABLED_FLAG STRUCTURE_ENABLED_FLAG               
,FKFS.DELIMITER CONCATENATED_SEGMENT_DELIMITER       
,FKSV.SEQUENCE_NUMBER  SEGMENT_SEQUENCE_NUM                 
,FKSV.NAME SEGMENT_NAME                         
,FKSV.SEGMENT_CODE SEGMENT_CODE                         
,FKSV.COLUMN_NAME SEGMENT_COLUMN_NAME                  
,FKSV.PROMPT SEGMENT_PROMPT                       
,FKSV.SHORT_PROMPT SEGMENT_SHORT_PROMPT                 
,FKSV.ENABLED_FLAG SEGMENT_ENABLED_FLAG                 
,FFVV.FLEX_VALUE_SET_NAME  DEFAULT_VALUE_SET_CODE               
,FKSI.REQUIRED_FLAG                         
,FKSI.DISPLAY_FLAG DISPLAYED_FLAG                       
,FKSV.CREATION_DATE                        
,FKSV.CREATED_BY                           
,FKSV.LAST_UPDATE_DATE                     
,FKSV.LAST_UPDATED_BY  
from    FND_KF_FLEXFIELDS_VL FIF    
       ,FND_KF_STRUCTURES_VL FKFS 
       ,FND_KF_SEGMENTS_VL FKSV
	   ,FND_KF_SEGMENT_INSTANCES FKSI
	   ,FND_FLEX_VALUE_SETS FFVV
where   FIF.APPLICATION_ID = FKFS.APPLICATION_ID
and     FIF.KEY_FLEXFIELD_CODE    = FKFS.KEY_FLEXFIELD_CODE 
and     FIF.KEY_FLEXFIELD_CODE   IN  (:p_kff_code) 
--and     FIF.NAME LIKE 'Location Flexfield'
AND FKFS.STRUCTURE_ID =  FKSV.STRUCTURE_ID
AND FKSV.segment_code = FKSI.segment_code
AND  FKSV.DEFAULT_VALUE_SET_ID = FFVV.flex_value_set_id
--AND  FKSV.last_update_date  >=  :p_last_refresh_date
order by FIF.KEY_FLEXFIELD_CODE ,FKFS.STRUCTURE_CODE , FKSV.SEQUENCE_NUMBER
