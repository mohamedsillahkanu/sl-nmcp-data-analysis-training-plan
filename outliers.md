# Outlier Correction: Before vs After Aggregation

## Overview

When working with composite variables (like `susp_u5` + `susp_ov5` = `total_susp`), the timing of outlier correction significantly impacts data integrity and analytical capabilities.

## General Principle

**Recommendation: Correct outliers BEFORE aggregation**

This approach maintains mathematical relationships between variables and preserves analytical flexibility.

## Example: Patient Suspected Cases Data

### Original Dataset
```
ID | susp_u5 | susp_ov5 | total_susp
1  | 2       | 8        | 10
2  | 1       | 7        | 8
3  | 50      | 9        | 59     # outlier in susp_u5
4  | 3       | 200      | 203    # outlier in susp_ov5  
5  | 2       | 6        | 8
```

## Method 1: Correct Components First (Recommended)

### Step 1: Identify and correct outliers in component variables
```
susp_u5 outliers: 50 (ID 3)
susp_ov5 outliers: 200 (ID 4)

Correction method: Replace with median
- susp_u5 median: 2
- susp_ov5 median: 8
```

### Step 2: Apply corrections
```
ID | susp_u5_clean | susp_ov5_clean
1  | 2             | 8
2  | 1             | 7  
3  | 2             | 9     # 50 → 2
4  | 3             | 8     # 200 → 8
5  | 2             | 6
```

### Step 3: Create total from cleaned components
```
ID | susp_u5_clean | susp_ov5_clean | total_susp_clean
1  | 2             | 8              | 10
2  | 1             | 7              | 8
3  | 2             | 9              | 11
4  | 3             | 8              | 11  
5  | 2             | 6              | 8
```

### Results
- **Mean total_susp:** 9.6
- **Mathematical consistency:** ✅ susp_u5 + susp_ov5 = total_susp
- **Analytical capability:** ✅ Can analyze age group patterns

## Method 2: Correct Total After Creation

### Step 1: Create total from original data
```
total_susp: [10, 8, 59, 203, 8]
```

### Step 2: Correct outliers in total
```
total_susp outliers: 59, 203
Correction: Replace with median (8)

total_susp_corrected: [10, 8, 8, 8, 8]
```

### Problems with this approach
```
ID | susp_u5 | susp_ov5 | total_susp_corrected | Relationship Check
1  | 2       | 8        | 10                   | ✅ 2 + 8 = 10
2  | 1       | 7        | 8                    | ✅ 1 + 7 = 8  
3  | 50      | 9        | 8                    | ❌ 50 + 9 ≠ 8
4  | 3       | 200      | 8                    | ❌ 3 + 200 ≠ 8
5  | 2       | 6        | 8                    | ✅ 2 + 6 = 8
```

### Results
- **Mean total_susp:** 8.4 (different from Method 1!)
- **Mathematical consistency:** ❌ Broken relationships
- **Analytical capability:** ❌ Cannot analyze components meaningfully

## Key Differences

| Aspect | Method 1 (Before) | Method 2 (After) |
|--------|-------------------|------------------|
| **Data Integrity** | Preserved | Compromised |
| **Component Analysis** | Possible | Limited/Invalid |
| **Mathematical Consistency** | Maintained | Broken |
| **Statistical Results** | Reliable | Potentially biased |
| **Validation Capability** | Full | Restricted |

## When Method 2 Might Be Acceptable

- **Single variable analysis only:** If you will never analyze component variables
- **Pre-aggregated data:** When working with data that's already summarized
- **Aggregate-level outliers:** When outliers only appear at the total level

## Best Practices

### For Composite Variables (susp_u5 + susp_ov5 = total_susp)

1. **Clean component variables first**
   ```python
   # Detect outliers in each component
   susp_u5_clean = remove_outliers(susp_u5)
   susp_ov5_clean = remove_outliers(susp_ov5)
   ```

2. **Create aggregated variable from cleaned components**
   ```python
   total_susp = susp_u5_clean + susp_ov5_clean
   ```

3. **Validate mathematical relationships**
   ```python
   assert all(susp_u5_clean + susp_ov5_clean == total_susp)
   ```

4. **Document the cleaning process**
   - Which variables had outliers
   - Correction method used
   - Number of values modified

## Conclusion

For composite variables like `susp_u5 + susp_ov5 = total_susp`, correcting outliers before aggregation:

- **Maintains data integrity**
- **Preserves analytical flexibility** 
- **Ensures consistent results**
- **Enables comprehensive validation**

The small additional effort in cleaning component variables first pays significant dividends in data quality and analytical capability.
