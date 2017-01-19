---
layout: post
title:  "iOS Interview Basic Questions"
date:   2019-01-19 15:14:54
categories: iOS
tags: iOS Interview
---

1. App lifecycle
..1 Inactive: The app is running in the foreground but is currently not receiving events.
..2 Active: The app is running in the foreground and is receiving events.
..3 Background: The app is in the background and executing code. Most apps enter this state briefly on their way to being suspended.
..4 Suspended: The app is in the background but is not executing code.  While suspended, an app remains in memory but does not execute any code. When a low-memory condition occurs, the system may purge suspended apps without notice to make more space for the foreground app.

2. when to use weakself
... avoid retain cycle

3. UI update in main queue. What should we do if we need to handle some event and update UI and it takes a long time?
...
```
dispatch_async(        
  dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0),
    ^{
      sleep(10);
      dispatch_async(dispatch_get_main_queue(), ^{
         self.alert.text = @"Waiting over";
      });
});
```

4. What considerations do you need when writing a UITableViewController which shows images downloaded from a remote server?
..1)Only download the image when the cell is scrolled into view, i.e. when cellForRowAtIndexPath is called.
..2)Downloading the image asynchronously on a background thread so as not to block the UI so the user can keep scrolling.
..3)When the image has downloaded for a cell we need to check if that cell is still in the view or whether it has been re-used by another piece of data. If it’s been re-used then we should discard the image, otherwise we need to switch back to the main thread to change the image on the cell.

5. What's your preference when writing UI's? Xib files, Storyboards or programmatic UIView?
...Programmatically constructing UIView's can be verbose and tedious, but it can allow for greater control and also easier separation and sharing of code. They can also be more easily unit tested.

6. How to debug memory leak?
...Instruments
