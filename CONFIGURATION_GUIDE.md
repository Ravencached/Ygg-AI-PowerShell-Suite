# Configuration and credential storage

The suite separates ordinary preferences from credentials. `config.example.json` documents the supported settings. Copy it to a private per-user location and edit that copy. **Do not** put API keys in JSON, the PowerShell profile, source control, or a command that becomes part of shell history.

## Create your private configuration

In PowerShell 7, run:

```powershell
$configDirectory = Join-Path $HOME '.config\YggAIPowerShellSuite'
New-Item -ItemType Directory -Path $configDirectory -Force
Copy-Item '.\config.example.json' (Join-Path $configDirectory 'config.json')
```

Edit the resulting `config.json`. Set `ProjectsPath` and `SharePath` to paths on your own computer, or leave them `null` to disable those shortcuts. The profile checks this location at startup. Keep this private file outside the cloned repository.

## Use Windows SecretManagement for API keys

PowerShell SecretManagement provides a common interface to secret vaults. The SecretStore extension stores secrets locally for the current user. Install the modules from a PowerShell 7 session:

```powershell
Install-Module Microsoft.PowerShell.SecretManagement -Scope CurrentUser
Install-Module Microsoft.PowerShell.SecretStore -Scope CurrentUser
```

Register and unlock a local vault, then store the key without putting it in a command literal:

```powershell
Register-SecretVault -Name YggLocal -ModuleName Microsoft.PowerShell.SecretStore -DefaultVault
$secureKey = Read-Host 'Gemini API key' -AsSecureString
Set-Secret -Name GeminiApiKey -Secret $secureKey
Remove-Variable secureKey
```

On first use, SecretStore may prompt you to create a vault password. Follow its prompt and retain that password safely. To retrieve the key for a request, use:

```powershell
$apiKey = Get-Secret -Name GeminiApiKey -AsPlainText
```

The plaintext exists in process memory while the request is made. Secret storage protects the saved value at rest; it cannot prevent a process running as your user from reading it when the application retrieves it.

The profile first checks the configured environment variable. If it is empty and SecretManagement is installed, it tries the configured `GeminiSecretName`. Set `UseGemini` to `false` for local-only routing.

## Reuse configuration in other PowerShell scripts

Other scripts can read the same JSON preferences without dot-sourcing the profile:

```powershell
$configPath = Join-Path $HOME '.config\YggAIPowerShellSuite\config.json'
$config = Get-Content -LiteralPath $configPath -Raw | ConvertFrom-Json
$projectsPath = $config.ProjectsPath
```

Retrieve a credential by its secret name only in the script that needs it:

```powershell
$apiKey = Get-Secret -Name $config.GeminiSecretName -AsPlainText
```

For Python, JavaScript, or other runtimes, use that platform's supported operating-system credential store integration. Do not have a web page or a config file write API keys as plaintext. The JSON settings are portable preferences, while credential retrieval should remain local to each application and user context.

## Use an environment variable instead

You can skip SecretManagement and set the configured key variable in the current session:

```powershell
$env:GEMINI_API_KEY = Read-Host 'Gemini API key' -MaskInput
```

The key is not saved to the profile or config file. Set it again in each new session. Avoid saving it as a machine-wide or user-wide environment variable if you want it protected from other processes running as your account.
