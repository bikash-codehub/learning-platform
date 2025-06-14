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

### 💻 Simple Java Code with Detailed Comments
```java
import java.util.*;

public class Solution {
    public void setZeroesSimple(int[][] matrix) {
        if (matrix == null || matrix.length == 0) {
            return;
        }
        
        int rows = matrix.length;
        int cols = matrix[0].length;
        
        // Phase 1: Scout for zeros
        Set<Integer> zeroRows = new HashSet<>();  // Remember which rows have zeros
        Set<Integer> zeroCols = new HashSet<>();  // Remember which columns have zeros
        
        System.out.println("🔍 PHASE 1: Scouting for zeros...");
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                System.out.println("   Checking position (" + i + "," + j + ") = " + matrix[i][j]);
                if (matrix[i][j] == 0) {
                    System.out.println("   🚨 ZERO FOUND at (" + i + "," + j + ")!");
                    zeroRows.add(i);
                    zeroCols.add(j);
                    System.out.println("   📝 Added row " + i + " and column " + j + " to cleanup list");
                }
            }
        }
        
        System.out.println("\n📝 CLEANUP LIST:");
        System.out.println("   Rows to zero: " + new ArrayList<>(zeroRows));
        System.out.println("   Columns to zero: " + new ArrayList<>(zeroCols));
        
        // Phase 2: Zero out rows
        System.out.println("\n🧹 PHASE 2: Zeroing rows...");
        for (int row : zeroRows) {
            System.out.print("   Zeroing row " + row + ": " + Arrays.toString(matrix[row]) + " → ");
            for (int j = 0; j < cols; j++) {
                matrix[row][j] = 0;
            }
            System.out.println(Arrays.toString(matrix[row]) + " ✅");
        }
        
        // Phase 3: Zero out columns  
        System.out.println("\n🧹 PHASE 3: Zeroing columns...");
        for (int col : zeroCols) {
            System.out.println("   Zeroing column " + col + "...");
            for (int i = 0; i < rows; i++) {
                matrix[i][col] = 0;
            }
            System.out.println("   ✅ Column " + col + " zeroed!");
        }
    }
}
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

### 💻 Optimized Java Code with Detailed Comments
```java
import java.util.Arrays;

public class Solution {
    public void setZeroesOptimized(int[][] matrix) {
        if (matrix == null || matrix.length == 0) {
            return;
        }
        
        int rows = matrix.length;
        int cols = matrix[0].length;
        
        // Phase 1: Check original state of margins
        boolean firstRowHadZero = false;
        boolean firstColHadZero = false;
        
        System.out.println("🔍 PHASE 1: Checking original state of margins...");
        
        // Check first row
        for (int j = 0; j < cols; j++) {
            if (matrix[0][j] == 0) {
                firstRowHadZero = true;
                System.out.println("   First row had zero at column " + j);
                break;
            }
        }
        
        // Check first column  
        for (int i = 0; i < rows; i++) {
            if (matrix[i][0] == 0) {
                firstColHadZero = true;
                System.out.println("   First column had zero at row " + i);
                break;
            }
        }
        
        System.out.println("   firstRowHadZero: " + firstRowHadZero);
        System.out.println("   firstColHadZero: " + firstColHadZero);
        
        // Phase 2: Use margins as notepad
        System.out.println("\n📝 PHASE 2: Using margins as notepad...");
        for (int i = 1; i < rows; i++) {  // Skip first row
            for (int j = 1; j < cols; j++) {  // Skip first column
                if (matrix[i][j] == 0) {
                    System.out.println("   Found zero at (" + i + "," + j + ")");
                    System.out.println("   📝 Making note: matrix[" + i + "][0] = 0 (row " + i + " needs zero)");
                    System.out.println("   📝 Making note: matrix[0][" + j + "] = 0 (col " + j + " needs zero)");
                    matrix[i][0] = 0;  // Note: this row needs zero
                    matrix[0][j] = 0;  // Note: this column needs zero
                }
            }
        }
        
        // Phase 3: Read notes and apply zeros
        System.out.println("\n📖 PHASE 3: Reading notes and applying zeros...");
        for (int i = 1; i < rows; i++) {
            for (int j = 1; j < cols; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    System.out.print("   Position (" + i + "," + j + "): ");
                    if (matrix[i][0] == 0) {
                        System.out.print("row " + i + " marked ");
                    }
                    if (matrix[0][j] == 0) {
                        System.out.print("col " + j + " marked ");
                    }
                    System.out.println("→ Set to 0");
                    matrix[i][j] = 0;
                }
            }
        }
        
        // Phase 4: Handle first row and column
        System.out.println("\n🎯 PHASE 4: Handling first row and column...");
        if (firstRowHadZero) {
            System.out.println("   Zeroing first row (it originally had zeros)");
            for (int j = 0; j < cols; j++) {
                matrix[0][j] = 0;
            }
        }
        
        if (firstColHadZero) {
            System.out.println("   Zeroing first column (it originally had zeros)");
            for (int i = 0; i < rows; i++) {
                matrix[i][0] = 0;
            }
        }
    }
}
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

## 🚀 Approach 3: Row-Column Sequential Marking

### 🎯 The Simple Strategy
This approach uses a straightforward 3-step process:
1. **🔍 ROW-WISE:** Traverse each row, when you find a 0, mark other elements in that row as -1 (keep 0s as 0s)
2. **🔍 COLUMN-WISE:** Traverse each column, when you find a 0, mark other elements in that column as -1 (keep 0s as 0s)  
3. **🎯 CONVERT:** Change all -1s back to 0s

**Key Insight:** We use -1 as a temporary marker to distinguish between original zeros and elements that need to become zero.

### 🎬 Animated Step-by-Step Walkthrough

Starting with Java 2D array: `{{1,2,3,4}, {5,6,7,0}, {9,2,0,4}}`

#### 🔍 PHASE 1: ROW-WISE TRAVERSAL

```
Step 1: Process Row 0 - [1,2,3,4]
┌─────────────────────┐    No zeros found in row 0
│  1   2   3   4  │   │    → No changes needed
│  5   6   7   0  │   │    
│  9   2   0   4  │   │    
└─────────────────────┘    
Current: {{1,2,3,4}, {5,6,7,0}, {9,2,0,4}}
```

```
Step 2: Process Row 1 - [5,6,7,0]
┌─────────────────────┐    Zero found at position (1,3)!
│  1   2   3   4  │   │    → Mark other elements in row 1 as -1
│ [-1][-1][-1] 0  │   │    → Keep the 0 as 0
│  9   2   0   4  │   │    
└─────────────────────┘    
Current: {{1,2,3,4}, {-1,-1,-1,0}, {9,2,0,4}}
```

```
Step 3: Process Row 2 - [9,2,0,4]
┌─────────────────────┐    Zero found at position (2,2)!
│  1   2   3   4  │   │    → Mark other elements in row 2 as -1
│ -1  -1  -1   0  │   │    → Keep the 0 as 0
│[-1][-1] 0 [-1] │   │    
└─────────────────────┘    
Current: {{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}
```

#### 🔍 PHASE 2: COLUMN-WISE TRAVERSAL

```
Step 4: Process Column 0 - [1,-1,-1]
┌─────────────────────┐    No zeros found in column 0
│  1   2   3   4  │   │    → No changes needed
│ -1  -1  -1   0  │   │    
│ -1  -1   0  -1  │   │    
└─────────────────────┘    
Current: {{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}
```

```
Step 5: Process Column 1 - [2,-1,-1]
┌─────────────────────┐    No zeros found in column 1
│  1   2   3   4  │   │    → No changes needed
│ -1  -1  -1   0  │   │    
│ -1  -1   0  -1  │   │    
└─────────────────────┘    
Current: {{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}
```

```
Step 6: Process Column 2 - [3,-1,0]
┌─────────────────────┐    Zero found at position (2,2)!
│  1   2 [-1]  4  │   │    → Mark other elements in column 2 as -1
│ -1  -1  -1   0  │   │    → Already -1, no change
│ -1  -1   0  -1  │   │    → Keep the 0 as 0
└─────────────────────┘    
Current: {{1,2,-1,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}
```

```
Step 7: Process Column 3 - [4,0,-1]
┌─────────────────────┐    Zero found at position (1,3)!
│  1   2  -1 [-1] │   │    → Mark other elements in column 3 as -1
│ -1  -1  -1   0  │   │    → Keep the 0 as 0
│ -1  -1   0  -1  │   │    → Already -1, no change
└─────────────────────┘    
Current: {{1,2,-1,-1}, {-1,-1,-1,0}, {-1,-1,0,-1}}
```

#### 🎯 PHASE 3: CONVERT ALL -1s TO 0s

```
Step 8: Final conversion
┌─────────────────────┐    Convert all -1s to 0s:
│  1   2   0   0  │   │    Final result: {{1,2,0,0},
│  0   0   0   0  │   │                   {0,0,0,0},
│  0   0   0   0  │   │                   {0,0,0,0}}
└─────────────────────┘    
```

### 💻 Java Implementation

```java
import java.util.Arrays;

public class Solution {
    public void setZeroesRowColumn(int[][] matrix) {
        if (matrix == null || matrix.length == 0) {
            return;
        }
        
        int rows = matrix.length;
        int cols = matrix[0].length;
        
        // Phase 1: Row-wise traversal
        System.out.println("🔍 PHASE 1: Row-wise traversal...");
        for (int i = 0; i < rows; i++) {
            System.out.println("   Processing row " + i + ": " + Arrays.toString(matrix[i]));
            
            // Check if this row has any zeros
            boolean hasZero = false;
            for (int j = 0; j < cols; j++) {
                if (matrix[i][j] == 0) {
                    hasZero = true;
                    break;
                }
            }
            
            // If row has zero, mark other elements as -1
            if (hasZero) {
                System.out.println("     → Found zero in row " + i + ", marking others as -1");
                for (int j = 0; j < cols; j++) {
                    if (matrix[i][j] != 0) {  // Don't change original zeros
                        matrix[i][j] = -1;
                    }
                }
                System.out.println("     → Row " + i + " after marking: " + Arrays.toString(matrix[i]));
            } else {
                System.out.println("     → No zeros found in row " + i);
            }
        }
        
        // Phase 2: Column-wise traversal
        System.out.println("\n🔍 PHASE 2: Column-wise traversal...");
        for (int j = 0; j < cols; j++) {
            System.out.print("   Processing column " + j + ": [");
            for (int i = 0; i < rows; i++) {
                System.out.print(matrix[i][j] + (i < rows-1 ? "," : ""));
            }
            System.out.println("]");
            
            // Check if this column has any zeros
            boolean hasZero = false;
            for (int i = 0; i < rows; i++) {
                if (matrix[i][j] == 0) {
                    hasZero = true;
                    break;
                }
            }
            
            // If column has zero, mark other elements as -1
            if (hasZero) {
                System.out.println("     → Found zero in column " + j + ", marking others as -1");
                for (int i = 0; i < rows; i++) {
                    if (matrix[i][j] != 0) {  // Don't change original zeros
                        matrix[i][j] = -1;
                    }
                }
            } else {
                System.out.println("     → No zeros found in column " + j);
            }
        }
        
        // Phase 3: Convert all -1s to 0s
        System.out.println("\n🎯 PHASE 3: Converting all -1s to 0s...");
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (matrix[i][j] == -1) {
                    matrix[i][j] = 0;
                    System.out.println("   Converted position (" + i + "," + j + ") from -1 to 0");
                }
            }
        }
        
        System.out.println("\n✅ Final result:");
        for (int i = 0; i < rows; i++) {
            System.out.println("   " + Arrays.toString(matrix[i]));
        }
    }
}
```

### 🎯 Detailed Dry Run Table

| Phase | Step | Action | Matrix State | Notes |
|-------|------|--------|--------------|-------|
| 1 | 1 | Process Row 0 | `{{1,2,3,4}, {5,6,7,0}, {9,2,0,4}}` | No zeros found |
| 1 | 2 | Process Row 1 | `{{1,2,3,4}, {-1,-1,-1,0}, {9,2,0,4}}` | Zero at (1,3), marked others as -1 |
| 1 | 3 | Process Row 2 | `{{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}` | Zero at (2,2), marked others as -1 |
| 2 | 4 | Process Col 0 | `{{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}` | No zeros found |
| 2 | 5 | Process Col 1 | `{{1,2,3,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}` | No zeros found |
| 2 | 6 | Process Col 2 | `{{1,2,-1,4}, {-1,-1,-1,0}, {-1,-1,0,-1}}` | Zero at (2,2), marked (0,2) as -1 |
| 2 | 7 | Process Col 3 | `{{1,2,-1,-1}, {-1,-1,-1,0}, {-1,-1,0,-1}}` | Zero at (1,3), marked (0,3) as -1 |
| 3 | 8 | Convert -1s | `{{1,2,0,0}, {0,0,0,0}, {0,0,0,0}}` | All -1s converted to 0s |

### ⚡ **Why This Approach Works:**
1. **Simple Logic:** Easy to understand - process rows first, then columns
2. **Clear Marking:** Uses -1 to distinguish between original zeros and new zeros
3. **No Conflicts:** Processing rows first, then columns avoids marking conflicts
4. **Preserves Originals:** Never overwrites original zeros during marking phase

### ⚠️ **Limitations:**
- **Assumes Non-Negative Values:** Matrix cannot contain -1 values originally
- **Time Complexity:** O(m×n) but with higher constant factor due to multiple passes
- **Space Complexity:** O(1) but modifies the matrix during processing

---

## ⚡ Complexity Analysis

### 🕐 Time Complexity
All three approaches: **O(m × n)** where m = rows, n = columns
- **Simple & Optimized:** We visit each cell at least once, at most twice
- **Row-Column Marking:** Multiple passes (rows + columns + conversion) but still O(m×n)

### 💾 Space Complexity
- **Simple Approach:** O(m + n) - We store row and column indices
- **Optimized Approach:** O(1) - We only use the matrix itself plus a few variables
- **Row-Column Marking:** O(1) - Uses -1 as temporary markers within the matrix

### 📊 Performance Comparison

| Approach | Time | Space | Readability | Interview Score | Notes |
|----------|------|-------|-------------|-----------------|-------|
| Simple (Extra Space) | O(m×n) | O(m+n) | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | Easy to understand |
| Optimized (Margins) | O(m×n) | O(1) | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Most elegant solution |
| Row-Column Marking | O(m×n) | O(1) | ⭐⭐⭐⭐ | ⭐⭐⭐ | Simple 3-step process |

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