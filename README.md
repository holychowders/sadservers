# SadServers Linux Troubleshooting Workthroughs and Write-Ups

<https://sadservers.com>

## Saint John (what is writing to this log file?)

<https://sadservers.com/scenario/saint-john>

### Description

> A developer created a testing program that is continuously writing to a log file /var/log/bad.log and filling up disk. You can check for example with `tail -f /var/log/bad.log`.
> This program is no longer needed. Find it and terminate it. Do not delete the log file.

### Solution

- `lsof /var/log/bad.log` to find processes using the bad log file
- `pkill badlog.py` to kill the process who was discovered using the log file

## Saskatoon (counting IPs)

<https://sadservers.com/scenario/saskatoon>

### Description

> There's a web server access log file at `/home/admin/access.log`. The file consists of one line per HTTP request, with the requester's IP address at the beginning of each line.
>
> Find what's the IP address that has the most requests in this file (there's no tie; the IP is unique). Write the solution into a file `/home/admin/highestip.txt`. For example, if your solution is "1.2.3.4", you can do `echo "1.2.3.4" > /home/admin/highestip.txt`

### Solutions

- Initial solution: `awk '{print $1}' access.log | sort | uniq -c | awk '{if ($1>200) print $1,$2}'`
  - `awk '{print $1}' access.log` to get the first column of `access.log` (IP addresses)
  - `sort` sorts the IPs so accesses from the same IP are in adjacent order for `uniq`
  - `uniq -c` counts adjacent unique IP accesses, and therefore provides an access count for each IP
  - `awk '{if ($1>200) print $1,$2}'` prints the access count of each unique IP address along with said IP address
- Latest solution: `cut -d ' ' -f1 access.log | sort | uniq -c | sort -n | tail -n 1`
    - `awk '{print $1}' access.log` was replaced with `cut -d ' ' -f1 access.log`, which is probably technically a bit faster than the `awk` solution, especially since I don't need awk interpreter overhead
    - `sort` and `uniq -c` are used the same as in the initial solution
    - `awk '{if ($1>200) print $1,$2}'` was replaced with `sort -n | tail -n 1`, which sorts the counts in descending order and takes the last, and therefore largest, count. This leaves us with the same output as with the initial solution, but without awk and a bunch of equality checks.

## Taipei (come a-knocking)

<https://sadservers.com/scenario/taipei>

### Description

> There is a web server on port :80 protected with [Port Knocking](https://en.wikipedia.org/wiki/Port_knocking). Find the one "knock" needed (sending a SYN to a single port, not a sequence) so you can `curl localhost`.

### Solution

- Attempt the blocked command: `curl -v localhost`, with `-v` for more info
- Assuming `nmap` is installed, since we know we can't attempt `root` access, and since we know we're just expected to knock with a SYN on the right port, we know we can probably just use a default `nmap` scan (`-sT` or "TCP connect scan") on `localhost`: `nmap localhost`
- The scan results show port 80 open, which means our knocking was probably successful (and, of course, that `nmap` is indeed installed)
- `curl localhost` now gets us a reply: `Who is there?`. Yes, it worked.
- However, I'd like to know which port knock opened up port 80...
  - After some testing on my local machine, I came up with this command:
    - `for port in {0..65535}; do nmap localhost -p $port >/dev/null; nmap localhost -p 80 | grep -qi 'open' && echo Port 80 unlocked after knocking on port $port && break; done;`
    - Which was terribly slow and didn't work at all (before I killed it). TODO: Maybe I'll look into it again later

## The Command Line Murders

<https://sadservers.com/scenario/command-line-murders>

### Description

> This is the [Command Line Murders](https://github.com/veltman/clmystery) with a small twist as in the solution is different
>
> Enter the name of the murderer in the file `/home/admin/mysolution`, for example `echo "John Smith" > ~/mysolution`
>
> Hints are at the base of the `/home/admin/clmystery` directory. Enjoy the investigation!

### Solution

- **NOTE:** The solution notes for this one will be sporadic
- The notes are all working from `~/clmystery/mystery/`
- `grep -i 'clue:' crimescene` per the instructions file
- People search for a female "Annabel":
  - Annabel Sun  F  26  Hart Place, line 40
    - Line 40 address search says: See interview #47246024
      - Interview says: This is not the New Zealand lady
  - Annabel Church  F  38  Buckingham Place, line 179
    - Line 179 address search says: See interview #699607
      - Interview says: Saw car leave: Blue Honda with plate starting with "L337" and ending with "9"
- Vehicle search for the blue Honda:
  - `grep -iA 5 'l337.*9' vehicles`
    ```
    ...
    License Plate L337DV9
    Make: Honda
    Color: Blue
    Owner: Joe Germuska
    Height: 6'2"
    Weight: 164 lbs
    --
    License Plate L3375A9
    Make: Honda
    Color: Blue
    Owner: Jeremy Bowers
    Height: 6'1"
    Weight: 204 lbs
    ...
    ```
- People search on owners of vehicles matching suspect description and suspect vehicle description:
  - `grep -iE 'joe.*germuska|jeremy.*bowers' people`
    - Joe Germuska  M  65  Plainfield Street, line 275
    - Jeremy Bowers  M  34  Dunstable Road, line 284
- Address search on possible suspects:
  - `sed -n '275p' streets/Plainfield_Street` says: See interview #29741223
    - Interview says: Not available to interview
  - `sed -n '284p' streets/Dunstable_Road` says: See interview #9620713
    - Interview says: "Home appears empty. After questioning neighbors, appears that the occupant may have left for a trip recently."
- Memberships search on two best suspects (based on wallet found supposedly dropped by suspect):
  - `grep --color -i 'joe.*germuska' memberships/Rotary_Club memberships/Terminal_City_Library memberships/Delta_SkyMiles memberships/Museum_of_Bash_History`
    - Joe is in member listings for all 4 clubs whose cards were discovered in the wallet left at the crime scene
  - `grep --color -i 'jeremy.*bowers' memberships/Rotary_Club memberships/Terminal_City_Library memberships/Delta_SkyMiles memberships/Museum_of_Bash_History`
    - Jeremy is in member listings for just 3 clubs
- Current best guess of Joe Germuska ends up being verified as the solution just before the timer runs out

---

*Created by `holychowders` on 2025-07-10*<br>
*See <https://github.com/holychowders>*<br>
*See <https://github.com/holychowders/sadservers>*<br>
