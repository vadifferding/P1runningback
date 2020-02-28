# Project 1: System Inspector

Author: Ginny Diffeding

See: https://www.cs.usfca.edu/~mmalensek/cs326/assignments/project-1.html 

## About this Project

This is a Unix Utitity that inspects a system and creates a summerized report, including the processing running and other detals concerning the VM. If you’ve ever used the top command from a shell, our program will be somewhat similar. This report will harvested using proc, the process information pseudo-filesystem.


### Program Options
Each portion of the display can be toggled with command line options. Here are the options:
$ ./inspector -h
Usage: ./inspector [-ho] [-i interval] [-p procfs_dir]

Options:
    * -h              Display help/usage information
    * -i interval     Set the update interval (default: 1000ms)
    * -p procfs_dir   Set the expected procfs mount point (default: /proc)
    * -o              Operate in one-shot mode (no curses or live updates)
    
    

### Included Files
There are several files included. These are:
   - <b>Makefile</b>: Including to compile and run the program.
   - <b>inspector.c</b>: The program driver
   - <b>display.c</b>: Includes functions that create the UI and format the information.
   - <b>procfs.c</b>: Includes functions to get the hardware information of the system. This infomation includes CPU model, number of processing units, load average, CPU usage, and memory usage. Also includes functions for creation and setting of task structs for processing and other helper functions for these tasks.
   - <b>util.c</b>: Includes functions that will map uid to a username and draw the percent bar.

There are also header files for each of these files.


To compile and run:

```bash
make
./inspector
```

## Testing

To execute the test cases, use `make test`. To pull in updated test cases, run `make testupdate`. You can also run a specific test case instead of all of them:

```
# Run all test cases:
make test

# Run a specific test case:
make test run=4

# Run a few specific test cases (4, 8, and 12 in this case):
make test run='4 8 12'
```



### Program Output
```bash
$ ./inspector
Hostname: steel-beans | Kernel Version: 5.4.15
CPU: AMD EPYC Processor (with IBPB), Processing Units: 2
Uptime: 3 days, 23 hours, 51 minutes, 19 seconds

Load Average (1/5/15 min): 0.05 0.02 0.00
CPU Usage:    [######--------------] 30.0%
Memory Usage: [##################--] 90.6% (14.5 / 16.0 GB)

Tasks: 101 total
2 running, 0 waiting, 98 sleeping, 1 stopped, 0 zombie

      PID |                 Task Name |        State |            User
----------+---------------------------+--------------+-----------------
     9783 |                      sshd |      running |            root
    11642 |                       top |      stopped |            root
    12701 |                 inspector |      running |            root



