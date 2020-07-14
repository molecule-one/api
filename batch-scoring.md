
## Molecule.one Batch Scoring API

We provide HTTP endpoints which allow you to:


- Create batch scoring request:
  ```sh
  curl .../api/v1/batch-search -X POST \
    -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
    -d '{"targets": ["<SMILES_1>", "<SMILES_2>", ...]}' 
  ```
  Where:
  -  `<TOKEN>` should be replaced with private API token you’ll get from us
  - `<SMILES_1>`  etc should be your target chemical compounds in SMILES format

  In response, you’ll receive an unique ID of your batch scoring request, i.e.:

  ```json
  {"createdAt":"2020-03-18T16:35:45.038Z","id":"617853b3-fc8b-47c6-a060-9dcae9a860de","size":2,"updatedAt":"2020-03-18T16:35:45.038Z"}
  ```

- Check batch scoring status:
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

- Remove data

  ```
  curl .../api/v1/batch-search/<ID> -X DELETE \
    -H "Content-Type: application/json" \
    -H "Authorization: ApiToken-v1 <TOKEN>"
  ```
  _Warning: only already processed results will be removed. If you wish to remove all data related to your batch scoring request, you should wait for it to process before calling this endpoint._
