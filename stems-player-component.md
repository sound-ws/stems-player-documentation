# The stems player browser component

## Pulling the player code

The Sound Web Services Stems Player is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can pull the npm package and use it in your application, [but first you must configure npm (or yarn) for use with GitHub Packages](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages).

then create a package.json with the stems-player package in it's dependencies.

```json
{
  "name": "@my-org/server",
  "version": "1.0.0",
  "description": "Browser app that uses the @sound-ws/stems-player package",
  "main": "index.js",
  "dependencies": {
    "@sound-ws/stems-player": "~1.0"
  }
}
```

then install the player

```bash
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

player.on('error', (err) => {
  console.error('An error occured', err);
});

fetch('http://my-backend-api/my-stems/stem-1234/index.json')
  .then((response) => {
    return response.json();
  })
  .then((data) => {
    player
      .load({
        stems: data.stems.map((stem) => {
          // Below the properties that are expected...
          return {
            id: stem.id,
            src: stem.src,
            waveform: stem.waveform,
            label: stem.label,
          };
        }),
      })
      .then(() => player.start());
  });
```

## API

## Events

- ready
- unload
- error
