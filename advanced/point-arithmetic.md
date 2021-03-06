# Elliptic curve point aritmhetic

A set of low-level APIs to perform computations over the edwards25519 curve, only useful to implement custom constructions.

Points are represented as their Y coordinate.

## Example

For a complete example using these functions, see the [SPAKE2+EE implementation](https://github.com/jedisct1/spake2-ee) for libsodium.

## Point validation

```c
int crypto_core_ed25519_is_valid_point(const unsigned char *p);
```

The `crypto_core_ed25519_is_valid_point()` function checks that `p` represents a point on the edwards25519 curve, in canonical form, on the main subgroup, and that the point doesn't have a small order.

It returns `1` on success, and `0` if the checks didn't pass.

## Hash-to-point (Elligator)

```c
int crypto_core_ed25519_from_uniform(unsigned char *p, const unsigned char *r);
```

The `crypto_core_ed25519_from_uniform()` function maps a 32 bytes vector `r` (usually the output of a hash function) to a point, and stores its compressed representation into `p`.

The point is guaranteed to be on the main subgroup.

## Scalar multiplication

```c
int crypto_scalarmult_ed25519(unsigned char *q, const unsigned char *n,
                              const unsigned char *p);
```

The `crypto_scalarmult_ed25519()` function multiplies a point `p` by a scalar `n` and puts the Y coordinate of the resulting point into `q`.

`q` should not be used as a shared key prior to hashing.

The function returns `0` on success, or `-1` if `n` is `0` or if `p` is not on the curve, not on the main subgroup, is a point of small order, or is not provided in canonical form.

Note that `n` is "clamped" (the 3 low bits are cleared to make it a multiple of the cofactor, bit 254 is set and bit 255 is cleared to respect the original design).

```c
int crypto_scalarmult_ed25519_base(unsigned char *q, const unsigned char *n);
```

The `crypto_scalarmult_ed25519_base(()` function multiplies the base point `(x, 4/5)` by a scalar `n` (clamped) and puts the Y coordinate of the resulting point into `q`.

The function returns `-1` if `n` is `0`, and `0` otherwise.

## Scalar multiplication without clamping

In order to prevent attacks using small subgroups, the `scalarmult` functions above clear lower bits of the scalar. This may be indesirable to build protocols that requires `n` to be invertible.

The `noclamp` variants of these functions do not clear these bits, and do not set the high bit either:

```c
int crypto_scalarmult_ed25519_noclamp(unsigned char *q, const unsigned char *n,
                                      const unsigned char *p);
```

```c
int crypto_scalarmult_ed25519_base_noclamp(unsigned char *q, const unsigned char *n);
```

These functions verify that `q` is on the prime-order subgroup before performing the multiplication, and return `-1` if this is not the case, or `0` on success.

## Point addition/substraction

```c
int crypto_core_ed25519_add(unsigned char *r,
                            const unsigned char *p, const unsigned char *q);
```

The `crypto_core_ed25519_add()` function adds the point `p` to the point `q` and stores the resulting point into `r`.

The function returns `0` on success, or `-1` if `p` and/or `q` are not valid points.

```c
int crypto_core_ed25519_sub(unsigned char *r,
                            const unsigned char *p, const unsigned char *q);
```

The `crypto_core_ed25519_sub()` function substracts the point `p` to the point `q` and stores the resulting point into `r`.

The function returns `0` on success, or `-1` if `p` and/or `q` are not valid points.

## Scalar arithmetic over L

Scalars should ideally be randomly chosen in the `[0..L[` interval, `L` being the order of the main subgroup (≈2^252).

This can be achieved with the following function:

```c
void crypto_core_ed25519_scalar_random(unsigned char *r);
```

`crypto_core_ed25519_scalar_random()` fills `r` with a `crypto_core_ed25519_SCALARBYTES` bytes representation of the scalar in the `]0..L[` interval.

A scalar in the `[0..L[` interval can also be obtained by reducing a possibly larger value:

```c
crypto_core_ed25519_scalar_reduce(unsigned char *r, const unsigned char *s);
```

The `crypto_core_ed25519_scalar_reduce()` function reduces `s` to `s mod L` and puts the ``crypto_core_ed25519_SCALARBYTES` integer into `r`.

Note that `s` is much larger than `r` (64 bytes vs 32 bytes). Bits of `s` can be left to `0`, but the interval `s` is sampled from should be at least 317 bits to ensure almost uniformity of `r` over `L`.

```c
int crypto_core_ed25519_scalar_invert(unsigned char *recip, const unsigned char *s);
```

The `crypto_core_ed25519_scalar_invert()` function computes the multiplicative inverse of `s` over `L`, and puts it into `recip`.

## Constants

* `crypto_scalarmult_ed25519_BYTES`
* `crypto_scalarmult_ed25519_SCALARBYTES`
* `crypto_scalarmult_ed25519_NONREDUCEDSCALARBYTES`
* `crypto_core_ed25519_BYTES`
* `crypto_core_ed25519_UNIFORMBYTES`

## Note

These functions were introduced in libsodium 1.0.16 and 1.0.17.
