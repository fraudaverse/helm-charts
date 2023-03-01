# Processing Service

## Release Notes

### v0.1.2 (processing v0.1.2)
- entity and reference fields do not require to access flat JSON anymore, but can be accessed with dictionary or dot notation in nested structures, e.g. `"entity": "card['id']"` or `"entity": "card.id"`
- RuleSets do not cancel computation when a referenced field is not found in the transaction. If this is the case, the related Rule will evaluate to false and the other rules will be evaluated
- RuleSets were not maintaining internal state of values (e.g. Rule 1 increases Score and Rule 2 Decreases Score, the prior increase will be ignored).
- maxValues are now working for Distinct Values when using Redis
- key namings in redis fixed, there was a bug for the `:` placements.
- "database" and "collection" field for mongo datastores now support envinroment variable expansion
