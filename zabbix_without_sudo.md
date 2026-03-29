Installing Zabbix without `sudo` (root) access is possible, but it is a bit more manual because you cannot use the standard package managers (`apt`, `dnf`, `yum`) or system directories like `/etc/` and `/usr/`.

You will need to **compile it from source** and install it into a directory you own, such as your home folder (`/home/user/zabbix`).

### Prerequisites
You must have the following build tools already installed on the machine (usually standard, but check with `gcc --version` and `make --version`):
* **C Compiler** (gcc)
* **Make**
* **PCRE Development files** (often required for the agent)

---

### Step-by-Step Installation (Agent Only)
*Most users without sudo only need the **Zabbix Agent** to monitor their local environment. Installing a full Server/Database without root is significantly more complex due to database dependencies.*

#### 1. Download the Source Code
Go to the [Zabbix Official Download page](https://www.zabbix.com/download_sources) or use `wget`:
```bash
wget https://cdn.zabbix.com/zabbix/sources/stable/6.0/zabbix-6.0.26.tar.gz
tar -zxvf zabbix-6.0.26.tar.gz
cd zabbix-6.0.26
```

#### 2. Configure the Build
The `--prefix` flag is the "magic" part. It tells Zabbix to install everything into a folder you control.
```bash
mkdir -p $HOME/zabbix_install
./configure --enable-agent --prefix=$HOME/zabbix_install
```


##### Error here - 

```
checking size of void *... 8

configure: using old pcre library by default

checking for pkg-config... no

configure: WARNING: proceeding without pkg-config

checking for libpcre support... no

configure: error: unable to use libpcre (libpcre check failed)
```

##### Resolution:
The short answer is **no, it will not run**—because it hasn't actually been installed yet.

The `configure` script is a "pre-flight check." When it hits an **error**, it stops dead in its tracks. It never created the "Makefile" needed to compile the code, so if you try to run `make install` right now, you’ll likely get an error saying `no targets specified and no makefile found`.

### Why did this happen?
Zabbix uses the **PCRE** (Perl Compatible Regular Expressions) library to handle things like regular expressions in log monitoring and item keys. The error `unable to use libpcre` means the compiler looked for that library on your system and couldn't find it.

Since you don't have `sudo` access, you can't just run `apt install libpcre3-dev`. You have two choices to move forward:

---

### Option 1: Disable PCRE (The "Quick" Way)
If you don't need advanced regular expression features (like complex log filtering), you can try to bypass this check. 

Try running the configure command with the `--without-libpcre` or `--with-libpcre2=no` flag (depending on your Zabbix version):

```bash
./configure --enable-agent --prefix=$HOME/zabbix_install --with-libpcre=no
```
*Note: If the script says it **must** have PCRE to function, this will also fail, and you'll have to move to Option 2.*

---

### Option 2: Install PCRE in your Home Directory (The "Proper" Way)
Since you are already in "manual mode," you can download and "install" the PCRE library into your home folder just like you're trying to do with Zabbix.

1. **Download and Build PCRE:**
   ```bash
   wget https://sourceforge.net/projects/pcre/files/pcre/8.45/pcre-8.45.tar.gz
   tar -xvf pcre-8.45.tar.gz
   cd pcre-8.45
   ./configure --prefix=$HOME/pcre_install
   make && make install
   cd ..
   ```

2. **Tell Zabbix where to find it:**
   Now, go back to your Zabbix folder and point the `configure` script to that new folder:
   ```bash
   cd zabbix-6.0.26
   ./configure --enable-agent --prefix=$HOME/zabbix_install --with-libpcre=$HOME/pcre_install
   ```



---

### How to tell if it actually worked
You will know the `configure` step was successful when you see a summary table at the end of the logs that looks like this:

```text
Configuration:
  Detected OS:           linux-gnu
  Install path:          /home/youruser/zabbix_install
  Compilation flags:     ...
  Enable agent:          yes
```

**Once you see that summary, you can proceed to run `make install`. Should I help you with the commands to link the library path so the agent can find PCRE when it starts up?**




#### 3. Compile and Install
```bash
make install
```
After this completes, your binaries will be located in `$HOME/zabbix_install/sbin/`.

---

### 4. Configuration and Running
Since you aren't root, you need to modify the configuration file to point to paths you have permission to write to.

1.  **Edit the config:**
    ```bash
    nano $HOME/zabbix_install/etc/zabbix_agentd.conf
    ```
2.  **Change these lines** to point to your home directory (otherwise it will try to write to `/var/log` and fail):
    * `LogFile=/home/yourusername/zabbix_install/zabbix_agentd.log`
    * `PidFile=/home/yourusername/zabbix_install/zabbix_agentd.pid`
3.  **Start the Agent:**
    ```bash
    $HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf
    ```

---

### Key Limitations
* **Privileged Ports:** You cannot run Zabbix on ports below **1024**. If you are setting up a server, you must change the default port (10051) to something like 10052.
* **Autostart:** Since you don't have `sudo`, you cannot add Zabbix to `systemctl`. You will need to add your start command to your `.bashrc` or a `crontab` (@reboot) to keep it running.
* **Dependencies:** If the machine is missing libraries like `libevent` or `pcre`, and you don't have `sudo` to install them, you would have to compile those from source as well—which can become a "dependency rabbit hole."



**Are you trying to install the full Zabbix Server (which requires a database like MySQL/PostgreSQL) or just the Agent to monitor this specific machine?**





## Restarting Zabbix

Because you installed Zabbix **manually in your home directory** (without `sudo`), you cannot use the standard `systemctl restart zabbix-agent` command. Instead, you have to manually stop the existing process and start a new one.

Here is the "no-sudo" way to restart your service:

### 1. Kill the existing process
First, find the Process ID (PID) of the running agent and kill it.

**The "Clean" way (using the PID file):**
If you configured a `PidFile` in your config (as we did in the previous steps):
```bash
kill $(cat $HOME/zabbix_install/zabbix_agentd.pid)
```

**The "Force" way (if the PID file is missing):**
```bash
pkill -u $(whoami) zabbix_agentd
```

---

### 2. Verify it has stopped
Run this to make sure no Zabbix processes are left hanging:
```bash
ps aux | grep zabbix_agentd
```
If you only see your `grep` command in the output, the agent is successfully stopped.

---

### 3. Start the Agent again
Now, launch the binary again, pointing it to your custom configuration file. 

**Crucial Note:** Since you rely on a local library (PCRE), you must ensure your `LD_LIBRARY_PATH` is active in this terminal session.

```bash
export LD_LIBRARY_PATH=$HOME/pcre_install/lib:$LD_LIBRARY_PATH
$HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf
```

---

### 4. Confirm it’s back online
Check the last few lines of your log file to ensure it started without errors (like port conflicts):
```bash
tail -n 20 $HOME/zabbix_install/zabbix_agentd.log
```



---

### Pro-Tip: Create a "Restart Alias"
Since typing those long paths is a headache, you can add a shortcut to your `~/.bashrc` file so you can restart it with one word.

1. Run: `nano ~/.bashrc`
2. Add this line at the bottom:
   ```bash
   alias zbx_restart='pkill -u $(whoami) zabbix_agentd && sleep 1 && export LD_LIBRARY_PATH=$HOME/pcre_install/lib:$LD_LIBRARY_PATH && $HOME/zabbix_install/sbin/zabbix_agentd -c $HOME/zabbix_install/etc/zabbix_agentd.conf && echo "Zabbix Agent Restarted"'
   ```
3. Run: `source ~/.bashrc`

Now, whenever you change your `UserParameter` for that **CPU Energy tracking**, you can just type **`zbx_restart`**.

**Did the agent start back up successfully, or did you get a "permission denied" error on the PID file?**

