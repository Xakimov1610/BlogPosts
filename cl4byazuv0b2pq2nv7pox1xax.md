## C# Action

``Program.cs``

```cs
using ActionTask;

public class Program
{
    // Action yasab IEnumurable ga Extension qo'shishga misol
    public static void Main(string[] args)
    {
        var sonlar = new int[] {1,2,3,4,5};

        sonlar.Print(System.Console.WriteLine);
        
        
        var sonlarString = new [] {"1","2","3","4"};

        sonlarString.PrintF(File.WriteAllText);
    }
}
```

`LinqExtansions.cs`

```cs
namespace ActionTask;

public static class LinqExtansions
{

    // Action yasab IEnumurable ga Extension qo'shishga misol
    public static void Action<T1>(T1 arg1){ }

    public static void Print<T1>(this IEnumerable<T1> toplam, Action<T1> callback)
    { 
        foreach(var item in toplam)
        {
            callback?.Invoke(item);
        }
    }


    // File -----
    public static void Action<T1,T2>(T1 arg1, T2 arg2){ }

    public static void PrintF<T1>(this IEnumerable<T1> toplam, Action<string, T1> callback)
    {
        foreach(var item in toplam)
        {
            callback.Invoke($"{Guid.NewGuid()}.txt",item);
        }
    }
}
```

`Result`

> To’plamdagi har bir element uchun alohida FILE yaratadi
