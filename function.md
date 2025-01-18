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
