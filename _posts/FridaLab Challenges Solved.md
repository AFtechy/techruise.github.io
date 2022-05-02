**Downloading the FridaLab APK on Genymotion: Challenges Solved!**
1. Go to [Rossmarks' Blog](https://rossmarks.uk/blog/fridalab/) and download the `FridaLab.apk`
2. Drag the apk from downloads to the Genymotion device.
Note that it requires you to perform some challenges before you can be able to access the app.
Luckily, i'll walk you through the process.
3. You will need `Jadx-gui` and `Visual Studio Code`.
4. Open `Visual Studio Code`
5. Under extensions, look for `Frida workbench` and install it.
6. Also look for `APKLab` extension and install it. (We’ll use it to decomiple the app)
7. Press `Ctrl + Shift + P`
8. Type APKLab and select the `Open an APK`.

![](2022-05-02-17-20-50.png)

9. Click on it ad wait for the libraries to download.
10. On the window that pops up, navigate to the location in which you downloaded the `FridaLab.apk`.
11. Select the app and open. This should lead you back to the vscode window.
13. Under the add features box, make sure to check the following boxes:
* decompile java
* --only-main-classes
* deobf
14. Press `ok` and go to the new vscode window opened.
15. open `java_src` folder. Notice the challenges under the `poo4uk/rossmarks.fridalab`.
16. Open challenge_01.java.
17. Click on the Frida icon in the left pane (should be on the bottom).
18. Scroll down to find the FridaLab apk on the phone interface.
19. Press the green play button to spawn the phone on the terminal.

**Challenge_01**

The first challenge requires you to `change class challenge_01's variable 'chall01 to: 1`.

To do  that, we need to inject a code that ensures that  when we call the `getChall01Int()` method in the challenge_01 class in MainActivity, the value of the variable `chall01` changes to `1`.
```css
if (Java.available) {
Java.perform(function(){
    var challenge_01 = Java.use('uk.rossmarks.fridalab.challenge_01');
    challenge_01.chall01.value = 1;

});
}
```

**Challenge_02**

Challenge 2 requiers you to just `Run chall02`.


![](2022-05-02-15-03-26.png)


You can find the `chall02()` method in `MainActivity` however as you will notice, it is currently not in use. That's because it is an instance method of MainActivity. so before running it we need to create a new running instance to call it.

We will opt for the `Java.choose` API that will make Frida comb through heap memory to locate our target class.

So here is the implementation:

```css
Java.perform(function() {
    Java.choose("uk.rossmarks.fridalab.MainActivity",{
        onMatch: function (instance) {
            main = instance
            main.chall02()

        },
        onComplete: function() {}
    });
});
```
**Challenge_03**

Challenge 3 requires us to make the `chall03()` method to return `true`.

For us to tweak the implementation of the function inside te target class, we will call the `implementation` method on the `instance` parameter, `chall03`. 

This gives us the leeway to change the code to always return the boolean value `true`.

Here goes:
```css
if (Java.available) {
    Java.perform(function() {
        var challenge_03 = Java.use('uk.rossmarks.fridalab.MainActivity');
        challenge_03.chall03.implementation = function(chall03)
    {
        return true;
    }
});
}
```


**Challenge_04**

If you managed to solve challenge 2 and 3, this one should be pretty easy as the concepts are more or less similar. 

We are basically required to call a function inside a class but this time round it needs to contain a string parameter (unlike in challenge 2).

In fact if you are half as lazy as iam the right intuition will be to copy the code from challenge 2, change the name of the variable and parse the string `"frida"` and voila!

Like this:
```css
Java.perform(function() {
    Java.choose("uk.rossmarks.fridalab.MainActivity",{
        onMatch: function (instance) {
            main = instance
            main.chall04("frida")

        },
        onComplete: function() {}
    });
});
```

You haven't given up already, or have you?

**Challenge_05**

The fifth challenge requires us to implement the code in such a way to always send frida to the `chall05()` method.

So what we need to achieve is to execute chall05 and with the argument as 'frida'.

```css
if (Java.availbale) {
    Java.perform(function() {
        var test = Java.use('uk.rossmarks.fridalab.MainActivity')
        test.chall05.implementation = function() {
           var Techruise = this.chall05('frida')
        
        return(Techruise);
        }
    });
}
```

**Challenge_06**

This one left a bad taste in my mouth. However, its very simple.

We need to run chall06() function after a 10-minute interval.
![](2022-05-02-15-56-13.png)

The function chall06() is calling confirm Chall06() as seen in the code below.

![](2022-05-02-16-12-39.png)

Looking at the `addChall06()`, we can see it sets chall06 variable nad it receives a random value after evey second by the class `MainActivity`.

What we need to do is call a method after every `10 seconds`, and at the same time parse an argument that changes the value of the `confirmChall06()` method in the `challenge_06` class to return true.

We can use the `setInterval()`.

So here's the solution:
```css
const start = setInterval(function() {
    Java.perform(function() {
        var challenge_06 = Java.use('uk.rossmarks.fridalab.challenge_06');
        var chall06 = challenge_06.chall06.value;
        console.log("Click");
        Java.choose("uk.rossmarks.fridalab.MainActivity",{
            onMatch: function (instance) {
                instance.chall06(chall06)
                console.log('Techruise')
        }, onComplete: function() {}
    });
    clearInterval(start);
});
}, 10000);
```

**Challenge_07**

in this challenge, we are required to bruteforce `check07Pin()` then confirm with `chall07()`.

So let's look at the `chall07` class.

![](2022-05-02-16-32-36.png)

it seems to parse the string value as a parameter for the check07Pin() method in the challenge_07 class, and it checks out if true.

![](2022-05-02-16-35-06.png)

The the `setChall07()` method gives a random value to chall 07, and then then it is called on the onCreate() method in the MainActivity.

what we need to do therefore is to use te `check07Pin()`function and compare it with every 4-digit number that exists untill we find a match. we then pass than number the function `chall07`.

Let's go:

```css
Java.perform(function() {
    var main;
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
        onMatch: function(instance) {
            main = instance;
        },
        onComplete: function() { }
    })
    for(var i=9999; i>999; i--) {
        var challenge_07 = Java.use('uk.rossmarks.fridalab.challenge_07');
        var str = i.toString();
        var password = str.padStart(4, '0');
        if (challenge_07.check07Pin(password)) {
            main.chall07(password);
            console.log(password);
            break;
        }
    }
});
```
Almost there slackers.


**Challenge_08**

Challenge 8 simply requires us to change the `'check'` button on the app to `'Confirm'`.

We need to use the `findViewById` (which ususally connects backed and UI ellements like buttons) to be able to locate and mainuplate our button element.

We also need to find the id of the check variable abd then use android.widget,Button class to make our `Confirm` button.

It's in the `C0247R.java` as illustrated below.

![](2022-05-02-17-16-33.png)

```css
Java.perform(function(){
    var main;
    Java.choose('uk.rossmarks.fridalab.MainActivity', {
        onMatch: function(instance) {
            main = instance;
        },
        onComplete: function() {
        }
    })
    var checkid = main.findViewById(2131165231);
    var button = Java.use('android.widget.Button');
    var check = Java.cast(checkid, button);
    var string = Java.use('java.lang.String');
    check.setText(string.$new("Confirm"));
})
```


I hope that was half as much fun for you as it was for me. :P

Note that this was just a high level overview. for more research, here is a helpful link:
* [Frida JavaScript API](https://frida.re/docs/javascript-api/#java).
