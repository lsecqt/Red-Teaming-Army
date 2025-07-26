# Setting up hMailServer as internal mail server
 
Did you ever wanted to have an internal mail server so you can setup and test your Phishing infrastructure? I got you here! 

If you prefer the video version of this documentation, including live debugging, feel welcomed:

[!embed](https://www.youtube.com/watch?v=_5w0AmwQBzI)

 !!!
 Important:
 Special thanks to all of my Patreon supporters.
By supporting my work on [Patreon](https://www.patreon.com/Lsecqt) you get access to internal documentation, private tools and videos.
!!!


## Installing hMailServer

My environment looks like this:

![[Pasted image 20250726190910.png]]

Here, the `lsec.local` domain is inside a dedicated Active Directory and network managed by the PFSense firewall and yes, everything is virtualized from the `PC`. I did not want to shoot up a new server just for mail purposes (mainly because of resources) but rather use one of the DCs. In this case, `DC01`.

!!!
The setup I will provided is okay for testing and local labs. This is not a setup you should go for on production environment. The whole idea is to easily setup a server from which we can later send emails and phishing campaigns!
!!!

So first things first, lets download hMailServer: https://www.hmailserver.com/download

It does not matter where you will download it. The installation steps are super trivial (next -> next -> finish). By default, hMailServer will be installed in `C:\\Program Files(x86\\hMailServer`

![[Pasted image 20250726191208.png]]

The next step is important if you want to use a dedicated database for it, however I went for the first option because of simplicity. This will setup a small database onto the DC itself, which hMailServer will constantly use.

![[Pasted image 20250726191300.png]]

Then, you need to setup your password for accessing the hMailServer admin console:

![[Pasted image 20250726191351.png]]

And then you are done! If everything went fine (and I am sure it will) you should see this:

![[Pasted image 20250726191422.png]]

Here, you can edit the default `localhost` connection or add a new one. I used the `DC01` as a hostname since this is the real hostname of my Domain Controller:

![[Pasted image 20250726191445.png]]

After you login to the console, you should configure a domain. My internal AD domain is called `lsec.local` so I will use the same name here:

![[Pasted image 20250726191544.png]]

After you add your domain, on the left pane, you should see the `Accounts` tab. From there we can either add a local mail account or use present AD account. I found hMailServer to be extremely easy when it comes to account creation and authorization concepts:

![[Pasted image 20250726191634.png]]

Here, in order to add an AD account, simply right click onto the Accounts tab -> Add AD account:

![[Pasted image 20250726192027.png]]

Then simply specify your Domain and the corresponding user account:

![[Pasted image 20250726192100.png]]

After this step, it is recommended to go to your DNS settings (on the DC) and add an MX record for your mail server:

![[Pasted image 20250726191750.png]]


## Installing Email Client

For the purpose of this demo, I use ThunderBird because of its simplicity, but feel free to improvise. You can download ThunderBird from here: https://www.thunderbird.net/en-US/thunderbird/all/ and it can run on any platform. Again, the installation of ThunderBird is trivial. After installing, you can then try to connect using the local or AD account you created during the previous steps:

![[Pasted image 20250726192718.png]]

!!!
Since you already added the MX record up to this point, the client machine should automatically know where to bind to. Missing the DNS steps might cause issues.
!!!

If you are unable to connect using your mail client and see something like this:

![[Pasted image 20250726191911.png]]

Then its highly possible that you are auto-blocked from the hMailServer. To resolve it, navigate to hMailServer admin console -> Settings -> Advanced -> IP Ranges. There you will see if there is a lockout and you can simply delete it:

![[Pasted image 20250726192258.png]]

It is also advised to create a new IP Ranges rule to allow emails from specific hosts or subnets:

![[Pasted image 20250726192339.png]]

Now to test the email server and clients, I will use my kali and the `sendemail` utility. Ideally for future phishing campaigns I will setup GoPhish (https://github.com/gophish/gophish) to handle that, but here we are not yet talking about phishing, but rather the infrastructural perspective.

In order to send emails from `sendemail`, you can use the following snippet:

```
sendemail -f <from@domain.com> -t <to@domain.com> -s <DomainServer IP / DNS> -xu <username> -xp <password>
```

![[Pasted image 20250726193039.png]]

And finally, we can send / receive emails:

![[Pasted image 20250726193016.png]]

Hope this was useful, if so consider a [sub](https://www.youtube.com/@Lsecqt) or a [support](https://www.patreon.com/c/Lsecqt)

Once again, special thanks to all of my Patreons!