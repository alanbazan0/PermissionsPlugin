## Permissions Plugin for Xamarin

Simple cross platform plugin to request and check permissions.

Want to read about the creation, checkout my [in-depth blog post](http://motzcod.es/post/133939517717/simplified-ios-android-runtime-permissions-with).

### Setup
* Available on NuGet: http://www.nuget.org/packages/Plugin.Permissions [![NuGet](https://img.shields.io/nuget/v/Plugin.Permissions.svg?label=NuGet)](https://www.nuget.org/packages/Plugin.Permissions/)
* Install into your PCL project and Client projects.
* Development NuGet: https://ci.appveyor.com/nuget/permissionsplugin

**Platform Support**

|Platform|Version|
| ------------------- | :-----------: |
|Xamarin.iOS|iOS 8+|
|Xamarin.Android|API 14+|
|Windows 10 UWP(Beta)|10+|

*See platform notes below

Build Status: [![Build status](https://ci.appveyor.com/api/projects/status/n0vn5715cx5f7rpy?svg=true)](https://ci.appveyor.com/project/JamesMontemagno/permissionsplugin)

### Android specific in your BaseActivity or MainActivity (for Xamarin.Forms) add this code:
```csharp
public override void OnRequestPermissionsResult(int requestCode, string[] permissions, [GeneratedEnum] Android.Content.PM.Permission[] grantResults)
{
    base.OnRequestPermissionsResult(requestCode, permissions, grantResults);
    PermissionsImplementation.Current.OnRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

You MUST set your Target version to API 25+ and Compile against API 25+:

### iOS Specific
When building against the iOS 10 SDK (Xcode 8) please be aware of the platform privacy changes. Based on what permissions you are using, you must add information into your info.plist. Please read the [following blog for more information](https://blog.xamarin.com/new-ios-10-privacy-permission-settings/). 

Due to API usage it is required to add the Calendar permission :(

<key>NSCalendarsUsageDescription</key>
<string>Needs Calendar Permission</string>

Even though your app may not use calendar at all. I am looking into a workaround for this in the future.


### API Usage

Call **CrossPermissions.Current** from any project or PCL to gain access to APIs.

**Should show request rationale**
```csharp
/// <summary>
/// Request to see if you should show a rationale for requesting permission
/// Only on Android
/// </summary>
/// <returns>True or false to show rationale</returns>
/// <param name="permission">Permission to check.</param>
Task<bool> ShouldShowRequestPermissionRationaleAsync(Permission permission);
```

**CheckPermissionStatus**
```csharp
/// <summary>
/// Determines whether this instance has permission the specified permission.
/// </summary>
/// <returns><c>true</c> if this instance has permission the specified permission; otherwise, <c>false</c>.</returns>
/// <param name="permission">Permission to check.</param>
Task<PermissionStatus> CheckPermissionStatusAsync(Permission permission);
```

**RequestPermissions**
```csharp
/// <summary>
/// Requests the permissions from the users
/// </summary>
/// <returns>The permissions and their status.</returns>
/// <param name="permissions">Permissions to request.</param>
Task<Dictionary<Permission, PermissionStatus>> RequestPermissionsAsync(params Permission[] permissions);
```

### In Action
Here is how you may use it with the Geolocator Plugin:

```csharp
try
{
    var status = await CrossPermissions.Current.CheckPermissionStatusAsync(Permission.Location);
    if (status != PermissionStatus.Granted)
    {
        if(await CrossPermissions.Current.ShouldShowRequestPermissionRationaleAsync(Permission.Location))
        {
            await DisplayAlert("Need location", "Gunna need that location", "OK");
        }

        var results = await CrossPermissions.Current.RequestPermissionsAsync(Permission.Location);
		//Best practice to always check that the key exists
		if(results.ContainsKey(Permission.Location))
        	status = results[Permission.Location];
    }

    if (status == PermissionStatus.Granted)
    {
        var results = await CrossGeolocator.Current.GetPositionAsync(10000);
        LabelGeolocation.Text = "Lat: " + results.Latitude + " Long: " + results.Longitude;
    }
    else if(status != PermissionStatus.Unknown)
    {
        await DisplayAlert("Location Denied", "Can not continue, try again.", "OK");
    }
}
catch (Exception ex)
{

    LabelGeolocation.Text = "Error: " + ex;
}
```

### Available Permissions
```csharp
/// <summary>
/// Permission group that can be requested
/// </summary>
public enum Permission
{
    /// <summary>
    /// The unknown permission only used for return type, never requested
    /// </summary>
    Unknown,
    /// <summary>
    /// Android: Calendar
    /// iOS: Calendar (Events)
    /// UWP: None
    /// </summary>
    Calendar,
    /// <summary>
    /// Android: Camera
    /// iOS: Photos (Camera Roll and Camera)
    /// UWP: None
    /// </summary>
    Camera,
    /// <summary>
    /// Android: Contacts
    /// iOS: AddressBook
    /// UWP: ContactManager
    /// </summary>
    Contacts,
    /// <summary>
    /// Android: Fine and Coarse Location
    /// iOS: CoreLocation (Always and WhenInUse)
    /// UWP: Geolocator
    /// </summary>
    Location,
    /// <summary>
    /// Android: Microphone
    /// iOS: Microphone
    /// UWP: None
    /// </summary>
    Microphone,
    /// <summary>
    /// Android: Phone
    /// iOS: Nothing
    /// UWP: None
    /// </summary>
    Phone,
    /// <summary>
    /// Android: Nothing
    /// iOS: Photos
    /// UWP: None
    /// </summary>
    Photos,
    /// <summary>
    /// Android: Nothing
    /// iOS: Reminders
    /// UWP: None
    /// </summary>
    Reminders,
    /// <summary>
    /// Android: Body Sensors
    /// iOS: CoreMotion
    /// UWP: Device Access Sensor Class
    /// </summary>
    Sensors,
    /// <summary>
    /// Android: Sms
    /// iOS: Nothing
    /// UWP: None
    /// </summary>
    Sms,
    /// <summary>
    /// Android: External Storage
    /// iOS: Nothing
    /// UWP: None
    /// </summary>
    Storage
    /// <summary>
    /// Android: Microphone
    /// iOS: Speech
    /// UWP: None
    /// </summary>
    Speech
}
```
Read more about android permissions: http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous


### IMPORTANT
#### Android:

You still need to request the permissions in your AndroidManifest.xml. Also ensure your MainApplication.cs was setup correctly from the CurrentActivity Plugin.

#### Windows 10 UWP
UWP has a limited set of supported permissions. You can see the documentation above, but current support: Contacts, Location, and Sensors.

#### Contributors
* Icon thanks to [Jérémie Laval](https://github.com/garuma)

Thanks!

#### License
Licensed under main repo license(MIT)
