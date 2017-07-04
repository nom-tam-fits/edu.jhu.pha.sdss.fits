# IVOA FITS Package

##About the IVOA FITS Package
The IVOA FITS package is a set of classes which can be used by any Java application to load and view FITS images. It is currently used to provide image viewing functionality in VO Enabled Mirage. FITS file data loading uses Tom McGlynn's nom.tam.fits package.

##Notes on the latest version
For version 0.3, some changes were made to accomodate a change in the underlying java imageio implementation. Specifically, it was necessary to change the code handling gzipped images. Consequently, the constructor for ImageInputStreamInputStream now throws IOException if it runs into problems, and the public static method gunzipIfNecessary(InputStream in) has been moved from the FITSReader class into the ImageInputStreamInputStream class and has been made protected. Application developers using these classes and/or methods should update affected code accordingly. Everyone else should be able to disregard this notice.

##Using the IVOA FITS Package to view FITS images
The IVOA FITS package is divided into two main subpackages, edu.jhu.pha.sdss.fits and edu.jhu.pha.sdss.fits.imageio. The preferred way to load a FITS image for viewing using the IVOA FITS Package is to register the included Java ImageIO SPI implementation using the following code:

  static
  {
    IIORegistry.getDefaultInstance().
      registerServiceProvider(new FITSReaderSpi());
  }

This code should be placed in some class that is guaranteed to be loaded before any FITS images are loaded. A FITS image can then be loaded using the normal Java ImageIO interface, as follows:

    BufferedImage image = ImageIO.read(in);

This will load any type of image supported natively by Java, as well as FITS images, provided that the SPI implementation has been registered as described above. To determine whether the image loaded is indeed a FITS image and to get a reference of the proper type, do something like the following:

  if(image instanceof FITSImage)
  {
    FITSImage fitsImage = (FITSImage)image;
  }

Here it should be clear that FITSImage is a subclass of BufferedImage, and can be treated as such. But to get any real use out of the image, it should be cast to FITSImage so that the extended functionality can be used. The SampleModel of the image will most likely (and should) be a PixelInterleavedSampleModel, and the ColorModel will be in the CS_sRGB color space with data type USHORT. These images can be displayed in any Java Component as is, but you will probably want to add your own colormap functionality to reduce them to byte range. As the resulting data is in the USHORT range, obviously many images must internally be scaled to that range, thus there are several common scaling algorithms offered as well as one relatively new one. These are explained in more detail in the javadocs for FITSImage. With each FITSImage comes associated a Histogram which can be used for various image intensity scaling purposes. The number of bins in the Histogram is 65535.

##Running an example
Included in the IVOA FITS package jar file is an example app which loads a FITS image and displays a slideshow of the image in various scalings with 5 seconds between slides. To run the example, type

java -Xmx128M -jar ivoafits-0.3.jar filename.fits

Here "filename.fits" of course should be replaced by the name of some actual image file. There is optional debugging output which prints the execution time of each rescaling. This option may be turned on by including the following argument to the above java command:

-Ddebug=on
