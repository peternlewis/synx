# Pseudorandom Number Generator

The generator maintains a 32-bit unsigned internal state initialized from a 4-byte seed. Each call updates the state using a linear congruential formula and applies a lightweight permutation to improve bit distribution. The generated values are deterministic for a given seed, ensuring that the same seed always produces the same sequence of random numbers.

# Generator Pseudocode
```
const MULTIPLIER = 147883721
const INCREMENT = 305663417
 
function seed(seed_value):
    state = seed_value
 
function next_u32():
    old_state = state
     
    // Advance state
    state = (old_state * MULTIPLIER + INCREMENT) mod 2^32
     
    // Output permutation
    scrambled = old_state
    scrambled = scrambled XOR (scrambled >> 16)
    scrambled = scrambled * MULTIPLIER
    scrambled = scrambled XOR (scrambled >> 16)
     
    return scrambled
```

