The Windows Engine Upgrade: A Reference Guide

This document explains the core modifications made to the roots.go file. It translates the technical Go logic into plain-English concepts so you can easily reference how your scanner interacts with the Windows operating system.
1. The "Don't Freeze the Computer" Guardrails

(Code: isWindowsBroadHomePath, isBroadHomeRoot)

    What it means: We added explicit rules that recognize Windows drive letters (like C:\ or D:\) and the main user directory (C:\Users).

    Why we did it: The scanner is designed to crawl deeply into folders to find dependencies. If a user accidentally pointed the scanner at the very top of their C:\ drive, it would try to scan millions of protected Windows system files, causing the application (and potentially the computer) to freeze. These guardrails instantly block those catastrophic "full-drive" scans.

2. The AppData Pathfinders

(Code: roamingAppDataDir, localAppDataDir)

    What it means: We created two dedicated helper functions that automatically locate the user's hidden Windows data folders: %APPDATA% (Roaming) and %LOCALAPPDATA% (Local).

    Why we did it: Unlike Mac or Linux, where tools are usually kept in neat, hidden folders in the home directory (like ~/.config), Windows scatters tool data across these two hidden directories. Chrome uses "Local", while Firefox and npm use "Roaming". These helpers ensure the scanner never has to guess where those folders are, even if a user has a highly customized Windows setup.

3. The "Smart Memory" Deduplicator

(Code: strings.ToLower(key) and the seen map in filterExistingRoots)

    What it means: We taught the scanner that on Windows, a folder named C:\Projects and c:\projects are the exact same thing. We also gave it a checklist (seen) to remember every folder it has already verified.

    Why we did it: Windows file systems are "case-insensitive," but the Go programming language is strictly case-sensitive. Without this fix, the scanner might resolve the same path twice with different capital letters, scan the folder twice, and show you duplicate vulnerability alerts in your Vue dashboard. This change makes the scanner significantly faster and keeps your data perfectly clean.

4. Catching "Install for Everyone" Tools

(Code: The case "windows": block in systemRoots)

    What it means: We instructed the scanner to look inside C:\Program Files and C:\Program Files (x86).

    Why we did it: When developers install tools like Python or Node.js on Windows, the installer often asks: "Install for just you, or all users?" If they click "all users," the tools are placed in the Program Files directory instead of their personal AppData folder. This change ensures we catch system-wide installations that our previous, user-focused scan would have completely missed.

5. Tracking Down Modern Tools and Quirky Browsers

(Code: baselineHomeCandidates and browserExtensionCandidateRoots)

    What it means: We expanded the search list to include modern package managers (like pnpm, Yarn, and Windows-specific Python managers like pyenv-win). We also added highly specific logic to find the Arc Browser.

    Why we did it: The original code was built for older, standard tools. We upgraded it for modern development workflows. The Arc Browser change is especially important: Arc on Windows is an "MSIX App," meaning Microsoft forces it to hide its extension data inside a deeply buried, randomized folder path. We added a specific "glob" pattern to hunt that exact folder down so you don't miss compromised browser extensions.

With this reference document saved, you have a complete map of the backend engine's architecture.