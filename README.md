Purpose
--------------

The App Store app updates mechanism is somewhat cumbersome and disconnected from the apps themselves. Users often fail to notice when new versions of an app are released, and if they do notice, the App Store's "download all" option means that users often won't see the release notes for the new version.

Whilst it is not possible to bypass the App Store and update an app from within the app itself, as this violates the App Store terms and conditions, there is no reason why an app should not inform the user that the new release is ready, and direct them to the App Store to download the update.

iVersion is a simple, drop-in class to allow iPhone and Mac App Store apps to automatically check for updates and inform the user about new features.

iVersion has an additional function, which is to tell users about important new features when they first run an app after downloading a new version.

NOTE: iVersion cannot tell if a given release is available to download, so make sure that you only update the remote versions file after apple has approved your app and it has appeared in the store.

One way to do this automatically is to replace the hard-coded remote versions file with a web service that dynamically polls the iTunes App Store services for your app details and extracts the version number and release notes. A sample implementation of such a service (written in PHP) is included as an example.


Supported iOS & SDK Versions
-----------------------------

* Supported build target - iOS 5.0 (Xcode 4.2)
* Earliest supported deployment target - iOS 4.0 (Xcode 4.2)
* Earliest compatible deployment target - iOS 3.0

NOTE: 'Supported' means that the library has been tested with this version. 'Compatible' means that the library should work on this iOS version (i.e. it doesn't rely on any unavailable SDK features) but is no longer being tested for compatibility and may require tweaking or bug fixes to run correctly.


Installation
--------------

To install iVersion into your app, drag the iVersion.h and .m files into your project.

To enable iVersion in your application you need to instantiate and configure iVersion *before* the app has finished launching. The easiest way to do this is to add the iVersion configuration code in your AppDelegate's `initialize` method, like this:

	+ (void)initialize
	{
		//configure iVersion
		[iVersion sharedInstance].appStoreID = 355313284;
		[iVersion sharedInstance].remoteVersionsPlistURL = @"http://example.com/versions.plist";
		[iVersion sharedInstance].localVersionsPlistPath = @"versions.plist";
	}

The above code represents the minimum configuration needed to make iVersion work, although there are other configuration options you may wish to add (documented below).

The exact same configuration code will work for both Mac and iPhone/iPad.

You will also need to add a plist file to your app containing the release notes for the current version, and host another copy on a web-facing server somewhere. The format for these plists is as follows:

	<?xml version="1.0" encoding="UTF-8"?>
	<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
	<plist version="1.0">
	<dict>
		<key>1.0</key>
		<array>
			<string>First release</string>
		</array>
		<key>1.1</key>
		<array>
			<string>NEW: Added  new snoodlebar feature</string>
			<string>FIX: Fixed the bugalloo glitch</string>
		</array>
		...
	</dict>
	</plist>

The root node of the plist is a dictionary containing one or more items. Each item represents a particular version of your application.

The key for each value must be a numerical version number consisting of one or more positive integers separated by decimal points. These should match the values you set for the Bundle Version (CFBundleVersion) key in your application's info.plist.

Each value should be an array of strings, each representing a single bullet point in your release notes. There is no restriction to the format of each release note - the approach in the example above is just a suggestion. You may prefer to put all your notes into a single string and add your own line formatting or indenting. You can also omit the release notes if you want and just have an empty <array/>.


Plist Tips
--------------

It is not recommended that you include every single version of your app in the release notes. In practice, if a user updates so infrequently that they miss three or more consecutive versions, you still probably only want to show them the last couple of releases worth of notes, so delete older releases from the file when you update.

Do not feel that the release notes have to exactly mirror those in iTunes or the Mac App Store. There is limited space in a modal alert and users won't want to read a lot of text whenever they launch your app, so it's probably best just to list the key new features in your versions plist.

The local and remote release notes plists do not have to match. Whilst it may be convenient to make them identical from a maintenance point of view, the local version works better if it is written in a slightly different tone. For example the remote release notes might read:

	Version 1.1
	
	- Fixed crashing bug
	- Added shiny new menu graphics
	- New sound settings

Whereas the local one might say:

	Check out the new sound options in the settings panel. You can access the settings from the home screen

There's no point in mentioning the bug fix or new graphics because the user can see this easily enough just by using the app. On the other hand, they might not notice the new sound options, or know how to find them.

Also don't always feel you have to include the local release notes file. If there are no features that need drawing attention to in the new release, just omit the file - it won't prevent you adding the remote versions file to prompt users to upgrade, and it won't prevent local release notes in future releases from working correctly.


Configuration
--------------

To configure iVersion, there are a number of properties of the iVersion class that can alter the behaviour and appearance of iVersion. These should be mostly self- explanatory, but they are documented below:

	@property (nonatomic, assign) NSUInteger appStoreID;

This should match the iTunes app ID of your application, which you can get from iTunes connect after setting up your app. This is only used for remote version updates, so you can ignore this if you do not intend to use that feature (although it should still be set to a valid integer value).

	@property (nonatomic, copy) NSString *remoteVersionsPlistURL;

This is the URL of the remotely hosted plist that iVersion will check for new releases. As noted above, make sure you only update this file after your new release has been approved by Apple and appeared in the store or you will have some very confused customers. For testing purposes, you may wish to create a separate copy of the file at a different address and use a build constant to switch which version the app points at. Set this value to nil if you do not want your app to check for updates automatically. Do not set it to an invalid URL such as example.com because this will waste battery, CPU and bandwidth as the app tries to check the invalid URL each time it launches.

	@property (nonatomic, copy) NSString *localVersionsPlistPath;

The file name of your local release notes plist used to tell users about new features when they first launch a new update. Set this value to nil if you do not want your app to display release notes for the current version.

	@property (nonatomic, copy) NSString *applicationName;

This is the name of the app displayed in the alert. It is set automatically from the info.plist, but you may wish to override it with a shorter or longer version.

	@property (nonatomic, copy) NSString *applicationVersion;

The current version number of the app. This is set automatically from the info.plist and it's probably not a good ideas to change it unless you know what you are doing. In some cases your bundle version may not match the publicly known "display" version of your app, in which case use the display version here. Note that the version numbers in the plist will be compared to this value, not the one in the info.plist.

	@property (nonatomic, assign) BOOL showOnFirstLaunch;

Specify whether the release notes for the current version should be shown the first time the user launches the app. If set to no it means that users who, for example, download version 1.1 of your app but never installed a previous version, won't be shown the new features in version 1.1.

	@property (nonatomic, assign) BOOL groupNotesByVersion;

If your release notes files contains multiple versions, this option will group the release notes by their version number in the alert shown to the user. If set to NO, the release notes will be shown as a single list.

	@property (nonatomic, assign) float checkPeriod;

Sets how frequently the app will check for new releases. This is measured in days but can be set to a fractional value, e.g. 0.5. Set this to a higher value to avoid excessive traffic to your server. A value of zero means the app will check every time it's launched.

	@property (nonatomic, assign) float remindPeriod;

How long the app should wait before reminding a user of a new version after they select the "remind me later" option. A value of zero means the app will remind the user every launch. Note that this value supersedes the check period, so once a reminder is set, the app won't check for new releases during the reminder period, even if new version are released in the meantime.

	@property (nonatomic, copy) NSString *inThisVersionTitle;

The title displayed for features in the current version (i.e. features in the local version s plist file).

	@property (nonatomic, copy) NSString *updateAvailableTitle;

The title displayed when iVersion detects a new version of the app has appeared in the remote versions plist.

	@property (nonatomic, copy) NSString *versionLabelFormat;

The format string for the release notes version separators. This should include a %@ placeholder for the version number, e.g "Version %@".

	@property (nonatomic, copy) NSString *okButtonLabel;
	
The button label for the button to dismiss the "new in this version" modal.

	@property (nonatomic, copy) NSString *ignoreButtonLabel;

The button label for the button the user presses if they do not want to download a new update.

	@property (nonatomic, copy) NSString *remindButtonLabel;

The button label for the button the user presses if they don't want to download a new update immediately, but do want to be reminded about it in future. Set this to nil if you don't want to display the remind me button - e.g. if you don't have space on screen.

	@property (nonatomic, copy) NSString *downloadButtonLabel;

The button label for the button the user presses if they want to download a new update.

	@property (nonatomic, assign) BOOL localChecksDisabled;

Set this to true to disable checking for local release notes. This is equivalent to setting the localVersionsPlistPath to nil, but may be more convenient.

	@property (nonatomic, assign) BOOL remoteChecksDisabled;

Set this to true to disable checking for new releases. This is equivalent to setting the remoteVersionsPlistURL to nil, but may be more convenient. You might connect this to an in-app user setting for toggling checks for updates.

	@property (nonatomic, assign) BOOL localDebug;

If set to YES, iVersion will always display the contents of the local versions plist, irrespective of the version number of the current build. Use this to proofread your release notes during testing, but disable it for the final release.

	@property (nonatomic, assign) BOOL remoteDebug;

If set to YES, iVersion will always display the contents of the remote versions plist, irrespective of the version number of the current build or the check/remind period settings. Use this to proofread your release notes during testing, but disable it for the final release.


Advanced properties
---------------

If the default iVersion behaviour doesn't meet your requirements, you can implement your own by using the advanced properties, methods and delegate. The properties below let you access internal state and override it:

	@property (nonatomic, retain) NSURL *updateURL;

The URL that the app will direct the user to if an update is detected and they choose to download it. If you are implementing your own download button, you should probably use the openAppPageInAppStore method instead, especially on Mac OS, as the process for opening the Mac app store is more complex than merely opening the URL.

	@property (nonatomic, copy) NSString *ignoredVersion;

The version string of the last app version that the user ignored. If the user hasn't ignored any releases, this will be nil. Set this to nil to clear the ignored version.

	@property (nonatomic, retain) NSDate *lastChecked;

The last date on which iVersion checked for an update. You can use this in combination with the checkPeriod to determine if the app should check again.

	@property (nonatomic, retain) NSDate *lastReminded;

The last date on which the user was reminded of a new version. You can use this in combination with the remindPeriod to determine if the app should check again. Set this to nil to clear the reminder delay.

	@property (nonatomic, assign) BOOL viewedVersionDetails;

Flag that indicates if the local version details have been viewed (YES) or not (NO).

	@property (nonatomic, assign) id<iVersionDelegate> delegate;

An object you have supplied that implements the iVersionDelegate protocol, documented below. Use this to detect and/or override iVersion's default behaviour. 


Advanced methods
---------------

	- (void)openAppPageInAppStore;

This method will open the application page in the Mac or iPhone app store, depending on which platform is running. You should use this method instead of the updateURL property if you are running on Mac OS as the process for launching the Mac app store is more complex than merely opening the URL.

	- (void)checkForNewVersion;

This method will trigger a new check for new versions, ignoring the checkPeriod and remindPeriod properties.

	- (NSString *)versionDetails;

This returns the local release notes for current version, or any versions that have been released since the last time the app was launched, depending on how many versions are included in the local version plist file. If this isn't the first time this version of the app was launched, only the most recent version is  included.


Delegate methods
---------------

The iVersionDelegate protocol provides the following methods that can be used intercept iVersion events and override the default behaviour. All methods are optional.

	- (BOOL)iVersionShouldCheckForNewVersion;

This is called if the checking criteria have all been met and iVersion is about to check for a new version. If you return NO, the check will not be performed. This method is not called if you trigger the check manually with the checkForNewVersion method.

	- (void)iVersionDidNotDetectNewVersion;

This is called if the version check did not detect any new versions of the application.

	- (void)iVersionVersionCheckFailed:(NSError *)error;

This is called if the version check failed due to network issues or because the remote versions plist file was missing or corrupt.

	- (void)iVersionDetectedNewVersion:(NSString *)version details:(NSString *)versionDetails;

This is called if a new version was detected.

	- (BOOL)iVersionShouldDisplayNewVersion:(NSString *)version details:(NSString *)versionDetails;

This is called immediately before the new version detected alert is displayed. Return NO to prevent the alert from being displayed. Note that if you are implementing the alert yourself you will need to set the lastChecked, lastReminded and ignoredVersion properties yourself, depending on the user response.

	- (BOOL)iVersionShouldDisplayCurrentVersionDetails:(NSString *)versionDetails;

This is called immediately before the current version new features alert is displayed. Return NO to prevent the alert from being displayed. Note that if you intend to implement this notification yourself, you will need to set the viewedVersionDetails flag manually.


Localisation
---------------

Although iVersion isn't localised, it is easy to localise without making any modifications to the library itself. All you need to do is provide localised values for all of the message strings by setting the properties above using NSLocalizedString(...). If you need to provide localised release notes, the simplest way to do this is to localise the remoteVersionsPlistURL property in the same way, providing a different URL for each language.


Web Service
---------------

Included with the source is a sample web service for automatically detecting app updates. The service is written in PHP, but is simple enough that it can be easily ported to other web scripting languages. The new version of this service makes use of Apple's authorised search APIs.

The service uses the following configuration constants:

	$platform
	
Apple's search APIs are platform specific (iPhone, iPad and Mac). Use one of the IPHONE, IPAD and MAC constants provided for this value.

	$app_store_id
	
The app store ID of your application.

	$developer
	
Your iTunes developer name as it appears on iTunes. This is used as a filter to reduce the amount of JSON the service has to sift through to find your app (unfortunately there currently appears to be no way to search for a specific app by ID).

	$country
	
The two-letter country code for the iTunes store you wish to get the application details from. There is no way to determine which app store a given user is using programmatically, so you'll either have to use the locale and hope that it's right, or hard-code for the US store and assume that your app will appear on all  stores at roughly the same time.

	$language
	
The language you want to return your release notes in. This is independent of the store country, so it's a good idea to base this on the user's locale by detecting this in the app and passing it through as a parameter in the service URL.

In principle these constants could all set by URL query parameter, but it may be unwise for you to host a general-purpose iVersion service URL in case other developers hot-link to it and use it for their own purposes (sucking up your bandwidth in the process). In general you should probably pass the language and country codes in dynamically and hard code the app details into the service.

The sample service also includes an optional caching feature. Since every iVersion request from every one of your apps will go via your web service, it may result in a lot of iTunes hits from your server. For a popular app this could be hundreds or thousands of hits per day. Apple might treat this suspicious and start rejecting hits from your server's IP address. By using the cache feature you can rate-limit this as well as improving performance and reducing your server's downstream bandwidth usage.

The service should fail gracefully if it has trouble connecting to iTunes. If anything goes wrong with retrieving or parsing the iTunes JSON it should return a valid plist file that simply contains no versions in the dictionary. If something more serious goes wrong, the iVersion library will silently ignore malformed version plists anyway, so it's pretty unlikely that using this service will break your app.

You may also find that the use of file_get_contents() for accessing a remote URL is not supported on your server. If that is the case, check out the PHP manual page for file_get_contents for suggestions on alternative implementations:

http://php.net/manual/en/function.file-get-contents.php 


Example Projects
---------------

When you build and run the basic Mac or iPhone example project for the first time, it will show an alert saying that a new version is available. This is because it has downloaded the remote versions.plist file and determined that the latest version is newer than the currently running app.

Quit the app, go into the iVersion-Info.plist file and edit the bundle version to 1.2. Now rebuild the app.

This time it will not say that a new version is available. In effect you have simulated an upgrade. Instead it will tell you about the new features in your currently installed version. This is because it has found that the bundle version of the current app is newer than the last recorded version that was launched, and has checked the local versions.plist file for a release notes entry for the new version.

If you dismiss the dialog and then quit and relaunch the app you should now see nothing. This is because the app has detected that the bundle version hasn't changed since you last launched the app.

To show the alerts again, delete the app from the simulator and reset the bundle version to 1.1. Alternatively, enable the debug settings to force the alerts to appear on launch.


Advanced Example
---------------

The advanced example demonstrates how you might implement a completely bespoke iVersion interface. Automatic version checking is disabled and instead the user can trigger a check by pressing the "Check for new version" button.

When pressed, the app display a progress wheel and then prints the result in a console underneath the button.