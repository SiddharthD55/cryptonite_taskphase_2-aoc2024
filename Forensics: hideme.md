In this write-up, I extracted a hidden flag from an image file using `binwalk`. This challenge was quite the adventure, filled with multiple steps, some troubleshooting, and learnings along the way.

![Screenshot (5)](https://github.com/user-attachments/assets/bccb506b-98cd-4c9d-a8f5-d311664d285b)


## Step 1: Analyzing the Image with Binwalk

To kick things off, I used the `binwalk` tool to analyze the `flag.png` file. Binwalk is a specialized tool for analyzing and extracting embedded files and executable code from binary files. Here’s the initial output of the `binwalk` command:

```sh
$ binwalk flag.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 512 x 504, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, compressed
39739         0x9B3B          Zip archive data, at least v1.0 to extract, name: secret/
39804         0x9B7C          Zip archive data, at least v2.0 to extract, compressed size: 2997, uncompressed size: 3152, name: secret/flag.png
43036         0xA81C          End of Zip archive, footer length: 22
```

From this output, I gathered important details about the `flag.png` file:
- **PNG Image**: The file is a PNG image with dimensions 512 x 504 pixels, using 8-bit color RGBA, and it is non-interlaced.
- **Zlib Compressed Data**: Found at offset 0x29.
- **Zip Archive**: Detected at offset 0x9B3B, with entries including a `flag.png` file inside a `secret/` folder.

## Step 2: Attempting Extraction with Binwalk

Encouraged by the findings, I attempted to extract the embedded files using the `-e` option with `binwalk`. However, I hit a snag—a permission-related exception. Binwalk requires third-party utilities to be run as the root user due to security considerations.

```sh
$ binwalk -e flag.png --run-as=root

Extractor Exception: Binwalk extraction uses many third party utilities, which may not be secure. If you wish to have extraction utilities executed as the current user, use '--run-as=root' (binwalk itself must be run as root).
----------------------------------------------------------------------------------------------------
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/binwalk/core/module.py", line 258, in __init__
    self.load()
  File "/usr/lib/python3/dist-packages/binwalk/modules/extractor.py", line 147, in load
    raise ModuleException("Binwalk extraction uses many third party utilities, which may not be secure. If you wish to have extraction utilities executed as the current user, use '--run-as=%s' (binwalk itself must be run as root)." % user_info.pw_name)
binwalk.core.exceptions.ModuleException: Binwalk extraction uses many third party utilities, which may not be secure. If you wish to have extraction utilities executed as the current user, use '--run-as=root' (binwalk itself must be run as root).
----------------------------------------------------------------------------------------------------
```

## Step 3: Running Binwalk with Correct Permissions

To resolve this, I decided to run the `binwalk` command with the necessary permissions by using root privileges:

```sh
$ sudo binwalk -e flag.png --run-as=root

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 512 x 504, 8-bit/color RGBA, non-interlaced
41            0x29            Zlib compressed data, compressed
39739         0x9B3B          Zip archive data, at least v1.0 to extract, name: secret/
39804         0x9B7C          Zip archive data, at least v2.0 to extract, compressed size: 2997, uncompressed size: 3152, name: secret/flag.png
43036         0xA81C          End of Zip archive, footer length: 22
```

This time, the command executed successfully without any issues.

## Step 4: Locating the Extracted Files

After executing the command, I navigated to the `_flag.png.extracted` directory to find the extracted files:

```sh
$ ls
Dockerfile                  _flag.png-2.extracted  chall.py:Zone.Identifier  ld-linux-x86-64.so.2                  out.txt                    tetris.pcapng
Dockerfile:Zone.Identifier  _flag.png.extracted    chall:Zone.Identifier     ld-linux-x86-64.so.2:Zone.Identifier  out.txt:Zone.Identifier    tetris.pcapng:Zone.Identifier
_flag.png-0.extracted       chall                  flag.png                  libc.so.6                             rAnDoM.py                  tetris.py
_flag.png-1.extracted       chall.py               flag.png:Zone.Identifier  libc.so.6:Zone.Identifier             rAnDoM.py:Zone.Identifier  tetris.py:Zone.Identifier
```

I found several extracted directories and navigated to `_flag.png.extracted`:

```sh
$ cd _flag.png.extracted
$ ls
29  29.zlib  9B3B.zip
```

## Step 5: Extracting the Zip Archive

The next step was to extract the contents of the `9B3B.zip` file:

```sh
$ unzip 9B3B.zip
Archive:  9B3B.zip
   creating: secret/
  inflating: secret/flag.png
```

This command created a `secret` directory containing the `flag.png` file.

## Step 6: Verifying the Extraction

To ensure everything was extracted correctly, I navigated to the `secret` directory and listed its contents:

```sh
$ cd secret
$ ls
flag.png
```

## Step 7: Viewing the Hidden Flag

Finally, I opened the `flag.png` file using an image viewer to view the hidden flag:

- On **Linux** (using `eog`):
  ```sh
  eog flag.png
  ```

- On **Windows** (using File Explorer):
  Simply double-click the `flag.png` file.

- On **MacOS** (using Preview):
  Simply double-click the `flag.png` file.

And there it was—the hidden flag revealed in all its glory!

## Learnings from This Challenge

This challenge was not just about extracting a hidden flag but also about the lessons learned along the way:

1. **Understanding File Structures**: Analyzing the `binwalk` output gave me a better understanding of file structures and how data can be embedded within files.
2. **Handling Permissions**: Encountering permission-related issues highlighted the importance of running commands with the appropriate privileges, especially when dealing with security-sensitive utilities.
3. **Tool Familiarity**: The process underscored the importance of familiarity with tools like `binwalk`, `unzip`, and command-line operations. Knowing how to navigate directories and manipulate files is crucial in challenges like this.
4. **Troubleshooting Skills**: The need to troubleshoot and resolve issues, such as missing utilities or incorrect permissions, reinforced the importance of problem-solving skills and perseverance.

## Conclusion

This challenge was a good exercise in using `binwalk` to analyze and extract hidden data from an image file. By meticulously following the steps, troubleshooting permissions issues, and using the right commands, I successfully uncovered the hidden flag.

![Screenshot (7)](https://github.com/user-attachments/assets/54a8458b-591a-43ae-800f-23a3d6e06de0)


![Screenshot (8)](https://github.com/user-attachments/assets/22709baf-6c0a-4a99-a226-77c8e53d122c)


![Screenshot (6)](https://github.com/user-attachments/assets/fefa4337-494c-4a8a-9c8c-d81f0c005d35)


**picoCTF{Hiddinng_An_imag3_within_@n_ima9e_85e04ab8}**
