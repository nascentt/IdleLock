# IdleLock
Kiosk tool to perform an action after the host is idle for a specific period of time.  Defaults to logging off the current user, but can run any executable, lock the machine, or display an alert.  Best paired with ForceAutoLogon and Guest users.

![Screenshot](screenshot.png)

# Autohotkey
This is an AutoHotkey Script. To create an executable from it download AutoHotkey from autohotkey.com and use AHK2EXE to compile it into IdleLock.exe

# Settings
IdleLock allows for customization through an ini file "IdleLock.ini" in the program's directory.
You can customize the following:

; Default: 30 minutes idle timeout  
idleTimeout = 1800000  
; Msgbox timeout (how long to show msgbox for) - Default: 3 minutes msgbox timeout  
msgboxTimeout = 180  
; Action To Perform on idleTimeout - Default: Launch on timeout (SUpports Launch, LogOff, Lock, Alert)  
Action = LogOff  
; Enable Logging - Default: True  
EnableLogging = 1  
; Log Path - Default: log to C:\Temp  
LogPath = "C:\Logs\"  
; Application to launch (if launch action set) - Default: Notepad  
LogPath = "C:\Windows\Notepad.exe"  
; Repeat Action if remaining idle - Default: False  
Repeat = 0  
; Show a warning msgbox before executing action - Default: True  
ShowWarning = 1  
; Show a custom Alert msgbox  
AlertMessage = Warning`nHost was idle.  
; Instead of disabling by ini (which would mean the exe starts and closes) it'd really be better to have a gpo that disabled the scheduled task with item-level targetting, however this would mean splitting up the kiosk gpos to have idlelock separate.  
; Overriding disable mechanism. Allows us to disable and prevent IdleLock through ini configuration.  
DisableIdleLock = 0  
