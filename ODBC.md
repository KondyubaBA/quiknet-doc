#### 1. Работа с базой
```c
#include <stdio.h>
#include <stdlib.h>
#include <io.h>
#include <fcntl.h>
#include <windows.h>
#include <sql.h>
#include <sqlext.h>

void printSQLError(SQLSMALLINT handleType, SQLHANDLE handle) {
    SQLWCHAR sqlState[6], sqlMessage[256];
    SQLINTEGER nativeError;
    SQLSMALLINT messageLength;

    for (SQLSMALLINT i = 1; SQLGetDiagRecW(handleType, handle, i, sqlState, &nativeError, sqlMessage, sizeof(sqlMessage) / sizeof(SQLWCHAR), &messageLength) == SQL_SUCCESS; i++) {
        sqlMessage[sizeof(sqlMessage) / sizeof(SQLWCHAR) - 1] = L'\0'; // Обеспечить нуль-терминатор
        wprintf(L"SQL State: %s, Error: %d, Message: %s\n", sqlState, nativeError, sqlMessage);
    }
}

int main() {
    // Установка кодовой страницы консоли на UTF-8
    _setmode(_fileno(stdout), _O_U8TEXT);

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

```
