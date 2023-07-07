# Get-URI-or-Bitmap-From-View


#Bitmap from View
```
    fun getBitmapFromView(view: View): Bitmap {
            //Define a bitmap with the same size as the view
            val returnedBitmap =
                Bitmap.createBitmap(view.width, view.height, Bitmap.Config.ARGB_8888)
            //Bind a canvas to it
            val canvas = Canvas(returnedBitmap)
            //Get the view's background
            val bgDrawable = view.background
            if (bgDrawable != null) //has background drawable, then draw it on the canvas
                bgDrawable.draw(canvas) else  //does not have background drawable, then draw white background on the canvas
                canvas.drawColor(Color.WHITE)
            // draw the view on the canvas
            view.draw(canvas)
            //return the bitmap
            return returnedBitmap
        }
```

#URI from Bitmap

```
    private fun getImageUri(image: Bitmap): Uri? {
//        val cachePath = File( Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES), "") //for public directory need write permission
        try {
            val cachePath = File(cacheDir, "images")   //save it to cashe Directory
            cachePath.mkdirs() // don't forget to make the directory
            val stream =
                FileOutputStream("$cachePath/image.png") // overwrites this image every time
            image.compress(Bitmap.CompressFormat.PNG, 100, stream)
            stream.close()

            val newFile = File(cachePath, "image.png")
            val contentUri =
                FileProvider.getUriForFile(this, "com.appdomain.fileprovider", newFile)
            return contentUri
        } catch (e: IOException) {
            e.printStackTrace()
        }
        return null
    }

```
#Add FileProvider in Manifest File

```
 <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="com.appdomain.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths"
                tools:replace="android:resource" />
        </provider>
```

#provider_paths.xml

```
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <cache-path
        name="shared_images"
        path="images/" />
</paths>
```

#Share Image 

```
 val uri = getImageUri(getBitmapFromView(binding.view))
            uri?.let {
                val shareIntent = Intent()
                shareIntent.action = Intent.ACTION_SEND
                shareIntent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION) // temp permission for receiving app to read this file
                shareIntent.setDataAndType(uri, contentResolver.getType(uri))
                shareIntent.putExtra(Intent.EXTRA_STREAM, uri)
                startActivity(Intent.createChooser(shareIntent, "Share with"))
            }
```

