# simd-accelerated-control-flow

This is an idea I had while learning about SIMD.

I think SIMD can be used to accelerate high indirection, **control flow** heavy code as used by dynamic languages, specifically **comparison operations**.

I had the idea we can spider our if statements in our code, work out every permutation of them and run multiple SIMD CMP sequentially (representing parallel if statements for multiple records) and **subtract those vectors** because that is commutative and then map that to an integer of permutation of straight line control flow and use a jump table to dispatch to that permutation of control flow. Essentially, we run 32 if statements for 32 records at a time. I call it control flow monomorphisation, because you have to generate code for each permutation of all your if statements. You can do the dispatch in a separate threads for even more throughput.

We number each combination of control flow for any number of if statements resulting control flow and then map it to a single number which we use a jump table to dispatch all the behaviour based on that data item.

Imagine you have 1,000,000 records and you have 5 if statements to do over them. The traditional approach would be to do 1,000,000 × 5 = 5,000,000 if statements. This processes 1 record at a time.

The alternative is that the 1,000,000 records are packed into a SIMDable 256 bit or 512 bit vector and we run 32 if statements at a time assuming each field is 8 bytes for 256 bit SIMD. So this requires 2 SIMD instructions per if statement (CMP, SUB). This gives us 62,000 SIMD instructions that processes 32 records at a time.

We do it by doing code generation for every permutation of code that a group of if statements maps to. This is **control flow polymorphisation**.

If you have all your data in a collection and you want to process it, we can process 256/8 or 32 or 512/8 64 records at the same time. The instruction is a **CMP**.

The idea is that we can compress all if statements into one switch table, which is amortized across multiple records at a time.

```
comparison_vector_1 = 0, 0, 0, 0, 0
comparison_vector_2 = 0, 0, 0, 0, 0
data_vector = 1, 2, 3, 4, 5
case_vector = 0, 0, 0, 0, 0 // this is used for dispatch to control flow
predicate_1_cmp = SIMD CMP operator comparison_vector_1 data_vector
SIMD SUB case_vector predicate_1_cmp

SIMD SUB case_vector predicate_1_cmp  // this raises (lowers) the integer into this if-statements/branches integer space
predicate_2_cmp = SIMD CMP operator comparison_vector_2 data_vector
SIMD SUB case_vector predicate_2_cmp

// then ideally in separate threads
switch (case_vector[idx]) {
    // dispatch to the exact permutation of the control flow graph without needing if-statements or branching
}
```

# how to compile

 * Scan all code for if statements and generate a list of all potential permutations of if statement branch taking.
 * Number each permutation.
 * Create a vector of the data to be processed.
