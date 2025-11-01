# Bah graphics

Note that this is very much work in progress.

The whole library is made to be CPU only (as it is more lightweight). It is possible to integrate OpenGL (will come as a third library in the future).

As such, the ui library is made to be really optimized on what it draws/redraws. If you tinker with the default elements behavior, you may need to do manual redraws and such.

The window object supports alpha blending, text drawing, antialiased primitive shapes drawing and IO events.

> Futher documentation soon.

There are two libraries inside of this library.

1. [Creating a raw window.](#window-only)
2. [Creating a window with UI.](#window-and-ui)
3. [Technical notes](#notes)

## Window only

If you only want to create a window and have a graphical context:
```bah
#import "graphics.bah"

//Needed to draw text, initialized in main()
uiFont font

events(w window*, event uint32) {

    //If the event is a draw event:
    //(initialization, resizing, and forced redraws)
    if event == windowEventDraw {
        //Clear the screen. This function only works in grayscale.
        w.clear(WHITE)

        //Measuring the width of the text to print.
        //This will be used to center it.
        textWidth = w.measureText(uiFont, "Hello!")[0]
        
        //Drawing the text in red, centered to the window.
        w.drawText(uiFont, "Hello!", RED, [w.width / 2 - textWidth / 2, w.height / 2 + 6])

        return
    }


    //If any key is pressed, close the window.
    if event == windowEventKeyDown {
        w.close()
    }
}

main(args []str) int {
    //This will get the default UI font of the system.
    //Default font size is 12.
    uiFont = getSystemUIfont()

    //Creating the window object.
    //The events function will be called on every event.
    w = window{
        events: events
    }

    //Launching the window with a width of 800, a height of 600 and a title.
    //This functions will not return until the window is closed.
    w.launch(800, 600, "Test window")
    return 0
}
```

## Window and UI

If you do not want to do anything from scratch inside your window, you can use the UI object instead.

```bah
#import "ui.bah"

//Will contain the name of the user.
//Initialized in main().
userInput uiTextInput

//Function called on button click.
printHelloWorld(btn uiButton*) {

    //If the user did not specified its name:
    if len(userInput.value) == 0 {
        //We make the input placeholder text red.
        INPUT_PLACEHOLDER_TEXT_COLOR = RED
        //Because no event happened on the input, it is not automatically redrawn.
        //We need to either redraw it or redraw the whole window (simpler).
        btn.window.redraw()
        return
    }

    //Greet the user in the console.
    println("Hello "+arrToStr(userInput.value)+".")

    //Close the window.
    btn.window.close()
}

main(args []str) int {
    //Creating the ui object.
    //Note that ui extends window so you can call every method of a window on the ui object. 
    ui = ui {}

    //Adding a text input asking for user's name.
    userInput = uiTextInput {
        //placeholder
        text: "Your name"
        //position as uiPos
        //This sets the position to 50%;50% of the window (centered).
        //In addition tot that, basis center makes the center of the element its position basis.
        //This means that the input will be centered to the screen
        pos: [percents(50).basis(uiBasisCenter), percents(50).basis(uiBasisCenter)]
    }

    //We add the text input to the UI.
    ui.addElement(&userInput)

    //Adding a button that will greet the user and close the window.
    btn = uiButton {
        //button text
        text: "Print"
        //Setting its position as centered (same as the text input).
        pos: [percents(50).basis(uiBasisCenter), percents(50).basis(uiBasisCenter)]
        //But to avoid overlap, we shift the button down by 30 pixels. 
        margin: [pixels(0), pixels(30)]
        //We define its onlick function (that will be called on click).
        onclick: printHelloWorld
    }
    
    //We add the button to the UI.
    ui.addElement(&btn)

    //Launching the window with a width of 400, a height of 300 and a title.
    ui.launch(400, 300, "Test UI")

    //This will be called once the window is closed.
    println("Bye!")
    return 0
}
```

## Notes

Internally, this library uses xcb, fontconfig, freetype2 and lib math to work. These are pretty standard dependencies that should by available in any graphical environement. 