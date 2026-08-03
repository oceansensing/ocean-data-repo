# ocean-data-repo

Real-time ocean data, fetched on a schedule and published as static JSON for
[oceansensing.org](https://oceansensing.org) to draw.

**This is not a public service.** The repository is public because GitHub
Pages requires it, not as an offer. There is no stability guarantee, no
versioning, no notice before a file changes shape or disappears, and no
support. If you need this data, go to the sources below — they publish it
properly and they are who deserve the credit and the traffic.

## What is here, and what is not

**The real-time data is never committed.** Currents, temperature, salinity,
storms, gliders, saildrones and floats are fetched into the Pages artifact on
every run; the previous run's copies are simply gone. That is the point of the
repository existing. One that commits what it fetches accumulates every
version of every file forever — the site this relieves had banked 356 MB of
superseded model grids for 130 MB of live data, none of it reclaimable without
rewriting history.

**The static data is committed**, in `map/`: the GEBCO isobaths and the
Natural Earth coastline and borders. The distinction is not convenience. The
seafloor does not change, so there is nothing to churn — and it *cannot* be
rebuilt here in any case, because contouring GEBCO needs a 7.5 GB grid that
is downloaded by hand and run once on a workstation. Committing it costs its
size once; fetching it would cost nothing less and would fail.

So a `git pull` gives you the seafloor and no ocean. For the ocean, fetch it
over HTTPS or run the pipelines yourself.

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

**The ESPC fields are a forecast, at T+36 from the run** — the hour the run
itself labels +36, not 36 hours from whenever this ran. That matters because
the run lands late: ESPC runs daily at 12Z and the aggregation ingests it 24
to 33 hours later, so its own T+0 is a field for yesterday and its T+36 is
about the present. Read `refTime` for the hour a file is valid at and
`modelRun` for the run it came from; `lead` is the difference, in hours.
OISST is an analysis and has neither a run nor a lead.

## Cadence

Hourly, offset from the hour. The ocean model runs once a day at 12Z, so most
runs republish the same model output against a fresher clock; the tiles are
rebuilt only when the run advances.
