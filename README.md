# File-Permission-Check-in-Windows
The loophole in C drive permissions to exploit
While installing applications (thin/thick client) in windows, we missed to check the following loophole to exploit it.
When a user install an application in any windows versions, it asks for the location (path) to install the software. The C: drive is considered as the primary hard drive of the system containing operating system as well as other system files. When we install any piece of software, by default it will installed in C: drive. However, when enough disk space is not available in C:drive,  user may prefer the installation in a different drive. This led to the evolution of a new feature which allows the user to select the installation path. This writeup is about a scenario where this feature can turn into a security bug.
For a better understanding we need to know about the Access Control List of C: drive
 

So from the screenshot it is understood that an authenticated user has the permission to modify since they were having Modify (M) access. And If we check the permission of sub directory inside C: drive, are having different Access Control structure and is given below:
 
This indicates that the sub directories in C: drive has different ACL rather than inheriting the ACL from parent directory(in this case C:drive), so that an attacker cannot modify its contents since there accesses are restricted to modify the contents.
The ACL of C: drive is more of like feature which allows any user can access data but creating new folder inheriting the ACL of C: drive could further lead to security bugs like DLL injection, information disclosure, privilege escalation etc.
On basis of this feature in windows, when a user installs a software in the C: drive, the installed folder inherits the ACL of C: drive. An attacker who has access to the guest account can modify or copy the data from the installed folder, which could lead to security issues like privilege escalation or information disclosure.
Here for eg: Consider installing a software Gpg4win in C: drive, the feature allows us to select the installation path to C:drive and the Gpg4win folder inherits the ACL of parent directory which is C:drive. So C:drive and Gpg4win has same ACL.
 
So here we can replace the kleopatra executable with calculator executable.
POC:
1. Install Gpg4win in C: drive from user account A
2. Open the user account B in the same system and replace the kleopatra executable in (C:\Gpg4win\kleopatra.exe)  with calc.exe in the name of kleopatra.
3. Open the user account A and execute the Kleopatra desktop app, a calculator will pop up.
 
Briefing of Bug:
The reason behind this ACL issue is, when the installer calls the CreateDirectory function, the security attribute is set to NULL. Thus the new directory gets a default security descriptor and the ACLs in the default security descriptor  are inherited from its parent directory (C:drive). This can be fixed by setting the correct security attribute based on the path being chosen or making mandatory to install in Program Files folder.


Reference Link
1.	https://www.techrepublic.com/article/windows-101-know-the-basics-about-ntfs-permissions/
2.	https://python-security.readthedocs.io/security.html#unsafe-python-2-7-default-installation-directory
3.	https://docs.microsoft.com/en-us/windows/desktop/api/fileapi/nf-fileapi-createdirectorya
