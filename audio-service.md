# The Audio Service (DRAFT)

The audio service is a serverless service which enables mixing of high quality WAV's based on the volume configuration extracted from the stems player.

## Installation

The Sound Web Services Audio Service installer package is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can use the installer to deploy the audio service into your own AWS environment, [but first you must configure npm (or yarn) for use with GitHub Packages](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages).

### Installing the ffmpeg layer

The audio service depends on an [ffmpeg layer](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:145266761615:applications~ffmpeg-lambda-layer) which must be deployed into the AWS environment. Make a note of the ARN which you will need below.

### Generating a shared secret

The algorithm used is `HMAC SHA256`. You can easily generate a secret here [here](https://cryptii.com/pipes/hmac)

### Installing the service

Run

```bash
# make sure your AWS credentials are set
# for $SHARED_SECRET use the shared secret that was created above
# for $FFMPEG_LAMBDA_ARN use the arn of the ffmpeg layer above
npx @sound-ws/audio-service deploy \
    --region eu-west-2 \
    --stage example \
    --secret $SHARED_SECRET \
    --ffmpeg-layer-arn $FFMPEG_LAMBDA_ARN \
    -y
```

This will deploy the audio service. The output will look something like this:

```bash
Service Information
service: sound-ws-audio-service
stage: example
region: eu-west-2
stack: sound-ws-audio-service-example
resources: 41
api keys:
  None
endpoints:
  POST - https://***.execute-api.eu-west-2.amazonaws.com/example/audio/create-mix
  ...
functions:
  ...
layers:
  None
```

Make a note of the `endpoints`. These will be needed later when sending data to the service.

### Security

For reasons of security, install the audio service in an otherwise empty AWS account.

### Updating the service

todo

## Interacting with the audio-service

### Creating a mix

In order to create a mix job we need to post some data to the audio service. Some things to note:

- The `sources` point to the source wav files.
- The `callbackUrl` is a url to which the mixed file will eventually be posted. This can (for example) be a [presigned S3 url](https://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html).
- The `getObjectUrl` is a url to which the browser will eventually be redirected. This can (for example) be a (signed) S3 url.

See also [this example](https://github.com/sound-ws/stems-player-example/blob/master/examples/server/handle-download.js)

```bash
  curl -X POST -d "$(cat << JSON
{
    "sources": [{
      "src": "https://my-stems/wavs/drums.wav?signature=...",
      "volume": 0.4,
    }, {
      "src": "https://my-stems/wavs/vocals.wav?signature=...",
      "volume": 0.2,
    }],
    "callbackUrl": "https://my-mixes.com/receive",
    "getObjectUrl": "https://my-mixes.com/redirect/my-file.wav?signature=..."
}
JSON)" -H 'application/json' https://***.execute-api.eu-west-2.amazonaws.com/example/audio/create-mix
```
