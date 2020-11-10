# Ninja Skills CTF

> Practise your Linux skills and complete the challenges.

<br>

Flag 1 - files owned by best-group group
- `find / -group best-group 2>/dev/null`
- files = D8B3 v2Vb

---

Flag 2 - which files contain an IP address
- ...
- ...

---

Flag 3 - 
- ...
- ...

---

Flag 4 - 
- ...
- ...

---

Flag 5 - finding file whose owner ID is 505
- `find / -uid 502 2>/dev/null`
- file = X1Uy

---

Flag 6 - which file is executable by everyone 
- `find / -perm /o+x -type f ! -user root 2>/dev/null`
- flag = 8V2L

---


