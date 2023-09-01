---
title: Cracking an Android App for Fun and Profit
subtitle: And learning how to protect yourself
date: 2022-09-22
tags: ["android", "security","mobile"]
---


What would you do, if you wanted a premium feature in an Android app, but were too greedy to spend ten bucks on it? Right, you invest >200$ worth your time for an unstable result! 

![It's about the stuff we learn along the way ](https://media.giphy.com/media/wMvESGxZ0Cqd2/giphy.gif)   


## What really is an App?
First off, you got to understand the basics. Android apps are stored in [apk files](https://en.wikipedia.org/wiki/Apk_(file_format)), which are just fancy zip files that adhere to some defined format (like containing an `AndroidManifest.xml` file). Most of the time they are written in either Java or Kotlin and are compiled down to [dalvik byte code](https://source.android.com/docs/core/runtime/dalvik-bytecode). Sadly most applications obfuscate their code to make the lives of reverse engineers – like us 😡 – a lot harder. But there is still a way to make sense of what we are given. So let's go!

## What we're cracking
I chose my target out of necessity. There is this cool app, *Songsterr*, which displays guitar songs, but a lot of features require you to buy the full version for 10$.   

 ![Screenshot of Songsterr guitar tab view](/2023-09-01-android-hacking-songsterrMainView.png)

I started by downloading the APK using a downloader website (just google it, but be aware of the fact, that they could just inject malicious code into the application). 

## Decompilation
We can't read the dalvik bytecode ourselves so we'll use [jadx](https://github.com/skylot/jadx) to translate it to readable java code. I added a few flags, to help deobfuscating the converted source code.    
 ```bash
jadx --threads-count 1 --show-bad-code --deobf --deobf-min 2 --deobf-use-sourcename --deobf-parse-kotlin-metadata target.apk
```    
Having a look at `AndroidManifest.xml` is a good first move. It tells you about the permissions the app requests and which [Intents](https://developer.android.com/guide/components/intents-filters) and Activities it contains. Next, I had a look into the `sources` directory.

![Decompiled source files](/2023-09-01-android-hacking-decompiledSourceFiles.png)

As you can see, most filenames are gibberish. So let's just look into the directories that still hold normal names. In my case `songsterr` under `com` contained readable java code, but not what I was looking for.

## Traffic Interception   
This part of the reverse-engineering process is optional, but seeing the network requests going in and out of an app gives a good overview of it's functionality and the third-party services it is using.
 One security measure most modern apps feature is [Certificate Pinning](https://security.stackexchange.com/questions/29988/what-is-certificate-pinning) which is why I ran [apk-mitm](https://github.com/shroudedcode/apk-mitm) on the unmodified Songsterr apk.
 In short, certificate pinning makes the application no longer trust the devices root certs, but ship with its own ones. I installed the patched APK onto my device and configured my phones WiFi to proxy all network packets through [mitmproxy](https://mitmproxy.org/) running on my laptop (in the same WiFi).  
I noticed a hell lot of analytic data being sent out every few seconds to google, Firebase and amplitude. They contained a **very** unique fingerprint of my device, extracting way more data then I had expected. Interestingly the request to one analytic provider also contained a **is_cracked** flag which, in my case, was true. `-.-` Since I didn't want them to know that I was playing around with their application, I went back to investigating the code.    

## Finding the responsible code    
My first attempt of searching for "crack" in `/sources` rewarded me rather quickly.

The below class luckily still had all functions with sane code.
```java
package com.songsterr;

import android.app.Application;
import g5.C1870d;

public final class CrackChecker {
    public final Application f3589a;

    public CrackChecker(Application application) {
        C1870d.m9000g(application, "context");
        this.f3589a = application;
    }

    // Successfully hacked :)
    public final boolean m10646a() {
        // return false; <- can't just do that here (I think)
        return Songsterr.f3591b && getFirstPackageSignatureHashCode(this.f3589a) != 1046298818;
    }

    public final native int getFirstPackageSignatureHashCode(Application application);
}
```
It seems obvious, that the one method returning a bool and checking a signature is telling the app whether it had been cracked or not. Sadly, we can't just modify the java code and recompile it. (If I'm wrong here, please tell me how one would do that). Instead, for modifying the application, we have to convert the dex to [smali](https://stackoverflow.com/questions/30837450/what-is-smali-code-android), which you can imagine as an assembly language like x64 (although it is higher level).

## Changing Application Behaviour
[Apktool](https://ibotpeaches.github.io/Apktool/) is great for converting dex code to smali. 
```bash
apktool d -b ./test.apk
```
 We can then move into the generated directory, modify the smali and finally recompile it to an APK. I found the code in the same directory as the java before.   
```smali
.method public final a()Z
    .locals 2

    sget-boolean v0, Lcom/songsterr/Songsterr;->b:Z

    if-eqz v0, :cond_0

    iget-object v0, p0, Lcom/songsterr/CrackChecker;->a:Landroid/app/Application;

    invoke-virtual {p0, v0}, Lcom/songsterr/CrackChecker;->getFirstPackageSignatureHashCode(Landroid/app/Application;)I

    move-result v0

    const v1, 0x3e5d40c2

    if-eq v0, v1, :cond_0

    const/4 v0, 0x0

    goto :goto_0

    :cond_0
    const/4 v0, 0x0

    :goto_0
	 // I added this ↓
	 const/4 v0, 0x0
    return v0
.end method
```
I added a `const/4 v0, 0x0` to always move 0 (or False) into the `v0` register before returning. Let's recompile the application (`apktool b dir -o patched.apk`)   
> NOTE:  I had to change set `extractNativeLibraries` in `AndroidManifest.xml` to `true` before recompiling worked!   

You then have to [sign and zipalign](https://stackoverflow.com/questions/10930331/how-to-sign-an-already-compiled-apk) your app and can then install it using adb! It worked, the *is_cracked* flag was false!!
![I was REALLY happy at this point](https://media.giphy.com/media/o75ajIFH0QnQC3nCeD/giphy.gif)

## Back to our original goal
With all we have learned, it is rather easy to fool the app into giving us premium access. I searched for "Premium" and found "PremiumActivity"
```java
 public void m609a(ActivityC0236p activityC0236p) {
        C1870d.m9000g(activityC0236p, "activity");
        if (C1870d.m9004c(((Premium) this.f23712b.getValue()).m616i(), Boolean.TRUE)) {
            this.f23711a.mo15554invoke();
        } else if (activityC0236p.isDestroyed()) {
        } else { ...
```
The code fetches some value, compares it to **TRUE** and if it all checks out invokes something (a PremiumActivity). I made it always go into *invoke-branch*.    
Decompiled, Changed the smali, Recompiled, Signed, Aligned, Installed, Waited and Finally – smiled in peace.   
 I satisfied my curiosity, expanded my hacking horizon and could finally learn new guitar songs :)

 
### Appendix
Everything here has been done for educational purposes only. All code is owned by the company that wrote it. (← at least I tried, I'm not a lawyer XD)  I have bought the full version, just wanted to explore android reverse engineering. I had luck that this app wasn't really obfuscated. If you want to ask or tell me anything, feel free to reach out!
