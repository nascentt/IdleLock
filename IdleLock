#NoTrayIcon
#singleinstance force
#Persistent
#InstallKeybdHook
#InstallMouseHook

StartTime := A_TickCount

;Read settings from IdleLock.ini if exists
;IniRead, OutputVar, Filename, Section, Key , Default
IniRead, idleTimeout, IdleLock.ini, Settings, idleTimeout, 1800000 ; Default: 30 minutes idle timeout
IniRead, msgboxTimeout, IdleLock.ini, Settings, msgboxTimeout, 180 ; Msgbox timeout (how long to show msgbox for) - Default: 3 minutes msgbox timeout
IniRead, actionType, IdleLock.ini, Settings, Action, LogOff ; Action To Perform on idleTimeout - Default: Launch on timeout
IniRead, enableLogging, IdleLock.ini, Settings, EnableLogging, 1 ; Enable Logging - Default: True
IniRead, logPath, IdleLock.ini, Settings, LogPath, "C:\Logs\" ; Log Path - Default: log to C:\Temp
IniRead, launchPath, IdleLock.ini, Settings, LogPath, "C:\Windows\Notepad.exe" ; Application to launch (if launch action set) - Default: Notepad
IniRead, repeat, IdleLock.ini, Settings, Repeat, 0 ; Repeat Action if remaining idle - Default: False
IniRead, showWarning, IdleLock.ini, Settings, ShowWarning, 1 ; Show a warning msgbox before executing action - Default: True
IniRead, alertMessage, IdleLock.ini, Settings, AlertMessage, Warning`nHost was idle. ; Show a custom Alert msgbox
;Instead of disabling by ini (which would mean the exe starts and closes) it'd really be better to have a gpo that disabled the scheduled task with item-level targetting, however this would mean splitting up the kiosk gpos to have idlelock separate.
IniRead, disableIdleLock, IdleLock.ini, Settings, DisableIdleLock, 0 ; Overriding disable mechanism. Allows us to disable and prevent IdleLock through ini configuration.

;Convert string to boolean as autohotkey sucks at interpreting booleans from iniRead
if (enableLogging = "true") {
	enableLogging := True
}
else if (enableLogging = "False") {
	enableLogging := False
}
if (showWarning = "true") {
	showWarning := True
}
else if (showWarning = "False") {
	showWarning := False
}
if (repeat = "true") {
	repeat := true
}
else if (repeat = "False") {
	repeat := False
}
if (disableIdleLock = "true") {
	disableIdleLock := true
}
else if (disableIdleLock = "False") {
	disableIdleLock := False
}

;remove quotes
StringReplace, logPath, logPath, ", , All ;" - semi-colon + quote suffix fixes syntax highlighting

if disableIdleLock {
	FormatTime, TimeString
	if enableLogging
		FileAppend, [%TimeString%] exit: disableIdleLock (%disableIdleLock%) - Exiting`n, %logPath%\idlelock.log
	ExitApp
}

minutes:= % Round(idleTimeout/1000/60)
msgboxTimeoutMinutes:= % Round(msgboxTimeout/60)

SetTimer,  CloseOnIdle, % idleTimeout

CloseOnIdle:
	FormatTime, TimeString
	processRuntime := (A_TickCount - StartTime) ;ticker in milliseconds since process started
	if enableLogging {
		FileAppend, [%TimeString%] timer: processRuntime (%processRuntime%) idleCounter (%A_TimeIdle%) idleTimeout (%idleTimeout%)`n, %logPath%\idlelock.log
		FileAppend,  A_TimeIdle ( %A_TimeIdle%)  A_TimeIdlePhysical (%A_TimeIdlePhysical%) A_TimeIdleKeyboard (%A_TimeIdleKeyboard%), A_TimeIdleMouse (%A_TimeIdleMouse%)`n, %logPath%\idlelock.log
	}
	
	;if idle for longer than the process has been running - means the host already auto logged out, so no need to logout again yet

	if (processRuntime < A_TimeIdle) {
		if enableLogging
			FileAppend, [%TimeString%] wait: idleCounter (%A_TimeIdle%) more than processRuntime (%processRuntime%) - not executing action yet (kiosk hasn't been used)`n, %logPath%\idlelock.log
		SetTimer,CloseOnIdle, % idleTimeout - A_TimeIdle ; set new timer in future
		return
	}

	if (A_TimeIdle>=idleTimeout) {
			FormatTime, TimeString
			if enableLogging
				FileAppend, [%TimeString%] idleCounter (%A_TimeIdle%) more than idleTimeout (%idleTimeout%)`n, %logPath%\idlelock.log
		if (A_TimeIdle < processRuntime) { ;no point logging off if not touched since last logoff
			if enableLogging
				FileAppend, [%TimeString%] idleCounter (%A_TimeIdle%) less than processRuntime (%processRuntime%)`n, %logPath%\idlelock.log
			if (showWarning) {
				FormatTime, TimeString
				if enableLogging
					FileAppend, [%TimeString%] notify: processRuntime: %processRuntime% idleCounter: %A_TimeIdle%`n, %logPath%\idlelock.log
				if (actionType = "Lock"){
					actionMessage = The session will lock in %msgboxTimeoutMinutes% minutes.
				}
				else if (actionType = "Logoff"){
					actionMessage = The session will be logged off in %msgboxTimeoutMinutes% minutes.
				}
				else if (actionType = "Launch"){
					actionMessage = The application (%launchPath%) will be launched in %msgboxTimeoutMinutes% minutes.
				}
				else if (actionType = "Alert"){
					actionMessage = The Alert will execute in %msgboxTimeoutMinutes% minutes.
				}

				;msgboxType = 32 ; Question mark and OK button
				msgboxType = 4128 ; Question mark and OK button System Modal
				
				msgboxMessage = Computer has been idle for %minutes% minutes.`n%actionMessage%`n`nWould you like to postpone and continue using this session?
				MsgBox, 4128, Host Idle, %msgboxMessage%, %msgboxTimeOut%
				IfMsgBox OK
					FormatTime, TimeString
				IfMsgBox OK
					FileAppend, [%TimeString%] ok: action prevented - user interaction`n, %logPath%\idlelock.log
				IfMsgBox OK
					return
			}

			FormatTime, TimeString
			if enableLogging
				FileAppend, [%TimeString%] action: Host idle - Performing action %actionType%`n, %logPath%\idlelock.log

			if (actionType = "Launch"){
				;Run C:\Windows\Notepad.exe
				Run %launchPath%
			}
			else if (actionType = "Lock"){
				DllCall("LockWorkStation")
			}
			else if (actionType = "Logoff"){
				;shutdown, 0 ;logs off user
				shutdown, 4 ;logs off user forcibly
			}
			else if (actionType = "Alert"){
				msgbox, 4144, Alert, %alertMessage% ; 4144 = Exclamation and System Modal
			}

			if (repeat = False) {
				SetTimer,CloseOnIdle, Off
				ExitApp
			}
		}
		else {
			FormatTime, TimeString
			if enableLogging
				FileAppend, [%TimeString%] wait: idleCounter (%A_TimeIdle%) more than processRuntime (%processRuntime%)`n, %logPath%\idlelock.log
		}
	}
	else {
		SetTimer,CloseOnIdle, % idleTimeout - A_TimeIdle
	}

return
