## C# Func

`Program.cs`

```cs
using FancLinq;

public class Program
{
    public static void Main(string[] args)
    {
        var nums = new int[] {1,2,3,4,5,6,7};
        var str = nums.Select((value) => SelectMethod(value))
                .ToList();
            
        str.ForEach(System.Console.WriteLine);

        var str2 = nums.Select((value1, value2) => SelectMethod(value1, value2))
                .ToList();

        str2.ForEach(System.Console.WriteLine);

        // MY Select ------------------

        var str3 = nums.MySelect(SelectMethod)
                .ToList();
        str3.ForEach(System.Console.WriteLine);

    }


    public static string SelectMethod(int arg1)
    {
        return string.Format($"sonlar : {arg1}");
    }

    public static string SelectMethod(int arg1, int arg2)
    {
        return string.Format($"son : {arg1} index : {arg2}");
    }

}
```

`LinqExtensions.cs`

```cs
namespace FancLinq;

public static class LinqExtensions
{
    public static IEnumerable<TResult> MySelect<T1,TResult>(this IEnumerable<T1> collection, Func<T1,TResult> callback)
    {
        // var result = new List<TResult>();
        foreach(var num in collection)
        {
            // result.Add(callback.Invoke(num));
            yield return callback.Invoke(num);
        }
        // return result;
    }
}
```

`Result`

```cs
sonlar : 1
sonlar : 2
sonlar : 3
sonlar : 4
sonlar : 5
sonlar : 6
sonlar : 7
son : 1 index : 0
son : 2 index : 1
son : 3 index : 2
son : 4 index : 3
son : 5 index : 4
son : 6 index : 5
son : 7 index : 6
sonlar : 1
sonlar : 2
sonlar : 3
sonlar : 4
sonlar : 5
sonlar : 6
sonlar : 7
```