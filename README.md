## Introduction
__Foresight.js__ gives webpages the ability to see if the user's device is capable of viewing high-resolution images (such as the 3rd generation iPad) before the image is requested. Additionally, it judges if the user's device currently has a fast enough network connection for high-resolution images. Depending on device display and network connectivity, __foresight.js__ will request the appropriate image for the webpage. Foresight deals with modifying context images, specifically _img_ elements. Media queries however should be used when dealing with CSS background-images, while foresight.js is used to handle inline _img_ elements.

This project's overall goal is to tackle these current issues faced by web developers designing for hi-res: [Challenges for High-Resolution Images](//github.com/adamdbradley/foresight.js/wiki/Challenges-for-High-Resolution-Images). Foresight is used to modify _img src_ attributes so browsers can request the correct image for the device, it does not however, resize the images themselves. See [Server Resizing Images](//github.com/adamdbradley/foresight.js/wiki/Server-Resizing-Images) for more information on a few options for requesting various images sizes from a server.



## Features
* Request hi-res images according to device pixel ratio
* Detect network connection speed
* Javascript Framework independent (ie: jQuery not required)
* Image dimensions set by percents will scale to the device's available width and display pixel ratio
* Fully customizable through configuration options
* Does not make multiple requests for the same image
* Default images load without javascript enabled
* Minifies down to roughly 5K



## HTML
One of the largest problems faced with dynamically deciding image quality is that by the time javascript is capable of viewing an _img_ in the DOM, the image has already been requested from the server. And on the flip side to that, if _img_ elements are built by javascript then they probably won't be viewed by search engines and browsers without javascript enabled will not be able to view the images. To overcome both of these challenges foresight.js uses the _noscript_ element with a child _img_ element.

    <noscript data-img-src="imagefile.jpg" data-img-width="320" data-img-height="240">
        <img src="imagefile.jpg" width="320" height="240"/>
    </noscript>

Using this structure allows us to still place _img_ elements within the context of the webpage, while also allowing search engines an javascript disabled browsers to view the images.



## NoScript Element
For foresight.js to use the _noscript_ element it requires three attributes: _data-img-src_,  _data-img-width_,  _data-img-height_. These attributes act the same as their respective attributes in an _img_ element.

The child _img_ element within the _noscript_ is the fallback image incase javascript is not enabled, and for search engines to view the default image. (Note: It'd be great to just use the _noscript_ child _img_ to get the image information from instead of duplicating it in the _noscript_ attributes, except our friends IE7 and IE8 do not put _noscript_ inner text info into the DOM.)



## High-Speed Network Connection Test
Currently most devices capable of hi-res displays are mobile devices, such as new iPhones or iPads. However, since they are "mobile" in nature and their data may be relying on cell towers, even if the device has a hi-res display, users with slow connectivity probably do not want to wait a long time while images download. In these cases, foresight.js does a quick network speed test to make sure the user's device can handle hi-res images. Additionally, it stores the devices network connection speed information for 30 minutes (or any customizable to any expiration period you'd like) so it does not continually perform speed tests. You can host your own speed test file to be downloaded, or you can use the default URI available for public use found in the foresight configuration.



## Src Modification
An _img src_ needs to be modified so the browser can request the correct image according to the device and network speed. How an image's src is formatted is entirely up to the URI format of the server, but foresight.js allows the _img src_ attribute to be customized to meet various formats.

Src modification types can be assigned in the _foresight.options.srcModification_ config, or individually for each image using the _noscript data-img-src-modification_ attribute. The possible values are __replaceDimensions__ or __rebuildSrc__ and described below.

__replaceDimensions__: The current src may already have dimensions within the URI. Instead of rebuilding the src entirely, just find and replace the original dimensions with the new dimensions. 

__rebuildSrc__: Rebuild the src by parsing apart the current URI and rebuilding it using the supplied _src format_. Review the Src Format section to see how to format the image URI's.


## Src Format
The src format is only required when using the _rebuildSrc_ src modification. The src format provides foresight with how the request image should be built. Each server's image request is different and the _srcFormat_ value allows the URI to be customized. The format can either be in the _foresight.options.srcFormat_ config, or individually for each image using the _noscript data-img-src-format_ attribute. Below are the various keys which are used to rebuild the src to request the correct image from the server. Each one is not required, and you should only use the keys which help build the src request for the server.

__{protocol}__: The protocol of the request. ie: _http_ or _https_

__{host}__: The host. ie: _cdn.mysite.com_ or _www.wikipedia.com_

__{port}__: The port number, but production systems will rarely use this. ie: _80_

__{directory}__: The directory (folder) within the path. ie: _/images/_

__{file}__: Includes both the file-name and file-extension of the image. ie: _myimage.jpg_

__{filename}__: Only the file-name of the image. ie: _myimage_

__{ext}__: Only the file-extension of the image. ie: _jpg_

__{query}__: The querystring. ie: _page=1&size=10_

__{requestWidth}__: The requested width of the image to load. This value will automatically be calculated, it's just that you need to tell foresight.js where to put the request width in the src. ie: _320_

__{requestHeight}__: The requested height of the image to load. This value will automatically be calculated, it's just that you need to tell foresight.js where to put the request height in the src. ie: _480_

__{pixelRatio}__: The requested pixel ratio of the image to load. This value will automatically be calculated, its just that you need to tell foresight.js where to put this info in the src. ie: _2_

_Again, not all of these keys are required inside your src URI. Src format is entirely dependent on how the server handles image requests._



#### Src Format Examples

    Example A: Width and height in their own folder:
    Output Src: http://cdn.mysite.com/images/640/480/myimage.jpg
    SrcFormat: {protocol}://{host}{directory}{requestWidth}/{requestHeight}/{file}

    Example B: Width and height in the querystring
    Output Src: http://cdn.mysite.com/images/myimage.jpg?w=640&h=480
    SrcFormat: {protocol}://{host}{directory}{file}?w={requestWidth}&h={requestHeight}

    Example C: Width in the filename, request to the same host
    Output Src: /images/320px-myimage.jpg
    SrcFormat: {directory}{requestWidth}px-{file}

    Example D: Pixel ratio in the filename
    Output Src: http://images.example.com/home/images/hero_2x.jpg
    SrcFormat: {protocol}://{host}{directory}{file}_{pixelRatio}x.jpg

    Example E: Width in the filename, actual Wikipedia.org src format
    Output Src: http://upload.wikimedia.org/wikipedia/commons/thumb/5/57/AmericanBadger.JPG/1024px-AmericanBadger.JPG
    SrcFormat: {protocol}://{host}{directory}{requestWidth}px-{file}



## Foresight Options
Foresight comes with default settings, but using the _foresight.options_ object allows you to customize it as needed. The easiest way to configure foresight.js is to include the _foresight.options_ configuration before the foresight.js script, such as:

    <script>
        foresight = {
            options: {
                srcModification: 'rebuildSrc',
                srcFormat: '{directory}{requestWidth}px-{file}'
            }
        };
    </script>
    <script src="foresight.js"></script>

__foresight.options.srcModification__: Which type of src modification to use, either _rebuildSrc_ or _replaceDimensions_. See the Src Modification section for more info.

__foresight.options.srcFormat__: The format in which a src should be rebuilt. See the Src Format section for more info.

__foresight.options.testConn__: Either _true_ or _false_ determining if foresight should test the network connection speed or not. Default is _true_

__foresight.options.minKbpsForHighSpeedConn__: Foresight considers a network connection to be either high-speed or not. High-speed connections requests hi-res images to be downloaded. However, everyone's interpretation of what is considered _high-speed_ should be a variable. By default, any connection that can download an image at a minimum of 800Kbps is considered high-speed. The value should be a number representing Kbps. Default value is _800_

__foresight.options.speedTestUri__: You can determine the URI for the speed test. By default it will use a foresight hosted image, but you can always choose your own URI for the test image. Default value is _//foresightjs.appspot.com/speed-test/100K_

__foresight.options.speedTestKB__: Foresight needs to know the filesize speed test file is so it can calculate the approximate network connection speed. By default it is downloading a 100KB file. The value should be a number representing KiloBytes. Default value is _100_

__foresight.options.speedTestExpireMinutes__: Speed tests do not need to be performed on every page. Instead you can set how often a speed test should be completed, and in between test you can rely on past test information. The value should be a number representing minutes. Default value is _30_

__foresight.options.maxImgWidth__: A max pixel size can be set on images. This is in reference to browser pixels. Default value is _1200_

__foresight.options.maxImgHeight__: A max pixel size can be set on images. This is in reference to browser pixels. Default value is _1200_

__foresight.options.maxImgRequestWidth__: A max pixel size can be set on how large of images can be requested from the server. Default value is _2048_

__foresight.options.maxImgRequestHeight__: A max pixel size can be set on how large of images can be requested from the server. Default value is _2048_

__foresight.options.forcedPixelRatio__: You can force the pixel ratio and override the device pixel ratio value. Default value undefined and falls back to the device pixel ratio.

Additionally, the foresight global options can be overwritten by each individual _noscript_ element if need be.



## NoScript data-img Attributes
__data-img-src__: _(Required)_ The src attribute of the image, which is the location image on the server.

__data-img-width__: _(Required)_ The pixel width according to the browser. Any adjusting to the device pixel ratio will be taken care of and request image automatically adjusted. Both _data-img-width_ and _data-img-height_ are required so we can always proportionally scale the image.

__data-img-height__: _(Required)_ The pixel height according to the browser. Any adjusting to the device pixel ratio will be taken care of and request image automatically adjusted. Both _data-img-width_ and _data-img-height_ are required so we can always proportionally scale the image.

__data-img-src-modification__: _(Optional)_ Which type of src modification to use, either _rebuildSrc_ or _replaceDimensions_. See the Src Modification section for more info.

__data-img-src-format__: _(Optional)_ The format in which a src should be rebuilt. See the Src Format section for more info.

__data-img-id__: _(Optional)_ The _img id_ attribute. If one is not assigned it will be assigned an _id_ attribute starting with 'fsImg' then followed by a random number.

__data-img-class__: _(Optional)_ The _img class_ attribute which can hold CSS class names.

__data-img-max-width__: _(Optional)_ Maximum browser pixel width this image should take. If this value is greater than the width it will scale the image proportionally.

__data-img-max-height__: _(Optional)_ Maximum browser pixel height this image should take. If this value is greater than the height it will scale the image proportionally.

__data-img-pixel-ratio__: _(Optional)_ By default an image's pixel ratio is figured out using the devices pixel ratio. You can however manually assign an image's pixel ratio which will override the default.



## Foresight Properties

After foresight executes there are a handful of properties viewable.

__foresight.images__: An array containing each of the foresight _img_ elements.

__foresight.devicePixelRatio__: The device's pixel ratio used by foresight. If the browser does not know the pixel ratio, such as old browsers, it defaults to 1.

__foresight.connTestMethod__: The connection test method provides info on how the device received its speed-test information. 

* _network_: The speed test information came directly from a network test.
* _networkSlow_: A 100KB file should be downloaded within 1 second on a 800Kbps connection. If the speed test takes longer than 1 second than we already know its not a high-speed connection. Instead of waiting for the response, just continue and set that this network connection is not a high-speed connection.
* _networkError_: When a speed-test network error occurs, such as a 404 response, the connTestMethod will equal networkError and will not be considered a high-speed connection.
* _localStorage_: A speed-test does not need to be executed on every webpage. The browser's localStorage function is used to remember the last speed test information. When the last speed-test falls outside of the _foresight.options.speedTestExpireMinutes_ option it execute a new speed-test again.
* _skip_: If the device pixel ratio equals 1 then the display cannot view hi-res images. Since high-resolution doesn't apply to this device, foresight doesn't bother testing the network connection.

__foresight.connKbps__: Number representing the estimated Kbps following a network connection speed-test. This value can also come from localStorage if the last test was within the _foresight.options.speedTestExpireMinutes_ option.

__foresight.isHighSpeedConn__: Boolean used to tell foresight if this device's connection is considered a high-speed connection or not.



## Foresight Events

__foresight.oncomplete__: Executed after foresight rebuilt each of the image src's and inserted them into the DOM.



## Foresight Debugging
Instead of including debugging code directly in the foresight.js, an additional javascript file has been included to help developers debug. By using the _foresight.oncomplete_ event and existing _foresight_ properties, the _foresight-debug.js_ file prints out relivate information to help debug. This is particularly useful for mobile devices since it is more difficult to view source code. Below is sample code to include the _foresight-debugger.js_ file and calling it when foresight completes:

    <script src="foresight-debugger.js"></script>
    <script>
        foresight = {
            options: {
                srcModification: 'rebuildSrc',
                srcFormat: '{directory}{requestWidth}px-{file}'
            },
            oncomplete: foresight_debugger
        };
    </script>
    <script src="foresight.js"></script>