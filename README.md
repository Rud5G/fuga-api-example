# fuga-api-example
This repository will help you how to use Fuga API to create releases, add tracks to it and have it ready for delivery.

Examples here are shown in `cUrl` command line for making HTTP requests and handle cookies.
For more examples in other languages, check our [public postman API docs](https://documenter.getpostman.com/view/1173967/fuga-api-example/7187Esp)

## Flow:
- [Login](#1-login)
- [Create an artist](#2-create-an-artist)
- [Create a label](#3-create-a-label)
- [Create a release with required metadata](#4-create-a-release)
- [Upload a cover art to the release](#5-upload-releases-cover-art-file)
- [Create a track with required metadata](#6-create-a-track)
- [Upload an audio file to the track](#7-upload-tracks-audio-file)
- [Link track to the release](#8-link-track-to-the-release)
- [Submit for review](#9-submit-for-review)

## 1. Login
First of all you need to `POST` to `/login` with the credentials provided. That will return a `set-cookie` (name: `connect.sid
`) in your response header which is needed for the following calls.

```
curl -c fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/login/ \
  -H 'content-type: application/json' \
  -d '{
    "password": "YOUR_PASSWORD_HERE",
    "name": "YOUR_USER_NAME"
  }'
```

Notice that we are using `-c` option to save a text file with the cookie information, that's managed by `cUrl` and we have to specify that for the next requests that needs authorization:

```
curl -L -b fuga-api-example-cookie-jar.txt  -X ...
```

If you prefer you can pass it as a Header in your `cUrl` requests:

```
-H 'Cookie: connect.sid={{CRAZY_COOKIE_VALUE}}' \
```

## 2. Create an artist
Creates an artist to be used in the release's creation.

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/artists \
  -H 'content-type: application/json' \
  -d '{
    "name": "My Awesome Artist"
  }'
```
In the `json` response you can save the `id` to be used later.

## 3. Create a label
Creates a label to be used in the release's creation.

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/labels \
  -H 'content-type: application/json' \
  -d '{
    "name": "My Awesome Label"
  }'
```
In the `json` response you can save the `id` to be used later.

## 4. Create a release
Finally, to create a release we need to provide some required fields, including the artist and label just created in previous steps. This is how the request should look like:

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/products \
  -H 'content-type: application/json' \
  -d '{
    "name": "Awesome Release",
    "label": {{LABEL_ID}},
    "catalog_number": "CAT-NUM-123",
    "upc": "{{UPC_CODE}}",
    "c_line_text": "album art copyright info",
    "p_line_text": "performing/recording copyright info",
    "c_line_year": 2017,
    "p_line_year": 2017,
    "genre": "ELECTRONIC",
    "artists": [{
      "primary": true,
      "id": {{ARTIST_ID}}
    }]
  }'
```

**upc**: It's a EAN-13 bar code associated with the release, that's how stores identifies your release among others. You can generate one online using this [web tool](https://www.free-barcode-generator.net/ean-13/).

**catalog_number**: It's an internal identifier used by labels and required in order to publish your release. It's a free string value, usually a combination of text and numbers.

**genre**: some of the allowed values for genre:
```
"ALTERNATIVE"
"BLUES"
"CLASSICAL"
"COUNTRY"
"DANCE"
"ELECTRONIC"
"FOLK"
"HIP_HOP_RAP"
"JAZZ"
"LATIN"
"NEW_AGE"
"OPERA"
"POP"
"R_B_SOUL"
"REGGAE"
"ROCK"
"VOCAL"
```

## 5. Upload release's cover art file
After creating the release you need the `cover_image.id` in order to start the upload process. Uploading something to Fuga API requires 3 steps:

**Cover art files restrictions**:
- Minimum dimensions of `1400x1400`;
- Allowed extensions: `JPG`, `PNG` or `GIF`;

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload/start \
  -H 'content-type: application/json' \
  -d '{
    "id": {{RELEASE_COVER_IMAGE_ID}},
    "type": "image"
  }'
```

This will return a response like `{ "id": "GENERATED_TOKEN" }`, this token is required to upload the file. 

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload \
  -H 'content-type: multipart/form-data' \
  -F uuid={{GENERATED_TOKEN}} \
  -F file=@{{FILE_PATH}} \
  -F partbyteoffset=0
```

Once it's done you need to tell the API the upload's finished:

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload/finish \
  -H 'content-type: application/json' \
  -d '{
    "uuid": "{{GENERATED_TOKEN}}",
    "filename": "whatever_filename_you_want"
  }'
```


## 6. Create a track
A track needs to be first created and then linked to a release.

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/assets \
  -H 'content-type: application/json' \
  -d '{
    "name": "My Awesome track",
    "isrc": {{ISRC_CODE}},
    "genre": "ELECTRONIC",
    "artists": [{
      "primary": true,
      "id": {{ARTIST_ID}
    }]
  }'
```



**ISRC_CODE**: The unique identifier for a track. That can also be generated for demonstration purposes in this [web tool](http://salomvary.github.io/random-isrc/)

*Note: you can create as many tracks you want and them link them all to your release, covered in [step 8](#8-link-track-to-the-release)* 


## 7. Upload track's audio file
To start the audio upload you need the track `id` that you just created in the previous step. Again, three steps are required in order to accomplish the upload:

**Audio files restrictions**:
- Needs a stereo, 16 or 24 bit between 44.1 and 96 kHz `flac` or `wav`;
- Duration must be greater than zero;

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload/start \
  -H 'content-type: application/json' \
  -d '{
    "id": {{TRACK_ID}},
    "type": "audio"
  }'
```

This will return a response like `{ "id": "GENERATED_TOKEN" }`, this token is required to upload the file. 

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload \
  -H 'content-type: multipart/form-data' \
  -F uuid={{GENERATED_TOKEN}} \
  -F file=@{{FILE_PATH}} \
  -F partbyteoffset=0
```

Once it's done you need to tell the API the upload's finished:

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/upload/finish \
  -H 'content-type: application/json' \
  -d '{
    "uuid": "{{GENERATED_TOKEN}}",
    "filename": "whatever_filename_you_want"
  }'
```

Check out `/api/catalog/assets/{TRACK_ID}/audio` to have a `mp3` version of your just uploaded track. You can even build a player for it :)

```
curl -L -b fuga-api-example-cookie-jar.txt -X GET \
  https://next.fugamusic.com/api/catalog/assets/{TRACK_ID}/audio >> my-downloaded-file.mp3
```

todo: VALIDATION


## 8. Link track to the release
To link a track to a release you need to `POST` to `/catalog/products/{id}/assets` with the track id in the body `{"id": track_id }`.

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/products/{{RELEASE_ID}}/assets \
  -H 'content-type: application/json' \
  -d '{
    "id": {{TRACK_ID}}
  }'
```

You can check your release details with a `GET` to `/api/catalog/products/{RELEASE_ID}`:

```
curl -L -b fuga-api-example-cookie-jar.txt -X GET \
  https://next.fugamusic.com/api/catalog/products/{{RELEASE_ID}}
```

And the track details:
```
curl -L -b fuga-api-example-cookie-jar.txt -X GET \
  https://next.fugamusic.com/api/catalog/assets/{{TRACK_ID}}
```


## 9. Submit for review
Now your release is ready to be submitted for review and delivery to the stores :)

Notice that there is a `state` property in the product|release is `PENDING`, by submitting it that will change to `SUBMITTED` and during the review process it can become either `ACCEPTED` or `REJECTED`.

In order to submit your release to review, `POST` to `/catalog/{RELEASE_ID}/reviews`:

```
curl -L -b fuga-api-example-cookie-jar.txt -X POST \
  https://next.fugamusic.com/api/catalog/products/{{RELEASE_ID}}/reviews \
  -H 'content-type: application/json' \
  -d '{
    "message": "Please accept my awesome release"
  }'
```

The response is the release itself with the property `last_review_item` filled up. Check it out.
