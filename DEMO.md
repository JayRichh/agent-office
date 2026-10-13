# Running the demo on Windows 11

A local, single-user demo run from this clone. No installer, no `irm | iex`.
Needs Node 20+ (tested on Node 25.1, npm 11.6). The repo uses npm (`package-lock.json`), not pnpm.

## One-time setup

```powershell
cd C:\Code\agent-office
npm install            # installs deps and builds dist/ (the `prepare` script)
```

## Start

Data goes to a separate demo home so it never touches `~/agent-office`:

```powershell
$h = 'C:\Code\agent-office-demo-home'
New-Item -ItemType Directory -Force $h | Out-Null
node C:\Code\agent-office\bin\agent-office.js --home $h --projects $h --port 4600 --password demo --no-open
```

Open http://localhost:4600 and sign in with the password `demo`.
The 2D view is http://localhost:4600/lite.

To run it hidden in the background instead:

```powershell
Start-Process node -ArgumentList 'bin/agent-office.js','--home',$h,'--projects',$h,'--port','4600','--password','demo','--no-open' `
  -WorkingDirectory C:\Code\agent-office -WindowStyle Hidden `
  -RedirectStandardOutput "$h\server.out.log" -RedirectStandardError "$h\server.err.log"
```

Started from a non-interactive shell, the first-run walkthrough (projects folder, `gh auth login`,
first repo) is skipped. Add a floor from the elevator in the office instead.

## Check it from a terminal

```powershell
curl.exe -s http://127.0.0.1:4600/api/health                                  # {"ok":true}
curl.exe -s -c $env:TEMP\ao.cj -H "content-type: application/json" -d '{\"password\":\"demo\"}' http://127.0.0.1:4600/api/login
curl.exe -s -b $env:TEMP\ao.cj -o NUL -w "%{http_code}`n" http://127.0.0.1:4600/lite   # 200
```

Without the cookie, `/` and `/lite` answer 302 to `/login`.

## Stop

In the foreground: Ctrl+C. In the background, stop every node process started from this clone
(the office, plus the terminal host it starts once a worker is hired):

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'node.exe'" |
  Where-Object { ($_.CommandLine -replace '/', '\') -like '*C:\Code\agent-office\*' } |
  ForEach-Object { Stop-Process -Id $_.ProcessId -Force }
```

## Reset

Delete `C:\Code\agent-office-demo-home` while the office is stopped.

## Note on quota

Hiring a worker at a desk starts a real `claude` session on the signed-in plan. For a look around
without spending quota, don't hire workers.
