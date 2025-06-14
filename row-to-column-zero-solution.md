# Row to Column Zero - Complete Visual Solution Guide

## 🎯 What's the Problem?

Imagine you have a grid of numbers (like a spreadsheet). Whenever you see a **0** in any cell, you need to:
1. Make the **entire row** of that 0 become all zeros
2. Make the **entire column** of that 0 become all zeros

**Real-world analogy:** Think of it like a "contamination" problem - whenever there's a 0, it "spreads" to contaminate its entire row and column.

### 📝 Quick Visual Understanding

```
BEFORE (Original Matrix):           AFTER (Result):
┌─────────────────────┐            ┌─────────────────────┐
│  1   2   3   4  │   │            │  1   2   0   0  │   │
│  5   6   7  [0] │   │    →       │  0   0   0   0  │   │ ← Row became 0
│  9   2  [0]  4  │   │            │  0   0   0   0  │   │ ← Row became 0
└─────────────────────┘            └─────────────────────┘
        ↑                                  ↑   ↑
   These 0s cause                     Columns became 0
   contamination
```

**Why did this happen?**
- 0 at position (1,3) → Row 1 and Column 3 become zero
- 0 at position (2,2) → Row 2 and Column 2 become zero

---

## 🚀 Approach 1: Simple Way (Brute Force)

### 🎯 The Strategy
Think of it like a 3-step cleaning process:
1. **🔍 SCOUT:** Find all contaminated spots (zeros)
2. **📝 RECORD:** Remember which rows and columns are affected
3. **🧹 CLEAN:** Go back and zero out the affected rows and columns

### 🎬 Animated Step-by-Step Walkthrough

Let's work with: `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]`

#### 🔍 PHASE 1: SCOUTING FOR ZEROS

```
Step 1: Start scanning from top-left
┌─────────────────────┐    Scanning position (0,0)
│ [1]  2   3   4  │   │    Value = 1 → Not zero, continue
│  5   6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 2: Continue scanning...
┌─────────────────────┐    Scanning position (0,1)
│  1  [2]  3   4  │   │    Value = 2 → Not zero, continue
│  5   6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 3: Continue scanning...
┌─────────────────────┐    Scanning position (0,2)
│  1   2  [3]  4  │   │    Value = 3 → Not zero, continue
│  5   6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 4: Continue scanning...
┌─────────────────────┐    Scanning position (0,3)
│  1   2   3  [4] │   │    Value = 4 → Not zero, continue
│  5   6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 5: Continue scanning...
┌─────────────────────┐    Scanning position (1,0)
│  1   2   3   4  │   │    Value = 5 → Not zero, continue
│ [5]  6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 6: Continue scanning...
┌─────────────────────┐    Scanning position (1,1)
│  1   2   3   4  │   │    Value = 6 → Not zero, continue
│  5  [6]  7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 7: Continue scanning...
┌─────────────────────┐    Scanning position (1,2)
│  1   2   3   4  │   │    Value = 7 → Not zero, continue
│  5   6  [7]  0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 8: 🚨 FOUND FIRST ZERO! 🚨
┌─────────────────────┐    Scanning position (1,3)
│  1   2   3   4  │   │    Value = 0 → ZERO FOUND!
│  5   6   7  [0] │   │    📝 Record: Row 1 needs zeroing
│  9   2   0   4  │   │    📝 Record: Column 3 needs zeroing
└─────────────────────┘    
Memory: rows_to_zero = {1}, cols_to_zero = {3}
```

```
Step 9: Continue scanning...
┌─────────────────────┐    Scanning position (2,0)
│  1   2   3   4  │   │    Value = 9 → Not zero, continue
│  5   6   7   0  │   │    
│ [9]  2   0   4  │   │    
└─────────────────────┘    
```

```
Step 10: Continue scanning...
┌─────────────────────┐    Scanning position (2,1)
│  1   2   3   4  │   │    Value = 2 → Not zero, continue
│  5   6   7   0  │   │    
│  9  [2]  0   4  │   │    
└─────────────────────┘    
```

```
Step 11: 🚨 FOUND SECOND ZERO! 🚨
┌─────────────────────┐    Scanning position (2,2)  
│  1   2   3   4  │   │    Value = 0 → ZERO FOUND!
│  5   6   7   0  │   │    📝 Record: Row 2 needs zeroing
│  9   2  [0]  4  │   │    📝 Record: Column 2 needs zeroing
└─────────────────────┘    
Memory: rows_to_zero = {1,2}, cols_to_zero = {2,3}
```

```
Step 12: Continue scanning...
┌─────────────────────┐    Scanning position (2,3)
│  1   2   3   4  │   │    Value = 4 → Not zero, continue
│  5   6   7   0  │   │    🎯 Scanning complete!
│  9   2   0  [4] │   │    
└─────────────────────┘    
Final Memory: rows_to_zero = {1,2}, cols_to_zero = {2,3}
```

#### 🧹 PHASE 2: CLEANING (ZEROING ROWS)

```
Step 13: Zero out Row 1
┌─────────────────────┐    Zeroing entire row 1...
│  1   2   3   4  │   │    Row 1: [5,6,7,0] → [0,0,0,0]
│ [0] [0] [0] [0] │   │    ✅ Row 1 cleaned!
│  9   2   0   4  │   │    
└─────────────────────┘    
Current state: [[1,2,3,4], [0,0,0,0], [9,2,0,4]]
```

```
Step 14: Zero out Row 2  
┌─────────────────────┐    Zeroing entire row 2...
│  1   2   3   4  │   │    Row 2: [9,2,0,4] → [0,0,0,0]
│  0   0   0   0  │   │    ✅ Row 2 cleaned!
│ [0] [0] [0] [0] │   │    
└─────────────────────┘    
Current state: [[1,2,3,4], [0,0,0,0], [0,0,0,0]]
```

#### 🧹 PHASE 3: CLEANING (ZEROING COLUMNS)

```
Step 15: Zero out Column 2
┌─────────────────────┐    Zeroing entire column 2...
│  1   2  [0]  4  │   │    Column 2: [3,0,0] → [0,0,0]
│  0   0  [0]  0  │   │    ✅ Column 2 cleaned!
│  0   0  [0]  0  │   │    
└─────────────────────┘    
Current state: [[1,2,0,4], [0,0,0,0], [0,0,0,0]]
```

```
Step 16: Zero out Column 3
┌─────────────────────┐    Zeroing entire column 3...
│  1   2   0  [0] │   │    Column 3: [4,0,0] → [0,0,0]
│  0   0   0  [0] │   │    ✅ Column 3 cleaned!
│  0   0   0  [0] │   │    
└─────────────────────┘    
FINAL RESULT: [[1,2,0,0], [0,0,0,0], [0,0,0,0]]
```

### 💻 Simple Code with Detailed Comments
```python
def setZeroes_simple(matrix):
    if not matrix:
        return matrix
    
    rows, cols = len(matrix), len(matrix[0])
    
    # Phase 1: Scout for zeros
    zero_rows = set()  # Remember which rows have zeros
    zero_cols = set()  # Remember which columns have zeros
    
    print("🔍 PHASE 1: Scouting for zeros...")
    for i in range(rows):
        for j in range(cols):
            print(f"   Checking position ({i},{j}) = {matrix[i][j]}")
            if matrix[i][j] == 0:
                print(f"   🚨 ZERO FOUND at ({i},{j})!")
                zero_rows.add(i)
                zero_cols.add(j)
                print(f"   📝 Added row {i} and column {j} to cleanup list")
    
    print(f"\n📝 CLEANUP LIST:")
    print(f"   Rows to zero: {sorted(zero_rows)}")
    print(f"   Columns to zero: {sorted(zero_cols)}")
    
    # Phase 2: Zero out rows
    print(f"\n🧹 PHASE 2: Zeroing rows...")
    for row in zero_rows:
        print(f"   Zeroing row {row}: {matrix[row]} → ", end="")
        for j in range(cols):
            matrix[row][j] = 0
        print(f"{matrix[row]} ✅")
    
    # Phase 3: Zero out columns  
    print(f"\n🧹 PHASE 3: Zeroing columns...")
    for col in zero_cols:
        print(f"   Zeroing column {col}...")
        for i in range(rows):
            matrix[i][col] = 0
        print(f"   ✅ Column {col} zeroed!")
    
    return matrix
```

### 🎯 Detailed Dry Run Table

| Step | Position | Value | Action | zero_rows | zero_cols | Matrix State |
|------|----------|-------|---------|-----------|-----------|--------------|
| 1 | (0,0) | 1 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 2 | (0,1) | 2 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 3 | (0,2) | 3 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 4 | (0,3) | 4 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 5 | (1,0) | 5 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 6 | (1,1) | 6 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 7 | (1,2) | 7 | Continue | {} | {} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| **8** | **(1,3)** | **0** | **Found Zero!** | **{1}** | **{3}** | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 9 | (2,0) | 9 | Continue | {1} | {3} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 10 | (2,1) | 2 | Continue | {1} | {3} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| **11** | **(2,2)** | **0** | **Found Zero!** | **{1,2}** | **{2,3}** | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| 12 | (2,3) | 4 | Continue | {1,2} | {2,3} | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` |
| **13** | **Row 1** | **-** | **Zero Row** | {1,2} | {2,3} | `[[1,2,3,4], [0,0,0,0], [9,2,0,4]]` |
| **14** | **Row 2** | **-** | **Zero Row** | {1,2} | {2,3} | `[[1,2,3,4], [0,0,0,0], [0,0,0,0]]` |
| **15** | **Col 2** | **-** | **Zero Col** | {1,2} | {2,3} | `[[1,2,0,4], [0,0,0,0], [0,0,0,0]]` |
| **16** | **Col 3** | **-** | **Zero Col** | {1,2} | {2,3} | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` |

---

## 🚀 Approach 2: Optimized Way (Constant Space)

### 🎯 The Genius Strategy
Instead of using extra memory, we'll use the matrix itself as our notepad!
**Key Insight:** Use the first row and first column as "sticky notes" to remember what needs to be zeroed.

### 🧠 The Clever Trick Visualized

```
Original Matrix - We'll use the margins as our notepad:
┌─────────────────────┐
│  1   2   3   4  │   │ ← This row becomes our "column notepad"
│  5   6   7   0  │   │
│  9   2   0   4  │   │
└─────────────────────┘
   ↑
   This column becomes our "row notepad"
```

### 🎬 Animated Step-by-Step Walkthrough

Starting with: `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]`

#### 🔍 PHASE 1: CHECK ORIGINAL STATE OF MARGINS

```
Step 1: Check if first row originally had zeros
┌─────────────────────┐    Checking: [1, 2, 3, 4]
│ [1] [2] [3] [4] │   │    Result: No zeros found
│  5   6   7   0  │   │    first_row_had_zero = False
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 2: Check if first column originally had zeros  
┌─────────────────────┐    Checking: [1, 5, 9]
│ [1]  2   3   4  │   │    Result: No zeros found
│ [5]  6   7   0  │   │    first_col_had_zero = False
│ [9]  2   0   4  │   │    
└─────────────────────┘    
```

#### 📝 PHASE 2: USE MARGINS AS NOTEPAD

```
Step 3: Scan inner matrix (skip first row/column)
┌─────────────────────┐    Scanning position (1,1)
│  1   2   3   4  │   │    Value = 6 → Not zero, continue
│  5  [6]  7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 4: Continue scanning...
┌─────────────────────┐    Scanning position (1,2)
│  1   2   3   4  │   │    Value = 7 → Not zero, continue
│  5   6  [7]  0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
```

```
Step 5: 🚨 FOUND ZERO - MAKE NOTES! 🚨
┌─────────────────────┐    Scanning position (1,3)
│  1   2   3  [0] │   │    Value = 0 → ZERO FOUND!
│ [0]  6   7   0  │   │    📝 Note in margin: matrix[1][0] = 0
│  9   2   0   4  │   │    📝 Note in margin: matrix[0][3] = 0
└─────────────────────┘    
Current state: [[1,2,3,0], [0,6,7,0], [9,2,0,4]]
```

```
Step 6: Continue scanning...
┌─────────────────────┐    Scanning position (2,1)
│  1   2   3   0  │   │    Value = 2 → Not zero, continue
│  0   6   7   0  │   │    
│  9  [2]  0   4  │   │    
└─────────────────────┘    
```

```
Step 7: 🚨 FOUND ANOTHER ZERO - MAKE MORE NOTES! 🚨
┌─────────────────────┐    Scanning position (2,2)
│  1   2  [0]  0  │   │    Value = 0 → ZERO FOUND!
│  0   6   7   0  │   │    📝 Note in margin: matrix[2][0] = 0
│ [0]  2   0   4  │   │    📝 Note in margin: matrix[0][2] = 0
└─────────────────────┘    
Current state: [[1,2,0,0], [0,6,7,0], [0,2,0,4]]
```

#### 📖 PHASE 3: READ NOTES AND APPLY ZEROS

```
Step 8: Read our notes and apply to inner matrix
┌─────────────────────┐    Reading notes:
│  1   2   0   0  │   │    - First column: 0,0 → Rows 1,2 need zero
│  0  [?] [?]  0  │   │    - First row: 0,0 → Cols 2,3 need zero
│  0  [?] [?]  4  │   │    
└─────────────────────┘    
```

```
Step 9: Apply zeros based on notes
┌─────────────────────┐    Position (1,1): Row 1 marked → Set to 0
│  1   2   0   0  │   │    Position (1,2): Col 2 marked → Set to 0  
│  0  [0] [0]  0  │   │    Position (2,1): Row 2 marked → Set to 0
│  0  [0] [0]  4  │   │    Position (2,2): Col 2 marked → Set to 0
└─────────────────────┘    
Current state: [[1,2,0,0], [0,0,0,0], [0,0,0,4]]
```

```
Step 10: Handle remaining positions
┌─────────────────────┐    Position (2,3): Row 2 marked → Set to 0
│  1   2   0   0  │   │    
│  0   0   0   0  │   │    
│  0   0   0  [0] │   │    
└─────────────────────┘    
Current state: [[1,2,0,0], [0,0,0,0], [0,0,0,0]]
```

#### 🎯 PHASE 4: HANDLE FIRST ROW/COLUMN

```
Step 11: Check if we need to zero first row
┌─────────────────────┐    first_row_had_zero = False
│  1   2   0   0  │   │    → Keep first row as is (except our notes)
│  0   0   0   0  │   │    
│  0   0   0   0  │   │    
└─────────────────────┘    
```

```
Step 12: Check if we need to zero first column  
┌─────────────────────┐    first_col_had_zero = False
│  1   2   0   0  │   │    → Keep first column as is (except our notes)
│  0   0   0   0  │   │    
│  0   0   0   0  │   │    FINAL RESULT ✅
└─────────────────────┘    
```

### 💻 Optimized Code with Detailed Comments
```python
def setZeroes_optimized(matrix):
    if not matrix:
        return matrix
    
    rows, cols = len(matrix), len(matrix[0])
    
    # Phase 1: Check original state of margins
    first_row_had_zero = False
    first_col_had_zero = False
    
    print("🔍 PHASE 1: Checking original state of margins...")
    
    # Check first row
    for j in range(cols):
        if matrix[0][j] == 0:
            first_row_had_zero = True
            print(f"   First row had zero at column {j}")
            break
    
    # Check first column  
    for i in range(rows):
        if matrix[i][0] == 0:
            first_col_had_zero = True
            print(f"   First column had zero at row {i}")
            break
    
    print(f"   first_row_had_zero: {first_row_had_zero}")
    print(f"   first_col_had_zero: {first_col_had_zero}")
    
    # Phase 2: Use margins as notepad
    print(f"\n📝 PHASE 2: Using margins as notepad...")
    for i in range(1, rows):  # Skip first row
        for j in range(1, cols):  # Skip first column
            if matrix[i][j] == 0:
                print(f"   Found zero at ({i},{j})")
                print(f"   📝 Making note: matrix[{i}][0] = 0 (row {i} needs zero)")
                print(f"   📝 Making note: matrix[0][{j}] = 0 (col {j} needs zero)")
                matrix[i][0] = 0  # Note: this row needs zero
                matrix[0][j] = 0  # Note: this column needs zero
    
    # Phase 3: Read notes and apply zeros
    print(f"\n📖 PHASE 3: Reading notes and applying zeros...")
    for i in range(1, rows):
        for j in range(1, cols):
            if matrix[i][0] == 0 or matrix[0][j] == 0:
                print(f"   Position ({i},{j}): ", end="")
                if matrix[i][0] == 0:
                    print(f"row {i} marked ", end="")
                if matrix[0][j] == 0:
                    print(f"col {j} marked ", end="")
                print(f"→ Set to 0")
                matrix[i][j] = 0
    
    # Phase 4: Handle first row and column
    print(f"\n🎯 PHASE 4: Handling first row and column...")
    if first_row_had_zero:
        print("   Zeroing first row (it originally had zeros)")
        for j in range(cols):
            matrix[0][j] = 0
    
    if first_col_had_zero:
        print("   Zeroing first column (it originally had zeros)")
        for i in range(rows):
            matrix[i][0] = 0
    
    return matrix
```

### 🎯 Detailed Dry Run Table (Optimized Approach)

| Phase | Step | Position | Value | Action | Matrix State | Notes |
|-------|------|----------|-------|---------|--------------|-------|
| 1 | 1 | First Row | [1,2,3,4] | Check | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` | first_row_had_zero = False |
| 1 | 2 | First Col | [1,5,9] | Check | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` | first_col_had_zero = False |
| 2 | 3 | (1,1) | 6 | Continue | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` | No action |
| 2 | 4 | (1,2) | 7 | Continue | `[[1,2,3,4], [5,6,7,0], [9,2,0,4]]` | No action |
| 2 | **5** | **(1,3)** | **0** | **Make Notes** | `[[1,2,3,0], [0,6,7,0], [9,2,0,4]]` | **Set matrix[1][0]=0, matrix[0][3]=0** |
| 2 | 6 | (2,1) | 2 | Continue | `[[1,2,3,0], [0,6,7,0], [9,2,0,4]]` | No action |
| 2 | **7** | **(2,2)** | **0** | **Make Notes** | `[[1,2,0,0], [0,6,7,0], [0,2,0,4]]` | **Set matrix[2][0]=0, matrix[0][2]=0** |
| 3 | 8 | (1,1) | 6 | Apply Zero | `[[1,2,0,0], [0,0,7,0], [0,2,0,4]]` | matrix[1][0]=0 → zero this |
| 3 | 9 | (1,2) | 7 | Apply Zero | `[[1,2,0,0], [0,0,0,0], [0,2,0,4]]` | matrix[0][2]=0 → zero this |
| 3 | 10 | (2,1) | 2 | Apply Zero | `[[1,2,0,0], [0,0,0,0], [0,0,0,4]]` | matrix[2][0]=0 → zero this |
| 3 | 11 | (2,2) | 0 | Apply Zero | `[[1,2,0,0], [0,0,0,0], [0,0,0,4]]` | Already zero |
| 3 | 12 | (2,3) | 4 | Apply Zero | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` | matrix[2][0]=0 → zero this |
| 4 | 13 | First Row | - | Check | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` | first_row_had_zero=False, no change |
| 4 | 14 | First Col | - | Check | `[[1,2,0,0], [0,0,0,0], [0,0,0,0]]` | first_col_had_zero=False, no change |

---

## ⚡ Complexity Analysis

### 🕐 Time Complexity
Both approaches: **O(m × n)** where m = rows, n = columns
- We visit each cell at least once
- In worst case, we might visit each cell twice (once to find zeros, once to set zeros)

### 💾 Space Complexity
- **Simple Approach:** O(m + n) - We store row and column indices
- **Optimized Approach:** O(1) - We only use the matrix itself plus a few variables

### 📊 Performance Comparison

| Approach | Time | Space | Readability | Interview Score |
|----------|------|-------|-------------|-----------------|
| Simple | O(m×n) | O(m+n) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| Optimized | O(m×n) | O(1) | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 🧪 Test Your Understanding

Try these step-by-step:

### 🔥 Challenge 1: Easy
```
Input:  [[1,0,3],
         [4,5,6]]

Expected Output: [[0,0,0],
                  [4,0,6]]
```
**Try the simple approach first, then the optimized!**

### 🔥 Challenge 2: Medium  
```
Input:  [[0,1,2],
         [3,4,5], 
         [6,7,8]]

Expected Output: [[0,0,0],
                  [0,4,5],
                  [0,7,8]]
```

### 🔥 Challenge 3: Tricky
```
Input:  [[1,2,3],
         [4,5,6],
         [7,8,9]]

Expected Output: [[1,2,3],
                  [4,5,6], 
                  [7,8,9]]
```
**Why does nothing change?**

---

## 🎯 Key Learning Points

1. **🧠 Problem Pattern:** This is a "marking and applying" problem - mark what needs to change, then apply changes
2. **💡 Space Optimization:** Sometimes we can repurpose existing data structures instead of creating new ones
3. **⚠️ Order Matters:** In the optimized approach, we must handle the margins last to avoid overwriting our notes
4. **🔍 Edge Cases:** Always consider what happens with empty matrices, single rows/columns, or matrices full of zeros
5. **📚 Learning Progression:** Master the simple approach first, then challenge yourself with optimization

## 🏆 Interview Tips

- **Start Simple:** Always explain the brute force approach first
- **Optimize Gradually:** Show the interviewer your thought process for optimization
- **Explain Trade-offs:** Discuss space vs time vs readability trade-offs
- **Handle Edge Cases:** Mention edge cases even if you don't code them
- **Test Your Solution:** Walk through a simple example to verify correctness

Now you're ready to master this problem! Remember: understanding comes before optimization. Good luck! 🚀