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

---

*Created by `holychowders` on 2025-07-10*<br>
*See <https://github.com/holychowders>*<br>
*See <https://github.com/holychowders/sadservers>*<br>
