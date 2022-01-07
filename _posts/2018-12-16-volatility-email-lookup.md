---
layout: post
title:  "Volatility E-mail Lookup"
date:   2018-12-16 15:03:36 +0530
categories: python
excerpt: "Find existing e-mail addresses from a process dump by combining Volatility & HaveIBeenPwned (Memory Forensics)"
image: "/images/memoryforensics.jpg"
---

Keywords: *Volatility, HaveIBeenPwned, Python*




# Linux Process E-mail Grabber
## About
During my semester abroad at LSU (*Geaux Tigers!*) I also had to do some serious course work. I enrolled for the Memory Forensics course taught by [Golden](https://www.cct.lsu.edu/~golden/), where I have learned some real deep stuff regarding memory forensics with the Volatility Framework.



## Why use this plugin?
This plugin can be used to scan memory for e-mail addresses, and determine e-mail age by matching it to the database of [HaveIBeenPwned](https://haveibeenpwned.com/). The reasoning here is that most (active) e-mails have probably been leaked at least once. This can be useful, as some malware could send details of keyloggers, etc. to a particular e-mail address. Indications of breaches could possibly determine the usage of an e-mail address, and could estimate (based on the date of leaks) how long a particular e-mail address has been around.

## Usage

1.  Install your volatility profiles for your system you want to analyze (download into:  
    `/volatility/plu‌​gins/overlays/linux)`

2.  install the python dependency ratelimit via the command line using pip:  
    `pip install ratelimit`

3.  Unpack the ZIP file and put the .py files in a directory.
    
    
4.  When running Volatility, specify the plugin directory with `--plugin=[PATH]`
    
    
5.  Run *ps_email_lookup* with the optional parameters `-p [process]` and `-D [dump directory]` in order to specify the process ID and dump directory, specifically.

Example:
```bash
python vol.py --plugins=/vagrant/plugin  --profile=LinuxXubuntu1404x64 -f /vagrant/NILES.lime ps_email_lookup -D /vagrant/dump
```
Output:
```bash
Volatility Foundation Volatility Framework 2.6
PID     E-mail                      Date            Breaches
4256    bash-maintainers@gnu.org    2017-08-28      2
```


### Parameters
- **-D (dump)**
    Directory to dump the process in.

    Example:
    ```bash
        -D /vagrant/dump
    ```
- **-p (process)**
    Process number(s) to limit the search to. Recommended if there are many processes running, or it will probably run out of memory and crash!

    Example:
    ```bash
        -p [102,233,2304]
    ```




# Timeline
## Setup
I started out with downloading a virtual machine, but ended upon multiple blue screens thanks to my Windows installation.. After disabling Hyper-V I needed to re-install virtualbox in order to fix this problem.

[Vagrant](http://vagrantup.com) is used to quickly setup virtual machine environments.
I use a vagrant box configured by *blu3wing* in order to get the framework running. This is basically a Linux environment with the right python version & dependencies installed in order to run volatility successfully.

Get vagrant running by creating the following *Vagrantfile*:
```bash
Vagrant.configure("2") do |config|
  config.vm.box = "blu3wing/dreamcatcher"
  config.vm.box_version = "2"

  config.vm.synced_folder "shared/", "/vagrant", create: true

  $script = "pip install ratelimit"
  config.vm.provision :shell, privileged: true, inline: $script

end
```
In the shared folder */shared*, I can put all our memory images and code, which gets synchronized with our box.

Pop up a bash shell, boot up the VM, and SSH into the box:
```bash
vagrant up
vagrant ssh
```
(To stop the VM after development, use `vagrant suspend`.)

Now that I are SSHd into the box, I can test it by going into the installation directory and running Volatility:
```bash
cd /opt/volatility #<-- Location of Volatility
python vol.py --info
```
## Development
The objective is to search the memory dump (or a process) for e-mail addresses, and send the results to an API. I separate this into two tasks: memory search & API Calling.

### Memory search
How do I analyze the memory to get the e-mail addresses?  A widely used plugin to find malware samples is the YARA scan. YARA scans the memory in 1MB chunks for predefined patterns. In our case, I can't really use YARA scan, as this method does not allow using regular expressions. This would leave us  only one way to identify an e-mail address using this method, which is by searching for the character `@`. 
However, I can still use Regular Expressions to search for e-mail addresses. The downside of this is that I need to dump the whole process, and run an exhaustive RegEx search on this. I chose to pursue this route, and thus I can further divide this problem in subtasks:

1. Process dumping (either to memory or to a file)
2. Searching a dump (for e-mail addresses)


#### Process dumping
Before I can dump a process, I need to find its memory location. To do this, I can use another plugin that has already been developed: *linux_pslist*! 

A problem I Ire stuck on was on how to call another plugin from your own, as documentation to create your own plugin was basically non-existent. So let's dive into the core of the plugins and do some debugging..

In some plugins, *unified_output* did some calculations (which should not be used like that!), but I found it hard to understand how Volatility's *TreeGrid* generator worked. Luckily, the *linux_pslist* plugin did only do their calculations in the *calculate* function.
I found out that by just importing the desired plugin and then calling its *calculate* function will return the variables I need! See below:
```python
tasks = linux_pslist.linux_pslist(self._config).calculate()
```

Now that I got the processes in a variable, let's dive deeper into the process dumping itself. I look at *volatility/plugins/procdump.py* in order to figure out how to dump a process and keep it in memory instead of dumping to a file. 
Below I see how *procdump.py* handles writing memory to a file:
```python
file_path = linux_common.write_elf_file(self._config.DUMP_DIR, task, task.mm.start_code)
```
When I continue to investigate how the *write_elf_file* function works, I stumble upon this (in *volatility/common.py*):

```python
def write_elf_file(dump_dir, task, elf_addr):
    file_name = re.sub("[./\\\]", "", str(task.comm))
    
    file_path = os.path.join(dump_dir, "%s.%d.%#8x" % (file_name, task.pid, elf_addr))
    
    file_contents = task.get_elf(elf_addr)
    
    fd = open(file_path, "wb")
    fd.write(file_contents)
    fd.close()
    
    return file_path
```
This gives us the information that the *task* struct contains the *get_elf* function, which would represent the contents of the file. I can directly use this function to grab a dump of the process!

#### Searching a dump
For debugging purposes, I wanted to know how some interesting strings looked like. I define interesting strings as strings with at least 4 characters.  Function to convert binary to interesting strings:
```python
def binaryToString(self, content):
    #print all elements that have at least 4 characters
    contentMatch = re.findall("[^\x00-\x1F\x7F-\xFF]{4,}", content)
    flatten = " ".join(str(x) for x in contentMatch)
    return flatten
```
Then the only thing remaining is finding e-mails from all those strings. I got the Regular Expression pattern from [here](https://www.tutorialspoint.com/python/python_extract_emails_from_text.htm). 
```python
def emailSearch(self, rawStrings):
    # Regex pattern to grab substring emails from a string
    # Copyright: https://www.tutorialspoint.com/python/python_extract_emails_from_text.htm
    pattern = r"[a-z0-9\.\-+_]+@[a-z0-9\.\-+_]+\.[a-z]+"
    return re.findall(pattern, rawStrings)
```


### API Calling
The last part is about calling the [HaveIBeenPwned](https://haveibeenpwned.com/) API to check for breached e-mails. I made sure to strictly enforce API limits by integrating the [ratelimit
](https://pypi.org/project/ratelimit/) plugin by Tomas Basham (make sure to install this first!). The API is limited to one request every 2 second, so I set this up with prepending:
```python
@sleep_and_retry
@limits(calls=1, period=2)
def lookupBreachAPI(self, email):
    ...
```
Using *requests* I can call the API for results regarding an e-mail address. If there is no entry, I get a 404 HTTP status code.
If the e-mail does exist in their database, I get a response like this (in JSON):
```json
[ { 
...
"Domain":"adobe.com", 
"BreachDate":"2013-10-04",
...
} ]
```
I collect all the breach dates (*BreachDate*), and try to sort them by date in ascending order. I wrote a function to sort the date by year, month and day:
```python
def dateSort(self, apiObject):
    splits = apiObject['BreachDate'].split('-')
    return splits[0], splits[1], splits[2]
```
By replacing the breaches with a sorted version, I now have them ordered:
```python
breaches = sorted(breaches, key=self.dateSort)
```

## Repository

The repository for this project can be found here:

**[Github - volatility-email-lookup.py](https://github.com/Rubenkl/volatility-email-lookup.py)**