# The stems player browser component

## Pulling the player code

The Sound Web Services Stems Player is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can pull the npm package and use it in your application, [but first you must configure npm (or yarn) for use with GitHub Packages](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages).

then create a package.json with the stems-player package in its dependencies.

```json
{
  "name": "@my-org/my-app",
  "version": "1.0.0",
  "description": "Browser app that uses the @sound-ws/stems-player package",
  "main": "index.js",
  "dependencies": {
    "@sound-ws/stems-player": "~1.0"
  }
}
```

then install the player

```sh
npm install
```

## Instantiating the Stems Player

1. Create a container where the Stems player should be placed into.

```html
<div id="my-stems-player"></div>
```

```js
// Either using es6
import { StemsPlayer } from '@sound-ws/stems-player';
// or es5
// const StemsPlayer = require('@sound-ws/stems-player');

const player = StemsPlayer.create('#my-stems-player', {
  // See the player options below
  renderer: {
    background: 'rgba(0,0,0,0.6)',
    color: 'white',
    backgroundStem: 'rgba(0,0,0,0.1)',
    waveform: {
      waveColor: 'red',
      progressColor: '#63ddb3',
      barWidth: 0,
      barGap: 0,
    },
  },
});
```

## Parameters

### Options for the Renderer

The renderer accepts the following options

```js
const player = StemsPlayer.create('#my-stems-player', {
  background: 'rgba(0,0,0,0.6)',
  color: 'white',
  backgroundStem: 'rgba(0,0,0,0.1)',
  waveform: {
    // see https://wavesurfer-js.org/docs/options.html for a full list of parameters
    waveColor: 'red',
    progressColor: '#63ddb3',
    barWidth: 0,
    barGap: 0,
  },
});
```

The HttpStreamDriver accepts the following options

```js
const player = StemsPlayer.create('#my-stems-player', {
  driver: {
    audioContext: myOwnAudioContext, // = optional
  },
});
```

## Loading the audio

```js
fetch('http://my-backend-api/my-stems/stem-1234/index.json')
  .then((response) => {
    return response.json();
  })
  .then((data) => {
    player
      .load({
        stems: data.stems.map((stem) => {
          return {
            id: stem.id,
            src: stem.src,
            waveform: stem.waveform,
            label: stem.label,
          };
        }),
      })
      .then(() => {
        // The player is ready to start playing
      });
  });
```

## API

You can also call various functions to control the player: `player.start, player.pause, player.stop, player.destroy, player.export`. Most should be self-explanatory, however we will highlight `destroy` and `export`.

### Destroying the player

To keep memory consumption in check, be sure to destroy the player when you no longer need it.

### Exporting data from the player

One can export the state of the player by calling `player.export`. This will then return a simple json object of the form

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

which can be used to send to The Sound Web Services Audio Service in order to generate a high quality mix. (This will be a separate component. Documentation on this to follow)

## Events

You can bind to these events

```js
player.on('ready', () => {
  player.play();
});

player.on('error', (err) => {
  console.error('An error has occured', err);
});
```
