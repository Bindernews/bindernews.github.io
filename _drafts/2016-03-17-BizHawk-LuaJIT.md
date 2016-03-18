---
layout: default
title: Using BizHawk with LuaJIT
---

# Introdution
BizHawk is a great emulator with Lua support. This allows controlling the emulator while it's
running and doing really cool things (e.g. [MarI/O](https://www.youtube.com/watch?v=qv6UVOQ0F44)).
Regular Lua is fast but we can do better. [LuaJIT](http://luajit.org/) is a Lua interpreter which is
faster than the regular interpreter. This document explains how to use LuaJIT with the BizHawk emulator.

## Pre-requisites
You'll need a few things to do this. First you need Visual Studio.
You can get the free edition for free! Use Google. Second you need to
download the [BizHawk source code](https://github.com/TASVideos/BizHawk). Third you need
the [LuaJIT source code](http://luajit.org/download.html). Make sure the Lua versions are
compatible. At the time of this writing BizHawk and LuaJIT both use Lua 5.1 but that may
change in the future. Once you have downloaded and extracted the source for both tools you
can go to the next section.

NOTES:
 * The LuaInterface directory is in the BizHawk source code.
 * I extracted the LuaJIT source to `LuaJIT` but you can call it anything you want.

## Making LuaJIT work with LuaDLL.cpp and LuaPerks
LuaInterface requires Lua be integrated with the .NET framework and
Microsoft's CLR and does this using `Lua/src/LuaDLL.cpp`. If you want to
use LuaJIT with LuaInterface you need to include this file in the DLL.
We also need `Lua/src/.libs/luaperks.lib`. This might be in `luaperks.7z`
so just extract it if that is the case. Now copy `LuaDLL.cpp` and
`luaperks.lib` into `LuaJIT/src/`. Now we need to edit `msvcbuild.bat`.

Edit `msvcbuild.bat` and go to line 74 (it might have changed since We need to replace

~~~
%LJCOMPILE% /MD /DLUA_BUILD_AS_DLL lj_*.c lib_*.c
@if errorlevel 1 goto :BAD
%LJLINK% /DLL /out:%LJDLLNAME% lj_*.obj lib_*.obj
~~~

with

~~~
%LJCOMPILE% /MD /DLUA_BUILD_AS_DLL lj_*.c lib_*.c
%LJCOMPILE% /MD /DLUA_BUILD_AS_DLL /clr LuaDLL.cpp
@if errorlevel 1 goto :BAD
%LJLINK% /DLL /out:%LJDLLNAME% lj_*.obj lib_*.obj LuaDLL.obj .libs\luaperks.lib
~~~

## Compiling LuaJIT
NOTE: If you have the MSVC compiler in your PATH you can ignore this next paragraph.
If you don't know what that means then read the next paragraph.

The normal Command Prompt doesn't have `cl` in its PATH. You need to go to your
start menu or start screen or whatever and search for "Visual Studio Tools". It
should be a folder. If you click on that it will open a folder with a few links
in it. You want the one that says "x86 Native Tools Command Prompt".

Now that you have the right command prompt open you can `cd` to the `LuaJIT/src`
directory. Once there run `msvcbuild.bat`. You should get a message saying
"=== Successfully built LuaJIT for Windows/x86 ===" and you're good to move on.

## Compiling LuaInterface with LuaJIT
We need to tell LuaInterface to use our custom DLL instead of the built-in one.
So start Visual Studio and open `LuaInterface/LuaInterface.sln`. There should be two projects:
"Lua514" (meaning Lua version 5.1.4) and "LuaInterface". You want to right-click "Lua514" and
click "Remove". Click "Remove" (not "Delete") on the dialog box that pops up and that's done.
Now LuaInterface doesn't know where to find the Lua DLL it needs to work so we need to fix that.
Expand the "LuaInterface" project and you should see "References". Right-click and select
"Add Reference". Now at the bottom of the new window you should see "Browse". Click that and
then go find `lua51.dll` which should be in the `LuaJIT/src` directory. Make sure your build
configuration is set to be "Release-LUAPERKS" and build it!

Now in `LuaInterface/LuaInterface/bin/x86/Release-LUAPERKS/` you should see `lua51.dll` and
`LuaInterface.dll`. This is it! We don't actually need to compile the rest of BizHawk. Go to
your folder where you have the COMPILED version of BizHawk. It should contain `EmuHawk.exe`.
In there is a folder named `dll`. Replace the `lua51.dll` and `LuaInterface.dll` in that folder
with the ones in the `Release-LUAPERKS` folder. Test to make sure it EmuHawk starts up correctly.

Great job!

