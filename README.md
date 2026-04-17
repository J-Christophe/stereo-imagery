# Stereo Imagery Extension Specification

- **Title:** Stereo Imagery
- **Identifier:** <https://stac-extensions.github.io/stereo-imagery/v1.0.0/schema.json>
- **Field Name Prefix:** stereo-img
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Stereo Imagery Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
This extension describes a stereo imagery **product** — the output derived from two or more
captures with varying viewing angles that allows deriving a 3-dimensional view from it.

The source captures used to produce the stereo product can be provided in different ways:
- as **assets** within the same Item (recommended)
- as separate Items linked via `derived_from` links
- as separate Items within a Collection, linked via `related` links

- Example:
  - [Collection](examples/collection.json): A Collection pointing to a pair of stereo imagery as Items.
  - [Item 1](examples/item1.json): The first Item of the pair.
  - [Item 2](examples/item2.json): The second Item of the pair.
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

> **Note on acquisition angles:** Individual image acquisition angles (emission angle, incidence angle,
> solar azimuth, spacecraft azimuth) are managed by the
> [view extension](https://stac-extensions.github.io/view/v1.0.0/schema.json).
> The stereo-imagery extension is compatible with the view extension: when present,
> `view:off_nadir`, `view:incidence_angle` and `view:sun_azimuth` provide the source data
> to compute `stereo-img:parallax_height_ratio` and `stereo-img:shadow_tip_distance`.
> The view extension is **recommended but not required**.
>
> Other metadata already covered by STAC core or other extensions should use the standard fields:
> - **Spectral information**: use `bands` (STAC core)
> - **Ground sampling distance**: use `gsd` (STAC core)
> - **Solar longitude** (Ls): seasonal position of the planet around the Sun, used to assess
>   seasonal compatibility between captures (e.g. frost patterns on Mars). This field is planned
>   to be added to the view extension (`view:solar_longitude`).

The fields in the table below can be used in these parts of STAC documents:
- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)

| Field Name                              | Type    | Description |
| --------------------------------------- | ------- | ----------- |
| stereo-img:group                        | string  | A unique identifier for the stereo group. Present on both the stereo product Item and the individual capture Items. Combined with `stereo-img:number`, uniquely identifies a capture within the group. |
| stereo-img:count                        | integer | **REQUIRED**. The total number of source captures in the group. 2 for stereo imagery (minimum), 3 for tri-stereo imagery, etc. Present on both the stereo product Item and the individual capture Items. |
| stereo-img:parallax_angle               | number  | Convergence angle between the two captures, in degrees. Human-readable indicator of stereo strength. Related to `stereo-img:parallax_height_ratio` (dp) by: `dp ≈ tan(parallax_angle)`. Limits: [5°, 45°] — Recommended: [20°, 30°] (Becker et al., 2015). |
| stereo-img:parallax_height_ratio        | number  | Stereo strength metric (dp) per Becker et al. (2015). Computed from the emission vectors and spacecraft azimuth of both captures. More accurate than `parallax_angle` for asymmetric oblique geometries. When the view extension is present, computed from `view:off_nadir` and `view:sun_azimuth` of each capture. For radar, substitute `cot(emi)` for `tan(emi)`. Limits: [0.1, 1.0] — Recommended: [0.4, 0.6] (Becker et al., 2015). |
| stereo-img:shadow_tip_distance          | number  | Illumination compatibility metric (dsh) per Becker et al. (2015). Quantifies the difference in illumination geometry between the two captures regardless of whether actual shadows are present. When the view extension is present, computed from `view:incidence_angle` and `view:sun_azimuth` of each capture. Limits: [0, 2.58] — Recommended: 0 (Becker et al., 2015). |
| stereo-img:delta_solar_azimuth_angle    | number  | Absolute difference in solar azimuth angle between the two captures, in degrees. Optional complement to `stereo-img:shadow_tip_distance` to ensure similar illumination. Limits: [0°, 100°] — Recommended: ≤20° (Becker et al., 2015). |
| stereo-img:gsd_ratio                    | number  | Ratio of the larger to the smaller GSD (`gsd`) between the two captures at a common ground point. Pairs with a ratio above 2.5 should be resampled to the lower resolution before stereo processing. Limits: [1.0, 2.5] (Becker et al., 2015). |
| stereo-img:overlap_percentage           | number  | Percentage of common surface coverage between the two captures, expressed as a ratio of the smaller area to the larger. Stereo convergence obtained by targeting captures off-nadir does not weaken the stereo geometry. Limits: [30%, 100%] — Recommended: [50%, 100%] (Becker et al., 2015). |
| stereo-img:delta_time                   | string  | Time elapsed between the two captures, expressed as an ISO 8601 duration (e.g. PT2M30S for 2 minutes 30 seconds). A large time difference may degrade stereo matching quality due to surface changes (snow, vegetation, tides, dust storms) or atmospheric variations between the two acquisitions. The acceptable range is highly dependent on the target surface and application. |

The field in the table below can be used in these parts of STAC documents depending on how source captures are provided:
- when source captures are provided as **separate Items** : in Item Properties
- when source captures are provided as **assets** of the stereo product Item : in Asset fields
- [x] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [x] Links (in Items)

| Field Name           | Type    | Description |
| -------------------- | ------- | ----------- |
| stereo-img:number    | integer | RECOMMENDED. The capture number within the group, must be > 0 and <= `stereo-img:count`. Combined with `stereo-img:group`, uniquely identifies a capture within the group. Not present on the stereo product Item itself. |

The order of the captures reflected in `stereo-img:number` can usually be derived
from the acquisition time (`datetime`) unless there is another specific order for the captures.

It is recommended to provide exact viewing angles, geometries and timestamps for each capture in the Item Properties.
Additionally, it is recommended to identify the actual stereo imagery in the assets with the role `data`.

### Recommended Values

The following table summarizes the limits and recommended values from Becker et al. (2015).
These are not absolute thresholds — acceptable ranges may vary based on the intended use and target terrain.

| Field                                 | Limits       | Recommended   |
| ------------------------------------- | ------------ | ------------- |
| stereo-img:parallax_angle             | [5°, 45°]    | [20°, 30°]    |
| stereo-img:parallax_height_ratio      | [0.1, 1.0]   | [0.4, 0.6]    |
| stereo-img:shadow_tip_distance        | [0, 2.58]    | 0             |
| stereo-img:delta_solar_azimuth_angle  | [0°, 100°]   | ≤ 20°         |
| stereo-img:gsd_ratio                  | [1.0, 2.5]   | [1.0, 2.5]    |
| stereo-img:overlap_percentage         | [30%, 100%]  | [50%, 100%]   |

### Technical Background

**Parallax/Height Ratio (dp)**

The `stereo-img:parallax_height_ratio` (dp) is computed from the emission angle and spacecraft
azimuth of each capture:

```
px = -tan(emi) * cos(scazgnd)
py =  tan(emi) * sin(scazgnd)
dp = sqrt((px1 - px2)² + (py1 - py2)²)
```

where `emi` is the emission angle (`view:off_nadir`) and `scazgnd` is the spacecraft azimuth
on the ground. For radar imagery, substitute `cot(emi)` for `tan(emi)`.

The `stereo-img:parallax_angle` is the human-readable equivalent: `dp ≈ tan(parallax_angle)`.

**Shadow-Tip Distance (dsh)**

The `stereo-img:shadow_tip_distance` (dsh) is computed from the incidence angle and solar
azimuth of each capture:

```
shx = -tan(inc) * cos(sunazgnd)
shy =  tan(inc) * sin(sunazgnd)
dsh = sqrt((shx1 - shx2)² + (shy1 - shy2)²)
```

where `inc` is the incidence angle (`view:incidence_angle`) and `sunazgnd` is the solar azimuth
on the ground (`view:sun_azimuth`).

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type         | Description |
| ------------ | ----------- |
| derived_from | Link from the stereo product Item to the individual capture Items used to produce it. |
| related      | Link to the other capture Items in the group, when captures are provided as separate Items. |

If the `related` relation type is used, it is **REQUIRED** to provide the `stereo-img:number` and `type` fields in the Link Object.
This allows clients to distinguish them from other "related" links.

## Search Examples

If you'd want to search for distinct stereo imagery groups you could use the
API [Filter Extension](https://github.com/stac-api-extensions/filter) as follows:

- Find all stereo products: `stereo-img:count = 2`
- Find all tri-stereo products: `stereo-img:count = 3`
- Find individual captures belonging to a group: `stereo-img:group = 'CFE521'`
- Find high-quality stereo pairs: `stereo-img:parallax_height_ratio >= 0.4 and stereo-img:parallax_height_ratio <= 0.6`
- Find pairs with good illumination compatibility: `stereo-img:shadow_tip_distance <= 0.5`
- Find pairs with sufficient overlap: `stereo-img:overlap_percentage >= 50`
- Find pairs acquired close in time: stereo-img:delta_time <= 'PT10M'

All examples are expressed in CQL Text.

## References

Becker, K.J., Archinal, B.A., Hare, T.M., Kirk, R.L., Howington-Kraus, E., Robinson, M.S., Rosiek, M.R. (2015).
*Criteria for Automated Identification of Stereo Image Pairs*.
46th Lunar and Planetary Science Conference, Abstract #2703.

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
