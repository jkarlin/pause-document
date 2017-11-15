# Pausing Documents

There are times for documents (notably those of iframes) to be busy, and times when the work they're doing is wasteful. When they're wasteful, document activity should be paused so that it doesn't bother the user or waste resources. With the PauseDocument API, web developers can pause (and later unpause) a frame's loading, rendering, and javascript execution.

Pausing may be initiated by conscientious script or by the browser in order to intervene on the user's behalf. If initiated by script, it can be unpaused by script. If initiated by the browser as part of an intervention, it can only be unpaused by the browser. 

The JavaScript API to pause a frame looks like:

```javascript
var frameElement = document.getElementById("hungryFrameID");
await frameElement.pause();
console.log("Frame is paused");
```

When you're ready to unpause the frame:
```javascript
await frameElement.unpause();
console.log("Frame is unpaused");
```

To detect if a frame is paused:

```javascript
if (frameElement.paused) 
  console.log("Rendering is paused");
```

## Pausing Details
### Pausing Loading
Existing loads will continue but new resource requests (fetches) will queue instead of starting. When unpaused, the queued requests will start.

### Pausing Rendering
The frame will stop its rendering pipeline and will continue to show the results of the last paint. The browser may, if necessary, rerender the frame (e.g., because the frame is resized or the frame buffers were dropped while offscreen and it scrolled back into view). The rAF event will not fire on rerender (because script is also paused).

### Pausing Script
Any running script task is completed and future tasks for the given frame on the event queue (e.g., events [including WebSocket] and timers) remain queued until the document is unpaused. UI events are discarded instead of queued since there can be so many of them. rAF events will also be discarded. 

## Use Cases
* Pausing frames that violate policies (e.g., [TransferSizePolicy](https://github.com/WICG/transfer-size)). This provides a gentle, yet firm response to misbehaving frames.
* Pausing resource-intensive frames that the user isn't currently paying attention to. For instance, a carousel of frames, where some of the frames are offscreen.
* As a mechanism for browsers to automatically intervene on resource-intensive frames.

## UX Considerations
The user is likely to become confused by the frame being paused, as it is no longer interactive. It is therefore advisable that if you pause a visible frame that you (the web developer) make it visually obvious to the user that it's not interactive (e.g., overlay a semi-transparent play button over the paused frame that unpauses when clicked).

## Pause Details
Each frame has two paused booleans, `apiPaused` and `uaPaused`. 

When a frame is paused by API, `apiPaused` is set to true for that frame (and its descendant frames). When a frame is paused via UA, `uaPaused` is set to true for that frame (and its descendant frames). A frame is paused if either `apiPaused` or `uaPaused` is true.

Pause is an asynchronous operation and returns a promise. It's async because there are multiple frames that need to coordinate. The promise resolves once all of the frame tree has updated and applied its paused options.

## Unpause Details
When `unpause` is called on a frame via API, `apiPaused` for the frame and its descendants is set to false. When a frame is unpaused via UA, `uaPaused` is set to false for that frame and its descendants. A frame is paused if both `apiPaused` and `uaPaused` are now false.

This means that the API can clear its own pausing options but not that of the UA and vice-versa. 

Unpause is an asynchronous operation and returns a promise that resolves after all of the frame tree has updated and applied its paused options.

## Paused details
`frameElement.paused` returns `apiPaused || uaPaused` for the given frame. It returns synchronously. If a `pause` or `unpause` operation is in progress, the result will depend on how far along the operation is.

## Paused/Unpaused Events
Paused frames, and parents of paused frames, may want to know when a frame has been paused. This is TBD.

## Cross-Origin Information Leaked
1. The effects of pausing may be observable to the parent. For instance, if a child frame's is paused, the parent might notice that loads become quicker (suggesting the child was loading resources). Or, script might become more responsive in the parent after the child frame is paused, suggesting the child was busy processing. These are things that could be discovered other ways, such as by unloading the frame.

2. The embedding page now has the ability to freeze the rendered output of a cross-origin frame at any point. Perhaps the cross-origin frame wants something to be displayed momentarily but now it's made permanent. Might this be used for nefarious purposes by the embedder?
