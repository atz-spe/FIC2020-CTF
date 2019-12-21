# First Step for First Flag

When the game begin, we start the first challenge, we arrive on the CTF platform and an empty "Flag" field asks us to enter a response. 

![ch01 Platform](/images/ch01-platform.png)

When we click on play and we make a mistake, we have a "NOPE" alert.

### INSPECTION

First reflex at this time, inspect the source code of the page.  
By entering this one, we quickly see a <script> block containing javascript.  
Let's see its content below:

```javascript
const play = () => {
  var game = new Array(
    116,
    228,
    203,
    270,
    334,
    382,
    354,
    417,
    485,
    548,
    586,
    673,
    658,
    761,
    801,
    797,
    788,
    850,
    879,
    894,
    959,
    1059,
    1071,
    1140,
    1207,
    1226,
    1258,
    1305,
    1376,
    1385,
    1431,
    1515
  );

  const u_u = "CTF.By.HexpressoCTF.By.Hexpresso";
  const flag = document.getElementById("flag").value;

  for (i = 0; i < u_u.length; i++) {
    if (u_u.charCodeAt(i) + flag.charCodeAt(i) + i * 42 != game[i]) {
      alert("NOPE");
      return;
    }
  }

  // Good j0b
  alert("WELL DONE!");

  document.location.replace(
    document.location.protocol +
      "//" +
      document.location.hostname +
      "/" +
      flag
  );
};
```

So we have an array of several numbers, a string `u_u`, and the string we enter in the field assigned to the variable `flag`.  

In the for loop having for limit the size of the string `u_u`, if at index `[i]`, the sum of the hexadecimal of the character of `u_u` and `flag` adds to `i * 42` is not equal to the number at index `[i]` in game, then the script returns NOPE to us and stops.  
On the contrary, if the calculation is correct, it returns WELL DONE and we can deduce that the flag is the value to put at the end of the url to reach challenge 02.

### THINKING AND WRITING CODE

We must therefore find at this time what to put in the string `flag` in order to fulfill the prerequisite.  
So I wrote a javascript script here:

```javascript
var game = new Array(
    116,
    228,
    203,
    270,
    334,
    382,
    354,
    417,
    485,
    548,
    586,
    673,
    658,
    761,
    801,
    797,
    788,
    850,
    879,
    894,
    959,
    1059,
    1071,
    1140,
    1207,
    1226,
    1258,
    1305,
    1376,
    1385,
    1431,
    1515
  );

const u_u = "CTF.By.HexpressoCTF.By.Hexpresso";

for (i = 0; i < u_u.length; i++) {
    process.stdout.write(String.fromCharCode(game[i] - i * 42 - u_u.charCodeAt(i)));
}
```

It takes the two elements that we know, the array `game` and the string `u_u`.  
To find the flag, we do the calculation in reverse!  
We take the number at the index `[i]` in game, subtract the value of `i * 42` as well as the hexadecimal of `u_u` at the index `[i]`.  
To display the string to use for `flag`, we print it with `process.stdout.write`.  
  
It only remains to run it in a terminal and go to the next challenge ;-)
