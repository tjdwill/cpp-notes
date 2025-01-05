# Lessons from using Eigen

- Transposing a temporary object leads to weird values (UB?). 
- `transposeInPlace()` only works for square matrices.
- If you want operations done on each coefficient, convert the matrix to an `Eigen::Array`.
    - Perform the operation and then convert back.

## Eigen Matrix ↔ `std::vector` conversion

- https://stackoverflow.com/questions/26094379/typecasting-eigenvectorxd-to-stdvector
- https://stackoverflow.com/questions/39951553/is-it-possible-to-flatten-an-eigen-matrix-without-copying


The following code snippet performs copy operations to instantiate the desired object:

```cpp
std::vector<double> stl_vec { 0, 1, 2 };

// convert to Eigen
auto eigen_vec = Eigen::Map<Eigen::Vector<double, 3>>(stl_vec.data());

// back to STL
std::vector<double> to_stl {};
to_stl.resize(eigen_vec.size());
VectorXd::Map(&to_stl[0], eigen_vec.size()) = eigen_vec;

// From a MatrixXd
/// May have to check col/row ordering.
auto to_std_vec(Eigen::MatrixXd const& input) -> std::vector<double>
{
    std::vector<double> stl {};
    stl.resize(input.size());
    Eigen::VectorXd vec = Eigen::Map<const Eigen::VectorXd>(input.data(), input.size());
    Eigen::VectorXd::Map(&stl[0], vec.size()) = vec;

    return stl;
}
```

**Important Note**: It is important that you be able to pass in mutable references to the vector
object. For some reason, Eigen code would not compile when I used a const reference to extract the
vector data. There is likely some aspect of Eigen that I am missing.

## Finding Coefficients with Specified Characteristics (Boolean Filtering)