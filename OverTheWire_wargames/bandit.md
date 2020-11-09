# Bandit

> The Bandit wargame is aimed at absolute beginners. It will teach the basics needed to be able to play other wargames.

---

level 0 - ssh 
- `ssh -p 2220 bandit0@bandit.labs.overthewire.org` + password [*bandit0*] 

---

level 1 - finding a file "readme"
- `ls`
- `cat readme`
- password = *9jbbUNNfktd78OOpsqOltutMc3MY1*
- `ssh -p 2220 bandit1@bandit.labs.overthewire.org` + password

---

level 2 - finding a file called "-"
- special file (stdin), need to run it as executable
- `cat ./-`
- password = *CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9*
- `ssh -p 2220 bandit2@bandit.labs.overthewire.org` + password

---

level 3 - reading a file with spaces in filename
- `cat "spaces in file name"`
- password = *boJ9jbbUNNfktd78OOpsqOltutMc3MY1* 
- `ssh -p 2220 bandit3@bandit.labs.overthewire.org` + password

---

level 4 - finding a hidden file 
- `ls -ali`
- `cd inhere`
- `ls -ali`
- `cat .hidden`
- password = pIwrPrtPN36QITSp3EQaw936yaFoFgAB
- `ssh -p 2220 bandit4@bandit.labs.overthewire.org` + password

---

level 5 - finding a human readable file
- `cd inhere`
- `cat ./-file07` - trial and error finding correct file 
- pasword = koReBOKuIDDepwhWk7jZC0RTdopnAYKh
- `ssh -p 2220 bandit5@bandit.labs.overthewire.org` + password

---

level 6 - finding a file thats; human-readable, 1033 bytes in size, not executable
- `cd inhere`
- `find . -readable -size 1033c ! -executable`
- `cat ./maybehere07/.file2`
- password = DXjZPULLxYr17uwoI01bNLQbtFemEgo7
- `ssh -p 2220 bandit6@bandit.labs.overthewire.org` + password

---

level 7 - finding a file (anywhere on server) that is; owned by user bandit7, owned by group6, 33 bytes in size
- `find / -user bandit7 -group bandit6 -size 33c`
- `cat /var/lib/dpkg/info/bandit7.password`
- password = HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
- `ssh -p 2220 bandit7@bandit.labs.overthewire.org` + password

---

level 8 - searching for text in a file ("millionth")
- `cat data.txt | grep millionth`
- password = cvX2JJa4CFALtqS87jk27qwqGhBM9plV
- `ssh -p 2220 bandit8@bandit.labs.overthewire.org` + password

---

level 9 - searching for a line of text that only occurs once in a file 
- `cat data.txt | sort | uniq -u`
- password = UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
- `ssh -p 2220 bandit9@bandit.labs.overthewire.org` + password

--- 

level 10 - searching a binary file for a text that is preceded by several "="
- can't grep binary file so need `strings` command to find human-readable strings in the file 
- `strings data.txt | grep =`
- password = truKLdjsbJ5g7yyJ2X2R0o3a5HQJFuLk
- `ssh -p 2220 bandit10@bandit.labs.overthewire.org` + password

---

level 11 - 

