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

void printSQLError(SQLSMALLINT handleType, SQLHANDLE handle) {
    SQLWCHAR sqlState[6], sqlMessage[256];
    SQLINTEGER nativeError;
    SQLSMALLINT messageLength;

    for (SQLSMALLINT i = 1; SQLGetDiagRecW(handleType, handle, i, sqlState, &nativeError, sqlMessage, sizeof(sqlMessage) / sizeof(SQLWCHAR), &messageLength) == SQL_SUCCESS; i++) {
        sqlMessage[sizeof(sqlMessage) / sizeof(SQLWCHAR) - 1] = L'\0'; // Обеспечить нуль-терминатор
        wprintf(L"SQL State: %s, Error: %d, Message: %s\n", sqlState, nativeError, sqlMessage);
    }
}

int test() {
    // Установка кодовой страницы консоли на UTF-8
    //_setmode(_fileno(stdout), _O_U8TEXT);

    SQLHENV hEnv;   // Окружение
    SQLHDBC hDbc;   // Соединение
    SQLHSTMT hStmt; // Дескриптор запроса
    SQLRETURN ret;  // Код возврата

    // Выделение окружения
    ret = SQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &hEnv);
    if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO) {
        printSQLError(SQL_HANDLE_ENV, hEnv);
        return -1;
    }

    // Установка версии ODBC
    ret = SQLSetEnvAttr(hEnv, SQL_ATTR_ODBC_VERSION, (SQLPOINTER)SQL_OV_ODBC3, 0);
    if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO) {
        printSQLError(SQL_HANDLE_ENV, hEnv);
        SQLFreeHandle(SQL_HANDLE_ENV, hEnv);
        return -1;
    }

    // Выделение дескриптора соединения
    ret = SQLAllocHandle(SQL_HANDLE_DBC, hEnv, &hDbc);
    if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO) {
        printSQLError(SQL_HANDLE_ENV, hEnv);
        SQLFreeHandle(SQL_HANDLE_ENV, hEnv);
        return -1;
    }

    // Подключение к базе данных с использованием интегрированной аутентификации
    const SQLWCHAR* connectionString = L"DRIVER={SQL Server};SERVER=localhost\\SQLEXPRESS;DATABASE=quik;Trusted_Connection=yes;";
    ret = SQLDriverConnectW(hDbc, NULL, connectionString, SQL_NTS, NULL, 0, NULL, SQL_DRIVER_COMPLETE);
    if (ret == SQL_SUCCESS || ret == SQL_SUCCESS_WITH_INFO) {
        wprintf(L"Подключение успешно!\n");
    }
    else {
        printSQLError(SQL_HANDLE_DBC, hDbc);
        SQLFreeHandle(SQL_HANDLE_DBC, hDbc);
        SQLFreeHandle(SQL_HANDLE_ENV, hEnv);
        return -1;
    }

    // Выделение дескриптора запроса
    ret = SQLAllocHandle(SQL_HANDLE_STMT, hDbc, &hStmt);
    if (ret != SQL_SUCCESS && ret != SQL_SUCCESS_WITH_INFO) {
        printSQLError(SQL_HANDLE_DBC, hDbc);
        SQLDisconnect(hDbc);
        SQLFreeHandle(SQL_HANDLE_DBC, hDbc);
        SQLFreeHandle(SQL_HANDLE_ENV, hEnv);
        return -1;
    }

    // SQL-запрос для создания таблицы
    const SQLWCHAR* createTableQuery =
        L"CREATE TABLE TestTable ("
        L"ID INT PRIMARY KEY IDENTITY(1,1), "
        L"Name NVARCHAR(100), "
        L"Age INT);";

    // Выполнение запроса на создание таблицы
    ret = SQLExecDirectW(hStmt, createTableQuery, SQL_NTS);
    if (ret == SQL_SUCCESS || ret == SQL_SUCCESS_WITH_INFO) {
        wprintf(L"Таблица 'TestTable' успешно создана!\n");
    }
    else {
        printSQLError(SQL_HANDLE_STMT, hStmt);
    }

    // Освобождение ресурсов
    SQLFreeHandle(SQL_HANDLE_STMT, hStmt);
    SQLDisconnect(hDbc);
    SQLFreeHandle(SQL_HANDLE_DBC, hDbc);
    SQLFreeHandle(SQL_HANDLE_ENV, hEnv);

    return 0;
}


void write_to_sql(const char* name, const char* code, int nsecs, int npars, const char* firmid) {
    
    SQLHENV henv;
    SQLHDBC hdbc;
    SQLHSTMT hstmt;

    // Инициализация ODBC
    SQLAllocHandle(SQL_HANDLE_ENV, SQL_NULL_HANDLE, &henv);
    SQLSetEnvAttr(henv, SQL_ATTR_ODBC_VERSION, (SQLPOINTER)SQL_OV_ODBC3, 0);
    SQLAllocHandle(SQL_HANDLE_DBC, henv, &hdbc);

    // Подключение к базе данных
    SQLWCHAR connStr[] = L"DRIVER={SQL Server};SERVER=localhost\\SQLEXPRESS;DATABASE=quik;Trusted_Connection=yes;";
    SQLDriverConnect(hdbc, NULL, connStr, SQL_NTS, NULL, 0, NULL, SQL_DRIVER_COMPLETE);

    // Подготовка SQL-запроса
    SQLAllocHandle(SQL_HANDLE_STMT, hdbc, &hstmt);
    
    // Буфер для SQL-запроса
    SQLWCHAR sql[512];

    // Форматирование SQL-запроса с использованием swprintf
    swprintf(sql, sizeof(sql) / sizeof(SQLWCHAR),
        L"INSERT INTO Class (firmid, name, code, npars, nsecs) VALUES (N'%hs', N'%hs', N'%hs', %d, %d)",
        firmid, name, code, npars, nsecs);

    // Выполнение SQL-запроса
    SQLExecDirect(hstmt, sql, SQL_NTS);

    // Освобождение ресурсов
    SQLFreeHandle(SQL_HANDLE_STMT, hstmt);
    SQLDisconnect(hdbc);
    SQLFreeHandle(SQL_HANDLE_DBC, hdbc);
    SQLFreeHandle(SQL_HANDLE_ENV, henv);
}


// Функция для обработки элемента таблицы
int process_element(lua_State* L) {
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

    // Запись в SQL
    test();
    write_to_sql(name, code, nsecs, npars, firmid);

    return 0; // Возвращаем 0, так как не возвращаем значения в Lua
}


static const struct luaL_Reg ls_lib[] = {
        {"run", process_element},
        {NULL, NULL}
};

LUALIB_API int luaopen_LuaDll(lua_State* L) {
    luaL_newlib(L, ls_lib);
    return 1;
}



```
