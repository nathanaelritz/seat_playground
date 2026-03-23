# Formal Verification: Cocos AI Attestation vs. Mitigated Design

This repository contains the ProVerif formal verification models and query results comparing the baseline **Cocos AI** intra-handshake attestation implementation against a mitigated design based loosely on the `Early Attestation` draft.

The repository is split into two environments:
1. **Root Directory (`/`)**: Models the baseline Cocos AI implementation, demonstrating diversion and relay vulnerabilities.
2. **Fix Directory (`/fix/`)**: Models the mitigated protocol, utilizing a session-bound attestation nonce and Delegated Credentials cross-certification, which is shown to close the relay attack paths.

## Running the Models

To run the baseline model from the root directory and evaluate all properties:
```bash
proverif205 -lib tls-lib-simple.pvl -lib sanity.pvl -lib relay_attacks.pvl -html traces tls13-multiagent.pv 2>&1 | tee log-flawed-binder.txt

To run the mitigated model:
```bash
cd fix
proverif205 -lib tls-lib-simple.pvl -lib sanity.pvl -lib relay_attacks.pvl -html traces tls13-multiagent.pv 2>&1 | tee log-mitigating-binder.txt
```

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
