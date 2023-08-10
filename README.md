# simd-accelerated-control-flow

I think SIMD can be used to accelerate control flow heavy code.

```
data_vector = 1, 2, 3, 4, 5
case_vector = 0, 0, 0, 0, 0
predicate_1_cmp = SIMD CMP predicate1 data_vector
SIMD ADD predicate_1_cmp 1
SIMD SUB case_vector predicate_1_cmp  // this raises the integer into this if-statements/branches integer space
predicate_2_cmp = SIMD CMP predicate2 data_vector
SIMD ADD predicate_2_cmp 1
SIMD SUB case_vector predicate_2_cmp


// ideally in separate threads
switch (case_vector[idx]) {
    // every permutation of code attached to data item
}
```
