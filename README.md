# What Is My IP - Logic App

An Azure Logic App that returns the client's public IP address in multiple formats (plain text, JSON, or JSONP).

## Features

- Returns client's public IP address
- Supports multiple response formats:
  - **Plain text** (default)
  - **JSON** format
  - **JSONP** format (for cross-domain JavaScript requests)
- Extracts real client IP from `X-Forwarded-For` header
- Simple HTTP GET endpoint

## Deployment

### Using ARM Template

1. Deploy the ARM template to your Azure subscription:

```bash
az deployment group create \
  --resource-group <your-resource-group> \
  --template-file azuredeploy.json \
  --parameters logicAppName=whatismyip-app location=swedencentral
```

2. After deployment, the trigger URL will be displayed in the outputs. Save this URL as it contains the required authentication tokens.

### Getting the Trigger URL

#### Via Azure Portal
1. Navigate to your Logic App in Azure Portal
2. Click on "Logic app designer" or "Overview"
3. Click on the HTTP trigger ("When_a_HTTP_request_is_received")
4. Copy the "HTTP POST URL" - this includes the SAS token

#### Via Azure CLI
```bash
# Get the trigger URL with SAS token
az rest --method post \
  --uri "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Logic/workflows/whatismyip-app/triggers/When_a_HTTP_request_is_received/listCallbackUrl?api-version=2019-05-01" \
  --query "value" -o tsv
```

#### Via PowerShell
```powershell
$triggerUrl = Get-AzLogicAppTriggerCallbackUrl \
  -ResourceGroupName "<your-resource-group>" \
  -Name "whatismyip-app" \
  -TriggerName "When_a_HTTP_request_is_received"
Write-Host $triggerUrl.Value
```

## Usage

Once deployed, you can call the Logic App endpoint with different format parameters:

### Plain Text Response (Default)
```bash
curl "https://prod-XX.region.logic.azure.com/workflows/.../invoke?api-version=2016-10-01&sp=...&sv=...&sig=..."
```
**Response:**
```
203.0.113.45
```

### JSON Response
```bash
curl "https://prod-XX.region.logic.azure.com/workflows/.../invoke?api-version=2016-10-01&sp=...&sv=...&sig=...&format=json"
```
**Response:**
```json
{
  "ip": "203.0.113.45"
}
```

### JSONP Response
```bash
curl "https://prod-XX.region.logic.azure.com/workflows/.../invoke?api-version=2016-10-01&sp=...&sv=...&sig=...&format=jsonp"
```
**Response:**
```javascript
callback({"ip":"203.0.113.45"});
```

## How It Works

1. The Logic App receives an HTTP GET request
2. Extracts the client IP from the `X-Forwarded-For` header
3. Checks the `format` query parameter
4. Returns the IP address in the requested format:
   - No format or unrecognized format → Plain text
   - `format=json` → JSON object
   - `format=jsonp` → JSONP callback

## ARM Template Parameters

| Parameter | Type | Default Value | Description |
|-----------|------|---------------|-------------|
| `logicAppName` | string | `whatismyip-app` | Name of the Logic App |
| `location` | string | `swedencentral` | Azure region for deployment |

## Deploy to Azure Button

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fgroovy-sky%2Fwhat-is-my-ip-logic-app%2Fmain%2Fazuredeploy.json)

## Important Notes

- The Logic App URL includes a SAS token for authentication. Keep this URL secure.
- The `X-Forwarded-For` header is used to get the real client IP when the Logic App is behind a proxy or load balancer.
- If the `X-Forwarded-For` header is not present, the Logic App will return `127.0.0.1` as a fallback.

## Example Use Cases

### Website Integration
```html
<!-- Display user's IP on a webpage -->
<script>
  fetch('YOUR_LOGIC_APP_URL&format=json')
    .then(response => response.json())
    .then(data => {
      document.getElementById('ip-address').textContent = data.ip;
    });
</script>
```

### Cross-Domain JSONP Request
```html
<script>
  function callback(data) {
    console.log('Your IP is: ' + data.ip);
  }
</script>
<script src="YOUR_LOGIC_APP_URL&format=jsonp"></script>
```

### Command Line
```bash
# Get your public IP
MY_IP=$(curl -s "YOUR_LOGIC_APP_URL")
echo "My IP is: $MY_IP"
```

## Troubleshooting

### Authentication Error
If you receive an `AuthorizationFailed` error:
- Ensure you're using the complete URL with SAS token parameters (`sp`, `sv`, `sig`)
- Get the correct URL from Azure Portal or using the CLI commands above

### Expression Evaluation Error
If you encounter expression evaluation errors:
- The template uses null-safe operators (`?`) to handle missing query parameters
- Ensure the ARM template is deployed correctly with the structured expression format

## License

MIT

## Contributing

Feel free to submit issues and pull requests.

## Author

Created by [@groovy-sky](https://github.com/groovy-sky)