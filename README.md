# Braiins OS+ Integration for evcc

A native Go charger implementation for [evcc](https://github.com/evcc-io/evcc) that integrates Braiins OS+ Bitcoin miners as PV surplus loads.

Tested with **Antminer S19K Pro** running Braiins OS+.

## Features

- **PV surplus mining** — evcc controls power target directly via Braiins OS+ API
- **Intelligent decrease** — waits before reducing power to avoid long DPS recovery cycles
- **DPS cooperation** — reads hardware DPS constraints and respects min/step values
- **Daily session reset** — resets evcc energy counter at 23:59
- **Token management** — automatic re-authentication when token expires

## Requirements

- Braiins OS+ with **Power Target mode** enabled in tuner settings
- evcc built from source (this integration requires compilation)

## Installation

1. Clone or fork [evcc](https://github.com/evcc-io/evcc)
2. Copy `braiins.go` → `charger/braiins.go`
3. Copy `braiins.yaml` → `templates/definition/charger/braiins.yaml`
4. Build evcc — see [CONTRIBUTING.md](https://github.com/evcc-io/evcc/blob/master/CONTRIBUTING.md) for build instructions

After building, configure the miner through the evcc UI under **Devices → Chargers → Braiins Bitcoin Miner**.

## Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `uri` | string | — | Miner IP or hostname |
| `user` | string | `root` | Username |
| `password` | string | — | Password |
| `maxPower` | int | `0` | Max power in W (0 = miner default) |
| `dailyReset` | bool | `true` | Reset energy counter daily at 23:59 |
| `intelligentDecrease` | bool | `true` | Gradual power decrease |
| `minDecreaseDuration` | duration | `5m` | Wait before decreasing power |
| `decreaseStepInterval` | duration | `3m` | Wait between decrease steps |
| `powerTargetStep` | int | `300` | Step size in watts |

## How it works

evcc treats the miner as a charger. In PV mode, evcc calculates available surplus and calls `MaxCurrentMillis` — the integration converts amps to watts (`A × 230V`) and sets the Braiins OS+ power target via the local API.

The **intelligent decrease** logic prevents the miner from immediately throttling on short cloud cover: it waits `minDecreaseDuration` before starting to decrease, then reduces in `powerTargetStep` increments with `decreaseStepInterval` between steps.

## Compatibility

Tested against Braiins OS+ API v1.5.0.

## License

MIT License — Copyright (c) 2025 Tobias Huber (https://github.com/TobiasHuber1980)

See [LICENSE](LICENSE) for details.
