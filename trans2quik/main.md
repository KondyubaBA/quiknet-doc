```csharp
using System.Runtime.InteropServices;
using System.Text;

namespace Trans2Quik;

internal class Program
{
    //Подключить dll к quik 
    [DllImport("trans2quik_api.dll", EntryPoint = "TRANS2QUIK_CONNECT", CallingConvention = CallingConvention.StdCall, CharSet = CharSet.Ansi)]
    public static extern long TRANS2QUIK_CONNECT(
        string lpstConnectionParamsString,
        ref long pnExtendedErrorCode,
        StringBuilder lpstrErrorMessage,
        uint dwErrorMessageSize);

    //Отключить dll от quik 
    [DllImport("trans2quik_api.dll", EntryPoint = "TRANS2QUIK_DISCONNECT", CallingConvention = CallingConvention.StdCall, CharSet = CharSet.Ansi)]
    public static extern long TRANS2QUIK_DISCONNECT(
        ref long pnExtendedErrorCode,
        StringBuilder lpstrErrorMessage,
        uint dwErrorMessageSize);

    //Проверка подключения quik к серверу
    [DllImport("trans2quik_api.dll", EntryPoint = "TRANS2QUIK_IS_QUIK_CONNECTED", CallingConvention = CallingConvention.StdCall, CharSet = CharSet.Ansi)]
    public static extern long TRANS2QUIK_IS_QUIK_CONNECTED(
       ref long pnExtendedErrorCode,
       StringBuilder lpstrErrorMessage,
       uint dwErrorMessageSize);


    //Подключить dll к quik 
    public static Trans2QuikStatus ConnectDllToQuik(string connectionParams, out long extendedErrorCode, out string errorMessage)
    {
        extendedErrorCode = -1;
        var bufferMsg = new StringBuilder(256);
        var result = TRANS2QUIK_CONNECT(connectionParams, ref extendedErrorCode, bufferMsg, (uint)bufferMsg.Capacity);
        errorMessage = bufferMsg.ToString();
        return (Trans2QuikStatus)result;
    }

    //Отключить dll от quik 
    public static Trans2QuikStatus DisconnectDllToQuik(out long extendedErrorCode, out string errorMessage)
    {
        extendedErrorCode = -1;
        var bufferMsg = new StringBuilder(256);
        var result = TRANS2QUIK_DISCONNECT(ref extendedErrorCode, bufferMsg, (uint)bufferMsg.Capacity);
        errorMessage = bufferMsg.ToString();
        return (Trans2QuikStatus)result;
    }



     


    public static Trans2QuikStatus IsConnectQuikToServer(out long extendedErrorCode, out string errorMessage)
    {
        extendedErrorCode = -1;
        var bufferMsg = new StringBuilder(256);
        var result = TRANS2QUIK_IS_QUIK_CONNECTED(ref extendedErrorCode, bufferMsg, (uint)bufferMsg.Capacity);
        errorMessage = bufferMsg.ToString();

        return (Trans2QuikStatus)result;
    }



    static void Main(string[] args)
    {

        var ret = IsConnectDllToQuik(out long extendedErrorCode1, out string errorMessage1);

        string connectionParams = @"C:\QUIK_SBER\"; // Замените на ваши параметры подключения

        // Используем обертку для подключения
        Trans2QuikStatus status = ConnectDllToQuik(connectionParams, out long extendedErrorCode, out string errorMessage);


        var ret2 = IsConnectDllToQuik(out long extendedErrorCode2, out string errorMessage2);

        // Вывод результатов
        Console.WriteLine($"Статус подключения: {status}");
        Console.WriteLine($"Код ошибки: {extendedErrorCode}");
        Console.WriteLine($"Сообщение об ошибке: {errorMessage}");
    }
}

```
