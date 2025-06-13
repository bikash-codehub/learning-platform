# Row to Column Zero - Complete Solution Guide

## Problem Statement

Given a 2D integer matrix A, make all elements in a row or column zero if A[i][j] = 0. Specifically, make the entire ith row and jth column zero.

**Constraints:**
- 1 <= A.size() <= 10³
- 1 <= A[i].size() <= 10³
- 0 <= A[i][j] <= 10³

**Example:**
```
Input:  [[1,2,3,4],
         [5,6,7,0],
         [9,2,0,4]]

Output: [[1,2,0,0],
         [0,0,0,0],
         [0,0,0,0]]
```

---

## Approach 1: Brute Force (Extra Space)

### Algorithm
1. Scan the matrix to find all positions where A[i][j] = 0
2. Store these positions in separate arrays/sets
3. Iterate through the matrix again and set elements to 0 if their row or column index is in the stored positions

### Visual Diagram
```
Step 1: Find zeros
[1,2,3,4]    Zero positions: (1,3), (2,2)
[5,6,7,0] ← Zero at (1,3)
[9,2,0,4] ← Zero at (2,2)
   ↑
   Zero at (2,2)

Step 2: Mark rows and columns
Rows to zero: {1, 2}
Cols to zero: {2, 3}

Step 3: Apply changes
[1,2,0,0] ← Col 2,3 = 0
[0,0,0,0] ← Row 1 = 0
[0,0,0,0] ← Row 2 = 0
```

### Code Implementation
```python
def setZeroes_bruteforce(matrix):
    if not matrix or not matrix[0]:
        return matrix
    
    rows, cols = len(matrix), len(matrix[0])
    zero_rows = set()
    zero_cols = set()
    
    # Step 1: Find all zero positions
    for i in range(rows):
        for j in range(cols):
            if matrix[i][j] == 0:
                zero_rows.add(i)
                zero_cols.add(j)
    
    # Step 2: Set rows to zero
    for row in zero_rows:
        for j in range(cols):
            matrix[row][j] = 0
    
    # Step 3: Set columns to zero
    for col in zero_cols:
        for i in range(rows):
            matrix[i][col] = 0
    
    return matrix
```

### Dry Run
**Input:** `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]`

| Step | Action | zero_rows | zero_cols | Matrix State |
|------|--------|-----------|-----------|--------------|
| 1 | Scan (0,0)=1 | {} | {} | Original |
| 2 | Scan (0,1)=2 | {} | {} | Original |
| 3 | Scan (0,2)=3 | {} | {} | Original |
| 4 | Scan (0,3)=4 | {} | {} | Original |
| 5 | Scan (1,0)=5 | {} | {} | Original |
| 6 | Scan (1,1)=6 | {} | {} | Original |
| 7 | Scan (1,2)=7 | {} | {} | Original |
| 8 | **Scan (1,3)=0** | **{1}** | **{3}** | Original |
| 9 | Scan (2,0)=9 | {1} | {3} | Original |
| 10 | Scan (2,1)=2 | {1} | {3} | Original |
| 11 | **Scan (2,2)=0** | **{1,2}** | **{2,3}** | Original |
| 12 | Scan (2,3)=4 | {1,2} | {2,3} | Original |
| 13 | Zero row 1 | {1,2} | {2,3} | `[[1,2,3,4], [0,0,0,0], [9,2,0,4]]` |
| 14 | Zero row 2 | {1,2} | {2,3} | `[[1,2,3,4], [0,0,0,0], [0,0,0,0]]` |
| 15 | Zero col 2 | {1,2} | {2,3} | `[[1,2,0,4], [0,0,0,0], [0,0,0,0]]` |
| 16 | Zero col 3 | {1,2} | {2,3} | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` |

**Time Complexity:** O(m×n) where m = rows, n = columns  
**Space Complexity:** O(m+n) for storing row and column indices

---

## Approach 2: Optimized (Constant Space)

### Algorithm
1. Use the first row and first column of the matrix itself as markers
2. Use two separate variables to track if the first row and first column originally had zeros
3. Mark zeros in the first row/column, then use these markers to set other elements to zero

### Visual Diagram
```
Original Matrix:
[1,2,3,4]
[5,6,7,0]  ← Zero at (1,3)
[9,2,0,4]  ← Zero at (2,2)

Step 1: Check first row and column for zeros
first_row_zero = False (no zeros in row 0)
first_col_zero = False (no zeros in col 0)

Step 2: Use first row/col as markers
[1,2,0,0]  ← matrix[0][2]=0, matrix[0][3]=0 (markers)
[0,6,7,0]  ← matrix[1][0]=0 (marker)
[0,2,0,4]  ← matrix[2][0]=0 (marker)

Step 3: Set zeros based on markers
[1,2,0,0]  ← Keep as is (first row)
[0,0,0,0]  ← Row 1: matrix[1][0]=0, so zero entire row
[0,0,0,0]  ← Row 2: matrix[2][0]=0, so zero entire row
```

### Code Implementation
```python
def setZeroes_optimized(matrix):
    if not matrix or not matrix[0]:
        return matrix
    
    rows, cols = len(matrix), len(matrix[0])
    first_row_zero = False
    first_col_zero = False
    
    # Step 1: Check if first row has zero
    for j in range(cols):
        if matrix[0][j] == 0:
            first_row_zero = True
            break
    
    # Step 2: Check if first column has zero
    for i in range(rows):
        if matrix[i][0] == 0:
            first_col_zero = True
            break
    
    # Step 3: Use first row and column as markers
    for i in range(1, rows):
        for j in range(1, cols):
            if matrix[i][j] == 0:
                matrix[i][0] = 0  # Mark in first column
                matrix[0][j] = 0  # Mark in first row
    
    # Step 4: Set zeros based on markers
    for i in range(1, rows):
        for j in range(1, cols):
            if matrix[i][0] == 0 or matrix[0][j] == 0:
                matrix[i][j] = 0
    
    # Step 5: Handle first row
    if first_row_zero:
        for j in range(cols):
            matrix[0][j] = 0
    
    # Step 6: Handle first column
    if first_col_zero:
        for i in range(rows):
            matrix[i][0] = 0
    
    return matrix
```

### Dry Run
**Input:** `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]`

| Step | Action | first_row_zero | first_col_zero | Matrix State |
|------|--------|----------------|----------------|--------------|
| 1 | Check first row | False | - | Original |
| 2 | Check first col | False | False | Original |
| 3 | Scan (1,1)=6 | False | False | Original |
| 4 | Scan (1,2)=7 | False | False | Original |
| 5 | **Scan (1,3)=0** | False | False | `[[1,2,3,0], [0,6,7,0], [9,2,0,4]]` |
| 6 | Scan (2,1)=2 | False | False | `[[1,2,3,0], [0,6,7,0], [9,2,0,4]]` |
| 7 | **Scan (2,2)=0** | False | False | `[[1,2,0,0], [0,6,7,0], [0,2,0,4]]` |
| 8 | Scan (2,3)=4 | False | False | `[[1,2,0,0], [0,6,7,0], [0,2,0,4]]` |
| 9 | Set (1,1): 0 or 0 | False | False | `[[1,2,0,0], [0,0,7,0], [0,2,0,4]]` |
| 10 | Set (1,2): 0 or 0 | False | False | `[[1,2,0,0], [0,0,0,0], [0,2,0,4]]` |
| 11 | Set (1,3): 0 or 0 | False | False | `[[1,2,0,0], [0,0,0,0], [0,2,0,4]]` |
| 12 | Set (2,1): 0 or 0 | False | False | `[[1,2,0,0], [0,0,0,0], [0,0,0,4]]` |
| 13 | Set (2,2): 0 or 0 | False | False | `[[1,2,0,0], [0,0,0,0], [0,0,0,4]]` |
| 14 | Set (2,3): 0 or 0 | False | False | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` |

**Time Complexity:** O(m×n) where m = rows, n = columns  
**Space Complexity:** O(1) - only using a few extra variables

---

## Complexity Comparison

| Approach | Time Complexity | Space Complexity | Pros | Cons |
|----------|----------------|------------------|------|------|
| Brute Force | O(m×n) | O(m+n) | Simple to understand and implement | Uses extra space |
| Optimized | O(m×n) | O(1) | Constant space, efficient | Slightly more complex logic |

## Key Insights

1. **Brute Force**: Straightforward but uses additional space to store row/column indices
2. **Optimized**: Uses the matrix itself as storage, achieving constant space complexity
3. **Edge Cases**: Handle empty matrices, single row/column matrices
4. **Order Matters**: In the optimized approach, process markers before setting the first row/column to avoid overwriting markers

## Recommended Approach

The **optimized approach** is preferred for:
- Memory-constrained environments
- Large matrices where space efficiency matters
- Interview scenarios where O(1) space is specifically requested

The **brute force approach** is better for:
- Quick prototyping
- When code readability is prioritized over space efficiency
- Learning purposes to understand the problem better