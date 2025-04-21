If all I have to do is keep my factors p and q secret, then I can save computation time by sharing the same modulus between all my friends. I'll give them unique e and d pairs to encrypt and decrypt messages. Sounds secure to me!

```
e = 58271
d = 16314065939355844497428646964774413938010062495984944007868244761330321449198604198404787327825341236658059256072790190934480082681534717838850610633320375625893501985237981407305284860652632590435055933317638416556532857376955427517397962124909869006289022084571993305966362498048396739334756594170449299859
N = 119082667712915497270407702277886743652985638444637188059938681008077058895935345765407160513555112013190751711213523389194925328565164667817570328474785391992857634832562389502866385475392702847788337877472422435555825872297998602400341624700149407637506713864175123267515579305109471947679940924817268027249
c = 107089582154092285354514758987465112016144455480126366962910414293721965682740674205100222823439150990299989680593179350933020427732386716386685052221680274283469481350106415150660410528574034324184318354089504379956162660478769613136499331243363223860893663583161020156316072996007464894397755058410931262938
```
```python
from Crypto.Util.number import *
from secret import flag

# This is my stuff! Don't look at it
p = getPrime(512)
q = getPrime(512)
N = p * q

e_priv = 0x10001
phi = (p - 1) * (q - 1)

d_priv = inverse(e_priv, phi)

m = bytes_to_long(flag)
c = pow(m, e_priv, N)

# This is your stuff!
e_pub = getPrime(16)
    
d_pub = inverse(e_pub, phi) 

print(f"e = {e_pub}")
print(f"d = {d_pub}")
print(f"N = {N}")
print(f"c = {c}")
```

## Solve
### Understanding the Original Code

The original source code shows a standard RSA implementation:

1. Two large primes `p` and `q` (512 bits each) are generated
2. The modulus `N = p * q` is calculated
3. A private exponent `e_priv = 0x10001` (65537) is used
4. The totient `phi = (p - 1) * (q - 1)` is calculated
5. The private key `d_priv = inverse(e_priv, phi)` is derived
6. The flag message `m` is encrypted using `c = pow(m, e_priv, N)`
Then the code provides you with:
- A new public exponent `e_pub` (a random 16-bit prime)
- The corresponding private key `d_pub = inverse(e_pub, phi)`
- The modulus `N`
- The ciphertext `c`
The critical vulnerability here is that you were given **both** `e_pub` and `d_pub` for the same `N`.

Knowing the private key `d` effectively means you can determine the prime factors of `N`, which breaks the entire RSA system.

```python
from Crypto.Util.number import long_to_bytes
import math

e = 58271
d = 16314065939355844497428646964774413938010062495984944007868244761330321449198604198404787327825341236658059256072790190934480082681534717838850610633320375625893501985237981407305284860652632590435055933317638416556532857376955427517397962124909869006289022084571993305966362498048396739334756594170449299859
N = 119082667712915497270407702277886743652985638444637188059938681008077058895935345765407160513555112013190751711213523389194925328565164667817570328474785391992857634832562389502866385475392702847788337877472422435555825872297998602400341624700149407637506713864175123267515579305109471947679940924817268027249
c = 107089582154092285354514758987465112016144455480126366962910414293721965682740674205100222823439150990299989680593179350933020427732386716386685052221680274283469481350106415150660410528574034324184318354089504379956162660478769613136499331243363223860893663583161020156316072996007464894397755058410931262938

def factor_n_using_ed(e, d, n):
    # Calculate k = e*d - 1, which is a multiple of φ(n)
    k = e * d - 1
    
    # Find a factor of n using k
    # Choose a random g and compute g^(k/2^t) mod n until we get a non-trivial factor
    # For simplicity, we'll use g = 2
    g = 2
    
    # Try different values of t
    t = 0
    while k % 2 == 0:
        k //= 2
        t += 1
    
    # Now k is odd
    for i in range(1, t + 1):
        x = pow(g, k * (2 ** (t - i)), n)
        if x != 1 and x != n - 1:
            y = math.gcd(x - 1, n)
            if y > 1:
                return y, n // y
    
    # If the above doesn't work, try a different approach
    # Based on the fact that ed ≡ 1 (mod φ(n))
    # This means ed - 1 = kφ(n) for some k
    # We know that φ(n) = (p-1)(q-1) = pq - p - q + 1 = n - p - q + 1
    # We can try to solve for p and q
    
    # Try factoring with Wiener's continued fraction approach
    for k in range(1, 100000):
        # Check if (e*d - 1) / k could be φ(n)
        if (e * d - 1) % k == 0:
            possible_phi = (e * d - 1) // k
            
            # φ(n) = n - (p+q) + 1, so p+q = n - φ(n) + 1
            sum_p_q = n - possible_phi + 1
            
            # Using the quadratic formula to find p and q
            # p and q are roots of x^2 - (p+q)x + pq = 0
            # Using the discriminant: sqrt((p+q)^2 - 4pq) = sqrt((p+q)^2 - 4n)
            discriminant = sum_p_q**2 - 4*n
            
            if discriminant >= 0:  # Ensure we can take the square root
                sqrt_discriminant = math.isqrt(discriminant)
                
                if sqrt_discriminant * sqrt_discriminant == discriminant:  # Perfect square check
                    p = (sum_p_q + sqrt_discriminant) // 2
                    q = (sum_p_q - sqrt_discriminant) // 2
                    
                    if p * q == n:  # Verify our factorization
                        return p, q
    
    return None, None

# Try to factor N
p, q = factor_n_using_ed(e, d, N)
print(f"p = {p}")
print(f"q = {q}")

if p is not None and q is not None:
    # Calculate phi (Euler's totient function)
    phi = (p - 1) * (q - 1)
    
    # Original encryption used e_priv = 0x10001
    e_priv = 0x10001
    d_priv = pow(e_priv, -1, phi)
    
    # Decrypt with the private key for e_priv
    m = pow(c, d_priv, N)
    flag = long_to_bytes(m)
    
    try:
        print("Flag as ASCII:", flag.decode('ascii'))
    except UnicodeDecodeError:
        print("Raw bytes:", flag)
        print("Printable characters:", ''.join(chr(b) if 32 <= b <= 126 else '.' for b in flag))
else:
    print("Failed to factor N. Trying direct decryption...")
    
    # As a fallback, let's try the straightforward approach
    m = pow(c, d, N)
    flag = long_to_bytes(m)
    
    try:
        print("Flag as ASCII (direct approach):", flag.decode('ascii'))
    except UnicodeDecodeError:
        print("Raw bytes (direct approach):", flag)
        print("Printable characters (direct approach):", ''.join(chr(b) if 32 <= b <= 126 else '.' for b in flag))
```