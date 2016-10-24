# Windows Iot Core - Windows UWP - SignalR - Azure - Demo

## Getting Started

Willkommen zum Demo-Projekt zum Vortrag "Fun with Windows 10 IoT Core".

Infos zur .NET User Group Paderborn findet Ihr hier:

http://dotnet-paderborn.azurewebsites.net/


In diesem Projekt ziege ich euch, wie man mit dem RaspberryPi GPIOs ansprechen kann (sowohl lesend als auch schreibend),
wie man ein Foto mit einer Webcam lokal speichern kann und wie man dies nun in die Azure hochladen kann.
Dabei wird in diesem Demo SignalR und ASP.NET WebAPI verwendet.
SignalR wird verwendet um Kommandos an den RaspberryPi zu schicken und die WebAPI um Bilder hoch- und runterladen zu k�nnen.

Kingt nach viel! Is aber ganz einfach :-) ...


## GPIO - LED an!

In diesem Code Snippet zeige ich euch, wie Ihr eine LED leuchten lassen k�nnt.
Es werde Licht...

```
using Windows.Devices.Gpio;

int pinNumber = 22;

var controller = GpioController.GetDefault();
var pin = this.controller.OpenPin(pinNumber);
pin.SetDriveMode(GpioPinDriveMode.Output);
pin.Write(GpioPinValue.Low);
```

## UWP - Camera ansprechen - cheeeeeese!

Nicht nur 3,3 Volt auf Pin, SPI, I�C und UART k�nnt Ihr mit Euren Raspberry ansprechen, auch alle
anderen UWP-Feature k�nnt Ihr nutzen.
Unter anderem eben auch Eure Webcam.
Aber Achtung! Windows Iot auf dem Raspberry hat nur einen eingeschr�nkten Treiber-Support.
Na wer kennt das noch aus alten Linux-Tagen ;-)

Eine Liste von unterst�tzen USB-Ger�ten findet Ihr hier:
https://developer.microsoft.com/en-us/windows/iot/docs/hardwarecompatlist

Wir spreche ich nun meine Webcam an?

```
using Windows.Media.Capture;
using Windows.Media.MediaProperties;
using Windows.UI.Xaml.Media.Imaging;

var file = await ApplicationData.Current.LocalFolder.CreateFileAsync("localFile.jpg", CreationCollisionOption.GenerateUniqueName);
var imgFormat = ImageEncodingProperties.CreateJpeg();
imgFormat.Height = this.height;
imgFormat.Width = this.width;

var mediaCapture = new MediaCapture();
await mediaCapture.InitializeAsync();
await mediaCapture.CapturePhotoToStorageFileAsync(imgFormat, file);

```

Ihr solltet nat�rlich vorher pr�fen, ob an dem Windows-Device auch eine Webcam angeschlossen ist.

Alternative k�nnt Ihr auch das Bild direkt in einen MemoryStream schreiben.

```
public IAsyncAction CapturePhotoToStreamAsync(
  ImageEncodingProperties type, 
  IRandomAccessStream stream
)
```


### SignalR - Websockets wenn es geht!

In diesem Demo wurde gezeigt, wie der Raspberry aus dem Internet gesteuert werden kann.
Azure IoT Hub ist sicherlich eine spannende L�sung.
Ich verwende in diesem Demo SignalR. 
Wie wir das auch in der Usergroup diskutiert haben, bieten Cloud-Dienste (im konkreten Azure IoT Hub) Out-of-the-box-L�sungen an.
Mann muss sich aber bewusst sein, dass man sich an einen spezifischen Anbieter bindet.
Eine Migration auf einen anderen Cloud-Anbieter k�nnte aufwendig werden.

Und nun zum Code:

```
using System;
using System.Web;
using Microsoft.AspNet.SignalR;
namespace DotNetUserGroupPaderborn
{
    public class PiHub : Hub
    {
        public void Send(string name)
        {
            // In der realen Welt sollte man sicherlich nicht nur eine String �bermitteln,
			// sondern spezifischere Kommandos mit JSON kodieren.
			// Wir reduzieren es in diesen Demo mal auf das N�tigste.
			// Broadcast an alle!
            Clients.All.broadcastMessage(name);
        }
    }
}
```

In der OWIN-Pipeline muss SignalR jetzt nur noch eingeh�ngt werden:

```
using Microsoft.Owin;
using Owin;

namespace DotNetUserGroupPaderborn
{
    public class Startup
    {
        public void Configuration(IAppBuilder app)
        {
			app.MapSignalR();
        }
    }
}
```

Und so k�nnt Ihr euch mit C#/.NET auf den SingalR-Hub anmelden:

```

var connection = new HubConnection("http://localhost:57624/") 
{ 
	TraceLevel = TraceLevels.All, 
	TraceWriter = Console.Out 
};

hubProxy = connection.CreateHubProxy("PiHub");
hubProxy.On<string>("broadcastMessage", LogMessage);

...

public static void LogMessage(string message)
{
	Console.WriteLine($"Data from SignalR recieved: {message}");
}

```

Viel Spass beim ausprobieren!










