**Preface**: Why even write this guide?

I want to RAID 2 partitions on the same drive. Most people would immediately think, well that's plain stupid - yes, BUT not if you have a dual actuator drive, like the Seagate Exos Mach 2!

I wanted to extract more performance out of the dual actuator hard drive I got, but unlike its SAS counterpart, the Exos Mach 2 SATA does not expose 2 LUNs to the system, it appears as a single drive; bummer, except...
In reality, you can split the drive in half and get the same performance out of each side, in parellel, which means raiding them should increase performance. But raid as it normally stands requires disks, not partitions, and Windows, along with many other RAID software will simply not allow you to even attempt to setup an array of partitions on the same drive. so I had to look into alternative solutions, and ultimately successfully tested mdadm coupled with [WinMD](https://github.com/maharmstone/winmd).

**Note**: I would highly suggest surface testing your drive before embarking on this adventure, because you don't want to wake up to data loss after setting this up - and you are the only one responsible for your data and actions, I can't be held liable for anything, do this at your own risk.

Here's are the before and after Crystal Disk Mark tests:

![DiskMark64_2023-11-18_13-03-11](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/e4adbec5-0d9f-4b97-ad23-52cf8f011446)  ![DiskMark32_2023-11-22_00-04-41](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/1506a2ca-18f6-4481-8d35-a41aea967f05)

# Preparing your drive

Get yourself a linux machine or a live USB distribution; I used Kali KDE through Ventoy with the persistence plugin, so I could keep what I install on it.

Once launched, go ahead and open a terminal and install mdadm with the following commands : `sudo apt-get update -y && sudo apt-get install -y mdadm`

Next, let's find which device your hard drive is listed as by running `lsblk`

In my case, the drive was **/dev/sdb**

Next let's prepare the drive with fdisk by running `fdisk /dev/sdb`

Here we will enter `g` to create a GPT partition, then `n` for the first partition, `n` again for the second partition and `w` to apply and save the changes.

Now the obvious question is, well how do I partition this in half? Easy, some big brained individual already thought of [that](https://www.reddit.com/r/linuxadmin/comments/5in11h/is_there_a_way_to_calculate_half_a_disk_space_in/).
Run this command: `echo $(( $(sudo blockdev --getsize /dev/sdb) / 2))` where sdb is your drive.
This number will be your first half's end sector.

Since dual actuator drives are pretty large, this might take a few minutes.

Once that's complete, we're ready to create our raid with the use of mdadm, but first, let's run `lsblk` again to confirm **/dev/sdb1** and **/dev/sdb2** are there and of the size we were expecting.

If everything looks good, run the following commands to setup raid 0 across your two newly created partitions: `sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb1 /dev/sdb2` 

You can then follow that up with `cat /proc/mdstat` to confirm that it was created successfully. It should show you something like `md0 : active raid0 sdb2[1] sdb1[0]`

If everything looks good here as well, we proceed with formatting our new raid with the file system of your choice. For simplicity's sake, since this is going into a Windows machine, I'll be using NTFS. Run the following: `sudo mkntfs -Q /dev/md0` (note that I used Q for quick, since I already surface tested my drive).

Congrats! That was the hardest part.

Now unmount/shutdown your linux machine or live USB, and proceed to your Windows PC.

# Windows setup

This part is pretty simple, go to https://github.com/maharmstone/winmd, download the release and follow the installation steps, it's literally just right clicking the .inf and agreeing to the install.

Once that's done, you can either pop in your disk into a SATA bay, or test it by using an external enclosure first, but either way, what you want to see is the following in your device manager, both a WinMD controller and a WinMD volume when the drive is connected.

![mmc_2023-11-21_21-23-53](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/42c15e73-5924-4614-80b4-e7e2ff5bee44)

You can also confirm that the partitions look good in your disk management, but note that they will not appear like other partitions and you cannot interact with them.

![mmc_2023-11-21_21-23-45](https://github.com/Endracion/mdadm-raid-mount-windows/assets/12702990/fd43d962-3d77-4cc6-afe2-ed2344c4d3c4)

That's it! Enjoy your not quite double performance on your dual actuator drive, under Windows as well.
