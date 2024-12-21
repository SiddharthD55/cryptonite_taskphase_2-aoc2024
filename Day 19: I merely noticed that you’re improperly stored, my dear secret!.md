Here,
- I will learn how to interact with an executable's API.
- I will intercept and modify internal APIs using Frida.
- I will hack a game using Frida's powerful capabilities.

## The Essence of Game Hacking
Game hacking is a fascinating yet niche area within the larger cybersecurity field. In 2023, the gaming industry generated a massive $183.9 billion, making it a prime target for attackers. I realized that attackers can exploit games by:

- Providing illegitimate ways to activate the game.
- Automating game actions using bots.
- Misusing the game’s internal logic to simplify gameplay.

Hacking a game requires diverse skills, including memory management, reverse engineering, and networking knowledge—especially if the game runs online.

## The Basics of Executables and Libraries
An **executable** is essentially a binary file containing the compiled code for a program, and it’s typically what I run when starting a game or application. Many games rely on **libraries** (e.g., `.so` files) to provide additional functionality.

### Libraries in Action
For instance, a library like `libmaths` might contain an `add()` function that performs the addition of two numbers. Applications trust these libraries to perform the calculations correctly. This gives attackers a potential opportunity to interfere with function calls between executables and libraries, allowing them to modify values or behavior.

## Getting Started with Frida
Frida is a powerful dynamic instrumentation toolkit that allows me to analyze, modify, and interact with running applications. It works by injecting a thread into the target process, which then executes code to enable interaction with the application.

### Example: Modifying a Simple Program
I began by using Frida on a simple program that repeatedly prints `Hello, 1!` to the console. My goal was to replace the `1` with `1337`.

1. **Trace the application** to see all the functions it calls:
   ```bash
   frida-trace ./main -i '*'
   ```
2. Frida automatically generates handler files in the `__handlers__` directory, where I could find the handler for the `say_hello()` function.
3. I modified the handler script for `say_hello()` to intercept the argument and replace its value with `1337`.

#### Updated Handler Script
```javascript
Interceptor.attach(Module.getExportByName(null, "say_hello"), {
    onEnter: function (args) {
        console.log("Original argument: " + args[0].toInt32());
        args[0] = ptr(1337); // Replace argument with 1337
    }
});
```

When I ran the program again using Frida, the console printed `Hello, 1337!` instead of `Hello, 1!`. It was exciting to see how Frida could intercept and change the function parameters in real time!

## Hacking the Game: TryUnlockMe

### Starting the Game
I began the game by running the executable:
```bash
cd /home/ubuntu/Desktop/TryUnlockMe && ./TryUnlockMe
```

### Level 1: Finding the OTP
1. I used Frida to trace functions in the game’s core library:
   ```bash
   frida-trace ./TryUnlockMe -i 'libaocgame.so!*'
   ```
2. The trace output showed the `_Z7set_otpi()` function, which was triggered when interacting with the NPC.
3. I edited the generated handler for the `set_otpi()` function to log its parameters and identify the OTP.

#### Handler for `set_otpi`
```javascript
defineHandler({
    onEnter(log, args, state) {
        log('_Z7set_otpi()');
        log("Parameter:" + args[0].toInt32());
    }
});
```

The logged OTP value helped me unlock the door in the game, and I was able to progress!

### Level 2: Free Shopping
In the second level, I needed to purchase an item. The function `_Z17validate_purchaseiii` was responsible for validating purchases, and it took three integer parameters.

1. I traced the function to log the parameters:
   ```javascript
   defineHandler({
       onEnter(log, args, state) {
           log('_Z17validate_purchaseiii()');
           log("PARAMETER 1: " + args[0]); // Item ID
           log("PARAMETER 2: " + args[1]); // Price
           log("PARAMETER 3: " + args[2]); // Player coins
       }
   });
   ```
2. Upon analyzing the logs, I deduced that the second parameter was the item price. By setting the price to `0`, I could buy any item for free!

#### Updated Handler for `validate_purchaseiii`
```javascript
defineHandler({
    onEnter(log, args, state) {
        log('_Z17validate_purchaseiii()');
        args[1] = ptr(0); // Set the price to 0
    }
});
```

After running the game again, I was able to purchase items without spending any coins!

### Level 3: Biometrics Check
The final level required bypassing a biometrics check. The relevant function was `_Z16check_biometricsPKc`, which took a string as an argument.

1. I logged the argument to inspect its content:
   ```javascript
   defineHandler({
       onEnter(log, args, state) {
           log('_Z16check_biometricsPKc()');
           log("PARAMETER: " + Memory.readCString(args[0]));
       },
       onLeave(log, retval, state) {
           log("The return value is: " + retval);
       }
   });
   ```
2. The return value from the function was `0`, indicating a failure. To bypass the check, I simply set the return value to `1` (True), making the game think the biometrics check passed.

#### Final Handler for `check_biometricsPKc`
```javascript
onLeave(log, retval, state) {
    retval.replace(ptr(1)); // Set return value to True
}
```

After this, the biometrics check was successfully bypassed, and I completed the level!

## Finally...
Using Frida to hack into TryUnlockMe was an exciting journey of learning and experimentation. I was able to intercept and modify function parameters, bypass checks, and manipulate game logic in real time. Frida’s dynamic instrumentation capabilities make it a powerful tool for anyone interested in reverse engineering and game hacking.

While I had fun with this challenge, I always remember the ethical side of hacking.

**Answers**

*What is the OTP flag?* **THM{one_tough_password}**
 
*What is the billionaire item flag?* **THM{credit_card_undeclined}**
 
*What is the biometric flag?* **THM{dont_smash_your_keyboard}**
