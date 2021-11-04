# Molecule.one Batch Scoring API v2

All API endpoints use JSON as data format for both incoming and outcoming data.
We provide HTTP endpoints allowing you to:

## Create batch scoring request

### Basic request

```sh
curl <DOMAIN>/api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...]}'
```

Where:

- `<DOMAIN>` is a host address of the instance that you use, e.g. app.molecule.one
- `<TOKEN>` should be replaced with the private API token you’ll get from us
- `<TARGET_1>` etc should be your target chemical compounds in SMILES format

In response, you’ll receive JSON containing the unique ID of your batch scoring request, e.g.:

```json
{
  "createdAt": "2020-03-18T16:35:45.038Z",
  "id": "617853b3-fc8b-47c6-a060-9dcae9a860de",
  "size": 2,
  "updatedAt": "2020-03-18T16:35:45.038Z"
}
```

### Starting materials

You can also provide your starting materials using the `starting_materials` array.

Example:

```sh
curl .../api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...], "starting_materials": ["<MATERIAL_1>", "<MATERIAL_2>", ...]}'
```

Where:

- `<TOKEN>` should be replaced with the private API token you’ll get from us
- `<TARGET_1>` etc should be your target chemical compounds in SMILES format
- `<MATERIAL_1>` etc should be your starting material chemical compounds in SMILES format

### Output detail level

You can control detail level of system output by using `detail_level` attribute.

Possible values:

- `"score"` (default) - return the score and other properties of the best synthetic pathway found: certainty, price of starting materials and number of steps.
- `"synthesis"` - if specified, our system will include the best synthesis pathway that it found in addition to other values.

Example:

```sh
curl .../api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...], "detail_level": "synthesis"}'
```

### Job priority

In order to prioritize some jobs over other ones add `priority` attribute with an integer value ranging from `1` to `10` where `10` means the **highest** priority. Default `priority` value is `5`.

Example:

```sh
curl .../api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...], "priority": 7}'
```

### Parameters

You can also configure your scoring request with additional parameters using the `parameters` object:

- `model`: `"gat" | "megan"` (default: `"gat"`)
  Machine Learning model, each one can give different results.
- `time_limit`: `boolean` (default: `false`)
  Break the searches that take too long to complete.
- `length`: `"regular" | "short"` (default: `"regular"`)
  Defines how long the search is running, and for how many different intermediate compounds the system proposes reactions leading to them. "short" search is sufficient for compounds where you expect the synthetic pathway to be no longer than 5 steps.
- `synthesis_scale`: `"kilogram" | "molar" | "millimolar" | "micromolar"` (default: `"molar"`)
  The amount of compound that the system optimizes the synthesis for. Higher scale will result in longer synthetic pathway, starting at cheaper starting materials.
- `supplier_redundancy`: `boolean` (default: `true`)
  Describes whether the system requires the starting materials to be available at at least three different vendors.
- `availability_tier`: `[1:5]` (default: `3`)
  Values meaning:
  - 1 - All starting materials ship in 1-5 business days.
  - 2 - All starting materials ship in 2-10 business days.
  - 3 - All starting materials ship within 4 weeks.
  - 4 - Starting materials may require synthesis and are usually shipped within 12 weeks.
  - 5 - Starting materials require custom quote (POA).
- `name`: `string` (dafault: part of id) Name of batch search

Example:

```sh
curl .../api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...], "parameters": {"model": "megan"}}'
```

### Parameter sets

You can apply the parameter sets created in our web application by setting the `preset` attribute to the unique name specified during parameter set creation.

Example:

```sh
curl .../api/v2/batch-search -X POST \
  -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
  -d '{"targets": ["<TARGET_1>", "<TARGET_2>", ...], "preset": "simple_params" }'
```

## Check batch request details

```sh
curl .../api/v2/batch-search/<ID> \
  -H "Content-Type: application/json" \
  -H "Authorization: ApiToken-v1 <TOKEN>"
```

Where:

- `<TOKEN>` should be replaced with the private API token
- `<ID>` should be replaced with the batch scoring ID you got in the previous step

In response, you’ll get information about your batch request, e.g.:

```json
{
  "id": "899db985-5957-4718-b45a-8770c6e2bb99",
  "name": "899db985",
  "size": 1,
  "input_params": {},
  "starting_materials_size": 0,
  "created_at": "2021-08-01T04:00:04.223Z",
  "source": "API_V2"
}
```

## Check batch scoring status

```sh
curl .../api/v2/batch-search-status/<ID> \
  -H "Content-Type: application/json" \
  -H "Authorization: ApiToken-v1 <TOKEN>"
```

Where:

- `<TOKEN>` should be replaced with the private API token
- `<ID>` should be replaced with the batch scoring ID you got in the previous step

In response, you’ll get information about your batch scoring processing progress, e.g.:

```json
{
  "queued": 92,
  "running": 4,
  "finished": 104,
  "error": 0
}
```

## Get partial or complete results

```sh
curl .../api/v2/batch-search-result/<ID> \
  -H "Content-Type: application/json" \
  -H "Authorization: ApiToken-v1 <TOKEN>"
```

Where:

- `<TOKEN>` should be replaced with the private API token
- `<ID>` should be replaced with the batch scoring ID you got in the previous step

In response, you’ll get batch scoring results, e.g.:

```js
[
  {
    "target_smiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
    "status": "ok",
    "result": 7.53411,
    "certainty": 0.58163,
    "price": 5232.5,
    "reaction_count": 5,
    "timed_out": false,
    "name": "sample"
  },
  ...
]
```

If the search was run with the `detail_level: "synthesis"` attribute, additional fields `synthesis` and `decomposition` will be returned:

```js
[
  {
    "target_smiles": "<TARGET_SMILES>",
    ...,
    "synthesis": {
      "target_id": "<TARGET_ID>",
      "compounds": {
        "<CMPD_ID>": {
          "id": "<CMPD_ID>",
          "type": "compound",
          "smiles": "<CMPD_SMI>",
          "product_of": "<RXN1_ID>" | null,
          "substrate_of": "<RXN2_ID>" | null
        },
        ...
      },
      "reactions": {
        "<RXN_ID>": {
          "id": "<RXN_ID>",
          "type": "reaction",
          "smiles": "<SUBS1_SMI>.<SUBS2_SMI>>><PROD_SMI>", // mapped reaction smiles
          "product_id": "<PROD_ID>",
          "substrates_ids": [
            "<SUBS1_ID>",
            "<SUBS2_ID>"
          ],
        },
        ...
      }
    },
    "decomposition": {
      "atoms": [
        "<ATOM1_ID>",
        "<ATOM2_ID>",
        "<ATOM3_ID>",
        "<ATOM4_ID>",
        ...
      ],
      "atoms_not_purchased": [
        "<ATOM1_ID>",
        ...
      ],
      "atoms_purchased": [
        "<ATOM2_ID>",
        ...
      ],
      "bonds": [
        [
          "<ATOM1_ID>",
          "<ATOM2_ID>"
        ],
        [
          "<ATOM3_ID>",
          "<ATOM4_ID>"
        ],
        ...
      ],
      "bonds_broken": [
        [
          "<ATOM1_ID>",
          "<ATOM2_ID>"
        ],
        ...
      ],
      "bonds_not_broken": [
        [
          "<ATOM3_ID>",
          "<ATOM4_ID>"
        ],
        ...
      ],
      "mapped_smiles": "<MAPPED_TARGET_SMILES>"
    }
  },
  ...
]
```

Where:

- `<ATOM1_ID>` etc represents an identifier of the atom mapped in `<MAPPED_TARGET_SMILES>`

You can fetch a subset of all returned values by specifying them in query string:

```sh
curl .../api/v2/batch-search-result/<ID>?only=target_smiles&only=result
```

Returns:

```js
[
  {
    "target_smiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
    "result": 7.53411
  },
  ...
]
```

## Remove data

```sh
curl .../api/v2/batch-search/<ID> -X DELETE \
  -H "Content-Type: application/json" \
  -H "Authorization: ApiToken-v1 <TOKEN>"
```

This will remove all compound data (targets, starting materials, results) and stop processing your batch scoring request.

## Result format description

- `target_smiles` - SMILES specified by the user as an input in the batch request.
- `status` - `"ok" | "running" | "error"` - Status of a given search. Pending searches are currently not listed in the output.
- `result` - Estimated synthetic complexity of the target compound is a value between 1 and 10 that describes how hard it is to synthesize a given compound - the higher, the harder the synthesis is. Number of reactions in a pathway is the most important factor.
  - 1-2 - The compound is commercially available and is relatively cheap.
  - 2-4 - There is a short and straightforward synthesis found for this compound.
  - 4-6 - A relatively long pathway, pathway with some uncertain reactions or with expensive starting materials was found.
  - 6-10 - A long pathway with a lot of unlikely reactions was found or there is no synthesis found at all.
- `price` - Estimated price (in USD) of starting materials needed for synthesis of fixed number of moles of the target compound. (\*possible null value).
- `certainty` - Aggregated certainty of all reactions found in the best synthesis pathway in the range of [0;1]. (\*possible null value).
- `reaction_count` - Number of reactions in the best synthesis pathway. (\*possible null value).
- `synthesis` - Structure containing the best synthetic pathway found by the system. (\*possible null value).
- `decomposition` - Structure containing information about how the reactions generated by our system can be used to decompose the target compound into commercially available starting materials.
- `timed_out` - Informs whether the search exceeded maximal allowed duration. This should only happen for compounds that are much bigger or much more complex than regular medicinal chemistry targets. Subsequent searches for the same compound may yield different results because of the variability of the search duration. If synthesis was found before exceeding the maximal duration, it will be used to compute the values above.
- `url` - Url address to our system, where the search result is presented in a human-readable way.
- `name` - Batch search name

\*possible null value - this field may be `null` if we weren't able to find any feasible pathway
