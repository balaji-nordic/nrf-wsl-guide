# Working with nRF DKs from WSL2 

This guide aims to help you (and the future me) get (re)started with working with nRF DKs from WSL2 🐧.

## Step 1: Attaching the DK to WSL2: 
- On Windwows, open up PowerShell with _Administrator privileges_ and enter the following command

  `usbipd wsl list`
- This will give you the BUS IDs of the DKs and other USB devices attached to your PC. Note the DK's BUS ID. It shows up something like this

``` 
BUSID  VID:PID    DEVICE                                                        STATE
2-2    1366:1055  JLink CDC UART Port (COM13), JLink CDC UART Port (COM11),...  Not attached
```

- The BUS ID for the DK listed above is `2-2`. 
- Attach it to WSL using the following command
  
  `usbipd wsl attach --busid={BUS-ID}`


:point_right: *Tip 1:* It is observed that on some machines, windows automatically disconnects the device ramdomly 🤷‍♀️. To workaround that, you may use one of the following methods.
  - Create a while loop on PowerShell (and leave it running)

  ```
  while (1)
  {
    usbipd wsl attach --busid 1-11
  }
  ```
  This bruteforces the attach thereby ensuring that your DK is always attached to WSL.
  - Use the `--auto-attach ` option. 👉 Note that this is only known to work on a single device. See https://github.com/dorssel/usbipd-win/issues/619


:point_right: *Tip 2:* When attaching the device, if you get a warning about your firewall blocking the connection, do the following from PowerShell (in administrator mode).

```
Set-NetFirewallProfile -DisabledInterfaceAliases "vEthernet (WSL)"
```

This brute-forces the attach thereby ensuring that your DK is always attached to WSL.

## Step 2: Setting UDEV rules

- In WSL, ensure that the following lines exist in `/etc/udev/rules.d/99-jlink.rules`
```
#
# Make sure that VCOM ports of J-Links can be opened with user rights
# We simply say that all devices from SEGGER which are in the "tty" domain are enumerated with normal user == R/W
#
SUBSYSTEM=="tty", ATTRS{idVendor}=="1366", MODE="0666", GROUP="dialout"
SUBSYSTEM=="tty", ATTRS{idVendor}=="c251", MODE="0666", GROUP="dialout"
SUBSYSTEM=="tty", ATTRS{idVendor}=="0d28", MODE="0666", GROUP="dialout"
#
```
- In WSL, ensure that that the `systemd-udevd` service is running. You can verify that by doing

  ```
  sudo service udev status
  ```
- If the service is not running, you can start it by doing

  ```
  sudo service udev start
  ```
  
- If the service was running, but you made changes to the `99-jlink.rules` file as mentioned above, then you will need to restart the `system-udev` by doing the following

  ```
  sudo service udev restart
  ```
  
👉 In some cases, you will also need to do  `sudo udevadm control --reload` after restarting `system-udev`.

This will enable you to fully take control of both programming and accessing the com ports of your DK. You can use the linux flavor of nRF Command Line tools to program from WSL and also use zephyr's `twister` to run on-target tests. 
Terminal applications like `minicom` will also be able to listen to the com ports exposed by the DK.

# Using windows version of nrfjprog from WSL

If 
- you are short on time or 
- if the above approach does not work or 
- if you are only interested in programming your device from WSL (you dont need to interact with the serial ports of the DK),
then you can do the following trick.

In WSL, 
- Create `/usr/local/bin/nrfjprog`
- Add the following lines in that file
``
#!/bin/bash
nrfjprog.exe $
``
- Add `/user/local/bin` to your PATH

Everytime you invoke `nrfjprog` from WSL, the windows version gets invoked and works just fine. Even commands using meta tools like `west flash` would work using this method.

Have fun 🥳

# Debugging from a WSL instance of VS COde (using nRFConnect for VS Code Extension)

- To be documented
