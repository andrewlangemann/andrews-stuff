Instructions to store your GitHub Copilot token in an encrypted way, and use that as a transient environment variable for Copilot running in Docker sandbox.
See docs on Docker sandboxes: https://docs.docker.com/ai/sandboxes/get-started/

1. Create a PAT in your GitHub settings with the "Copilot Requests" permission.

2. Run this in PowerShell:
```powershell
# Encrypts the token and saves it as a DPAPI-protected file, bound to your Windows user account
$token = Read-Host -Prompt 'Enter your GitHub Copilot token' -AsSecureString
$plainBytes = [System.Text.Encoding]::UTF8.GetBytes(
    [System.Net.NetworkCredential]::new('', $token).Password
)
$encryptedBytes = [System.Security.Cryptography.ProtectedData]::Protect(
    $plainBytes,
    $null,
    [System.Security.Cryptography.DataProtectionScope]::CurrentUser
)
$storagePath = Join-Path $env:APPDATA 'gh_copilot_token.dpapi'
[System.IO.File]::WriteAllBytes($storagePath, $encryptedBytes)
Write-Host "Token encrypted and saved to: $storagePath"
```

3. Add this function to your PowerShell profile (you can access the file with the `$PROFILE` variable):
```powershell
function Invoke-CopilotSandbox {
    <#
    .SYNOPSIS
        Runs GitHub Copilot CLI inside a Docker sandbox.
        The GH_TOKEN is loaded from a DPAPI-encrypted file tied to your
        Windows user account. No third-party modules required.
    .PARAMETER SandboxArgs
        Any additional arguments passed through to `docker sandbox run`.
    .EXAMPLE
        copilot-sandbox ~/my-project
        copilot-sandbox ~/my-project --yolo
    #>
    param(
        [Parameter(ValueFromRemainingArguments = $true)]
        [string[]]$SandboxArgs
    )

    $storagePath = Join-Path $env:APPDATA 'gh_copilot_token.dpapi'
    $token = $null

    if (Test-Path $storagePath) {
        try {
            $encryptedBytes = [System.IO.File]::ReadAllBytes($storagePath)
            $plainBytes = [System.Security.Cryptography.ProtectedData]::Unprotect(
                $encryptedBytes,
                $null,
                [System.Security.Cryptography.DataProtectionScope]::CurrentUser
            )
            $token = [System.Text.Encoding]::UTF8.GetString($plainBytes)
        } catch {
            Write-Warning "Failed to decrypt token: $_"
        } finally {
            # Zero out the byte arrays from memory immediately after use
            if ($plainBytes) { [System.Array]::Clear($plainBytes, 0, $plainBytes.Length) }
            if ($encryptedBytes) { [System.Array]::Clear($encryptedBytes, 0, $encryptedBytes.Length) }
        }
    }

    # Fall back to interactive prompt if no encrypted file exists yet
    if (-not $token) {
        Write-Warning @"
No encrypted token found at: $storagePath
Store it once by running:

  `$token = Read-Host -Prompt 'Enter your GitHub Copilot token' -AsSecureString
  `$bytes  = [System.Text.Encoding]::UTF8.GetBytes([System.Net.NetworkCredential]::new('', `$token).Password)
  `$enc    = [System.Security.Cryptography.ProtectedData]::Protect(`$bytes, `$null, [System.Security.Cryptography.DataProtectionScope]::CurrentUser)
  [System.IO.File]::WriteAllBytes('$storagePath', `$enc)
"@
        $secureToken = Read-Host -Prompt 'Enter your GitHub Copilot token' -AsSecureString
        $token = [System.Net.NetworkCredential]::new('', $secureToken).Password
    }

    # Inject into the environment only for this process scope
    $env:GH_TOKEN = $token
    try {
        docker sandbox run copilot @SandboxArgs
    } finally {
        # Scrub from environment and memory the moment the sandbox exits
        Remove-Item Env:\GH_TOKEN -ErrorAction SilentlyContinue
        $token = $null
    }
}

Set-Alias copilot-sandbox Invoke-CopilotSandbox
```

4. Now just call `copilot-sandbox` in PowerShell any time you want a Docker sandbox running Copilot
