clamav
======

Ansible role which helps to install and configure ClamAV.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: Example of how to install ClamAV in the default configuration
  hosts: all
  roles:
    - clamav

- name: Example of how change the configuration
  hosts: all
  vars:
    # Disable mail scanning for Clamd
    clamav_clamd_scan_mail: "no"
    # Add new options into the Clamd config file
    clamav_clamd_config__custom:
      Debug: "yes"
      ScanXMLDOCS: "yes"
    # Disable syslog for the FreshClam
    clamav_freshclam_log_syslog: "no"
    # Add new options into the FreshClam config file
    clamav_freshclam_config__custom:
      LogVerbose: "yes"
      LogRotate: "yes"
  roles:
    - clamav
```


Role variables
--------------

```
# Whether to install EPEL YUM repo
clamav_epel_install: "{{ yumrepo_epel_install | default(true) }}"

# EPEL YUM repo URL
clamav_epel_yumrepo_url: "{{ yumrepo_epel_url | default('https://dl.fedoraproject.org/pub/epel/$releasever/$basearch/') }}"

# EPEL YUM repo GPG key
clamav_epel_yumrepo_gpgkey: "{{ yumrepo_epel_gpgkey | default('https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-$releasever') }}"

# Additional EPEL YUM repo params
clamav_epel_yumrepo_params: "{{ yumrepo_epel_params | default({}) }}"

# Package to be installed (explicit version can be specified here)
clamav_pkgs: "{{
    ['clamav', 'clamav-update', 'clamav-scanner-systemd']
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    ['clamd'] }}"

# Clamav service name
clamav_service: "{{
    'clamd@scan'
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    'clamd' }}"


# Max age of the ClamAV DB before update (in days)
clamav_db_age: 7

# Whether to run ClamAV DB update if it's too old
clamav_db_update_run: yes

# Force ClamAV to update the DB
clamav_db_update_force: no

# Default ClamAV DB update command options
clamav_db_update_command_options__default:
  - --quiet
  - --no-warnings

# Custom ClamAV DB update command options
clamav_db_update_command_options__custom: []

# Final ClamAV DB update command options
clamav_db_update_command_options: "{{
  clamav_db_update_command_options__default +
  clamav_db_update_command_options__custom }}"

# ClamAV DB update command
clamav_db_update_command: /usr/bin/freshclam {{ clamav_db_update_command_options | join(' ') }}


# Path to the ClamScan file
clamav_scan_log_file: "{{ clamav_log_dir }}/clamscan_{{ clamav_scan_cron_time }}.log"

# Root path where to start to scan
clamav_scan_root: /

# Default ClamAV scan command options
clamav_scan_command_options__default:
  - --config-file={{ clamav_clamd_file }}
  - --log={{ clamav_scan_log_file }}
  - --fdpass
  - --infected

# Default ClamAV scan command options
clamav_scan_command_options__custom: []

# Default ClamAV scan command options
clamav_scan_command_options: "{{
  clamav_scan_command_options__default +
  clamav_scan_command_options__custom }}"

# ClamAV scan command
clamav_scan_command: /usr/bin/clamdscan {{ clamav_scan_command_options | join(' ') }} {{ clamav_scan_root }}

# Whether to setup Clamd Scan cron job
clamav_scan_cron: yes

# How often to run the Clamd Scan [hourly|daily|weekly]
clamav_scan_cron_time: daily

# Random delay before the Clamd Scan runs from the cron job
clamav_scan_cron_job_delay: 45

# Command to run as a cron job for the Clamd Scan
clamav_scan_cron_job: sleep $[RANDOM\%{{ clamav_scan_cron_job_delay }}]m ; {{ clamav_scan_command }}


# Whether to setup FreshClam cron job
clamav_freshclam_cron: "{{ clamav_scan_cron }}"

# How often to run the Clamd Scan [hourly|daily|weekly]
clamav_freshclam_cron_time: "{{ clamav_scan_cron_time }}"

# Random delay before the Clamd Scan runs from the cron job
clamav_freshclam_cron_job_delay: "{{ clamav_scan_cron_job_delay }}"

# Command to run as a cron job for the Clamd Scan
clamav_freshclam_cron_job: sleep $[RANDOM\%{{ clamav_freshclam_cron_job_delay }}]m ; {{ clamav_db_update_command }}


# ClamAV user and group
clamav_user: "{{
    'clamscan'
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    'clam' }}"
clamav_group: "{{ clamav_user }}"


# Log dir permissions (if not /var/log)
clamav_log_dir_mode: 0770

# Log file permissions
clamav_log_file_mode: 0660

# DB dir permissions
clamav_db_dir_mode: 0770


# ClamAV directories
clamav_log_dir: "{{
    '/var/log'
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    '/var/log/clamav' }}"
clamav_run_dir: "{{
    '/var/run/clamd.scan'
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    '/var/run/clamav' }}"


# Location of the Clamd config
clamav_clamd_file: "{{
    '/etc/clamd.d/scan.conf'
  if ansible_os_family == 'RedHat' and ansible_distribution_major_version | int >= 7
  else
    '/etc/clamd.conf' }}"

# Values of the default options in the Clamd config
clamav_clamd_log_file: "{{ clamav_log_dir ~ '/clamd.log' }}"
clamav_clamd_log_file_max_size: 0
clamav_clamd_log_time: "yes"
clamav_clamd_log_syslog: "yes"
clamav_clamd_pid_file: "{{ clamav_run_dir ~ '/clamd.pid' }}"
clamav_clamd_temporary_directory: /var/tmp
clamav_clamd_database_directory: /var/lib/clamav
clamav_clamd_local_socket: "{{ clamav_run_dir ~ '/clamd.sock' }}"
clamav_clamd_fix_stale_socket: "yes"
clamav_clamd_tcp_socket: 3310
clamav_clamd_tcp_addr: 127.0.0.1
clamav_clamd_max_connection_queue_length: 30
clamav_clamd_max_threads: 50
clamav_clamd_read_timeout: 300
clamav_clamd_user: "{{ clamav_user }}"
clamav_clamd_allow_supplementary_groups: "yes"
clamav_clamd_scan_pe: "yes"
clamav_clamd_scan_elf: "yes"
clamav_clamd_detect_broken_executables: "yes"
clamav_clamd_scan_ole2: "yes"
clamav_clamd_scan_mail: "yes"
clamav_clamd_scan_archive: "yes"
clamav_clamd_archive_block_encrypted: "no"
clamav_clamd_exclude_path__default:
  - ^/(proc|sys|dev)/
clamav_clamd_exclude_path__custom: []
clamav_clamd_exclude_path: "{{
  clamav_clamd_exclude_path__default +
  clamav_clamd_exclude_path__custom }}"

# Default Clamd config
clamav_clamd_config__default:
  LogFile: "{{ clamav_clamd_log_file }}"
  LogFileMaxSize: "{{ clamav_clamd_log_file_max_size }}"
  LogTime: "{{ clamav_clamd_log_time }}"
  LogSyslog: "{{ clamav_clamd_log_syslog }}"
  PidFile: "{{ clamav_clamd_pid_file }}"
  TemporaryDirectory: "{{ clamav_clamd_temporary_directory }}"
  DatabaseDirectory: "{{ clamav_clamd_database_directory }}"
  LocalSocket: "{{ clamav_clamd_local_socket }}"
  FixStaleSocket: "{{ clamav_clamd_fix_stale_socket }}"
  TCPSocket: "{{ clamav_clamd_tcp_socket }}"
  TCPAddr: "{{ clamav_clamd_tcp_addr }}"
  MaxConnectionQueueLength: "{{ clamav_clamd_max_connection_queue_length }}"
  MaxThreads: "{{ clamav_clamd_max_threads }}"
  ReadTimeout: "{{ clamav_clamd_read_timeout }}"
  User: "{{ clamav_clamd_user }}"
  AllowSupplementaryGroups: "{{ clamav_clamd_allow_supplementary_groups }}"
  ScanPE: "{{ clamav_clamd_scan_pe }}"
  ScanELF: "{{ clamav_clamd_scan_elf }}"
  DetectBrokenExecutables: "{{ clamav_clamd_detect_broken_executables }}"
  ScanOLE2: "{{ clamav_clamd_scan_ole2 }}"
  ScanMail: "{{ clamav_clamd_scan_mail }}"
  ScanArchive: "{{ clamav_clamd_scan_archive }}"
  ArchiveBlockEncrypted: "{{ clamav_clamd_archive_block_encrypted }}"
  ExcludePath: "{{ clamav_clamd_exclude_path }}"

# Custom Clamd config
clamav_clamd_config__custom: {}

# Final Clamd config
clamav_clamd_config: "{{
  clamav_clamd_config__default.update(clamav_clamd_config__custom) }}{{
  clamav_clamd_config__default }}"


# Location of the FreshClam config
clamav_freshclam_file: /etc/freshclam.conf

# Values of options in the FreshClam config
clamav_freshclam_update_log_file: "{{ clamav_log_dir ~ '/freshclam.log' }}"
clamav_freshclam_log_syslog: "{{ clamav_clamd_log_syslog }}"
clamav_freshclam_database_directory: "{{ clamav_clamd_database_directory }}"
clamav_freshclam_database_owner: "{{ clamav_user }}"
clamav_freshclam_database_mirror:
  - database.clamav.net
  - database.clamav.net

# Default FreshClam config
clamav_freshclam_config__default:
  UpdateLogFile: "{{ clamav_freshclam_update_log_file }}"
  LogSyslog: "{{ clamav_freshclam_log_syslog }}"
  DatabaseDirectory: "{{ clamav_freshclam_database_directory }}"
  DatabaseOwner: "{{ clamav_freshclam_database_owner }}"
  DatabaseMirror: "{{ clamav_freshclam_database_mirror }}"

# Custom FreshClam config
clamav_freshclam_config__custom: {}

# Final FreshClam config
clamav_freshclam_config: "{{
  clamav_freshclam_config__default.update(clamav_freshclam_config__custom) }}{{
  clamav_freshclam_config__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)


License
-------

MIT


Author
------

Jiri Tyr
