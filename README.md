# About

The goal of this project is to make it very easy to convert .7z files to "hashes" which hashcat can crack with mode: -m 11600 = 7-Zip

# Requirements

Software:  
- Perl must be installed (should work on *nix and windows with perl installed) 
- Perl module "Compress::Raw::Lzma" (this is needed because .7z header will be compressed whenever there are more files and non-encrypted header is used)

Note: for windows users the [release page](https://github.com/philsmd/7z2hashcat/releases) provides executable files (.exe) which should work AS-IS without the need to install perl or perl modules.
Attention: the release version (7z2hashcat.exe) might not be up-to-date with the newest source code all the time, therefore please prefer to use the source code version (instead of the .exe version), especially when you experience some problems and want to report issues.


# Installation and first steps

Note: this paragraph is only intended for users that do not use the release version for windows.
You should be able to just run 7z2hashcat.exe if you are a windows user.

* Clone this repository:  
    git clone https://github.com/philsmd/7z2hashcat.git  
* Enter the repository root folder:  
    cd 7z2hashcat
* Run it:  
    ./7z2hashcat.pl file.7z
* Copy output to a file (or redirect output to a file (>) directly) and run it with hashcat using mode -m 11600 = 7-Zip

# Command line parameters 

You can use several files on the command line like this:   
    ./7z2hashcat.pl file1.7z file2.7z  
    ./7z2hashcat.pl \*.7z  
    ./7z2hashcat.pl seven_zip_files/\*  

Note: on windows you can use the release files (.exe) and therefore you shouldn't forget to replace the ".pl" extension with ".exe"
Note2: you can also use the perl script on windows directly after installing the [requirements](#requirements) e.g. perl 7z2hashcat.pl ...

# Hacking / Missing features

* More features
* CLEANUP the code, use more coding standards, make it easier readable, everything is welcome (submit patches!)
* testing/add support for "external files"
* keep it up-to-date with 7zip source code and/or p7zip
* improvements and all bug fixes are very welcome 
* solve and remove the TODOs (if any exist)
* and,and,and

# Credits and Contributors 
Credits go to:  
  
* philsmd, hashcat project

# License/Disclaimer

License: belongs to the PUBLIC DOMAIN, donated to hashcat, credits MUST go to hashcat and philsmd for their hard work. Thx  
  
Disclaimer: WE PROVIDE THE PROGRAM “AS IS” WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE Furthermore, NO GUARANTEES THAT IT WORKS FOR YOU AND WORKS CORRECTLY
