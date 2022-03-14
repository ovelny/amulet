# Smartctl usage

No-nonsense guide to monitor the health of HDDs and SSDs with smartctl.

## Run and check tests

```bash
# Run a short test
sudo smartctl -t short /dev/sd[x]
# Run a long test
sudo smartctl -t long /dev/sd[x]
# Check progress of running test
sudo smartctl -a /dev/sd[x] | grep "of test"
# List test results
sudo smartctl -l selftest /dev/sd[x]
```

## Monitor health

```bash
# Check SMART reports
sudo smartctl -H /dev/sd[x]
# Check detailed info and all SMART attributes
sudo smartctl -a /dev/sd[x]
```

Suggested attributes that might indicate a failing HDD:

* SMART 5: Reallocated_Sector_Count
  * 1-4 keep an eye on it, more than 4 replace

* SMART 187: Reported_Uncorrect
  * 1 or more replace

* SMART 188: Command_Timeout
  * 1-13 keep an eye on it, more than 13 replace

* SMART 197: Current_Pending_Sector_Count
  * 1 or more replace

* SMART 198: Offline_Uncorrectable
  * 1 or more replace

Suggested attributes that might indicate a failing SSD:

* Check for Percent_Lifetime_Remain or similarly named attributes
* Check for SMART 5 increasing
* Check for SMART 180 decreasing

## References

* https://blog.programster.org/smartmontools-and-smartctl-cheatsheet
* https://superuser.com/questions/1171760/how-to-determine-how-dead-a-hdd-is-from-smartctl-report
* https://www.reddit.com/r/homelab/comments/kaaqma/percent_lifetime_remain_failing_now/
