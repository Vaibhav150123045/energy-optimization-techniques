### Monitoring stats using zabbix in linux cli

To track the CPU load using your locally installed Zabbix agent, you can either do a **one-time check** to see the current numbers or set up a **continuous monitor** (like a DIY dashboard in your terminal).

Since you are running without `sudo`, the agent is restricted to what your user profile can see, but luckily, CPU load is globally readable on Linux.

---

### 1. The "Right Now" Check
Use the agent's test flag (`-t`) to pull the current Load Average. In Linux, "Load" is usually measured over 1-minute, 5-minute, and 15-minute intervals.

**To see the 1-minute average:**
```bash
$HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf -t system.cpu.load[all,avg1]
```

**To see the 5-minute average (usually more stable):**
```bash
$HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf -t system.cpu.load[all,avg5]
```

---

### 2. Live Terminal Monitoring (The "Watch" Command)
If you want to keep a small window open to watch the CPU load change in real-time (updating every 2 seconds), use the Linux `watch` command:

```bash
watch "$HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf -t system.cpu.load[all,avg1]"
```
*Press **Ctrl+C** to stop watching.*

---

### 3. Understanding the Output
When Zabbix returns a value like `[float|1.450000]`, here is what that number actually means relative to your machine:

* **0.0 - 1.0:** The CPU is handling tasks easily (on a single-core system).
* **1.0:** The CPU is at exactly 100% capacity.
* **Above 1.0:** Processes are waiting in a queue to be processed (the system is "lagging").

> **Note:** If you have a multi-core CPU (e.g., 4 cores), a load of **4.0** is technically 100% utilization. You can check your core count with `nproc`.



---

### 4. Tracking via Zabbix Get (Internal Network)
If the agent is running as a service, it's better to use `zabbix_get`. This simulates exactly how a Zabbix Server would "ask" your machine for data:

```bash
$HOME/zabbix_install/bin/zabbix_get -s 127.0.0.1 -p 10050 -k system.cpu.load[all,avg1]
```

---

### 5. Advanced: CPU Utilization %
"Load" and "Utilization" are different. Load is about the process queue; Utilization is about the percentage of time the CPU was busy. To see the **Idle percentage** (100 minus this is your usage):

```bash
$HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf -t system.cpu.util[,idle]
```

**Would you like to set up a "Trigger" alias so you can just type `cpu_check` to see these stats instantly?**
