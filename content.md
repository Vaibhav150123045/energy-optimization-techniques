1. Mapping of Zabbix key to information
2. Thermal zone 0, 1, 2, 3, 4
3. rapl:0, rapl:0:1, rapl:0:1, rapl:0:2 


```cat custom_metrics.conf 
UserParameter=system.energy.usage,cat /sys/class/powercap/intel-rapl:0/energy_uj
UserParameter=cpu.energy.usage,cat /sys/class/powercap/intel-rapl:0:0/energy_uj
UserParameter=gpu.energy.usage,cat /sys/class/powercap/intel-rapl:0:1/energy_uj
UserParameter=dram.energy.usage,cat /sys/class/powercap/intel-rapl:0:2/energy_uj

UserParameter=system.power.consumption,cat /sys/class/powercap/intel-rapl:0/energy_uj
UserParameter=cpu.power.consumption,cat /sys/class/powercap/intel-rapl:0:0/energy_uj
UserParameter=gpu.power.consumption,cat /sys/class/powercap/intel-rapl:0:1/energy_uj
UserParameter=dram.power.consumption,cat /sys/class/powercap/intel-rapl:0:2/energy_uj

UserParameter=cpu0.freq,cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
UserParameter=cpu1.freq,cat /sys/devices/system/cpu/cpu1/cpufreq/scaling_cur_freq
UserParameter=cpu2.freq,cat /sys/devices/system/cpu/cpu2/cpufreq/scaling_cur_freq
UserParameter=cpu3.freq,cat /sys/devices/system/cpu/cpu3/cpufreq/scaling_cur_freq

UserParameter=temp.zone0,cat /sys/class/thermal/thermal_zone0/temp
UserParameter=temp.zone1,cat /sys/class/thermal/thermal_zone1/temp
UserParameter=temp.zone2,cat /sys/class/thermal/thermal_zone2/temp
UserParameter=temp.zone3,cat /sys/class/thermal/thermal_zone3/temp
UserParameter=temp.zone4,cat /sys/class/thermal/thermal_zone4/temp

UserParameter=cpu.usage.total, bash /home/scs/script.sh
```

````cat zabbix_agentd.conf
EnableRemoteCommands=1

DebugLevel=2

LogFile=/var/log/zabbix-agent/zabbix_agentd.log
LogFileSize=0

Timeout=15

PidFile=/var/run/zabbix/zabbix_agentd.pid

Server=monitor
ServerActive=monitor:10051

Include=/etc/zabbix/zabbix_agentd.conf.d/custom_metrics.conf
````



pidstat -urd -h -T ALL 5 > file_path.txt

sudo chef-client --once


Reload the daemon (tell systemd there is a new file):
sudo systemctl daemon-reload

Enable it (so it starts on boot):
sudo systemctl enable pidstat-monitor.service

Start it:
sudo systemctl start pidstat-monitor.service
