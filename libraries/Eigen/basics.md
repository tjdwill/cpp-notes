# Eigen Basics


## [Reductions](https://eigen.tuxfamily.org/dox/group__TutorialReductionsVisitorsBroadcasting.html)

Reduction operations are operations that take multi-value data such as matrices or arrays and reduce them to a
single value. This is done, for example, via the `.sum()` method.

### Partial Reductions

Reductions that operate column- or row-wise. These are done with `.colwise()` or `.rowwise()` followed by the
reduction call.
