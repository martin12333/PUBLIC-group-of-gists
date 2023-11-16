Send - Syntax & Usage | AutoHotkey v1 

Send / SendRaw / SendInput / SendPlay / SendEvent
=================================================

Sends simulated keystrokes and mouse clicks to the [active](WinActivate.htm) window.

Send Keys
SendRaw Keys
SendInput Keys
SendPlay Keys
SendEvent Keys

Parameters
----------

Keys

The sequence of keys to send. As with other commands, the comma in front of the first parameter is optional.

By default (that is, if neither SendRaw nor the [Raw mode](#Raw) or [Text mode](#Text) is used), the characters `^+!#{}` have a special meaning. The characters `^+!#` represent the modifier keys Ctrl, Shift, Alt and Win. They affect only the very next key. To send the corresponding modifier key on its own, enclose the key name in braces. To just press (hold down) or release the key, follow the key name with the word "down" or "up" as shown below.

#modifierkeys td:not(:last-child) { white-space: nowrap; text-align: center }

| Symbol | Key | Press | Release | Examples |
| --- | --- | --- | --- | --- |
| ^   | {Ctrl} | {Ctrl down} | {Ctrl up} | `Send ^{Home}` presses Ctrl+Home |
| +   | {Shift} | {Shift down} | {Shift up} | `Send +abC` sends the text "AbC"  <br>`Send !+a` presses Alt+Shift+A |
| !   | {Alt} | {Alt down} | {Alt up} | `Send !a` presses Alt+A |
| #   | {LWin}  <br>{RWin} | {LWin down}  <br>{RWin down} | {LWin up}  <br>{RWin up} | `Send #e` holds down Win and presses E |

**Note:** As capital letters are produced by sending Shift, `A` produces a different effect in some programs than `a`. For example, `!A` presses Alt+Shift+A and `!a` presses Alt+A. If in doubt, use lowercase.

The characters `{}` are used to enclose [key names and other options](#keynames), and to send special characters literally. For example, `{Tab}` is Tab and `{!}` is a literal exclamation mark.

\[v1.1.27+\]: Enclosing a plain ASCII letter (a-z or A-Z) in braces forces it to be sent as the corresponding virtual keycode, even if the character does not exist on the current keyboard layout. In other words, `Send a` produces the letter "a" while `Send {a}` may or may not produce "a", depending on the keyboard layout. For details, see the [remarks below](#AZ).

Send variants
-------------

**Send:** By default, Send is synonymous with SendEvent; but it can be made a synonym for SendInput or SendPlay via [SendMode](SendMode.htm).

**SendRaw:** Similar to Send, except that all characters in _Keys_ are interpreted and sent literally. See [Raw mode](#Raw) for details.

**SendInput** and **SendPlay** \[v1.0.43+\]: SendInput and SendPlay use the same syntax as Send but are generally faster and more reliable. In addition, they buffer any physical keyboard or mouse activity during the send, which prevents the user's keystrokes from being interspersed with those being sent. [SendMode](SendMode.htm) can be used to make Send synonymous with SendInput or SendPlay. For more details about each mode, see [SendInput](#SendInputDetail) and [SendPlay](#SendPlayDetail) below.

**SendEvent** \[v1.0.43+\]: SendEvent sends keystrokes using the same method as the pre-1.0.43 _Send_ command. The rate at which keystrokes are sent is determined by [SetKeyDelay](SetKeyDelay.htm).

Special modes
-------------

The following modes affect the interpretation of the characters in _Keys_ or the behavior of key-sending commands such as Send, SendInput, SendPlay, SendEvent and [ControlSend](ControlSend.htm). These modes must be specified as `{x}` in _Keys_, where x is either Raw, Text, or Blind. For example, `{Raw}`.

### Raw mode

The Raw mode can be either enabled with `{Raw}`, SendRaw or [ControlSendRaw](ControlSend.htm), which causes all subsequent characters, including the special characters `^+!#{}`, to be interpreted literally rather than translating `{Enter}` to Enter, `^c` to Ctrl+C, etc. For example, both `Send {Raw}{Tab}` and `SendRaw {Tab}` send `{Tab}` instead of Tab.

The Raw mode does not affect the interpretation of [escape sequences](../misc/EscapeChar.htm), [variable references](../Variables.htm#retrieving) and [expressions](../Variables.htm#Expressions). For example, ```SendRaw ``100`%``` sends the string `` `100%``. When using [ControlSend](ControlSend.htm), it is also necessary to escape literal commas (`` `,``).

### Text mode \[v1.1.27+\]

The Text mode can be enabled with `{Text}`, which is similar to the Raw mode, except that no attempt is made to translate characters (other than `` `r``, `` `n``, `` `t`` and `` `b``) to keycodes; instead, the [fallback method](#fallback) is used for all of the remaining characters. For SendEvent, SendInput and [ControlSend](ControlSend.htm), this improves reliability because the characters are much less dependent on correct modifier state. This mode can be combined with the Blind mode to avoid releasing any modifier keys: `Send {Blind}{Text}your text`. However, some applications require that the modifier keys be released.

`` `n``, `` `r`` and `` `r`n`` are all translated to a single Enter, unlike the default behavior and Raw mode, which translate `` `r`n`` to two Enter. `` `t`` is translated to Tab and `` `b`` to Backspace, but all other characters are sent without translation.

\[v1.1.29+\]: Like the Blind mode, the Text mode ignores [SetStoreCapsLockMode](SetStoreCapsLockMode.htm) (that is, the state of CapsLock is not changed) and does not [wait for Win to be released](../Hotkeys.htm#win-l). This is because the Text mode typically does not depend on the state of CapsLock and cannot trigger the system Win+L hotkey. However, this only applies when _Keys_ begins with `{Text}` or `{Blind}{Text}`.

### Blind mode

The Blind mode can be enabled with `{Blind}`, which gives the script more control by disabling a number of things that are normally done automatically to make things work as expected. `{Blind}` must be the first item in the string to enable the Blind mode. It has the following effects:

* The Blind mode avoids releasing the modifier keys (Alt, Ctrl, Shift, and Win) if they started out in the down position. For example, the hotkey `+s::Send {Blind}abc` would send ABC rather than abc because the user is holding down Shift.
* Modifier keys are restored differently to allow a Send to turn off a hotkey's modifiers even if the user is still physically holding them down. For example, `^space::Send {Ctrl up}` automatically pushes Ctrl back down if the user is still physically holding Ctrl, whereas `^space::Send {Blind}{Ctrl up}` allows Ctrl to be logically up even though it is physically down.
* [SetStoreCapsLockMode](SetStoreCapsLockMode.htm) is ignored; that is, the state of CapsLock is not changed.
* [Menu masking](_MenuMaskKey.htm) is disabled. That is, Send omits the extra keystrokes that would otherwise be sent in order to prevent: 1) Start Menu appearance during Win keystrokes (LWin/RWin); 2) menu bar activation during Alt keystrokes. However, the Blind mode does not prevent masking performed by the keyboard hook following activation of a hook hotkey.
* Send does not wait for Win to be released even if the text contains an L keystroke. This would normally be done to prevent Send from triggering the system "lock workstation" hotkey (Win+L). See [Hotkeys](../Hotkeys.htm#win-l) for details.

The Blind mode is used internally when [remapping a key](../misc/Remap.htm). For example, the remapping `a::b` would produce: 1) "b" when you type "a"; 2) uppercase "B" when you type uppercase "A"; and 3) Ctrl+B when you type Ctrl+A.

`{Blind}` is not supported by SendRaw or [ControlSendRaw](ControlSend.htm); use `{Blind}{Raw}` instead.

The Blind mode is not completely supported by [SendPlay](#SendPlayDetail), especially when dealing with the modifier keys (Ctrl, Alt, Shift, and Win).

Key names
---------

The following table lists the special keys that can be sent (each key name must be enclosed in braces):

| Key name | Description |
| --- | --- |
| {F1} - {F24} | Function keys. For example: {F12} is F12. |
| {!} | !   |
| {#} | #   |
| {+} | +   |
| {^} | ^   |
| {{} | {   |
| {}} | }   |
| {Enter} | Enter on the main keyboard |
| {Escape} or {Esc} | Esc |
| {Space} | Space (this is only needed for spaces that appear either at the beginning or the end of the string to be sent -- ones in the middle can be literal spaces) |
| {Tab} | Tab |
| {Backspace} or {BS} | Backspace |
| {Delete} or {Del} | Del |
| {Insert} or {Ins} | Ins |
| {Up} | ↑ (up arrow) on main keyboard |
| {Down} | ↓ (down arrow) on main keyboard |
| {Left} | ← (left arrow) on main keyboard |
| {Right} | → (right arrow) on main keyboard |
| {Home} | Home on main keyboard |
| {End} | End on main keyboard |
| {PgUp} | PgUp on main keyboard |
| {PgDn} | PgDn on main keyboard |
| {CapsLock} | CapsLock (using [SetCapsLockState](SetNumScrollCapsLockState.htm) is more reliable on Win 2k/XP). Sending {CapsLock} might require `[SetStoreCapsLockMode](SetStoreCapsLockMode.htm) Off` beforehand. |
| {ScrollLock} | ScrollLock (see also: [SetScrollLockState](SetNumScrollCapsLockState.htm)) |
| {NumLock} | NumLock (see also: [SetNumLockState](SetNumScrollCapsLockState.htm)) |
| {Control} or {Ctrl} | Ctrl (technical info: sends the neutral virtual key but the left scan code) |
| {LControl} or {LCtrl} | Left Ctrl (technical info: sends the left virtual key rather than the neutral one) |
| {RControl} or {RCtrl} | Right Ctrl |
| {Control down} or {Ctrl down} | Holds Ctrl down until {Ctrl up} is sent. To hold down the left or right key instead, replace Ctrl with LCtrl or RCtrl. |
| {Alt} | Alt (technical info: sends the neutral virtual key but the left scan code) |
| {LAlt} | Left Alt (technical info: sends the left virtual key rather than the neutral one) |
| {RAlt} | Right Alt (or AltGr, depending on keyboard layout) |
| {Alt down} | Holds Alt down until {Alt up} is sent. To hold down the left or right key instead, replace Alt with LAlt or RAlt. |
| {Shift} | Shift (technical info: sends the neutral virtual key but the left scan code) |
| {LShift} | Left Shift (technical info: sends the left virtual key rather than the neutral one) |
| {RShift} | Right Shift |
| {Shift down} | Holds Shift down until {Shift up} is sent. To hold down the left or right key instead, replace Shift with LShift or RShift. |
| {LWin} | Left Win |
| {RWin} | Right Win |
| {LWin down} | Holds the left Win down until {LWin up} is sent |
| {RWin down} | Holds the right Win down until {RWin up} is sent |
| {AppsKey} | Menu (invokes the right-click or context menu) |
| {Sleep} | Sleep |
| {ASC nnnnn} | Sends an Alt+nnnnn keypad combination, which can be used to generate special characters that don't exist on the keyboard. To generate ASCII characters, specify a number between 1 and 255. To generate ANSI characters (standard in most languages), specify a number between 128 and 255, but precede it with a leading zero, e.g. {Asc 0133}.<br><br>Unicode characters may be generated by specifying a number between 256 and 65535 (without a leading zero). However, this is not supported by all applications. For alternatives, see the section below. |
| {U+nnnn} | \[AHK_L 24+\]: Sends a Unicode character where _nnnn_ is the hexadecimal value of the character excluding the 0x prefix. This typically isn't needed in Unicode versions of AutoHotkey, where Send and ControlSend automatically support Unicode text.<br><br>[SendInput()](https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-sendinput) or [WM_CHAR](https://learn.microsoft.com/windows/win32/inputdev/wm-char) is used to send the character and the current Send mode has no effect. Characters sent this way usually do not trigger shortcut keys or hotkeys. |
| {vkXX}  <br>{scYYY}  <br>{vkXXscYYY} | Sends a keystroke that has virtual key XX and scan code YYY. For example: `Send {vkFFsc159}`. If the sc or vk portion is omitted, the most appropriate value is sent in its place.<br><br>The values for XX and YYY are hexadecimal and can usually be determined from the [main window](../Program.htm#main-window)'s View->[Key history](KeyHistory.htm) menu item. See also: [Special Keys](../KeyList.htm#SpecialKeys)<br><br>**Warning:** Combining vk and sc in this manner is valid only with Send. Prior to \[v1.1.27\], hotkeys permitted but ignored any non-hexadecimal characters following XX. |
| {Numpad0} - {Numpad9} | Numpad digit keys (as seen when NumLock is ON). For example: {Numpad5} is 5. |
| {NumpadDot} | . (numpad period) (as seen when NumLock is ON). |
| {NumpadEnter} | Enter on keypad |
| {NumpadMult} | * (numpad multiplication) |
| {NumpadDiv} | / (numpad division) |
| {NumpadAdd} | + (numpad addition) |
| {NumpadSub} | - (numpad subtraction) |
| {NumpadDel} | Del on keypad (this key and the following Numpad keys are used when NumLock is OFF) |
| {NumpadIns} | Ins on keypad |
| {NumpadClear} | Clear key on keypad (usually 5 when NumLock is OFF). |
| {NumpadUp} | ↑ (up arrow) on keypad |
| {NumpadDown} | ↓ (down arrow) on keypad |
| {NumpadLeft} | ← (left arrow) on keypad |
| {NumpadRight} | → (right arrow) on keypad |
| {NumpadHome} | Home on keypad |
| {NumpadEnd} | End on keypad |
| {NumpadPgUp} | PgUp on keypad |
| {NumpadPgDn} | PgDn on keypad |
| {Browser_Back} | Select the browser "back" button |
| {Browser_Forward} | Select the browser "forward" button |
| {Browser_Refresh} | Select the browser "refresh" button |
| {Browser_Stop} | Select the browser "stop" button |
| {Browser_Search} | Select the browser "search" button |
| {Browser_Favorites} | Select the browser "favorites" button |
| {Browser_Home} | Launch the browser and go to the home page |
| {Volume_Mute} | Mute/unmute the master volume. Usually equivalent to `[SoundSet](SoundSet.htm), +1, , mute`. |
| {Volume_Down} | Reduce the master volume. Usually equivalent to `[SoundSet](SoundSet.htm) -5`. |
| {Volume_Up} | Increase the master volume. Usually equivalent to `[SoundSet](SoundSet.htm) +5`. |
| {Media_Next} | Select next track in media player |
| {Media_Prev} | Select previous track in media player |
| {Media_Stop} | Stop media player |
| {Media\_Play\_Pause} | Play/pause media player |
| {Launch_Mail} | Launch the email application |
| {Launch_Media} | Launch media player |
| {Launch_App1} | Launch user app1 |
| {Launch_App2} | Launch user app2 |
| {PrintScreen} | PrtSc |
| {CtrlBreak} | Ctrl+Pause |
| {Pause} | Pause |
| {Click \[Options\]}  <br>\[v1.0.43+\] | Sends a mouse click using the same options available in the [Click command](Click.htm). For example, `Send {Click}` would click the left mouse button once at the mouse cursor's current position, and `Send {Click 100 200}` would click at coordinates 100, 200 (based on [CoordMode](CoordMode.htm)). To move the mouse without clicking, specify 0 after the coordinates; for example: `Send {Click 100 200 0}`. The delay between mouse clicks is determined by [SetMouseDelay](SetMouseDelay.htm) (not [SetKeyDelay](SetKeyDelay.htm)). |
| {WheelDown}, {WheelUp}, {WheelLeft}, {WheelRight}, {LButton}, {RButton}, {MButton}, {XButton1}, {XButton2} | Sends a mouse button event at the cursor's current position (to have control over position and other options, use [{Click}](Click.htm) above). The delay between mouse clicks is determined by [SetMouseDelay](SetMouseDelay.htm). WheelLeft/Right require \[v1.0.48+\], but have no effect on operating systems older than Windows Vista.<br><br>LButton and RButton correspond to the "physical" left and right buttons when used with Send, but the "logical" left and right buttons when used with hotkeys. In other words, if the user has swapped the buttons via system settings, `{LButton}` performs a logical right click, but a physical left click activates the `RButton::` hotkey. Likewise for `{RButton}` and `LButton::`. To always perform a logical click, use [{Click}](Click.htm) instead. |
| {Blind} | Enables the [Blind mode](#blind), which gives the script more control by disabling a number of things that are normally done automatically to make things generally work as expected. `{Blind}` must occur at the beginning of the string. |
| {Raw}  <br>\[v1.0.43+\] | Enables the [Raw mode](#SendRaw), which causes the following characters to be interpreted literally: `^+!#{}`. Although `{Raw}` need not occur at the beginning of the string, once specified, it stays in effect for the remainder of the string. |
| {Text}  <br>\[v1.1.27+\] | Enables the [Text mode](#SendText), which sends a stream of characters rather than keystrokes. Like the Raw mode, the Text mode causes the following characters to be interpreted literally: `^+!#{}`. Although `{Text}` need not occur at the beginning of the string, once specified, it stays in effect for the remainder of the string. |

Repeating or Holding Down a Key
-------------------------------

**To repeat a keystroke:** Enclose in braces the name of the key followed by the number of times to repeat it. For example:

Send {DEL 4}  _; Presses the Delete key 4 times._
Send {S 30}   _; Sends 30 uppercase S characters._
Send +{TAB 4}  _; Presses Shift-Tab 4 times._

**To hold down or release a key:** Enclose in braces the name of the key followed by the word **Down** or **Up**. For example:

Send {b down}{b up}
Send {TAB down}{TAB up}
Send {Up down}  _; Presses down the up-arrow key._
Sleep 1000  _; Keeps it down for one second._
Send {Up up}  _; Releases the up-arrow key._

When a key is held down via the method above, it does not begin auto-repeating like it would if you were physically holding it down (this is because auto-repeat is a driver/hardware feature). However, a [Loop](Loop.htm) can be used to simulate auto-repeat. The following example sends 20 tab keystrokes:

Loop 20
{
    Send {Tab down}  _; Auto-repeat consists of consecutive down-events (with no up-events)._
    Sleep 30  _; The number of milliseconds between keystrokes (or use [SetKeyDelay](SetKeyDelay.htm))._
}
Send {Tab up}  _; Release the key._

By default, Send will not automatically release a modifier key (Control, Shift, Alt, and Win) if that modifier key was "pressed down" by sending it. For example, `Send a` may behave similar to `Send [{Blind}](#blind){Ctrl up}a{Ctrl down}` if the user is physically holding Ctrl, but `Send {Ctrl Down}` followed by `Send a` will produce Ctrl+A. _DownTemp_ and _DownR_ can be used to override this behavior. _DownTemp_ and _DownR_ have the same effect as _Down_ except for the modifier keys (Control, Shift, Alt, and Win).

**DownTemp** tells subsequent sends that the key is not permanently down, and may be released whenever a keystroke calls for it. For example, `Send {Control DownTemp}` followed later by `Send a` would produce A, not Ctrl+A. Any use of Send may potentially release the modifier permanently, so _DownTemp_ is not ideal for [remapping](../misc/Remap.htm) modifier keys.

\[v1.1.27+\]: **DownR** (where "R" stands for [remapping](../misc/Remap.htm), which is its main use) tells subsequent sends that if the key is automatically released, it should be pressed down again when send is finished. For example, `Send {Control DownR}` followed later by `Send a` would produce A, not Ctrl+A, but will leave Ctrl in the pressed state for use with keyboard shortcuts. In other words, _DownR_ has an effect similar to physically pressing the key.

If a character does not correspond to a virtual key on the current keyboard layout, it cannot be "pressed" or "released". For example, `Send {µ up}` has no effect on most layouts, and `Send {µ down}` is equivalent to `Send µ`.

General Remarks
---------------

**Characters vs. keys:** By default, characters are sent by first translating them to keystrokes. If this translation is not possible (that is, if the current keyboard layout does not contain a key or key combination which produces that character), the character is sent by one of following fallback methods:

* SendEvent and SendInput use [SendInput()](https://learn.microsoft.com/windows/win32/api/winuser/nf-winuser-sendinput) with the [KEYEVENTF_UNICODE flag](https://learn.microsoft.com/windows/win32/api/winuser/ns-winuser-keybdinput#keyeventf_unicode). \[v1.1.27+\]: ANSI builds of AutoHotkey convert the character to Unicode before sending it. Prior to v1.1.27, ANSI builds used the Alt+nnnnn method.
* SendPlay uses the [Alt+nnnnn](#asc) method, which produces Unicode only if supported by the target application.
* ControlSend posts a [WM_CHAR](https://learn.microsoft.com/windows/win32/inputdev/wm-char) message.

**Note:** Characters sent using any of the above methods usually do not trigger keyboard shortcuts or hotkeys.

\[v1.1.27+\]: For characters in the range a-z or A-Z (plain ASCII letters), each character which does not exist in the current keyboard layout may be sent either as a character or as the corresponding virtual keycode (vk41-vk5A):

* If a naked letter is sent (that is, without modifiers or braces), or if [Raw](#Raw) mode is in effect, it is sent as a character. For example, `Send {Raw}Regards` sends the expected text, even though pressing R (vk52) produces some other character (such as К on the Russian layout). `{Raw}` can be omitted in this case, unless a modifier key was put into effect by a prior Send.
* If one or more modifier keys have been put into effect by the Send command, or if the letter is wrapped in braces, it is sent as a keycode (modified with Shift if the letter is upper-case). This allows the script to easily activate standard keyboard shortcuts. For example, `^c` and `{Ctrl down}c{Ctrl up}` activate the standard Ctrl+C shortcut and `{c}` is equivalent to `{vk43}`.

If the letter exists in the current keyboard layout, it is always sent as whichever keycode the layout associates with that letter (unless the [Text mode](#SendText) is used, in which case the character is sent by other means). In other words, the section above is only relevant for non-Latin based layouts such as Russian.

**Modifier State:** When Send is required to change the state of the Win or Alt modifier keys (such as if the user was holding one of those keys), it may inject additional keystrokes (Ctrl by default) to prevent the Start menu or window menu from appearing. For details, see [#MenuMaskKey](_MenuMaskKey.htm).

**BlockInput Compared to SendInput/SendPlay:** Although the [BlockInput](BlockInput.htm) command can be used to prevent any keystrokes physically typed by the user from disrupting the flow of simulated keystrokes, it is often better to use [SendInput](#SendInputDetail) or [SendPlay](#SendPlayDetail) so that keystrokes and mouse clicks become uninterruptible. This is because unlike BlockInput, SendInput/Play does not discard what the user types during the send; instead, such keystrokes are buffered and sent afterward.

When sending a large number of keystrokes, a [continuation section](../Scripts.htm#continuation) can be used to improve readability and maintainability.

Since the operating system does not allow simulation of the Ctrl+Alt+Del combination, doing something like `Send ^!{Delete}` will have no effect.

**Send may have no effect** on Windows Vista or later if the active window is running with administrative privileges and the script is not. This is due to a security mechanism called User Interface Privilege Isolation.

SendInput \[v1.0.43+\]
----------------------

SendInput is generally the preferred method to send keystrokes and mouse clicks because of its superior speed and reliability. Under most conditions, SendInput is nearly instantaneous, even when sending long strings. Since SendInput is so fast, it is also more reliable because there is less opportunity for some other window to pop up unexpectedly and intercept the keystrokes. Reliability is further improved by the fact that anything the user types during a SendInput is postponed until afterward.

Unlike the other sending modes, the operating system limits SendInput to about 5000 characters (this may vary depending on the operating system's version and performance settings). Characters and events beyond this limit are not sent.

**Note:** SendInput ignores SetKeyDelay because the operating system does not support a delay in this mode. However, when SendInput reverts to [SendEvent](#SendEvent) under the conditions described below, it uses `[SetKeyDelay](SetKeyDelay.htm) -1, 0` (unless SendEvent's KeyDelay is `-1,-1`, in which case `-1,-1` is used). When SendInput reverts to [SendPlay](#SendPlayDetail), it uses SendPlay's KeyDelay.

If a script _other than_ the one executing SendInput has a [low-level keyboard hook](_InstallKeybdHook.htm) installed, SendInput automatically reverts to [SendEvent](#SendEvent) (or [SendPlay](#SendPlayDetail) if `[SendMode](SendMode.htm) InputThenPlay` is in effect). This is done because the presence of an external hook disables all of SendInput's advantages, making it inferior to both SendPlay and SendEvent. However, since SendInput is unable to detect a low-level hook in programs other than \[AutoHotkey v1.0.43+\], it will not revert in these cases, making it less reliable than SendPlay/Event.

When SendInput sends mouse clicks by means such as [{Click}](#Click), and `[CoordMode](CoordMode.htm) Mouse, Relative` is in effect (the default), every click will be relative to the window that was active at the start of the send. Therefore, if SendInput intentionally activates another window (by means such as alt-tab), the coordinates of subsequent clicks within the same command will be wrong because they will still be relative to the old window rather than the new one.

SendPlay \[v1.0.43+\]
---------------------

**Warning:** SendPlay may have no effect at all if UAC is enabled, even if the script is running as an administrator. For more information, refer to the [FAQ](../FAQ.htm#uac).

SendPlay's biggest advantage is its ability to "play back" keystrokes and mouse clicks in a broader variety of games than the other modes. For example, a particular game may accept [hotstrings](../Hotstrings.htm#SendMode) only when they have the [SendPlay option](../Hotstrings.htm#SendMode).

Of the three sending modes, SendPlay is the most unusual because it does not simulate keystrokes and mouse clicks per se. Instead, it creates a series of events (messages) that flow directly to the active window (similar to [ControlSend](ControlSend.htm), but at a lower level). Consequently, SendPlay does not trigger hotkeys or hotstrings.

Like [SendInput](#SendInputDetail), SendPlay's keystrokes do not get interspersed with keystrokes typed by the user. Thus, if the user happens to type something during a SendPlay, those keystrokes are postponed until afterward.

Although SendPlay is considerably slower than SendInput, it is usually faster than the traditional [SendEvent](#SendEvent) mode (even when [KeyDelay](SetKeyDelay.htm) is -1).

Both Win (LWin and RWin) are automatically blocked during a SendPlay if the [keyboard hook](_InstallKeybdHook.htm) is installed. This prevents the Start Menu from appearing if the user accidentally presses Win during the send. By contrast, keys other than LWin and RWin do not need to be blocked because the operating system automatically postpones them until after the SendPlay (via buffering).

SendPlay does not use the standard settings of [SetKeyDelay](SetKeyDelay.htm) and [SetMouseDelay](SetMouseDelay.htm). Instead, it defaults to no delay at all, which can be changed as shown in the following examples:

[SetKeyDelay](SetKeyDelay.htm), 0, 10, **Play**  _; Note that both 0 and -1 are the same in SendPlay mode._
[SetMouseDelay](SetMouseDelay.htm), 10, **Play**

SendPlay is unable to turn on or off CapsLock, NumLock, or ScrollLock. Similarly, it is unable to change a key's state as seen by [GetKeyState](GetKeyState.htm) unless the keystrokes are sent to one of the script's own windows. Even then, any changes to the left/right modifier keys (e.g. RControl) can be detected only via their neutral counterparts (e.g. Control). Also, SendPlay has other limitations described on the [SendMode page](SendMode.htm#Play).

Unlike [SendInput](#SendInputDetail) and [SendEvent](#SendEvent), the user may interrupt a SendPlay by pressing Ctrl+Alt+Del or Ctrl+Esc. When this happens, the remaining keystrokes are not sent but the script continues executing as though the SendPlay had completed normally.

Although SendPlay can send LWin and RWin events, they are sent directly to the active window rather than performing their native operating system function. To work around this, use [SendEvent](#SendEvent). For example, `SendEvent #r` would show the Start Menu's Run dialog.

Related
-------

[SendMode](SendMode.htm), [SetKeyDelay](SetKeyDelay.htm), [SetStoreCapsLockMode](SetStoreCapsLockMode.htm), [Escape sequences (e.g. `%)](../misc/EscapeChar.htm), [ControlSend](ControlSend.htm), [BlockInput](BlockInput.htm), [Hotstrings](../Hotstrings.htm), [WinActivate](WinActivate.htm)

Examples
--------

[](#ExBasic)Types a two-line signature.

Send Sincerely,{enter}John Smith

[](#ExModifier)Selects the File->Save menu (Alt+F followed by S).

Send !fs

[](#ExBrace)Jumps to the end of the text then send four shift+left-arrow keystrokes.

Send {End}+{Left 4}

[](#ExSendInputRaw)Sends a long series of [raw characters](#Raw) via the fastest method.

[SendInput](#SendInputDetail) {Raw}A long series of raw characters sent via the fastest method.

[](#ExVar)Holds down a key contained in a [variable](../Variables.htm).

MyKey := "Shift"
Send % "{" MyKey " down}"  _; Holds down the Shift key._

