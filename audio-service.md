# The Sound Web Services Audio Service (DRAFT)

The audio service is a serverless service which enables mixing of high quality WAV's based on the volume configuration extracted from the stems player.

## Installation

The Sound Web Services Audio Service installer package is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can use the installer to deploy the audio service into your own AWS environment, [but first you must configure npm (or yarn) for use with GitHub Packages](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages).

### Installing the ffmpeg layer

The audio service depends on an (ffmpeg layer)[https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:145266761615:applications~ffmpeg-lambda-layer] which must be deployed into the AWS environment. Make a note of the ARN which you will need below.

### Generating a shared secret

todo

### Installing the service

Run

```bash
# make sure your AWS credentials are set
# for secret use the shared secret that was created above
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
  GET - https://***.execute-api.eu-west-2.amazonaws.com/example/audio/{uuid}/status
  GET - https://***.execute-api.eu-west-2.amazonaws.com/example/audio/{uuid}
functions:
  createAudioMix: example-stems-service-createAudioMix
  getAudioMixStatus: example-stems-service-getAudioMixStatus
  getAudioMix: example-stems-service-getAudioMix
  consumeJobResponseMessage: example-stems-service-consumeJobResponseMessage
  accountAuthorizer: example-stems-service-accountAuthorizer
  jobAuthorizer: example-stems-service-jobAuthorizer
  consumeJob: example-stems-service-consume-job
layers:
  None
```

Make a note of the `endpoints`. These will be needed later when sending data to the service.

### Security

For reasons of security, it is strongly adviced to install the audio service in an otherwise empty AWS account.

## Interacting with the audio-service

Use the [AudioServiceClient](https://github.com/sound-ws/audio-service-client) to interact with the service.
