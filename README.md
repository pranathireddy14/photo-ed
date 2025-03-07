# photo-ed
Fast and advanced TIFF decoder
Download and include the `UTIF.js` file in your code. If you're in NodeJS or otherwise using NPM, run:
```sh
npm install utif
```
Download and include the `UTIF.js` file in your code. 

#### `UTIF.decode(buffer)`
* `buffer`: ArrayBuffer containing TIFF or EXIF data
* returns an array of "IFDs" (image file directories). Each IFD is an object, keys are "tXYZ" (XYZ is a TIFF tag number), values are values of these tags. You can get the the dimension (and other properties, "metadata") of the image without decompressing pixel data.

#### `UTIF.decodeImage(buffer, ifd)`
* `buffer`: ArrayBuffer containing TIFF or EXIF data
* `ifd`: the element of the output of UTIF.decode()
* If there is an image inside the IFD, it is decoded and three new properties are added to the IFD:
* * `width`: the width of the image
* * `height`: the height of the image
* * `data`: decompressed pixel data of the image

TIFF files may have various number of channels and various color depth. The interpretation of `data` depends on many tags (see the [TIFF 6 specification](http://www.npes.org/pdf/TIFF-v6.pdf)). The following function converts any TIFF image into a 8-bit RGBA image.

#### `UTIF.toRGBA8(ifd)`
* `ifd`: image file directory (element of "ifds" returned by UTIF.decode(), processed by UTIF.decodeImage())
* returns Uint8Array of the image in RGBA format, 8 bits per channel (ready to use in context2d.putImageData() etc.)

### Example

```javascript
function imgLoaded(e) {
  var ifds = UTIF.decode(e.target.response);
  UTIF.decodeImage(e.target.response, ifds[0])
  var rgba  = UTIF.toRGBA8(ifds[0]);  // Uint8Array with RGBA pixels
  console.log(ifds[0].width, ifds[0].height, ifds[0]);
}

var xhr = new XMLHttpRequest();
xhr.open("GET", "my_image.tif");
xhr.responseType = "arraybuffer";
xhr.onload = imgLoaded;   xhr.send();
```
## Use TIFF images in HTML

If you are not a programmer, you can use TIFF images directly inside the `<img>` element of HTML. Then, it is enough to call `UTIF.replaceIMG()` once at some point.

#### `UTIF.replaceIMG()`
```html
<body onload="UTIF.replaceIMG()">
...
<img src="image.tif" />  <img src="dog.tif" /> ...
```
And UTIF.js will do the rest. Internally, the "src" attribute of the image will be replaced with a new URI of the image (base64-encoded PNG). Note, that you can also insert DNG, CR2, NEF and other raw images into HTML this way.
## Encoding TIFF images

You should not save images into TIFF format in the 21st century. Save them as PNG instead (e.g. using [UPNG.js](https://github.com/photopea/UPNG.js)). If you still want to use TIFF format for some reason, here it is.

#### `UTIF.encodeImage(rgba, w, h, metadata)`
* `rgba`: ArrayBuffer containing RGBA pixel data
* `w`: image width
* `h`: image height
* `metadata` [optional]: IFD object (see below)
* returns ArrayBuffer of the binary TIFF file. No compression right now.

#### `UTIF.encode(ifds)`
* `ifds`: array of IFDs (image file directories). An IFD is a JS object with properties "tXYZ" (where XYZ are TIFF tags)
* returns ArrayBuffer of binary data. You can use it to encode EXIF data.

## Dependencies
TIFF format sometimes uses Inflate algorithm for compression (but it is quite rare). Right now, UTIF.js calls [Pako.js](https://github.com/nodeca/pako) for the Inflate method.
