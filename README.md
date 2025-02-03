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

Would you like an implementation example in C or Python? 🚀
