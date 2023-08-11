.. _hana_sharedmemory:

####################
HANA Shared Memory
#####################

`http://hanadba.blogspot.com/p/shared-memory-usage-in-hana.html`

Contrary to heap memory, which is allocated using the 'malloc' system call, shared memory is provided using the 'shmget' call. The results of shared memory creation can then be viewed on OS level using the ipcs command. The following approach can be used to display the shared memory of one particular HANA process on operating system side:
Get the process pid: ps -ef | grep
ipcs -p | grep then displays all segments that were created by this particular process:
Example: ipcs -p | grep 4221
86999065  hanadm  4221   4221
87064602  hanadm  4221 4221
87130139  hanadm  4221 4221
The size of a particular segment can then be further examined using the command ipcs -m -i
ipcs -m -i 86999065 
The sum of all those shared memory segments is then equivalent to the output of the statement:

SELECT SHARED_MEMORY_ALLOCATED_SIZE FROM M_SERVICE_MEMORY
WHERE PROCESS_ID = '4221'

Shared Memory Management:

when we can clear the shared memory ?
only the time when the db is down or not coming up..

You call cleanipc when the SAP instance is stopped and NOT when the system is up and running, which will crash the system.

The cleanipc command is used to clear the shared memeory occupied by the SAP system.
Mostly, this command is used (by adm user) while restarting an SAP system.So, you stop the system using stopsap, run the cleanipc command to clear the memory and then start the system using startsap.

The syntax to run cleanipc is:
First switch to the sidadm user

To list the shared memory segments:
Command: cleanipc   show
Example: cleanipc 02 show

To remove the shared memory segments:
Command: cleanipc remove
Example: cleanipc 02 remove

However, sometimes, cleanipc fails to cleanup some semaphores saying: Semaphore Key: XX remove failed **** - errno = 1 (Not owner) Command: cleanipc 06 remove

In this case, login as root and run the following commands:
ipcs –s | grep “the number from the semaphore”
ipcrm –s “the number from the above command”
The above command ipcrm clears up

cleanipc –remove all
 ipcs –m|grep
 ipcrm –m
 ipcs –s|grep 
 ipcrm –s

or use the command ipcs grep ,cleanipc all
