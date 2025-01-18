#### 1. Получить все ключи table
```lua
function keys_table(data)
    local str = ""
    for key in pairs(data) do
        str = str .. key .. ", "
    end
    if #str > 0 then
        str = str:sub(1, -3)
    end
    return str
end
```

#### 2. Перевести тип table в тип string
```lua
function table_to_string(data)
    local function serialize(value)
        if type(value) == "table" then
            return table_to_string(value)
        else
            return tostring(value)
        end
    end

    local keys = {}
    local result = {}

    for key in pairs(data) do
        table.insert(keys, key)
    end

    for _, key in ipairs(keys) do
        local value = serialize(data[key])
        if type(data[key]) == "table" then
            value = "{" .. value .. "}"
        end
        table.insert(result, key .. "=" .. value)
    end

    return "{" .. table.concat(result, " ") .. "}" 
end

```
