# Specification of the *Gemina* format

This document describes the binary format of encrypted data and the
procedures for encrypting and decrypting data. It also sets rules for
the API that implementations should provide.

## Description

Currently there are four versions. All versions use AES in CBC mode,
PKCS #7 padding, and HMAC-SHA256 as message authentication code (MAC).

Lengths of the secret keys in bits:

| Version | AES | HMAC |
| :-----: | --: | ---: |
|    1    | 128 |  128 |
|    2    | 128 |  256 |
|    3    | 192 |  256 |
|    4    | 256 |  256 |

A *Gemina* key is the concatenation of both keys.

To derive a *Gemina* key from a password PBKDF2HMAC-SHA256 with a 128 bit
salt and 100,000 iterations is used.

The initialization vector (IV) for CBC, the key, and the salt
for PBKDF2HMAC must be created in a cryptographically secure way.

### Binary format

The following table shows the order in which the data items are
concatenated to form the binary data and the length in bytes
for each item:

|    Item    | `key` | `pwd` |   Notes   |
| ---------- | ----: | ----: | --------- |
| Version    |     1 |     1 | see below |
| Salt       |     0 |    16 |           |
| IV         |    16 |    16 |           |
| Ciphertext |  n*16 |  n*16 | n>0       |
| MAC        |    32 |    32 |           |

(`key`: encrypted with secret key; `pwd`: encrypted with password)

Version byte (hexadecimal):
- Version 1: `8a`
- Version 2: `8b`
- Version 3: `8c`
- Version 4: `8d`

### Encryption

- Split the *Gemina* key into an encryption key and a key for creating the MAC,
  each with the appropriate length according to the  selected version.
- Create a 16 bytes long unique IV.
- Pad the given data according to PKCS #7.
- Encrypt the padded data with AES in CBC mode using the first key and the created IV.
- Compute the message authentication code with HMAC-SHA256 and the second key from the
  concatenation of the version byte, the salt (if using a password), the IV and the
  ciphertext created above.
- Append the MAC to the before concatenated data to create the final output.

### Decryption

- Determine the version by evaluating the first byte of the supplied data.
- If using a password get the salt by taking 16 bytes from the second byte on
  and compute the *Gemina* key with PBKDF2HMAC.
- Split the *Gemina* key into a decryption key and a key for creating the MAC,
  each with the appropriate length according to the  determined version.
- Get the message authentication code by taking the last 32 bytes from the
  given data, recompute the MAC from the data without these 32 bytes with
  the second key and compare the values.
- Get the IV by taking 16 bytes from the 18th (if using a password) or the
  second byte on.
- Get the ciphertext by taking the bytes after the IV from the given data omitting
  the last 32 bytes.
- Decrypt the ciphertext with AES in CBC mode by using the first key and the IV.
- Unpad the plaintext according to PKCS #7 to create the final output.

## Implementations

### API

Any API must at least provide functions for encrypting and decrypting data with
a *Gemina* key or with a password (from which the *Gemina* key is derived).

The encryption functions should take the *Gemina* key or the password, plaintext data,
and the selected version and return binary data in the format defined above.

The decryption functions should take the *Gemina* key or the password and the binary
data in the format defined above and return the plaintext data.

Functions for verifying the authenticity and the integrity of the data should also be
provided. This verification must be done in the decryption functions before the data
is decrypted.

There should be a function which returns a cryptographically secure created *Gemina* key
of the required length.

See also the [source code][] and the [documentation][] of the reference implementation.

[source code]: https://github.com/andreas19/pygemina/blob/master/src/gemina/__init__.py
[documentation]: https://andreas19.github.io/pygemina/mod_api.html

### Known implementations

| Name                         | Programming language | Notes                    |
| ---------------------------- | -------------------- | ------------------------ |
| [PyGemina][]                 | Python 3             | reference implementation |
| [Go-Gemina][]                | Go                   |                          |
| [Gemina4J][]                 | Java                 |                          |
| [Gemina unit][]              | Object Pascal        | unit in [PascalUnits][]  |
| [Gemina-RS][]                | Rust                 |                          |

[PyGemina]: https://andreas19.github.io/pygemina/mod_api.html
[Go-Gemina]: https://pkg.go.dev/github.com/andreas19/go-gemina/gemina
[Gemina4J]: https://andreas19.github.io/gemina4j/
[Gemina unit]: https://andreas19.github.io/pascal-units/Gemina.html
[PascalUnits]: https://andreas19.github.io/pascal-units/index.html
[Gemina-RS]: https://docs.rs/gemina/

## Links
- [Crypto 101](https://www.crypto101.io/)
- [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard)
- [CBC](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#CBC)
- [PKCS #7 padding (see note #2)](https://tools.ietf.org/html/rfc2315#section-10.3)
- [HMAC](https://en.wikipedia.org/wiki/HMAC)
- [PBKDF2](https://en.wikipedia.org/wiki/PBKDF2)
- [SHA-256](https://en.wikipedia.org/wiki/SHA-2)


[![License](https://i.creativecommons.org/l/by/4.0/88x31.png "Creative Commons Attribution 4.0 International License")](https://creativecommons.org/licenses/by/4.0/)
