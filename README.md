# Increasing-XMRig-Performance-on-Ubuntu

## run the approriate build
There are various techniques for getting the bits.

    https://xmrig.com/docs/miner/build/ubuntu

## Official Performance Articles and Scripts
Here is all of them in one place. As we go through the guide, we also give each link as needed.

    https://xmrig.com/docs/miner/randomx-optimization-guide
    https://xmrig.com/docs/miner/hugepages
    https://github.com/xmrig/xmrig/blob/dev/scripts/enable_1gb_pages.sh
    https://github.com/xmrig/xmrig/blob/dev/scripts/randomx_boost.sh

## Credit
Thanks to `github` user `@DeeDeeRange` for her comment on avoiding benign `msr` errors:
https://github.com/xmrig/xmrig/issues/1973.*

## Basic Build
If you do not have a system up and running already, start here first and get Ubuntu all ready:

    https://github.com/mikenizo808/How-To-Crypto-Mine-for-Monero-XMR-on-Ubuntu-20.04-with-XMRig



## Show Current `hugepages` Setting
Let's jump right in and start looking at things we are interested in.

    sudo cat /proc/sys/vm/nr_hugepages

## Configure `hugepages`
Set to recommended `1280` pages. This is permanent and only needs to be added once.

    sudo bash -c "echo vm.nr_hugepages=1280 >> /etc/sysctl.conf"

## Optional - Show the resulting entry in `sysctl.conf`

    grep huge /etc/sysctl.conf

## Example Output

    grep huge /etc/sysctl.conf
    vm.nr_hugepages=1280

*Note: Now, the next time you run `xmrig`, you will see huge page support listed at startup.*

## Configure 1GB huge pages
Even though we have `hugepages` setup, we can still get additional performance by also enabling `1GB pages`. YOu will also notice that until yo udo this, your startup sequence in `xmrig` will remind you that `1gb paging` is not configured yet.

We can activate this setting by echoing a `3` into `nr_hugepages` (default is `0`), but finding the directory could be tricky the first time so download and run the script:

    https://github.com/xmrig/xmrig/blob/dev/scripts/enable_1gb_pages.sh

## Checking Output
Your path may vary, but for me it looks like this:

        #example
        cat /sys/devices/system/node/node0/hugepages/hugepages-1048576kB/nr_hugepages
        3
    
*Note: The above shows us what we have configured (if anything) for `1GB pages` setting; A result of `0` means `none` (default), and `3` means `1GB pages` are configured.*


## Install `msr-tools` with `apt`
This is required to work with `msr` registers, which we will do a lot.  Specifically, `msr-tools` will give us read (`rdmsr`) and write (`wrmsr`) tools to configure our system as needed.

    sudo apt update
    sudo apt install msr-tools

## Confirm Install
You can use `apt list` or `which` to confirm you have the bits.

    #list with apt (should say installed)
    sudo apt list msr-tools

    #or, use which
    which rdmsr
    which wrmsr

## Download and run performance script
This next script detects your CPU type and sets things up for the best `msr` configuration for `xmrig`. The settings are lost after reboot. 

*Note: This must be run as `sudo`.*


    sudo wget -o ./randomx_boost.sh https://raw.githubusercontent.com/xmrig/xmrig/dev/scripts/randomx_boost.sh
  
    sudo chmod +x ./randomx_boost.sh

    sudo ./randomx_boost.sh

*Note: These settings above can double your hashrate, so feel free to test before and after to see your numbers.*

## Expected: Still getting `msr` Errors on startup
This is expected and you can safely ignore the `msr` errors.  You can clearly see that your hashrate is very high when your `msr` is working.  If in doubt reboot without running the `randomx_boost.sh` and see what your hashrate is.

Read on for some examples of `msr` without and then with `sudo`. Then, we show how to avoid using `sudo`.

## `Error` without `sudo`
Even after applying all scripts and commands it still will not read the msr without root access. Read on for work-arounds later.

    [2022-01-17 12:50:41.997]  msr      cannot read MSR 0xc0011020
    [2022-01-17 12:50:41.997]  msr      FAILED TO APPLY MSR MOD, HASHRATE WILL BE LOW
    
## `Success` With `sudo`
With `sudo` it works no problems. But wait, we are not recommending `sudo`.  Read on.

    [2022-01-17 13:57:04.096]  msr      register values for "ryzen_17h" preset have been set successfully (0 ms)

*We can see above that running with `sudo` works because now XMRig has no problems issuing the `msr` commands. They are doing this to essentially double your hashrate, but does require `sudo` or having run the approriate `msr` settings already as `sudo` to get the high hashrate.*

## Running `xmrig` with no config file
One will be generated for you and you will never see it or worry about it.

    xmrig -o gulf.moneroocean.stream:10128 \
    -u 48VWWKR6kEzByCxSZSTtJYMUgZzM83eTfMAZKbkirmktDMn7XcXaQxsRxja1GACmSaVmHxi5GDg9HGDtRTtSjnm36GZ1CqU \
    -p ubuntu4

*Note: By running without a config file, you will be sure to see errors about `msr`.*

## Running `xmrig` with a config file
You can add a very complex or a very simple config, but it must be in `.json` format.

Here we add the minimal amount of configuration to control the settings related to `msr`. This is located in the `randomx` object are of the config file, or you can use only this section as your entire config as shown below.

## Dealing with `msr` settings in `config.json`
By default the `rdmsr` and `wrmsr` are set to `true`, so we change them to `false`. This avoids the benign startup errors about `msr` when launching `xmrig`.

*Note: Remember that to use this means that we did the settings already for `msr` since rebooting (i.e. by running `randomx_boost.sh` as `sudo`). Then we can tell `xmrig`, I got this yo, by using a config like the one below.*

{

    "randomx": {
        "init": -1,
        "init-avx2": -1,
        "mode": "auto",
        "1gb-pages": true,
        "rdmsr": false,
        "wrmsr": false,
        "cache_qos": false,
        "numa": true,
        "scratchpad_prefetch_mode": 1
    }
}

*Note: Save the above as `config.jason` and place it into your `xmrig` directory*.

## Launch `xmrig` with `--config`
Just as you would expect, simply point to your config file.  Here is my full setup, which keeps all the normal stuff on the main command line, though I could add more to my config as desired to save me from typing pool address, etc.

    ./xmrig --config config.json \
    -o gulf.moneroocean.stream:10128 \
    -u 48VWWKR6kEzByCxSZSTtJYMUgZzM83eTfMAZKbkirmktDMn7XcXaQxsRxja1GACmSaVmHxi5GDg9HGDtRTtSjnm36GZ1CqU \
    -p ubuntu4

*Important: Remember where you see `-p` for `password` never put a real password there! This is a poorly named field that is just a friendly name for your instance (i.e. pi003, ubuntu4, etc.). Not that is matters, but you should not even use the hostname of your machine for this; it is better served with a good name like hercules, or similar.  You get the idea. This name shows up on the boards for you (and everyone) to see and track.*

## No more `msr` Errors
Now your hashrate is still sky high, and no more `msr` errors! Also, you are not running as `sudo`, and this is important.

*Tip: If in doubt, or you just do not care, you can always just run `xmrig` as `sudo` and the `msr` will be handled automatically by default.  However, now you know how to get the best of both worlds (security and high hashrate).*

## Summary
In this write-up we walked through some techniques to increase performance of `xmrig` mining on `Ubuntu`. By following these steps, my hasrate went from 4000 to 8000, though your results will vary.  Have fun!
