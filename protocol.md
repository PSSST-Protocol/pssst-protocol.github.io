{% include lib/mathjax.html %}

# Details of the PSSST Protocol

**A full paper detail the protocol and its security analysis is in progress. Come back soon!**

PSSST is designed to be simple and light weight. As a result it deliberately avoids complex packet structures and has very limited options. All field sizes for a given cipher suite are fixed, save for the message body which is always _the rest of the packet_. The packet header is a small 4 bytes and contains just critical flags and a cipher suite identifier. Since the key distribution is performed out-of-band it is easy to also check cipher suite compatibility out-of-band, which allows server implementations to be _straight line code_ once an initial switch on the cipher suite has been performed.

All cipher suites supported by PSSST are forms of Discrete Log Integrated Encryption Schemes (DLIES) and currently all specified suites use elliptic curve discrete logs as their basis. Each suite pre-defines a group _G_ used for key exchange, a key derivation function _KDF_ used to convert the raw result from the group operation into symmetric keys and an Authenticated Encryption with Additional Data (AEAD) scheme $$E(m, d)$$ used to protect both confidentiality and integrity of the message _m_ and the integrity of the additional data _d_. In the default cipher suite the [X25519](https://tools.ietf.org/html/rfc7748) key exchange scheme with a static key on the server side is used for _G_, [SHA256](https://csrc.nist.gov/publications/detail/fips/180/4/final) is used to form the key derivation function _KDF_ and [AES](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf) use in [Galois Counter Mode](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf) is used as _E_ to encrypt and authenticate the packets. Optionally, to facilitate client authentication the encrypted part of a request may contain proof that either the sender knew the client's private key or was capable of computing arbitrary discrete logarithms (which is assumed to be hard in _G_).

In the absence of the full paper describing the protocol details it is recommended that you look at the existing implementations for more detail.

