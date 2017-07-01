# LuaMemorySnapshotDump
Lua memory snapshot dump utility, used for memory leak detection。

About this code: http://www.cnblogs.com/yaukey/p/unity_lua_memory_leak_trace.html.

Just run "Example.lua" for a quick start.

Compatible with Lua 5.1, 5.2 and 5.3.

Support dumping all lua object reference information in memory sorted by reference count into local txt file.

Support comparing two dumped memory reference file and outputing the increased memory information into another file.

Support filtering the result file by include/exclude the custom keywords.
