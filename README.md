# mdadm-raid-mount-windows

**Preface**: Why even write this guide?

Simple really, I wanted to extract more performance out of the dual actuator hard drive I got, but unlike its SAS counterpart, the Exos Mach 2 SATA does not expose 2 LUNs to the system, it appears as a single drive; bummer.
In reality, you can split the drive in half and get the same performance out of each side, in parellel, which means raiding them should increase performance. But raid as it normally stands requires disks, not partitions, so I had to look into alternative solutions, and finally stumbled upon and successfully tested mdadm coupled with WinMD: https://github.com/maharmstone/winmd

I would also suggest surface testing your drive before starting on this adventure, because you don't want to wake up to data loss after setting this up - and you are only one responsible for your data and actions, I can't be held liable for anything, do this at your own risk.

Here's are the before and after Cystal Disk Mark tests:

![DiskMark64_2023-11-18_13-03-11](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/e4adbec5-0d9f-4b97-ad23-52cf8f011446)

![DiskMark32_2023-11-22_00-04-41](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/1506a2ca-18f6-4481-8d35-a41aea967f05)

# Setup

Get yourself a linux machine or a live USB distribution; I used Kali KDE through Ventoy with the persistence plugin, so I could keep what I install on it.

Once launched, go ahead an open a terminal and install mdadm with the following commands : `sudo apt-get update -y && sudo apt-get install -y mdadm`

Next, let's find what device your hard drive by running `lsblk`

In my case, the drive was **/dev/sdb**

Next let's prepare the drive with fdisk by running `fdisk /dev/sdb`

Here we will run g to create a GPT partition, then n for the first partition, n for the second partition and w to apply and save the changes.

Now the obvious question is, well how do I partition this in half? Easy, some big brained individual already thought of that: https://www.reddit.com/r/linuxadmin/comments/5in11h/is_there_a_way_to_calculate_half_a_disk_space_in/
Run this command: `echo $(( $(sudo blockdev --getsize /dev/sdb) / 2))` where sdb is your drive.
This number will be your end sector half.

Since dual actuator drives are pretty large, this might take a few minutes.

Once that's complete, we're ready to create our raid with the use of mdadm, but first, let's run `lsblk` again to confirm **/dev/sdb1** and **/dev/sdb2** are there and of the size we were expecting.

If everything looks good, run the following commands to setup raid 0 across your two newly created partitions: `sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdb2` 

You can then follow that up with `cat /proc/mdstat` to confirm that it was created successfully. It should show you something like `md0 : active raid0 sdb2[1] sdb1[0]`

If everything looks good here as well, we proceed with formatting our new raid with the file system of your choice. For simplicity's sake, since this is going into a Windows machine, I'll be using NTFS. Run the following: `sudo mkntfs -Q /dev/md0` (not that I used Q for quick, since I already surface tested my drive).

Congrats! That was the hardest part.

Now unmount/shutdown your linux machine or live USB, and proceed to your Windows PC.

