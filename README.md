Feather is a tarsnap backup scheduler that performs and maintains a set of
backups as defined by a yaml configuration file.

### Features
- Dynamic scheduling
- Keep an arbitrary number of backups of each schedule type
- Restrict schedules based on time of day
- Restrict feather run to a certain amount of wall time (max_runtime)
- Multiple backup paths per tarsnap
- Multiple exclude list per tarsnap

### Requirements
- Python 2
- pyyaml

### Usage
Feather is designed to be run from cron like this:

 */5 * * * * /usr/local/bin/feather /usr/local/etc/feather.yaml

### Configuration
The best way to understand feather is to read an example configuration file:

```yaml
# Global paths, if different from default
cachedir: /usr/home/drue/tarsnap/cachedir
keyfile: /usr/home/drue/tarsnap/tarsnap.key
binpath: /usr/local/bin/

# perform a checkpoint every checkpoint_bytes, don't cross filesystems
backup_args: "--one-file-system --checkpoint-bytes 104857600"

# Kill the script after N seconds.
max_runtime: 3600

# Define the schedule
#
# period: Seconds. A backup is taken every period.
# always_keep: Number of backups to keep of a particular period, before
#              pruning old backups. Backups younger than now()-period are
#              never removed.
# implies:  Include another defined schedule. i.e. if WEEKLY implies
#           MONTHLY, and you ask for WEEKLY backups, you will get WEEKLY
#           and MONTHLY.
# before/after: Restrict running to a certain time of day (UTC)
#
schedule:
  - monthly:
    - period: 2592000 # 30 days
    - always_keep: 12
    - before: "0600"
  - weekly:
    - period: 604800 # 7 days
    - always_keep: 6
    - after: "0200"
    - before: "0600"
    - implies: monthly
  - daily:
    - period: 86400 # 1 day
    - always_keep: 14
    - after: "0200"
    - before: "0600"
    - implies: weekly
  - hourly:
    - period: 3600
    - always_keep: 24
    - implies: daily
  - realtime:
    - period: 900
    - always_keep: 10
    - implies: hourly

# Define individual backups
# Path can be a directory or a file.
# Path can be a list or a single item.
backups:
  - _usr_local:
    - schedule: daily
    - path: /usr/local
    - exclude: /usr/local/bin
  - _etc:
    - schedule: realtime
    - path: /etc
  - music:
    - schedule: monthly
    - workingpath: /home
    - path: 
      - joe/music
      - bob/music
    - exclude:
      - joe/music/bieber # Not paying to back this up
      - bob/music/backstreet_boys # not paying to store this either

```
