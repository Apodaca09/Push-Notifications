# Notificaciones en apps .NET MAUI utilizando Firebase

Video en youtube: https://youtu.be/pimfkp5XDWc?si=OFtLAZmeu194PLFb

## Requisitos 
* Utilizar .Net en los proyectos

## App que recibira notificaciones (App Negocios en el ejemplo)
* Instalar el paquete nuget Plugin.Firebase versión 2.0.10
* Instalar el paquete Plugin.Firebase.Crashlytics versión 2.0.2
* Especificar el ItemGroup para incluir Xamarin.AndroidX.Preference cuando sea .net8.0-android especificar version 1.2.1.3

## App que envia o genera la notificación (App Clientes en el ejemplo)
* Instalar el paquete nuget FirebaseAdmin

## Pasos App Negocios
1. Crear el proyecto de aplicación utilizando .Net 8
2. Definir un nombre para la app del negocio
3. Ir al administrador de paquetes nuget e instalar Plugin.Firebase versión 2.0.10
4. Instalar Plugin.Firebase.Crashlytics version 2.0.2
5. Ir al archivo del proyecto y especificar el ItemGroup de Xamarin
```XAML
<ItemGroup Condition="'$(TargetFramework)' == 'net8.0-android'">
	<PackageReference Include="Xamarin.AndroidX.Preference">
		<Version>1.2.1.3</Version>
	</PackageReference>
</ItemGroup>
```
6. Ir a https://firebase.google.com/?hl=es
7. Seleccionar Go to console
8. Agregar un nuevo proyecto
9. Definir un nombre para el proyecto en Firebase
10. Registrar la aplicacion del negocio en Firebase, ir al archivo del proyecto e identificar entre las etiquetas
```XAML
<ApplicationId></ApplicationId>
```
 copiar el id y pegarlo, poner un nombre a la app en el recuadro siguiente.
12. Después dar siguiente y siguiente y descargar el archivo json, guardarlo en una ubicación conocida.
13. Ir al proyecto del negocio en visual studio, agregar el archivo json ```google-service.json``` arrastrandolo al proyecto.
14.  Abrir las propiedades del archivo ```google-service.json``` y cambiar en Accion de compilacion a ```GoogleServiceJson```
15.  Ir a la carpeta ```Platforms>Android>Resources>values``` y agregar un nuevo elemento, ir a Datos y seleccionar archivo XML
16.  Nombrar el archivo como ```strings.xml```, agregar al archivo lo siguiente:
```XAML
<resources>
	<string name="com.google.firebase.crashlytics.mapping_file_id">none</string>
</resources>
```
despues en las propiedades del archivo en Accion de compilacion seleccionar ```AndroidResource```

16. Ir a la carpeta ```Platforms>Android``` y abrir el archivo ```MainActivity.cs```, agregar las siguientes librerias:
```C#
using Android.App;
using Android.Content;
using Android.Content.PM;
using Android.OS;
using Plugin.Firebase.CloudMessaging;
```
17. Dentro de la clase ```MainActivity: MauiAppCompactActivity```, copiar el codigo siguiente:
``` C#
protected override void OnCreate(Bundle savedInstanceState)
{
    base.OnCreate(savedInstanceState);
    HandleIntent(Intent);
    CreateNotificationChannelIfNeeded();
}

protected override void OnNewIntent(Intent intent)
{
    base.OnNewIntent(intent);
    HandleIntent(intent);
}

private static void HandleIntent(Intent intent)
{
    FirebaseCloudMessagingImplementation.OnNewIntent(intent);
}

private void CreateNotificationChannelIfNeeded()
{
    if (Build.VERSION.SdkInt >= BuildVersionCodes.O)
    {
        CreateNotificationChannel();
    }
}

private void CreateNotificationChannel()
{
    var channelId = $"{PackageName}.general";
    var notificationManager = (NotificationManager)GetSystemService(NotificationService);
    var channel = new NotificationChannel(channelId, "General", NotificationImportance.Default);
    notificationManager.CreateNotificationChannel(channel);
    FirebaseCloudMessagingImplementation.ChannelId = channelId;
    FirebaseCloudMessagingImplementation.SmallIconRef = Resource.Drawable.splash;
}
```
18. Ir al archivo ```MauiProgram.cs``` asegurarse de incluir las siguientes librerias:
``` C#
using Plugin.Firebase.Auth;
using Plugin.Firebase.Bundled.Shared;
using Microsoft.Extensions.Logging;
using Plugin.Firebase.Crashlytics;
using Microsoft.Maui.LifecycleEvents;
```
19. Entre las librerias y el namespace poner el condicional:
``` C#
#if IOS
using Plugin.Firebase.Bundled.Platforms.iOS;
#else
using Plugin.Firebase.Bundled.Platforms.Android;
#endif
```
20. Debajo de la funcion CreateMauiApp, pegar las siguientes funciones:
``` c#
 private static MauiAppBuilder RegisterFirebaseServices(this MauiAppBuilder builder)
        {
            builder.ConfigureLifecycleEvents(events =>
            {
#if IOS
            events.AddiOS(iOS => iOS.FinishedLaunching((app, launchOptions) => {
                CrossFirebase.Initialize(CreateCrossFirebaseSettings());
                return false;
            }));
#else
                events.AddAndroid(android => android.OnCreate((activity, _) =>
                    CrossFirebase.Initialize(activity, CreateCrossFirebaseSettings())));
                CrossFirebaseCrashlytics.Current.SetCrashlyticsCollectionEnabled(true);
#endif
            });

            builder.Services.AddSingleton(_ => CrossFirebaseAuth.Current);
            return builder;
        }

        private static CrossFirebaseSettings CreateCrossFirebaseSettings()
        {
            return new CrossFirebaseSettings(isAuthEnabled: true,
            isCloudMessagingEnabled: true, isAnalyticsEnabled: true);
        }
```
21. Ir a ```Platforms>Android``` y abrir el archivo ```AndroidManifest.xml```, dando clic derecho>abrir con> Editor XML (texto)
22. Pegar entre las etiquetas ```<application></application>``` y despues guardar lo siguiente:
``` XAML
<receiver android:name="com.google.firebase.iid.FirebaseInstanceIdInternalReceiver" android:exported="false" />
<receiver android:name="com.google.firebase.iid.FirebaseInstanceIdReceiver" android:exported="true" android:permission="com.google.android.c2dm.permission.SEND">
	<intent-filter>
		<action android:name="com.google.android.c2dm.intent.RECEIVE" />
		<action android:name="com.google.android.c2dm.intent.REGISTRATION" />
		<category android:name="${applicationId}" />
	</intent-filter>
</receiver>
```
23. Ir al archivo ```MainPage.cs```  agregar la liberia:
``` C#
using Plugin.Firebase.CloudMessaging;
```

25. Eliminar el codigo que hay en el evento OnCounterClicked, hacer el avento asincrono declarando ```async``` entre ```private``` y ```void```, despues pegar el codigo siguiente dentro del evento:
``` C#
await CrossFirebaseCloudMessaging.Current.CheckIfValidAsync();
var token = await CrossFirebaseCloudMessaging.Current.GetTokenAsync();
```
25. Poner un punto de interrrupcion en la linea de var token, guardar y ejecutar la aplicacion.
26. Una vez iniciada la app, hacer clic en el boton y debuggearhasta llegar ala linea del token, copiar el token y guardarlo
en un archivo como ```token.txt```.
26. Dar continuar a la app, ir a Firebase en el apartado de accesos directos del proyecto, ingresar en ```Messaging```
27. Dar en crear la primera campaña, seleccionar ```Mensajes de Firebase Notifications``` y despues en crear.
28. Rellenar el formulario para la notificacion, despues dar en enviar mensaje de prueba, ir al archivo del token y copiarlo, para pegarlo en Agregar token de registro de FMC y despues clic en el icono de agregar. Una vez agregado dar clic en probar
la notificacion no debe tardar en llegar, hasta ahora la app del negocio esta configurada para recibir notificaciones de Firebase, una vez obtenido el token se puede eliminar el codigo de ```OnCounterClicked```

## Los pasos siguientes son para enviar las notificaciones desde la app del cliente

1. Crear nuevo proyecto en .Net 8
2. Ir al administrador de paquetes nuget y en examinar buscar FirebaseAdmin, instalar la version 2.4.1, cerrar el administrador
3. Recuerdan el archivo ```token.txt```, lo buscaremos donde lo guardamos para cortarlo y lo movemos en el proyecto en ```Resources>Raw```
4.  Iremos a la consola de Firebase, damos clic en el icono de engranaje y seleccionamos Configuracion del proyecto, despues vamos al apartado de cuentas de servicio y damos clic en el boton de Generar nueva clave privada, descargar el archivo json, y renombrarlo como ```admin_sdk.json.``` Moverlo a ```Resources>Raw```
5.  Abrir el archivo ```MainPage.cs```, agregar las librerias siguientes:
``` C#
using FirebaseAdmin;
using FirebaseAdmin.Messaging;
using Google.Apis.Auth.OAuth2;
```
6. Pegar las funciones siguientes en ```MainPage.cs```:
``` C#
private async Task<string> GetToken()
{
    var stream = await FileSystem.OpenAppPackageFileAsync("token.txt");
    var reader = new StreamReader(stream);
    string strToken = reader.ReadToEnd();
    return strToken;
}

private async void ReadFireBaseAdminSdk()
{
    var stream = await FileSystem.OpenAppPackageFileAsync("admin_sdk.json");
    var reader = new StreamReader(stream);

    var jsonContent = reader.ReadToEnd();

    if (FirebaseMessaging.DefaultInstance == null)
    {
        FirebaseApp.Create(new AppOptions()
        {
            Credential = GoogleCredential.FromJson(jsonContent)
        });
    }
}
```
7. Eliminar el contenido del evento ```OnCounterClicked``` y pegar el codigo:
``` c#
string _deviceToken = Convert.ToString(await GetToken());
var androidNotificationObject = new Dictionary<string, string>();
androidNotificationObject.Add("NavigationID", "2");

var pushNotificationRequest = new PushNotificationRequest
{
    notification = new NotificationMessageBody
    {
        title = "Notificación",
        body = "Tienes un nuevo pedido"
    },
    data = androidNotificationObject,
    registration_ids = new List<string>() { _deviceToken }
};

var obj = new Message
{
    Token = _deviceToken,
    Notification = new Notification
    {
        Title = "Title",
        Body = "Message Body",
    },
    Data = androidNotificationObject,
};
var response = await FirebaseMessaging.DefaultInstance.SendAsync(obj);
```
8. Debajo de ```InitializeComponent();``` poner ```ReadFireBaseAdminSdk();```
9. Agregar una nueva clase y llamarla PushNotificationRequest, dentro de esta clase agregar:
``` C#
public List<string> registration_ids { get; set; } = new List<string>();
public NotificationMessageBody notification { get; set; }
public object data { get; set; }
```
10.  En el mismo archivo declarar la clase siguiente:
``` C#
public class NotificationMessageBody
 {
     public string title { get; set; }
     public string body { get; set; }
 }
```
11. Ejecutamos la app y probamos si se recibe la notificacion.

## Linea para corregir el error XA0129
``` XAML
<EmbedAssembliesIntoApk>true</EmbedAssembliesIntoApk>
