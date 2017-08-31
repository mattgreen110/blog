---
title: Search Text Inside Of A PDF Using PHP 
date: 03/10/2015
taxonomy:
    category: blog
    tag: [flysystem, xpdf, your mom]
---

__End result:__ The ability for users to search for terms within a given file or collection of files.

I like to find the quickest way to do something with a relatively low learning curve. After searching for a while and trying a couple packages that didn't work out, I ended up using XPDF, Flysystem and good ol' PHP. I managed to get a working solution in an afternoon.

##What do I want this thing to do?
* Return a collection of PDFs within a given directory
* Loop through that collection and extract the text from each PDF
* Search this text for a given term and return a boolean value
* Return a collection of PDFs that were a match (returned true)

###1. Install [XPDF](http://www.foolabs.com/xpdf/)

The easiest way to install locally is to just use [homebrew](http://brew.sh/). 

```javascript
$ brew install xpdf
```

To find the path of the binary: 
```javascript
$ which xpdf
/usr/local/bin/xpdf
```

For some of my production servers, I use [arcustech](https://www.arcustech.com/) so I just asked them to install it and they did. They are awesome. 

Now on XPDF's website you won't find an award winning design or a page labeled "documentation". I'll save you the trouble, go [here](http://www.foolabs.com/xpdf/README). In this file you see a section titled "Running Xpdf". This is pretty much all you need. That section is below:

###2. Running Xpdf 

* To run xpdf, simply type: __xpdf file.pdf__
* To generate a PostScript file, hit the "print" button in xpdf, or run pdftops: __pdftops file.pdf__
* To generate a plain text file, run pdftotext: __pdftotext file.pdf__

There are five additional utilities (which are fully described in their man pages):
* __pdfinfo__ - dumps a PDF file's Info dictionary (plus some other useful information)
* __pdffonts__ - lists the fonts used in a PDF file along with various information for each font
* __pdfdetach__ - lists or extracts embedded files (attachments) from a PDF file
* __pdftoppm__ - converts a PDF file to a series of PPM/PGM/PBM-format bitmaps
* __pdfimages__ - extracts the images from a PDF file

I wanted to include this because it's pretty useful. For this we just need the "pdftotext" command. 

###3. Scan Directory and Return a collection of PDFs

 __For the architecture police:__
For the sake of simplicity and readability I am not worrying too much about architecture here. Use your discretion depending on your app. You may want to use commands, events, services, IOC container, etc... but it depends on so many factors. These examples are a foundation that you can apply to the architecture and naming conventions that suit whatever you are building. 
 
[Flysystem](http://flysystem.thephpleague.com/) is a package from [The League](http://thephpleague.com/). Their [docs](http://flysystem.thephpleague.com/api/) are straightforward and will get you going. I will be using it for much of the file system functionality. 

This simple snippet will return an array of filenames in the given directory ($path): 

```php
$this->filesystem = new Filesystem(new Adapter($path));
$directoryContents = $this->filesystem->listContents();
```

###4. Loop through each pdf and extract the text

Below is a simple function that filters and returns the pdfs within $directoryContents

```php 
/**
 * getPdfsFromDirectory
 * @param $directory
 * @return array
 */
public function getPdfsFromDirectory($directory)
{
    $pdfs = [];
    foreach($directory as $key => $value)
    {
        if ($value['extension'] == 'pdf') {
            $pdfs[] = $value['path'];
        }
    }

    return $pdfs;
}
```

And we need a method for extracting the text out of the PDF. This uses Symfony's Process Class.

```php 
/**
 * getFileContents
 * @param $file
 * @param $path
 * @return mixed
 */
public function getFileContents($file)
{

    if ($filesystem->getVisibility($file) === 'private') {
        $filesystem->setVisibility($file, 'public');
    }

    $temp = 'temp.txt';
    $command = '/usr/local/bin/pdftotext -enc UTF-8 ' . $this->path . $file . ' ' . $this->path . $temp;
    $process = new Process($command);

    try {

        $process->run();
        return $filesystem->readAndDelete($temp);

    } catch(\Exception $e) {

        return $e->getMessage();

    }
}
```

Now we need a method that we can pass along the keyword we are searching for. 

```php
/**
 * searchDir
 * @param $keyword
 * @param null $path
 * @return array
 */
public function searchDir($keyword)
{
    $this->filesystem = new Filesystem(new Adapter($this->path));
    $directoryContents = $this->filesystem->listContents();
    
    $results = [];

    // scan each file in that array, save it to .txt file
    foreach($this->getPdfsFromDirectory($directoryContents) as $key => $value)
    {
        // Extract the text out of the file
        $fileData = $this->getFileContents($value, $this->path);

        // If the text contains our keyword, add it to the result
        if(str_contains($fileData, $keyword)) {
            $results[] = $value;
        }
    }

    return $results;

}
```

Here is the entire Service Class.

```php

use League\Flysystem\Filesystem;
use League\Flysystem\Adapter\Local as Adapter;
use Symfony\Component\Process\Process;

class SearchFileSystemService {

    protected $filesystem;
    protected $path;
    
    /**
     * @param $path
     */
    public function __construct($path)
    {
        $this->path = $path;
    }

    /**
     * getPdfsFromDirectory
     * @param $directory
     * @return array
     */
    public function getPdfsFromDirectory($directory)
    {
        $pdfs = [];
        foreach($directory as $key => $value)
        {
            if ($value['extension'] == 'pdf') {
                $pdfs[] = $value['path'];
            }
        }
    
        return $pdfs;
    }

    /**
     * searchDir
     * @param $keyword
     * @param null $path
     * @return array
     */
    public function searchDir($keyword)
    {
        $this->filesystem = new Filesystem(new Adapter($this->path));
        $directoryContents = $this->filesystem->listContents();

        $results = [];
        
        // scan each file in that array, save it to .txt file
        foreach($this->getPdfsFromDirectory($directoryContents) as $key => $value)
        {
           // Extract the text out of the file
           $fileData = $this->getFileContents($value, $this->path);
   
           // If the text contains our keyword, add it to the result
           if(str_contains($fileData, $keyword)) {
               $results[] = $value;
           }
        }

        return $results;

    }

    /**
     * getFileContents
     * @param $file
     * @param $path
     * @return mixed
     */
    public function getFileContents($file)
    {
        if ($this->filesystem->getVisibility($file) === 'private') {
            $this->filesystem->setVisibility($file, 'public');
        }

        $temp = 'temp.txt';
        $command = '/usr/local/bin/pdftotext -enc UTF-8 ' . $this->path . $file . ' ' . $this->path . $temp;
        $process = new Process($command);

        try {

            $process->run();
            return $filesystem->readAndDelete($temp);

        } catch(\Exception $e) {

            return $e->getMessage();

        }
    }

} 
```

Here I am using str_contains() to do the searching but for more options use a function that leverages regular expressions if you want. The way you would use this service class would look something like this: 

```php

$files = new SearchFileSystemService('directory/path/here');
$searchResults = $files->searchDir('keyword');
 
 ```

This would return an array of file paths you can do whatever you want with. Unless you don't care about performance, this use case is not very practical. If you are searching within large PDFs or many PDFs you probably want to go another route because it will be pretty slow. You can extract the text and save it to a database that you can index for searching, or push it to something like elasticsearch and run cron jobs to keep everything in sync. There are many options and it depends on your use case and the problem you are trying to solve.

This all assumes you are using a Linux environment. For Windows you can bypass most of this with a command called "findstr". It's similar to grep and searches the Windows file system with support for searching Word docs, PDFs, Excel files, etc. You can search with regular expressions, use flags to specify output and much more. Not only all of that but it's MUCH faster since it leverages Windows File System index. I'd like to post about "findstr" but I would also like to play MarioKart that night in the future so who knows.
 









