# Sprint 2: Hash 

## Hash function
> A hash function H takes input with any length and return a fixed-length output. 

A good hash function has 3 properties:
1. One-way (preimage resistance): given `y`, it is not possible to find `x` so that `H(x) = y`
2. Hard to find another input (second-preimage resistance): given `x`, it is not posible to find `x'` $\neq$ `x` that yields the same hash value
3. Collision resistance: it is not possible to find any `x` $\neq$ `x'` with `H(x) = H(x')`

Also, the output must be random. Changing 1 input bit should cause half of output bits to alter.

## Some hash functions
- MD5: collision found -> unsafe
- SHA-1: collision found -> unsafe
- SHA-2: safe
- SHA-3: safer than SHA-2 but run slower on standard CPUs, softwares; less compatible with current systems -> less common than SHA-2

## SHA-3
> SDitH implements SHA-3. SHA-3 is based on **Keccak** algorithm.

### Keccak

> uses a sponge construction to process input data of arbitrary length and produce a hash output.

The sponge construction consists of two main phases:

- Absorbing: The input data is divided into blocks and absorbed into an internal state.

- Squeezing: The hash output is extracted from the internal state.

### State

Keccak uses an internal state represented by a three-dimensional array:

$$A[x][y][z]$$

SHA-3 uses Keccak-f[1600], which has following properties:
- The state consists of a 5 × 5 array of lanes.
- Each lane contains 64 bits.
- Total size = 5 x 5 x 64 = 1600 bits


The state can be represented as:

A[0][0]  A[1][0]  A[2][0]  A[3][0]  A[4][0]

A[0][1]  A[1][1]  A[2][1]  A[3][1]  A[4][1]

A[0][2]  A[1][2]  A[2][2]  A[3][2]  A[4][2]

A[0][3]  A[1][3]  A[2][3]  A[3][3]  A[4][3]

A[0][4]  A[1][4]  A[2][4]  A[3][4]  A[4][4]

Each element represents a 64-bit lane.

### Rate and capacity

The internal state is divided into two parts:

$$b = r + c$$

Where:

- b: total state size.

- r: rate, the part used to absorb input data and extract output.

- c: capacity, the part that contributes to the security of the construction.

Example: SHA3-256:

$$b = 1600,\quad r = 1088,\quad c = 512$$

$$1600 = 1088 + 512$$


The rate determines how many input bits can be processed in each absorption block, while the capacity is an important security parameter.

### Sponge Construction

The sponge construction has two main phases:

### Absorbing Phase

The input message is first padded according to the padding rule of the specific Keccak-based function, then the padded message is divided into blocks of r bits.


For each input block:

- XOR the block into the rate portion of the internal state.

- Apply the Keccak-f permutation to the entire state.

- Repeat until all input blocks have been processed.

- The operation can be expressed as:

$$S_{i+1} = f(S_i \oplus P_i)$$

Where:

- S_i is the current internal state.

- P_i is the input block.

- f is the Keccak-f permutation.

### Squeezing Phase

After all input blocks have been absorbed:

Extract output bits from the rate portion of the state.

If more output bits are required than the rate can provide, apply the permutation again.

Continue extracting output until the requested output length is reached.

(Eg: SHA3-256 -> output length is 256 bits.)


### Theta (θ)

Purpose: Mix information between the columns of the state.

First, calculate the XOR of all five lanes in each column:

$$C[x] = A[x][0] \oplus A[x][1] \oplus \cdots \oplus A[x][4]$$

Then, calculate:

$$D[x] = C[x-1] \oplus ROT(C[x+1],1)$$

Finally, update each lane:

$$A'[x][y] = A[x][y] \oplus D[x]$$

The column indices are calculated modulo 5.

Theta helps propagate changes across different columns of the state.

5.2. Rho (ρ)

Purpose: Rotate the bits within each lane.

Each 64-bit lane is rotated by a specific offset.

Unlike a normal bit shift, a rotation preserves all bits. Bits shifted out from one end are inserted back at the other end.

This operation changes the positions of bits within each lane.

5.3. Pi (π)

Purpose: Rearrange the positions of lanes in the state.

The transformation is defined as:

$$B[y][(2x+3y)\bmod 5] = A'[x][y]$$

Pi moves each lane to a different position in the 5 × 5 state array.

While Rho changes bit positions within a lane, Pi changes the positions of entire lanes.

5.4. Chi (χ)

Purpose: Introduce non-linearity into the state transformation.

The transformation is:

$$A'[x][y] = B[x][y] \oplus ((\neg B[x+1][y]) \land B[x+2][y])$$

The indices are calculated modulo 5.

Chi uses NOT, AND, and XOR operations.

It is the non-linear step of the Keccak round and plays an important role in providing cryptographic security.

5.5. Iota (ι)

Purpose: Break symmetry between rounds.

Iota XORs a round-dependent constant into the first lane of the state:

$$A'[0][0] = B[0][0] \oplus RC$$

Where RC is the round constant corresponding to the current round.

This prevents all rounds from having identical symmetric structures.
