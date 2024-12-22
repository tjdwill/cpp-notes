# Lessons from using Eigen

- Transposing a temporary object leads to weird values (UB?). 
- `transposeInPlace()` only works for square matrices.

## Eigen Matrix <--> `std::vector` conversion

- https://stackoverflow.com/questions/26094379/typecasting-eigenvectorxd-to-stdvector


The following code snippet performs copy operations to instantiate the desired object:

```cpp
// This code works with regular Eigen::Matrix as well. Just ensure the dimensions are
correct.
std::vector<double> stl_vec { 0, 1, 2 };

// convert to Eigen
auto eigen_vec = Eigen::Map<Eigen::Vector<double, 3>>(stl_vec.data());

// back to STL
std::vector<double> to_stl;
to_stl.reserve(eigen_vec.size());
VectorXd::Map(&to_stl[0], eigen_vec.size()) = eigen_vec;
```


