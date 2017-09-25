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

## Pause Details
Each frame will have a `pausedCounterUA` and a `pausedCounterAPI` member, which initialize to zero and may never drop below zero. When the sum of the two members is zero, the frame is unpaused, else paused.

When `pause` is called on a frame by the API, `pausedCounterAPI` is incremented in the frame and in its descendant frames. When a frame is paused by the UA, `pausedCounterUA` is incremented in the frame and its descendant frames.

## Unpause Details
When `unpause` is called on a frame by the API, `pausedCounterAPI` is decremented for the frame and its descendants. If the counter is already zero, it is not decremented. Likewise, if the frame is unpaused by UA, the `pausedCounterUA` is decremented in the frame and its descendants, not to go below zero.

## Paused details
For same-origin frames, `frameElement.paused` will return true if the sum of the frame's `pausedCounterUA` and `pausedCounterAPI` is greater than zero. For cross-origin frames, `frameElement.paused` will return true if `pausedCounterAPI` is greater than zero. This is to ensure that intervention information about a cross-origin frame is not leaked.

## Behavior of a Paused Frame
A paused frame behaves like a static image. Any animated images or media elements pause. No future javascript events (e.g., onload, onclick) are fired, no enqueued tasks are run, and no default handlers (e.g., navigating on a click) are fired within the paused frame. Instead, the events will be queued. Further, while paused, the frame will not navigate (e.g., meta refresh will not work), CSS animations won't run, and the frame won't render, therefore there are no  `requestAnimationFrame` (rAF) callbacks.

There are a few times that the frame might need to be rerendered (e.g., on frame resize or scrolling). When that happens, the frame will be rendered again, and rAF will be fired, as needed.


## Privacy Issues

1. Pausing a frame causes it to stop processing. Measuring task responsiveness before and after pausing may help the embedding page to determine the CPU load of a cross-origin frame at a given moment. This could already be accomplished by measuing the load before and after unloading the frame. But this is less intrusive.
