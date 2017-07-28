# LuaMemorySnapshotDump
Lua memory snapshot dump utility, used for memory leak detection。

## About
- About this code: http://www.cnblogs.com/yaukey/p/unity_lua_memory_leak_trace.html. (Chinese)
- Just run "Example.lua" for a quick start.
- Compatible with Lua 5.1, 5.2 and 5.3.
- Support dumping all lua object reference information in memory sorted by reference count into local txt file.
- Support comparing two dumped memory reference file and outputing the increased memory information into another file.
- Support filtering the result file by include/exclude the custom keywords.

## Usage

First of all, you need to require `MemoryReferenceInfo.lua` in your file, like:

```lua
local mri = require(MemoryReferenceInfo)
```

Next step, you need to dump the lua memory snapshot at first time somewhere:

```lua
-- Before dumping, collect garbage first.
collectgarbage("collect")
mri.m_cMethods.DumpMemorySnapshot("./", "1-Before", -1)

```

By default, ```DumpMemorySnapshot``` will output a txt file whose name is suffixed with current time stamp in order to dump file continuously without modifying the file name manually each time you dump, but you can disable time stamp suffix by setting the config:

```lua

-- Setting the config not add time stamp at the end of the file name.
mri.m_cConfig.m_bAllMemoryRefFileAddTime = false

```

Generally, you need to set this config at the very begining just below the ```local mri = require(MemoryReferenceInfo)```.

How to read the result? All references are lined, and seperated by tab into 3 columns which are type:address/name, reference chain, reference count. Lines are sorted by references count in descend. So, you can open this result file by ```Microsoft Excel```, who can process the columns and tabs automaticly. 

For example: ```function: 0x7f85f8e0e3f0	registry.2[_G].Author.Ask[line:33@file:example.lua]	1``` says that there is a function whose address is 0x7f85f8e0e3f0, table registry has a table value field named "2" who is actually "_G", _G has a table "Author", "Author" has a function "Ask" in file "example.lua" at line 33, it's "Ask" reference the function, and there is only 1 reference in total. That's all!

As you want to check the memory leak, so you need another memory snapshot to see if it is bigger than the older one, even you need to compare this with the older one to see what exactly increased.

So, after running some time, it's time to take another memory snapshot:

```lua

-- Dump memory snapshot again.
collectgarbage("collect")
mri.m_cMethods.DumpMemorySnapshot("./", "2-After", -1)

```

Now, you have two files stored in your current directory, named: ```LuaMemRefInfo-All-[1-Before].txt``` and ```LuaMemRefInfo-All-[2-After].txt```.

Then, you can now compare the two files to see the increased data:

```lua

mri.m_cMethods.DumpMemorySnapshotComparedFile("./", "Compared", -1, 
"./LuaMemRefInfo-All-[1-Before].txt", 
"./LuaMemRefInfo-All-[2-After].txt")

```

This output a file named like ```LuaMemRefInfo-All-[20170720-234651]-[Compared].txt``` who contains the data increased in ```2-After``` but not exist in ```1-Before```.

Sometimes, you want to find all the references of an object(table, function, userdata, thread, string), you can do this:

```lua

-- Find all the references who referenced object "_G.Author".
collectgarbage("collect")
mri.m_cMethods.DumpMemorySnapshotSingleObject("./", "SingleObjRef-Object", -1, "Author", _G.Author)

```

You can also find string references like this:

```lua

-- Find all the references who referenced string "yaukeywang".
collectgarbage("collect")
mri.m_cMethods.DumpMemorySnapshotSingleObject("./", "SingleObjRef-String", -1, "Author Name", "yaukeywang")

```

The output results will show all the details for that.

In large project, due to the complex of the lua state, the result file may be pretty lary, but you can do this to filter the data:

```lua

-- Filter all content that include the keyword "Author" (ignore case), and save as a new file.
mri.m_cBases.OutputFilteredResult("./LuaMemRefInfo-All-[2-After].txt", "Author", true, true)

-- Filter all content that exclude the keyword "Author"  (ignore case), and save as a new file.
mri.m_cBases.OutputFilteredResult("./LuaMemRefInfo-All-[2-After].txt", "Author", false, true)

```

With these utilities, i think you can fix the memory leak quickly.

## More

For method ```DumpMemorySnapshot``` the last 2 parameters is the root object name and the root object to search from, with default value "registry" and "debug.getregistry()", which you may not change in most case, it will finally dump the entrie lua state memory snapshot. But if you want to search from another object e.g. "_G", you can set it manually.

```lua

-- Only dump memory snapshot searched from "_G".
collectgarbage("collect")
mri.m_cMethods.DumpMemorySnapshot("./", "1-Before", -1, "_G", _G)

```

It will only dump the memory snapshot in "_G".

## Note

In order to display the ```"string"``` type in single line by dumping output result, all the value ```'\r'```, ```'\n'``` are replaced by the "real" character: ```'\\n'```.
