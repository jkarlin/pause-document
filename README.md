# Pausing Frames

There are times for frames to be busy, and times when the work they're doing is wasteful. When they're wasteful, frames should be paused so that they don't bother the user. When a frame is paused, it completes its current tasks and resource loads but doesn't run any further tasks (e.g., event handlers, promises resolving, video loops) until the frame is unpaused. 

Pausing may be initiated by conscientious script or by the browser in order to intervene on the user's behalf. If initiated by script, it can be unpaused by script. If initiated by the browser as part of an intervention, it can only be unpaused by the browser. 

The JavaScript API to pause a frame looks like:

```javascript
var frameElement = document.getElementById("hungryFrameID");
await frameElement.pause();
console.log("Frame is paused");
```

And when you're ready to unpause the frame:
```javascript
await frameElement.unpause();  // does nothing if frame isn't paused
console.log("Frame is unpaused");
```

To detect if a frame is paused:
```javascript
if (frameElement.paused) 
  alert("The frame is paused");
```

## Use Cases
* Pausing frames that violate policies (e.g., [TransferSizePolicy](https://github.com/WICG/transfer-size)). This provides a gentle, yet firm response to misbehaving frames.
* Pausing resource-intensive frames that the user isn't currently paying attention to. For instance, a carousel of frames, where some of the frames are offscreen.
* As a mechanism for browsers to automatically intervene on resource-intensive frames.

## UX Considerations
The user is likely to become confused by the frame becoming a static image. E.g., clicking on links won't work. It is therefore advisable that if you pause a visible frame that you (the web developer) make it visually obvious to the user that it's not interactive (e.g., overlay a semi-transparent play button over the paused frame).

## Pausing Details
When a frame is paused, the frame effectively becomes a static image. Any animated images or media elements pause. No javascript events (e.g., onload, onclick) are fired, and no default handlers (e.g., navigating on a click) are fired within the paused frame. Instead, the events will be queued. Further, while paused, the frame will not navigate (e.g., meta refresh will not work), CSS animations won't run, and the frame won't render, therefore there are no  `requestAnimationFrame` (rAF) callbacks. 

There are a few times that the frame might need to be rerendered (e.g., on frame resize or the browser dropped a texture while the frame was offscreen). When that happens, the frame will be rendered once, and rAF will correspondingly be called once beforehand.

