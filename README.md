FH6 Offline Hotkey Trainer
A lightweight, fully external hotkey trainer for Forza Horizon 6 (Steam, 64-bit). Press a hotkey while driving and get credits, wheelspins, skill points, a million XP, every car in the game for free, free upgrades, and absurd car physics — then restore everything back before you quit. No DLL injection, no save-file editing, no launcher wrappers: it just reads and writes the game's memory through the Win32 API while the game runs.
⚠️ Single-player / OFFLINE use ONLY. Never use this in any online or multiplayer mode. The trainer deliberately works through the game's local/offline data path (which is why it works at all), and online play is verified server-side. Keep your EULA clean and your bans non-existent: offline only.
Features
Key	Effect	Type
F1	Credits = 999,999,999	Toggle
F2	Wheelspins = 999	Toggle
F3	Super Wheelspins = 999	Toggle
F4	Skill Points = 999	Toggle
F5	Grant +1,000,000 XP (earn a little XP in-game once first)	One-shot
F6	Unlock Everything — all cars free ($0), every car visible in the Autoshow, install/purchase/drivable flags set for all cars, all cars added to your garage buckets	One-shot
F7	Free Upgrades — price = 0 on all upgrade tables (engine, turbo, tires, brakes, aero…)	One-shot
F8	Max Speed physics — torque ×2, drag ×0.5, road grip ×3 (+ wet/off-road tire friction ×1.5)	Toggle
F9	No Skill Break — skill chains never end	Toggle
F10	Drift Score ×10	Toggle
F11	Sell Payout ×10 — cars pay 10× when sold	Toggle
F12	Freeze AI — AI/traffic cars stop updating their speed	Toggle (experimental)
HOME	Autoshow Unlock (showroom visibility only)	One-shot
END	Revert all SQL features (car prices, autoshow visibility, install flags, physics…)	Revert
DEL	Exit the trainer (restores all memory hooks first)	Exit
For the value toggles (F1–F4, F9–F12): while a cheat is ON, whenever the game writes that counter, the trainer's configured value is written instead — the counter stays pinned until you toggle it OFF. When you toggle OFF, the game just goes back to writing its own values (whatever the counter currently holds stays).
How it works
For the curious (and the paranoid — the full C# source is included in the download):
External, zero injection. A plain console app written in C# (.NET Framework 4.8, pre-compiled x64). It attaches to 
ForzaHorizon6.exe
 with 
OpenProcess
 (after enabling 
SeDebugPrivilege
), and touches the game only through 
ReadProcessMemory
 / 
WriteProcessMemory
 / 
VirtualProtectEx
. No DLLs are injected, no game files are modified.
AOB signature scanning, not fixed addresses. Features are located by Array-of-Bytes patterns (
?
 = wildcard) scanned inside the game's main module, so the trainer works around ASLR and across sessions. Before patching anything, the trainer validates the actual bytes it found against the expected instruction bytes — if a game update changes a function, the trainer refuses to patch garbage and tells you instead.
Detour hooks ("code caves"). For each profile cheat (F1–F4, F9–F12), the trainer allocates executable memory near the target function, copies a small asm stub into it and redirects the function through 
E9
 relative jumps. The stub checks a one-byte ON/OFF flag plus a 32-bit value that the trainer writes — toggling a hotkey literally just flips that flag byte and updates the value. Targets are pre-resolved before any hook is installed, because some of FH6's value setters (wheelspins/skill points) share functions and are only a dozen bytes apart.
CRC self-check bypass. FH6 integrity-checks its own executable code at runtime. The trainer resolves the game's CRC-check function pointer and redirects it to an immediate 
ret
 stub. Every 10 seconds it briefly restores the original bytes and pointer, lets the game's check run against pristine code, then re-applies the patches — this is what keeps hooks stable long-session without crashing.
XP grant (F5). The trainer hooks the 
AddTotalXP
 function and captures the player "owner" object pointer the first time the game legitimately awards you XP (a skill chain, smashing a board, a drift…). Pressing F5 then allocates a tiny bit of shellcode in the game process and calls 
CreateRemoteThread
 to invoke 
AddTotalXP(owner, 1,000,000)
. That's why F5 needs one real XP gain first — the pointer has to have been captured.
Database features (F6, F7, F8, HOME, END). FH6's car catalog / profile data lives in an SQLite-style database in the game's memory. The trainer resolves the database object and its 
ExecuteQuery
 entry through signatures, then executes the cheat SQL in-process via a remote thread. Every modified table is first copied to a 
_backup_*
 shadow table, so END can restore the exact original values.
Clean exit. Pressing DEL (or closing via the console) restores every patched byte to its original state, frees the allocated cave memory, restores the CRC pointer, and detaches from the process. Nothing lingers after the trainer closes.
(Console log messages are currently in pt-BR — the feature names and hotkeys are universal.)
Requirements
Windows 10 / 11, 64-bit
.NET Framework 4.8 (preinstalled on Windows 10 20H2+ and Windows 11 — nothing to download)
Forza Horizon 6 (Steam)
Administrator rights — required so the trainer can open the game process and write its memory. This is why UAC prompts you.
Installation
Extract the archive anywhere you like — Desktop, 
C:\Tools\
, wherever. It does not need to go into the game folder.
Run either one of these:
Double-click 
Launch FH6 Trainer.bat
 (recommended — it handles elevation automatically), or
Right-click 
FH6Trainer.exe
 → Run as administrator.
Accept the UAC prompt. A console window opens: FH6 Offline Hotkey Trainer.
That's it. The trainer can be launched before or after the game — a watcher thread scans for 
ForzaHorizon6
 every 2 seconds and attaches automatically, printing 
FORZA HORIZON 6 detectado!
 when it connects.
Building from source instead (optional — the 
FH6Trainer.cs
 source file is included):
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /nologo /optimize /platform:x64 /target:exe /out:FH6Trainer.exe /win32manifest:app.manifest FH6Trainer.cs
How to use
Start Forza Horizon 6 and load into the world — be actually driving, not sitting in a menu.
Launch the trainer (see Installation). Wait for the 
detectado!
 message.
Keep the game in the foreground. The trainer only reads hotkeys while the FH6 window is the focused window — you can safely alt-tab to browse, type, or stream without accidentally toggling cheats.
Press a hotkey and watch the status block in the console update to 
[ON]
 / 
[off]
.
For F1–F4: the configured number is applied the next time the game updates that counter — start/finish a race, spend or receive credits, buy a spin, etc. While the cheat stays ON, the counter is pinned to the configured value.
F5 (
Grant XP
): if you get the "Owner XP not captured" message, just earn a bit of XP in-game first (drift, skill chain, smash boards) then press F5 again. Each press = +1,000,000 XP. Press multiple times to stack.
F6 (
Unlock Everything
): run it while loaded in the world, then check the Autoshow / your garage — you may need to back out to a menu once for the catalog to refresh.
F7 (
Free Upgrades
): applies to all upgrade shop tables; the next upgrade you install costs 0.
F8 (
Max Speed physics
): first press applies, second press reverts.
Before switching cars, saving state you care about, or exiting the game: press END to restore every database value from its backup, and DEL to cleanly exit the trainer (while the game is focused — otherwise just close the console window).
⚠️ Important warning about F8 (physics)
If the game performs an autosave while F8 is active, the modified physics values can be baked into the car's save data (your suspension/handling stays absurd after the trainer is gone). Make it a habit: F8 ON → drive → END → then save/quit. The standalone END revert restores from the trainer's own backup tables, so values snap back to exactly what they were before you enabled it.
Recommendations
Back up your save before your first session, same advice as for any trainer.
Enable cheats only for the session you want them, END before quitting — you keep full control of what persists in your profile.
Troubleshooting
Message in the console	What it means / what to do

assinatura não encontrada nesta build
	FH6 was updated and a signature no longer matches. The trainer intentionally refuses to patch unknown code. Wait for a trainer update (or update the signatures yourself in the source).

bytes não correspondem
	Same cause as above, or another trainer/tool has already patched the same function. Close the other tool and re-attach.

Owner XP não capturado
	Earn a little XP in-game first (skill chain, drift, smash a board), then press F5 again.

Falha ao ler o módulo principal
	The trainer isn't elevated. Run as Administrator.

alvo já patched por outra tool
	Another trainer already has hooks on the same code region.
Game crash while driving	Most commonly caused by F8 physics values on certain cars. Press END to revert physics and toggle F8 off first next time. Some occasional crashes are also known to happen in offline mode independent of the trainer.
Other common questions:
Nothing happens when I press hotkeys → the game window isn't focused. Hotkeys only fire with FH6 in the foreground.
UAC prompt appears every launch → normal; the 
.bat
 uses PowerShell 
Start-Process -Verb RunAs
, which elevates each time.
My antivirus flagged it → see below.
Anti-virus false positives
Any tool that opens a game process and writes its memory is designed to behave like what heuristic AV scanners flag. This trainer is open source (GPL-3.0) and ships as source as well as binary — if you'd rather not trust the pre-compiled exe, compile it yourself with the command in the Installation section. The only Win32 APIs it uses are the classic memory-reading ones; there's no network code, no persistence, no Registry writes.
