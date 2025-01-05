# Eigen Basics

A lot of this document will be a rehash of [the
documentation](https://eigen.tuxfamily.org/dox/group__DenseMatrixManipulation__chapter.html), but in a structure designed with myself in mind. 

## Matrix 

The cornerstone of the Eigen library is the [`Matrix`](https://eigen.tuxfamily.org/dox/classEigen_1_1Matrix.html) class. It is a templatized class that takes six
parameters, three of which are defaulted:

1. `typename Scalar`: The type of a given matrix coefficient (`int`, `double`, `bool`, etc.)
2. `int RowsAtCompileTime`: The number of rows in the matrix at compile time.
3. `int ColsAtCompileTime`: number of columns in the matrix at compile time.
4. `int Options`: Various options via a bit-field. The most important one so far is [storage order](https://eigen.tuxfamily.org/dox/group__TopicStorageOrders.html)
5. `int MaxRowsAtCompileTime`: Specify upper-bound for number of rows (can help prevent dynamic allocations)
6. `int MaxColsAtCompileTime`: Specify upper-bound for number of columns (help prevent dynamic allocations)
 
Typically, we are only concerned with the first three parameters:

```cpp
Matrix<int, 2, 3> m; // A matrix of integers with two rows and three columns
```

Specifying a numeric value for the matrix dimensions results in a *fixed-size* matrix. However, we
may not know the size until runtime. We can specify one or dimension of the matrix as
`Eigen::Dynamic`, creating a *dynamic-size* matrix. Unlike fixed-size matrices, dynamic-size
matrices are allocated on the heap and are resizable. The documentation recommends using dynamic
sizes for matrices with more than ~32 elements (or when you simply don't know the size of course).

`Vector`s are special cases of `Matrix` with a col size of 1 (`Eigen::Vector`) or a row size of 1 (`Eigen::RowVector`).

### Useful typedefs

Source: https://eigen.tuxfamily.org/dox/group__TutorialMatrixClass.html

> We rarely specify Eigen objects using the pull bracketed syntax. Instead, we use the convenience
> typedefs provided by the library:
> 
>    - `MatrixNt` for `Matrix<type, N, N>`. For example, MatrixXi for Matrix<int, Dynamic, Dynamic>.
>    - `MatrixXNt` for `Matrix<type, Dynamic, N>`. For example, MatrixX3i for Matrix<int, Dynamic, 3>.
>    - `MatrixNXt` for `Matrix<type, N, Dynamic>`. For example, Matrix4Xd for Matrix<d, 4, Dynamic>.
>    - `VectorNt` for `Matrix<type, N, 1>`. For example, Vector2f for Matrix<float, 2, 1>.
>    - `RowVectorNt` for `Matrix<type, 1, N>`. For example, RowVector3d for Matrix<double, 1, 3>.
> 
> Where:
> 
> - `N` can be any one of 2, 3, 4, or X (meaning Dynamic).
> - `t` can be any one of `i` (meaning int), `f` (meaning float), `d` (meaning double), `cf` (meaning
>   complex<float>), or `cd` (meaning complex<double>). The fact that typedefs are only defined for
>   these five types doesn't mean that they are the only supported scalar types. For example, all
>   standard integer types are supported, see Scalar types.

### Basic Constructors

We can construct matrices in many ways.

1. Default constructor
    - Does not perform dynamic memory allocation
    - Never initializes the matrix coefficients
    - For dynamic-size matrices, the default size is 0-by-z.
2. Size-specifying constructor
    - Allocates coefficients with the desired size `(row, cols)` (or just one or the other for
      vectors)
    - Does not initialize coeficients
3. Initializing w/ a list of coefficients
    - Think: Numpy.
    - Use a list of initializer lists (where each list represents a row)

```cpp
// Default constructor
MatrixXd m;  // 0x0 dynamic matrix of doubles
Matrix3i a;  // 3x3 fixed matrix of integers

MatrixXf b(2, 5);  // 2x5 dynamic matrix of floats.

MatrixXi c {
    {0, 1},
    {2, 3},
};
```

### Accessing Coefficients

Use the `operator()`. Do NOT use the `operator[]`; it doesn't work for multi-dimensional matrices
(so, non-vectors). To access the coefficient located at row `i`, column `j`, use `m(i, j)`):

```cpp
Matrix2i m;
m(0, 0) = 0;
m(0, 1) = 1;
m(1, 0) = 2;
m(1, 1) = 3;

std::cout << "Matrix m:\n" << m << "\n";

/* Outputs

Matrix m:
0 1 
2 3

*/
```

### Resizing

Dynamic matrices can be resized. Fixed cannot. Resize a matrix `m` with [the `resize()`
method](https://eigen.tuxfamily.org/dox/classEigen_1_1PlainObjectBase.html#a9fd0703bd7bfe89d6dc80e2ce87c312a).
If the size changes at all, all of the matrix coefficients are reset to an uninitialized state. No
operation occurs if the matrix is resized to its exact current dimensionality.

Polling functions:

- `rows()`: The number rows in `m`
- `cols()`: The number of columns in `m`
- `size()`: The number of elements in `m`

Assigning a matrix on the right-hand side to a matrix on the left-hand size will copy the right's
data to the left, resizing if necessary. However, if the left matrix is fixed-size, attempted resizing
is an error.

### [Storage Order](https://eigen.tuxfamily.org/dox/group__TopicStorageOrders.html)

Eigen allows the user to specify how a given matrix is stored in memory. Though it represents a
multi-dimensional object, `Eigen::Matrix` objects are stored linearly in memory, so the storage
order can affect locality which in turn affects access time for certain operations. By default,
objects are stored in `ColMajor` order, meaning for an `ixj` matrix `m`, the first `i` elements of
some `*(m.data())` operation represents the first column.

Specifying `RowMajor` will store the matrix coefficients by row.

Note, however, that neither specification affects how coefficients are accessed; `m(i,j)` still
returns that element regardless of storage order. Storage order is an implementation detail (a very
important one though).

```cpp
using namespace Eigen;
Matrix<double, 2, 2, RowMajor> m_row_major {
    {0, 1},
    {2, 3},
};
Matrix<double, 2, 2> m_col_major {
    {0, 1},
    {2, 3};
}

std::cout << "In memory (row-major):" << "\n";
for (int i {0}; i < m_row_major.size(); ++i)
    std::cout << *(m_row_major.data() + i) << " ";
std::cout << "\n\n";

std::cout << "In memory (column-major):" << "\n";
for (int i {0}; i < m_col_major.size(); ++i)
    std::cout << *(m_col_major.data() + i) << " ";
std::cout << "\n\n";


/* Output

IN memory (row--major):
0 1 2 3

In memory (column-major):
0 2 1 3

*/
```

### [Basic Operations](https://eigen.tuxfamily.org/dox/group__TutorialMatrixArithmetic.html)

The `Matrix` class is intended for linear algebra operations, so linear algebra rules apply. Two
matrices may be added or subtracted if they have the same dimensions. A matrix can be
multiplicatively scaled by a constant scalar. Two matrices can be multiplied according to the rules
of matrix multiplication. Division is not a valid operation between matrices (though valid between a
matrix and a scalar).

A matrix can be transposed via the
[`transpose()`](https://eigen.tuxfamily.org/dox/classEigen_1_1DenseBase.html#a43cbcd866a0737eb56642c2e992f0afd)
method. Similarly there are the `conjugate()` and `adjoint()` methods as well. It should be noted
that, [due to aliasing](https://eigen.tuxfamily.org/dox/group__TopicAliasing.html), we cannot
replace a matrix `m` by it's transposition via direct assignment; we must construct a temporary.
*Aliasing* in Eigen is an assignment operation in which the same matrix object appears on both sides
of the assignment operator. See the linked page for getting around aliasing.

**TODO**: Add additional notes about aliasing if I ever run across it in my work.  

In addition to the aforementioned operations, we have the typical dot and cross products via the
`dot()` and `cross()` methods. Finally, we have reduction operations that reduce a matrix to a
single value. This, for example, is done by the `sum()` method which sums all coefficients in the
matrix. Partial reductions, reductions that perform row- or column-wise operations, are done with
`.colwise()` or `.rowwise()`  methods followed by a call to the desired reduction function.

[More about reductions can be found here.](https://eigen.tuxfamily.org/dox/group__TutorialReductionsVisitorsBroadcasting.html)

## Arrays

The other staple class of the Eigen library is
[`Eigen::Array`](https://eigen.tuxfamily.org/dox/classEigen_1_1Array.html). Whereas the `Matrix`
class is intended for linear algebra, `Array` is for general coefficient-based operations.
Syntactically, they are defined with the same parameter interface as `Matrix`; the only difference
is the name. Eigen provides convenient typedefs for this class as well with a similar format as that
defined above for matrices. However, the form `ArrayNt` is exclusively for one-dimensional arrays.
For 2-d arrays, we use `ArrayNNt`.

### Operations

`Array` operations differ from `Matrix` operations because the former are coefficient-wise
operations. For example, multiplying two arrays simply multiplies their respective coefficients
together (meaning the two arrays must have the same dimensions).

View the [Quick reference guide](https://eigen.tuxfamily.org/dox/group__QuickRefPage.html) for more
functions.

## Matrix ↔ Array Conversion

First, let's understand expression templates. Eigen operators don't perform the operations
themselves; they create objects that represent the operation to be performed. So, we can 
"convert" between the two types by calling the respective methods that return the desired type
expression. 

- `Array.matrix()` converts an `Array` into a `Matrix` expression
- `Matrix.array()` converts a `Matrix` into an `Array` expression.

This allows us to mix operation types (ex. I've used this to perform coefficient-wise boolean
comparisons to find specific coefficients with desired characteristics).
