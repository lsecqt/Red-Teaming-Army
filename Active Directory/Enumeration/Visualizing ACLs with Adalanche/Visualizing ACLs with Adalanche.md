# Visualizing ACLs with Adalanche 

I was always a fan of trying new tools in order to create a personal arsenal for edged cases. Recently, I found a tool called [Adalanche](https://github.com/lkarlslund/Adalanche), which is capable of enumerating and visualizing [ACLs](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) between entities in the scope of the [Active Directory](https://en.wikipedia.org/wiki/Active_Directory).

Usually, Active Directory misconfigurations can be found within the ACLs, and they can often lead to obtaining domain administrative privileges by chaining several lateral movement and privilege escalation techniques together. A very simple example for that can be a vulnerable [ADCS](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/active-directory-certificate-services-overview) server to [ESC1](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/) attack. 
Another example could be finding out that the current owned user is local administrator on some machine and after data exfiltration, you find domain admin credentials.

Mapping such attack vectors can be complicated without such tools, and while you should not be dependant of them, they are here to help, and they certainly do!

While [BloodHound](https://github.com/BloodHoundAD/BloodHound) is my rank #1 tool for enumerating and visualizing the Active Directory, I was also thrilled to try Adalanche, mainly because of curiosity in terms of UI, practical use and evasiveness. Turned out that this tool might be a hidden gem! 

I already deployed a video about this topic on my [channel](https://www.youtube.com/watch?v=PG2J0uILL1Q), so if you prefer watching a video instead of reading, feel welcomed!

Also make sure to join my [Discord](https://discord.gg/bgSpdheEgu) where we share experience, knowledge and doing CTF together.

And if you have further appreciation for my work, don't hesitate to become my [Patreon](https://www.patreon.com/Lsecqt)!

# Why not just use BloodHound?

Now here comes the question, why bother with Adalanche when I have BloodHound?

The answer is very simple, it is always a good idea to have alternatives for specific tools. Also, alternatives creates competition and this is a fundamental process of improving both of the sides, so its a win = win situation.

Additionally, as you might already know, [SharpHound](https://github.com/BloodHoundAD/SharpHound) (The data collector for BloodHound) is extremely signatured by various security mechanisms. I am aware that the signatures and the behavioral detections can be bypassed but sometimes its not a trivial process. For Instance it is possible to land into an environment that is extremely well network segmented, so that you cannot get a C2 implant to run or you should rely on some kind of bind shells on specific ports, which not all C2 framework actually supports. Additionally, if the segmentation is combined with enforced endpoint protection, it can become super challenging to execute SharpHound and collect data.

On the other hand, Adalanche is a tool that can work as both a collector and a visualizer at the same time, so let's get an idea of what it actually looks like.

# Adalanche Overview

Adalanche is go-written tool for collecting and analyzing data from Active Directory. It is capable of extracting potential attack vectors such as [unconstrained delegation](https://lsecqt.github.io/Red-Teaming-Army/active-directory/unleashing-the-power-of-unconstrained-delegation/), outdated servers, users with administrative privileges and more. It is extremely fast and compatible with each modern Operating System (OS).

One of the coolest features about Adalanche is that it is self-sufficient, which means, you do not need:
- Database (like Neo4j)
- Specific engine or runtime installed (like dotnet runtime)
- Additional software (like a web server)

All you need is the [compiled binary](https://github.com/lkarlslund/Adalanche/releases) and luck that you are in a vulnerable environment.
Since the Adalanche is go-written, the same code can be compiled for both windows and *nix systems. 

!!!
It is always a good idea to obfuscate the code and compile it yourself. Currently the tool does not get signatured but most likely this will change in the near future!
!!!

Adalanche can be run directly, with no arguments if it is launched from domain joined windows machine. On the other hand it can also mimic [bloodhound.py](https://github.com/dirkjanm/BloodHound.py), scraping the Active Directory if correct credentials and env data is supplied. In both of the cases, Adalanche stores the gathered data in a folder called, ```data```. Now let's analyze the different methods on how to get it running!

## Case 1: I am operating from a domain joined computer

In this scenario it is enough to just download and execute the binary. This will perform all the scraping automatically, host it on ```127.0.0.1:8080``` and navigating your default browser to the web view.

If everything went smooth, you should see something like this:
