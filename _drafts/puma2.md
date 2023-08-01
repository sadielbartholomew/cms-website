---
layout: page-fullwidth
title: puma2
teaser: Getting started with PUMA2
---

## PUMA2

We are moving to a new server PUMA2, hosted at ARCHER2, 
and jointly managed by CMS and EPCC.

Having PUMA2 hosted at ARCHER2 will make the system easier for users to navigate, 
and for CMS to manage. 
In particular, you will use the same account to access both systems. 

Upgrading the server also allows us to support forthcoming UM infrastructure
such as cylc-8, which will eventually be controlled via a web interface. 
Unfortuantely, the puma server we've been using recently was only an interim solution after the demise of our old machine,
and not suitable as a long-term replacement. 

***This is the current process for testing. I haven't yet tried copying my files over and using PUMA2 for real.***

### 1. Applying for an account 

If you don't already have an ARCHER2 account, 
follow [these instructions](https://docs.archer2.ac.uk/quick-start/quickstart-users/#request-an-account-on-archer)

To request access to PUMA2: 
* Login to [SAFE](https://safe.epcc.ed.ac.uk/)
* From the "Login accounts" menu, select your username
* Scroll down near the bottom, and select "Add Machine"
* At the bottom of the next page, under "Machine to join", select "puma2", and click "Join"

You will be sent an email when your PUMA2 account is ready for use. 

### 2. Logging in to PUMA2

PUMA2 is accessed from the ARCHER2 login nodes, and uses the same username and password. 

[Login to ARCHER2](https://docs.archer2.ac.uk/quick-start/quickstart-users/#login-to-archer2) as usual.

**Important: We recommend users ***do not forward their ssh-agent*** from their local system, 
as this can cause problems with Rose/cylc job submission.**

Once you are on the ARCHER2 login nodes, type ```ssh -Y puma2```. Enter your ARCHER2 password when prompted.
  
You should now be logged into to PUMA2. To go back to the ARCHER2 login nodes, type ```exit```

### 3. Setting up passwordless access

You can set up a passphrase-less ssh-key to acces PUMA2, 
so that you don't have to type your password every time.

**Important: You still need to use 2-factor authentication to access the ARCHER2 login nodes, 
with a strong passphrase protecting your ARCHER2 ssh-key**

* From the ARCHER2 login nodes, type 

  ```ssh-keygen -t rsa -C your@email.ac.uk -f ~/.ssh/id_rsa_puma2```

* At the prompt, press enter for an empty passphrase.

* Copy the key over to PUMA2:
  ```ssh-copy-id -i ~/.ssh/id_rsa_puma2 puma2```

* Next, create a file called ```~/.ssh/config```, and add the following lines: 
  ```
  Host puma2
        IdentityFile ~/.ssh/id_rsa_puma2
        ForwardX11 yes
  ````

* Test it works by typing ```ssh puma2```.
  Note that this should have set up X11 forwarding, so you no longer need the `-Y`.
  You should not be prompted for your password. 

### 4. Copying over your files 

***This is a placeholder***

### 5. Setting up your PUMA2 environment 

***Some of this will be added to user's enviornment centrally.***

To setup your environment, copy the standard ```.profile``` and ```.bashrc```. 
```
cd
cp ~um1/um-training/.profile .
cp ~um1/um-training/.bashrc . 
```

To pick up these settings, logout of PUMA2 and back in again (or source the files): 

You should be prompted for your MOSRS password, then username. 
To check this is setup correctly, see if you can access the UM repository: 
```
fcm info fcm:um.x
```

### 6. Setting up your ssh agent 

As on the old puma server, you need to have an ssh-agent running in order to submit jobs to 
ARCHER2 and JASMIN.

***Would we get users to copy over their old .ssh ?***

* Copy your ARCHER ssh-keys to PUMA2. ***How?***

* Copy the `ssh-setup` script to your `.ssh` directory.

  ```
  cp ~um1/um-training/setup/ssh-setup ~/.ssh
  ```

* Log out of PUMA2 and back in again to start the ssh-agent (or run ```. ~/.ssh/ssh-setup```)

* Add your ARCHER key to the ssh agent: 
  ```
  ssh-add ~/.ssh/id_rsa_archer
  ```
  Type your passphrase when prompted
  
* If it doesn't exist, create the file: ```~/.ssh/config```,
  and add the following lines:
  ```
  Host ln* 
     IdentityFile ~/.ssh/id_rsa_archer

  Host *
     ForwardAgent no
  ```

* To test this is working, run:
  ```
  rose host-select archer2
  ```
  It should return one of the login nodes, e.g. ```ln01```.
  If it returns a message like ```[WARN] ln03: (ssh failed)``` then something has gone wrong with the ssh setup. 


### 7. Setting up your ARCHER2 environment 

***Users will not need to do this***

***The default version of Rose and cylc are the same, 
so you can skip this step if you don't want to mess up your A2 setup.***

To test the PUMA2-managed Rose, cylc and FCM, edit your ```.bash_profile``` (or wherever), 
and replace this line: 
```
. /work/y07/shared/umshared/bin/rose-um-env
```
with this one: 
```
. /work/y07/shared/umshared/bin/rose-um-env-puma2
```

Check you are picking up the PUMA2 versions, with: 
```
which cylc
```
This should return 
```
/work/y07/shared/umshared/metomi/bin/cylc
```

### 8. Restarting suites 

***This is a placeholder***

### 9. UMUI 

***This is a placeholder***

Is anything point to ```~um``` or ```/home/um```? This is now ```~um1``` or ```/home/n02/n02/um1```.