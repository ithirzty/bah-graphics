# Bah graphics

Note that this is very much work in progress.

The whole library is made to be CPU only (as it is more lightweight). It is possible to integrate OpenGL (will come as a third library in the future).

As such, the ui library is made to be really optimized on what it draws/redraws. If you tinker with the default elements behavior, you may need to do manual redraws and such.

The window object supports alpha blending, text drawing, antialiased primitive shapes drawing and IO events.

This library is still very early WIP it may still contain bugs and is not as fast as what it could be.

## Try it out:
If you do not have [the bah compiler](https://github.com/ithirzty/bah-asm) installed yet, dot it first.
```bash
git clone https://github.com/ithirzty/bah-graphics
cd bah-graphics
./examples/explorer.bah
```
<center>

![explorer example](/examples/assets/explorer.png)
(this examples weighs 400KB and takes 8MB of RAM)

</center>

There are two libraries inside of this library.

1. [Creating a raw window.](#window-only)
2. [Creating a window with UI.](#window-and-ui)
3. [Documentation](#documentations)
4. [Examples](#examples)
5. [Technical notes](#notes)

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

main(args []str) int {
    //Creating the ui object.
    //Note that ui extends window so you can call every method of a window on the ui object. 
    ui = ui {}

    //Add a button
    ui.addElement(new uiButton {
        text: "Press me to exit"

        //Place it 50% on both axis, position based on its center.
        //By default, its position would be top left corner based.
        pos: [
            percents(50).basis(uiBasisCenter),
            percents(50).basis(uiBasisCenter)
        ]

        //When the button is clicked, it will execute this.
        onclick: function(btn uiButton*) {
            btn.window.close()
        }
    })

    //Launching the window with a width of 400, a height of 300 and a title.
    ui.launch(400, 300, "Test UI")

    //This will be called once the window is closed.
    println("Bye!")
    return 0
}
```

## Documentations

This is the documentation for both ui.bah and graphics.bah.

**NOTE: if you use ui.bah, you should not care about the window structure's mechanism, go directly to [The ui structure](#the-ui-structure) documentation.**

### The window structure

The window structure is mandatory to create a graphical window.
It works by capturing the thread once you call .launch() and will enter an event loop.
Then, you shall set window.events to your event callbacks (draw calls, mouse event, keyboard event...).

Fields:
- `.width` (size of the window),
- `.height` (size of the window),
- `.isRunning` boolean set to true while the window is openned.

Methods:
- `.isKeyModifier(keyMod uint32) bool` returns true if a key modifier (Key_CTRL, Key_SHIFT, Key_ALT, Key_ALT_GR) is currently pressed.
- `.getEventKey() byte` returns the keyboard key code (not useful in most cases).
- `.getEventChar() uint32` returns the unicode codepoint of the key associated with the event, (special chars are: Char_backspace,
Char_delete,
Char_arr_left,
Char_arr_right,
Char_arr_up,
Char_arr_down,
Char_left_shift,
Char_right_shift,
Char_enter,
Char_Left_CTRL,
Char_Right_CTRL,
Char_tab,
Char_shit_tab,
Char_caps_lock).
- `.getEventButton() byte` returns the button number (MOUSE_LEFT_BUTTON,
MOUSE_MIDDLE_BUTTON,
MOUSE_RIGHT_BUTTON,
MOUSE_SCROLL_DOWN,
MOUSE_SCROLL_UP,
MOUSE_SCROLL_LEFT,
MOUSE_SCROLL_RIGHT).
- `.getEventFiles() []str` returns the list of file that have been dropped on the window.
- `.getCursorPosition() [int, int]` returns the coordinates of the mouse cursor. These are window coordinates where [0,0] is top left.
- `.setClipboard(s str)` sets the contents of the clipboard.
- `.getClipboard(callback function(window*, str, ptr), data ptr)` fetches the contents of the clipboard then calls the callback(window, clipboard contents, user data).
- `.setDragFileURI(file str)` sets the file path to send to a window that asks it after a file drop
- `.setCursor(name str)` sets the cursor icon. Uses X cursor names.
- `.moveTo(x uint, y uint)` moves the window to a specified position.
- `.setOpacity(level float)` changes the opacity of a running window.
- `.setIcon(img image)` set the window icon.
- `.setMinSize() and .setMaxSize(width uint, height uint)` set the min/max resizable size of the window.
- `.setTitle(title str)` changes the title of a running window.
- `.`
- `.redraw()` triggers a windowEventDraw.
- `.close()` closes the window.
- `.setScissor(pos [int,int], size [int, int])` enables scissoring of the window, making it possible to only draw inside its rectangle.
- `.disableScissor()` disables scissoring.
- `.setPixel(pos [int,int], color rgbColor)` sets a single pixel to a color.
- `.getRect(pos [int, int], size [uint, uint], buff []rgbColor)` fills a given buffer with the pixels of a rectangle on the screen.
- `.drawRect(pos [int, int], size [uint, uint], color rgbColor)` draws a rectangle.
- `.drawRectImage(pos [int, int], size [uint, uint], buff []rgbColor)` draws a rectangle with a given image buffer instad of a single color.
- `.drawCircle(pos [int, int], radi uint, color rgbColor)` draws a circle.
- `.drawRoundedRect(pos [int, int], size [uint, uint], radi uint, color rgbColor)` draws a rectangle with a given border radius.
- `.drawText(fnt font, text str, color rgbColor, pos [int, int]) [int,int]` draws unicode text using the specified [font] returns the location of a next caracter (as measureText()).
- `.drawChar(fnt font, c uint32, color rgbColor, pos [int,int]) [int,int]` draws a single character.
- `.drawByteArr(fnt font, text []byte, from uint, to uint, color rgbColor, pos [int, int])` draws given range of text from an array of bytes.
- `.measureText(fnt font, text str) [uint, uint]` returns the dimensions in pixels of a text.
- `.measureByteArr(fnt font, text []byte, from uint, to uint) [uint, uint]` returns the dimensions in pixels of a given range of text.
- `.toScreenCoord(pos [int,int]) [int,int]` converts window coordinates to screen coordinates. 
- `.clear(color rgbColor)` clears the window with a given gray scale color.
- `.launch(width uint, height uint, title str)` launches the window, note that this function will exit when the window has been closed.


Globals:
windowEventInitialize
windowEventDraw
windowEventMouseMove
windowEventClose
windowEventMouseDown
windowEventMouseUp
windowEventKeyDown
windowEventKeyUp
windowEventFocusIn
windowEventFocusOut
windowEventResize
windowEventDragIn
windowEventDrag
windowEventDrop

### Fonts
The font structure.

Functions:
- `getSystemUIfont() font` returns the default system UI font.

Fields:
- `.currX` and `.currY` are the current location.
- `.fontSize` is the current font size (should not be set directly).

Methods:
- `.load(path str) bool` loads a TTF font from a file path.
- `.setSize(s uint)` sets the font size.
- `.writeChar(c uint32, color rgbColor, setPixel function(ptr, [int,int], rgbColor), window ptr)` writes a single char to a window with a given setPixel function.
- `.measureChar(c uint32)` advances .currX and .currY but does not draw.

### Colors
The rgbColor structure.

Fields:
`.r`, `.g`, `.b`, `.a` represents the red, green, blue and opacity composant of a color in a range from 0 to 255. By default, `.a` is set to 255.

Methods:
- `.lum(l byte) rgbColor` returns a color with an applied brughtness dimming l [0;255].
- `.mult(l byte) rgbColor` returns a color with every component multiplied by l [0;255].
- `.toLab() [float,float,float]` converts rgb to lab color space.
- `.fromLab(c [float,float,float])` sets the current color from a lab color space
- `.interpolate(c rgbColor, x int) rgbColor` interpolates between two colors.

### Images
The image structure.

Fields:
- `.path: str` the current image path.
- `.width` and `.height` the dimensions of the loaded image.
- `.buf: []rgbColor` the pixel buffer.

Methods:
- `.load(s str) bool` loads (only png for now) an image of specified file path s.

### The ui structure
The ui structure extends [the window structure](#the-window-structure). Every field and method can be reused.

Additional fields:
- `.font` the current UI font, is set to the default system UI font. 
- `.elements` the array containing every root [elements](#ui-elements) of the window. Should not be set directly but can be read.
- `.focusedElement` the currently focused element.

Additional methods:
- `.addElement(elem uiElement*)` adds an element at the root level of the window.
- `.setFocus(elem uiElement*)` sets focus on a given element.
- `.find(id str) uiElement*` finds an element with the given id.

Globals:
uiEventElementAdded: element is added to the ui tree
uiEventElementHoverIn: element is started to be hovered
uiEventElementHoverOut: element is not hovered anymore
uiEventElementHovered: the cursor is moving hover the element
uiEventElementClicked: element is clicked
uiEventElementFocusIn: element is starting to be in focus
uiEventElementFocusOut: element is no longer in focus
uiEventElementMouseDown: a mouse button is being pressed over the element
uiEventElementFileDrag: a file is being dragged over the element
uiEventElementFileDrop: a file has been droped on the element
uiEventElementFileDropOutside: the element has been droped on another window

### UI elements
Every element drawn on the screen is based on the uiElement structure.
An element can contain children that shall be drawn only inside of it.

There are base elements that you can use to build your interface. You can also create more from scratch using the base uiElement structure:

Fields:
- `.pos: [uiPos,uiPos]` its position.
- `.margin: [uiPos,uiPos]` offsets the position of the element.
- `.size: [int,int]` the size of the element. Note that drawing outside of the rectangle defined by .pos and .size is illegal.
- `.padding: [uint,uint,uint,uint]` the space between the border and the contents of the element [left, top, right, bottom].
- `.innerMargin: [uint,uint,uint,uint]` the scissor box offset of the element.
- `.offsets: [int,int]` offsets every elements inside of this element (scrolling).
- `.fontSize: uint` its font size.
- `.cursor: str` its cursor.
- `.absolutePosition: bool` weither or not the element should be affected by scrolling and padding its parent.
- `.id: str` an optional id to be able to .find() the element.
- `.elements: []uiElement*` all of its children elements. Should not be set directly, use .addElement() instead.
- `.parent: uiElement*` the parent element. If null, it is a root element.
- `.window: window*` the current window/ui.
- `.contextItems: []uiContextItem` a list of context item. These are the shortcut shown in context menu when right clicking on an element.
 A uiContextItem has `.ctrl`, `.shift`, `.alt` booleans and a `.key` to define the shortcut that should trigger it. It also has a `.name` that is shown in the context menu and a `.callback: function(uiContextItem*, ptr)` called when triggered.
- `.focusable: bool = true` sets if an element should be focusable or not. An element that is not focusable cannot have contextItems.
- `.dragTarget: bool` sets weither or not something (a file) can be draggeed on the element.
- `.fileURI: str` if not null, this enables the element to be dragged out of the window or a .dragTarget=true element. This is the file path that will be passed to the other window.
- `.manualDrawingMode: bool = false` use if you know what you are doing.
- `.hovered`, `.clicked`, `.focused` are boolean that describe the state of the element when drawing it. These should not be set.
- `._events` and `._draw` shall be set to your .draw() and .events() method when making new element types inside of yout ._init() method.

Methods:
- `.find(id str) uiElement*` finds an element inside of another element.
- `.getSize() [int,int]` returns the size of an element.
- `.getMargin() [int,int]` returns the calculated margins of an element.
- `.getPosition() [int,int]` returns the calculated position of an element.
- `.getInnerSize() [uint,uint]` returns the size of the space occupied by its children. 
- `.remove()` removes an element.
- `.addElement(elem uiElement*)` adds an element as child.
- `.addElementNoRedraw(elem uiElement*)` adds an element as child but does not trigger a redraw of the element. This can be used to insert multiple elements and then calling .redraw().
- `.redrawIn(ms uint)` schedules a redraw in x miliseconds (for animation).
- `.redraw()` immediate redraw

Functions:
- `pixels(px int) uiPos` returns a position structure of x pixels.
- `percents(pc int) uiPos` returns a position structure of x percents. Note that a basis can then be applied on the position with `.basis(uiBasisCenter or uiBasisEnd)` that defines if the position should be calculated at the top left, center or bottom right of the element.


#### uiText
A text that dinamically grows in size and can be selectable (focusable) or not.

Fields:
- `.text: str` the text to display. Should not be changed directly once it is drawn. Use .setText() instead.
- `.color: rgbColor` text color.
- `.bgColor: rgbColor` background color, by default transparent.

Methods:
- `.getSelection() str` returns the current text selection.
- `.setText(s str)` changes the text and redraws the element.

Globals:
UI_TEXT_COLOR = rgbColor{0, 0, 0}
UI_TEXT_SELECTION_COLOR = rgbColor{120, 120, 255}


#### uiTextInput
A text selection input.

Fields:
- `.text: str` the placeholder text to show when it has no value.
- `.onsubmit: function(uiTextInput*)` callback called on enter key press.
- `.value: []byte` the value.
- `.carret: uint` the position from the start of the text of the carret (cursor).

Globals:
INPUT_BACKGROUND_COLOR = rgbColor{220, 220, 220}
INPUT_FOCUSED_BACKGROUND_COLOR = rgbColor{220, 220, 220}
INPUT_HOVERED_BACKGROUND_COLOR = rgbColor{191, 191, 255}
INPUT_PLACEHOLDER_TEXT_COLOR = rgbColor{60,60,60}
INPUT_TEXT_COLOR = rgbColor{0,0,0}
INPUT_TEXT_SELECTION_COLOR = rgbColor{120, 120, 255}


#### uiButton
A clickable button.

Fields:
- `.text: str` the text of the button.
- `.onclick: function(uiButton*)` callback called on click / enter key press.
- `.value: []byte` the value.
- `.radius: uint` the border radius of the button.

Globals:
BUTTON_BACKGROUND_COLOR = rgbColor{220, 220, 220}
BUTTON_BACKGROUND_HOVERED_COLOR = rgbColor{191, 191, 255}
BUTTON_BACKGROUND_CLICKED_COLOR = rgbColor{120, 120, 255}
BUTTON_TEXT_COLOR = rgbColor{0,0,0}
BUTTON_CLICKED_TEXT_COLOR = rgbColor{255,255,255}


#### uiBox
A container element that can be overflowed and scrolled.
This is the only element that has scrollbars.

Fields:
- `.backdrop: function(uiBox*, ui*)` callback called before drawing its children. This is used for drawing a backdrop inside the box or changing its size...


#### uiVerticalArray / uiHorizontalArray
A container that cannot be overflowed as it grows dynamically.
It displays its children as an array on a single axis.

Fields:
- `.spacing: int` spacing between elements in pixels.
- `.dynamicSize: bool = true` the other axis (not the one on which children are spread) grows dynamicaly by default as it is set to true. If set to false, you will have to set manualy its size. 

#### uiVerticalGridArray / uiHorizontalGridArray
A container lays all of its children on a grid and grows dynamically in its main axis.

Fields:
- `.spacing: int` spacing between elements in pixels.

#### uiVerticalSeparator / uiHorizontalSeparator
An element that takes two containers as children and separate them on its main axis with a draggable handle.

Fields:
- `.backdrop: function(uiVerticalSeparator*, ui*)` same as uiBox's callback, called before draw call.
- `.separation: int` to set a default separation offset.
- `.maxSep: int` the maximum separation offset after which the separator cannot be dragged anymore.
- `.minSep: int` the minimum separation offset after which the separator cannot be dragged anymore.

Globals:
UI_SEPARATOR_COLOR = rgbColor{100, 100, 100}

## Examples
You can find examples in the [example directory](/examples/).

![chat example](/examples/assets/chat.png)

## Notes

Internally, this library uses xcb, fontconfig, freetype2, libpng and libmath to work. These are pretty standard dependencies that should by available in any graphical environement. 