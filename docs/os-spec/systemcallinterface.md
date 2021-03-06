---
title: 'High Level System Call Interface'
original_url: 'http://eXpOSNitc.github.io/os_spec-files/systemcallinterface.html'
hide: 
    - navigation
---
 

Application programmers interact with the Operating System using the system calls. System calls are stored in the disk and are loaded into memory when the OS is loaded by the bootstrap loader. When a process invokes a system call, the process is interrupted and control goes to the corresponding interrupt service routine of the kernel, resulting in a switch from user mode to kernel mode. Once the system call is carried out, the control goes back to the application program, with a switch back to the user mode.

The system calls of eXpOS are classified into file system calls, process system calls and system calls for access control and synchronization.

\* In addition to this, [Multiuser](multiuser.md) extension of eXpOS includes system calls for handling multiple users.
 
  

### File System Calls

#### Create

Arguments: Filename (String), Permission (Integer)

Return Value:

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success/File already exists</td>
</tr> 
<tr>
<td>-1</td>
<td>No free inode table entry</td>
</tr>
</table>

Description: The Create operation takes as input a filename and creates an empty file by that name. If a root entry for the file already exists, then the system call returns 0 (success). Otherwise, it creates a root entry for the file name, sets the file type to DATA and file size to 0.

\* In [Multiuser](multiuser.md) implementation, the file permission is either 0 or 1, indicating file is open-access or exclusive respectively. If the exclusive flag is set, write and delete system calls will fail except when made by the owner or the root.

#### Delete

Arguments: Filename (String)

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Permission Denied</td>
</tr> 
<tr>
<td>-2</td>
<td>File is open</td>
</tr>
</table>

Description: Delete removes the file from the file system and removes its root entry. A file that is currently opened by any application cannot be deleted. Root file and Executable files also cannot be deleted.

\* In addition to this in [Multiuser](multiuser.md) implementation, data files which are created with exclusive permission, cannot be deleted by any user other than the owner, root or kernel.

#### Open

Arguments: Filename (String)

Return Value:

<table class="table table-bordered">
<tr>
<td>File Descriptor (Integer)</td>
<td>Success, the return value is the file descriptor for the opened file.  </td>
</tr> 
<tr>
<td>-1</td>
<td>File not found or file is not a data file or root file</td>
</tr> 
<tr>
<td>-2</td>
<td>System has reached its limit of open files</td>
</tr>
<tr>
<td>-3</td>
<td>Process has reached its limit of resources</td>
</tr>
</table>

Description: For a process to read/write a file, it must first open the file. Only data and root files can be opened. The Open operation returns a file descriptor. An application can open the same file several times and each time, a different descriptor will be returned by the Open operation. The file descriptor must be passed as argument to other file system calls, to identify the open instance of the file.

The OS associates a file pointer with every open instance of a file. The file pointer indicates the current location of file access (read/write). The Open system call sets the file pointer to 0 (beginning of the file).

#### Close

Arguments: File Descriptor (Integer)

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>File Descriptor given is invalid</td>
</tr> 
<!-- <tr>
<td>-2</td>
<td>File is locked by the calling process.</td>
</tr> -->
</table>

Description: After all the operations are done, the user closes the file using the Close system call. The file descriptor ceases to be valid once the close system call is invoked.

#### Read

Arguments: File Descriptor (Integer) and a Buffer (a String/Integer variable) into which a word is to be read from the file

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>File Descriptor given is invalid</td>
</tr> 
<tr>
<td>-2</td>
<td>File pointer has reached the end of file</td>
</tr> 
</table>

Description: The file descriptor is used to identify an open instance of the file. The Read operation reads one word from the position pointed by the file pointer and stores it into the buffer. After each read operation, the file pointer advances to the next word in the file.

#### Write

Arguments: File Descriptor(Integer) and the word to be written

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>File Descriptor given is invalid</td>
</tr> 
<tr>
<td>-2</td>
<td>No disk space / File Full</td>
</tr> 
<tr>
<td>-3</td>
<td>Permission denied</td>
</tr>
</table>

Description: The file descriptor is used to identify an open instance of the file. The Write operation writes the word passed as argument to the position pointed by the file pointer of the file. After each Write operation, the file pointer advances to the next word in the file. Root file and Executable files cannot be written.

\* In addition to this in [Multiuser](multiuser.md) implementation, data files which are created with exclusive permission, cannot be written into by any user other than the owner, root or kernel.

#### Seek

Arguments: File Descriptor (Integer) and Offset (Integer) specifying the number of positions by which the file pointer has to be shifted

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>File Descriptor given is invalid</td>
</tr> 
<tr>
<td>-2</td>
<td>Offset value moves the file pointer to a position outside the file</td>
</tr>
</table>

Description: The Seek operation allows the application program to change the value of the file pointer so that subsequent Read/Write is performed from a new position in the file. The new value of the file pointer is determined by adding the offset to the current value. (A negative Offset will move the pointer backwards). An Offset of 0 will reset the pointer to the beginning of the file. An offset that moves beyond the end of the file will set the file pointer to the end of the file, then returns -2. This is useful to append data to the file.

  
  

### Process System Calls

#### Fork

Arguments: None

Return Value:

<table class="table table-bordered">
<tr>
<td>PID (Integer) </td>
<td>Success, the return value to the parent is the process descriptor(PID) of the child process.</td>
</tr> 
<tr>
<td>0</td>
<td>Success, the return value to the child.</td>
</tr> 
<tr>
<td>-1</td>
<td>Failure, Number of processes has reached the maximum limit. Returns to the parent</td>
</tr> 
</table>

Description: Replicates the process invoking the system call. The heap, code and library regions of the parent are shared by the child. A new stack is allocated to the child and the parent's stack is copied into the child's stack.

When a process executes the Fork system call, the child process shares with the parent all the file and semaphore descriptors previously acquired by the parent. Semaphore/file descriptors acquired subsequent to the fork operation by either the child or the parent will be exclusive to the respective process and will not be shared.

\* In [Multiuser](multiuser.md) implementation, the child process inherits the userid of the parent process.

#### Exec

Arguments: File Name (String) of the executable file (which must be of XEXE format)

Return Value:
<table class="table table-bordered">
<tr>
<td>-1</td>
<td>File not found or file is of invalid type</td>
</tr> 
<tr>
<td>-2</td>
<td>Out of memory or disk swap space</td>
</tr> 
</table>

Description: Exec destroys the present process and loads the executable file given as input into a new memory address space. A successful Exec operation results in the extinction of the invoking application and hence never returns to it. All open instances of file and semaphores of the parent process are closed. However, the newly created process will inherit the PID of the calling process.

\* In [Multiuser](multiuser.md) implementation, the newly created process inherits the userid of the invoking process.

#### Exit

Arguments: None

Return Value: None

Description: Exit system call terminates the execution of the process which invoked it and destroys its memory address space. The calling application ceases to exist after the system call and hence the system call never returns. All processes waiting for this process using [Wait system call](systemcallinterface.md#system-calls-for-access-control-and-synchronization), are awakened on exit of this process. Exit never fails.

\* In [Multiuser](multiuser.md) implementation, if the process created by the login process (shell) exits, all processes with that userid has to be killed and the login process that was waiting for its exit has to be awakened.

#### Getpid

Arguments: None

Return Value:
<table class="table table-bordered">
<tr>
<td>Process Identifier (Integer) </td>
<td>Success</td>
</tr> 
</table>

Description: Returns the process identifier of the invoking process. The system call does not fail.

#### Getppid

Arguments: None

Return Value:
<table class="table table-bordered">
<tr>
<td>Parent Process Identifier (Integer)</td>
<td>Success</td>
</tr> 
</table>

Description: Returns to the calling process the value of the process identifier of its parent. The system call does not fail.

#### Shutdown

Arguments: None

Return Value:

<table class="table table-bordered">
<tr>
<td>-1</td>
<td>Permission denied</td>
</tr> 
</table>

Description: Shutdown system call terminates all processes and halts the machine.

\* In [Multiuser](multiuser.md) implemention of eXpOS, only processes with the user as either the root or kernel can execute shutdown system call.



### System Calls for access control and synchronization

#### Wait

Arguments: Process Identifier (Integer) of the process for which the current process has to wait.

Return Value:

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Given process identifier is invalid or it is the pid of the invoking  process.</td>
</tr> 
</table>

Description: The current process is blocked till the process with PID given as argument executes a Signal system call or exits. Note that the system call will fail if a process attempts to wait for itself.

#### Signal

Arguments: None

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr>
</table>

Description: All processes waiting for the signaling process are resumed. The system call does not fail.

#### Semget

Argument: None

Return Value :

<table class="table table-bordered">
<tr>
<td>SEMID (Integer) </td>
<td>Success, returns a semaphore descriptor(SEMID)</td>
</tr> 
<tr>
<td>-1</td>
<td>Process has reached its limit of resources </td>
</tr> 
<tr>
<td>-2</td>
<td>Number of semaphores has reached its maximum</td>
</tr> 
</table>

Description: This system call is used to obtain a binary semaphore. eXpOS has a fixed number of semaphores. The calling process can share the semaphore with its child processes using the fork system call

#### Semrelease

Arguments: Semaphore Descriptor (Integer)

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Semaphore Descriptor is invalid</td>
</tr>                     
</table>

Description: This system call is used to release a semaphore descriptor held by the process.

#### SemLock

Arguments: Semaphore Descriptor (Integer)

Return Value:

<table class="table table-bordered">
<tr>
    <td>0</td>
    <td>Success or the semaphore is already locked by the current process</td>
</tr> 
<tr>
    <td>-1</td>
    <td>Semaphore Descriptor is invalid</td>
</tr>
</table>

Description: This system call is used to lock the semaphore. If the semaphore is already locked by some other process, then the calling process goes to sleep and wakes up only when the semaphore is unlocked. Otherwise, it locks the semaphore and continues execution.

#### SemUnLock

Arguments: Semaphore Descriptor (Integer)

Return Value:

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Semaphore Descriptor is invalid</td>
</tr> 
<tr>
<td>-2</td>
<td>Semaphore was not locked by the calling process</td>
</tr> 
</table>

Description: This system call is used to unlock a semaphore that was previously locked by the calling process. It wakes up all the processes which went to sleep trying to lock the semaphore while the semaphore was locked by the calling process.

  
  

### Multiuser System Calls

#### Newusr

Arguments: User name, Password

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr>
<tr>
<td>-1</td>
<td>User already exists</td>
</tr> 
<tr>
<td>-2</td>
<td>Permission denied</td>
</tr>
<tr>
<td>-3</td>
<td>Number of users have reached the system limit</td>
</tr>
</table>

Description: This system call is used to create a new user. It checks whether the user already exists. If not, it creates a new user with the username and password specified. This system call can be executed only by the shell process of root user.

#### Remusr

Arguments: User name

Return Value:

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>User does not exist</td>
</tr> 
<tr>
<td>-2</td>
<td>Permission denied</td>
</tr>
<tr>
<td>-3</td>
<td>Undeleted files exist for the user</td>
</tr> 
</table>

Description: This system call is used to remove an existing user. This system call can be executed from the shell proces of the root user. The root user and kernel cannot be removed.

#### Setpwd

Arguments: User name, New Password

Return Value:

<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr>
<tr>
<td>-1</td>
<td>Unauthorised attempt to change password</td>
</tr> 
<tr>
<td>-2</td>
<td>The user does not exist.</td>
</tr>     
</table>

Description: This system call is used to change the password of an existing user. This system call can be exected only from the shell process. A user can set only his/her password. Root user can set any user's password.

#### Getuid

Arguments: User name

Return Value:

<table class="table table-bordered">
<tr>
<td>User Identifier</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Invalid Username</td>
</tr>         
</table>

Description: If the username is valid, this system call returns the userid corresponding to the username. Otherwise, it returns -1.

#### Getuname

Arguments: User ID

Return Value:

<table class="table table-bordered">
<tr>
<td>User Name</td>
<td>Success</td>
</tr>
<tr>
<td>-1</td>
<td>Invalid UserID</td>
</tr> 
</table>

Description: If the userid is valid, this system call returns the username corresponding to the userid. Otherwise, it returns -1.

#### Login

Arguments: User name, Password

Return Value:
<table class="table table-bordered">
<tr>
<td>0</td>
<td>Success</td>
</tr> 
<tr>
<td>-1</td>
<td>Invalid username or password</td>
</tr> 
<tr>
<td>-2</td>
<td>Permission denied</td>
</tr> 
</table>

Description: This system call is used to login a new user. It can be executed only from the login process. It verifies the user. Upon successful login, a new user process (shell) with the userid of the user specified is created and the calling process goes to sleep till the exit of the newly created shell process.

#### Logout

Arguments: None

Return Value: -1 on error, otherwise does not return

Description: This system call is used to logout the current user. It can be invoked only from the shell process (PID = 2). When the logout system call is invoked, all running processes of the current user are terminated and all resources released. Idle and init/Login will be the only processes running after the execution of Logout. Login is woken up at the end of logout.