## C# Event Practices ''VideoConverter"

`Program.cs`
```cs
using VideoConverter;

var bytes = System.Text.Encoding.ASCII.GetBytes("Hello world");

var convert = new VideoConvert(bytes);

convert.ConversionStarted += HandleConversionStarted;

convert.ConversionFinished += HandleConversionFinished;

await convert.ConvertAsync();

void HandleConversionFinished(object sender, ConversionFinishedEventArgs e)
{
    System.Console.WriteLine($"Finished : {e.ElapsedTime.TotalMilliseconds}");
    System.Console.WriteLine($"Conversion data : {System.Convert.ToBase64String(e.ByteVideo)}");

    // agar VideoConvert.cs da _bytes ni shunda hech narsa qilmasdan qaytarsak tekshirish imkonimiz bo'ladi

    System.Console.WriteLine($"Bytni Stringga pars qilamiz : {System.Text.Encoding.ASCII.GetString(e.ByteVideo)}");
}

void HandleConversionStarted(object sender, ConversionStartedEventArgs e)
{
    System.Console.WriteLine($"Conversion started : {e.StartTime:HH:mm:ss.fff}");
}
```

`VideoConvert.cs`

```cs
namespace VideoConverter;
public class VideoConvert
{
   private byte[] _bytes;
   public VideoConvert(byte[] bytes)
   {
       _bytes = bytes;
   } 

   public event EventHandler<ConversionStartedEventArgs> ConversionStarted;
   public event EventHandler<ConversionFinishedEventArgs> ConversionFinished;

   public async Task ConvertAsync()
   {
       var startTime = DateTime.Now;

       ConversionStarted?.Invoke(this, new ConversionStartedEventArgs(startTime));

       var convertedByte = new byte[_bytes.Length];

       await Task.Run(() =>
       {
           for(int i = 0; i < _bytes.Length; i++)
           {
               convertedByte[i] = (byte)(_bytes[i] * 10 % 255);
               Thread.Sleep(100);
           }
       });

       var finishTime = DateTime.Now;

       ConversionFinished?.Invoke(this, new ConversionFinishedEventArgs(finishTime - startTime, convertedByte));
   }
}
```

`ConversionStartedEventArgs.cs`

```cs
namespace VideoConverter;
public class ConversionStartedEventArgs : EventArgs
{
    public DateTime StartTime { get; set; }
    public ConversionStartedEventArgs(DateTime startTime)
    {
        StartTime = startTime;
    }
}
```

`ConversionFinishedEventArgs.cs`

```cs
namespace VideoConverter;
public class ConversionFinishedEventArgs : EventArgs
{
    public TimeSpan ElapsedTime { get; set;}
    public byte[] ByteVideo { get; set; }

    public ConversionFinishedEventArgs(TimeSpan elapsedTime, byte[] byteVideo)
    {
        ByteVideo = byteVideo;
        ElapsedTime = elapsedTime;
    }
}
```

`Result`

```cs
Conversion started : 18:05:18.906
Finished : 1215.91
Conversion data : 0vU8PFpBqlp4POs=
Bytni Stringga pars qilamiz : ??<<ZA?Zx<?
```