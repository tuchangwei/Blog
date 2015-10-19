title: CLOC - Count Lines Of Code on Mac
date: 2014-12-06 09:41:44
tags: CLOC
---
1. Download the [cloc-1.62.pl](http://sourceforge.net/projects/cloc/files/latest/download?source=files), rename it to cloc.pl then copy it.
2. Open 'finder' and browse to usr/local/bin on your Mac partition
paste it there
3. In your favorite editor make new text file and paste the following:
```bash
 #!/bin/bash
if [ "$1" == "/" ]; then
    echo "wrong input, cloc can't work on."
else
    perl /usr/local/bin/cloc.pl "$1" 
fi 
```
4. Save it as 'cloc' [without extension] on same folder [/usr/local/bin]
5. Ppen 'terminal' from 'applications'
6. Browse to the folder by typing 'cd /usr/local/bin'
7. Type the following 'chmod +x cloc' in order to give permission for the file 
8. We made to execute using cloc is so easy now, you only need to open terminal and type 'cloc /path/to/your/project/'.

screenshot:
![image](/2014/12/06/CLOC-Count-Lines-Of-Code-on-Mac/screenshot.jpg)

Refer to: [http://me.syrex.me/2011/10/cloc-count-lines-of-code-on-mac.html](http://me.syrex.me/2011/10/cloc-count-lines-of-code-on-mac.html)