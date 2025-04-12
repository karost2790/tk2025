## 1.Uninstall and Clean Up Existing nvm-windows
### 1.Open Command Prompt as Administrator:
  -Press Win + S, type cmd, right-click Command Prompt, select Run as administrator.
  -Click Yes on UAC prompt.
### 2.Remove nvm-windows Directory:
  ```bash
  rmdir /s /q C:\nvm4w
  ```
### 3.Remove Node.js Symlink:
  ```bash
  rmdir /s /q "C:\Program Files\nodejs"
  ```
### 4.Remove nvm Data:
  ```bash
  rmdir /s /q C:\Users\thanit\AppData\Local\nvm
  ```
### 5.Check for Manual Node.js:
- If Node.js was installed manually:
  - Go to Control Panel > Programs > Uninstall a program.
  - Uninstall “Node.js” if listed.
- Or run:
  ```bash
  dir "C:\Program Files\nodejs"
  ```
  - If exists, delete it (already done above).

## 2.Reinstall nvm-windows
- 1.Download nvm-windows:
    - Open a browser, go to https://github.com/coreybutler/nvm-windows/releases.
    - Download nvm-setup.exe (e.g., version 1.1.12).
    - Save to F:\Ai-srv.

- 2.Run Installer:
    - Double-click F:\Ai-srv\nvm-setup.exe.
    - Click Next, accept the license.
    - Set nvm install path to C:\nvm4w (to match your preference).
    - Click Install, then Finish

- 3.Close Command Prompt:
  ```bash
  exit
  ```
## 3.Verify nvm-windows Installation
- 1.Open New Command Prompt:
  - Win + R, cmd, Enter.
- 2.Check nvm Version:
  ```bash
  nvm version
  ```
  - Expect: 1.1.12.
  - If Failed:
    - Add C:\nvm4w to PATH:
    ```bash
    set PATH=%PATH%;C:\nvm4w
    ```
  - Try again.






  


 




