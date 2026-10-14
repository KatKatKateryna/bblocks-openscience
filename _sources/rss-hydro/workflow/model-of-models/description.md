## Model of Models workflow

Provenance of three activities of the RSS-Hydro Model of Models (MoM) flood workflow, GFMS only.

| Activity | Activity type | Uses | Generates |
|---|---|---|---|
| Cell attribution (optional) | `mom:WatershedCellAttribution` | watershed shapefile, GFMS grid | GFMS cell lookup |
| Per-watershed aggregation | `mom:PerWatershedAggregation` | GFMS file, cell lookup, parameters | per-watershed aggregates |
| Hazard score accumulation | `mom:HazardScoreAccumulation` | aggregates, previous summary, GloFAS file, static tables, parameters | summary with duration, `Final_Attributes`, `Attributes_Clean` |

Each activity is `dcterms:isPartOf` one `mom:WorkflowRun` activity.

Cell attribution runs only when the lookup is missing or outdated; otherwise the lookup points to the earlier run that built it. The lookup derives from the versioned watershed shapefile (`mom:watershedFingerprint`) and the GFMS grid (`mom:gridSignature`), which ties each aggregate to the grid and watersheds it was clipped with; the grid must match the GFMS data. Parameters and static tables ship with the software version (`dcterms:hasVersion`, `dcterms:isPartOf`).

GFMS and GloFAS files state `mom:dataType`, `dcterms:format`, `mom:dataTime` (time the values refer to) and `generatedAtTime` (time the file was produced). The schema enforces this; `rules.shacl` checks the links between activities.
