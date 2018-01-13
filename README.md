# Playground for disk encryption
Here you will find some examples and quick setup for testing various disk encryption options

# What is this
This is my small attempt at making disk encryption more usable for everyone. Though this is oriented towards sysadmins it shouldn't be too hard to follow.

# What encryptions are available
Note that this list might not be correct. We sysadmins are lazy and usually forget to update the documentation.

The current tested and working encryptions are:

 * Veracrypt - the successor of truecrypt
 * LUKS or more widely known as crypto-luks

# How can i try this
First of all you must be at least marginally familiar with linux. Then you will need the following

 * Virtualbox from Oracle (or the OSS variant)
 * Vagrant from Hashicorp

After cloning this repo just do `vagrant up` and in some minutes you will have few VM's with encrypted disks (not the root ones though)

The encrypted disk is mounted under data. The disk itself is just 4 GB so not much but good for testing stuff (and doing some light performance testing).

# What are the passwords
Please note that using paswords available on the web is bad bad bad idea. You are supposed to change them.

Notwithstanding that

 * veracrypt - `veracrypt`
 * luks - `cryptoluks`

# License
This work is licensed under the Creative Commons Attribution-ShareAlike 4.0 International License. To view a copy of this license, visit http://creativecommons.org/licenses/by-sa/4.0/ or send a letter to Creative Commons, PO Box 1866, Mountain View, CA 94042, USA.
