SAMT Desktop release assets

macOS Apple Silicon:
  Open SAMT-Desktop-macOS-arm64.dmg and drag SAMT-Desktop.app
  to Applications before launching it.

Windows x64:
  Extract SAMT-Desktop-Windows-x64.zip and run
  SAMT-Desktop/SAMT-Desktop.exe.

Files ending in .partNN are 1500 MiB parts of one Windows ZIP.
Rejoin them in filename order before unzipping.

macOS / Linux:
  cat SAMT-Desktop-Windows-x64.zip.part* > SAMT-Desktop-Windows-x64.zip

Windows PowerShell:
  $parts = Get-ChildItem -File SAMT-Desktop-Windows-x64.zip.part* | Sort-Object Name
  $out = [System.IO.File]::Create('SAMT-Desktop-Windows-x64.zip')
  foreach ($part in $parts) {
    $in = [System.IO.File]::OpenRead($part.FullName)
    $in.CopyTo($out)
    $in.Dispose()
  }
  $out.Dispose()

Verify downloads against SHA256SUMS.txt before use.
