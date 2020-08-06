# Serving private content while using the HttpStreamDriver

The Stems Player Browser Component comes with a http-stream driver which consumes m3u8 files which contain an index of urls pointing towards segments of stems.

1 suppose the browser fetches some stem data from your api

```js
import { create } from '@sound-ws/stems-player';

const player = create(document.querySelector('#player'), {...});

fetch(`${yourapi}/stems`)
  .then((response) => {
    // which will return [{ id, src: '...m3u8', waveform: '....json', label }]
    return response.json();
  })
  .then((data) => {
    player.load({
      stems: data,
    });
  });
```

2 the player will then load the m3u8 files for each stem (contained in the data returned in step 1). A m3u8 file looks like:

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

The thing to note here is that the browser needs to be able to access both the m3u8 file _and each segment individually_. Supposing that all these files are hosted via a private CDN, which requires a signed url, either,

a) both the m3u8, and each segment needs to have a fully qualified signed url

however that would mean that the m3u8 file cannot simply be hosted as a static file in an s3 bucket _since each of the segments requires a dynamically generated signed url_.

b) one uses [signed cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-cookies.html). This would mean a single cookie can give access to multiple files and one doe not need to generate a signed url for each file individually.

c) Implement a redirect endpoint: the urls to the m3u8 file and the segments do not point directly to the CDN, but instead to a backend (which may authorize the user based on an existing session cookie), which will then issue a redirect to the cloudfront. The player will follow the redirect. This is what is implemented on [www.sound.ws/stems](www.sound.ws/stems). For example:

```shell
# 1 fetch the stems data from the backend
curl 'https://www.sound.ws/stems/api/stems/?scope=default'
# which results in
# [
#   {
#     "label": "Drums demo",
#     "stems": [
#       {
#         "id": "TRACK_0.0",
#         "label": "Drums B",
#         "waveform": "https://www.sound.ws/stems/api/redirect/default/drums/json/106 DRUMS B_01.5.json",
#         "src": "https://www.sound.ws/stems/api/redirect/default/drums/mp3/106 DRUMS B_01.5.m3u8"
#       },
#       ...
#     ],
#     "id": "TRACK_0"
#   }
# ]

# 2 player.load is called with the stem data

# 3 then the player/browser will request the m3u8 data via this redirect endpoint (and not directly from the CDN).
curl 'https://www.sound.ws/stems/api/redirect/default/drums/mp3/106%20BELL_05.3.m3u8' -i;
# HTTP/2 302
# location: https://***.cloudfront.net/...?Expires=
# [...]

# 4 The browser will then follow the redirect and download the m3u8
# ...
# #EXTINF:10.005333,
# 106 BELL_05.3-000.mp3
# ...

# 5 the player will then proceed to download each stem-segment using the same redirect mechanism
curl 'https://www.sound.ws/stems/api/redirect/default/drums/mp3/BELL_05.3-000.mp3' -i
# HTTP/2 302
# content-type: application/json
# content-length: 733
# location: https://***.cloudfront.net/...?Expires=
```

## See also

- [https://forums.aws.amazon.com/thread.jspa?threadID=158215](https://forums.aws.amazon.com/thread.jspa?threadID=158215)
- [https://stackoverflow.com/questions/39975303/how-to-serve-hls-streams-from-s3-in-secure-way-authorized-authenticated](https://stackoverflow.com/questions/39975303/how-to-serve-hls-streams-from-s3-in-secure-way-authorized-authenticated)
