# ocean-data-repo

Real-time ocean data, fetched on a schedule and published as static JSON for
[oceansensing.org](https://oceansensing.org) to draw.

**This is not a public service.** The repository is public because GitHub
Pages requires it, not as an offer. There is no stability guarantee, no
versioning, no notice before a file changes shape or disappears, and no
support. If you need this data, go to the sources below — they publish it
properly and they are who deserve the credit and the traffic.

## What is here

Nothing, in git terms. The data is **fetched into the Pages artifact and
never committed** — that is the entire point of the repository existing.

A repository that commits what it fetches accumulates every version of every
file forever: the site this replaced carried 356 MB of superseded model grids
in its history for 130 MB of live data, and none of it could be reclaimed
without rewriting history. Here, each run publishes and the previous run is
gone. The repository stays a few hundred kilobytes of workflow and prose no
matter how long it runs or how much it serves.

So: no `git pull` will give you the data. Fetch it over HTTPS, or run the
pipelines yourself.

## Where it comes from

The pipelines are not here either. They live in the site repository —
[`oceansensing/oceansensing.github.io/scripts/`](https://github.com/oceansensing/oceansensing.github.io/tree/main/scripts)
— and this workflow checks that repository out and runs them. One copy of
the code, so the shape of what is published cannot drift from the contract
the map reads it against
([`packages/ocean-map/schema.ts`](https://github.com/oceansensing/oceansensing.github.io/blob/main/packages/ocean-map/schema.ts)).

| data | source |
| --- | --- |
| storms | NOAA National Hurricane Center |
| saildrones | NOAA PMEL |
| gliders | IOOS, NOC/BODC, OTN, VOTO |
| Argo floats | Argo GDAC via Ifremer |
| currents, SST, salinity | US Navy ESPC-D-V02, via HYCOM's OPeNDAP |
| observed SST | NOAA PSL OISST v2.1 |

Each file names its own source and, where it has one, the model run it came
from.

## Cadence

Hourly, offset from the hour. The ocean model runs once a day at 12Z, so most
runs republish the same model output against a fresher clock; the tiles are
rebuilt only when the run advances.
