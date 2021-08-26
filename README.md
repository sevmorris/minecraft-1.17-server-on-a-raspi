
## Install Java 16 and Spigot Minecraft server 1.17 on a Raspberry Pi 4
This is a quick and somewhat dirty writeup to document the steps I was able to do (and repeat) to get a Spigot v1.17 server running on a RasPi 4 (which is in an enclosure with a fan and heatsinks).
I plan to prettify it a bit later, but it's otherwise complete. I'm relatively new to Linux, so change as you see fit if it makes sense.

Much of the following was sourced from [this excellent and comprehensive post](https://lemire.me/blog/2016/04/02/setting-up-a-robust-minecraft-server-on-a-raspberry-pi/) from [Daniel Lemire](https://github.com/lemire)'s blog, with a big assist from [this Reddit post](https://www.reddit.com/r/raspberry_pi/comments/o6wiuc/how_to_install_minecraft_117_server_on_raspberry/) by [Thamesmonster](https://www.reddit.com/user/Thamesmonster/).
<br><br>

**UPDATE APT INDEX**
<br><br>

```
sudo apt update
```
<br><br>

**INSTALL GNU SCREEN**
<br><br>

```
sudo apt install netatalk screen avahi-daemon
```
<br>

**INSTALL JAVA 16**
<br><br>

From your Home folder make a bin dir and go to that folder.

```
mkdir bin && cd bin
```
<br>

Go [here](https://adoptopenjdk.net/releases.html?variant=openjdk16&jvmVariant=hotspot) and select version **OpenJDK 16**.

Select **Linux** in the OS tab, and select **arm32** for the architecture.

Copy the **JRE** link (not the JDK) and download it to the current directory.

```
wget <link you just copied>
```
<br>

List the folder contents, copy the name of the downloaded file, and unpack it.

```
tar -xvf <name of the file you just copied>.tar.gz
```
<br>

Use `ls` to see the new folder name and go to the `bin` folder inside that folder. For example, this is my current folder:

```
cd jdk-16.0.1+9-jre/bin
```
<br>

Enter the command `pwd` and copy the path. For example, mine is: `/home/pi/bin/jdk-16.0.1+9-jre/bin`

To add this folder to your $PATH:

```
nano ~/.bashrc
```

Add the following new line:

`export PATH="<path you just copied>:$PATH"`
<br>

Save and exit nano and reload the .bashrc file.

```
source ~/.bashrc
```
<br>

**DOWNLOAD AND BUILD THE MINECRAFT SERVER**
<br><br>

Go ~/:

```
~
```
<br>

Make a directory for the server and go to that folder.

```
mkdir minecraft && cd minecraft
```
<br>

Download the server.

```
wget https://hub.spigotmc.org/jenkins/job/BuildTools/lastSuccessfulBuild/artifact/target/BuildTools.jar
```
<br>

Build the server.

```
java -jar BuildTools.jar
```
<br>

In order to be able to start the Minecraft server, you must first accept the license terms. To do this, execute this command:

```
echo "eula = true" > eula.txt
```
<br>

 This creates a new file which indicates that you accepted these license terms.
<br><br>

Get the name/version number of the server file.

```
ls spigot*.jar
```
<br>

Make sure the server name/version is current as found in the previous step and start the server.

```
java -jar -Xms1008M -Xmx2048M spigot-1.17.1.jar nogui
```
<br>

**CREATE A SERVER SCRIPT AND RUN IT AUTOMATICALLY AT BOOT**
<br><br>

Create a new file.

```
nano minecraft.sh
```
<br>

Add the following lines, making sure to use the current server version and correct user (assuming the default user pi here).

```
if ! screen -list | grep -q "minecraft"; then
  cd /home/pi/minecraft
  screen -S minecraft -d -m java -jar  -Xms1008M -Xmx2048M spigot-1.17.1.jar nogui
fi
```
<br>

Confirm the file was formatted correctly.

```
wc -l minecraft.sh
```

If correctly formatted, the shell will return the following, indicating the file contains 4 lines:

`$ 4 minecraft.sh`
<br><br>

Check the syntax of the script. This should return no errors.

```
bash -n minecraft.sh
```
<br>

Make the script executable.

```
chmod +x minecraft.sh
```
<br>

Enable the server to start automatically

```
sudo nano /etc/rc.local
```

Enter the following line right before the exit command:

`su -l pi -c /home/pi/minecraft/minecraft.sh`
<br><br>

**Start the server**

```
./minecraft.sh
```
<br>

Access the server console.

```
screen -r minecraft
```

Disconnect from the console when finished by pressing the following keys:

`ctrl-a  d`
<br><br>

Move temp files to memory.

```
sudo nano /etc/fstab
```

It should look something like this:

```
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p6  /boot           vfat    defaults          0       2
/dev/mmcblk0p7  /               ext4    defaults,noatime  0       1
```

Append the following as a new line:

`tmpfs /tmp tmpfs nodev,nosuid,size=1M 0 0`
<br><br>

Open the the console and stop the server, then reboot the RasPi.

The Minecraft server _should_ start automatically on boot.
