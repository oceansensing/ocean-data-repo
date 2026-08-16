# ocean-data-repo

Real-time ocean data, fetched on a schedule and published as static JSON for
[oceansensing.org](https://oceansensing.org) to draw.

**This repository is frozen. It stopped publishing on 2026-08-16.**
[`realtime-data-repo`](https://github.com/oceansensing/realtime-data-repo)
publishes the same contract from a rebuilt pipeline — products with
per-run fates, a published health record, per-product failure isolation —
and the site's `MAP_DATA` has pointed at it since 2026-08-14. This one kept
running afterwards as a warm standby, so that switching back was one string
in `src/config.ts` rather than a rebuild; that period is over.

**Frozen, not removed.** Nothing was deleted. The history, the committed
static data and the last published tree all stand, and GitHub Pages goes on
serving that tree — so the data under it is a snapshot of 2026-08-16 and
ages from here. **Anything reading it is reading a fossil**, which is why it
is left readable rather than taken down.

What changed is one file: `.github/workflows/publish.yml` lost its schedule
and its push trigger and kept `workflow_dispatch`, so the pipeline is idle
but restartable by hand. The freeze is written in that file rather than only
switched off in the Actions UI, because a workflow disabled through the API
says nothing to someone reading the repository.

To bring it back, both halves are needed: restore the crons, and point
`MAP_DATA` back here. The design case for the successor is written in its
README.

**This is not a public service.** The repository is public because GitHub
Pages requires it, not as an offer. There is no stability guarantee, no
versioning, no notice before a file changes shape or disappears, and no
support. If you need this data, go to the sources below — they publish it
properly and they are who deserve the credit and the traffic.

**License:** the workflow and documentation are copyright (c) 2026 Donglai
Gong and C4PO, all rights reserved — see [LICENSE.md](LICENSE.md). The
scientific data belongs to its sources, as above.

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
| 10 m wind | ECMWF IFS open data (CC BY 4.0) |

Each file names its own source and, where it has one, the model run it came
from.

**The ESPC fields are a forecast, at T+36 from the run** — the hour the run
itself labels +36, not 36 hours from whenever this ran. That matters because
the run lands late: ESPC runs daily at 12Z and the aggregation ingests it 24
to 33 hours later, so its own T+0 is a field for yesterday and its T+36 is
about the present. Read `refTime` for the hour a file is valid at and
`modelRun` for the run it came from; `lead` is the difference, in hours.
OISST is an analysis and has neither a run nor a lead.

**The wind is a nowcast**, and that is the models differing rather than an
inconsistency: IFS runs four times a day and publishes within hours, so the
step nearest now is genuinely about now. Its `lead` is therefore small and
counted the same way — hours after its own run.

## Cadence

**Hourly at :05 for everything, and :25 and :45 for the storms alone** —
nominally. GitHub dispatches scheduled workflows late, measured on this
repository at **19 to 37 minutes** past the cron time over fourteen
consecutive runs, so read those as three publishes an hour rather than as
clock times.

The full run is hourly, offset from the hour. The ocean model runs once a day
at 12Z, so most runs republish the same model output against a fresher clock;
the tiles are rebuilt only when the run advances.

The two light runs exist because the National Hurricane Center does not
publish on this schedule: an advisory every three hours, every two once
watches or warnings are up, and a special advisory or outlook whenever a
forecaster judges one is needed. Against an hourly publish that is up to an
hour of avoidable lag on the product most likely to be read while it matters.

A light run fetches the storms, the saildrones, the gliders and the floats,
and nothing else — no HYCOM, no ECMWF, no tile builds. Everything else is
seeded from the previous publish, which is the same machinery that already
degrades an upstream outage into stale data rather than a blank layer. So
what it deploys is this hour's ocean under a fresher set of storms.

It restores the tile caches even though it does not build them: GitHub Pages
replaces the whole site on every deploy, so a tree assembled without them is
one where every reader past zoom 4 silently drops to the coarse grid. If the
cache misses outright the light run **skips its own deploy** rather than
publishing the gap — the previous publish stays up, the storms are then up to
another twenty minutes old, and the :05 run rebuilds regardless.
