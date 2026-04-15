# Stereo Imagery Extension Specification

- **Title:** Stereo Imagery
- **Identifier:** <https://stac-extensions.github.io/stereo-imagery/v1.0.0/schema.json>
- **Field Name Prefix:** stereo-img
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @m-mohr

This document explains the Stereo Imagery Extension to the
[SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
This extension helps to describe stereo imagery, tri-stereo imagery, etc.
This can be any group of 2+ captures with varying viewing angles that allows derive a 3-dimensional view from it.
The overlap of the captures should be significantly high (usually 75 % or more) and
the time difference between the captures should be minimal (usually a couple of minutes or less).

- Example:
  - [Collection](examples/collection.json): A Collection pointing to a pair of stereo imagery as Items.
  - [Item 1](examples/item1.json): The first Item of the pair.
  - [Item 2](examples/item2.json): The second Item of the pair.
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The extension allows to provide the captures as multiple Items.

The fields in the table below can be used in these parts of STAC documents:
- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)

| Field Name                              | Type    | Description |
| --------------------------------------- | ------- | ----------- |
| stereo-img:count                        | integer | **REQUIRED**. The total number of captures in the group. 2 for stereo imagery (minimum), 3 for tri-stereo imagery, etc. |
| stereo-img:group                        | string  | A unique identifier for the group of captures. Helps to search for all images of a group. |
| stereo-img:ground_sampling_distance     | number  | Width of a pixel projected on the surface, in meters. |
| stereo-img:solar_longitude              | number  | Solar longitude at acquisition time, in degrees [0°, 360°]. Used to assess seasonal compatibility between captures, particularly to avoid differences in frost patterns or surface albedo (e.g. relevant for Mars). |
| stereo-img:spectral_range               | object  | Spectral range of the image. See [Spectral Range Object](#spectral-range-object). Substantial differences in wavelength between captures may degrade stereo matching quality. Recommended: same filter or instrument-dependent spectral proximity. |

The fields in the table below can be used in these parts of STAC documents:
- [x] Item Properties (incl. Summaries in Collections)
- [x] Links (in Items)

| Field Name           | Type    | Description |
| -------------------- | ------- | ----------- |
| stereo-img:number    | integer | RECOMMENDED. The capture number in the group, must be > 0 and <= the total count. |

The order of the captures that is reflected in `stereo-img:number` can usually be derived
from the acquisition time (`datetime`) unless there's another specific order for the captures.

### Stereo Quality Metrics

The fields below describe stereo pair quality, evaluated at a common ground point between two captures.
They are based on Becker et al. (2015) — see [References](#references).

> **Note on acquisition angles:** Individual image acquisition angles (emission angle, incidence angle,
> solar azimuth, spacecraft azimuth) are managed by the
> [view extension](https://stac-extensions.github.io/view/v1.0.0/schema.json).
> The stereo-imagery extension is compatible with the view extension: when present,
> `view:off_nadir`, `view:incidence_angle` and `view:sun_azimuth` provide the source data
> to compute `stereo-img:parallax_height_ratio` and `stereo-img:shadow_tip_distance`.
> The view extension is **recommended but not required**.

The fields in the table below can be used in these parts of STAC documents:
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)

| Field Name                              | Type   | Description |
| --------------------------------------- | ------ | ----------- |
| stereo-img:parallax_angle               | number | Convergence angle between the two captures, in degrees. Human-readable indicator of stereo strength. Related to `stereo-img:parallax_height_ratio` (dp) by: `dp ≈ tan(parallax_angle)`. Limits: [5°, 45°] — Recommended: [20°, 30°]. |
| stereo-img:parallax_height_ratio        | number | Stereo strength metric (dp) per Becker et al. (2015). Computed from the emission vectors and spacecraft azimuth of both captures. More accurate than `parallax_angle` for asymmetric oblique geometries. When the view extension is present, computed from `view:off_nadir` and `view:sun_azimuth` of each capture. For radar, substitute `cot(emi)` for `tan(emi)`. Limits: [0.1, 1.0] — Recommended: [0.4, 0.6]. |
| stereo-img:shadow_tip_distance          | number | Illumination compatibility metric (dsh) per Becker et al. (2015). Quantifies the difference in illumination geometry between the two captures regardless of whether actual shadows are present. When the view extension is present, computed from `view:incidence_angle` and `view:sun_azimuth` of each capture. Limits: [0, 2.58] — Recommended: 0. |
| stereo-img:delta_solar_azimuth_angle    | number | Absolute difference in solar azimuth angle between the two captures, in degrees. Optional complement to `shadow_tip_distance` to ensure similar illumination. Limits: [0°, 100°] — Recommended: ≤20°. |
| stereo-img:gsd_ratio                    | number | Ratio of the larger to the smaller Ground Sampling Distance between the two captures at a common ground point. Pairs with a ratio above 2.5 should be resampled to the lower resolution before stereo processing. Limits: [1.0, 2.5]. |
| stereo-img:overlap_percentage           | number | Percentage of common surface coverage between the two captures, expressed as a ratio of the smaller area to the larger. Stereo convergence obtained by targeting captures off-nadir does not weaken the stereo geometry. Limits: [30%, 100%] — Recommended: [50%, 100%]. |

#### Recommended Values

The following table summarizes the limits and recommended values from Becker et al. (2015).
These are not absolute thresholds — acceptable ranges may vary based on the intended use and target terrain.

| Field                                    | Limits       | Recommended   |
| ---------------------------------------- | ------------ | ------------- |
| stereo-img:ground_sampling_distance      | —            | < 1/3 DTM GSD |
| stereo-img:parallax_angle                | [5°, 45°]    | [20°, 30°]    |
| stereo-img:parallax_height_ratio         | [0.1, 1.0]   | [0.4, 0.6]    |
| stereo-img:shadow_tip_distance           | [0, 2.58]    | 0             |
| stereo-img:delta_solar_azimuth_angle     | [0°, 100°]   | ≤ 20°         |
| stereo-img:gsd_ratio                     | [1.0, 2.5]   | [1.0, 2.5]    |
| stereo-img:overlap_percentage            | [30%, 100%]  | [50%, 100%]   |

#### Spectral Range Object

| Field Name         | Type   | Description |
| ------------------ | ------ | ----------- |
| min_wavelength_nm  | number | Minimum wavelength in nanometers. |
| max_wavelength_nm  | number | Maximum wavelength in nanometers. |
| filter_name        | string | Name or identifier of the spectral filter used. |

#### Technical Background

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

It is recommended to provide exact viewing angles, geometries and timestamps for each capture in the Item Properties.
Additionally, it is recommended to identify the actual stereo imagery in the assets with the role `data`.

## Relation types

The following types should be used as applicable `rel` types in the
[Link Object](https://github.com/radiantearth/stac-spec/tree/master/item-spec/item-spec.md#link-object).

| Type    | Description |
| ------- | ----------- |
| related | Link to the other captures in the group. |

If the `related` relation type is used, it is **REQUIRED** to provide the `stereo-img:number` and `type` fields in the Link Object.
This allows clients to distinguish them from other "related" links.

## Search Examples

If you'd want to search for distinct stereo imagery groups you could use the
API [Filter Extension](https://github.com/stac-api-extensions/filter) as follows:

- Find one image per pair for stereo-imagery: `stereo-img:count = 2 and stereo-img:number = 1`
  (and then navigate from the first to the other images)
- Only find tri-stereo imagery: `stereo-img:count = 3`
- Find high-quality stereo pairs: `stereo-img:parallax_height_ratio >= 0.4 and stereo-img:parallax_height_ratio <= 0.6`
- Find pairs with good illumination compatibility: `stereo-img:shadow_tip_distance <= 0.5`
- Find pairs with sufficient overlap: `stereo-img:overlap_percentage >= 50`

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
