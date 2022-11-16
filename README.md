# Magma ECIES

![image](https://user-images.githubusercontent.com/97805339/202318104-e58f49a4-1c76-453f-9765-ef95a6d105cb.png)

On the **HN**(Home Network) side, first, a **Key Pair**(HN Public Key and HN Private Key) is generated using Elliptic Curve Algorithm i.e. either `Curve25519/X25519` or `NIST Secp256R1` Elliptic Curve. The HN Public Key shall be stored in the **USIM**.

**Note:** *The HN Public Key of the network operator is also publicly accessible.*

On the USIM side, similarly, an **Ephemeral Key Pair** is generated using the same `Elliptic Curve Cryptography` but unlike the HN key pair, this pair is intended to use only for a short period of time or for a single transaction, that's why it is called Ephemeral Key Pair. The UE Ephemeral Public Key is then shared publicly so that it can be accessible by the HN.

An `EVP Key Agreement` is established between UE and HN, in which both agree on calculating a **Shared Secret** that can be further used as the basis for a key for some `symmetric encryption algorithm`. This Shared Secret is calculated from the peer's public key and self-private key using `Profile A` or `Profile B` Elliptic Curve Cryptography.

**Note:** *In Magma, this Shared Secret is termed as `Shared Key`*.

After this, a long **Shared Key Stream** is derived on both the UE and HN side from the UE Public Key and the Shared Key using the `KDF ANSI-X9.63` algorithm.

**Note:** *This **Shared KeyStream** is also known as `Shared Key` in the magma code, but for a better understanding, we will simply call it Shared KeyStream.*

This generated Shared KeyStream is further used to derive/divided into other keys such as **AES Key**, **MAC Key**, **AES Count** ,and **AES Nonce(Number Used Once)**. 

The 3 derived keys i.e. `AES Key`, `AES Count` ,and `AES Nonce` are taken as input in the `AES Cryptography in CTR(Counter) mode` to generate an **AES-128 Encryption Key** which is further used in `Symmetric Encryption` of **PlainText(SUPI/MSIN)** to **CipherText(SUCI).**

On the other side, HN uses the same 3 derived keys to generate the **AES-128 Decryption Key** which is further used for `Symmetric Decryption` of the **CipherText(SUCI)** to **Plaintext(SUPI/MSIN)**.

**Note:** *AES-128 Key is used for both to encrypt and decrypt a block of messages.*

But, the CipherText cannot be sent alone over the network, so it is sent along with a **UE MAC Value** and **UE Ephemeral Public Key** so that the subscriber can be identified by the network.

The **MAC Value** is generated from the **MAC Key** which is obtained from the shared keystream using a hashing algorithm `HMAC-SHA-256`.

**Note:** *Hashing Algorithm `HMAC-SHA-256` is a kind of signature(also called **digest**), constructed from the `SHA-256` hash function and used as a **Hash-based Message Authentication Code(HMAC)**. This algorithm mixes a message(SUCI) with a secret key(MAC Key) to generate a MAC Value. This MAC Value cannot be interrupted in any way to obtain the original inputs, only the user with the same parameters can generate the same MAC Value.*

Alike UE, HN generates its own MAC Value using the same obtained MAC Key from the Shared KeyStream. After getting the UE MAC Value, HN compares it with its MAC Value and if it matches, HN confirms that the message is from a genuine user and starts the decryption process. But if it is not, it has been considered that the message is not the original one or may be altered by a third party.

After successful verification, HN decrypts the **cipherText(SUCI)** to obtain **SUPI**.
