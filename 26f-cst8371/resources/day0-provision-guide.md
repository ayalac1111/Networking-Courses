# Day0 provisioning guide

**Applies to every lab.** Each lab handout names the `lab` value to type (for example `l05`) and any step beyond Day0. Tool reference: [networking-tools](https://github.com/ayalac1111/networking-tools).

## Before you start

- A lab PC with PowerShell, Python, the BLUE NIC, a console cable, and the lab's pod hardware.
- The handout for your lab, open: it gives the `lab` value, the topology, and the port table.
- Your course username, assigned U number, CORE switch member number, and console COM port.

## Checklist

1. **Connect the PC to the course network.**

   - [ ] Cable the PC's **BLUE NIC** to the **white jack** in your pod.
   - [ ] In **T113**, the NIC uses DHCP. Run `ipconfig` and verify BLUE shows **203.0.113.2xx**.
   - [ ] In any other room, set the address manually: **203.0.113.2xx**, mask **255.255.255.0**, gateway **203.0.113.254**. `xx` is your two-digit station number (station 5 is `203.0.113.205`).

2. **Open PowerShell as administrator.**

   - [ ] Click **Start**, type **PowerShell**, right-click **Windows PowerShell**, select **Run as administrator**, and select **Yes**. The title must start with **Administrator: Windows PowerShell**.
   - [ ] Keep this window open for the whole Day0 procedure. Variables belong to the window.

3. **Download and extract the package.** The lab handout gives you this block with your lab value filled in. `l05` below is an example.

   - [ ] Download over the course network. Enter the server password `cisco` when prompted:

     ```powershell
     Set-Location "$env:USERPROFILE\Desktop"
     $lab = "l05"
     scp cisco@192.0.2.69:configs/$lab.zip "$env:USERPROFILE\Desktop\"
     ```

   - [ ] In File Explorer, right-click the `.zip` and select **Extract All...**. Choose your **Desktop**, remove the extra folder from the suggested path, and select **Extract**. If a folder named for the lab appears, open it and move its contents onto the Desktop.
   - [ ] Confirm the files sit directly on the Desktop with `templates` intact (`$lab` is your lab value):

     ```text
     Desktop/
       lab_setup.ps1
       day0_provision.py
       day0-$lab.yaml
       package-info.yaml
       templates/
     ```

   - [ ] Anything else the package holds (extra files, host setup, per-lab checks) is described in the lab handout.
   - [ ] If the package has no `lab_setup.ps1`, use **Manual setup** at the end of this guide for steps 3b and 4, then continue at step 5.

4. **Run the setup script.** It asks for your username, U number, CORE member number, and console port, checks each value against the lab's limits, confirms the package is complete, and creates your evidence files from the package templates.

   - [ ] Allow scripts in this window only. This changes nothing else on the PC:

     ```powershell
     Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force
     ```

   - [ ] Run the script with a **dot and a space** in front. The dot keeps your values in this window; without it they are lost when the script ends:

     ```powershell
     . .\lab_setup.ps1
     ```

   - [ ] Answer each prompt. A value outside the lab's allowed range (the handout states it) is rejected and asked again.
   - [ ] Read the status it prints: `SETUP OK` or `SETUP FAILED: missing or invalid ...`. `SETUP OK` means every device template, every evidence template, and every evidence file is present. Then read the values line (`lab=... username=... U=... coreMember=... port=...`). If the status is FAILED or a value is wrong, fix it and run the script again.
   - [ ] The script lists each evidence file as `created` or `kept`. It writes your lab and username into the header of each file, then moves the supplied `$lab-cNN.txt` templates into `templates\evidence`, so only your files stay on the Desktop. A rerun never overwrites an evidence file that exists, so your pasted output is safe. If your values were wrong, copy anything you pasted somewhere safe, delete the wrong `$lab-cNN-<username>.txt` files, and run the script again; it reads the originals from `templates\evidence`.
   - [ ] If it reports no templates, the lab handout says what to do.

5. **Cable the lab.**

   - [ ] Cable the topology exactly as the lab handout shows. Trace each cable and check both port labels against the handout's port table.
   - [ ] Leave the console cable free for step 7.

6. **Close PuTTY.**

   - [ ] One application per console port. Close PuTTY and any other program using the console adapter. Do not reopen it until Day0 finishes.
   - [ ] Find the adapter's COM port in Device Manager under Ports. It must match `$consolePort`.

7. **Provision every device the lab handout lists, one at a time.** The steps below use one device. Do them once per device, in the order the handout gives. `EDGE` is an example name.

   - [ ] Connect the console cable to the device.
   - [ ] Set the device name exactly as the handout names it, for example:

     ```powershell
     $device = "EDGE"
     ```

   - [ ] **[Optional]** Preview. A preview sends nothing:

     ```powershell
     python day0_provision.py --config day0-$lab.yaml --device $device --u $u --username $username --core-member $coreMember --dry-run
     ```

   - [ ] Push the configuration:

     ```powershell
     python day0_provision.py --config day0-$lab.yaml --device $device --u $u --username $username --core-member $coreMember --port $consolePort
     ```

   - [ ] Read the device's verification summary. Every line must show ✔, with one exception: the lab handout may list interface rows that can show down until the neighbouring device is provisioned. Such a row may show ✘ only while that neighbour is still unprovisioned, and only if the `got` line shows the correct address with a down status.
   - [ ] Stop for anything else: a configuration line the device rejected, a failed SSH check, a routing process the handout says must be absent, or a wrong address. Do not re-push and do not add routing. Keep the error output and raise your hand.
   - [ ] A down row that the handout does not list, or whose neighbour is already provisioned, is a cabling problem: fix the cabling and check again with the handout's read-only check, not with another push. If it stays down, raise your hand.
   - [ ] Move the console cable to the next device in the handout's list, change `$device`, and repeat from the preview. Stop when every listed device is provisioned.
   - [ ] **Final check, when the handout gives one.** After every device is provisioned, run the handout's read-only check on each device. Every line must now show ✔ with no exceptions. Never re-push Day0 to clear a link row.

## Manual setup

Use this only for a package that has no `lab_setup.ps1`. Keep the window from step 2 open and run the blocks in order.

**Step 3b — set your values.** Type the `lab` value exactly as the handout states:

```powershell
Set-Location "$env:USERPROFILE\Desktop"
$lab = (Read-Host "Lab package name, exactly as in the handout (example: l05)").Trim()
$username = (Read-Host "Course username").Trim()
$u = (Read-Host "Assigned U number").Trim()
$coreMember = (Read-Host "CORE switch member number").Trim()
$consolePort = (Read-Host "Console port, for example COM3").Trim()
"lab=$lab  username=$username  U=$u  coreMember=$coreMember  port=$consolePort"
```

Read the printed line. If a value is wrong, run the block again.

**Step 4 — create the evidence files, if the package has templates.** Reads each template from the Desktop or, on a rerun, from `templates\evidence`. It never overwrites an evidence file that exists, then moves any Desktop template into `templates\evidence`:

```powershell
$archive = "templates\evidence"
New-Item -ItemType Directory -Force $archive | Out-Null
$names = @(Get-ChildItem "$lab-c??.txt", "$archive\$lab-c??.txt" -ErrorAction SilentlyContinue |
    ForEach-Object { $_.Name } | Sort-Object -Unique)
foreach ($name in $names) {
    $evidenceFile = $name -replace '\.txt$', "-$username.txt"
    if (Test-Path $evidenceFile) {
        Write-Host "kept     $evidenceFile"
    } else {
        $source = if (Test-Path $name) { $name } else { "$archive\$name" }
        (Get-Content $source -Raw).
            Replace('{U}', "$u").
            Replace('{USERNAME}', $username) |
            Set-Content $evidenceFile -Encoding ascii
        Write-Host "created  $evidenceFile"
    }
    if (Test-Path $name) { Move-Item $name $archive -Force }
}
Get-ChildItem "$lab-c??-$username.txt" | Select-Object Name, Length
```

The last line lists your evidence files. The handout says how many to expect. This path does not check ranges or print `SETUP OK`; compare the printed values line with your assignment yourself.

## Success / Failure

Check each row before moving on. A failure on any row stops the procedure: fix the cause, then continue. The only exception is an interface row the handout lists as pending its neighbour, which may show down until that neighbour is provisioned. Never re-push Day0 or add routing to get past a failure.

| Item | Success | Failure |
|---|---|---|
| Package | `SETUP OK` after step 4. After Manual setup: the evidence listing shows one `$lab-cNN-<username>.txt` for every template, with the count the handout gives | `SETUP FAILED: missing or invalid ...`: files inside a subfolder, an incomplete package, or an evidence file with `{U}` or `{USERNAME}` still in it. Fix it and run the script again. After Manual setup: a file missing from the listing |
| Values | The printed line matches your assignment | A wrong lab, username, U, member, or port: run the script again |
| Each device | The verification summary shows ✔ on every line, except rows the handout lists as pending an unprovisioned neighbour | Any other ✘, or an interface reported down: fix the cabling and check again, or raise your hand |
| Final check | When the handout gives one, ✔ on every line for every device, with no exceptions | Any ✘ |

## Rules

- Omitting `--port` is a dry run. A preview sends nothing, and rendered configuration is not evidence.
- The bundled `day0_provision.py` supports `--core-member`. Do not substitute an older version.
- A COM-port serial push does not drive a CML console. CML needs its own console and reset procedure.
- Lab-specific verification tables and troubleshooting are in each handout.

## Troubleshooting

| Symptom | Check |
|---|---|
| Config or template not found | PowerShell is in the Desktop folder (`Set-Location "$env:USERPROFILE\Desktop"`); files sit directly on the Desktop, not in a subfolder |
| Unresolved variable | U, username, CORE member flag, and `$lab`; rerun `. .\lab_setup.ps1` and read the last line it prints |
| "Running scripts is disabled on this system" | Run `Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass -Force` in this window, then run the script again |
| `lab_setup.ps1` not found | PowerShell is in the Desktop folder; the file sits directly on the Desktop, not in a subfolder; if the package has no script, use Manual setup |
| Script says to run it with a dot, or your values are empty afterwards | Run it as `. .\lab_setup.ps1` (dot, space, then the path) |
| Serial port busy | Close PuTTY; confirm the COM port in Device Manager |
| Wrong physical interface | Stop; match the cabling to the handout's port table |
| Verify failure | Keep the failed check and the full output; raise your hand |
| Stale configuration from a previous session | Ask the instructor for a reset; do not proceed with stale state |
| Variable lost (new PowerShell window) | Open it as administrator, set `$lab = "l05"` (your lab value), and run `. .\lab_setup.ps1` again; it keeps existing evidence files |

Revision: October 2, 2026 — lab-independent; uses `$lab`, `$lab.zip`, and `lab_setup.ps1`.
