# MiniGrafx API

This chapter explains the available methods of the graphics library.

## Constructor

The constructor creates the graphic library and allocates the frame buffer memory.
Parameters:
* DisplayDriver: the hardware driver for your library. See [How to implement a new driver](Driver.md)
* bitsPerPixel: bit depth, currently supported are 1, 2, 4 and 8
* palette: array with length 2^bitsPerPixel containing the mapping from palette color to display color

```C++
MiniGrafx(DisplayDriver *driver, uint8_t bitsPerPixel, uint16_t *palette);
```

Example:
```C++
#define BITS_PER_PIXEL 1
uint16_t palette[] = {ILI9341_BLACK, // 0
                      ILI9341_WHITE};

ILI9341_SPI tft = ILI9341_SPI(TFT_CS, TFT_DC);
MiniGrafx gfx = MiniGrafx(&tft, BITS_PER_PIXEL, palette);
```

## changeBitDepth

With changeBitDepth you can change the bit depth even after creating the MiniGrafx object. Please bear in mind that this operation deallocates the reserved frame buffer memory and allocates new memory according to the new
bit depth. Or in other words the content of the frame buffer before calling this method will be lost if it has
not yet been written to the display.

```C++
void changeBitDepth(uint8_t bitsPerPixel, uint16_t *palette);
```

## init

With the init function you initialize the driver as well as the graphics library. Usually you only do this
once in the setup method:

```C++
gfx.init();
```


## commit

The commit() method calls the driver to write the content of the frame buffer to the display. Without calling
commit() the content of the frame buffer will not become visible. You usually do this at the end of all the drawing in one loop iteration:

```C++
gfx.commit();
```

## fillBuffer and clear

With clear() and fillBuffer() you fill the frame buffer with one value; you erase it. The only difference between
the two is that clear() fills the frame buffer with the palette index 0, whereas with fillBuffer(uint8_t) you
can define which color should be used to fill the buffer.

```C++
void clear();
void fillBuffer(uint8_t pal);
```

## setColor

With setColor you define the currently active color for many drawing operations. The parameter is the palette
index of the color you want to set. The range of color is thus between 0 and 2^bitDepth - 1. For a bith depth
of 4 it can be 0 and 15.

```C++
void setColor(uint16_t color);
```

## setPixel and getPixel

Many higher level drawing operations use setPixel to draw into the frame buffer. Using this atomic function might
not be as fast as possible but increases flexibility and portability. getPixel reads out the palette index at
the given index.

```C++
void setPixel(uint16_t x, uint16_t y);
uint16_t getPixel(uint16_t x, uint16_t y);
```

# Extended functionality over original library

## Constructor

Constructors is extended with allocateBuffer parameter. If true (default) constructor allocates memory buffer according screen size an bit depth.
If false allocation is not performed and needs to be done manually.

```C++
MiniGrafx(DisplayDriver *driver, uint8_t bitsPerPixel, uint16_t *palette, bool allocateBuffer = true);
MiniGrafx(DisplayDriver *driver, uint8_t bitsPerPixel, uint16_t *palette, uint16_t width, uint16_t height, bool allocateBuffer = true);
```

## drawBmpFromFile

Draw .bmp file from file system. (x, y) left-top conner. If directWrite is true drwaing bypassed memory buffer and draw direct
to display. 16-bit (only for directWrite by now) and 24-bit uncompressed Bmp is supported. By now x an y must be positive.

```C++
void drawBmpFromFile(String filename, int16_t x, int16_t y, bool directWrite = false);
```

# Commit operations

## commit

(xPos, yPos) left-top corner of buffer at display. Buffer width must fit display (yPos + buffer width <= dispaly width). Buffer height can exceed display. By now x an y must be positive.

```C++
void commit(int16_t xPos = 0, int16_t yPos = 0);
```

## onCommit

Assigns function called on each commit. Returns currently assigned function. So you can build chain of functions called on commit.
If no default callback function will be called no display update process will be performed.
If cbMiniGrafx parameter is nullptr or not specified default function will be restored.

```C++
cbMiniGrafx onCommit(cbMiniGrafx=nullptr);
```

```C++
typedef void (*cbMiniGrafx)(MiniGrafx* gfx, uint16_t x, uint16_t y);
```

# Buffer operations

# initializeBuffer

1. With no parameters allocates buffer to fit whole screen. If buffer already allocated nothing will be done.
2. If heigh and width (h, w) provided allocates buffer to fit requested size and configures library to be limited to draw in this constarains. If buffer already allocated nothing will be done.
3. If preallocated buffer is specified too assigns the buffer for drwaing. If buffer already allocated it will no be freed automaticly.

```C++
void initializeBuffer(uint16_t w = 0, uint16_t h = 0, uint8_t* preallocated = nullptr);
```

## getBuffer

Returns pointer to screen buffer currently used.

```C++
uint8_t getBuffer();
```
## freeBuffer

```C++
void freeBuffer();
```
