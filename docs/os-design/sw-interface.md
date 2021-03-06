---
title: 'Low Level System Call Interface'
original_url: 'http://eXpOSNitc.github.io/os_design-files/Sw_interface.html'
hide:
    - toc
    - navigation
---


INT 4 to INT 18 are software interrupt handlers. They are used by the user program to access kernel
services (e.g. Read from a file, fork a child process etc). 

The OS design is fixed such that only the user program can call the software interrupts. They should not
be called from inside another interrupt handler or kernel module.

The system call interface is presented below.


<table class="table-bordered" style="text-align: center;">
<thead>
    <tr>
    <th style="text-align: center;">System Call</th>
    <th style="text-align: center;">System Call Number</th>
    <th style="text-align: center;">Interrupt Routine Number</th>
    <th style="text-align: center;">Argument 1</th>
    <th style="text-align: center;">Argument 2</th>
    <th style="text-align: center;">Argument 3</th>
    <th style="text-align: center;">Return Value</th>
    </tr>
</thead>
<tbody>
<tr>
<td rowspan="2">Create</td>
<td rowspan="2">1</td>
<td rowspan="2">4</td>
<td rowspan="2">File Name (String)</td>
<td rowspan="2">Permission (Integer) <span style="color:red"> *</span>
</td>
<td rowspan="2">-</td>
<td>0 - Success / File alredy exists
</tr>
<td>-1 - No free inode table entry</tr>

</tr>

<tr>
<td rowspan="3">Delete</td>
<td rowspan="3">4</td>
<td rowspan="3">4</td>
<td rowspan="3">File Name (String)</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success/File not found
</tr>
<td>-1 - Permission denied</tr>
<td>-2 - File is open</tr>
</tr>

<tr>
<td rowspan="3">Seek</td>
<td rowspan="3">6</td>
<td rowspan="3">5</td>
<td rowspan="3">File Descriptor</td>
<td rowspan="3">Offset</td>
<td rowspan="3">-</td>
<td>0 - Success
</tr>
<td>-1 - File Descriptor is invalid</tr>
<td>-2 - Offset moves File pointer outside file</tr>
</tr>

<tr>
<td rowspan="4">Open</td>
<td rowspan="4">2</td>
<td rowspan="4">5</td>
<td rowspan="4">File Name</td>
<td rowspan="4">-</td>
<td rowspan="4">-</td>
<td> File Descriptor - Success
</tr>
<td> -1 - File Not found or file is not data file or root file</tr>
<td>-2 - System has reached its limit of open files</tr>
<td>-3 - Process has reached its limit of resources</tr>
</tr>

<tr>
<td rowspan="2">Close</td>
<td rowspan="2">3</td>
<td rowspan="2">5</td>
<td rowspan="2">File Descriptor</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success
</tr>
<td>-1 - File Descriptor is invalid</tr>
</tr>


<tr>
<td rowspan="3">Read</td>
<td rowspan="3">7</td>
<td rowspan="3">6</td>
<td rowspan="3"> File Descriptor (-1 for terminal read)</td>
<td rowspan="3"> Word Address (Buffer)</td>
<td rowspan="3">-</td>
<td>0 - Success
</tr>
<td>-1 - File Descriptor given is invalid</tr>
<td>-2 - File pointer has reached the end of file</tr>
</tr>

<tr>
<td rowspan="4">Write</td>
<td rowspan="4">5</td>
<td rowspan="4">7</td>
<td rowspan="4">File Descriptor(-2 for terminal write)</td>
<td rowspan="4">Word to write</td>
<td rowspan="4">-</td>
<td>0 - Success
</tr>
<td>-1 - File Descriptor given is invalid</tr>
<td>-2 - No disk space / File Full</tr>
<td>-3 - Permission denied</tr>
</tr>

<tr>
<td rowspan="3">Fork</td>
<td rowspan="3">8</td>
<td rowspan="3">8</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>PID - Success (in parent process)
</tr>
<td>0 - Success (in child process)</tr>
<td>-1 - Failure (in parent process), Number of processes has reached maximum limit </tr>
</tr>
<tr>
<td rowspan="1">Exec</td>
<td rowspan="1">9</td>
<td rowspan="1">9</td>
<td rowspan="1">File Name</td>
<td rowspan="1">-</td>
<td rowspan="1">-</td>
<td>-1 - File not found or file is not executable
</tr>
</tr>

<tr>
<td>Exit</td>
<td>10</td>
<td>10</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>Getpid</td>
<td>11</td>
<td>11</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>current PID - Success</td>
</tr>

<tr>
<td>Getppid</td>
<td>12</td>
<td>11</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>parent PID - Success</td>
</tr>

<tr>
<td rowspan="2">Wait</td>
<td rowspan="2">13</td>
<td rowspan="2">11</td>
<td rowspan="2">PID</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success
</tr>
<td>-1 - Given PID is invalid or it is PID of invoking process</tr>
</tr>

<tr>
<td>Signal</td>
<td>14</td>
<td>11</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>0 - Success</td>
</tr>

<tr>
<td rowspan="3">Semget</td>
<td rowspan="3">17</td>
<td rowspan="3">13</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>SEMID - Success
</tr>
<td>-1 - Process has reached its limit of resources</tr>
<td>-2 - Number of semaphores has reached its maximum</td>
</tr>

<tr>
<td rowspan="2">Semrelease</td>
<td rowspan="2">18</td>
<td rowspan="2">13</td>
<td rowspan="2">SEMID</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success
</tr>
<td>-1 - Invalid SEMID</tr>
</tr>

<tr>
<td rowspan="2">SemLock</td>
<td rowspan="2">19 </td>
<td rowspan="2">14</td>
<td rowspan="2">SEMID</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success or semaphore is already locked by the current process
</tr>
<td>-1 - invalid SEMID </tr>
</tr>
<tr>
<td rowspan="3">SemUnLock</td>
<td rowspan="3">20</td>
<td rowspan="3">14</td>
<td rowspan="3">SEMID</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success
</tr>
<td>-1 - Invalid SEMID</tr>
<td>-2 - Semaphore was not locked by the calling process</tr>
</tr>

<tr>
<td>Shutdown</td>
<td>21</td>
<td>15</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-1 - Permission denied</td>
</tr>

<tr>
<td rowspan="4">Newusr</td>
<td rowspan="4">22</td>
<td rowspan="4">16</td>
<td rowspan="4">User Name</td>
<td rowspan="4">Password</td>
<td rowspan="4">-</td>
<td>0 - Success
</tr>
<td>-1 - User already exists</tr>
<td>-2 - Permission denied</tr>
<td>-3 - No. of users have reached the system limit.</tr>
</tr>
<tr>
<td rowspan="4">Remusr</td>
<td rowspan="4">23</td>
<td rowspan="4">16</td>
<td rowspan="4">User Name</td>
<td rowspan="4">-</td>
<td rowspan="4">-</td>
<td>0 - Success
</tr>
<td>-1 - User does not exist</tr>
<td>-2 - Permission denied</tr>
<td>-3 - Undeleted files exist for the user</tr>
</tr>
<tr>
<td rowspan="3">Setpwd</td>
<td rowspan="3">24</td>
<td rowspan="3">16</td>
<td rowspan="3">User Name</td>
<td rowspan="3">New password</td>
<td rowspan="3">-</td>
<td>0 - Success
</tr>
<td>-1 - Unauthorised attempt to change password</tr>
<td>-2 - The user does not exist.</tr>
</tr>
<tr>
<td rowspan="2">Getuname</td>
<td rowspan="2">25</td>
<td rowspan="2">16</td>
<td rowspan="2">User ID</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>User Name - Success
</tr>
<td>-1 - Invalid User ID</td>
</tr>
<tr>
<td rowspan="2">Getuid</td>
<td rowspan="2">26</td>
<td rowspan="2">16</td>
<td rowspan="2">User Name</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>User ID - Success
</tr>
<td>-1 - Invalid username</td>
</tr>
<tr>
<td rowspan="3">Login</td>
<td rowspan="3">27</td>
<td rowspan="3">17</td>
<td rowspan="3">User Name</td>
<td rowspan="3">Password</td>
<td rowspan="3">-</td>
<td>0 - Success
</tr>
<td>-1 - Invalid username or password</tr>
<td>-2 - Permission denied</tr>
</tr>
<td rowspan="1">Logout</td>
<td rowspan="1">28</td>
<td rowspan="1">12</td>
<td rowspan="1">-</td>
<td rowspan="1">-</td>
<td rowspan="1">-</td>
<td rowspan="1">-1 - permission denied</td>
</tr>

<tr>
<td>Test0</td>
<td>96</td>
<td>18</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>Test1</td>
<td>97</td>
<td>18</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>Test2</td>
<td>98</td>
<td>18</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>Test3</td>
<td>99</td>
<td>18</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td style="color:red">Test4 <span style="color:red">**</span></td>
<td>100</td>
<td>19</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td style="color:red">Test5 <span style="color:red">**</span></td>
<td>101</td>
<td>19</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td style="color:red">Test6 <span style="color:red">**</span></td>
<td>102</td>
<td>19</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td style="color:red">Test7 <span style="color:red">**</span></td>
<td>103</td>
<td>19</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>
</tbody>
</table>

<span style="color:red">*</span>If the file is created with permission set to [EXCLUSIVE](../support-tools/constants.md), then write and delete system calls will fail when executed by any user other than the owner or the root (see [here](../support-tools/constants.md)).


<span style="color:red">**</span> These System Calls are available only on eXpOS running on [NEXSM](../arch-spec/nexsm.md) (a two-core extension of XSM) machine.