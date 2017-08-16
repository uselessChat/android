The Android Support Library [PrintHelper](https://developer.android.com/reference/android/support/v4/print/PrintHelper.html) class provides a simple way to print of images. The class has a single layout option, setScaleMode(), which allows you to print with one of two options:

* SCALE_MODE_FIT
  * This option sizes the image so that the whole image is shown within the printable area of the page.
* SCALE_MODE_FILL
  * This option scales the image so that it fills the entire printable area of the page. Choosing this setting means that some portion of the top and bottom, or left and right edges of the image is not printed. This option is the default value if you do not set a scale mode.

# [Printing Content](https://developer.android.com/training/printing/index.html)
* Printing a Photo
* Printing an HTML Document
* Printing a Custom Document
