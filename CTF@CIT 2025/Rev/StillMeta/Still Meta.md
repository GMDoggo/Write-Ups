# About
We have to deobfuscate lua and get the flag
![](../Images/Pasted%20image%2020250428092911.png)

# Solve
Courtesy of Grok AI and LuaJit I was able to get a script that was able to decode the entire obfuscated lua.

```
-- Load the script as a chunk
local chunk = assert(loadfile("stillmeta.lua"), "Failed to load stillmeta.lua")

-- Create a custom environment with fallback to _G
local env = setmetatable({}, { __index = _G })
env._G = env

-- Ensure print works in the environment
env.print = print

-- Set the environment
setfenv(chunk, env)

-- Execute the chunk
local success, result = pcall(chunk)
if not success then
    error("Error in stillmeta.lua: " .. result)
end

-- If it returned something, print it
if result ~= nil then
    print("Returned value:", result)
end

-- Inspect the environment for any globals set by stillmeta.lua
for key, value in pairs(env) do
    print(key, type(value))
    if type(value) == "string" then
        print("  Value:", value)
    elseif type(value) == "table" then
        for tk, tv in pairs(value) do
            print("  ", tk, tv)
        end
    elseif type(value) == "function" then
        -- If it's a function, try calling it
        local ok, func_result = pcall(value)
        if ok then
            print("  Function result:", func_result)
        end
    end
end
```

- Loads the Script: loadfile("stillmeta.lua") loads the obfuscated script as a chunk.

- Custom Environment: A new environment (env) is created with _G as its fallback, ensuring the script runs in isolation and doesn’t pollute the global environment.

- Executes the Script: pcall(chunk) runs the script, catching any errors.

- Inspects Results: If the script returns a value, it’s printed. The script then iterates over env to print all globals (variables, functions, tables) set by stillmeta.lua. For functions, it attempts to call them and print their results.

- Purpose: This script is likely used for debugging or reverse-engineering stillmeta.lua. By running it in a controlled environment and inspecting the environment, you can see what variables, functions, or data the obfuscated script creates, helping to understand its purpose.

![](../Images/Pasted%20image%2020250428093028.png)