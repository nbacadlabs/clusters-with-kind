
### Extracting the data from Microsoft Graph
1. Sign-in to Microsoft graph https://developer.microsoft.com/en-us/graph/graph-explorer
2. On the right pane, select ```GET``` ```Version``` ```API```
   example query: ```https://graph.microsoft.com/v1.0/applications?$select=displayName,appId,passwordCredentials```
3. click on ```Run query```
4. at the bottom, click on ```Response preview```
5. create a file called ```filename.json``` in your directory
6. select all data on ```Response preview``` and paste it in the created json file
7. delete the @odata* lines. 
8. That completes the data preparation and sanitization.

### Uing jq to extract data from json file and output it as .csv file.

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
