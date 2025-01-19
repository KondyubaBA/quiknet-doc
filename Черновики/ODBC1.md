```c
#define LUA_LIB
#define LUA_BUILD_AS_DLL
#include <locale.h>
#include <lua.h>
#include <lualib.h>
#include <lauxlib.h>
#include <wchar.h>
#include <stdio.h>
#include <stdlib.h>
#include <io.h>
#include <fcntl.h>
#include <windows.h>
#include <sql.h>
#include <sqlext.h>

//#pragma comment(lib, "ws2_32.lib")

const SQLWCHAR connStr[] = L"DRIVER={SQL Server};SERVER=localhost\\SQLEXPRESS;DATABASE=quik;Trusted_Connection=yes;";

SQLHENV _henv;
SQLHDBC _hdbc;


static SQLHENV create_enviroment() {
    SQLHENV henv;
    SQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &henv);
    SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (SQLPOINTER)SQL_OV_ODBC3, 0);

    return henv;
}

static SQLHDBC create_connection() {
    SQLHDBC hdbc;
    SQLAllocHandle(SQL_HANDLE_DBC, _henv, &hdbc);   
    SQLDriverConnect(hdbc, NULL, connStr, SQL_NTS, NULL, 0, NULL, SQL_DRIVER_COMPLETE);

    return hdbc;
}

static int execute_insert(SQLWCHAR* sql) {
    SQLHSTMT hstmt;
    SQLAllocHandle(SQL_HANDLE_STMT, _hdbc, &hstmt);

    SQLExecDirect(hstmt, sql, SQL_NTS);

    SQLFreeHandle(SQL_HANDLE_STMT, hstmt);

    return 0;
}


static void free_resources(SQLHDBC hdbc, SQLHENV henv ) {
    SQLDisconnect(hdbc);
    SQLFreeHandle(SQL_HANDLE_DBC, hdbc);
    SQLFreeHandle(SQL_HANDLE_ENV, henv);
}


static int insert_class(lua_State* L) {
    setlocale(LC_ALL, "");
    luaL_checktype(L, 1, LUA_TTABLE); // Проверяем, что первый аргумент - таблица

    // Извлечение значений из таблицы
    const char* name = NULL;
    const char* code = NULL;
    int nsecs = 0;
    int npars = 0;
    const char* firmid = NULL;

    lua_getfield(L, 1, "firmid");
    firmid = lua_tostring(L, -1);
    lua_pop(L, 1);

    lua_getfield(L, 1, "name");
    name = lua_tostring(L, -1);
    lua_pop(L, 1);

    lua_getfield(L, 1, "code");
    code = lua_tostring(L, -1);
    lua_pop(L, 1);

    lua_getfield(L, 1, "npars");
    npars = (int)lua_tointeger(L, -1);
    lua_pop(L, 1);

    lua_getfield(L, 1, "nsecs");
    nsecs = (int)lua_tointeger(L, -1);
    lua_pop(L, 1);

    // Сохранение в файл
    FILE* file = fopen("output.txt", "a"); // Открываем файл в режиме добавления
    if (file != NULL) {
        fprintf(file, "firmid: %s, name: %s, code: %s, npars: %d, nsecs: %d\n",
            firmid, name, code, npars, nsecs);
        fclose(file); // Закрываем файл
    }
    else {
        fprintf(stderr, "Ошибка при открытии файла для записи.\n");
    }

    SQLWCHAR sql[512];
    swprintf(sql, sizeof(sql) / sizeof(SQLWCHAR),
        L"INSERT INTO Class (firmid, name, code, npars, nsecs) VALUES (N'%hs', N'%hs', N'%hs', %d, %d)",
        firmid, name, code, npars, nsecs);

    // Выполнение SQL-запроса
    execute_insert(sql);

    return 0;
}


static int lua_create_connection(lua_State* L) {
    _henv = create_enviroment();
    _hdbc = create_connection();
    return 0;
}

static int lua_free_resources(lua_State* L) {
    free_resources(_hdbc, _henv);
    return 0;
}


static const struct luaL_Reg ls_lib[] = {
    {"create_connection", lua_create_connection},
    {"insert_class", insert_class},
    {"free_resoures", lua_free_resources},
    {NULL, NULL}
};

LUALIB_API int luaopen_LuaDll(lua_State* L) {
    luaL_newlib(L, ls_lib);
    return 1;
}



```
