# prism-atoms-slave

A small console program that lets a second Windows laptop with an NVIDIA GPU train some of the atoms of a Prism model. The Prism machine hands it one atom, then more while it keeps up, and takes some back when it makes the master wait.

- Download `PrismAtomsSlave-win-x64.zip` from the Releases page, unzip it, and follow [SETUP.md](SETUP.md).
- It needs an MQTT broker (Mosquitto) that both machines can reach. The broker address is never built in: set `PRISM_BROKER`.
- Binary only. The program is proprietary software under the licence in [LICENSE](LICENSE): limited use without a licence.

```
prism-atoms-slave <room-code> [--broker host[:port]] [--max-atoms n] [--probe-only] [--quiet] [--cpu]
```

Exit codes: 0 normal, 1 cannot run, 2 bad arguments or no broker set, 3 no CUDA GPU found.

Copyright (c) 2026 Dongyang Stephen Chen, trading as Evaluated Applications. All rights reserved.
