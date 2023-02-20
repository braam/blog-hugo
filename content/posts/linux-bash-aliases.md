---
title: "Linux Bash Aliases"
date: 2022-01-31T15:55:21+01:00
draft: false
tags:
- Linux
---

### Overview/backup of my .bash_aliases
* one-liners aliases
* functions based aliases to accept arguments.

```bash
## ONE-LINERS ##
#Always add space at the end from sudo.
alias sudo="sudo "

#Get public IP and other information
alias myPublicIP="curl https://ifconfig.co/json"

#Ping open ports with ncurses, when PABX is rebooting this is handy to see when management is back online.
alias httping="httping -K"

#Create a hotspot from wired to wireless. (see https://github.com/oblique/create_ap)
alias hotspot="/home/braam/git/create_ap/./create_ap wlp1s0 enp0s31f6 Mint_AP Wifi_adhoc123"

#Easily flush dns cache
alias flushdns="systemd-resolve --flush-caches"

#Music CD ripping
alias cdripping="abcde -o mp3:-V2"

#list DHCP server who gives us a lease.
alias findDHCP="sudo grep -IR "DHCPOFFER" /var/log/*"


## FUNCTIONS ##
function weather {
 if [ -z "$1" ]; then
    # display usage if no paramters given
    echo "Usage: weather <location>"
 else
    curl http://wttr.in/~$1
 fi
}

#Test incoming public port on currently connect public IP.
function testPublicPort {
 if [ -z "$1" ]; then
    # display usage if no paramters given
    echo "Usage: testPublicPort <portnumber>"
 else
    curl ifconfig.co/port/$1
 fi
}

#Get location from IP address.
function getIpLocation {
 if [ -z "$1" ]; then
    # display usage if no paramters given
    echo "Usage: getIPLocation <IP address>"
 else
    curl ifconfig.co/json?ip=$1
 fi
}

#Replaces spaces with underscore.
function replaceSpace() {
 # Traverse all files in directory
 for file in *' '*
   do
     mv -- "$file" "${file// /_}"
 done
}

#Convert wav files to MOH audio format.
function wavPABX {
 if [ -z "$1" ]; then
    # display usage if no parameters given
    echo "Usage: wavPABX <path/file_name>.<mp3|m4a|wav|..."
    echo        "wavPABX <path/file_name_1.ext> [path/file_name_2.ext]"
 else
    for n in $@
    do
      if [ -f "$n" ] ; then
          ffmpeg -i "$n" -acodec pcm_s16le -ac 1 -ar 8000 _$n
      else
          echo "'$n' - file does not exist"
          return 1
      fi
    done
 fi
}

#Replaces extension
function extensionSwap {
 if [ "$#" -lt 3 ]; then
    # display usage if no parameters given
    echo "Usage: extensionSwap <oldExtension> <newExtension> <files>"
    echo $#
 else
   rename "s/$1/$2/" *
 fi
}

#Extract command for all kind of archived file types.
function extract {
 if [ -z "$1" ]; then
    # display usage if no parameters given
    echo "Usage: extract <path/file_name>.<zip|rar|bz2|gz|tar|tbz2|tgz|Z|7z|xz|ex|tar.bz2|tar.gz|tar.xz>"
    echo "       extract <path/file_name_1.ext> [path/file_name_2.ext] [path/file_name_3.ext]"
 else
    for n in $@
    do
      if [ -f "$n" ] ; then
          case "${n%,}" in
            *.tar.bz2|*.tar.gz|*.tar.xz|*.tbz2|*.tgz|*.txz|*.tar) 
                         tar xvf "$n"       ;;
            *.lzma)      unlzma ./"$n"      ;;
            *.bz2)       bunzip2 ./"$n"     ;;
            *.rar)       unrar x -ad ./"$n" ;;
            *.gz)        gunzip ./"$n"      ;;
            *.zip)       unzip ./"$n"       ;;
            *.z)         uncompress ./"$n"  ;;
            *.7z|*.arj|*.cab|*.chm|*.deb|*.dmg|*.iso|*.lzh|*.msi|*.rpm|*.udf|*.wim|*.xar)
                         7z x ./"$n"        ;;
            *.xz)        unxz ./"$n"        ;;
            *.exe)       cabextract ./"$n"  ;;
            *)
                         echo "extract: '$n' - unknown archive method"
                         return 1
                         ;;
          esac
      else
          echo "'$n' - file does not exist"
          return 1
      fi
    done
 fi
}

#Uses transfer.sh for sharing files from command line very easily.
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

#Get all mp3 files from web-index (Google search, index:of mp3)
mp3get() {
    wget -c -A '*.mp3' -r -l 1 -nd $1
}

#Get all mkv files from web-index (Google search, index:of mkv)
mkvget() {
    wget -c -A '*.mkv' -r -l 1 -nd $1
}

#Create one time sharing notes easily.
1tyme() { 
    # check arguments
    if [ $# -eq 0 ]; 
    then 
        echo "No arguments specified. Usage: $0 <note>"
        return 1
    fi
    
    #post note, combine full url
    printf "https://1ty.me/" && curl -s "https://1ty.me/?mode=ajax&cmd=create_note" --data-raw 'note='$1'&email=&reference=&newsletter=' | jq -r '.url'
}
```
