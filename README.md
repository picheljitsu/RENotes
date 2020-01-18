# RENotes
the current process can be queried as ? @$proc

kd> ? @$proc
Evaluate expression: -2050188616 = 85cc9ab8
kd>
the single question mark ? represents masm expresssion evaluator you can use ?? question marks to turn on c++ expression evaluator

kd> ?? @$proc->UniqueProcessId
void * 0x00000538
kd>
the above Current process pid is also represented by @$tpid

kd> ? @$tpid
Evaluate expression: 1336 = 00000538
kd> ?? @$tpid
unsigned int 0x538
kd>
the pcr is represented by @$pcr

you can mix and match the expression evaluator the @@() lets you insert a c++ expression into a masm evaluation

kd> ? @$pcr
Evaluate expression: -2097779712 = 82f66c00
kd> ?  @@(@$pcr->PrcbData.CurrentThread)
Evaluate expression: -2048806120 = 85e1b318
kd>
in my system PsGetCurrentProcess is as follows

kd> uf nt!PsGetCurrentThread
nt!PsGetCurrentThread:
82e72b99 64a124010000    mov     eax,dword ptr fs:[00000124h]
82e72b9f c3              ret
kd>
you can directly get the raw contents of this segment:offset

kd> ? poi(fs:00000124)
Evaluate expression: -2048806120 = 85e1b318
kd>
the current thread is denoted by @$thread

kd> ? @$thread
Evaluate expression: -2048806120 = 85e1b318
kd>
ApcState is not an array it is a structure

kd> ?? @$thread->Tcb.ApcState
struct _KAPC_STATE
   +0x000 ApcListHead      : [2] _LIST_ENTRY [ 0x85e1b358 - 0x85e1b358 ]
   +0x010 Process          : 0x85cc9ab8 _KPROCESS
   +0x014 KernelApcInProgress : 0 ''
   +0x015 KernelApcPending : 0 ''
   +0x016 UserApcPending   : 0 ''
kd>
you can get the offset to Process mention in you post like this

kd> ?? &(@$thread->Tcb.ApcState.Process)
struct _KPROCESS ** 0x85e1b368
kd> ?? *(unsigned long *)&(@$thread->Tcb.ApcState.Process)
unsigned long 0x85cc9ab8
kd>
in my system PsGetCurrentProcss is as follows

kd> uf nt!PsGetCurrentProcess
nt!PsGetCurrentProcess:
82ec5fce 64a124010000    mov     eax,dword ptr fs:[00000124h]
82ec5fd4 8b4050          mov     eax,dword ptr [eax+50h]
82ec5fd7 c3              ret
kd>
raw query

kd> ? poi(30:124)
Evaluate expression: -2048806120 = 85e1b318
kd> ? poi(poi(30:124)+50)
Evaluate expression: -2050188616 = 85cc9ab8
kd>
expression query

kd> ? @@(@$prcb->CurrentThread->ApcState.Process)
Evaluate expression: -2050188616 = 85cc9ab8

***********************************************************
WinDbg already displays the name of exceptions when it knows it:

(15c0.1370): Break instruction exception - code 80000003 (first chance)
You get more details with .exr -1:

0:009> .exr -1
ExceptionAddress: 77d5000c (ntdll!DbgBreakPoint)
   ExceptionCode: 80000003 (Break instruction exception)
  ExceptionFlags: 00000000
NumberParameters: 1
   Parameter[0]: 00000000
You can also display NTSTATUS codes as proposed by @rrirower:

0:009> !gle
LastErrorValue: (Win32) 0 (0) - The operation completed successfully.
LastStatusValue: (NTSTATUS) 0 - STATUS_WAIT_0
And these status codes can be decoded with !error. It will consider Win32, Winsock, NTSTATUS and NetApi errors:

0:009> !error 0000071a 
Error code: (Win32) 0x71a (1818) - The remote procedure call was cancelled.
***********************************************************
