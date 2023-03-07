# Motivation / Problem to Solve
In modern database system, CPU cost of volcano model is high
1. For pulling a single record, next() is a expensive virtual function call
2. Bad code and data locality
3. Not able to leverage compiler optimization(loop unrolling and SIMD) and cpu branch prediction

# Strengths
1. Maximize data and instruction locality
2. Leverage the current strength and future improvement of compiler code optimization
3. Code generation system have clean interface and be understandable 
4. Good branch prediction and data prefetch

# Design
Keep data in register as long as possible, data materialization points as the boundary of execution stages.
1. pipeline breaker: move data out of cpu register
2. full pipeline breaker: need to materialize all records before continue to process

parallelization
1. inter-query parallelism: query assigned to different workers
2. intra-query parallelism: partition input data to be handled by different processors
3. inter-tuple parallelism: SIMD

# Design Decision
Generate C++ text code vs LLVM IR(preferred)
1. LLVM IR compilation time is shorter(milliseconds)
2. LLVM IR can be JIT executed.
3. LLVM IR can call C++ functions for complex logic.

Inline the whole complex query into a function vs compile blocking loop as a separate function(preferred)
1. Code size can grow exponentially
2. Large function size result in bad instruction locality
3. Need to try best efforts to make sure critical paths don’t cross function boundary




