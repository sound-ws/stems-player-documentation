# The Audio Service (self-hosted) (DRAFT)

The audio service is a serverless service which enables mixing of high quality WAV's based on the volume configuration extracted from the stems player.

## Technology

Internally, the download API uses:

- AWS API gateway
- AWS Lambdas running Nodejs
- AWS SQS
- AWS Dynamodb
- A ffmpeg lambda layer which needs to be pre-installed

## Installation

The Sound Web Services Audio Service installer package is hosted on a private github npm repository. Once you have been granted read-access to this repository, you can use the installer to deploy the audio service into your own AWS environment, [but first you must configure npm (or yarn) for use with GitHub Packages](using-github-packages.md).

### Installing the ffmpeg lambda layer

The audio service depends on an [ffmpeg layer](https://serverlessrepo.aws.amazon.com/applications/arn:aws:serverlessrepo:us-east-1:145266761615:applications~ffmpeg-lambda-layer) which must be deployed into the AWS environment. Make a note of the ARN which you will need below.

### Generating a shared secret

The algorithm used is `HMAC SHA256`. You can generate a secret here [here](https://cryptii.com/pipes/hmac)

### Dependencies

The installer depends on [serverless framework](https://www.serverless.com). Make sure that is installed.

In addition the AWS credentials used for deployment needs to have sufficient permissions.

```json
    "Version": "2012-10-17",
    "Statement": [
        {
            ...
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "lambda:*",
                "s3:*",
                "iam:*",
                "logs:*",
                "apigateway:*",
                "sqs:*",
                "dynamodb:*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

### Installing the service

Run

```bash
# make sure your AWS credentials are set
# stage can be anything, e.g. uat|prod
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

### Updating the service

Simply run `npx @sound-ws/audio-service deploy ...` against an existing STAGE. Seprate upgrading instructions will be provided [here](upgrading.md), if updating to a new major version requires additional steps.

### Destroying the service

Go to the AWS console / Cloudformation and delete the relevant stack (e.g. `sound-ws-audio-service-example`.).

## Interacting with the audio-service: Creating a mix

We provide here some low-level description on how to interact with the audio service. However some of this complexity will be handled by the [Audio Service Client](https://github.com/sound-ws/audio-service-client).

In order to create a mix job we need to post some data to the audio service. Some things to note:

- The `sources` is an array of objects `{ src: "...", volume: 1 }`, where `src` points to the source wav file and `volume` is a number indicating the volume to be used when mixing.
- The `callbackUrl` is a url to which the mixed file will eventually be posted. This can (for example) be a [presigned S3 url](https://docs.aws.amazon.com/AmazonS3/latest/dev/PresignedUrlUploadObject.html).
- The `getObjectUrl` is a url to which the browser will eventually be redirected. This can (for example) be a (signed) S3 url.
- The token is a valid jsonwebtoken

See also [this example](https://github.com/sound-ws/stems-player-example/blob/master/examples/server/handle-download.js)

```bash
DATA="$(cat << JSON
{
  "sources": [
    {
      "src": "https://my-stems/wavs/drums.wav?signature=...",
      "volume": 0.4
    },
    {
      "src": "https://my-stems/wavs/vocals.wav?signature=...",
      "volume": 0.2
    }
  ],
  "callbackUrl": "https://my-mixes.com/receive",
  "getObjectUrl": "https://my-mixes.com/redirect/my-file.wav?signature=..."
}
JSON
)"

# Generate a jwt using the secret
# see also https://github.com/sound-ws/stems-player-example/blob/master/examples/server/handle-download.js for an example
TOKEN=***

curl -X POST -d "$DATA" -H "Content-Type: application/json" -H "Authorization: Bearer $TOKEN" https://vyprqjhbc4.execute-api.eu-west-2.amazonaws.com/example/audio/create-mix -i
# which results in
# HTTP/2 202
# ...
# location: https://***.execute-api.eu-west-2.amazonaws.com/example/audio/1eb73580-d65c-11ea-914e-c3cb25a88108/status?token=***
# access-control-expose-headers: Location, Refresh
# refresh: 5; url=https://***.execute-api.eu-west-2.amazonaws.com/example/audio/1eb73580-d65c-11ea-914e-c3cb25a88108/status?token=***

# {"message":"Your request has been accepted. Redirecting to https://***.execute-api.eu-west-2.amazonaws.com/example/audio/1eb73580-d65c-11ea-914e-c3cb25a88108/status?token=*** in 5 seconds.","job":{"uuid":"1eb73580-d65c-11ea-914e-c3cb25a88108","status":"STATUS_QUEUED","createdAt":"Tue, 04 Aug 2020 14:09:32 GMT","completedAt":null,"timeTaken":null}}

```

The `location` header contains a url which provides information on the status of the mixing job which can be polled

```bash
curl https://***.execute-api.eu-west-2.amazonaws.com/example/audio/3be429e0-d65e-11ea-8aa7-a5222775c1c3/status?token=*** -i
# which results in
# HTTP/2 200
# content-type: application/json
# ...

# {"message":"Something went wrong while processing your request. Please try again later.","job":{"uuid":"3be429e0-d65e-11ea-8aa7-a5222775c1c3","status":"STATUS_FAILED","createdAt":"Tue, 04 Aug 2020 14:24:40 GMT","completedAt":"Tue, 04 Aug 2020 14:24:43 GMT","timeTaken":2574,"error":{"description":"The audio could not be processed","code":6}}}
```

This response above indicated a failure, together with a brief description of the error.

If the job was successful the response would look something like:

```bash
curl https://***.execute-api.eu-west-2.amazonaws.com/example/audio/3be429e0-d65e-11ea-8aa7-a5222775c1c3/status?token=*** -i
# which results in
# HTTP/2 303
# location: https://***.execute-api.eu-west-2.amazonaws.com/example/audio/1eb73580-d65c-11ea-914e-c3cb25a88108?token=***
# ...

# {"message":"Something went wrong while processing your request. Please try again later.","job":{...}}}
```

The url in the `location` header can then be consulted:

```bash
curl https://***.execute-api.eu-west-2.amazonaws.com/example/audio/3be429e0-d65e-11ea-8aa7-a5222775c1c3?token=*** -i
# which results in
# HTTP/2 201
# location: https://my-mixes.com/redirect/my-file.wav?signature=...
# refresh: 5; url=https://my-mixes.com/redirect/my-file.wav?signature=...
# ...

# {"message":"The job completeted, redirecting to ...","job":{...}}}
```

Which leads to a response with a location header pointing to the `getObjectUrl` which was provided by the client above.

## Security

For reasons of security, install the audio service in an otherwise empty AWS account.

The `getObjectUrl` will provide access to the mixed audio. The sources also provide access to your audio. These data will be briefly stored in the service, but will be automatically deleted after 15 minutes.
However, you should not provide urls with excessively long expiry. Expiry of any signed url should be maximum 15 minutes, or less - but longer enough to allow processing to complete.

## Example

See also the [example](https://github.com/sound-ws/stems-player-example)

## Issues

Please log issues [here](https://github.com/sound-ws/audio-service/issues). (Private repo)
