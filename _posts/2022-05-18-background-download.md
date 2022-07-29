---
layout: post
title: "Background downloads"
date: 2022-05-18 10:13:17 +0200
categories: jekyll update
---

# Overview

Large, time-consuming downloads are usually expected to be continued even if a user leaves an app.
This is a quick summary of how to add basic support for background downloads.
Checkout [the sample project][sample-project] to test how it works in practice.
If you need a throughout explanation be sure to read [this article by Apple][apple-doc].

# The steps

1. Create an `URLSession` with background configuration using [background(withIdentifier identifier: String)][background-with-identifier]
1. Implement [application(\_:handleEventsForBackgroundURLSession:completionHandler:)][application-handle-events] - retain the `completionHandler` to be used later.
1. Implement [urlSession(\_:downloadTask:didFinishDownloadingTo:)][url-session-did-finish-downloading] - move the downloaded file to the permanent location
1. Implement [urlSessionDidFinishEvents(forBackgroundURLSession:)][url-session-did-finish-events] - update the UI if needed and call the `completionHandler` received by the `AppDelegate`

# Step 1: Create a background session

For download tasks that are independed from an app's lifecycle you need to use a special `URLSession`.
The listing below show how such session can be created.
There is an exception here: if your app declares background modes you might not need such session (see [this][apple-doc]).

The background session needs to have an identifier, that is unique within your app.
This identifier is needed to restore the session across app launches.

<!-- prettier-ignore-start -->
{% highlight swift %}
let config = URLSessionConfiguration.background(
  withIdentifier: "background-session")
let session = URLSession(
  configuration: config, delegate: self, delegateQueue: nil)
{% endhighlight %}
<!-- prettier-ignore-end -->

Note that passing `nil` as a `delegateQueue` tells the system to create the queue itself.
If you provide your own queue make sure that it's serial to ensure the correct order of the delivered events.

# Step 2: Implement the app delegate callback

The [application(\_:handleEventsForBackgroundURLSession:completionHandler:)][application-handle-events] method is called if the app is relaunched to handle the completed download.
The `completionHandler` will be needed later to tell the system that it should update the app's screenshot (for the App Switcher).

<!-- prettier-ignore-start -->
{% highlight swift %}
func application(
  _ application: UIApplication,
  handleEventsForBackgroundURLSession identifier: String,
  completionHandler: @escaping () -> Void)
{
  backgroundCompletionHandler = completionHandler
}
{% endhighlight %}
<!-- prettier-ignore-end -->

# Step 3: Save the downloaded file

The downloaded file is stored in a temporary location.
When the download finishes the system calls [urlSessionDidFinishEvents(forBackgroundURLSession:)][url-session-did-finish-events] and makes the temporary file available until the method returns.
In this example we'll use `FileManager` to move it to the static location in the `Documents` folder.

<!-- prettier-ignore-start -->
{% highlight swift %}
func urlSession(
  _ session: URLSession,
  downloadTask: URLSessionDownloadTask,
  didFinishDownloadingTo temporaryFile: URL)
{
  try! FileManager.default.moveItem(at: temporaryFile, to: localFile)
}
{% endhighlight %}
<!-- prettier-ignore-end -->

# Step 4: Update the UI

The last step us to update the app's UI, so that the download status is visible when a user scrolls through the App Switcher.
The delegate method can be called from an arbitrary queue, so be sure to dispatch to the main one before calling any of the `UIKit` classes.
`backgroundCompletionHandler` must also be caled from the main queue.

<!-- prettier-ignore-start -->
{% highlight swift %}
  func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
    DispatchQueue.main.async {
      backgroundCompletionHandler?()
    }
  }
{% endhighlight %}
<!-- prettier-ignore-end -->

# Final notes

- Custom `URLProtocol` implementations are not supported for background downloads, because the operation takes place in another process.
- You can provide an estimated download size to help the system allocate resources and manage battery life.

# Related topics

- You can further improve the experience by supporting resumable downloads. Note that this needs to be supported by the server.
- [Article - Downloading Files from Websites][download-from-websites]
- [Article - Managing Your App's Life Cycle][app-lifecycle]
- [Article - Configuring Background Execution Modes][background-modes]

[apple-doc]: https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background
[sample-project]: https://github.com
[download-from-websites]: https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_from_websites
[app-lifecycle]: https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle
[background-modes]: https://developer.apple.com/documentation/xcode/configuring-background-execution-modes
[background-with-identifier]: https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1407496-background
[application-handle-events]: https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622941-application
[url-session-did-finish-events]: https://developer.apple.com/documentation/foundation/urlsessiondelegate/1617185-urlsessiondidfinishevents
[url-session-did-finish-downloading]: https://developer.apple.com/documentation/foundation/urlsessiondelegate/1617185-urlsessiondidfinishevents
