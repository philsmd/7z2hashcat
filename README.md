# About

The goal of this project is to make it very easy to convert password-protected .7z files (and .sfx files which contain .7z files) to "hashes" which hashcat can crack with mode: -m 11600 = 7-Zip

# Requirements

Software:  
- if you are using the windows operating system, you could just use the [release version](https://github.com/philsmd/7z2hashcat/releases)
- if you are using a different operating system or if you prefer to not use the .exe file:
  - Perl must be installed (should work on *nix and windows with perl installed)
  - Perl module "Compress::Raw::Lzma" is required (this is needed because the .7z header will be compressed whenever it contains several files and a non-encrypted header is used)
    - example on how to install it on ubuntu:  
    `sudo apt install libcompress-raw-lzma-perl`  
    or  
    `sudo cpan Compress::Raw::Lzma`
  - Perl module "File::Basename" is required for the John The Ripper output format (with file names in the hash)

Note: for windows users the [release page](https://github.com/philsmd/7z2hashcat/releases) provides executable files (.exe) which should work AS-IS without the need to install perl or perl modules.  
Attention: the release version (7z2hashcat.exe) might not be up-to-date with the newest source code all the time, therefore please prefer to use the source code version (instead of the .exe version), especially when you experience some problems and want to report issues.


# Installation and first steps

Note: this paragraph is only intended for users that do not use the release version for windows.
You should be able to just run the `7z2hashcat` executable (`.exe`) within `cmd` (see note below) if you are a windows user.

* Clone this repository:  
    ```git clone https://github.com/philsmd/7z2hashcat.git```
* Enter the repository root folder:  
    ```cd 7z2hashcat```
* Run it:  
    ```perl 7z2hashcat.pl file.7z```
* Copy output to a file (or redirect output to a file (>) directly) and run it with hashcat using mode -m 11600 = 7-Zip

Note: independent from how you launch `7z2hashcat`, i.e. either the perl script directly or the windows executable, you always need to launch your shell/`cmd`/`konsole`/`xterm` application first and only afterwards start `7z2hashcat`, otherwise, if you for instance just double-click the script/executable, it might appear to you that the program just opens and closes immediately.

# Command line parameters

The usage is very simple: you just specify the path to the 7-Zip file as the first command line argument.  
  
You can also use multiple files on the command line like this:  
```
perl 7z2hashcat.pl file1.7z file2.7z file3.sfx
perl 7z2hashcat.pl *.7z
perl 7z2hashcat.pl seven_zip_files/*
perl 7z2hashcat.pl splitted_7z_files/huge_file.7z.*
```
  

Note: on windows you can use the release files (.exe) and therefore you shouldn't forget to replace the ".pl" extension with ".exe"  
Note2: you can also use the perl script on windows directly after installing the [requirements](#requirements) e.g. ```perl 7z2hashcat.pl ...```

# Explanation of the hash format

The following paragraph explains some details about the output of 7z2hashcat.  
You do not need to understand or know all this information for just cracking hashes. Instead, this is just some documentation about the different fields within the output.  
  
7z2hashcat outputs one hash per line. Warning and error messages are outputted to STDERR and therefore shouldn't interfere with the outputted "hashes".  
  
Each hash line has several fields separated by the dollar character ($), but some fields can sometimes be omitted (indicated by "always outputted: no" in the table below). This depends whether the fields are needed or not.  
  
This is an overview of the output:  
  
| $ | content of the field       | always outputted | Explanation                                                                                     |
|---|----------------------------|------------------|-------------------------------------------------------------------------------------------------|
| $ | "7z"                       | yes              | the literal string "7z" indicates the type of the hash                                          |
| $ | [data type indicator]      | yes              | a number ranging from 0 to 255 to indicate truncation and compression (see below)               |
| $ | [cost factor]              | yes              | the cost factor indicates how many iterations need to be performed (2 ^ [cost factor])          |
| $ | [length of salt]           | yes              | the length of the following field (the salt)                                                    |
| $ | [salt]                     | yes              | the hexadecimal output of the salt                                                              |
| $ | [length of iv]             | yes              | the length of the initialization vector (values from 0 to 16)                                   |
| $ | [iv]                       | yes              | the initialization vector in hexadecimal form                                                   |
| $ | [CRC32]                    | yes              | the actual "hash" aka the CRC checksum in decimal form                                          |
| $ | [length of encrypted data] | yes              | the length of the encrypted data (see [encrypted data])                                         |
| $ | [length of decrypted data] | yes              | the length of the output of the AES decryption of [encrypted data]                              |
| $ | [encrypted data]           | yes              | the encrypted data itself (this field in some cases could be truncated, see below)              |
| $ | [length of data for CRC32] | no               | optional field indicating the length of the first "file" in case decompression needs to be used |
| $ | [coder attributes]         | no               | optional field indicating the comma-separated list of attributes for the decompressor(s)        |
| $ | [preprocessor attributes]  | no               | optional field indicating the comma-separated list of attributes for the preprocessor           |

The **data type indicator** is a special field and needs some further explanation:  
  
This field is the first field after the hash signature (i.e. after "$7z$").  
Whenever the data is longer than the value of PASSWORD_RECOVERY_TOOL_DATA_LIMIT (see 7z2hashcat.pl) and an AES padding attack is possible, the value will be 128 and [data] will be truncated (a warning message will be shown in case the data limit was reached but padding attack is not applicable).  
  
If no truncation is used/possible:  
- the value will be 0 if the data doesn't need to be decompressed to check the CRC32 checksum
- all values different from 128, but greater than 0 indicate that the data must be decompressed as follows:  
   - Lower nibble (4 bits, type & 0xf):
     - 1 means that the data must be decompressed using the LZMA1 decompressor
     - 2 means that the data must be decompressed using the LZMA2 decompressor
     - 3 means that the data must be decompressed using the PPMD decompressor
     - 4 reserved (future use)
     - 5 reserved (future use)
     - 6 means that the data must be decompressed using the BZIP2 decompressor
     - 7 means that the data must be decompressed using the DEFLATE decompressor
     - 8-15 reserved (future use)
   - Upper nibble (4 bits, (type >> 4) & 0xf):
     -  1 means that the data must be post-processed using BCJ (x86)
     -  2 means that the data must be post-processed using BCJ2 (four data streams needed)
     -  3 means that the data must be post-processed using PPC (big-endian)
     -  4 means that the data must be post-processed using IA64
     -  5 means that the data must be post-processed using ARM (little-endian)
     -  6 means that the data must be post-processed using ARMT (little-endian)
     -  7 means that the data must be post-processed using SPARC
     -  8 unavailable since this indicates TRUNCATION (128 == 0x80 == 8 << 4)
     -  9 means that the data must be post-processed using DELTA
     - 10 means that the data must be post-processed using 7zAES
     - 11-15 reserved (future use)

Truncated data can only be verified using the padding attack and therefore combinations between truncation and a compressor are not meaningful/allowed.  
  
Therefore, whenever the value is 128 or 0, neither coder attributes nor the length of the data for the CRC32 check is within the output.  
  
On the other hand, for all values above or equal 1 and smaller than 128, both coder attributes and the length of the decompressed data for CRC32 check is within the output.  
  
The following table should sum up the most common data type indicator values pretty nicely:  

| data type indicator | Explanation  |
| --------------------|--------------|
| 0                   | uncompressed |
| 1                   | LZMA         |
| 2                   | LZMA2        |
| 3                   | PPMD         |
| 6                   | BZIP2        |
| 7                   | DEFLATE      |
| 128                 | truncated    |

Whenever the data needs to be either (pre)processed by multiple (2+) filters or whenever the data needs to be decompressed by multiple (2+) decompression algorithms, the attribute list (the fields `preprocessor attributes` and `coder attributes` accordingly) will be a comma-separated list of fields with type, order/position and attribute indicators.  
  
The rules for this **Multiple Compressor(s)/Preprocessor(s)** list are as follows:  
- all (additional) methods/coders need to be specified in the output hash format (the fields `coder attributes` and `preprocessor attributes` accordingly), even if they have no attributes
- the format is a comma-separated list of type and order indicator (for both fields: compressor and preprocessor attributes), followed by an underscore (_), followed by the attribute itself
- the **first attribute**/item/codec for both fields (compressor and preprocessor attributes) needs no type and order indicator, only attributes
- the type and position/order of the first decompressor/filter are implied (it's always the first decompressor/filter and the type is in the `data type indicator` field)
- this also means that the field `data type indicator` (see format above) only indicates the **first** decompressor (if used) and (also, if used) the first preprocessing filter (this is also due to compatibility reasons with older formats)
- for the additional (2+) decompressors/filters, the first number after the comma is the type and order indicator, after which follows an underscore (_) and the attributes themself (could be empty/no attribute value for some codecs)
- special case: the `coder attributes` field could start with a comma (and has also a type and order indicator) in the unlikely case where during archive generation a filter was applied after the final compression of the data. Otherwise it's always the case that the decompressor needs to be applied first
  
Multiple Compressor(s)/Preprocessor(s) type and order indicator (it is one combined field/number):  
- upper nibble (4 bits, indicator >> 4) indicates the compressor/preprocessor **method**/type/codec (LZMA2, Delta, BCJ etc)
- lower nibble (4 bits, indicator & 0xf) indicates the order/position (the step at which the (de)compressor/(pre)processor needs to be applied). The values are incrementing, the counter starts with 1

# Sensitive data warning

WARNING: as you can see from the hash format explanation above the hashes themself could sometimes contain sensitive data (in some cases the data is both encrypted and compressed). You should be careful when it comes to sharing the output of 7z2hashcat because people that understand the format might be able to extract sensitive data out of the decrypted (and decompressed) data.

# hc_to_7z

For debugging / troubleshooting purposes, we have also developed a tool that tries to reverse the work of `7z2hascat` to try to generate a valid `7-Zip` (`*.7z`) archive file from hash lines (output of `7z2hashcat`). This is a proof of concept that you can find here: https://github.com/philsmd/hc_to_7z

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
  
Disclaimer: WE PROVIDE THE PROGRAM “AS IS” WITHOUT WARRANTY OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE  
  
NO GUARANTEES THAT IT WORKS FOR YOU AND WORKS CORRECTLY
