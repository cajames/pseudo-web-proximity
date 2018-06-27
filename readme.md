# Pseudo Web Proximity

> Fun project using the camera to mimic a phone's proximity sensor.

This was a project to try mimic a proximity sensor on a web-app, similar to what's available on native apps.

Currently, the [Web Proximity Events spec](https://developer.mozilla.org/en-US/docs/Web/API/Proximity_Events) is still in draft mode. So in the mean-time this is a _kinda functional_ work-around.

Notes:

- Fell back to using the `navigator.getUserMedia` API due to some issues using the newer [`navigator.mediaDevices.getUserMedia`](https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia) API on mobile safari.
- Mobile safari is an intersting use case and hence is still a work in progress.