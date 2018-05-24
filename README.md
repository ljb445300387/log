# log.
netstat -anpt | grep redis | awk -F" " {'print $5'} | awk -F":" {'print $1'}| sort -rh | uniq -c
