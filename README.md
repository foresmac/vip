# VOKAL Image Proxy

`vip` is an image proxy designed for easily resizing and caching images.


## Usage

Images are served up with a URI that contains a bucket name, as well as a 
unique identifier for the image:
        
        images.example.com/mybucket/5272a0e7d0d9813e21

When images are requested they are placed into an in-memory cache to make repeated
requests for that image faster.

You can resize an image on the fly by providing an `?s=X` parameter that specifies
a maximum width for the image. For example, if you want to resize an image down to a 160 pixel thumbnail:
        
        images.example.com/mybucket/5272a0e7d0d9813e21?s=160

Resized images may also be center-cropped by passing `?c=true` in the querystring:
        
        images.example.com/mybucket/5272a0e7d0d9813e21?s=160&c=true

The thumbnail will then be cached to both `groupcache` and S3. If the image leaves
the in-memory cache it will not need to be resized again.

For performance reasons, `vip` has a configurable maximum width, set via the environment
variable `VIP_MAX_WIDTH`. You'll want to balance your own app's needs with memory needed to
cache larger images, though the default max is a reasonable 720 pixels.

You can also limit the maximum filesize that `vip` can accept by specifying `VIP_SIZE_LIMIT` in megabytes (e.g. `VIP_SIZE_LIMIT=10`). The default is 5MB, which is sufficient for JPEG photos from most mobile devices.

Images are uploaded through `vip` to generate the serving URL. Upload requests should
have the image encoded as the body. `Content-Type` and authentication headers will also
need to be provided. The route for uploads is: `images.example.com/upload/mybucket`

This will upload the supplied image into `mybucket` and return a JSON-encoded serving URL:
```json
{
    "url": "http://images.example.com/mybucket/5272a0e7d0d9813e21"
}
```


## Deployment

`vip` can be deployed with Docker:

        $ docker run -e AWS_SECRET_ACCESS_KEY=... \
            -e AWS_ACCESS_KEY_ID=... vokalinteractive/vip

It is recommended that you deploy `vip` behind SSL and with an authentication token. This
token can be generated by your own application, but should be used by clients when uploading.

Authentication is only checked during uploads. The expected format for authentication is in the
`X-Vip-Token` header:
```
X-Vip-Token: c5411c3aac6f2c6d55a1fdc2d0a98c49
```

To set the token for your deployment, pass it as the environment variable `AUTH_TOKEN` when deploying:
```bash
$ docker run -e AUTH_TOKEN=... -e AWS_SECRET_ACCESS_KEY=... \
    -e AWS_ACCESS_KEY_ID=... vokalinteractive/vip
```

To configure CORS support for browser-based clients, supply a comma separated list (no spaces) of allowed hosts in the environment variable `ALLOWED_ORIGIN`. Setting `ALLOWED_ORIGN=*` allows any host; setting `ALLOWED_ORIGIN=*.project.com` allows any subdomain of project.com. For staging setups, you likely want to allow `localhost` as well, or any upload tests from drone or a local dev environment will fail. Ex: `ALLOWED_ORIGIN=localhost,*.project.com`. (_Note:_ Requests from an allowed origin _do not_ require an `X-Vip-Token`.)


## Cloudfront

`vip` can (and should be) used behind a CDN like Amazon Cloudfront. To use `vip` behind the 
Cloudfront CDN setup a custom origin with the FQDN of your proxy, and also permit `POST` HTTP 
methods on the distribution.

![alt text](https://images.vokalinteractive.com/vokalvip/c528c0a28a980402a236267e60009422?s=650 "VIP was here")


## Maintainer

Chris Foresman (@foresmac) chris.foresman@vokal.io
