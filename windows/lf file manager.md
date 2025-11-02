Short answer: `lf` reads your `lfrc` from your XDG config location if available, and on Windows falls back to the usual AppData locations.

Details & practical paths

- Newer `lf` versions respect `XDG_CONFIG_HOME` on Windows, so it will look for:
    
    ```
    %XDG_CONFIG_HOME%\lf\lfrc
    ```
    
    (if `XDG_CONFIG_HOME` is unset, that typically defaults to `%USERPROFILE%\.config` → `%USERPROFILE%\.config\lf\lfrc`). ([GitHub](https://github.com/gokcehan/lf/releases?utm_source=chatgpt.com "Releases · gokcehan/lf"))
    
- Historically and as a Windows-specific fallback, `lf` also looks here (system/user):
    
    ```
    C:\ProgramData\lf\lfrc          (system-wide)
    C:\Users\<you>\AppData\Roaming\lf\lfrc   (user-specific)
    ```
    
    some docs and users mention `%LOCALAPPDATA%\lf\lfrc` (`C:\Users\<you>\AppData\Local\lf\lfrc`) as well — behavior varied between versions before XDG support was added. ([man.archlinux.org](https://man.archlinux.org/man/lf.1?utm_source=chatgpt.com "lf(1) — Arch manual pages"))
    

What I recommend

1. Easiest: put your file at `%APPDATA%\lf\lfrc` (that is `C:\Users\<you>\AppData\Roaming\lf\lfrc`) — many docs/examples expect that.
    
2. Or, set `XDG_CONFIG_HOME` and put `lfrc` at `%XDG_CONFIG_HOME%\lf\lfrc` (recommended if you share configs with Linux). Example in PowerShell:
    
    ```powershell
    # make xdg dir and copy example
    $env:XDG_CONFIG_HOME = "$env:USERPROFILE\.config"
    New-Item -ItemType Directory -Force "$env:XDG_CONFIG_HOME\lf"
    Copy-Item path\to\lfrc.example "$env:XDG_CONFIG_HOME\lf\lfrc"
    ```
    
3. If in doubt, check what your `lf` binary is actually reading by starting `lf -doc` or check its docs/manpage. ([GitHub](https://github.com/gokcehan/lf "GitHub - gokcehan/lf: Terminal file manager"))
    

If you want, tell me:

- which `lf` version you installed (`lf -version`) and whether you prefer `%APPDATA%` or `.config` style, and I’ll give the exact command(s) to create/copy the `lfrc` into the right place.