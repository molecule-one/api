
## Molecule.one Batch Scoring API

All API endpoints use JSON as data format for both incoming and outcoming data.
We provide HTTP endpoints allowing you to:

- **Create batch scoring request**:
  ```sh
  curl .../api/v1/batch-search -X POST \
    -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
    -d '{"targets": ["<SMILES_1>", "<SMILES_2>", ...]}' 
  ```
  Where:
  -  `<TOKEN>` should be replaced with private API token you’ll get from us
  - `<SMILES_1>`  etc should be your target chemical compounds in SMILES format

  In response, you’ll receive JSON containing an unique ID of your batch scoring request, i.e.:

  ```json
  {"createdAt":"2020-03-18T16:35:45.038Z","id":"617853b3-fc8b-47c6-a060-9dcae9a860de","size":2,"updatedAt":"2020-03-18T16:35:45.038Z"}
  ```

  You can also configure your scoring request with additional parameters using the `params` object:
  - `exploratory_search`: `boolean` (default: `false`) - our system will include reactions will fewer supporting information - it may provide better results at the cost of increased duration.

  Example:
  ```sh
  curl .../api/v1/batch-search -X POST \
    -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
    -d '{"targets": ["<SMILES_1>", "<SMILES_2>", ...], "params": {"exploratory_search": true}}'
  ```

- **Check batch scoring status**:
  ```sh
  curl .../api/v1/batch-search/<ID> \
    -H "Content-Type: application/json" \
    -H "Authorization: ApiToken-v1 <TOKEN>"
  ```

  Where:
  -  `<TOKEN>` should be replaced with private API token
  - `<ID>` should be replace with batch scoring ID you got in previous step

  In response, you’ll get information about your batch scoring processing progress, i.e.:
  `{"queued":92,"running":4,"finished":104,"error":0}`

  -  Get partial or complete results:
  ```
  curl .../api/v1/batch-search-result/<ID> \
    -H "Content-Type: application/json" \
    -H "Authorization: ApiToken-v1 <TOKEN>"
  ```

  Where:
  -  `<TOKEN>` should be replaced with private API token
  - `<ID>` should be replace with batch scoring ID you got in previous step

  In response, you’ll get batch scoring results, i.e.:
  ```json
      [
        {
          "targetSmiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
          "status": "ok",
          "result": 1822.891357421875,
          "reactionCount": 1
        },
      ...
      ]
  ```    

- **Remove data**

  ```
  curl .../api/v1/batch-search/<ID> -X DELETE \
    -H "Content-Type: application/json" \
    -H "Authorization: ApiToken-v1 <TOKEN>"
  ```
This will remove all data compound data and stop processing your batch scoring request.

FAQ:

- Q: Why do I get different results using batch scoring request and molecule.one's webapp for the same target?
- A: Request with exploratory_search turned on runs the same search as the one accessible via the web interface. By default it's disabled - it runs search with smaller computational budget, so for some compounds the batch score may not find the pathways found via the web interface.
