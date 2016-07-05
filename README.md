# Write-up-NDH2K1

## RSA

### Given files and hint
	
We were only given a single zip archive containing the following files:
		
*rsa_pub
*aes_key_cipher
*ciphermessage
	
We know from the topic that:
	
*RSA cipher was used to encrypt aes\_key\_cipher
*aes\_key\_cipher was used to encrypt ciphermessage
*aes was used without salt
		
### Solution
	
First we want to gather information about the given RSA public key, since we need to decode the AES key. Let's ask openssl :

```
openssl rsa -pubin -in rsa_pub -text
```

So we are facing RSA with a 2048bits modulo and a classical value of 65537 for the public exponent. The first thing to do is to check whether or not a factorization for the modulus as already been done. Let's ask factordb :

```
2999962175...03<624> = 10038779 · 2988373561...57<617>
```

Nice ! We already have a smooth factorization for this long-ass modulus. We know that the private exponent is the modular inverse of the public exponent modulo phi(modulus). Let's recover the private exponent and get that private key (i'll keep factordb notation for convenience):

```python
from Crypto.PublicKey import RSA
import gmpy

# Init known RSA paramaters
n = long(2999962175...03<624>)
e = long(65537)
p = long(10038779)
q = long(2988373561...57<617>)

# Get priv exponent
d = long(gmpy.invert(e,(p-1)*(q-1)))

# Get and print priv key in pem format
key = RSA.construct((n,e,d))
print RSA.exportKey(key)
```
Aaaaaaand we get :

```
-----BEGIN RSA PRIVATE KEY-----
MIIFMQIBAAKCAQQAjaVpGbUm1FIlrO1L5kUizvBKY5ELn2/+prESVUEBO+RdSLb7
JnG3VA5qTgtV46nkxFqNX1SgaZxlMtShKH+ssAixxW411gHcKp4uZlGJ6qNdIte+
olLB7PJwMatlfVs16CzecPglnS4U6YbzYuPobnvY5IEqUvLozC9puLDJWXeP2yQK
jhfLlXJFcBLYO2xycpDlC459oo/r36v1I9oDuUuUMir0IfnKAq3mBNqrks2cKCRE
NvEV/b7XbSyFAHrKf85JiTr4DnlVYy7HuJ9W+rAYduD+iCmaNzQNQ5yy4Bs8B+YM
iEdCG/4EnVmVQGsm9KCLINFJs8YarqNTHWL4+NHihwIDAQABAoIBBACDIRCrLBah
FuGWW1qHgAuAxzdLzR3eObIv/qEdR6MeAmY2QmMga2lvm21I54rKStCNAspH1CG+
ix/LJ1boMjTjsUPxowKJyUFP4yr3EaCyPvqiRCZdciDUm/0230fUfxpl8133u16P
CucGeuevMfs4ffgz4OtmpCU6JoxdtbN1J8vRWz3kkDNCVco7WeDdPgS1vcR6KZeB
HwYb9Xk+VjAHVN5YM8IPPSbkYlkNJ0c8/PEKIL4ynLGXOJvCR2zi1HZIhl/f9mA0
zUf5DD2DMQhByzOOtqWOV3/3Yk2tdlVZZ3v2OGiO3/NXEAEL6ZyG41WkG82blg2C
sxPCDaHe5HPzdkdZAoIBAQDsuYuEnwgauOUvcjEJhNMJY+6gBpyLm41DggIo1a8T
a5PMkBSBy8kyAJo5dtWZRXqbEor92ruC075A4we/pkw2hoBH9ZoeTa2t5BpzfFNh
L7wpxUtKtGEAEUTBgnTWBRReuiLgxiwqBa2ISwr7ns187Jz5LET/Bb5SJxmZsM7G
t9+z7bpQvp7NMVz9NCR9TF+oxisOimfSK8+nrdF0eMRDk7FSTgSNsGLbywS9BK64
4zjJabhOdG9wGOPLrPXLnUn6STeqGD+eJRPEW9SW8zrvA9iU+hGUuhiQmtn1E/ES
tAHCQY13B8B8v1nMDNYNuyx9+O8H6XhP1lnkrf7bLHPlAgQAmS37AoIBAGzBjcyf
N5z3Ryv2HXtPD5mn1LCmePNWwp66Mv3JtkaIzP1VUGaVVljnl/NAmj9xgTOPYFXi
UPV5DFZJN30gDLGcN4FX37d+XoWeX1yhSLlEsgDKyJ2Io2vhgyIYKk9NRB+FCpMT
2KRxuVj9iQ0y1xtGpZOAeC5l2BtsJUHLziPTxC2o0UlZWiHRRPR7KSx7kxM5//wN
MEeJozxZCfqlpR2a1AOJHmRuHez2p7WjWhZNJgC61lcM/UmV1cn0K3ShTaR0UOOP
gmLIi/1RZyj2lCPNM4q2HK+kk7aTvgDvaXj7RDeX4ENIR3HIg5vQZeMnE/jIDiEn
bQPY6bG7EnRlRtUCAz6h8wKCAQEAqXRCDolOPWZhhMRqUB0sekouUCBD7ICwDzlg
RJ7sT1sGCEtRc6c+clnn5p2qppcgIv/GbJDPKGsYeCakFY4Kf4sfff4NaPGL2BVs
MCNqXyl1WkrD58QtWBQgzjFcvLiZTnFScKrwi/5ohK0mjUFjkdXV2WLOp1T00+PC
tnyTF6APzrqjp7HUL2Ec3oVr3ErCxoq8ZgvJYsNdzUfkEkNHDRxHnV/7h4otEZvB
rgBYI5+yUYkiOnQiPpffPQQVWmIKN6wrGtCQWAnkS7alh9x35uopBLA8Nq+O/7dZ
DRbu8Ihi09wRu1ngJk2EeBlOg9jX//x10r2nq3EG+tDYkxSjoA==
```

We just copy-paste this nice private key to priv_key.pem and decrypt aes_key_cipher with it:

```
openssl rsault -decrypt -in aes_key_cipher -inkey priv_key.pem
```

Which gives us:

```
rsa_hackerzvoice_for_win
```

Noice. Seems like we got the hardest part. Let's decrypt the ciphertext with this password for AES, assuming they used aes-256-cbc:

```
openssl enc -aes-256-cbc -d -in ciphermessage -out ciphermessage.dec -k "rsa_hackerzvoice_for_win" -nosalt
```

Which smoothly gives us:

```
hey dude!

keep it up! https://www.youtube.com/watch?v=1F81S50xL8I

btw, i think the flag for this chall should be: ndh2k16_cac4015707

freeman
```	
