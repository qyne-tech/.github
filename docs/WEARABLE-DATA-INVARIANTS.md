# Wearable data invariants

**Read this before designing any feature that reads, writes, displays or derives
wearable data.** It is the standing set of constraints every such feature has to
satisfy. It does not restate the reasoning: that lives in the ADRs it links to.

## 0. Accuracy outranks everything

Correct and incomplete beats complete and wrong. A measurement we did not take
is `null`, never `0`, because a zero claims the athlete did not move. A value we
cannot attribute to a source is unattributed, not guessed. A statistic we cannot
convert is not converted. When a feature has to choose between showing something
and showing something true, it shows the true thing or shows nothing.

This is not a style preference. Readiness, recovery and training load are all
derived from this data, so a wrong number does not look wrong, it looks like a
physiological change the athlete did not have.

## 1. Readings are keyed per athlete, per wearable

The identity of a measurement is `(user_id, wearable_id, metric, sampled_at)`,
enforced as a unique index on `wearable_readings`. Not per source, not per
provider, not per device row.

The same measurement arriving through two sources of one band is one
measurement. Anything that aggregates, dedupes, backfills or upserts must key on
that tuple or it will double-count.

## 2. A wearable is not the transport it arrives through

Two levels, and they are not interchangeable:

- **`wearables`** is what the athlete owns. It is what a device tab is. It
  survives changing phone and changing health platform.
- **`wearable_devices`** is a **source**: one BLE peripheral on one phone, one
  HealthKit origin, one cloud account.

A BLE peripheral id is minted per phone on iOS, and a health platform is an
aggregator, so the transport is a bad proxy for the device. Read
[ADR-0005](https://github.com/qyne-tech/qyne-service/blob/develop/docs/decisions/0005-wearable-vs-source.md)
before touching identity, and
[ADR-0004](https://github.com/qyne-tech/qyne-service/blob/develop/docs/decisions/0004-per-device-identity.md)
for what it supersedes.

## 3. An athlete wears several at once

Multiple wearables are concurrent, not sequential. A band, a chest strap and a
watch can all report the same metric over the same minutes. No feature may
assume one active device, and none may silently pick a winner. Where the rules
cannot decide, the athlete does: `POST /wearables/{id}/merge` and
`PATCH /wearables/{id}` exist for exactly that.

## 4. Every feature must hold for every source

Not the one in front of you. The current set, with how each physically arrives:

| Provider key | Arrives as | Status |
|---|---|---|
| `jc_vita` | `ble` | QYNE band, live |
| `apple_health` | `healthkit` | live, read on-device |
| `health_connect` | `health_connect` | live, read on-device |
| `whoop` | `cloud` | live, OAuth pull with its own module |
| `garmin` | `cloud` | reserved key, no integration |
| `fitbit` | `cloud` | reserved key, no integration |
| `oura` | `cloud` | reserved key, no integration |

`garmin`, `fitbit` and `oura` are declared in `PROVIDER_KEYS` with brand
normalisation and display names, and have no ingest behind them. Treat them as
present-but-empty rather than absent: dispatch that assumes a provider is
implemented will break when one lands.

Confirm semantics from the source itself, never from how our code currently
behaves. For the band that means the vendored SDK at `qyne-app/vendor/`; for the
rest it means the provider's published docs.

## 5. Adding a source must need no migration

`provider` and `metric` are text, validated at the application layer in
`canonical.ts`. That is deliberate. A design that requires a schema change to
add a provider, or that enumerates providers in a switch outside the registry,
is the wrong design.

There are two integration shapes, and a new source is one of them:

- **Push** sources send samples for `normalize()` to translate. Implement
  `WearableProvider` and register it in `ProviderRegistry`. Controller, service
  and storage need no changes.
- **Pull** sources are OAuth account links we fetch summaries from. They own a
  module with their own client, token custody and webhook receiver, the shape
  `src/whoop` already has.

## 6. Known cross-source hazards

These have each caused a real defect. Check them by name.

- **HRV is not one statistic.** HealthKit reports SDNN, Health Connect reports
  RMSSD, and there is no conversion between them. They are stored as `hrv_sdnn`
  and `hrv_rmssd` precisely so a change of phone does not read as a drop in
  recovery. Never merge, average or plot them as one series. The band reports
  plain `hrv` and its vendor SDK names no statistic at all.
- **WHOOP currently stores RMSSD as plain `hrv`.** `whoop.mapper.ts` reads
  `recovery.score.hrv_rmssd_milli` and files it under `hrv`, so that bucket
  today holds both known RMSSD and the band's unspecified statistic. This is a
  known inconsistency, not the intended end state.
- **Units diverge by platform.** HealthKit returns distance in metres, Health
  Connect in kilometres. Convert at the edge and state the unit on the sample.
- **Identifiers are scoped.** A BLE peripheral id belongs to a phone, not a
  band. The band's MAC, read with opcode `0x22`, is the stable one and belongs
  in `source.hardwareId`, never in `externalId`.

## Before you design, answer these

1. What does each of the seven providers do for this feature, including the
   reserved ones?
2. What is the identity of anything written, and does it match
   `(user_id, wearable_id, metric, sampled_at)`?
3. Does it still behave when the athlete wears three devices reporting the same
   metric at once?
4. Does adding an eighth provider later require touching it?
5. Where a value is missing or unattributable, does it stay missing?
