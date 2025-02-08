# Jagged Diagonal Storage


### **Practical Storage Calculation for Jagged Diagonal Storage (JDS)**  

Jagged Diagonal Storage (JDS) reduces storage overhead compared to traditional sparse matrix formats like Compressed Sparse Row (CSR) or Coordinate (COO). Let's break down the storage requirements for each component.  

---

### **1. Storage Components in JDS**
For a sparse matrix of size \( m \times n \) with \( nnz \) (non-zero elements):

- **VAL (Values Array)**  
  - Stores all non-zero elements.  
  - Requires **\( nnz \times \) size of one element** (e.g., 4 bytes for `float` or 8 bytes for `double`).  

- **COL_IDX (Column Indices Array)**  
  - Stores column indices of non-zero elements.  
  - Requires **\( nnz \times \) size of integer** (typically 4 bytes for `int`).  

- **ROW_PTR (Row Pointers Array)**  
  - Points to the start of each row in `VAL`.  
  - Requires **\( m \times \) size of integer** (4 bytes per row).  

- **PERM (Row Permutation Array)**  
  - Stores the original row indices before sorting.  
  - Requires **\( m \times \) size of integer** (4 bytes per row).  

---

### **2. Practical Storage Calculation Example**
Consider a **4 × 5 sparse matrix** with **9 non-zero elements**:  

| Format  | Formula for Storage (Bytes) | Example (4x5 matrix, 9 `int` values) |
|---------|-----------------------------|--------------------------------------|
| **VAL**     | \( nnz \times \) size of element (4B for `int`) | \( 9 \times 4 = 36B \) |
| **COL_IDX** | \( nnz \times \) size of integer (4B) | \( 9 \times 4 = 36B \) |
| **ROW_PTR** | \( m \times \) size of integer (4B) | \( 4 \times 4 = 16B \) |
| **PERM**    | \( m \times \) size of integer (4B) | \( 4 \times 4 = 16B \) |
| **Total Storage** | Sum of all | **104 Bytes** |

---

### **3. Comparison with Other Sparse Formats**
| Format | Storage Formula | Example Storage (Bytes) |
|--------|----------------|-------------------------|
| **COO (Coordinate List)** | \( nnz \times ( \) size of value \( + \) size of row index \( + \) size of column index \( ) \) | \( 9 \times (4+4+4) = 108B \) |
| **CSR (Compressed Sparse Row)** | \( nnz \times ( \) size of value \( + \) size of column index \( ) + m \times \) size of integer | \( 9 \times (4+4) + 4 \times 4 = 100B \) |
| **JDS (Jagged Diagonal Storage)** | \( nnz \times ( \) size of value \( + \) size of column index \( ) + 2m \times \) size of integer | **104B** |

👉 **JDS offers efficient storage while optimizing sparse matrix-vector multiplications (SpMV), particularly in GPU applications.**  


# Packed Diagonal Storage

Packed Diagonal Storage (PDS) is a compact way to store banded matrices efficiently by only storing the nonzero diagonals. This is useful in scientific computing and numerical linear algebra, where large sparse matrices appear frequently.  

### **Structure of Packed Diagonal Storage (PDS)**
For an \( n \times n \) band matrix with a bandwidth of \( w \), only the diagonals within the bandwidth are stored. These diagonals are packed into a 2D array, reducing memory usage compared to full matrix storage.

If we define:
- **\( l \)**: Number of subdiagonals (below the main diagonal)
- **\( u \)**: Number of superdiagonals (above the main diagonal)
- The **bandwidth** is \( l + u + 1 \), which is the number of diagonals stored.

### **Storage Format**
A band matrix is stored in a 2D array of size \( (l + u + 1) \times n \), where:
- Each **row** represents a diagonal.
- The **main diagonal** is in the middle row.
- Superdiagonals are stored in upper rows.
- Subdiagonals are stored in lower rows.

For example, consider a \( 5 \times 5 \) matrix with bandwidth 3 (one subdiagonal, main diagonal, and one superdiagonal):

\[
\begin{bmatrix}
a_{1,1} & a_{1,2} & 0      & 0      & 0 \\
a_{2,1} & a_{2,2} & a_{2,3} & 0      & 0 \\
0      & a_{3,2} & a_{3,3} & a_{3,4} & 0 \\
0      & 0      & a_{4,3} & a_{4,4} & a_{4,5} \\
0      & 0      & 0      & a_{5,4} & a_{5,5}
\end{bmatrix}
\]

The **Packed Diagonal Storage (PDS) format** will be:

\[
\begin{bmatrix}
*    & a_{1,2} & a_{2,3} & a_{3,4} & a_{4,5} \\
a_{1,1} & a_{2,2} & a_{3,3} & a_{4,4} & a_{5,5} \\
a_{2,1} & a_{3,2} & a_{4,3} & a_{5,4} & *   
\end{bmatrix}
\]

Here, `*` represents unused space.

### **Advantages of PDS**
- **Memory Efficient**: Avoids storing unnecessary zero elements.
- **Better Performance**: Useful for matrix operations in iterative solvers (e.g., LU decomposition, Gauss-Seidel method).
- **Cache-Friendly**: Improves cache locality in numerical computations.


# Block Compressed Row Storage (BCRS)

## Introduction
Block Compressed Row Storage (BCRS) is a sparse matrix storage format that extends Compressed Row Storage (CSR) by grouping matrix elements into small dense blocks. This improves cache efficiency and performance, particularly for structured matrices found in scientific computing.

---

## Key Features
- **Efficient storage of block-structured sparse matrices**
- **Reduces indexing overhead** compared to CSR
- **Improves cache locality** by storing data in contiguous blocks
- **Optimized for structured scientific computing problems**

---

## Example Matrix (6×6 Sparse Matrix)
```
| 1  0  2  0  0  0 |
| 0  3  0  4  0  0 |
| 5  0  6  0  0  7 |
| 0  8  0  9 10  0 |
| 0  0 11  0 12  0 |
| 0  0  0 13  0 14 |
```
- **Nonzero elements**: 12
- **Block size**: 2×2

---

## Storage in CSR Format
CSR stores:
1. **Values (`val`)**: Stores all 12 nonzero elements.
2. **Column Indices (`col_ind`)**: One index per nonzero.
3. **Row Pointers (`row_ptr`)**: One pointer per row.

### **CSR Storage Size** (in terms of elements)
- **Values**: `12`
- **Column Indices**: `12`
- **Row Pointers**: `7`
- **Total Storage**: `12 + 12 + 7 = 31` elements

---

## Storage in BCRS Format (2×2 Blocks)
### **Block Partitioning**
```
| (1,0,0,3) | (2,0,0,4) | (0,0,0,0) |
| (5,0,0,8) | (6,0,0,9) | (0,0,7,10) |
| (0,0,11,0) | (0,0,12,0) | (0,13,0,14) |
```
Each block is stored as a **dense** 2×2 matrix.

### **BCRS Storage Size** (in terms of elements)
- **Values (`val`)**: `7 × 4 = 28`
- **Column Indices (`col_ind`)**: `7`
- **Row Pointers (`row_ptr`)**: `4`
- **Total Storage**: `28 + 7 + 4 = 39` elements

---

## Comparison: CSR vs BCRS
| Format | Values | Column Indices | Row Pointers | Total Storage |
|--------|--------|---------------|--------------|--------------|
| **CSR** | 12 | 12 | 7 | **31 elements** |
| **BCRS (2×2)** | 28 | 7 | 4 | **39 elements** |

### **Key Observations**
- **BCRS requires more storage** due to block padding.
- **BCRS improves performance** for structured matrices by reducing index overhead.
- **If the matrix lacks a natural block structure, CSR is better.**

---

## When to Use BCRS?
✅ Use BCRS if:
- The matrix has a **natural block structure** (e.g., FEM, CFD problems).
- Optimizing **matrix-vector multiplication (SpMV)** performance.
- **Reducing indexing overhead** is important.

❌ Avoid BCRS if:
- The matrix has **arbitrary sparsity patterns**.
- Memory efficiency is a higher priority than computation speed.

---


