# Changelog

## v0.1.4 (processing v0.1.4)

### Changed

- Special characters like dots, spaces, etc are not replaced anymore when stored as `Value`s in `StoredValue` computes

### Fixed

- Error when calculating the `...Since` compute methods of `StoredValue`s

### Deprecated

- Support for StoredValue in MongoDB (currently not maintained anymore)

## v0.1.3 (processing v0.1.3)

### Added

- The reference value for a `DistinctValue` compute can also be of type integer

## v0.1.2 (processing v0.1.2)

- entity and reference fields do not require to access flat JSON anymore, but can be accessed with dictionary or dot notation in nested structures, e.g. `"entity": "card['id']"` or `"entity": "card.id"`
- RuleSets do not cancel computation when a referenced field is not found in the transaction. If this is the case, the related Rule will evaluate to false and the other rules will be evaluated
- RuleSets were not maintaining internal state of values (e.g. Rule 1 increases Score and Rule 2 Decreases Score, the prior increase will be ignored).
- maxValues are now working for Distinct Values when using Redis
- key namings in redis fixed, there was a bug for the `:` placements.
- "database" and "collection" field for mongo datastores now support environment variable expansion