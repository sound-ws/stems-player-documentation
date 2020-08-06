# The Stems Player Browser Component (DRAFT)

The [Sound Web Services Stems Player](https://www.sound.ws/stems) can play stems in the browser using audio served directly using Content Delivery Networks, such as Cloudfront.

## Browser Support

The Player works in [browsers supporting Web Audio](https://caniuse.com/#feat=audio-api). This includes most modern browsers.

In addition, the player component is transpiled by babel to support `">0.25%, not dead", "not ie 11", "not op_mini all"`, meaning

- Any browser which at the time of compilaton has more than `0.25%` usage.
- IE 11 and Opera Mini are not supported since neither supports the [web-audio api](https://caniuse.com/#search=webaudio)
- "Not dead" is defined as ["browsers without official support or updates for 24 months. Right now it is IE 10, IE_Mob 11, BlackBerry 10, BlackBerry 7, Samsung 4 and OperaMobile 12.1."](https://github.com/browserslist/browserslist/blob/master/README.md).

## Pulling the player code

The Sound Web Services Stems Player is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can pull the npm package and use it in your application, [but first you must configure npm (or yarn) for use with GitHub Packages](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages).

then simply execute `npm install @sound-ws/stems-player@1.0.0` in the application root (containing the package.json).

## Instantiating the Stems Player

Create a container where the Stems player should be placed into.

```html
<div id="my-stems-player"></div>
```

Using a module bundler such as webpack, create an instance of the player.

```js
// Either using es6 import
import { create } from '@sound-ws/stems-player';
// or es5 commonjs/requirejs
// const { create } = require('@sound-ws/stems-player');

const player = create('#my-stems-player', {
  // See the player options below
});
```

Or load the stems-player library into your HTML page

```html
<script src="node_modules/@sound-ws/stems-player/dist/sound-ws-stems-player.js"></script>
<div id="my-stems-player"></div>
...
<script>
  const player = SoundWS.StemsPlayer.create('#my-stems-player', {
    // See the player options below
  });
</script>
```

## Options

The player accepts the following options

```js
const player = StemsPlayer.create('#my-stems-player', {
  renderer: {
    background: 'rgb(52,58,64)',
    backgroundStem: 'rgba(74,82,90)',
    color: 'white',
    accentColor: '#39d49e',
    waveform: {
      // The stems player uses the drawer from wavesurfer
      // see https://wavesurfer-js.org/docs/options.html for a full list of accepted parameters for styling the waveform
      waveColor: '#bbb',
      progressColor: '#39d49e',
    },
  },
  driver: {
    audioContext: myOwnAudioContext, // = optional
  },
});
```

## Loading the audio

### The source M3U8 file

The player consumes a set of m3u8 files. Each m3u8 file represents a single stem, split out into segments of varying duration and start times.

```txt
#EXTM3U
#EXT-X-VERSION:3
#EXT-X-MEDIA-SEQUENCE:0
#EXT-X-ALLOW-CACHE:YES
#EXT-X-TARGETDURATION:11
#EXTINF:10.002667,
FC_EVO_311_1_Bring_You_Down_5_Piano-000.mp3
#EXTINF:10.002667,
FC_EVO_311_1_Bring_You_Down_5_Piano-001.mp3
#EXTINF:10.002667,
FC_EVO_311_1_Bring_You_Down_5_Piano-002.mp3
#EXTINF:10.002667,
...
#EXT-X-ENDLIST
```

Look here for [instructions on how to generate segments of the source stems and m3u8 files](https://github.com/sound-ws/docker-generate-stems).

### Example loading

```js
// We're assuming a hypothetical endpoint, maintained by the client, that returns data relating to stems
fetch('http://my-backend-api/my-stems/stem-1234')
  .then((response) => {
    return response.json();
  })
  .then((data) => {
    player
      .load({
        'TRACK1234', // any sort of track id
        label: data.trackName,
        stems: data.stems.map((stem) => {
          return {
            id: stem.id, // any
            label: stem.label, // string
            src: stem.src, // string (a url pointing to a m3u8 document)
            waveform: stem.waveform, // string (a url pointing to a json document)
          };
        }),
      })
      .then(() => {
        player.start();
      });
  });
```

Please make sure that when loading any files from another server that the response contains adequate [CORS headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS).

## Waveforms

Due to the fact that the player only downloads segments of each stem at a time, generating the waveform on the client is not feasible. Therefore the waveforms need to be pre-generated. The player currently uses the same waveform drawer as Wavesurfer [where you can find instructions on how to generate the waveforms](https://wavesurfer-js.org/faq/).

## API

You can also call various functions to control the player: `player.start(), player.pause(), player.stop(), player.destroy(), player.export(), player.seek(percentage)`. Most should be self-explanatory, however we will highlight `player.destroy()` and `player.export()`.

### Destroying the player

To keep memory consumption in check, be sure to destroy the player when you no longer need it.

### Exporting data from the player

One can export the state of the player by calling `player.export()`. This will then return a simple json object of the form.

```json
{
  "id": "TRACK1234",
  "volume": 1,
  "label": "The name of TRACK1234",
  "stems": [
    {
      "volume": 0.5,
      "id": "TRACK1234_DRUMS",
      "label": "Drums"
    },
    {
      "volume": 0.1,
      "id": "TRACK1234_PIANO",
      "label": "Piano"
    }
  ]
}
```

which can be used to [send to The Sound Web Services Audio Service](integration.md) in order to generate a high quality mix.

### Events

You can bind to these events

```js
// when the player has initialised and is ready to start playing
player.on('ready', () => {});

// When the user seeks
player.on('play', () => {});

// When the user pauses
player.on('pause', () => {});

// When the user seeks
// typeof pct === number
player.on('seek', ({ pct }) => {});

// When the player starts loading data
player.on('buffering-start', () => {});

// When the player is done loading data
player.on('buffering-end', () => {});

// When an error occurs
// error instanceof Error
player.on('error', (error) => {});

// When the player completed playback
player.on('end', () => {});

// When the player is destroyed
player.on('destroy', () => {});
```

## Example

See also the [example](https://github.com/sound-ws/stems-player-example)

## Issues

Please log issues [here](https://github.com/sound-ws/stems-player/issues). (Private repo)

## Links

- https://developers.google.com/web/updates/2017/09/autoplay-policy-changes
- https://wavesurfer-js.org/docs/options.html
