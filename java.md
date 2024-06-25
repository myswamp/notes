# ConcurrentHashMap
  - CAS + synchronized before 8 segments

# Unsafe
  - direct memory access outside of jvm heap
  - low level operations, building lock, atomic data structures etc
  - performing reflective-like operations without the overhead of reflection
  - no type safety, lack of security checks, platform dependant, less portable

# synchronized
  - monitor enter & eixt, 
  - object header, mark word, 
  - biased lock, lightweight lock(cas), heavyweight lock, 
  - lock coarsening, 
  - lock elimination

# Dead lock detection
  - jconsole

# volatile
  - visibility, 
  - no atomicity

# happens before 
  

# CAS
  - ABA problem, solution: versioning; AtomicStampReference


# AQS
  - volative int state & CLH queue (FIFO)  
                                  


  
