---
layout: post
title: "Upload files to S3 with Laravel Vapor and Dropzone.js"
published: true
excerpt_separator: <!--more-->
---

For a startup project in Laravel we'll be launching soon I need to upload multiple files. With our AWS hosted stack deployed via Laravel Vapor the logical choice for storage is S3. To keep the load off our lambda "backend" I want it to upload to S3 straight from the browser using pre-signed URLs. For the frontend functionality I chose [Dropzone](https://www.dropzonejs.com/) because I have experience with it from previous projects. (Note to self: Check out [Uppy](https://uppy.io/))

I'm assuming you have an AWS account and permission setup with your access key and bucket configured in your .env and vapor.

At the end of this post we will have a drag-and-drop multi-file uploader directly to S3 using Vapor functionality.

<!--more-->

## Setting up Dropzone with Vapor

Let's start with our front-end setup. Install Dropzone and the Vapor js package:
```bash
npm i dropzone laravel-vapor
```

Add a form element to your blade view. We don't give it a dropzone class as we want to initialize it ourselves so we can target it later. Our action will be the record we will want to associate the uploads with. In this case a product will have many photos. So for every uploaded file we will store a photo associated to the product.
```html
<form action="{{ route('products.photos.store', $product->id) }}" id="dropzone-form"></form>
```

On the JS side of things we initialize the Dropzone:
```js
const dropzoneForm = new Dropzone("#dropzone-form");
```

And we override the `uploadFiles` function:
```js
Dropzone.prototype.uploadFiles = async files => files.forEach(uploadFile);
```

The uploadFiles method gets called with a number of items based on the `parallelUploads` setting and how many files are processing at that time. We call uploadFile for each:

```js
async function uploadFile(file) {
  const s3response = await Vapor.store(file, {
    progress: progress => {
      const percentage = Math.round(progress * 100 * 0.9);
      dropzoneForm.emit("uploadprogress", file, percentage);
    }
  });

  ...
}	
```

The `Vapor.store` call does most of the work for us. It first gets a presigned url from our backend and then uploads the file to the given url. The `progress` callback ties nicely into Dropzone's `uploadprogress` event. Notice that the maximum progress percentage is 90% here. That's because we are not done yet. Continuing `uploadFile`:


```js
async function uploadFile(file) {
  ...
  
  const itemResponse = await axios.post(dropzoneForm.element.action, {
    filename: file.name,
    filetype: file.type,
    tmp: s3response.key
  });
  
  ...
}
```

`s3response` contains a key pointing us to the temporary file in the S3 bucket. We still need to report back to our backend so we can do something useful with the upload. We use axios to make the ajax call here. This is what the call in `itemResponse` takes care of. We report back to our own backend and set the progress to 100%. Let's finish up our `uploadFile` function:

```js
async function uploadFile(file) {
  ...

  file.status = Dropzone.SUCCESS;

  dropzoneForm.emit("uploadprogress", file, 100);
  dropzoneForm.emit("complete", file);
  dropzoneForm.processQueue();
}
```

We finish up our JS by telling Dropzone we're done with the file. The last call is to kick off `processQueue` again triggering dropzone to pick up some more pending uploads.

To summarize, we now have the following steps in place:
- Show a dropzone
- Generate a pre-signed URL
- Upload directly to our S3 bucket
- Receive the key where our file is located
- Call our backend with the 

## Tying things up on the backend

Our form action points to the `products.photos.store` route, so let's implement it in our `PhotosController`:

```php
public function store(Request $request, $product_id)
{
    photo = Photo::create([
        'product_id' => $product_id,
        'filename' => $request->input('filename'),
        'filetype' => $request->input('filetype'),
    ]);

    Storage::copy(
        '/' . $request->input('tmp'),
        "/products/$product_id/photos/$photo->id"
    );
}
```

With `Storage::copy` the file gets moved to a more useful path related to the `$photo` record we just created.

If we want to show our photo somewhere we can get a url from S3:

```php
Storage::temporaryUrl(
    "/products/$product_id/photos/$photo->id",
    now()->addMinutes(100)
)
```

## Wrapping up

That's all you need to get your file uploads going. Your store action would also be a great place to dispatch a job to generate thumbnails for your newly uploaded photos. But we'll leave that exercise for some other time.
