OpenWrt Collectd
================

This role configures [collectd](https://www.collectd.org/) on [OpenWrt](https://www.openwrt.org/) targets.

Requirements
------------

This role requires the `community.general` collection.

Moreover, it requires a working [Python](https://www.python.org/) installation on the target system or [gekmihesg's Ansible library for OpenWrt](https://github.com/gekmihesg/ansible-openwrt) on the Ansible controller.

Role Variables
--------------

* `openwrt_collectd_config`  
  A dictionary of global configuration options for collectd.
  Dictionary keys are options names and dictionary values the respective configuration values.
  In order to translate collectd configuration syntax into YAML, use the following:
  * To repeat an option multiple times, make the value a list.  
  * For a configuration block, use the block type and optionally name as the dictionary key and set the value to a dictionary of the settings in that block.  
  * To use multiple unnamed blocks of the same type, use an arbitrary, unique name in `()`, e.g. `Include (1)`.
  The name in `()` will not be added to the collectd configuration and is only used to make the YAML key unique.  
  Refer to the [collectd documentation](https://www.collectd.org/documentation/manpages/collectd.conf.html) for valid options and their meaning.
  Note that OpenWrt does not package all collectd plugins and adds a few new plugins and options.
  There is no need to use `LoadPlugin` as this role automatically loads all configured plugins.
  Also, don't use the `PreCacheChain` and `PostCacheChain` options.
  See `openwrt_collectd_precachechains` and `openwrt_collectd_postcachechains` instead.
* `openwrt_collectd_plugin_config`  
  A dictionary of collectd plugins to use and their configuration.
  Dictionary keys are plugin names and dictionary values are in turn dictionary describing the plugin configuration.
  Refer to `openwrt_collectd_config` for how to map collectd configuration syntax to YAML.
  To use a plugin that does not require any configuration, add it to `openwrt_collectd_plugin_config` with an empty value.
  To change how a plugin is loaded, add the desired options as a dictionary under the `LoadPlugin` key.  
  This role comes with a few default setting for some plugins.
  In order to unset the defaults without setting a value, set them to an empty value.  
  Note that this role handles the setting `RRATimespan` (of the `rrdtool` plugin) specially.
  Time values may be given in human-readable form (e.g. `1hour`) instead of seconds.
  If `openwrt_collectd_luci` is enabled, the human-readable values will be used in the LuCI web UI.
* `openwrt_collectd_precachechains`, `openwrt_collectd_postcachechains`  
  A dictionary of filter chains to be added to the `PreCacheChain` and `PostCacheChain`, respectively.
  Dictionary keys are custom chain names and dictionary values are the respective chain configuration, i.e. rules and targets.
  Refer to `openwrt_collectd_config` for how to map collectd configuration syntax to YAML.
* `openwrt_collectd_rrd_backup_file`  
  Path to a file where a backup of collectd's RRD database is stored.
  When set, a system service is set up that restores the backup before starting collectd and creates a backup after stopping collectd.
  This allows writing RRD files to a tmpfs location (to reduce flash writes) but nonetheless preserving the data on reboots.
  Defaults to `/etc/collectd/rrdbackup.tgz` if the `rrdtool` plugin is enabled and the `DataDir` option is not set to a non-volatile path and otherwise to `false`.
  Set to `false` or an empty string to disable RRD backups.
* `openwrt_collectd_rrd_backup_time`  
  This option allows to periodically backup the RRD database while collectd is running.
  Set it to a time specification as understood by cron to control when to write a backup.
  Unset by default.
* `openwrt_collectd_luci`  
  Whether to install the LuCI integration for collectd.
  Defaults to `false`.

Dependencies
------------

This role does not depend on any specific roles.

Example Configuration
---------------------

The following is a short example for some of the configuration options this role provides:

```yaml
openwrt_collectd_config:
  Interval: 30
  ReadThreads: 2
openwrt_collectd_plugin_config:
  chrony:
  cpu:
    ReportByCpu: true
  df:
    LoadPlugin:
      Interval: 600
    MountPoint:
    FSType:
      - ext4
  rrdtool:
    RRATimespan:
      - 1hour
      - 1day
      - 1week
      - 1month
      - 1year
openwrt_collectd_precachechains:
  PreCache:
    Rule Drop Stratum:
      Match regex:
        Plugin: '^chrony$'
        Type: '^clock_stratum$'
      Target: stop
openwrt_collectd_rrd_backup_time: '0 */6 * * *'
```

License
-------

MIT
