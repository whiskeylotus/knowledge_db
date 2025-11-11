# Mobile

## Tools

- APK tool
- JADX
- MobSF
- frida

## Manifest.xml

Declares app components, permissions, exported activities/services/providers. 

- Allow Backup & Debuggable Flag
- Android Clear Text Traffic
- Minimum SDK Version
- User & Application Permission
- Default App Permissions

## Internal communications and structure

- IPC mechanisms
- Insecure local sockets, exposed services, custom URL schemes
- Unencrypted local data and logs

## Frida

Dynamic analysis toolkit used for runtime API hooking on Android / iOS.
Lets you intercept function calls and modify arguments/returns leading to bypass protections or script behaviors 

- SSL pinning, root bypass, logging secret

Install frida-tools on your OS
Install the good OS/Arch version of frida-server on your android/iOS and run it
Then you can interact from your OS

## Intercepting

- Proxying HTTP(S) via Burp/mitmproxy with certificate pinning bypass methods.
- Runtime hooks (Frida) to bypass pinning or log plaintext before SSL layer.
- Certificate pinning detection and mitigations.

How do you bypass certificate pinning in an Android app? 
Options: patch native libraries, hook SSL functions with Frida, modify network stack, or use an OS-level trust store if allowed.

## Why app crashes

- Tamper detection or anti-debugging mechanims
