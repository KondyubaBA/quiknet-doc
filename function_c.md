1. Функция си
```c
const char* GetClassList()
{
    lua_getglobal(L_global, "getClassesList");

    if (lua_pcall(L_global, 0, 1, 0) != LUA_OK) {
        sprintf_s(sErr, sizeof(sErr), "Ошибка при вызове функции 'getClassList': %s", lua_tostring(L_global, -1));
        lua_settop(L_global, 0);
        return NULL;
    }
    
    const char* result = luaL_checkstring(L_global, 1);
    lua_settop(L_global, 0);

    return result;
}
```
примечание если вызывать в цикле возникает ошибка - неизвестно почему. Это ошибка именно в LUA.

2. Функция си
```c
int Message(const char* message, int icon) {
    lua_getglobal(L_global, "message");
    lua_pushstring(L_global, message);
    lua_pushnumber(L_global, icon);

    if (lua_pcall(L_global, 2, 1, 0) != LUA_OK) {
        sprintf_s(sErr, sizeof(sErr), "Ошибка при вызове функции 'message': %s", lua_tostring(L_global, -1));
        lua_settop(L_global, 0);
        return 0;
    }

    int result = (int)luaL_checknumber(L_global, -1);
    lua_settop(L_global, 0);

    return result;
}
```

3.Вспомогательные функции
```c
// Вспомогательная функция для извлечения строковых значений
const char* get_field_string(lua_State* L, const char* field) {
    lua_getfield(L, 1, field);
    const char* value = lua_tostring(L, -1);
    lua_pop(L, 1);
    return value;
}

// Вспомогательная функция для извлечения целых значений
int get_field_integer(lua_State* L, const char* field) {
    lua_getfield(L, 1, field);
    int value = (int)lua_tointeger(L, -1);
    lua_pop(L, 1);
    return value;
}

// Вспомогательная функция для извлечения значений типа float
float get_field_float(lua_State* L, const char* field) {
    lua_getfield(L, 1, field); // Извлекаем значение по ключу field
    float value = (float)lua_tonumber(L, -1); // Преобразуем значение в float
    lua_pop(L, 1); // Удаляем значение со стека
    return value; // Возвращаем извлеченное значение
}
```
