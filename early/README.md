# Formal Verification: Attested TLS (draft-fossati-seat-early-attestation-03)

This repository contains the ProVerif formal verification model and query results for the **Early Attestation** extension in TLS 1.3 (`draft-fossati-seat-early-attestation-03`). 

~~~

let s_attest_base    = hash(StrongHash, (log_SH, s_attest_binder_label)) in
let s_attest_binder  = hash(StrongHash, (s_attest_base, (pubEK,pubLTK))) in
let rdata_nonce = s_attest_binder in
out(io, rdata_nonce); (* nonce is a public commitment *)
 
(* [...] *)

let dataSign = (rdata_nonce,dev_status1) in
let quote = (dataSign, sign(privAK,dataSign)) in

~~~

The model demonstrates the cryptographic isolation of the public commitment used for the binding attestation nonce and evaluates the security properties (secrecy and injective agreement) of applicationo data under various key compromise scenarios. 

> Note: All results hold under standard cryptographic assumptions: negotiated parameters use strong Diffie-Hellman groups, a collision-resistant hash function, and no malformed group elements are exchanged.

~~~
Run as follows with subfolder <traces>:
    proverif205 -lib tls-lib-simple.pvl -html traces tls13-multiagent.pv 2>&1 | tee log.txt
~~~

## 1. Injective Agreement over Application Data  (Authentication)

Based on the model, the queries prove **Injective Agreement**: If a client receives post-handshake application data, it must be mathematically guaranteed that the *exact* corresponding server actually sent that *exact* data (preventing Man-in-the-Middle, Relay, and Replay attacks).

| Query Condition (Client receives data $\to$ Server sent data, UNLESS...) | Result | Cryptographic Meaning |
| :--- | :--- | :--- |
| `... ==> ServerSendsAppData(...) || LeakedAK || LeakedLTK` | **TRUE** | Agreement holds as long as `AK` or `LTK` are secure. |
| `... ==> ServerSendsAppData(...) || LeakedAK || LeakedEK` | **TRUE** | Agreement holds as long as `AK` or `EK` are secure. |
| `... ==> ServerSendsAppData(...) || LeakedLTK || LeakedEK` | **TRUE** | Agreement holds as long as `LTK` or `EK` are secure. |
| **`... ==> ServerSendsAppData(...) || LeakedAK`** | **TRUE** | **Zero Relay/Replay:** The hardware signature over the unique session binder guarantees injective agreement. The attacker cannot forge the connection without the `AK`. |
| `... ==> ServerSendsAppData(...) || LeakedEK` | **FALSE** | If `EK` leaks, the attacker can forge the connection unless other keys protect it. |
| `... ==> ServerSendsAppData(...) || LeakedLTK` | **FALSE** | If `LTK` leaks, the attacker can forge the connection unless other keys protect it. |

**Conclusion on Agreement:**
The inclusion of the attestation binder inside the Evidence (`quote`) creates full Injective Agreement. By tying the hardware state to the highly-entropic, session-specific TLS running transcript, the connection becomes strictly bound. Replay attacks are mathematically impossible because the random nonces inside the transcript hash ensure the binder is entirely unique per connection.

## 2. Secrecy Properties for Application Traffic Keys

These queries evaluate the conditions under which an attacker can compromise the post-handshake server application traffic key (`ks`). 

The query structure is: `If the client receives valid app data AND the attacker learns the traffic key (ks), what MUST have been compromised?`

| Query Condition (If Attacker knows `ks`, then...) | Result | Cryptographic Meaning |
| :--- | :--- | :--- |
| `... ==> LeakedAK(pubAK) || LeakedLTK(pubLTK)` | **TRUE** | `ks` is secure unless at least the Attestation Key or Long-Term Key leaks. |
| `... ==> LeakedAK(pubAK) || LeakedEK(pubEK)` | **TRUE** | `ks` is secure unless the Attestation Key or Ephemeral Key leaks. |
| `... ==> LeakedEK(pubEK) || LeakedLTK(pubLTK)` | **TRUE** | `ks` is secure unless the Ephemeral Key or Long-Term Key leaks. |
| **`... ==> LeakedAK(pubAK)`** | **TRUE** | **Load-Bearing Key:** The attacker CANNOT compromise the session key `ks` without compromising the hardware Attestation Key (`AK`). |
| `... ==> LeakedLTK(pubLTK)` | **FALSE** | The Long-Term Key alone is not the sole load-bearing pillar; the session can be compromised without `LTK` leaking (e.g., if `AK` is compromised). |
| `... ==> LeakedEK(pubEK)` | **FALSE** | The Ephemeral Key alone is not the sole load-bearing pillar. |

**Conclusion on Secrecy:** The formal model proves that the hardware Attestation Key (`AK`) acts as a hard cryptographic backstop. Because the TEE signs the active session's public key and transcript hash (via the binder), an attacker cannot establish a spoofed session to read post-handshake data without possessing the private hardware key, even if traditional software keys have weaknesses.

## 3. Reachability and Sanity Checks

The model asserts a series of sanity checks to ensure the protocol is fully executable and that the client and server state machines align perfectly during an honest execution.

### Protocol Completion
* **`Query not event(ClientReceivesAppData(...)) is false.`**
    * **Result:** Reachable.
    * **Meaning:** An honest client and server can successfully negotiate the TLS handshake, exchange hardware attestation, derive the exact same traffic keys, and successfully exchange post-handshake application data.

### Key and State Agreement (Step-Wise Tests)
The model executes step-wise synchronization checks (`C_Test_...` and `S_Test_...`) to guarantee both peers compute the exact same state machine variables:

* **Nonces:** Both peers agree on `cr` (Client Random) and `sr` (Server Random).
* **Key Derivations:** Both peers independently derive identical values for `kc` (Client Traffic Key), `ks` (Server Traffic Key), `ems` (Exporter Master Secret), and `rms` (Resumption Master Secret).
* **Transcripts:** Both peers maintain mathematically identical running transcript hashes at `log_SH`, `log_CRT`, `log_CV`, and `log_SFIN`.

## Copyright and License

Copyright 2025 Muhammad Usama Sardar, Mariam Moustafa, and Tuomas Aura.
Copyright 2026 Nathanael Ritz.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

## Disclaimers

This work is a unique adaptation and extension, developed apart from the original
academic model released under the Apache 2.0 license: 

https://github.com/CCC-Attestation/formal-spec-id-crisis/tree/main

Original TLS 1.3 library model produced by Blanchet et al. is available
at: https://github.com/Inria-Prosecco/reftls/blob/master/pv/tls-lib.pvl

