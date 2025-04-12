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


  


 




