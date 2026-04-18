# Player API

The Video.js player instance provides methods and properties to control video playback programmatically.

## Methods

### Playback Control

#### `play()`
Start video playback.

```js
player.play();
```

#### `pause()`
Pause video playback.

```js
player.pause();
```

#### `paused()`
Returns `true` if the player is paused.

```js
if (player.paused()) {
  player.play();
}
```

### Time Control

#### `currentTime(seconds)`
Get or set the current playback time in seconds.

```js
// Get current time
const currentTime = player.currentTime();

// Seek to 30 seconds
player.currentTime(30);
```

#### `duration()`
Returns the duration of the video in seconds.

```js
const duration = player.duration();
```

### Volume Control

#### `volume(percent)`
Get or set the volume as a decimal between 0 and 1.

```js
// Get current volume
const volume = player.volume();

// Set volume to 50%
player.volume(0.5);
```

#### `muted(muted)`
Get or set the muted state.

```js
// Get muted state
const isMuted = player.muted();

// Mute the player
player.muted(true);
```

### Source Management

#### `src(source)`
Get or set the video source.

```js
// Set source
player.src({
  src: 'https://example.com/video.mp4',
  type: 'video/mp4'
});
```

#### `poster(src)`
Get or set the poster image URL.

```js
player.poster('https://example.com/poster.jpg');
```

### UI Control

#### `controls(show)`
Get or set the visibility of the control bar.

```js
// Hide controls
player.controls(false);
```

#### `fullscreen(enter)`
Enter or exit fullscreen mode.

```js
// Enter fullscreen
player.requestFullscreen();

// Exit fullscreen
player.exitFullscreen();
```

## Events

Listen to player events using the `on` method.

```js
player.on('play', () => {
  console.log('Playback started');
});

player.on('pause', () => {
  console.log('Playback paused');
});

player.on('ended', () => {
  console.log('Playback ended');
});
```

## Properties

#### `readyState`
Returns the current ready state of the player.

#### `networkState`
Returns the current network state.

#### `error`
Returns the current error object if any.
