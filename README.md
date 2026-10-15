# Pegasus-Chrysaor-Android-Analysis
Technical deep-dive threat intelligence report on the Android variant of Pegasus/Chrysaor spyware and cyberweapon. Covers native architecture, Binder IPC, ptrace injection, and screenshot extraction.

Threat Intelligence Report: Pegasus/Chrysaor Android Espionage Malware (Spyware)

Executive Summary
Recent deep-dive reverse engineering of the Android variant of the Pegasus spyware (commonly attributed to Chrysaor indicators) reveals a modular, multi-binary architecture designed for stealthy surveillance, process injection, remote command execution, and local data exfiltration. The toolkit relies on specialized components handling privilege escalation, Inter-Process Communication (IPC) via Android Binder, system-level debugging primitives (ptrace), and real-time screen capture with compression.

Sample Identification & Static Telemetry
A. Identification & Digital Signature Analysis (Hatching Triage)
Sample SHA-256: bd8cda80aaee3e4a17e9967a1c062ac5c8e4aefd7eaa3362f54044c2c94db52a

Analyzed Filename: giga nigga.apk (~1.06 MB – 1.1 MB)

Triage Score: 6 / 10

Code & Signature Anomalies:

Digest Mismatch: The file structure was altered or tampered with after initial signing.

Insecure Codesign Algorithm: Legacy/weak cryptographic algorithms were utilized for signing (SHA-1 digest).

Untrusted Signer: Generic self-signed certificate (CN=Unknown, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown).

Requested Framework Permissions (21 Dangerous Permissions out of 50+ declared):

Demands near-total device control: BIND_DEVICE_ADMIN, ACCESS_SURFACE_FLINGER (critical for direct framebuffer/graphic interaction), CALL_PRIVILEGED, BRICK, DEVICE_POWER, CONTROL_LOCATION_UPDATES, FORCE_STOP_PACKAGES, ACCESS_CHECKIN_PROPERTIES, ACCESS_COARSE_LOCATION, ACCESS_FINE_LOCATION, ACCESS_LOCATION_EXTRA_COMMANDS, ACCESS_MOCK_LOCATION, ACCESS_NETWORK_STATE, ACCESS_WIFI_STATE, ACCOUNT_MANAGER, AUTHENTICATE_ACCOUNTS, BATTERY_STATS, BIND_APPWIDGET, BIND_INPUT_METHOD, BIND_REMOTEVIEWS, BIND_WALLPAPER, BLUETOOTH, BLUETOOTH_ADMIN, BROADCAST_PACKAGE_REMOVED, BROADCAST_SMS, BROADCAST_STICKY, BROADCAST_WAP_PUSH, CALL_PHONE, CAMERA, CHANGE_COMPONENT_ENABLED_STATE, CHANGE_CONFIGURATION, CHANGE_NETWORK_STATE, CHANGE_WIFI_MULTICAST_STATE, CHANGE_WIFI_STATE, CLEAR_APP_CACHE, CLEAR_APP_USER_DATA, DELETE_CACHE_FILES, DELETE_PACKAGES, DIAGNOSTIC, DISABLE_KEYGUARD, DUMP, EXPAND_STATUS_BAR, FACTORY_TEST, FLASHLIGHT, FORCE_BACK, GET_ACCOUNTS, GET_PACKAGE_SIZE, GET_TASKS, GLOBAL_SEARCH, HARDWARE_TEST, INJECT_EVENTS, INSTALL_LOCATION_PROVIDER, INSTALL_PACKAGES, INTERNAL_SYSTEM_WINDOW, INTERNET, KILL_BACKGROUND_PROCESSES, MANAGE_ACCOUNTS, MANAGE_APP_TOKENS, MASTER_CLEAR, MODIFY_AUDIO_SETTINGS, MODIFY_PHONE_STATE, MOUNT_FORMAT_FILESYSTEMS, MOUNT_UNMOUNT_FILESYSTEMS, NFC, PERSISTENT_ACTIVITY, PROCESS_OUTGOING_CALLS, READ_CALENDAR, READ_CONTACTS, READ_FRAME_BUFFER, READ_HISTORY_BOOKMARKS, READ_INPUT_STATE, READ_LOGS, READ_PHONE_STATE, READ_SMS, READ_SYNC_SETTINGS, READ_SYNC_STATS, REBOOT, RECEIVE_BOOT_COMPLETED, RECEIVE_MMS, RECEIVE_SMS, RECEIVE_WAP_PUSH, RECORD_AUDIO, REORDER_TASKS, RESTART_PACKAGES, SEND_SMS, SET_ACTIVITY_WATCHER, SET_ALARM, SET_ALWAYS_FINISH, SET_ANIMATION_SCALE, SET_DEBUG_APP, SET_ORIENTATION, SET_PREFERRED_APPLICATIONS, SET_PROCESS_LIMIT, SET_TIME, SET_TIME_ZONE, SET_WALLPAPER, SET_WALLPAPER_HINTS, SIGNAL_PERSISTENT_PROCESSES, STATUS_BAR, SUBSCRIBED_FEEDS_READ, SUBSCRIBED_FEEDS_WRITE, SYSTEM_ALERT_WINDOW, UPDATE_DEVICE_STATS, USE_CREDENTIALS, USE_SIP, VIBRATE, WAKE_LOCK, WRITE_APN_SETTINGS, WRITE_CALENDAR, WRITE_CONTACTS, WRITE_EXTERNAL_STORAGE, WRITE_GSERVICES, WRITE_HISTORY_BOOKMARKS, WRITE_HISTORY_BOOKMARKS, WRITE_SECURE_SETTINGS, WRITE_SETTINGS, WRITE_SMS, WRITE_SYNC_SETTINGS, READ_HISTORY_BOOKMARKS, ACCESS_PROVIDER, EmailProvider, WRITE_APN_SETTINGS, MODIFY_AUDIO_SETTINGS, WAKE_LOCK, SET_WALLPAPER_HINTS, READ_SYNC_SETTINGS, READ_SYNC_STATS, NFC, READ_SYNC_SETTINGS, UNINSTALL_SHORTCUT, CHANGE_WIFI_MULTICAST_STATE, INTERNET, NFC, ACCESS_NOTIFICATION_POLICY, CHANGE_NETWORK_STATE, READ_SYNC_STATS, UNINSTALL_SHORTCUT.

B. Industry Consensus & Detection Metrics (VirusTotal)
Detection Ratio: 47 / 68 security vendor engines flagged the file as malicious (Community Score: -105).

Primary Threat Family: Trojan-Spy.AndroidOS.Pegasus / Chrysaor (attributed to NSO Group).

Sandbox Attributes (Yomi Hunter): Trojan, Dropper, Spyware, Obfuscated, Contains ELF (Native Executable Libraries/Binaries).

Binary Architecture & Component Breakdown
The spyware package is segregated into distinct modular binaries, each engineered for a specific phase of compromise and post-exploitation persistence:

1. cmdshell & sucopier — Execution and Privilege Management
cmdshell: Functions as a lightweight command execution conduit, managing shell interactions and system command pipelines.

sucopier: Operates as a privilege escalation and file-staging utility, designed to copy files into protected system directories or leverage root binaries (su) when available.

2. libk — Advanced IPC and Dynamic Injection Library
Architecture: A heavy, object-oriented C++ shared library (libk.so) deeply integrated with the Android runtime and system services.

Binder IPC Integration: Imports native Android runtime libraries (libbinder.so, libutils.so) and utilizes mangled C++ symbols for the android::Parcel class (_ZNK7android6Parcel4dataEv, _ZNK7android6Parcel12readString16Ev), enabling direct serialization and manipulation of low-level Binder transactions.

Dynamic Loading & C++ Exceptions: Features full Run-Time Type Information (RTTI), standard exception handling (__cxa_begin_catch, _Unwind_Resume), and dynamic symbol resolution primitives (dlopen, dlsym) to load secondary payloads or hooks into memory covertly.

3. addk — Process Injection & Debugging Engine
Architecture: A 32-bit ARM standalone executable interacting with the standard Android dynamic linker (/system/bin/linker).

Core Mechanism: Acts as a process injection manager leveraging Linux ptrace capabilities (ptrace, waitpid). It pauses, inspects, and modifies the address space of running legitimate processes.

Payload Delivery: Employs symbols such as _inject_start_s, _inject_function_param_s, and _dlopen_addr_s to construct shellcode wrappers that force target processes to load malicious shared libraries dynamically.

4. take_screen_shot — Visual Surveillance & Compression Module
Architecture: A specialized surveillance utility dedicated to capturing device displays and packaging them for exfiltration.

Hardware Interaction: Interfaces with low-level graphics drivers and framebuffers using ioctl, lseek, and read to query screen dimensions and extract raw pixel data accurately.

Compression Pipeline: Unlike simpler spyware samples, this module bundles full libz (deflate, deflateInit2_, crc32) and libpng integration (png_IHDR, png_IDAT, png_IEND). It compresses captured framebuffers directly into valid, lightweight PNG image structures in memory before storage or transmission.

Indicators of Compromise (IoC) Summary & Technical Takeaways
Target Environment: Android ARM 32-bit architectures.

Persistence & Evasion: Utilizes custom exception handlers, dynamic symbol resolution to bypass static import analysis, and process hollowing/injection via ptrace.

Capabilities:

Real-time audio/visual surveillance (including compressed screenshot generation).

System-wide command execution and root-level staging.

Deep integration with Android Service Manager via Binder IPC.

Prepared by: Threat Intelligence & Malware Analysis Lab

7z Password: infected.
