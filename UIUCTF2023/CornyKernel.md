# Corny Kernel 

This challenge involves obtaining a flag by understanding the behavior of a linux kernel module. 

The challenge provides us with a file called pwnymodule.c and remote access to a server to run our commands. 

## Step 1: Examining pwnymodule.c
- - -
We are provided with pwnymodule.c. I first examined the file to see if I could understand what the module was doing. 


The first thing that caught my eye was this: 

```
extern const char *flag1, *flag2;
```

We found our flags. Since they are extern, the actual values are defined somewhere else on the server. This module has pointers to them and can access the flags directly. 



The next thing I noticed were these two functions. 
```
static int __init pwny_init(void)
{
	pr_alert("%s\n", flag1);
	return 0;
}

static void __exit pwny_exit(void)
{
	pr_info("%s\n", flag2);
}
```
[Google Fu](https://www.kernel.org/doc/html/latest/core-api/printk-basics.html) helped me learn that pr_info and pr_alert use printk, which is like printf but it prints to the kernel log. 
This means that these two functions print the flag. 


The kernel module calls these two functions below: 

```
module_init(pwny_init);
module_exit(pwny_exit);
```

[Google Fu](https://tldp.org/LDP/lkmpg/2.4/html/x281.htm) tells me that module_init and module_exit define what functions to call when ever the module is loaded and removed from the kernel. 

If we figure out how to load and remove the module, the flags will be printed out to the kernel log. 

## Step 2: Server Actions
- - -

On the server, we only have one file in our current directory. 
```
/root # ls
pwnymodule.ko.gz
/root #
```

We don't have a pwnymodule.c at all. I know that a .gz file is a compressed file from previous experience, and google tells me that a .ko file is a kernel object file. This means our pwnymodule.c file has already been compiled and compressed. We need to decompress it. 


I [learned](https://linuxize.com/post/how-to-unzip-gz-file/) that to unzip a .gz file I can use the following command. 

```
/root # gzip -d pwnymodule.ko.gz
/root # ls
pwnymodule.ko
/root #
```

Now that we have our uncompressed module, we need to [load](https://linux.die.net/man/8/insmod) it. 

```
/root # insmod pwnymodule.ko
[  432.583070] pwnymodule: uiuctf{m4ster_
/root #
```
Loading the module shows half of the flag. We need to [remove](https://linux.die.net/man/8/rmmod) the module in order to show the other half. 


```
/root # rmmod pwnymodule.ko
```

There was no printout to the terminal. Looking back at the source code, the exit function used pr_info, which I assume does not print to our stdout compared to pr_alert. 

Still we need to [access our kernel logs](https://man7.org/linux/man-pages/man1/dmesg.1.html) to find our flag. 

```
/root # dmesg | tail
[    0.141288] Freeing unused kernel image (rodata/data gap) memory: 1452K
[    0.141291] Run /init as init process
[    0.141292]   with arguments:
[    0.141293]     /init
[    0.141293]   with environment:
[    0.141294]     HOME=/
[    0.141294]     TERM=linux
[    0.147268] mount (31) used greatest stack depth: 13464 bytes left
[   11.188522] pwnymodule: uiuctf{m4ster_
[   15.657667] pwnymodule: k3rNE1_haCk3r}
/root #
```

We now have the complete flag!


