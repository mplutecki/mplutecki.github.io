---
layout: post
title: "A checklist for implementing background downloads"
date: 2022-05-18 10:13:17 +0200
categories: jekyll update
---

# Overview

- Downloading files in background is not that common
- Requires knowledge related to networking and app's lifecycle
- Extensive tutorial is here: [Apple doc][apple-doc]
- This post is a summary/cookbook with a sample project
- If your app declares background modes you don't need to create a special background session. For details see the [Apple doc][apple-doc]

## The steps

1. Create an `URLSession` with background configuration using [background(withIdentifier identifier: String)](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1407496-background)
1. Implement [application(\_:handleEventsForBackgroundURLSession:completionHandler:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1622941-application) - retain the `completionHandler` to be used later.
1. Implement [urlSessionDidFinishEvents(forBackgroundURLSession:)](https://developer.apple.com/documentation/foundation/urlsessiondelegate/1617185-urlsessiondidfinishevents) - move the downloaded file to the permanent location (usually inside `Documents` directory)
1. Implement [urlSession(\_:downloadTask:didFinishDownloadingTo:)](https://developer.apple.com/documentation/foundation/urlsessiondownloaddelegate/1411575-urlsession) - update the UI if needed and call the `completionHandler` received by the `AppDelegate`

The following part includes more detailed explanation of each step.

# Step 1: Create a background session

- A special session is needed to continue downloads even after the app terminates
- You need to assign an identifier that enables you to restore the session on the next app launch
- The identifier must be unique within your app

<!-- prettier-ignore-start -->
{% highlight swift %}
let sessionIdentifier = "background-session"
let config = URLSessionConfiguration.background(
  withIdentifier: sessionIdentifier)
let session = URLSession(
  configuration: config, delegate: self, delegateQueue: nil)
{% endhighlight %}
<!-- prettier-ignore-end -->

- Notice that if `nil` is passed as a `delegateQueue` the system creates it's own serial queue to deliver the delegate events.
- An interesting nuance is that URL session retains it's delegate until it's ivalidated.

# Step 2: Implement app delegate

- this is called when the app is relaunched by the system to handle the finished download task
- the `completionHandler` will be needed to let the system know that we've finished handling the download. We'll call it later.

<!-- Maybe we need to explain what's exacly an event? -->

<!-- prettier-ignore-start -->
{% highlight swift %}
func application(
  _ application: UIApplication,
  handleEventsForBackgroundURLSession identifier: String,
  completionHandler: @escaping () -> Void) 
{
  guard identifier == sessionIdentifier else { return }
  backgroundCompletionHandler = completionHandler
}
{% endhighlight %}
<!-- prettier-ignore-end -->

# Step 3: Move the downloaded file

- Here the download has finished successfully and the system gives us the temporary file
- We need to move the file to a permanent location, here we use the default `FileManager` to move it to the `Documents` folder

<!-- prettier-ignore-start -->
{% highlight swift %}
func urlSession(
  _ session: URLSession,
  downloadTask: URLSessionDownloadTask,
  didFinishDownloadingTo location: URL)
{
  let targetLocation = FileManager
    .default.urls(for: .documentDirectory, in: .userDomainMask)
    .first?
    .appendingPathComponent("filename")
  try? FileManager.default.moveItem(at: location, to: targetFileLocation)
}
{% endhighlight %}
<!-- prettier-ignore-end -->

# Step 4: Update the UI and let the system know

- you can update the UI

<!-- prettier-ignore-start -->
{% highlight swift %}
  func urlSessionDidFinishEvents(forBackgroundURLSession session: URLSession) {
    // The background handler must be called on the main queue.
    DispatchQueue.main.async {
      backgroundCompletionHandler?()
    }
  }
{% endhighlight %}
<!-- prettier-ignore-end -->

# Final notes

- Background downloads are handled by a separate process therefore no custom `URLProtocol` classes are allowed
- Provide estimated download size to help the system allocate resources for the operation

# Related topics

- suspending, resuming download
- the app's lifecycle
- background modes
- downloading files from websites [Doc](https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_from_websites)

# References

<!-- - Custom `URLProtocol` is not allowed for background sessions
- -->

[apple-doc]: https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background
