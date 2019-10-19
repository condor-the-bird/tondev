TON Client Library is shipped with a crypto module that contains the following set of crypto functions for TON blockchain.

- math and random: generate_random_bytes, modular_power, factorize;
- sha256, sha512;
- generate_random_ed25519_keys;
- scrypt;
- menmonic: mnemonic_get_words, mnemonic_generate_random, mnemonic_from_entropy, mnemonic_is_valid, mnemonic_seed_from_phrase_and_salt, mnemonic_entropy_from_phrase;
- HD Keys: hdkey_xprv_from_mnemonic, hdkey_secret_from_xprv, hdkey_public_from_xprv, hdkey_derive_from_xprv, hdkey_derive_from_xprv_path;
- NaCl: nacl_sign_keys, nacl_sign_keys_from_secret, nacl_box_keys, nacl_box_keys_from_secret_key, nacl_secret_box, nacl_secret_box_open, nacl_box, nacl_box_open, nacl_sign, nacl_sign_open, nacl_sign_detached;
- Key store: keystore_add, keystore_remove, clear.

## Key Store

The crypto module holds a set of key pairs accessible through a handle. A key pair can be added to a key store or removed from it. Once added to a keystore, a key pair gets a handle assigned to it. This handle can be used in relevant crypto functions instead of the key pair itself.

## Reference Links

Follow the link to our github repository to get the list of our crypto-functions: <https://github.com/tonlabs/ton-client-js/blob/75270514d6e1051fe7159b7bcc79f1110ebe6d1b/types.js#L55>

These functions are standard and familiar to the professional audience. For a brief guide, follow the link: <https://github.com/dchest/tweetnacl-js/blob/master/README.md#documentation> (search for functions starting with `nacl`).