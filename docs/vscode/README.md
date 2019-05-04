# Visual Studio Code

## console.log() Faster with Turbo Console Log

Source: [Scotch.io](https://scotch.io/bar-talk/consolelog-faster-with-turbo-console-log)

_Published: March 11, 2019_ by [Chris Sevilleja](https://scotch.io/@chris)

I've recently been [live-coding on Twitch](https://www.twitch.tv/chrisoncode) and the thing that I've loved about it is all the users in the chat that are helping me out by giving me their own tips and tricks.

In our most recent stream, we started talking about my VS Code extensions and one that was recommended to me that I'd never heard of before was [Turbo Console Log](https://marketplace.visualstudio.com/items?itemName=ChakrounAnas.turbo-console-log).

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_800,q_auto:good,f_auto/v1552330629/ldq87diebh6pcgapzoyj.png)

In a nutshell, it lets you take a variable and quickly write the whole console.log statement with keyboard shortcut ```ctrl + alt + l``` for Windows or ```cmd + opt + l``` for Mac.

Here's a gif of it in action:

![img](https://scotch-res.cloudinary.com/image/upload/w_800,q_auto:good,f_auto/v1552330596/qgy7pcsrescj8g9fv7tk.mp4)

### Why Turbo Console Log?

I'm a sucker for any keyboard shortcut that can save me from writing some characters. Absolutely love working with Vim.

> If you told me of an extension or keyboard shortcut that saved me typing even 3 characters, I'll jump on it and dedicate adding that to my arsenal of keyboard shortcuts.

The JavaScript (ES6 Code Snippets) extension has a convenient snippet where you can type:

``` javascript
// Type this and press tab
clg 

// Output: console.log()
```

### Extras: Removing All console.log()

In addition to creating ```console.log()``` messages quickly, Turbo Console Log can:

* **D**elete all console.log() messages: ```shift + alt + d```
* **C**omment all console.log() messages: ```shift + alt + c```
* **U**ncomment console.log() messages: ```shift + alt + u```

![img](https://scotch-res.cloudinary.com/image/upload/dpr_1,w_800,q_auto:good,f_auto/v1552404940/p4ikv7hibu5uggveprjv.png)

### Conclusion

Yes I know that the Debugger for Chrome extension is useful also to find out what is in your code, but sometimes you just gotta console it! For some tutorials on debugging in VS Code, check out these:

* [Debugging JavaScript in Google Chrome and Visual Studio Code](https://scotch.io/tutorials/debugging-javascript-in-google-chrome-and-visual-studio-code)
* [Debugging Node Code in VS Code](https://scotch.io/tutorials/debugging-node-code-in-vs-code)
* [Debugging Create React App Applications in Visual Studio Code](https://scotch.io/tutorials/debugging-create-react-app-applications-in-visual-studio-code)