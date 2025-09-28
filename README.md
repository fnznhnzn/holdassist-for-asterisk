# holdassist-for-asterisk
iOS 26 and the iPhone 17 can do it, so can we in Asterisk / FreePBX

When put on (endless) hold by some 1st level support, just transfer the call to 4653 (as in "HOLD") and hang up. The operator will hear "Press zero to connect" and your phone will ring.

Two places in FreePBX GUI:
1. Admin => Custom Destination with target 6453
2. Application => Misc Application with 6453 as Feature Code and the above as Custom Destination als Target.

Two places in the file system
1. Put audio for "Press zero..." in /var/lib/asterisk/sounds/custom (see below)
2. Put the following into /etc/asterisk/extensions_custom.conf

```ini
[macro-dialout-trunk-predial-hook]
; The extension that transfered the call (so it can be called back). Only available before dialing, so it must be put in a global variable
exten => s,n,Set(__ORIGINAL_CALLER=${AMPUSER})

[custom-4653]
; Start of call in this context
exten => s,1,Answer()
    same => n,NoOp(Original caller: ${ORIGINAL_CALLER})
    same => n,NoOp(Callee/DIALEDPEERNUMBER: ${DIALEDPEERNUMBER})
    same => n,Background(custom/PressZeroToReconnect)
    same => n,WaitExten(2)

; User presses 0 → reconnect
exten => 0,1,NoOp(Caller pressed 0, dialing original caller)
    same => n,Dial(PJSIP/${ORIGINAL_CALLER})
    same => n,NoOp(Back from dial, returning to prompt)
    same => n,Goto(s,2)

; Timeout handler
exten => t,1,NoOp(TIMEOUT reached in custom-4653)
    same => n,Goto(s,2)     ; replay prompt

; Invalid input handler
exten => i,1,NoOp(INVALID input in custom-4653)
    same => n,Goto(s,2)     ; replay prompt

; Hangup
exten => h,1,NoOp(Hangup from custom-4653)
    same => n,Hangup()
```
<sub>Inspired by and adapted to FreePBX from https://nerdvittles.com/tired-of-waiting-on-hold-heres-a-simple-asterisk-fix, where they have an excellent audio file for the purpose</sub>
