# Seven cattle Node.js SDK

The SDK suitable for Node.js 0.4.7 and above, built on seven cattle cloud storage official API. If your server is a web-based program written in Node.js using this SDK, allows you to very easy way to securely store data on seven cattle cloud storage. So that the end user of your application for high-speed uploads and downloads, but also makes your server more lightweight.
 
## Installation  

You can install from npm

```shell
npm install node-qiniu
```

You can also download and install from Github

```shell
$ git clone git://github.com/qiniu/node-qiniu
$ cd node-qiniu
$ npm install .
```

## Test

Seven cattle Node.js SDK Use Mocha unit testing.

```shell
$ npm install -g mocha
$ make test
```

## Use

### Configuration `qiniu.config()`

Set global parameters, including the need for AccessKey 和 SecretKey，You can also set up other such CallbackURL And other parameters, will be postponed until all the space.

```js
qiniu.config({
  access_key: '------',
  secret_key: '------'
});
```

### Bucket

Obtain spatial objects and operations.

```js
var imagesBucket = qiniu.bucket('qiniu-sdk-test');
// Such operations can be
// var imagesBucket = new qiniu.Bucket('qiniu-sdk-test');
```

#### Upload file

**1. `Bucket.putFile()`**

Upload a file, the parameters for the Key, file address to be uploaded (can be an absolute address, or a relative address), and the third is an optional parameter options, which is the second upload using the special settings PutToken fourth is an optional parameter callback (callback), if passed Promise object callback function returned by putFile respond.

```js
// Ordinary upload
imagesBucket.putFile('exampleKey', __dirname + '/assets/example.jpg', function(err, reply) {
  if (err) {
    return console.error(err);
  }

  console.dir(reply);
});
// Special parameters
imagesBucket.putFile('exampleKey_1', __dirname + '/assets/example.jpg', {
  // Upload used for the second set of Token, here set uploaded by identity
  endUser: 'foobar'
}, function(err, reply) {
  if (err) {
    return console.error(err);
  }

  console.dir(reply);
});

// Promise object seven cattle Node.js SDK provides follow Promise / A (+) standard, use. Then responds method

imagesBucket.putFile('exampleKey_2', __dirname + '/assets/example.jpg')
  .then(
    function(reply) {
      // Upload successful
      console.dir(reply);
    },
    function(err) {
      // Upload Failed
      console.error(err);
    }
  );
```

**2. `Bucket.createPutStream()`**

Seven cattle Node.js SDK-based flow (Stream) mode of operation for developers familiar with Node.js streaming operations to provide a convenient, high-performance API.

```js
var puttingStream = imagesBucket.createPutStream('exampleKey_3');
var readingStream = fs.createReadStream(__dirname + '/assets/example.jpg');

readingStream.pipe(puttingStream)
  .on('error', function(err) {
    console.error(err);
  })
  .on('end', function(reply) {
    console.dir(reply);
  });
```

#### Download file

`Bucket.getFile()`And`Bucket.createGetStream()`

Get files and upload files equally simple, and Node.js natively file system (File System) API that there are similarities.

```js
imagesBucket.getFile('exampleKey', function(err, data) {
  if (err) {
    return console.error(err);
  }

  // data To Buffer object that contains file data
});
```

Similarly, access to files can also be performed using streaming operations
```js
var gettingStream = imagesBucket.createGetStream('exampleKey');
var writingStream = fs.createWriteStream(__dirname + '/assets/example_tmp.jpg');

gettingStream.pipe(writingStream)
  .on('error', function(err) {
    console.error(err);
  })
  .on('finish', function() {
    // File data has been written to the local file system
  });
```

### `Image` Picture Operation

Seven cattle Node.js SDK provides `Image` class for image resources to operate.

Use Bucket.image() Method to obtain an image of the object

```js
var image = imagesBucket.image('exampleKey');
```

#### `Image.imageInfo()`

Image.imageInfo The method can be used to obtain images of picture information resources.
See details：[http://docs.qiniu.com/api/v6/image-process.html#imageInfo](http://docs.qiniu.com/api/v6/image-process.html#imageInfo)

```js
image.imageInfo(function(err, info) {
  if (err) {
    return console.error(err);
  }

  console.dir(info);
});
```

#### `Image.exif()`

Image.imageView The method used to generate the specified size of thumbnails.
See details：[http://docs.qiniu.com/api/v6/image-process.html#imageView](http://docs.qiniu.com/api/v6/image-process.html#imageView)

```js
image.exif(function(err, exif) {
  if (err) {
    return console.error(err);
  }

  console.dir(exif);
});
```

#### `Image.imageView()`

Image.imageView The method used to generate the specified size of thumbnails.
See details：[http://docs.qiniu.com/api/v6/image-process.html#imageView](http://docs.qiniu.com/api/v6/image-process.html#imageView)

```js
image.imageView({
  mode    : 2,
  width   : 180,
  height  : 180,
  quality : 85,
  format  : 'jpg'
}, function(err, imageData) {
  if (err) {
    return console.error(err);
  }

  // imageData Processing the image data after the
});
```

Among them, the method of image objects with pictures of all the data can be used to return the current operation. The callback is not passed in the second parameter, and call a method in parentheses after the call `. Stream ()` method will return an output IO will continue to stream data.

```js
var imageViewStream = image.imageView({
  mode    : 2,
  width   : 180,
  height  : 180,
  quality : 85,
  format  : 'jpg'
}).stream();
var writingStream = fs.createWriteStream(__dirname + '/assets/example_thumbnail.jpg');

imageViewStream.pipe(writingStream)
  .on('error', function(err) {
    console.error(err);
  })
  .on('finish', function() {
    // Thumbnails have written to the local file system
  });
```

Such as these:
```js
image.imageMogr(...).stream();
image.watermark(...).stream();
image.alias(...).stream();
```

#### `Image.imageMogr()`

Image.imageMogr The method used to call the advanced image processing interface and returns the processed image data.
See details:  [http://docs.qiniu.com/api/v6/image-process.html#imageMogr](http://docs.qiniu.com/api/v6/image-process.html#imageMogr)

```js
image.imageMogr({
  thumbnail : '300x500',
  gravity   : 'NorthWest',
  crop      : '!300x400a10a10',
  quality   : 85,
  rotate    : 90,
  format    : 'jpg'
}, function(err, imageData) {
  if (err) {
    return console.error(err);
  }

  // Use imageData operation
});
```

#### `Image.watermark()`

Image.watermark The method used to generate images with a watermark, image watermark and text watermark API supports watermark image modes.
See details：http://docs.qiniu.com/api/v6/image-process.html#watermark

```js
image.watermark({
  mode: 1,
  image: 'http://www.b1.qiniudn.com/images/logo-2.png',
  dissolve: 70,
  gravity: 'SouthEast',
  dx: 20,
  dy: 20
}, function(err, imageData) {
  if (err) {
    return console.error(err);
  }

  // Use imageData operation
});
```

#### `Image.alias()`

Image.alias The method used to return data format established data processing, the use of this method requires [seven cattle Developer Platform] (https://portal.qiniu.com) in the settings operation.
Where, `Image.alias ()` method `Asset` class inherits key being used.

```js
image.alias('testalias', function(err, imageData) {
  if (err) {
    return console.error(err);
  }

  // Use imageData operation
});
```

### `Asset` Resource Operation

Seven cattle Node.js SDK provides a `Asset` class resources for an operation. 

Get key corresponding to the resource object
```js
var picture = imagesBucket.key('exampleKey');
```

#### `Asset.url()`

`Asset.url()` Method can obtain the resource for accessing the URL

```js
var picUrl = picture.url();
```

#### `Asset.entryUrl()`

`Asset.entryUrl()`The method can obtain the required resources for the API call EncodedEntryURL. 
But Node.js SDK, most developers do not need their own API to use.:)

```js
var encodedPicUrl = picture.entryUrl();
```

#### `Asset.stat()`

Asset.stat The method can obtain the resources, such as file size, MIME data types stat.

```js
picture.stat(function(err, stat) {
  if (err) {
    return console.error(err);
  }

  console.dir(stat);
  /**
   * {
   *  hash      : <FileEtag>, // stringHash value types, file 
   *  fsize     : <FileSize>, // int type, file size (unit: byte) 
   *  mimeType  : <MimeType>, // string type of media file types, such as "image/gif" 
   *  putTime   : <PutTime>   // int64 type, file upload to seven cattle cloud time (Unix timestamp)
   * }
   */
});
```

#### `Asset.move()`

`Asset.move()`A method for moving the resource to the specified location. 
The first parameter can be derived from other resource object Bucket belongs.

```js
picture.move(imagesBucket.key('exampleKey_4'), function(err) {
  if (err) {
    return console.error(err);
  }

  // Here there is no return value, if there is no error, the operation was successful, the same as the following method
});
```

#### `Asset.copy()`

`Asset.copy()`The method used to create a copy of this resource, and saved to a specified resource location.

```js
imagesBucket.key('exampleKey_4').copy(picture, function(err) {
  if (err) {
    return console.error(err);
  }
});
```

#### `Asset.remove()`

`Asset.remove()`The method used to remove the current resource.

```js
imagesBucket.key('exampleKey_4').remove(function(err) {
  if (err) {
    return console.error(err);
  }
});
```

#### `Asset.token()`

`Asset.token()`The method used to generate the current resource download the certificate.

```js
var getToken = imagesBucket.key('exampleKey').token();

console.dir(getToken);
/*=>
  {
    url: '<Full download url>',
    token: '<Get token>',
    requestUrl: 'URL without token'
  }
 */
  }
```

#### `Asset.download()`

`Asset.download()`The method used to generate the current download link resources. (Does not include downloads certificate)

```js
var url = imagesBucket.key('exampleKey').download();
```

### `Batch` Resources batch operation 

In support resources operations on a single file at the same time, seven cattle cloud storage also supports multiple files in bulk to view, delete, copy and move operations.
See details：http://docs.qiniu.com/api/v6/rs.html#batch

The controller generates a batch operation
```js
var batch = qiniu.batch();
```

`Batch` Most parameters and resource object `Asset` similar support to view, move, copy and delete operations.

```js
batch
  // Get file information
  .stat(imagesBucket.key('exampleKey'))
  // Mobile Resources
  .move(imagesBucket.key('exampleKey'), imagesBucket.key('exampleKey_5'))
  // Copy Resources
  .copy(imagesBucket.key('exampleKey_5'), imagesBucket.key('exampleKey'))
  // Delete Resource
  .remove(imagesBucket.key('exampleKey_5'))
  // Perform operations 
  // Before and after each operation are in accordance with the order of execution
  .exec(function(err, results) {
    if (err) {
      return console.error(err);
    }

    console.dir(results);
    // results The result of each operation
  });
```

### `Fop` Pipeline Operation

Seven cattle cloud storage provides a very useful resource processing API, can be used to operate a variety of processing resources. 

Example: an original thumbnail, then marked another picture as a watermark on the thumbnail 

() `Method to create Fop pipeline operator, and operates using` Asset.fop.

```js
var image = imagesBucket.key('exampleKey');
// Image.fop Methods inherited Asset Class

image.fop()
  // The picture thumbnail
  .imageView({
    mode   : 2,
    height : 200
  })
  // For the watermarked image
  .watermark({
    mode  : 1,
    image : 'http://www.b1.qiniudn.com/images/logo-2.png'
  })
  // Perform operations
  .exec(function(err, imageData) {
    if (err) {
      return console.error(err);
    }

    // imageData As watermarked thumbnail data
  });
```

#### `Fop.token()`

The method used to generate the current Fop download the certificate.

```js
var image = imagesBucket.key('exampleKey');
// Image.fop Methods inherited Asset Class

var getToken = image.fop()
  // The picture thumbnail
  .imageView({
    mode   : 2,
    height : 200
  })
  // For the watermarked image
  .watermark({
    mode  : 1,
    image : 'http://www.b1.qiniudn.com/images/logo-2.png'
  })
  .token();

console.log(getToken.url);
```

## Module structure 
! [Module structure] (http://ww2.sinaimg.cn/large/7287333fgw1e8263cvxeaj20mr0glgmp.jpg)

## License 

    (The MIT License)
    
    Copyright (c) 2010-2013 Will Wen Gunn <willwengunn@gmail.com> and other contributors
    
    Permission is hereby granted, free of charge, to any person obtaining
    a copy of this software and associated documentation files (the
    'Software'), to deal in the Software without restriction, including
    without limitation the rights to use, copy, modify, merge, publish,
    distribute, sublicense, and/or sell copies of the Software, and to
    permit persons to whom the Software is furnished to do so, subject to
    the following conditions:
    
    The above copyright notice and this permission notice shall be
    included in all copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
    EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
    MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
    IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
    CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
    TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
    SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/iwillwen/node-qiniu/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

