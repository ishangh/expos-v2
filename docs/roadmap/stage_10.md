---
title: 'Stage 10 :
                        Console output (4 Hours)'
---
<div class="panel-collapse collapse" id="collapse10">
 <div class="panel-body">
  <!-- Begin Learning Objectives-->
  <div class="container col-md-12">
   <div class="section_area">
    <ul class="list-group">
     <li class="list-group-item" style="background:#dff0d8">
      <span class="fa fa-book">
      </span>
      <a data-toggle="collapse" href="#lo10">
       Learning
                                Objectives
      </a>
      <div class="panel-collapse expand" id="lo10">
       <ul>
        <li style="margin-bottom: -2px">
         <span class="fa fa-hand-o-right">
         </span>
         Familiarise with the
         <a href="abi.html" target="_blank">
          low level system call
                                      interface
         </a>
         in eXpOS.
        </li>
        <li style="margin-bottom: -2px">
         <span class="fa fa-hand-o-right">
         </span>
         Familiarise with the console output mechanism in eXpOS.
        </li>
       </ul>
      </div>
     </li>
     <li class="list-group-item" style="background:#dff0d8">
      <span class="fa fa-book">
      </span>
      <a data-toggle="collapse" href="#lo10a">
       Pre-requisite Reading
      </a>
      <div class="panel-collapse expand" id="lo10a">
       <ul>
        <li style="margin-bottom: -2px">
         <span class="fa fa-hand-o-right">
         </span>
         Read and understand the
         <a href="os_design-files/stack_smcall.html" target="_blank">
          Kernel
                                      Stack Management during system calls
         </a>
         before proceeding further.
        </li>
       </ul>
      </div>
     </li>
    </ul>
   </div>
  </div>
  <!-- End Learning Objectives-->
  <p>
   In Stage 7, we wrote a user program and used the BRKP instruction to view the result in debug
                        mode.

                        In this stage, we will modify the program such that the result is printed directly to the
                        terminal.
                        The terminal print is acheived by issuing a write system call from the user program.
                        The write system call is serviced by interrupt routine 7.
  </p>
  <b style="font-size:18px">
   Modifications to the user program
  </b>
  <br/>
  <br/>
  <p>
   A system call is an OS routine that can be invoked from a user program. The OS provides system
                        call routines for various
                        services like writing to a file/console, forking a process etc. Each system call routine is
                        written inside
                        some software interrupt handler. For example, the write system call of eXpOS is coded inside
                        the INT 7 handler.
                        An interrupt handler may contain code for several system calls.
                        (For example, in the eXpOS implementation on XSM, the routines for create and delete system
                        calls
                        are coded inside the INT 4 handler - find details
   <a href="os_design-files/Sw_interface.html" target="_blank">
    here
   </a>
   ). To identify the correct routine, the OS assigns a unique system
                        call number to each system call routine. To invoke a system call from a program, the program
                        must pass the system call number (along with other arguments to the system call) and invoke the
                        corresponding
                        software interrupt using the INT instruction. The arguments and the system call number are
                        passed through the user
                        program's stack.
  </p>
  <p>
   When a program invokes a system call, the system switches from user mode to kernel mode.
                        Hence, system calls run in
                        kernel mode and thus have access to all the hardware resources. Upon completing the call, the
                        system call places
                        return value of the call into designated position in the user program's stack and returns
                        to the calling program using the IRET. Since the IRET instruction switches mode back to user
                        mode, the user program
                        resumes execution after the call in user mode. The user program extracts the return values of
                        the call from the
                        user stack.
  </p>
  <p>
   In this stage, we will write a small kernel routine for handling console write. This is part
                        of the functionality of the
                        write system call (system call number 5) programmed inside the INT 7 handler. You will
                        implement the full functionality
                        of the write system call in later stages.
  </p>
  <p>
   The user program of Stage 7 is modified such that a write system call is issued to print the
                        contents of
                        register R1 to the terminal. You will no longer need to run the program in debug mode.
                        This is because once we implement the system call service for console output, this system call
                        can be used
                        by the user program to print the output to the console.
                        A user program must execute the following steps to invoke the system call:
  </p>
  <br/>
  <ol style="list-style-type:decimal;margin-left:2px">
   <li>
    Save the registers in use to the user stack (in the program below R0, R1, R2 are saved). As
                          per the specification, since the user program calls system call routine , the OS expects that
                          it saves its own context (registers in use) before issuing the system call.
   </li>
   <li>
    Push the system call number and arguments to the stack. For the Write system call, the
                          system call number is 5. Argument 1 is the file descriptor which is -2 for the terminal.
                          Argument 2 is the word which has to be written to the terminal. Here the word we are going to
                          write is present in R1. By convention, all system calls have 3 arguments. As we do not have a
                          third argument in this case, push any register, say R0 on to the stack. (In this case the
                          last argument will be ignored by the system call handler.) Refer to the low level system call
                          interface for write
    <a href="os_design-files/Sw_interface.html" target="_blank">
     here
    </a>
    .
   </li>
   <li>
    Push any register, say R0 to allocate space for the return value.
   </li>
   <li>
    Invoke the interrupt by "INT 7" instruction.
   </li>
   //The following code will be executed after return from the system call.
   <br/>
   <br/>
   <p>
    Normally, the return value of a system call gives information regarding whether the system
                          call succeeded or
                          whether there was an error etc. In some cases, the system call returns a value which is to be
                          used later in program (for instance, the open system call returns a file descriptor). In the
                          present case, since console write never fails, we ignore the return
                          value.
   </p>
   <li>
    Pop out the return value, the system call number and arguments which were pushed on the
                          stack prior to the system call.
   </li>
   <li>
    Restore the register context from the stack (in the following program R0,R1,R2 are
                          restored).
   </li>
  </ol>
  <p>
   The resulting program is given below.
   <div>
    <pre>
0
2056
0
0
0
0
0
0
MOV R0, 1
MOV R2, 5
GE R2, R0
JZ R2, 2110
MOV R1, R0
MUL R1, R0

// saving register context
PUSH R0
PUSH R1
PUSH R2

// pushing system call number and arguments
MOV R0, 5
MOV R2, -2
PUSH R0
PUSH R2
PUSH R1
PUSH R0

//  pushing space for return value
PUSH R0
INT 7

// poping out return value and ignore
POP R1

// pop out argumnets and system call number and ignore
POP R1
POP R1
POP R1
POP R1

//  restoring the register context
POP R2
POP R1
POP R0

ADD R0, 1
JMP 2058

INT 10
	</pre>
   </div>
  </p>
  <br/>
  <br/>
  <div style=" width: 100%">
   <p style="font-size:18px">
    Contents of the stack before and after the INT instruction
   </p>
   <img height="350px" src="img/system_call_stack1.png" style="float:left; margin-right: 100px; margin-left:75px;" width="350px"/>
   <img height="350px" src="img/system_call_stack2.png" style="float:right; margin-left: 100px; margin-right:75px;" width="350px"/>
  </div>
  <br/>
  <br/>
  Now, you will write the system call handler for processing the write request.
  <br/>
  <br/>
  <b style="font-size:20px">
   INT 7
  </b>
  <br/>
  <p>
   The write operation is handled by Interrupt 7.
                        The word to be printed is passed from the user program through its user stack as the
                        second argument to the interrupt routine. The interrupt routine retrieves this word from the
                        stack
                        and writes the word to the terminal using the OUT instruction.
  </p>
  <p>
   Detailed instructions for doing so are given below
  </p>
  <ol style="list-style-type:decimal;margin-left:2px">
   <li>
    Set the MODE FLAG field in the
    <a href="os_design-files/process_table.html" target="_blank">
     process
                            table
    </a>
    to the system call number which is 5 for write system call. To get the process
                          table of current process, use the PID obtained from the
    <a href="os_design-files/mem_ds.html#ss_table" target="_blank">
     system status table
    </a>
    . MODE FLAG field in the process table is used to
                          indicate whether the current process is executing in a system call, exception handler or user
                          mode.
    <div>
     <pre>[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 5;</pre>
    </div>
   </li>
   <li>
    Store the value of user SP in a register as we need it for further computations.
    <div>
     <pre>alias userSP R0;
userSP = SP;</pre>
    </div>
   </li>
   <li>
    Switch the stack from user stack to kernel stack.
    <ol style="margin-left:20px;list-style-type:disc;">
     <li>
      Save the value of SP in the user SP field of
      <a href="os_design-files/process_table.html" target="_blank">
       Process Table
      </a>
      entry of the process.
     </li>
     <li>
      Set the value of SP to beginning of the kernel stack.
     </li>
    </ol>
    Details can be found at
    <a href="os_design-files/stack_smcall.html" target="_blank">
     Kernel
                            Stack Management during system calls
    </a>
    .
   </li>
   <li>
    First we have to access argument 1 which is file descriptor to check whether it is valid or
                          not. In user mode, logical addresses are translated to physical address by the machine using
                          its
    <a href="arch_spec-files/paging_hardware.html" target="_blank">
     address translation scheme
    </a>
    .
                          Since interrupts are executed in the kernel mode, the actual physical address is used to
                          access memory
                          locations. Hence to access the file descriptor (argument 1) we must calculate the physical
                          address of the memory location where it is stored. According to system call conventions,
                          userSP - 4 is the location of the argument 1. So we will manually address translate userSP -
                          4 (See contents of the stack after INT instruction in above image for reference).
    <br/>
    <br/>
    <div>
     <pre>alias physicalPageNum R1;
alias offset R2;
alias fileDescPhysicalAddr R3;
physicalPageNum = [PTBR + 2 * ((userSP - 4)/ 512)];
offset = (userSP - 4) % 512;
fileDescPhysicalAddr = (physicalPageNum * 512) + offset;
alias fileDescriptor R4;
fileDescriptor=[fileDescPhysicalAddr];</pre>
    </div>
   </li>
   <li>
    Check whether the file descriptor obtained in above step is valid or not. In this stage it
                          should be -2 because file descriptor for console is -2. (see details
    <a href="os_design-files/Sw_interface.html" target="_blank">
     here
    </a>
    .) Write an IF condition to check whether file descriptor is -2 or
                          not.
    <br/>
    <br/>
    <div>
     <pre>if (fileDescriptor != -2)
then
	 //code when argument 1 is not valid
else
	 //code when argument 1 is valid
endif;</pre>
    </div>
   </li>
   <li>
    If the file descriptor is not equal to -2, store -1 as a return value. According to system
                          call convention, return value is stored at memory location userSP -1 in the user stack.
                          Calculate physical address of the return value corresponding to userSP - 1 using address
                          translation mechanism.
    <br/>
    <br/>
    <div>
     <pre>if (fileDescriptor != -2)
then
	 alias physicalAddrRetVal R5;
	 physicalAddrRetVal = ([PTBR + 2 * ((userSP - 1) / 512)] * 512) + ((userSP - 1) % 512);
	 [physicalAddrRetVal] = -1;
else
	 //code when argument 1 is valid
endif;</pre>
    </div>
   </li>
   <br/>
   <p>
    The following three steps has to be included in the else block.
   </p>
   <li>
    Calculate physical address of the argument 2 and extract the value from it , which is the
                          word to be printed to the console.
    <div>
     <pre>alias word R5;
word = [[PTBR + 2 * ((userSP - 3) / 512)] * 512 + ((userSP - 3) % 512)];</pre>
    </div>
   </li>
   <li>
    Write the word to the terminal using the print instruction.
    <div>
     <pre>print word;</pre>
    </div>
   </li>
   <li>
    Set the return value as 0 indicating success.
                          According to system call convention, return value is stored at memory location userSP -1 in
                          the user stack.
    <div>
     <pre>alias physicalAddrRetVal R6;
physicalAddrRetVal = ([PTBR + 2 * (userSP - 1)/ 512] * 512) + ((userSP - 1) % 512);
[physicalAddrRetVal] = 0;</pre>
    </div>
   </li>
   <li>
    Outside the else block, set back the value of SP to point to top of user stack.
    <pre> SP = userSP;</pre>
   </li>
   <li>
    Reset the MODE FLAG field in the process table to 0. Value 0 indicates that process is
                          running in user mode.
    <div>
     <pre>[PROCESS_TABLE + [SYSTEM_STATUS_TABLE + 1] * 16 + 9] = 0;</pre>
    </div>
   </li>
   <li>
    Pass control back to the user program using the ireturn statement.
   </li>
   <br/>
  </ol>
  <b>
   Modifications to the OS startup code
  </b>
  <br/>
  <br/>
  <p>
   Add code in the OS startup code to load INT7 from disk to memory.
  </p>
  <pre>
loadi(16,29);
loadi(17,30);
</pre>
  <br/>
  <b>
   Making Things Work
  </b>
  <br/>
  <br/>
  <p style="text-indent: 0px">
   <code>
    Note :
   </code>
   <b>
    [Implementation Tip]
   </b>
   From this stage
                        onwards, you have to load multiple files using XFS-interface. To make things easier, create a
                        batch file containing XFS-interface commands to load the required files and run this batch file
                        using
   <b>
    run
   </b>
   command. See the usage of
   <b>
    run
   </b>
   command in
   <a href="./support_tools-files/xfs-interface.html" target="_blank">
    XFS-interface documentation
   </a>
   .
  </p>
  <ol style="list-style-type:decimal;margin-left:2px">
   <li>
    Save this file in your UNIX machine as $HOME/myexpos/spl/spl_progs/sample_int7.spl
   </li>
   <li>
    Compile this program using the SPL compiler.
   </li>
   <li>
    Load the compiled XSM code as INT 7 into the XSM disk using XFS Interface.
   </li>
   <li>
    Run the Machine with timer disabled.
   </li>
  </ol>
  <br/>
  <p style="text-indent: 0px">
   <code>
    Note:
   </code>
   Starting from the next stage, you will be writing
                        user programs using a high level language called
                        ExpL. ExpL allows you to write programs that invoke system calls using the
   <a href="os_spec-files/dynamicmemoryroutines.html" target="_blank">
    exposcall() function
   </a>
   .
                        The ExpL compiler will automatically generate code to translate your high level function call
                        to a call to the
   <a href="abi.html" target="_blank">
    eXpOS
                          library
   </a>
   and the library contains code to translate the call to an INT invocation as done
                        by you
                        in this stage. The next stage will introduce you to ExpL.
  </p>
  <div class="container col-md-12">
   <div class="section_area">
    <ul class="list-group">
     <li class="list-group-item">
      <a data-toggle="collapse" href="#collapseq3">
       <b>
        Q1.
       </b>
       Why should we calculate the
                                physical address of userSP-3 and userSP-1
                                seperately instead of calculating one and adding/subtracting the difference from the
                                calculated value?
      </a>
      <div class="panel-collapse collapse" id="collapseq3">
       Suppose the physical address corresponding to logical address in userSP be - say 5000.
                                it may not be the case that 4997 is the physical address corresponding to the logical
                                address
                                userSP-3. Similarly the physical address corresponding to userSP-1 need not be 4999.
                                The problem
                                is that the stack of a process spreads over two pages and these two physical pages need
                                not be contiguous.
                                Hence, logical addresses which are close together may be far separated in physical
                                memory.
      </div>
     </li>
    </ul>
   </div>
  </div>
  <br/>
  <br/>
  <p>
   <b style="color:#26A65B">
    Assignment 1 :
   </b>
   Write a program to print the first 20 numbers and
                        run the
                        system with timer enabled.
  </p>
  <a data-toggle="collapse" href="#collapse10">
   <span class="fa fa-times">
   </span>
   Close
  </a>
 </div>
</div>