# Encrypted Pastebin — Web/Crypto — 3

## Challenge Info
- **Category:** Web/Crypto
- **Points:** 3 (3 flags)
- **Files/URL given:** Hacker101 CTF instance URL
- **Description provided:** An encrypted pastebin service. Users can create encrypted pastes via a form, and view them via `?post=<ciphertext>` parameter.

## TL;DR
- **Flag 1:** `^FLAG^1fd8fe727722f2c9baa7dfa876398ba8b76b1b7fe144c7473c6f159e60f4dabe$FLAG$`
- **Flag 2:** `^FLAG^51d984d08a9073e3b4051ab169d31eff4f538efa6db68fc7eb89e72cc8746a38$FLAG$`
- **Vulnerability/trick:** Flag 1 — CBC ciphertext manipulation (zeroing IV bytes reveals flag on decrypt error). Flag 2 — padding oracle attack on AES-128-CBC with a static server key, using a non-standard base64 alphabet (`-` for `+`, `!` for `/`, `~` for `=`).

## Recon
- **What I ran first:** Opened the challenge URL, found a form with `title` and `body` fields. Submitted a test paste, got redirected to `/?post=<ciphertext>`. Appended random garbage to `?post=` and observed error messages — server returns `PaddingException` when PKCS7 padding is invalid.
- **What I found:** The app uses AES-128-CBC with a static key. The ciphertext uses a custom base64 alphabet. The `PaddingException` error string in responses provides a working padding oracle — exactly the primitive needed for a CBC byte-at-a-time attack.

## Rabbit Holes
- **Tried:** Ran `poattack` with the ciphertext string directly in double quotes. The tool appeared to complete (100% sent) but blocks 2, 3, 4 returned as all-zero bytes while others had real varied hex.
  - **Why it failed:** The ciphertext contained `!` characters, which bash interprets as history expansion even inside double quotes. `set +H` or single quotes fixes this.
- **Tried:** Manually copying the decoded hex by hand into the poattack command.
  - **Why it failed:** Typo in hex introduced a "not divisible by blockSize" error. Fixed by storing the hex in a bash variable instead.
- **Tried:** Running poattack at default concurrency.
  - **Why it failed:** High concurrency caused unreliable oracle responses. Fixed with `-c 1`.

## Solve Steps
### Flag 1 — Basic Encryption Manipulation
1. Intercept the form POST submission in Burp Suite.
2. Replace the encrypted form input value with a single character `d`.
3. Forward the modified request. The server processes the malformed ciphertext and the flag appears in the response.

Why this works: Sending an invalid/non-decryptable ciphertext triggers an error path that leaks the flag.

### Flag 2 — Padding Oracle Attack
1. Identify the padding oracle: submitting a ciphertext with invalid PKCS7 padding returns a response containing `PaddingException`. Valid padding returns a different response (HTML or `UnicodeDecodeError`).

2. Note the non-standard base64 alphabet used by the server:
   - `~` → `=`
   - `!` → `/`
   - `-` → `+`

3. Obtain a fresh ciphertext by submitting a new paste, or decode one from a `?post=` URL.

4. Run `poattack` (padding-oracle-attacker) with the ciphertext decoded to hex, single-quoted to prevent bash history expansion:
   ```bash
   set +H
   poattack hex:7b58123cfac072893cd5e45f8b1d6edb... 16 \
     --error "PaddingException" -c 1 \
     -e 'base64(-!~)'
   ```

   Parameters explained:
   - `hex:<hex>` — raw ciphertext bytes as hex (decoded from custom base64 first)
   - `16` — AES block size
   - `--error "PaddingException"` — oracle response detection string
   - `-c 1` — single-threaded for reliability
   - `-e 'base64(-!~)'` — tells poattack to re-encode forged blocks back into the custom base64 alphabet before sending them to the oracle

5. The tool decrypts all blocks. The plaintext is a JSON payload:
   ```json
   {"flag": "^FLAG^51d984d08a9073e3b4051ab169d31eff4f538efa6db68fc7eb89e72cc8746a38$FLAG$", "id": "2", "key": "G6YW83X8o!Pzdgqj7s3BUA~~"}
   ```

Why this works: AES-128-CBC with a static IV/key allows byte-at-a-time decryption via a padding oracle. By submitting manipulated ciphertexts and observing whether the server accepts or rejects the padding, each plaintext byte can be recovered with at most 256 oracle queries per byte. The custom base64 is just a character substitution — it doesn't add security.

## Exploit / Script

An alternative manual exploit using Python's `padding-oracle-attacker` library or a custom script can also be used (available on request).

## Flag Capture
```
Flag 1: ^FLAG^1fd8fe727722f2c9baa7dfa876398ba8b76b1b7fe144c7473c6f159e60f4dabe$FLAG$
Flag 2: ^FLAG^51d984d08a9073e3b4051ab169d31eff4f538efa6db68fc7eb89e72cc8746a38$FLAG$
```
