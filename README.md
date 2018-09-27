### Objective
The main objective of this blog post is to help you create iOS Native Popups Using Unity

## Step 1 : Introduction
A popup is one small screen or some alert message which asks a user to take some action.

**Here, we will be creating three types of pop-ups:**

- **Message Popup**     Single Action
- **Confirmation Popup**    Two Actions
- **Rate-US Popup**     Three Actions

**Now lets create some simple popup.**

## Step 2 : Setup scene in Unity
Create new Unity project and save scene to your assets folder.

**Create three buttons for three popups:**
![](http://www.theappguruz.com/app/uploads/2016/05/unity-setup.png)

## Step 3 : Create script and assign all button reference
Create script and name it as you wish. I have named it **PopupView.cs**. Now let's write some code for adding event listener on button click.

Create three methods for each button and assign reference on button click event. Create enum for message state returned from Android native dialog actions.

```csharp
public enum MessageState
{
    OK,
    YES,
    NO,
    RATED,
    REMIND,
    DECLINED,
    CLOSED
}
    #region PUBLIC_VARIABLES
    // App ID to rate it
    [Tooltip("market://details?id=BUNDLE-ID")]
    public string gameLink = "market://details?id=com.tag.tabletennis3D";
    #endregion 
#region BUTTON_EVENT_LISTENER
    // Dialog Button click event
    public void OnDialogPopUp()
    {
        NativeDialog dialog = new NativeDialog("TheAppGuruz", "Do you wants to know about TheAppGuruz");
        dialog.SetUrlString("http://theappguruz.com/");
        dialog.init();
    }
    // Rate Button click event
    public void OnRatePopUp()
    {
        NativeRateUS ratePopUp = new NativeRateUS("Like this game?", "Please rate to support future updates!");
        ratePopUp.SetAppLink(gameLink);
        ratePopUp.InitRateUS();    
    }
    // Message Button click event
    public void OnMessagePopUp()
    {
        NativeMessage msg = new NativeMessage("TheAppGuruz", "Welcome To TheAppGuruz");
    }
    #endregion 
```

**Now lets register delegate event listener for native popup actions.**

```csharp
#region UNITY_DEFAULT_CALLBACKS
void OnEnable()
{
    // Register all Delegate event listener
    IOSRateUsPopUp.onRateUSPopupComplete += OnRateUSPopupComplete;
    IOSDialog.onDialogPopupComplete += OnDialogPopupComplete;
    IOSMessage.onMessagePopupComplete += OnMessagePopupComplete;
}
void OnDisable()
{
    // Deregister all Delegate event listener
    IOSRateUsPopUp.onRateUSPopupComplete -= OnRateUSPopupComplete;
    IOSDialog.onDialogPopupComplete -= OnDialogPopupComplete;
    IOSMessage.onMessagePopupComplete -= OnMessagePopupComplete;
}
#endregion
#region DELEGATE_EVENT_LISTENER
// Raise when click on any button of rate popup
void OnRateUSPopupComplete(MessageState state)
{
    switch (state)
    {
        case MessageState.RATED:
        Debug.Log("Rate Button pressed");
        break;
        case MessageState.REMIND:
        Debug.Log("Remind Button pressed");
        break;
        case MessageState.DECLINED:
        Debug.Log("Declined Button pressed");
        break;
    }
}
// Raise when click on any button of Dialog popup
void OnDialogPopupComplete(MessageState state)
{
    switch (state)
    {
        case MessageState.YES:
        Debug.Log("Yes button pressed");
        break;
        case MessageState.NO:
        Debug.Log("No button pressed");
        break;
    }
}
// Raise when click on ok button of message popup
void OnMessagePopupComplete(MessageState state)
{
    Debug.Log("Ok button Clicked");
}
#endregion
```

## Step 4 : Create script for interact with Objective-c code

Now, Create a script to directly make interaction with iOS code – (Objective-C) named **IOSNative.cs**

```csharp
#define DEBUG_MODE
using UnityEngine;
using System.Collections;
#if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
using System.Runtime.InteropServices;
#endif
public class IOSNative
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    [DllImport("__Internal")]
    private static extern void _TAG_ShowRateUsPopUp(string title, string message, string rate, string remind, string declined);
    [DllImport("__Internal")]
    private static extern void _TAG_ShowDialog(string title, string message, string yes, string no);
    [DllImport("__Internal")]
    private static extern void _TAG_ShowMessage(string title, string message, string ok);
    [DllImport("__Internal")]
    private static extern void _TAG_RedirectToAppStoreRatingPage(string appId);
    [DllImport("__Internal")]
    private static extern void _TAG_RedirectToWebPage(string urlString);
    [DllImport("__Internal")]
    private static extern void _TAG_DismissCurrentAlert();
    #endif
public static void showRateUsPopUP(string title, string message, string rate, string remind, string declined)
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_ShowRateUsPopUp(title, message, rate, remind, declined);
    #endif
}
public static void showDialog(string title, string message, string yes, string no)
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_ShowDialog(title, message, yes, no);
    #endif
}
public static void showMessage(string title, string message, string ok)
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_ShowMessage(title, message, ok);
    #endif
}
public static void RedirectToAppStoreRatingPage(string appleId)
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_RedirectToAppStoreRatingPage(appleId);
    #endif
}
public static void RedirectToWebPage(string urlString)
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_RedirectToWebPage(urlString);
    #endif
}
public static void DismissCurrentAlert()
{
    #if (UNITY_IPHONE && !UNITY_EDITOR) || DEBUG_MODE
    _TAG_DismissCurrentAlert();
    #endif
}
}
```

## Step 5 : Create scripts to create different popups
As I mentioned above we will be creating three types of pop-ups, Let's create scripts to create different popups

#### MESSAGE POPUP
**A)** Create **NativeMessage.cs** for Basic setup of simple message popup:

```csharp
public class NativeMessage
{
#region PUBLIC_FUNCTIONS
public NativeMessage(string title, string message)
{
init(title, message, "Ok");
}
public NativeMessage(string title, string message, string ok)
{
init(title, message, ok);
}
private void init(string title, string message, string ok)
{
#if UNITY_IPHONE
IOSMessage.Create(title, message, ok);
#endif
}
#endregion
}
```

**B)** Create **IOSMessage.cs** for simple message popup:

```csharp
public class IOSMessage : MonoBehaviour
{               
#region DELEGATE
public delegate void OnMessagePopupComplete(MessageState state);
public static event OnMessagePopupComplete onMessagePopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnMessagePopupComplete(MessageState state)
    {
        if (onMessagePopupComplete != null)
            onMessagePopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string ok;
    #endregion
    #region PUBLIC_FUNCTIONS
    public static IOSMessage Create(string title, string message)
    {
        return Create(title, message, "Ok");
    }
    public static IOSMessage Create(string title, string message, string ok)
    {
        IOSMessage dialog;
        dialog = new GameObject("IOSMessagePopUp").AddComponent<IOSMessage>();
        dialog.title = title;
        dialog.message = message;
        dialog.ok = ok;
        
        dialog.init();
        return dialog;
    }
    public void init()
    {
        IOSNative.showMessage(title, message, ok);
    }
    #endregion
    #region IOS_EVENT_LISTENER
    public void OnPopUpCallBack(string buttonIndex)
    {
        RaiseOnMessagePopupComplete(MessageState.OK);
        Destroy(gameObject);
    }
    #endregion
}
```

#### CONFIRMATION POPUP
**A)** Create **NativeDialog.cs** for basic setup of dialog message popup:

```csharp
public class NativeDialog
{
    #region PUBLIC_VARIABLES
    string title;
    string message;
    string yesButton;
    string noButton;
    public string urlString;
    #endregion
    #region PUBLIC_FUNCTIONS
    public NativeDialog(string title, string message)
    {
        this.title = title;
        this.message = message;
        this.yesButton = "Yes";
        this.noButton = "No";
    }
    public NativeDialog(string title, string message, string yesButtonText, string noButtonText)
    {
        this.title = title;
        this.message = message;
        this.yesButton = yesButtonText;
        this.noButton = noButtonText;
    }
    public void SetUrlString(string urlString)
    {
        this.urlString = urlString;
    }
    public void init()
    {
        #if UNITY_IPHONE
        IOSDialog dialog = IOSDialog.Create(title, message, yesButton, noButton);
        dialog.urlString = urlString;
        #endif
    }
    #endregion
}
```

**B)** Create **IOSDialog.cs** for dialog message popup:

```csharp
public class IOSDialog : MonoBehaviour
{
    #region DELEGATE
    public delegate void OnDialogPopupComplete(MessageState state);
    public static event OnDialogPopupComplete onDialogPopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnOnDialogPopupComplete(MessageState state)
    {
        if (onDialogPopupComplete != null)
            onDialogPopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string yes;
    public string no;
    public string urlString;
    #endregion
    #region PUBLIC_FUNCTIONS
    // Constructor
    public static IOSDialog Create(string title, string message)
    {
        return Create(title, message, "Yes", "No");
    }
    public static IOSDialog Create(string title, string message, string yes, string no)
    {
        IOSDialog dialog;
        dialog = new GameObject("IOSDialogPopUp").AddComponent<IOSDialog>();
        dialog.title = title;
        dialog.message = message;
        dialog.yes = yes;
        dialog.no = no;
        dialog.init();
        return dialog;
    }
    public void init()
    {
        IOSNative.showDialog(title, message, yes, no);
    }
    #endregion
    #region IOS_EVENT_LISTENER
    public void OnDialogPopUpCallBack(string buttonIndex)
    {
        int index = System.Convert.ToInt16(buttonIndex);
        switch (index)
        {
            case 0: 
                IOSNative.RedirectToWebPage(urlString);
                RaiseOnOnDialogPopupComplete(MessageState.YES);
                break;
            case 1: 
                RaiseOnOnDialogPopupComplete(MessageState.NO);
                break;
        }
        Destroy(gameObject);
    }
    #endregion
}
```

#### RATE-US POPUP

**A)** Create **NativeRateUS.cs** for Basic setup of rate-us popup:

```csharp
public class NativeRateUS
{
   #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string yes;
    public string later;
    public string no;
    public string appleId;
    #endregion
    #region PUBLIC_FUNCTIONS
    // Constructor
    public NativeRateUS(string title, string message)
    {
        this.title = title;
        this.message = message;
        this.yes = "Rate app";
        this.later = "Later";
        this.no = "No, thanks";
    }
    // Constructor
    public NativeRateUS(string title, string message, string yes, string later, string no)
    {
        this.title = title;
        this.message = message;
        this.yes = yes;
        this.later = later;
        this.no = no;
    }
    // Set AppID to rate app
    public void SetAppleId(string _appleId)
    {
        appleId = _appleId;
    }
    
    // Initialize rate popup
    public void InitRateUS()
    {
        #if UNITY_IPHONE
        IOSRateUsPopUp rate = IOSRateUsPopUp.Create(title, message, yes, later, no);
        rate.appleId = appleId;
        #endif
    }
    #endregion
}
```

**B)** Create **IOSRateUsPopUp.cs** for rate-us popup:

```csharp
public class IOSRateUsPopUp : MonoBehaviour
{
    #region DELEGATE
    public delegate void OnRateUSPopupComplete(MessageState state);
    public static event OnRateUSPopupComplete onRateUSPopupComplete;
    #endregion
    #region DELEGATE_CALLS
    private void RaiseOnOnRateUSPopupComplete(MessageState state)
    {
        if (onRateUSPopupComplete != null)
            onRateUSPopupComplete(state);
    }
    #endregion
    #region PUBLIC_VARIABLES
    public string title;
    public string message;
    public string rate;
    public string remind;
    public string declined;
    public string appleId;
    #endregion
    #region PUBLIC_FUNCTIONS
    public static IOSRateUsPopUp Create()
    {
        return Create("Like the Game?", "Rate US");
    }
    public static IOSRateUsPopUp Create(string title, string message)
    {
        return Create(title, message, "Rate Now", "Ask me later", "No, thanks");
    }
    public static IOSRateUsPopUp Create(string title, string message, string rate, string remind, string declined)
    {
        IOSRateUsPopUp popup = new GameObject("IOSRateUsPopUp").AddComponent<IOSRateUsPopUp>();
        popup.title = title;
        popup.message = message;
        popup.rate = rate;
        popup.remind = remind;
        popup.declined = declined;
        popup.init();
        return popup;
    }
    public void init()
    {
        IOSNative.showRateUsPopUP(title, message, rate, remind, declined);
    }
    #endregion
    #region IOS_EVENT_LISTENER
    public void OnRatePopUpCallBack(string buttonIndex)
    {
        int index = System.Convert.ToInt16(buttonIndex);
        switch (index)
        {
            case 0: 
                IOSNative.RedirectToAppStoreRatingPage(appleId);
                RaiseOnOnRateUSPopupComplete(MessageState.RATED);
                break;
            case 1:
                RaiseOnOnRateUSPopupComplete(MessageState.REMIND);
                break;
            case 2:
                RaiseOnOnRateUSPopupComplete(MessageState.DECLINED);
                break;
        }
        Destroy(gameObject);
    }
    #endregion
}
```

### Note :
In this classes (B section of each popup) we have created gameobject and we are using this gameobject-name to get event callback. We are using this names in next section (in Objective-C file **UnitySendMessage()** )

## Step 6 : Setup iOS file

"Awesome, you are done with the basic code! Now, let’s code to create popups using Objective-C."

To do that,Create new xcode project to create Objective-C files. If you don’t know about xcode and how to create new xcode project then please Learn Basic of Creating Xcode Project in iOS.

"Don’t worry about the code for now. Just copy it and paste it in your file. If you face any issues in creating project or file then you can download a source code from the bottom of this blog post. And once you are done with downloading the project, you can copy all iOS file into your Unity project’s Plugins folder."

Coming back to Xcode, create new Objective-C file named DataConvertor to convert data.

#### DataConverter.h

```swift
#import <Foundation/Foundation.h>
@interface DataConvertor : NSObject
+ (NSString*) charToNSString: (char*)value;
+ (const char *) NSIntToChar: (NSInteger) value;
+ (const char *) NSStringToChar: (NSString *) value;
@end
```
#### DataConverter.m

```swift
#import "DataConvertor.h"
@implementation DataConvertor
+(NSString *) charToNSString:(char *)value {
    if (value != NULL) {
        return [NSString stringWithUTF8String: value];
    } else {
        return [NSString stringWithUTF8String: ""];
    }
}
+(const char *)NSIntToChar:(NSInteger)value {
    NSString *tmp = [NSString stringWithFormat:@"%ld", (long)value];
    return [tmp UTF8String];
}
+ (const char *)NSStringToChar:(NSString *)value {
    return [value UTF8String];
}
@end
```

Create another file named **IOSNativePopUpsManager** to get call from Unity script, and show popups.

#### IOSNativePopUpsManager.h
```swift
#import <Foundation/Foundation.h>
#import "DataConvertor.h"
@interface IOSNativePopUpsManager : NSObject
+ (void) unregisterAllertView;
@end
```

#### IOSNativePopUpsManager.m

```swift
#import "IOSNativePopUpsManager.h"
@implementation IOSNativePopUpsManager
static UIAlertController* _currentAllert = nil;
+ (void) unregisterAllertView {
    if(_currentAllert != nil) {
        _currentAllert = nil;
    }
}
+(void) dismissCurrentAlert {
    if(_currentAllert != nil) {
        [_currentAllert dismissViewControllerAnimated:NO completion:nil];
        _currentAllert = nil;
    }
}
+(void) showRateUsPopUp: (NSString *) title message: (NSString*) msg b1: (NSString*) b1 b2: (NSString*) b2 b3: (NSString*) b3 {
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:title message:msg preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *rateAction = [UIAlertAction actionWithTitle:b1 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSRateUsPopUp", "OnRatePopUpCallBack", [DataConvertor NSIntToChar:0]);
    }];
    UIAlertAction *laterAction = [UIAlertAction actionWithTitle:b2 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSRateUsPopUp", "OnRatePopUpCallBack", [DataConvertor NSIntToChar:1]);
    }];
    UIAlertAction *declineAction = [UIAlertAction actionWithTitle:b3 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSRateUsPopUp", "OnRatePopUpCallBack", [DataConvertor NSIntToChar:2]);
    }];
    [alertController addAction:rateAction];
    [alertController addAction:laterAction];
    [alertController addAction:declineAction];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alertController animated:YES completion:nil];
    _currentAllert = alertController;
}
+ (void) showDialog: (NSString *) title message: (NSString*) msg yesTitle:(NSString*) b1 noTitle: (NSString*) b2{
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:title message:msg preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *yesAction = [UIAlertAction actionWithTitle:b1 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSDialogPopUp", "OnDialogPopUpCallBack", [DataConvertor NSIntToChar:0]);
    }];
    UIAlertAction *noAction = [UIAlertAction actionWithTitle:b2 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSDialogPopUp", "OnDialogPopUpCallBack", [DataConvertor NSIntToChar:1]);
    }];
    [alertController addAction:yesAction];
    [alertController addAction:noAction];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alertController animated:YES completion:nil];
    _currentAllert = alertController;
}
+(void)showMessage: (NSString *) title message: (NSString*) msg okTitle:(NSString*) b1 {
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:title message:msg preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction *okAction = [UIAlertAction actionWithTitle:b1 style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
    [IOSNativePopUpsManager unregisterAllertView];
    UnitySendMessage("IOSMessagePopUp", "OnPopUpCallBack", [DataConvertor NSIntToChar:0]);
    }];
    [alertController addAction:okAction];
    [[[[UIApplication sharedApplication] keyWindow] rootViewController] presentViewController:alertController animated:YES completion:nil];
    _currentAllert = alertController;
}
extern "C" {
    // Unity Call
    void _TAG_ShowRateUsPopUp(char* title, char* message, char* b1, char* b2, char* b3) {
        [IOSNativePopUpsManager showRateUsPopUp:[DataConvertor charToNSString:title] message:[DataConvertor charToNSString:message] b1:[DataConvertor charToNSString:b1] b2:[DataConvertor charToNSString:b2] b3:[DataConvertor charToNSString:b3]];
    }
    void _TAG_ShowDialog(char* title, char* message, char* yes, char* no) {
        [IOSNativePopUpsManager showDialog:[DataConvertor charToNSString:title] message:[DataConvertor charToNSString:message] yesTitle:[DataConvertor charToNSString:yes] noTitle:[DataConvertor charToNSString:no]];
    }
    void _TAG_ShowMessage(char* title, char* message, char* ok) {
        [IOSNativePopUpsManager showMessage:[DataConvertor charToNSString:title] message:[DataConvertor charToNSString:message] okTitle:[DataConvertor charToNSString:ok]];
    }
    void _TAG_DismissCurrentAlert() {
        [IOSNativePopUpsManager dismissCurrentAlert];
    }
}
@end
```

### Note :
In this class, we are using **UnitySendMessage()** to send a message to Unity and we are using gameobject-name as a parameter. This must match with gameobject which is created in particular popup class.

Now create a new file named IOSNativeUtility to redirect control from application to rate page or any other web page.

#### IOSNativeUtility.h

```swift
#import <Foundation/Foundation.h>
#import "DataConvertor.h"
#if UNITY_VERSION < 450
#include "iPhone_View.h"
#endif
@interface IOSNativeUtility : NSObject
@property (strong) UIActivityIndicatorView *spinner;
+ (id) sharedInstance;
- (void) redirectToRatigPage: (NSString *) appId;
@end
```

#### IOSNativeUtility.m

```swift
#import "IOSNativeUtility.h"
@implementation IOSNativeUtility
static IOSNativeUtility *_sharedInstance;
static NSString* templateReviewURLIOS7 = @"itms-apps://itunes.apple.com/app/idAPP_ID";
NSString *templateReviewURL = @"itms-apps://ax.itunes.apple.com/WebObjects/MZStore.woa/wa/viewContentsUserReviews?type=Purple+Software&id=APP_ID";
+ (id)sharedInstance {
    if (_sharedInstance == nil) {
        _sharedInstance = [[self alloc] init];
    }
    return _sharedInstance;
}
-(void) redirectToRatigPage:(NSString *)appId {
    #if TARGET_IPHONE_SIMULATOR
        NSLog(@"APPIRATER NOTE: iTunes App Store is not supported on the iOS simulator. Unable to open App Store page.");
    #else
    NSString *reviewURL;
    NSArray *vComp = [[UIDevice currentDevice].systemVersion componentsSeparatedByString:@"."];
    if ([[vComp objectAtIndex:0] intValue] >= 7) {
        reviewURL = [templateReviewURLIOS7 stringByReplacingOccurrencesOfString:@"APP_ID" withString:[NSString stringWithFormat:@"%@", appId]];
    } else {
        reviewURL = [templateReviewURL stringByReplacingOccurrencesOfString:@"APP_ID" withString:[NSString stringWithFormat:@"%@", appId]];
    }
    NSLog(@"redirecting to iTunes page, IOS version: %i", [[vComp objectAtIndex:0] intValue]);
    NSLog(@"redirect URL: %@", reviewURL);
    [[UIApplication sharedApplication] openURL:[NSURL URLWithString:reviewURL]];
    #endif
}
-(void) openWebPage:(NSString *)urlString{
    #if TARGET_IPHONE_SIMULATOR
        NSLog(@"APPIRATER NOTE: iTunes App Store is not supported on the iOS simulator. Unable to open App Store page.");
    #else
        NSURL *url = [ [ NSURL alloc ] initWithString:urlString];
        [[UIApplication sharedApplication] openURL:url];
    #endif
}
extern "C" {
    void _TAG_RedirectToAppStoreRatingPage(char* appId) {
        [[IOSNativeUtility sharedInstance] redirectToRatigPage: [DataConvertor charToNSString:appId ]];
    }
    void _TAG_RedirectToWebPage(char* urlString){
        [[IOSNativeUtility sharedInstance] openWebPage:[DataConvertor charToNSString:urlString]];
    }
}
@end
```

Now, copy all Objective-C files from the XCode project directory to Unity project’s Plugins directory.

If you face any issues in creating Xcode project or Objective-C file then you can download the source code and just copy-paste all Objective-C file from iOS folder to Plugins/iOS folder.
