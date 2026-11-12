# YggAIPowerShellSuite

A small PowerShell 7 profile with quality-of-life settings and optional AI explanations for the latest PowerShell error. It can help newer users understand failures and give experienced users a quick diagnostic starting point.

## Features

- Optional folder shortcuts for projects and shared storage
- `here` opens the current directory in File Explorer
- `why` displays PowerShell's detailed error view
- `why-ai` gives a short explanation of the latest PowerShell error
- `why-longai` asks for a longer explanation with a suggested read-only check
- Gemini-first responses with LM Studio fallback, or local-only mode with `-Local`
- Local model override with `-Model`
- Session title distinguishes standard and elevated PowerShell windows
- PSReadLine inline history suggestions

## Requirements

**Required**

- PowerShell 7
- PSReadLine, included with current PowerShell 7 releases

**Optional AI backends**

- Gemini API key stored in the environment variable configured by `GeminiApiKeyEnvVar`. Requests send the error message and error ID to Google's Gemini API.
- LM Studio with its local server enabled, a chat-completions model loaded, and an API identifier matching `LocalModel`.

For the local backend, an instruct model that returns ordinary text is recommended. Reasoning models can spend a small response-token budget on internal reasoning and return no visible explanation. Increase `QuickMaxTokens` if using one.

The default LM Studio endpoint is `http://127.0.0.1:1234/v1/chat/completions`. Without a Gemini key, the helper uses LM Studio directly. If neither service is available, it displays a request error.

## Install

1. Download `Microsoft.PowerShell_profile.ps1` and review it before installing. PowerShell runs profile scripts automatically when a session starts.
2. In PowerShell 7, check the profile path:

   ```powershell
   $PROFILE
   ```

3. Back up any existing profile, then copy the downloaded file to the path shown by `$PROFILE`. Create the parent directory if needed.
4. Edit the configuration block at the top of the profile. Replace the project and share path placeholders with paths on your computer, or set either value to `$null` to disable its shortcut.
5. Open a new PowerShell 7 session, or reload the profile with:

   ```powershell
   . $PROFILE
   ```

### Configure Gemini

Create an API key in [Google AI Studio](https://aistudio.google.com/apikey). Set it in each PowerShell session without placing it in the profile:

```powershell
$env:GEMINI_API_KEY = Read-Host 'Gemini API key' -MaskInput
```

The key variable name is configurable. This session-only method means you must set it again in a new session. Choose a secure secret store if you want persistent access.

### Configure LM Studio

Start LM Studio's local server and load a compatible chat model with the identifier configured in `LocalModel`. Then test with a harmless error followed by:

```powershell
why-ai -Local
```

## Commands

| Command | Behavior |
|---|---|
| `projects` | Change to the configured projects folder |
| `share` | Change to the configured share folder |
| `here` | Open the current folder in File Explorer |
| `why` | Show details for the latest PowerShell error |
| `why-ai` | Request a concise explanation, using Gemini then local fallback |
| `why-ai -Local` | Send the error only to LM Studio |
| `why-ai -Model 'model-id'` | Select a different local model for this request |
| `why-longai` | Request a longer explanation and a read-only diagnostic check |
| `why-longai -Local` | Request the longer answer from LM Studio only |

The helper uses the latest PowerShell error record. Native programs can report failure through an exit code without creating a useful PowerShell error record, so this profile does not yet diagnose those failures automatically. Error messages are limited to 2,000 characters before being sent to either model.

## Customize

The `$YggProfileConfig` block near the top of the profile contains the values most people will want to edit:

| Setting | Purpose |
|---|---|
| `ProjectsPath`, `SharePath` | Paths used by the folder shortcuts. Set to `$null` to omit a shortcut. |
| `WindowTitle`, `AdminWindowTitle` | Titles for standard and elevated sessions. |
| `UseGemini` | Set to `$false` to route all explanations locally. |
| `GeminiModel`, `GeminiApiKeyEnvVar`, `GeminiEndpoint` | Gemini model, key variable name, and endpoint. |
| `LocalModel`, `LocalEndpoint` | LM Studio model identifier and endpoint. |
| `GeminiTimeoutSeconds`, `LocalTimeoutSeconds` | Request timeouts. |
| `QuickMaxTokens`, `LongMaxTokens` | Response length limits. |

Edit the prompt strings in `why-ai` to adjust the answer style or tailor the diagnostic advice to another shell or workflow. Review any suggested command before running it, especially in an elevated session.

## Privacy

- No API key is included in this repository.
- When Gemini is enabled and its key is set, `why-ai` sends the latest error message and error ID to Gemini. Error details may contain usernames, paths, URLs, or other sensitive data. Use `-Local` for a specific request or set `UseGemini = $false` for local-only operation.
- The prompt asks the model to treat error text as data. Model responses are still suggestions. Review commands before running them.
- Keep personal paths, hostnames, API keys, and profile backups out of public commits.

## Support

If this project saves you time, you can support its development and other nonsense app development on [Ko-fi](https://ko-fi.com/ravencache).

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/S7T3213L8I)

## License

This project is available under the [MIT License](LICENSE).
