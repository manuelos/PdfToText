# INTRODUCTION #

The PdfToText class has been designed to extract textual contents from a PDF file.

It's pretty simple to use :

	include ( 'PdfToText.phpclass' ) ;

	$pdf 	=  new PdfToText ( 'sample.pdf' ) ;
	echo $pdf -> Text ; 		// or you could also write : echo ( string ) $pdf ;

The same PdfToText object can be reused to process additional files :

	$pdf -> Load ( 'sample2.pdf' ) ;
	echo $pdf -> Text ;

Additionally, the **PdfToText** class provides support methods for getting the page number of any text in the underlying PDF file.

Look at the class' blog for an overview on the underlying mechanics that are involved into extracting text contents from pdf files.

Examples are also provided in the **examples/** directory. Please have a look at the [examples/README.md](examples/README.md "README.md") file for a brief explanation on their structure.

**IMPORTANT** : the **PdfToText** class generates UTF8-encoded text. If your default character set is not UTF-8, you may need to add the following **meta** tag in the &lt;head&gt; part of your HTML page :

	<meta charset="utf-8" />


# FEATURES #

Text rendering in a PDF file is made using an obscure language which provides multiple ways to position the same text at the same location on a page. You could say for example :

	. Goto coordinates (x,y)
	. Render text ( "Mail : someone@somewhere.com" )

Or :

	. Goto next line
	. Goto (x1,y)
	. Render text ( "Mail" )
	. Goto (x2, y) 
	. Render text ( ":" )
	. Goto (x3, y)
	. Render text ( "someone@somewhere.com" )

(note that I'm using a pseudo-language here).
Both pieces of code would probably display the same text at the same position, by using rather different ways.

This is why the **PdfToText** class tracks the following information from the drawing-instruction stream to provide more accurate text rendering (even if the output is only pure text) :

- The currently selected font is tracked. This is important because :

	- Each font in a PDF file can have its own character map. This means in this case that characters to be drawn using the Adobe language do not specify actual character codes, but an index into the font's character map.
	- The current font size is memorized ; this helps to evaluate what is the current y-coordinate when relative positioning instructions are used (such as "goto next line"). Although approximative, this works in a great majority of cases
	
- If multiple strings are rendered using identical y-coordinate, they will be grouped onto the same line. Note that they must appear sequentially in the instruction flow for this trick to work
- Sub/super-scripted text is usually written at a slightly different y-coordinate than the line it appears in. Such a situation is detected, and the sub/super-scripted text will correctly appear onto the same line

These symptoms will not appear if the *PDFOPT\_BASIC\_LAYOUT* option is specified.

# KNOWN ISSUES #

Here is a list about known issues for the **PdfToText** class ;  I'm working on solving them, so I hope this whole paragraph will soon completely disappear !

- Unwanted line breaks may occur within text lines. This is due to the fact that the pdf file contains drawing instructions that use relative positioning. This is especially true for file created with generators such as **PdfCreator**. However, some provisions have been made to try to track put text with roughly the same y-coordinates onto the same line. This limitation does not apply if the *PDFOPT\_BASIC\_LAYOUT* option is specified.
- Encrypted PDF files are not supported 

# A NOTE FOR WINDOWS USERS #

An Apache server on Linux platforms allocates a default stack size of 8Mb for its threads. This value is set to 1Mb on Windows platforms.

However, some regular expressions used by the **PdfToText** class may cause the PHP PCRE extension to require a little bit more than 1Mb of stack space when processing certain PDF files.

Such a situation will cause your Windows Apache server to crash and your browser to display a message such as : *Connection reset*. This behavior affect several products such as EasyPHP, XAMPP or Wamp.

To solve this issue, you will have to enable the **mpm** module in your *httpd.conf* file and define a new stack size, as in the following example, given for a Wamp server :

		Include conf/extra/httpd-mpm.conf
		ThreadStackSize 8388608
  


# TESTING #

I have tested this class against dozens of documents from various origins, and tested the output generated from each sample document by the *PdfCreator*, *PrimoPdf* and *PDF Pro* tools.

I also compared the output of the **PdfToText** class with that of *Acrobat Reader*, when you choose the *Save as...Text* option. In many situations, the class performs better in positioning the final text than *Acrobat Reader* does.  

However, all of that will not guarantee that it will work in every situation ; so, if you find something weird or not functioning properly using the **PdfToText** class, feel free to contact me on this class' blog, and/or send me a sample PDF file at the following email address :

		christian.vigh@wuthering-bytes.com

# OTHER LINKS #

This class can also be found here :

[http://www.phpclasses.org/package/9732-PHP-Extract-text-contents-from-PDF-files.html](http://www.phpclasses.org/package/9732-PHP-Extract-text-contents-from-PDF-files.html "http://www.phpclasses.org/package/9732-PHP-Extract-text-contents-from-PDF-files.html")
  
and here, where you will also find a FAQ section and be able to upload your PDF file samples for live testing :

[http://www.pdftotext.eu](http://www.pdftotext.eu "http://www.pdftotext.eu")

and also here :

[https://github.com/christian-vigh-phpclasses/PdfToText](https://github.com/christian-vigh-phpclasses/PdfToText "https://github.com/christian-vigh-phpclasses/PdfToText")

# REFERENCE #

## METHODS ##

### Constructor ###

	$pdf 	=  new PdfToText ( $filename = null, $options = self::PDFOPT_NONE, $user\_password = false, $owner\_password = false ) ;

Instantiates a **PdfToText** object. If a filename has been specified, its text contents will be loaded and made available in the *Text* property (otherwise, you will have to call the *Load()* method for that).

See the *Options* property for a description of the *$options* parameter.

The *$user\_password* and *$owner\_password* parameters specify the user/owner password to be used for decrypting a password-protected file (note that this class is not a password cracker !).

In the current version, decryption of password-protected files is not yet supported.

### Load ( $filename, $user\_password = false, $owner\_password = false ) ###

Loads the text contents of the specified filename.

The *$user\_password* and *$owner\_password* parameters specify the user/owner password to be used for decrypting a password-protected file (note that this class is not a password cracker !).

In the current version, decryption of password-protected files is not yet supported.

The method returns the decoded text contents, which are also available through the *Text* property.

### LoadFromString ( $contents, $user\_password = false, $owner\_password = false ) ###

Loads the text contents of the specified PDF contents.

The *$user\_password* and *$owner\_password* parameters specify the user/owner password to be used for decrypting a password-protected file (note that this class is not a password cracker !).

In the current version, decryption of password-protected files is not yet supported.

The method returns the decoded text contents, which are also available through the *Text* property.


### GetPageFromOffset ( $offset ) ###

Given a byte offset in the Text property, returns its page number in the pdf document.

Page numbers start from 1.


### text\_strpos, text\_stripos ###

    $result		=  $pdf -> text_strpos  ( $search, $start = 0 ) ;
    $result		=  $pdf -> text_stripos ( $search, $start = 0 ) ;

These methods behave as the strpos/stripos PHP functions, except that :

- They operate on the text contents of the pdf file (Text property)
- They return an array containing the page number and text offset. $result [0] will be set to the page number of the searched text, and $result [1] to its offset in the Text property

Parameters are the following :

- *$search* (string) : String to be searched.
- *$start* (integer) : Start offset in the pdf text contents.

The method returns an array of two values containing the page number and text offset if the searched string has been found, or *false* otherwise.

### document\_strpos, document\_stripos ###

    $result		=  $pdf -> document_strpos  ( $search, $group_by_page = false ) ;
    $result		=  $pdf -> document_stripos ( $search, $group_by_page = false ) ;

Searches for ALL occurrences of a given string in the pdf document. The value of the $group_by_page parameter determines how the results are returned :

- When true, the returned value will be an associative array whose keys will be page numbers and values arrays of offset of the found string within the page
- When false, the returned value will be an array of arrays containing two entries : the page number and the text offset.

For example, if a pdf document contains the string "here" at character offset 100 and 200 in page 1, and position 157 in page 3, the returned value will be :

- When *$group\_by\_page* is false :

		[ [ 1, 100 ], [ 1, 200 ], [ 3, 157 ] ]

- When *$group\_by\_page* is true :

		[ 1 => [ 100, 200 ], 3 => [ 157 ]
	
The parameters are the following :

- *$search* (string) : String to be searched.

- *$group\_by\_page (boolean) : Indicates whether the found offsets should be grouped by page number or not.

The method returns an array of page numbers/character offsets or *false* if the specified string does not appear in the document.


### text\_match, document\_match ###

    $status		=  $pdf -> text_match ( $pattern, &$match = null, $flags = 0, $offset = 0 ) ;
    $status		=  $pdf -> document_match ( $pattern, &$match = null, $flags = 0, $offset = 0 ) ;

*text\_match()* calls the preg\_match() PHP function on the pdf text contents, to locate the first occurrence of text that matches the specified regular expression.

*document\_match()* calls the preg\_match\_all() function to locate all occurrences that match the specified regular expression.

Note that both methods add the PREG\_OFFSET\_CAPTURE flag when calling preg\_match/preg\_match\_all so you should be aware that all captured results are an array containing the following entries :

- Item [0] is the captured string
- Item [1] is its text offset
- The *text\_match()* and *document\_match()* methods add an extra array item (index 2), which contains the number of the page where the matched text was found

Parameters are the following :

- *$pattern* (string) : Regular expression to be searched.

- *$match* (any) : Output captures. See preg\_match/preg\_match\_all.

- *$flags* (integer) : PCRE flags. See preg\_match/preg\_match\_all.

- *$offset* (integer) : Start offset. See preg\_match/preg\_match\_all.

As for their PHP counterparts, these methods return the number of matched occurrences, or *false* if the specified regular expression is invalid.

## PROPERTIES ##
 
This section describes the properties that are available in a **PdfTText** object. Note that they should be considered as read-only.

### Author ###

Author name, as inscribed in the PDF file.

### AutoSavedImageFiles ###

When the *PDFOPT\_AUTOSAVE\_IMAGES* flag has been specified, this array of strings will contain the filenames where the auto-saved images have been put.

### BlockSeparator ###

A string to be used for separating chunks of text. The main goal is for processing data displayed in tabular form, to ensure that column contents will not be catenated. However, this does not work in all cases.

The default value is the empty string.

When the *PDFOPT\_BASIC\_LAYOUT* option is specified, this property is used to separate parts of text that are visually on a different x-axis on the same line. In this case, the default separator will be a white space. 

### CIDTablesDirectory ###

Path to the directory containing CID mapping tables.

### CreationDate ###

A string containing the document creation date, in UTC format. The value can be used as a parameter to the *strtotime()* PHP function.

### CreatorApplication ###

Application used to create the original document.

### DocumentStartOffset ###

A PDF document normally starts with a string of the form :

	%PDF-a.b[.c.d...]

where "a" and "b" (and the "c", "d", ... following) represent the version number of the PDF file.

Some PDF documents may come with garbage at the beginning ; this is "illegal" of course, but Acrobat Reader is able to cope with that. So can do the **PdfToText** class...

This property holds the byte offset in the input file where the starting "%PDF" string has been found.


### EncryptionAlgorithm ###

Algorithm used for password-protected files.

The Adobe documentation states :

A code specifying the algorithm to be used in encrypting and decrypting the document :

- 0 : An alternate algorithm that is undocumented and no longer supported, and whose use is strongly discouraged.
- 1 : Algorithm 3.1.
- 2 [PDF 1.4] : Algorithm 3.1, but allowing key lengths greater than 40 bits.
- 3 [PDF 1.4] : An unpublished algorithm allowing key lengths up to 128 bits. This algorithm is unpublished as an export requirement of the U.S. Department of Commerce.

### EncryptionAlgorithmRevision ###

The revision number of the Standard security handler that is required to interpret this dictionary. The revision number is :

- 2 : for documents that do not require the new encryption features of PDF 1.4, meaning documents encrypted with an *EncryptionAlgorithm* value of 1 and using *EncryptionFlags* bits 1– 6
- 3 : for documents requiring the new encryption features of PDF 1.4, meaning documents encrypted with an *EncryptionAlgorithm* value of 2 or greater or that use the extended *EncryptionFlags* bits 17–21.

### EncryptionFlags ###

A set of *PDFPERM\_** constants describing which operations are authorized on a password-protected PDF file.

### EncryptionKeyLength ###

Defined only when *EncryptionAlgorithm* is 2 or 3. Length of key, in bits, used for encryption and decryption. The size is a multiple of 8, with a minimum value of 40 and maximum value of 128.

### EncryptionMode ###

One of the *PDFCRYPT\_\** constants.

This value is set to *PDFCRYPT\_NONE if the PDF file is not password-protected.

### EncryptMetadata ###

A flag coming from a password-protected file that says is the document metadata is also encrypted.

### EOL ###

The string to be used for line breaks. The default is PHP\_EOL.

### ExtraTextWidth ###

This property is expressed in percents ; it gives the extra percentage to add to the values computed by
the **PdfTexterFont::GetStringWidth()** method.

This value is basically used when computing text positions and string lengths with the PDFOPT_BASIC_LAYOUT option : the computed string length is shorter than its actual length (because of extra spacing determined by character kerning in the font data, among other details). 

To determine whether two consecutive blocks of text on the same should be separated by a space, the class will empirically add this extra percentage to the computed string length. The default value is -5 (percent).

### Filename ###

Name of the file whose text contents have been extracted. This value will be an empty string if the
**LoadFromString()** method has been called instead of **Load()**.

### ID, ID2 ###

A pair of unique ids generated for the document. The value of **ID** is used for decrypting password-protected documents.

The second id is not clearly described in the Pdf specifications.

### ImageAutoSaveFileTemplate ###

A template string to be used for generating filenames when the *PDFOPT\_AUTOSAVE\_IMAGES* flag has been specified.

It can contain the following *print*-like formats :

- **%p** : Replaced by the directory where the input PDF file resides.
- **%f** : Replaced by the filename part (without suffix) of the input PDF file.
- **%s** : Replaced by the suffix appropriate to the image format given by the **ImageAutoSaveFormat** property.
- **%d** : Replaced by a sequential value, starting from 1, which gives the image number.

The default value for this property is :

	$ImageAutoSaveFileTemplate	=   "%p/%f.%d.%s" ;

Using the template above, if your input filename is "/tmp/test.pdf" and contains 3 images, then you will get the following output images :

- /tmp/test.1.jpg
- /tmp/test.2.jpg
- /tmp/test.3.jpg

You can also specify a width with the "%d" format specifier ; the numbering will then be padded with leading zeroes. For example, the following template using the same example PDF file as above :

	$pdf -> ImageAutoSaveFileTemplate	=   "%p/%f.%3d.%s" ;

will generate the following filenames :

- /tmp/test.001.jpg
- /tmp/test.002.jpg
- /tmp/test.003.jpg


### ImageAutoSaveFormat ###

When the *PDFOPT\_AUTOSAVE\_IMAGES* flag has been specified, indicates the format to be used for saving the images found in the PDF file. It can be any of the constants defined by the *gd* library regarding image formats : 

			IMG_JPEG		=>  'jpg',
			IMG_JPG			=>  'jpg',
			IMG_GIF			=>  'gif',
			IMG_PNG			=>  'png',
			IMG_WBMP		=>  'wbmp',
			IMG_XPM			=>  'xpm'

Note that the association between the constant and corresponding file suffix is automatically handled.

### Images ###

An array of objects inheriting from the **PdfImage** class. Currently, only the **PdfJpegIMage** class is implemented.

The class currently supports the following properties :

- *ImageResource* : a resource that can be used with the Php *imagexxx()* functions to process image contents.

The following methods are available :

- *SaveAs ( $output\_file, $image\_type = IMG\_JPG )* : Saves the current image to the specified output file, using the specified file format (one of the predefined PHP constants : IMG\_JPG, IMG\_GIF, IMG\_PNG, IMG\_XBMP and IMG\_XBM).
- *Output ( )* : Echoes the image contents to the standard output.

Currently, images stored in proprietary Adobe format are not processed and will not appear in this array.

Note that images will be extracted only if the PDFOPT\_DECODE\_IMAGE\_DATA is enabled. 

### ImageCount ###

Number of images found in the supplied PDF file. This number will only take into account the images whose format is recognized by the **PdfToText** class.

### ImageData ###

An array of associative arrays that contain the following entries :

- 'type' : Image type. Can be one of the following :
	- 'jpeg' : Jpeg image type.	Note that in the current version, only jpeg images are processed.
- 'data' : Raw image data.

Note that image data will be extracted only if the PDFOPT\_GET\_IMAGE\_DATA is enabled. 

### IsEncrypted ###

This property is set to *true* if the Pdf file is encrypted through some kind of password protection scheme.

### MaxExecutionTime ###

Specifies a maximum execution time in seconds for processing a single file. If this limit is reached, a **PdfToTextTimeoutException** exception will be thrown before PHP terminates the script. This allows the script to gracefully handle the error instead of PHP itself.

Positive values are indicated in seconds. Negative values are subtracted from the *max\_execution\_time* PHP setting to compute the maximum execution time allowed.

If the computed timeout value is out of range, the retained execution time will be *max\_execution\_time* minus one second.

The value of this property is taken into account only if the *PDFOPT\_ENFORCE\_EXECUTION\_TIME* option has been specified.

### MaxExtractedImages ###

Maximum number of images to be extracted. The default is the value 0, meaning that all images will be selected for output if the *PDFOPT\_GET\_IMAGE\_DATA* or *PDFOPT\_AUTOSAVE\_IMAGES* option is set.


### MaxGlobalExecutionTime ###

This static property is the same as **MaxExecutionTime**, except that it works globally. If you have to process *x* files, then it will ensure that the global execution time does not exceed the value of this property.

The value of this property is taken into account only if the *PDFOPT\_ENFORCE\_GLOBAL\_EXECUTION\_TIME* option has been specified.
 

### MaxSelectedPages ###

Maximum number of pages to be selected. The default is the value 0, meaning that all pages will be selected for output.

A value of 1 will extract the contents of the first page only, which can be useful if your PDF file is large and you're only interested by the contents of the first page.

When this number is negative, selection starts from the end of the file : -1 means "extract the last page", -2 means "extract the last two pages", and so on.

### MemoryUsage, MemoryPeakUsage ###

Report the memory (peak) used by the *Load()* method.

### MinSpaceWidth ###

Sometimes, characters (or blocks of characters) are separated by an offset which is counted in 1/1000's of text units. For certain ranges of values, when displayed on a graphical device, these consecutive characters appear to be separated by one space (or more). Of course, when generating ascii output, we would like to have some equivalent of such spacing.

This is what the *MinSpaceWidth* property is meant for : insert an ascii space in the generated output whenever the offset found exceeds *MinSpaceWidth* text units. 

Note that if the *PDFOPT\_REPEAT\_SEPARATOR* flag is set for the *Options* property, the number of spaces inserted will always be based on a multiple of 1000, even if *MinSpaceWidth* is less than 1000. This means that if *MinSpaceWidth* is 200, and the *Options* property has the *PDFOPT\_REPEAT\_SEPARATOR* flag set, AND the offset between two chunks of characters is 1000 text units, only one space will be inserted, not 5 (which would be the result of 1000/200).

### ModificationDate ###

A string containing the last document modification date, in UTC format. The value can be used as a parameter to the *strtotime()* PHP function.

### Options ###

A combination of the following flags :

- *PDFOPT\_REPEAT\_SEPARATOR* : Sometimes, groups of characters are separated by an integer value, which specifies the offset to subtract to the current position before drawing the next group of characters. This quantity is expressed in thousands of "text units". The **PdfToText** class considers that if this value is less than -1000, then the string specified by the *Separator* property needs to be appended to the result before the next group of characters. If this flag is specified, then the *Separator* string will be appended (*offset* % 1000) times.
- *PDFOPT\_GET\_IMAGE\_DATA* : Store image data from the Pdf file to the **ImageData** array property.
- *PDFOPT\_DECODE\_IMAGE\_DATA* : Decode image data and put it in the **Images** array property.
- *PDFOPT\_AUTOSAVE\_IMAGES* : Auto-saves the images found in the PDF file, using a filename template given by the **ImageAutoSaveFileTemplate** property. The **ImageAutoSaveFormat** property will define the image format to be used when generating the image files. The default output format is *IMG\_JPEG*. Note that the **Images** property will be left empty. This flag has been introduced to save internal memory if you only need to extract images. 
- *PDFOPT\_IGNORE\_TEXT_LEADING* : This option must be used when you notice that an unnecessary amount of empty lines are inserted between two text elements. This is the symptom that the pdf file contains only relative positioning instructions combined with big values of text leading instructions. 
- *PDFOPT\_RAW\_LAYOUT* : Renders the text as it comes from the PDF file. This may sometimes lead to out-of-order text or strings concatenated in an inappropriate way, but this option is to be preferred if you only need to index contents or focus on performance. This is the default option.
- *PDFOPT\_BASIC\_LAYOUT* : Tries to render the text in the order you can see it with Acrobat Reader. Note that the elements will not be mapped in the output exactly as they appear with Acrobat Reader : elements physically disjoint on the x-axis will be separated by a space by default. The **BlockSeparator** property can be used to modify this separator. The following text for example :
-
		Company1							Company2
		address1							address2
		city1 								city2

will be rendered as :

		Company1 Company2
		address1 address2
		city1 city2

or, if you set the **BlockSeparator** property to "#", the output will be :

		Company1#Company2
		address1#address2
		city1#city2

- *PDFOPT\_NO\_HYPHENATED\_WORDS* : When specified, tries to join back hyphenated words into a single word. For example, the following text :

		this is a sam-
		ple text using hyphe-
		nated words that can split
		over seve-
		ral lines.
	
will be rendered as :

		this is a sample
		text using hyphenated
		words that can split
		over several lines.

- *PDFOPT\_ENFORCE\_EXECUTION\_TIME* : when specified, the **MaxExecutionTime** property will be checked against the PHP setting *max\_execution\_time*. If the time taken to process a single file may risk to take more time than the value in seconds defined for this property, a **PdfToTextTimeout** exception will be thrown before PHP tries to terminate the script execution.
- *PDFOPT\_ENFORCE\_GLOBAL\_EXECUTION\_TIME* : when specified, the **MaxGlobalExecutionTime** static property will be checked against the PHP setting *max\_execution\_time*. If the time taken to process all PDF files since the start of the script may risk to take more time than the value in seconds defined for this property, a **PdfToTextTimeout** exception will be thrown before PHP tries to terminate the script execution.
- *PDFOPT\_NONE* : Default value. No special processing flags apply.

### Pages ###

Associative array containing individual page contents. The array key is the page number, starting from 1.

### PageSeparator ###

String to be used when building the *Text* property to separate individual pages. The default value is a newline.

### ProducerApplication ###

Application used to generate the PDF file contents.

### Separator ###

A string to be used for separating blocks when a negative offset less than -1000 thousands of characters is specified between two sequences of characters specified as an array notation. This trick is often used when a pdf file contains tabular data.

The default value is a space.

### Statistics ###

An associative array that contains the following entries :

- 'TextSize' : Contains the total size in bytes represented by the Postscript-like instructions that draw the document contents
- 'OptimizedTextSize' : Not all Postscript-like instructions for drawing page contents are significant ; since the parsing is done in pure PHP, it is very slow. This is why removing useless instructions with the preg\_replace() builtin function will in most cases significantly reduce the size of the data to be parsed. This entry gives the total size of the data that will be effectively parsed after removing the useless instructions.

### Text ###

A string containing the whole text extracted from the underlying pdf file. Note that pages are separated with a form feed.

### Title ###

Document title, as specified in the author information object.

### Utf8Placeholder ###

When a Unicode character cannot be correctly recognized, the Utf8Placeholder property will be used as a substitution.

The string can contain format specifiers recognized by the sprintf() function. The parameter passed to sprintf() is the Unicode codepoint that could not be recognized (an integer value).

The default value is the empty string, or the string '[Unknown character 0x%08X]' when debug mode is enabled.

Note that if you change the *PdfToText::$DEBUG** variable **after** the first instantiation of the class, then you will need to manually set the value of the *PdfToText::Utf8PlaceHolder* static property.

## CONSTANTS ##

### PDFOPT\_\* ###

The PDFOPT\_\* constants are a set of flags which can be combined when either instantiating the class or setting the *Options* property before calling the **Load** method. It can be a combination of any of the following flags :

- *PDFOPT\_REPEAT\_SEPARATOR* : Sometimes, groups of characters are separated by an integer value, which specifies the offset to subtract to the current position before drawing the next group of characters. This quantity is expressed in thousands of "text units". The **PdfToText** class considers that if this value is less than -1000, then the string specified by the *Separator* property needs to be appended to the result before the next group of characters. If this flag is specified, then the *Separator* string will be appended (*offset* % 1000) times.
- *PDFOPT\_GET\_IMAGE\_DATA* : Store image data from the Pdf file to the **ImageData** array property.
- *PDFOPT\_DECODE\_IMAGE\_DATA* : Decode image data and put it in the **Images** array property.
- *PDFOPT\_AUTOSAVE\_IMAGES* : Auto-saves the images found in the PDF file, using a filename template given by the **ImageAutoSaveFileTemplate** property. The **ImageAutoSaveFormat** property will define the image format to be used when generating the image files. The default output format is *IMG\_JPEG*. Note that the **Images** property will be left empty. This flag has been introduced to save internal memory if you only need to extract images. 
- *PDFOPT\_IGNORE\_TEXT_LEADING* : This option must be used when you notice that an unnecessary amount of empty lines are inserted between two text elements. This is the symptom that the pdf file contains only relative positioning instructions combined with big values of text leading instructions. 
- *PDFOPT\_RAW\_LAYOUT* : Renders the text as it comes from the PDF file. This may sometimes lead to out-of-order text or strings concatenated in an inappropriate way, but this option is to be preferred if you only need to index contents or focus on performance. This is the default option.
- *PDFOPT\_BASIC\_LAYOUT* : Tries to render the text in the order you can see it with Acrobat Reader. Note that the elements will not be mapped in the output exactly as they appear with Acrobat Reader : elements physically disjoint on the x-axis will be separated by a space by default. The **BlockSeparator** property can be used to modify this separator. The following text for example :
 
		Company1							Company2
		address1							address2
		city1 								city2

will be rendered as :

		Company1 Company2
		address1 address2
		city1 city2

or, if you set the **BlockSeparator** property to "#", the output will be :

		Company1#Company2
		address1#address2
		city1#city2

- *PDFOPT\_NO\_HYPHENATED\_WORDS* : When specified, tries to join back hyphenated words into a single word. For example, the following text :

		this is a sam-
		ple text using hyphe-
		nated words that can split
		over seve-
		ral lines.
	
will be rendered as :

		this is a sample
		text using hyphenated
		words that can split
		over several lines.

- *PDFOPT\_ENFORCE\_EXECUTION\_TIME* : when specified, the **MaxExecutionTime** property will be checked against the PHP setting *max\_execution\_time*. If the time taken to process a single file may risk to take more time than the value in seconds defined for this property, a **PdfToTextTimeout** exception will be thrown before PHP tries to terminate the script execution.
- *PDFOPT\_ENFORCE\_GLOBAL\_EXECUTION\_TIME* : when specified, the **MaxGlobalExecutionTime** static property will be checked against the PHP setting *max\_execution\_time*. If the time taken to process all PDF files since the start of the script may risk to take more time than the value in seconds defined for this property, a **PdfToTextTimeout** exception will be thrown before PHP tries to terminate the script execution.
- *PDFOPT\_NONE* : Default value. No special processing flags apply.

### Pages ###

Associative array containing individual page contents. The array key is the page number, starting from 1.

### PageSeparator ###

String to be used when building the *Text* property to separate individual pages. The default value is a newline.


### VERSION ###

Current version of the **PdfToText** class, as a string containing a major, minor and release version numbers. For example : "1.2.19".

## Exceptions ##

The **PdfToText** class can throw any of the following exceptions :

### PdfToTextException ###

This is the base class for all exceptions thrown by the **PdfToText** class.

### PdfToTextDecodingException ###

This exception is thrown if an error occurs when decoding a PDF object. Normally, most of these exceptions are thrown only if debug mode is activated.

### PdfToTextDecryptionException ###

Occurs when decryption failed on an encrypted file.

### PdfToTextTimeoutException ###

This exception is thrown only if one of the following conditions occur :

- The *PDFOPT\_ENFORCE\_EXECUTION\_TIME* option has been set, and processing one file took more time than the number of seconds computed from	the **$MaxExecutionTime** property
- The *PDFOPT\_ENFORCE\_GLOBAL\_EXECUTION\_TIME* option has been set, and processing all the files your script had to process took more than the number of seconds computed from the **$MaxGlobalExecutionTime** static property