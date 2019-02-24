---
layout: post
title: Reversing.kr Easy Crackme Writeup
categories: Reversing
---

## Environment
* Host OS: Kali linux 2018.4
* Guest OS: Windows 7 Service Pack 1
* Virtualization: Virtualbox 5.2.22 
* Debugger: IDA Pro Free 5.0

## Explanation
<a href="http://reversing.kr">Reversing.kr</a> is a website which has some of reverse engineering challenges.
This is a write-up of Easy Crackme on that website.


## Solution
### 1. Running the app
When we run the app, we have a small dialogue and textbox.
As we put a rundom string and put the button, we have a message "Incorrect Password".
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-22-22-47-15.png)
This is likely we have to identify what is the "Password" by reverse engineering.

### 2. Opening with IDA Pro
To analyze this application, we can open the app with IDA Pro.
Since we can see this application retrieves the input data in the textarea, we can assume that "GetDlgItemText" Windows API is used.
We can find it in a subprocess"sub_401080".
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-23-11-27-39.png)

### 3. Getting password
According to the manual of <a href="https://docs.microsoft.com/en-us/windows/desktop/api/winuser/nf-winuser-getdlgitemtexta">GetDlgItemText</a>, we can find where is the memory location the input data was stored.
{% highlight c %}
UINT GetDlgItemTextA(
  HWND  hDlg,	    // A handle to the dialog box that contains the control
  int   nIDDlgItem, // The identifier of the control whose title or text is to be retired
  LPSTR lpString,   // The buffer to receive the title or text
  int   cchMax      // The maximum length, in characters, of the string to be copied to the buffer pointed to by lpString. If the length of the string, including the null character, exceeds the limit, the string is truncated.
);
{% endhighlight %}
In this case, the value of lpString is esp+0x08
{% highlight shell %}
String= byte ptr -64h
lea eax, [esp+6Ch+String] # lea eax, [esp+0x08]
push eax; lpString
{% endhighlight %}
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-24-16-30-20.png)
After calling the GetDlgItemTextA, we can express the address is esp+0x04 since each argument of GetDlgItemTextA is 1 byte.
We can find 1st cmp instruction in the next line.
{% highlight shell %}
cmp byte ptr [esp+0x05], 61h
{% endhighlight %}
The address of input chars is esp+0x04 so this is comparing 2nd character of password and Ascii Character "a".
Under the 1st comparison, we can find 2nd one.
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-23-11-35-45.png)
At first, there is a instruction
{% highlight shell %}
push 2
{% endhighlight %}
This time, the address of input chars changes to esp+0x08.
{% highlight shell %}
lea ecx, [esp+0x0Ah]
push offset a5y; "5y"
push ecx;
call strncmp
{% endhighlight %}
This means, this strncmp is comparing "5y" and 3rd, 4th chars of input.
Next, we can see this section.
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-24-16-33-23.png)
{% highlight shell %}
push ebx
push esi
{% endhighlight %}
After these 2 of push instruction, the address of input chars changes to exp+0x0C.
{% highlight shell %}
lea eax, [esp+70h+var_60] # lea eax, [esp+0x10]
{% endhighlight %}
In eax, there is an address of 5th chars of input.
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-24-16-35-30.png)
Then, comparing [eax] and [esi] with dl and bl.
After that, there 2 pop instructions
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-24-16-37-05.png)
Then, address of input chars goes to [esp+0x04].
Finally we can see there instructions.
![placeholder](https://inar1.github.io/public/images/2018-12-23/2018-12-24-16-38-01.png)
{% highlight shell %}
cmp [esp+68h+String],45h # cmp [esp+4], 45h
{% endhighlight %}
We can figure out 1st character of the input should be Ascii Character "E".
According to these information, we can figure out the password is
{% highlight shell %}
Ea5yR3versing
{% endhighlight %}
