# Fix Can't Connect to File Share Obsolete SMB1 protocol
While accessing a remote file share, I got the error You can’t connect to the file share because it’s not secure. This share requires the obsolete SMB1 protocol. I got this error on my Dell Precision laptop after I installed Windows 10 1903 on it.

In addition to that above error, the fix mentioned in this post also applies if you see any of the below errors while accessing file shares.

*   The specified network name is no longer available.
*   Unspecified error 0x80004005
*   System Error 64
*   The specified server cannot perform the requested operation.
*   Error 58

Obsolete SMB1 protocol
----------------------

So why did I get this error ?. The reason is that SMBv1 protocol is now obsolete. Microsoft strongly advises consumers to use SMB2 or higher protocol.

In [Windows 10 Fall Creators Update](https://www.prajwaldesai.com/best-features-of-windows-10-fall-creators-update/) and later versions, the Server Message Block version 1 (SMBv1) network protocol is no longer installed by default. It is superseded by SMBv2 and later protocols starting in 2007.

Hence due to the above reason, I got the error while accessing a share from my Windows 10 machine. Furthermore if you install Windows 10 Enterprise 1903, it no longer contains the SMBv1 client. However SMBv1 can still be reinstalled in all editions of Windows 10.

Here is the complete error message – You can’t connect to the file share because it’s not secure. This share requires the obsolete SMB1 protocol, which is unsafe and could expose your system to attack. Your system requires SMB2 or higher. For more info on resolving this issue, see: [https://go.microsoft.com/fwlink/?linkid=852747](https://support.microsoft.com/en-us/help/4034314/smbv1-is-not-installed-by-default-in-windows)

[![](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap1.png)
](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap1.png)

Cannot Connect to File Share
----------------------------

To quickly fix the error You can’t connect to the file share because it’s not secure :-

*   On your computer, open **Control Panel**. Click Programs.
*   Click on Turn Windows features on or off link.
*   Expand the SMB 1.0/CIFS File Sharing Support option. Check the box **SMB 1.0/CIFS Client**.
*   Click the OK button.
*   Restart the computer now.

An alternate method to enable SMB1 Protocol is via PowerShell. Here is how you do it.

On your Computer, open the PowerShell and run the below command. This command gives you details about SMB1Protocol.

Get-WindowsOptionalFeature –Online –FeatureName SMB1Protocol

[![](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap2.jpg)
](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap2.jpg)

Run the below command to Enable SM1Protocol on your computer.

Enable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol

After you execute this command, you must restart your computer. After you restart, login to the computer and you shouldn’t see the file share access error again.  
[![](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap3.jpg)
](https://www.prajwaldesai.com/wp-content/uploads/2019/07/You-cant-connect-to-the-file-share-because-its-not-secure-Snap3.jpg)

![](https://www.prajwaldesai.com/wp-content/uploads/2020/07/cropped-PD-Logo-90.png)

Prajwal Desai is a Microsoft MVP in Intune and SCCM. He writes articles on SCCM, Intune, Windows 365, Azure, Windows Server, Windows 11, WordPress and other topics, with the goal of providing people with useful information.