---
title: "Transfer Files From Shell"
date: 2022-01-21T21:47:09+01:00
draft: false
tags:
- linux
- terminal
---

I often need to transfer big files between customers and office, or between customers or whatever.. The main problem is, that it has to be done fast and without much interaction. I discovered the webservice [transfer.sh](https://transfer.sh), it's fast easy and scriptable. You can mail / encrypt / check viruses etc..

You can in bash use this simple alias to easily use the transfer command:

```
transfer() { 
    # check arguments
    if [ $# -eq 0 ]; 
    then 
        echo "No arguments specified. Usage:\necho transfer /tmp/test.md\ncat /tmp/test.md | transfer test.md"
        return 1
    fi

    # get temporarily filename, output is written to this file show progress can be showed
    tmpfile=$( mktemp -t transferXXX )
    
    # upload stdin or file
    file=$1

    if tty -s; 
    then 
        basefile=$(basename "$file" | sed -e 's/[^a-zA-Z0-9._-]/-/g') 

        if [ ! -e $file ];
        then
            echo "File $file doesn't exists."
            return 1
        fi
        
        if [ -d $file ];
        then
            # zip directory and transfer
            zipfile=$( mktemp -t transferXXX.zip )
            cd $(dirname $file) && zip -r -q - $(basename $file) >> $zipfile
            curl -k --progress-bar --upload-file "$zipfile" "https://transfer.sh/$basefile.zip" >> $tmpfile
            rm -f $zipfile
        else
            # transfer file
            curl -k --progress-bar --upload-file "$file" "https://transfer.sh/$basefile" >> $tmpfile
        fi
    else 
        # transfer pipe
        curl -k --progress-bar --upload-file "-" "https://transfer.sh/$file" >> $tmpfile
    fi
    # cat output link
    cat $tmpfile
    # cleanup
    rm -f $tmpfile
}
```

