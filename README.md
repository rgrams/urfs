
## UnRestricted FileSystem - For LÖVE

A small module that uses LuaJIT's FFI to bypass the arbitrary restrictions on [love.filesystem](https://love2d.org/wiki/love.filesystem) so you can read and write files and folders anywhere.

### About

[PhysFS](https://icculus.org/physfs/docs/html/index.html) is the underlying C library that powers [love.filesystem](https://love2d.org/wiki/love.filesystem). It offers some filesystem features that Lua's standard `io` library does not, such as: listing all files & folders in a directory, and creating new directories (in a cross-platform way).

Unfortunately, Löve hamstrings these features by restricting their use to only a [few possible locations](https://love2d.org/wiki/love.filesystem):
* The "save directory" (such as: C:\\Users\\bob\\AppData\\Roaming...).
* The folder that your main.lua is in.
* The folder containing your .love file, _if_ the game is fused.
* Folders that the user [drag-and-drops](https://love2d.org/wiki/love.directorydropped) onto the game window while it is running.

Also Löve  _only_ ever allows _write_ access to the save directory.

Usually this is fine, if you're making a normal, small game, but if you want to make any kind of editor or tool, custom file dialogs, add certain modding capabilities, or anything else that would require greater filesystem access, then you are out of luck.

"URFS" uses the FFI to call PhysFS functions directly, bypassing Löve's added restrictions so you can do what you want.

Since PhysFS is already included in Löve, you don't need to add any .dlls or .sos or install anything else on your system. Everything in love.filesystem will work without modification; URFS only gives alternatives to the functions that are restricted (mount & unmount) or nonexistent (get/setWriteDir) to give the love.filesystem functions access to other directories.

### Setup

Add urfs.lua to your project and require it.

```lua
-- Example:
local urfs = require "urfs"
```

### Functions

#### urfs.mount(archive, [mountPoint], [appendToPath])
Add a directory or archive (.zip) to the list of search paths for reading files. This means the files and folders inside of `archive` will be accessible to the "reading" functions from love.filesystem, such as [love.filesystem.getInfo](https://love2d.org/wiki/love.filesystem.getInfo), [love.filesystem.getDirectoryItems](https://love2d.org/wiki/love.filesystem.getDirectoryItems), and [love.fileSystem.getRealDirectory](https://love2d.org/wiki/love.fileSystem.getRealDirectory), among others. They will also be accessible to the asset loading functions like [love.graphics.newImage](https://love2d.org/wiki/love.graphics.newImage) and so on.

_PARAMETERS:_
* **`archive`** - <kbd>string</kbd> - The path of the directory or archive to mount.
* **`mountPoint`** - <kbd>string</kbd> - _Optional_ - The virtual path that `archive` will be mounted to. This can overlap existing virtual paths. The default of `nil` (equivalent to an empty string) will add the files from `archive` to the root of the virtual filesystem.
* **`appendToPath`** - <kbd>string</kbd> - _Optional_ - Whether `archive` should be searched before or after already-mounted archives when reading files. By default (`appendToPath = nil` or `false`) previously-mounted files will be accessed if paths overlap. Pass in `true` to have the new files from `archive` be prioritized first.

_RETURNS:_
* **`isSuccess`** - <kbd>bool</kbd> - `true` if `archive` was mounted successfully, false if not.
* **`errorMsg`** - <kbd>string | nil</kbd> - The error message if mounting failed, or `nil` if it was successful.

See: [PHYSFS_mount](https://icculus.org/physfs/docs/html/physfs_8h.html#a8eb320e9af03dcdb4c05bbff3ea604d4) for more info.

#### urfs.unmount(archive)
Remove a directory or archive that was previously mounted from the search path.

_PAREMETERS_
* **`archive`** - <kbd>string</kbd> - The original path of the directory or archive to unmount. This must equal the `archive` parameter previously passed to `urfs.mount`.

_RETURNS:_
* **`isSuccess`** - <kbd>bool</kbd> - `true` if `archive` was unmounted successfully, false if not.
* **`errorMsg`** - <kbd>string | nil</kbd> - The error message if unmounting failed, or `nil` if it was successful.

See: [PHYSFS_unmount](https://icculus.org/physfs/docs/html/physfs_8h.html#aab0e2ba90aa918b2ee1ed7c40293b442) for more info.

#### urfs.setWriteDir(dir)
PhysFS (and love.filesystem) always has a _single_ directory that is used to write files and directories. This function lets you change that directory. This affects all love.filesystem functions that modify or create files and directories, such as: [love.filesystem.createDirectory](https://love2d.org/wiki/love.filesystem.createDirectory), [love.filesystem.newFile](https://love2d.org/wiki/love.filesystem.newFile), and [love.filesystem.write](https://love2d.org/wiki/love.filesystem.write). By default, Löve sets the write dir to the save directory.

_PARAMETERS:_
* **`dir`** - <kbd>string</kbd> - The new absolute path to the directory that will be used to write files and directories.

_RETURNS:_
* **`isSuccess`** - <kbd>bool</kbd> - `true` if the write dir was set successfully, false if not.
* **`errorMsg`** - <kbd>string | nil</kbd> - The error message if `setWriteDir` failed, or `nil` if it was successful.

See: [PHYSFS_setWriteDir](https://icculus.org/physfs/docs/html/physfs_8h.html#a36c408d40b3a93c8f9fc02a16c02e430) for more info.

#### urfs.getWriteDir()
Get the current directory used by love.filesystem to write files and directories.

_RETURNS:_
* **`writeDir`** - <kbd>string</kbd> - The current absolute path to the directory used to write files and directories. If you have not already used `urfs.setWriteDir` to change the write dir, this will return the same value as [`love.filesystem.getSaveDirectory()`](https://love2d.org/wiki/love.filesystem.getSaveDirectory).

See: [PHYSFS_getWriteDir](https://icculus.org/physfs/docs/html/physfs_8h.html#a6533ff91180a4c8abfe24d458f6b9915) for...not much more info.
