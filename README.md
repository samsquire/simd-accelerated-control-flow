# simd-accelerated-control-flow

This is an idea I had while learning about SIMD.

I think SIMD can be used to accelerate high indirection, **control flow** heavy code as used by dynamic languages, specifically **comparison statements**.

We number each combination of control flow for any number of if statements resulting control flow and then map it to a single number which we use a jump table to dispatch all the behaviour based on that data item.

Imagine you have 1,000,000 records and you have 5 if statements to do over them. The traditional approach would be to do 1,000,000 × 5 = 5,000,000 if statements.

The alternative is that the 1,000,000 records are packed into a SIMDable 256 bit or 512 bit vector and we run 32 if statements at a time. So this requires 3 SIMD instructions per if statement (CMP, ADD, SUB). This gives us 1,000,000 × 3 = 3,000,000 SIMD instructions That processes 32 records at time.

We do it by doing code generation for every permutation of code that a group of if statements maps to. This is **control flow polymorphisation**.

If you have all your data in a collection and you want to process it, we can process 256/8 or 32 or 512/8 64 records at the same time. The instruction is a **CMP**.

The idea is that we can compress all if statements into one switch table, which is amortized across multiple records at a time.

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

# how to compile

 * Scan all code for if statements and generate a list of all potential permutations of if statement branch taking.
 * Number each permutation.
 * Create a vector of the data to be processed.
