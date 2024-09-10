# Shell-Project

Tom Landzaat
Operating Systems I
Aug. 11, 2024

Abstract 

The BigShell project aims to incorporate built-in commands, job control and signal handling to a Unix-based shell. This project is coded in C and follows the 1-2008 POSIX standard. 
This shell is designed to be user friendly and contains many safeguards to prevent unexpected program behavior given bad user inputs. There are also numerous lines of error checking logic to prevent non graceful program exit upon a failed process.
Completion of the project includes passing all tests provided in gradescope. This includes testing commands, changing signal disposition, proper management of jobs and more rigorous tests. Not only does the implementation of each function have to be correct, a deep understanding is required to connect how all functions are related. 
Key functionality in the built-in commands section include, cd, exit and set/unset for variable modification. Likewise for signal handling, key functionality includes managing signals like SIGINT and SIGTSTP, ignoring interrupting signals when desired and being able to interrupt system calls such as read(). Lastly, the key functionality of job control includes determining when to fork, managing foreground and background processes and keeping track of all process, group, and job ids. The following report goes into more detail on the design and implementation of the BigShell assignment as well as hurdles overcome along the way.
 
I. INTRODUCTION

The scope of the BigShell project is to research and implement shell functionality based on the skeleton code provided by Ryan Gambord. The key objective of this project is coding said functionalities in C and passing all tests provided in Gradescope. C is the language of choice given its expansive collection of libraries and ability to go low level into memory management and allocation. The chosen code editor to complete this project was Github codespaces. This tool allows for remote connection to the codebase from any device, syncing of the codebase and embedded Git actions. 

The motivations for this project are to gain experience programming in C and specifically on operating systems. This project links together all of the operating systems theory we have learned for the past 8 weeks and challenges our practical implementation. Interactive shells are a handy tool to work with and important to understand. By building shell functionality given Ryan’s skeleton code to help us along, the goal is to have a deeper understanding based on the hands-on nature of the project. More specifically to have a better understanding of job control, forking, signal handling, pipelining and error handling. All of these topics are commonplace for computer scientists, especially those in the field of operating systems and embedded systems.

An objective of this project is to stray away from functions that are not best practice or are known to have vulnerabilities. Some examples include the known buffer overflow exploit when using scanf, a function to scan for user input [11]. Other examples include using portable functions from external libraries that will work on many different systems and behave the same across all of them. One bad example is signal() from the <signal.h> library, on different versions of Unix, it behaves differently [6]. And likewise, when programming our own functions, ensuring they are versatile, readable, and keeping hard coded sections to a minimum.

Although shell scripting is not as common as it was in the 1980 due to the increase of guis, it still has some advantages. According to experts in the field, guis provide the better visual format and are more user friendly, however, experienced users can more quickly navigate a shell and its line interface consumes less system resources [10]. Many populars shells today include the Bash shell (Bourne again shell), the Zsh shell and the Fish shell. Although the BigShell implementation may be unique, the concept of Unix shells is no new idea and many solutions already exist to meet user’s needs. 

The report that follows is structured to provide cohesive documentation of the BigShell project process. The report starts with a detailed description of the program structure, then is followed by an in-depth account of the program implementation and finally it addresses any challenges encountered along the way and the solutions to overcome them. 



II. PROGRAM STRUCTURE

The BigShell program consists of many files; reference, src, the makefile and a ReadMe. The reference file contains an executable file that matches all of the desired criteria and functionality. This executable file was helpful in the programming process to verify what the correct outputs should be. Inside the src file is where all of the program files (.c) and header files (.h) are located and where all of my changes were made. To compile the program one simply changes their working directory to the one that contains this file and types “make” into the terminal. The make process is further discussed in the implementation section but for now the important part is an executable file named bigshell is created in the same directory. To run this program type ./bigshell into the terminal and the program is launched. The program's functionality is based on Unix shells so many familiar commands will work in the program like cd, exit, ls and more.

One section of the program that is crucial to the functionality of the shell is signal.c. In this file, all of the signal handling is managed such as ignoring signals, ensuring signals interrupt system calls and restoring signals back to their old disposition after changing them. This is accomplished with the sigaction() function which is built into the <signal.h> library. This process can also be accomplished with the signal() function, however, it is known to be less portable as its behavior varies by the OS [6]. When using the sigaction() function, it accepts a signal, an action struct which contains the new action and an action struct that contains the old action. If the user wants to ignore the SIGINT signal and store the old behavior, like when the BigShell is running and there are no active jobs, it can be accomplished by filling out the sigaction() function. The function would look like: sigaction(SIGINT, &ignore_action, &old_sigint). To add a further level of verification we can check if the sigaction call succeeded; -1 signifies a fail and 0 is success. We can check if the return value of sigaction is -1 and if so, print out the errno and exit the program.

Moving on to the most important part of the program and the entry point, the main() function [1]. The main() function is located in bigshell.c and when the program is run, the lines in main() are executed in order and call functions declared in all the other program files. The main() function includes lots of error checking as soon as the program starts, including successful library includes and signal handling initialization. After initial checks, the main loop of the program starts, where the shell waits for an input until the user provides one. After the user provides an input, the program will parse the input and verify if it is a recognized command and if so call it, otherwise print an error. During specific stages of command execution the shell will handle signals differently given implementation of signals.c, such as ignoring the signal or interrupting the command execution. 

In order to correctly parse the information provided by the user, parser.c needs to be fully implemented. Parser.c breaks the input into tokens which are either words or operators. The program then tries to match tokens to commands and then stores them as a command struct in a list waiting to execute. The variables that make up a command struct include its name, arguments, I/O redirections and the control operator. In the BigShell project, the control operator is specified at the end of a command and are as follows: ‘:’ specifies that a command should run in the foreground, ‘&’ specifies it should be run in the background and ‘|’ specifies its output should be piped to another command input, also called pipelining. 

Next, the program executes the commands in the list in order and with their respective control operator. First, the program checks for any current pipelines then if any pipelines are needed by this command. Then runner.c will determine if the process is neither a foreground command nor a builtin command and fork the process. If the process is forked, it will re-assign the child pid to the new process id.

Job control, specified in jobs.c and wait.c is the next crucial link in the chain. Job control manages multiple processes that are happening at once by assigning all jobs a unique job id and stopping and resuming jobs as needed [13]. Job control also verifies the output of completed jobs to ensure success and the exit status of the last job is stored in a shell variable.


III. IMPLEMENTATION DETAILS

The main resource used for information on this project was Ryan Gambord’s textbook "BigShell Specification." This text included detailed information about operating systems basics, the C language, how to manage and store data, and processes. In addition to delving into the shell command language and POSIX standards, Ryan provided code snippets for example projects. These code snippets highlighted how to use certain functions and the correct syntax for operations. This bridged the gap between theory and application and was a life saver for the how-tos upon implementation. In addition to Ryan Gambord’s text, Ed Discussion was a great resource for connecting with fellow students and relating to similar hardships among students. There is no better feeling than having the exact same frustrating issue as someone else and being able to figure out together what the issue is and solving it. A similar resource posted on Ed was the OSU CS online discord where students could post their problems and get advice from fellow students or instructors. All of these resources proved useful but not as helpful as the office hours provided by TAs. Being able to talk through challenging functions with someone who has previously completed the assignment, greatly increased my understanding and ability to implement the correct solution.

As previously stated, Github was the chosen version control software due to its modern features.
Github provided the most seamless access to the updated codebase across multiple devices and had the convenience of Github codespaces. Github codespaces provided an IDE with Git integration and useful features like word autocomplete and syntax highlighting.

Another tool that assisted the development process was makefile. Makefiles allow for easy recompilation of programs, especially programs with multiple files. Instead of having to recompile each file when one change is made, the makefile compiles only the individual files that have updated. The result is an updated object file along with the still up to date object files from unchanged files. With all of these object files, the script in the makefile links all object files together to create one executable file. All of this happens behind the scenes if the user types make in the working directory of said files. 

For this project, the makefile was more complex to account for a debugging build and a release build. These can be performed with ‘make debug’ and ‘make release’ respectively. The main difference is that the debugging build has debugging flags turned on which allows for use of gdb and similar applications to trace through the code and ensure steps in the code are taken correctly. The release build on the other hand is optimized for performance and is desired for the final build. Another thoughtful feature is the ‘make clean’ target. When using the makefile, it compiles an object file (.o) for each .c file, and once linked, the object files sit around until the next time make is called. This can cause clutter in the repository or filesystem, but the ‘make clean’ call will remove all of the object files. 

Throughout this project there are many instances of error checking, even on simple functions such as getting the status of a specific job given the job id. This has proved to be extremely helpful with debugging and figuring out which section of code specifically is responsible for the incorrect behavior which causes tests to fail. In the above example, if we fail to get the status of a specific job and the program exits we now know that the job id is incorrect or does not exist in the program. 

Overall, the development of the BigShell project was a comprehensive process that involved using a variety of libraries, debugging, and lots of research. The project’s success was only possible with resources such as Github codespaces for development, Ryan Gambord’s text as reference and makefile for efficient recompilation.  The combination of these resources and a structured plan ensured that the BigShell project met its requirements and passed all Gradescope tests. 


IV. CHALLENGES AND SOLUTIONS

The main challenge for this project was understanding how all of the working pieces were connected as this is my biggest project to date. Lots of research was necessary as the program consisted of many different files and included many different libraries. Thankfully, the systems are well documented online and being able to brainstorm solutions with my teammate Carter helped my understanding and implementation. One difficulty along the way was working on wait.c, the program that deals with updating the variable “params.status.” This variable stores the exit status of the last job. After completion of the program, the tests were not completely passed. More specifically, when testing exit with the last job exit status if no arguments were used when exiting, just exit. This worked fine when the job exited normally by itself, however, I had some issues when the job exited after being signaled to do so. The first thought to troubleshoot was including many dprintf/gprintf statements with values of variables along the way. This worked well and showed that my implementation was not correct as some values printed were not as expected. After reading the man pages on the subject of built-ins in the <sys/wait.h> library and working with Carter, we discovered that jobs exited by a signal should exit with 128 + N [9]. The reason is better readability, for example exiting with 137 signifies that the program was terminated by a signal (128) and the signal was SIGKILL (9).

Another challenge faced during this project was the time commitment required. Working lots of hours this summer and working on the BigShell project meant I had to manage my time effectively. This could sometimes be hard since this project required lots of readings to even start understanding the skeleton code. After that point, implementing my own code was difficult and didn’t work so well which led to debugging and more reading on why my implementation did not work. After struggling for a bit, I made time in my schedule to attend office hours and I found it immensely helpful to discuss the problem I was having with someone knowledgeable on the subject. The TAs in office hours were very good about pointing students in the right direction and I realized it was something I was missing out on as this was my first online CS course.

For future projects of this size and caliber I learned that including lots of error checking and print statements along the way can greatly reduce the time debugging. Not only that but also, talking through the problem with a peer greatly helps my understanding when it comes to implementation. And for the best results, starting the project early, because it is initially a lot and can be stressful but it is very easy to pick up where you left off and keep tackling pieces of the project.

V. CONCLUSION

This program was extensive and stressful to process when starting the project. However, once started could be broken down into more manageable bite sized pieces. From analyzing the skeleton code to implementing my own functions, I have learned a great deal about the C programming language and the inner workings of a shell. This knowledge has many practical implementations in industry and it feels rewarding that I was able to complete a program that I had limited prior knowledge of how to solve before this course. In addition to gaining more experience working with C, I was introduced to the world of version control programs and learned how to use Git with a more extensive codebase.

If this project were to receive more funding the main objective would be to implement more shell features and more advanced tasks for advanced users. As it currently stands, the shell meets all of the required functionality, however, could be improved by more user testing and revisions. One thing the popular Fish Shell excels in is auto command suggestions and a vibrant colorful interface which increases readability [12]. These features appeal to all audiences but also specifically help novice shell users overcome the hurdle of the shell’s intricacies. Therefore these features should be prioritized given more funding for the BigShell project. 

Overall, the BigShell project has been a cornerstone experience in my C journey. It has not only challenged my knowledge and skills but now instilled confidence in my programming ability since such a complex project is now under my belt. Completion of this project takes a lot of knowledge yet this is only the tip of the iceberg in the operating system world, so my learning is far from done.


REFERENCES

[1]	R. Gambord, "BigShell Specification," Operating Systems I [Online], August 8 2024. Available: https://rgambord.github.io/cs374/assignments/bigshell/

[2]	IEEE Standard for Information Technology - Portable Operating System Interface (POSIX(R)), IEEE Std 1003.1-2008. doi: 10.1109/IEEESTD.2008.4694976.

[3]	D. M. Ritchie, and K. Thompson, "The UNIX time-sharing system," Commun. ACM vol. 17, no. 7, pp. 365-375, Jul. 1974. doi: https://doi.org/10.1145/361011.361061

[4]	Bash. (2024). GNU Project.

[5]     IEEE Std 1003.1, The Open Group Base Specifications Issue 6, https://pubs.opengroup.org/onlinepubs/009695399/basedefs/sys/wait.h.html. 

[6]     M. Kerrisk, “The Linux Programming Interface,” Signal(7) - linux manual page, https://man7.org/linux/man-pages/man7/signal.7.html. 

[7]     P. Koopman, “How to Write an Abstract,” How to write an abstract, https://users.ece.cmu.edu/~koopman/essays/abstract.html. 

[8]     M. Kerrisk, “The Linux Programming Interface,” Ctype.h- linux manual page, https://man7.org/linux/man-pages/man0/ctype.h.0p.html.

[9]     G. D. Maayan, “An overview of linux exit codes,” AgileConnection, https://www.agileconnection.com/article/overview-linux-exit-codes#:~:text=128%3A%20Invalid%20exit%20argument%E2%80%94Returned,by%20a%20fatal%20error%20signal.

[10]   A. Froehlich, “What are the advantages and disadvantages of CLI and Gui?: TechTarget,” Networking, https://www.techtarget.com/searchnetworking/answer/What-are-the-advantages-and-disadvantages-of-CLI-and-GUI. 

[11]   W. Du, Computer Security: A Hands-on Approach. CreateSpace Independent Publishing Platform, 2022. 

[12]     A. Kili, “Aaron Kili,” 5 Most Frequently Used Open Source Shells for Linux, https://www.tecmint.com/different-types-of-linux-shells/ (accessed Aug. 11, 2024). 

[13]	D. M. Ritchie, and K. Thompson, "The UNIX time-sharing system," Commun. ACM vol. 17, no. 7, pp. 365-375, Jul. 1974. doi: https://doi.org/10.1145/361011.361061



