### Using jq

#### Sample json file

```
{
  "value": [
    {
      "displayName": "App One",
      "appId": "1111",
      "passwordCredentials": [
        {
          "displayName": "Secret A",
          "endDateTime": "2023-10-01T00:00:00Z"
        }
      ]
    },
    {
      "displayName": "App Two",
      "appId": "2222",
      "passwordCredentials": [
        {
          "displayName": "Secret B",
          "endDateTime": "2025-12-01T00:00:00Z"
        }
      ]
    }
  ]
}
```
> [!IMPORTANT]
> This is just an example of json file and should be considered as such.

#### Below how to extract data from the json file using jq
a. Extract all appNames
   ```
   jq '.value[].displayName' apps.json
   ```
b. Extract appNames + Secrets
  ```
    jq -r '.value[] | {App: .displayName, Secrets: .passwordCredentials[] | {Secret: .displayName, Expiry: .endDateTime}}' apps.json
  ```

c. Extract only expired secrets
  ```
    jq -r '
    .value[] |
    . as $app |
    .passwordCredentials[] |
    select(.endDateTime < "2025-01-01T00:00:00Z") |
    {
        AppName: $app.displayName,
        AppId: $app.appId,
        SecretName: .displayName,
        Expiry: .endDateTime
    }
    ' apps.json
  ```

d. Export to CSV
  ```
    jq -r '
    .value[] |
    . as $app |
    .passwordCredentials[] |
    select(.endDateTime < "2025-01-01T00:00:00Z") |
    [$app.displayName, $app.appId, .displayName, .endDateTime] |
    @csv
    ' apps.json > expired_secrets.csv

    ```
