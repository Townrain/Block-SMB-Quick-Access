# Block SMB Quick Access (v1.1)


A Windhawk mod that prevents SMB/UNC network paths from being added to Quick Access in Windows Explorer.

## Quick Start

### Prerequisites

1. Install [Windhawk](https://windhawk.net/) (version 1.4 or newer)
2. Windows 10 or Windows 11 (64-bit)

### Installation

#### Option 1: Manual Installation (Recommended)

1. Open Windhawk
2. Go to **Settings** → **Advanced** → **Open mods directory**
3. Copy `block-smb-quick-access.wh.cpp` to the mods directory
4. Go back to Windhawk main window
5. Find "Block SMB Quick Access" in the mods list
6. Click **Enable**
7. Restart Explorer or log out/in

#### Option 2: Using Windhawk Editor

1. Open Windhawk
2. Click **New mod** (or press Ctrl+N)
3. Copy the contents of `block-smb-quick-access.wh.cpp`
4. Paste into the editor
5. Click **Compile and load**
6. The mod will be saved and enabled automatically

### Configuration

After enabling the mod, you can configure it in Windhawk:

| Setting | Default | Description |
|---------|---------|-------------|
| Log blocked paths | Enabled | Log when SMB path is blocked (useful for debugging) |
| Block all UNC paths | Disabled | Block all UNC paths, not just SMB |
| Verbose logging | Disabled | Log all SHAddToRecentDocs calls (very verbose) |

### Verification

To verify the mod is working:

1. Enable "Log blocked paths" in mod settings
2. Open Windhawk debug logs (Settings → Advanced → Show log output)
3. Open a file from a network share (e.g., `\\server\share\file.txt`)
4. Check logs for Blocked SMB/UNC path` entry
5. Verify the file does NOT appear in Quick Access

### Troubleshooting

| Problem | Solution |
|---------|----------|
| Mod doesn't appear in list | Verify file is in correct mods directory |
| Mod fails to compile | Check Windhawk version (need 1.4+) |
| SMB paths still appear | Restart Explorer after enabling mod |
| No log entries | Enable "Verbose logging" temporarily |

### Uninstallation

1. Open Windhawk
2. Find "Block SMB Quick Access" in mods list
3. Click **Disable** or **Remove**
4. Restart Explorer

## Technical Details

### How It Works

The mod hooks `SHAddToRecentDocs` in `shell32.dll`. When Explorer calls this function:

1. Extract the path from the SHARD parameter (supports all 8 types)
2. Check if path starts with `\\` (UNC path)
3. If UNC path → silently drop the call
4. If local path → call original function

### SHARD Types Supported

| Type | Value | Description |
|------|-------|-------------|
| SHARD_PIDL | 0 | Pointer to PIDL |
| SHARD_PATHA | 1 | ANSI string path |
| SHARD_PATHW | 2 | Unicode string path |
| SHARD_APPIDINFO | 3 | IShellItem + AppID |
| SHARD_APPIDINFOIDLIST | 4 | PIDL + AppID |
| SHARD_LINK | 5 | IShellLink |
| SHARD_APPIDINFOLINK | 6 | IShellLink + AppID |
| SHARD_SHELLITEM | 7 | IShellItem |

### Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    explorer.exe                              │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                 Windhawk Engine                         ││
│  │  ┌─────────────────────────────────────────────────────┐││
│  │  │         block-smb-quick-access.wh.cpp               │││
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │││
│  │  │  │ Wh_ModInit  │  │ Hook Func   │  │ SMB Filter  │ │││
│  │  │  │ (设置钩子)  │  │ (拦截函数)  │  │ (路径过滤)  │ │││
│  │  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘ │││
│  │  │         │                │                │         │││
│  │  │         ▼                ▼                ▼         │││
│  │  │  Wh_SetFunctionHook  ShouldBlock()  IsUncPathW()   │││
│  │  │  (注册钩子)         (判断是否阻止)  (UNC检测)      │││
│  │  └─────────────────────────────────────────────────────┘││
│  └─────────────────────────────────────────────────────────┘│
│                           │                                  │
│                           ▼                                  │
│              shell32.dll!SHAddToRecentDocs                   │
└─────────────────────────────────────────────────────────────┘
```

## Comparison with Original Plan

| Aspect | Original Plan | Windhawk Mod |
|--------|---------------|--------------|
| Code | 500+ lines, 3 executables | 350 lines, 1 file |
| Build | CMake + Visual Studio | Windhawk auto-compile |
| Install | Batch scripts + scheduled task | GUI click |
| Injection | Manual SetWindowsHookEx | Windhawk handles |
| DllMain | Loader lock workarounds | Not needed |
| Explorer restart | Manual monitoring | Auto-reload |
| Architecture | x86/x64 separate builds | @architecture tag |
| AV/EDR risk | High | Low |

## License

MIT License

## Credits

- Windhawk by [Ramen Software](https://ramensoftware.com/)
- Based on technical analysis of Windows Shell API

## Changelog

### v1.1 (2026-05-29)

- **Bug fix**: Added `SLGP_UNCPRIORITY` flag to `IShellLink::GetPath` to correctly detect UNC paths instead of mapped drive letters
- **Optimization**: Simplified `IsUncPathW`/`IsUncPathA` functions by removing redundant `wcslen`/`strlen` calls
- **Cleanup**: Removed unused `g_blockAllUNC` setting variable
- **Cleanup**: Replaced custom `_STRUCT` definitions with official Windows SDK structures (`SHARDAPPIDINFO`, `SHARDAPPIDINFOIDLIST`, `SHARDAPPIDINFOLINK`)
- **Docs**: Removed incorrect numeric comments from SHARD switch cases

### v1.0 (2026-05-29)

- Initial release
