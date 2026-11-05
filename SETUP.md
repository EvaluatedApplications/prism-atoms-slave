# Atoms slave: setup and run (for the agent on the second laptop)

This page is for whoever (or whichever agent) sets up the SLAVE laptop. The master is the owner's main laptop running Prism. Your job: get `prism-atoms-slave.exe` running there so it trains some of the master's atoms over the network. Do only what is listed. If a step would break one of the owner's rules or touch something not listed here (firmware, BIOS, driver flashing, power or clock limits, other people's files), stop and say so first.

## What this machine needs
- Windows with an NVIDIA GPU and a current NVIDIA driver (`nvidia-smi` must print the card). No .NET install needed, the zip is self-contained.
- Mains power, sleep and hibernate off while it runs (`powercfg /change standby-timeout-ac 0`, `powercfg /hibernate off` is NOT needed, only the standby timeout). Restore the old values afterwards and tell the owner what you changed.
- A network path to the MQTT broker (Mosquitto) the owner already runs. Ask for its host and port; never guess one and never use any internet default.

## Get the program
Download `PrismAtomsSlave-win-x64.zip` from the Releases page of this repository and unzip it to `C:\PrismAtomsSlave\`. Do not edit its files. Do not build from source (there is none here).

## Check the broker first
```powershell
Test-NetConnection <broker-host> -Port <broker-port>      # TcpTestSucceeded must be True
```
If False: the master's Windows firewall or Mosquitto listener is the usual cause (Mosquitto 2 only listens on localhost unless its config has `listener 1883` and `allow_anonymous true`, and the firewall must allow TCP 1883). Report which; do not change the master yourself.

The messages are large (about 36 MiB per direction per step at 48 atoms, split into chunks). If the broker drops them, the master's mosquitto.conf may need `max_packet_size 0` and `message_size_limit 0`; report it, the owner decides.

## Probe, then run
```powershell
cd C:\PrismAtomsSlave
$env:PRISM_BROKER = "<broker-host>"          # add  $env:PRISM_BROKER_PORT = "<port>"  if not 1883
.\prism-atoms-slave.exe <room-code> --probe-only
```
`<room-code>` is the room name chosen on the master (it defaults to `1234`; the master shows it in the Model tab). The slave can be started BEFORE the master: it prints `waiting for room <code>` and retries every 5 seconds until the master opens the room, and goes back to waiting if the master stops. Only Ctrl+C ends it.

Then the real run (leave the window open):
```powershell
.\prism-atoms-slave.exe <room-code>
```
Watch the `ramp` lines: the master hands this machine 1 atom, then more while it keeps up, and takes some back when it makes the master wait. A line saying the slave is parked means the split does not pay at this network speed; that is a valid result, report it, do not try to force it.

Stop with Ctrl+C once; it finishes the step and sends its atoms home. Never kill the process mid-step unless it hangs for over 2 minutes after Ctrl+C.

## Options
`--max-atoms N` cap atoms on this machine (use if VRAM runs out), `--quiet` fewer lines, `--cpu` smoke test with no GPU (never for real runs). `--wire f16` is refused on purpose until the owner approves it.

## Exit codes`n0 normal (Ctrl+C). 1 cannot run (message printed). 2 bad arguments or no `PRISM_BROKER` set. 3 no CUDA GPU found.

## Report back (short)
1. `nvidia-smi` card name and VRAM, driver version.
2. `Test-NetConnection` result and the link type (wired gigabit or Wi-Fi).
3. The `--probe-only` output.
4. After a run of at least 5 minutes: atoms held at the end, the ramp line pattern, any `lost`, `refused` or `failed` lines, and the master's step time if the owner shares it.
5. Every setting you changed and its original value.

## Known limits
One slave per session, Memory coupling only, float32 only. Ramp constants are untuned.