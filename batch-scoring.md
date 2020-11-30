
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
  - `exploratorySearch`: `boolean` (default: `false`) - our system will include reactions will fewer supporting information - it may provide better results at the cost of increased duration.
  - `detailLevel`: `"score" | "synthesis"` (default: `"score"`) - if the specified level is `"synthesis"`, our system will include the best synthesis pathway that it found in addition to other values.

  Example:
  ```sh
  curl .../api/v1/batch-search -X POST \
    -H "Content-Type: application/json" -H "Authorization: ApiToken-v1 <TOKEN>"  \
    -d '{"targets": ["<SMILES_1>", "<SMILES_2>", ...], "params": {"exploratorySearch": true}}'
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

-  **Get partial or complete results**:
  ```
  curl .../api/v1/batch-search-result/<ID> \
    -H "Content-Type: application/json" \
    -H "Authorization: ApiToken-v1 <TOKEN>"
  ```

  Where:
  -  `<TOKEN>` should be replaced with private API token
  - `<ID>` should be replace with batch scoring ID you got in previous step

  In response, you’ll get batch scoring results, i.e.:
  ```js
      [
        {
          "targetSmiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
          "status": "ok",
          "result": 7.53411,
          "certainty": 0.58163,
          "price": 5232.5,
          "reactionCount": 5,
          "timedOut": false
        },
      ...
      ]
  ```    

  If the search was run with the `detailLevel: "synthesis"` parameter, additional field `synthesis` will be returned:
  ```js
      [
        {
          "targetSmiles": "<TARGET_SMILES>",
          ...,
          "synthesis": {
            "targetId": "<TARGET_ID>",
            "compounds": {
              "<CMPD_ID>": {
                "id": "<CMPD_ID>",
                "type": "compound",
                "smiles": "<CMPD_SMI>",
                "productOf": "<RXN1_ID>" | null,
                "substrateOf": "<RXN2_ID>" | null
              },
              ...
            },
            "reactions": {
              "<RXN_ID>": {
                "id": "<RXN_ID>",
                "type": "reaction",
                "smiles": "<SUBS1_SMI>.<SUBS2_SMI>>><PROD_SMI>", // mapped reaction smiles
                "productId": "<PROD_ID>",
                "substratesIds": [
                  "<SUBS1_ID>",
                  "<SUBS2_ID>"
                ],
              },
              ...
            }
          }
        },
      ...
      ]
  ```    
  
  You can fetch a subset of all returned values by specifying them in query string:
  ```
  curl .../api/v1/batch-search-result/<ID>?only=targetSmiles&only=result
  ```
  
  Returns:
  ```js
      [
        {
          "targetSmiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
          "result": 7.53411
        },
      ...
      ]
  ```    
  
  You can format the floating point scores returned by the system (`certainty`, `result`, `price`) to given number of significant digits using `precision` parameter: 
  ```
  curl .../api/v1/batch-search-result/<ID>?precision=3
  ```
  
  Returns:
  ```js
      [
        {
          "targetSmiles": "Cc1ccc(cc1Nc2nccc(n2)c3cccnc3)NC(=O)c4ccc(cc4)CN5CCN(CC5)C",
          "status": "ok",
          "result": "7.53",
          "certainty": "0.581",
          "price": "5230",
          "reactionCount": 5,
          "timedOut": false
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

## Result format description:
 - `targetSmiles` - Smiles specified by the user as an input in the batch request
 - `status` - `"ok" | "running" | "error"` - status of a given search. Pending searches are currently not listed in the output.
 - `result` - Estimated synthetic complexity of the target compound, based on the best pathway that was found, mapped to range [1;10]. Value `-1` is used as an error code.
 - `price` - Estimated price (in USD) of starting materials needed for synthesis of fixed number of moles of the target compound. (*possible null value)
 - `certainty` - Aggregated certainty of all reactions found in the best synthesis pathway in range of [0;1]. (*possible null value)
 - `reactionsCount` - Number of reactions in the best synthesis pathway. (*possible null value)
 - `synthesis` - Structure containing the best synthetic pathway found by the system. (*possible null value)
 - `timedOut` - Informs whether the search exceeded maximal allowed duration. This should happen only in case of compounds that are much bigger or much more complex than regular medicinal chemistry targets. Subsequent searches for the same compound may yield different results because of the variability of the search duration. If synthesis was found before exceeding the maximal duration, it will be used to compute the values above.

*possible null value - this field may be `null` if we weren't able to find any feasible pathway

FAQ:

- Q: Why do I get different results using batch scoring request and molecule.one's webapp for the same target?
- A: Request with exploratorySearch turned on runs the same search as the one accessible via the web interface. By default it's disabled - it runs search with smaller computational budget, so for some compounds the batch score may not find the pathways found via the web interface.
