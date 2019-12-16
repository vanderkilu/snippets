### NODE-SNIPPETS

This is a collection of tiny libs and tools i build in vanilla node.js, with no npm, no external frameworks in order to get a deeper understanding of node and its inbuilt libraries. I will be penning down how i implemented some of these libs/tools


### LIB / TOOLS 

#### Select-Options

This is one of the tools i implemented while digging deeper into node.js. Built entirely in node.js
with no external tools or libraries. This basically allows you to select from a list of options. This is not fully optimal and is for illustration/learning purpose.

![preview](https://github.com/vanderkilu/snippets/blob/master/select-options.png)


##### How i built the select-options
The core of the program is the `readline.emitKeyPressEvents(stream)` method of the `readline` module.The `readline module` is one of the standard(inbuilt) node.js lib that enables you to read from the console. The `readline.emitKeyPressEvents(stream)` enables you to listen to keyboard events on a stream.
The stream in our case is the standard input(we will be reading from the console), that is `process.stdin`. We can listen to input from the `process.stdin` by subscribing to the keypress event. That is `process.stdin.on(keypress, keyPressedHandler)`. Below is a code sample for subscribing and listening to the keyboard event on the standard input.

```js
    const input = process.stdin
    input.setRawMode(true)
    input.resume()
    input.on('keypress', keyPressedHandler)
```

Now in the `keyPressedHandler` function we check to see which key was pressed. The keys we check include up key, down key, escape, ctrl+c and we handle the response appropriately as indicated in the code sample.

```js
const keyPressedHandler = (_, key) => {
    if (key) {
        const optionLength = selectOption.options.length - 1 
        if ( key.name === 'down' && selectOption.selectIndex < optionLength) {
            selectOption.selectIndex += 1
            selectOption.createOptionMenu()
        }
        else if (key.name === 'up' && selectOption.selectIndex > 0 ) {
            selectOption.selectIndex -= 1
            selectOption.createOptionMenu()
        }
        else if (key.name === 'escape' || (key.name === 'c' && key.ctrl)) {
            selectOption.close()
        }
    }
}
```


This code sample increase/decrease the selectIndex or quit the application based on the key pressed.
The `selectIndex` will later be used as the index for choosing the selected option. Once we have the `selectIndex` we create the option Menu. The sample code for creating the option menu is indicated below.

```js
selectOption.createOptionMenu = () => {
    const optionLength = selectOption.options.length
    if (selectOption.isFirstTimeShowMenu) {
        selectOption.isFirstTimeShowMenu = false
    }
    else {
        output.write(ansiEraseLines(optionLength))

    }
    const padding = selectOption.getPadding(20)
    const cursorColor = ansiColors(selectOption.selector, 'green')

    for (let i= 0; i < optionLength; i++) {
        
        const selectedOption = i === selectOption.selectIndex //1
                                ? `${cursorColor} ${selectOption.options[i]}` //2
                                : selectOption.options[i] //3
        const ending = i !== optionLength-1 ? '\n' : '' //4
        output.write(padding + selectedOption + ending) //5
    }
}
```
The most important part of the sample code is the part labelled in the comment from `1 -5`. What we are doing is selecting the chosen option by comparing `selectIndex` to `i`, our current iteration index. If they are the same then we concatenate our selector, indicated by `*` with the selected option else we just get the option at the current iteration. We then place each option(selected or unselected) option on a different line except the last one. We finally write to the console.

Additional code samples like 
```js
const ansiEraseLines = (count) => {
    //adapted from sindresorhus ansi-escape module
    const ESC = '\u001B['
    const eraseLine = ESC + '2K';
    const cursorUp = (count = 1) => ESC + count + 'A'
    const cursorLeft = ESC + 'G'

    let clear = '';

	for (let i = 0; i < count; i++) {
		clear += eraseLine + (i < count - 1 ? cursorUp() : '');
	}

	if (count) {
		clear += cursorLeft;
	}

	return clear;

}
```
is used for creating a helper function to help us clear the console.

The code sample  
```js
const ansiColors = (text, color) => {
    const colors = {
        'green': 32,
        'blue': 34,
        'yellow': 33   
    }
    if (colors[color]) `\x1b[${colors[color]}m${text}\x1b[0m`
    //default for colors not included
    return `\x1b[32m${text}\x1b[0m`

    
}
```
is used for generating console colors.

Read the full select-options source [here](./selectOptions/index.js) to get a clear glimpse into how i implemented it. 



