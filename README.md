# Pausing Frames

There are times for frames to be busy, and times when the work they're doing is wasteful. When they're wasteful, frames should be paused so that they don't bother the user. When a frame is paused, it doesn't run script and it doesn't load resources. It waits patiently to be resumed. 

Pausing may be initiated by conscientious script or by the browser in order to intervene on the user's behalf. If initiated by script, it can be resumed by script and optionally by the user (via a play button). If initiated by the browser, it can be resumed by the browser and optionally by the user.

The JavaScript API to pause a frame looks like:

```javascript
var frameElement = document.getElementById("hungryFrameID");
frameElement.pause({showUI: true});
```

And when you're ready to resume the frame:
```javascript
frameElement.resume();  // does nothing if frame isn't paused
```

To detect if a frame is paused:
```javascript
if (frameElement.paused) 
  alert("The frame is paused");
```

## Use Cases
* Pausing frames that violate policies (e.g., [TransferSizePolicy](https://github.com/WICG/transfer-size)). This provides a gentle, yet firm response to misbehaving frames.
* Pausing resource-intensive frames that the user isn't currently paying attention to. Why run a game's tight graphics loop while the user is busy reading the instructions in a different frame?
* As a mechanism for browsers to automatically intervene on resource-intensive frames.


## Pausing Details
When a frame is paused, the browser will:

1. finish any currently executing script, but the rest of the frame's event queue is deferred until the frame is resumed. This means that queued promise resolutions and event firings won't happen until the frame has resumed.

2. defer any network requests created by the frame until the frame has resumed.

3. make it visually obvious to the user that the frame is paused if `showUI` is true. Imagine the frame turning black with a big play button on it. When the user presses the play button, the frame is resumed. If `showUI` is false, no visual change will be made apparent to the user. The frame will just be static and only resumable via script.


