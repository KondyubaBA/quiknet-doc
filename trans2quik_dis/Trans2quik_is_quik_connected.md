```asm
.text:0000000180005330 ; Exported entry   4. TRANS2QUIK_IS_QUIK_CONNECTED
.text:0000000180005330
.text:0000000180005330 ; =============== S U B R O U T I N E =======================================
.text:0000000180005330
.text:0000000180005330
.text:0000000180005330                 public TRANS2QUIK_IS_QUIK_CONNECTED
.text:0000000180005330 TRANS2QUIK_IS_QUIK_CONNECTED proc near  ; DATA XREF: .rdata:off_180031018↓o
.text:0000000180005330                                         ; .pdata:000000018003769C↓o
.text:0000000180005330
.text:0000000180005330 arg_0           = qword ptr  8
.text:0000000180005330 arg_8           = qword ptr  10h
.text:0000000180005330
.text:0000000180005330                 mov     [rsp+arg_0], rbx
.text:0000000180005335                 mov     [rsp+arg_8], rsi
.text:000000018000533A                 push    rdi
.text:000000018000533B                 sub     rsp, 20h
.text:000000018000533F                 mov     ebx, r8d
.text:0000000180005342                 mov     rdi, rdx
.text:0000000180005345                 mov     rsi, rcx
.text:0000000180005348                 call    sub_180003DE0
.text:000000018000534D                 cmp     cs:dword_180034774, 0
.text:0000000180005354                 jnz     short loc_180005381
.text:0000000180005356                 call    sub_180003DF0
.text:000000018000535B                 lea     r8, aDllIsNotConnec ; "DLL is not connected to QUIK."
.text:0000000180005362                 mov     edx, ebx
.text:0000000180005364                 mov     rcx, rdi
.text:0000000180005367                 call    sub_180003BA0
.text:000000018000536C                 mov     eax, 7
.text:0000000180005371                 mov     rbx, [rsp+28h+arg_0]
.text:0000000180005376                 mov     rsi, [rsp+28h+arg_8]
.text:000000018000537B                 add     rsp, 20h
.text:000000018000537F                 pop     rdi
.text:0000000180005380                 retn
.text:0000000180005381 ; ---------------------------------------------------------------------------
.text:0000000180005381
.text:0000000180005381 loc_180005381:                          ; CODE XREF: TRANS2QUIK_IS_QUIK_CONNECTED+24↑j
.text:0000000180005381                 mov     rcx, cs:qword_1800347A8
.text:0000000180005388                 mov     r9d, ebx
.text:000000018000538B                 mov     r8, rdi
.text:000000018000538E                 mov     rdx, rsi
.text:0000000180005391                 call    sub_180005280
.text:0000000180005396                 mov     ebx, eax
.text:0000000180005398                 call    sub_180003DF0
.text:000000018000539D                 mov     rsi, [rsp+28h+arg_8]
.text:00000001800053A2                 mov     eax, ebx
.text:00000001800053A4                 mov     rbx, [rsp+28h+arg_0]
.text:00000001800053A9                 add     rsp, 20h
.text:00000001800053AD                 pop     rdi
.text:00000001800053AE                 retn
.text:00000001800053AE TRANS2QUIK_IS_QUIK_CONNECTED endp
.text:00000001800053AE
.text:00000001800053AE ; ---------------------------------------------------------------------------
.text:00000001800053AF algn_1800053AF:                         ; DATA XREF: .pdata:000000018003769C↓o
.text:00000001800053AF                 align 10h
```


```cpp
uint32_t dword_180034774; // Объявление переменной
CRITICAL_SECTION CriticalSection;

EnterCriticalSection(&CriticalSection);
if (dword_180034774 != 0) {
    // Переход к коду, который соответствует loc_180005381
    loc_180005381(); // Вызов функции или выполнение кода, соответствующего метке
}
LeaveCriticalSection(&CriticalSection);

void loc_180005381() {
    // Код, который должен выполняться, если dword_180034774 не равно 0
}
```
