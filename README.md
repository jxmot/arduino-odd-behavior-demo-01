# arduino-odd-behavior-demo-01

This repository contains a demonstration of a compile error that occurs depending on the position of a `typedef struct`.

# Overview

This error is specific to the Arduino IDE (*version 1.83*). I fist noticed it while working on an **ESP8266** project where I was adding in some code to an `ino` file. At first I had looked for any syntax errors or errors where the intent wasn't matched to the code. Eventually I distilled the code down to its simplest form and could still reproduce the error.

# Problem Details

There are two *primary* folders within this repo, `typedefstruct_pass` and `typedefstruct_fail`. Each contans a single `ino` file.

When the failing example is verified the error is a on a `typedef` symbol and the result is -  "*was not declared in this scope*".

The unusual aspect of the failure is that it appears to be related to the position of the `typedef` within the `ino` file. 

There is a third folder - `typedefstruct_fail_void` that is described later in this document, it is found under [Additional Issue](#additional-issue).

## Passing Example 

**`typedefstruct_pass.ino`** :

```
typedef struct {
   const char* name;
   int time;
   float value;
} SensorData;

void setup() {
}

void loop() {
}

SensorData sdata;

bool testfunc(SensorData &_data) {

}
```

## Failing Example 

**`typedefstruct_fail.ino`** :

```
void setup() {
}

void loop() {
}

typedef struct {
   const char* name;
   int time;
   float value;
} SensorData;

/* no error on the next line, why? */
SensorData sdata;

bool testfunc(SensorData &_data) {

}
```

### Compiler Error

The following is the first error to appear when verifying the `ino` file :

```
typedefstruct_fail:17: error: 'SensorData' was not declared in this scope

 bool testfunc(SensorData &_data) {

               ^
```

What I find to be odd about this is the line `SensorData sdata;` does not generate an error. The function below it should be within the *same scope*.

# Additional Issue

There is a third folder - `typedefstruct_fail_void`, containing an `ino` file that also fails verification. And this is just as odd as the previously mentioned failure.

## Failing Example

The only difference between `typedefstruct_fail` and `typedefstruct_fail_void` is this :

```
bool testfunc(SensorData &_data) {

}
```

**VS.**

```
void testfunc(SensorData &_data) {

}
```

The compiler generates this error :

```
typedefstruct_fail_void:17: error: variable or field 'testfunc' declared void
```

**NOTE:** The passing example has no errors regardless of the return type for `testfunc()`.

# Other Testing

An *online* compiler (<http://cpp.sh/>) was used on all three files and did not report any *compile time* errors. Linkage errors occured only because of a missing `main()`.

# Additional Research

I have registered on the Arduino.cc forum and read the following posts - 

[Read this before posting a programming question ...](https://forum.arduino.cc/index.php?topic=97455.0)

which led to -  

[How to avoid the quirks of the IDE sketch file pre-preprocessing](http://www.gammon.com.au/forum/?id=12625)

Neither source appeared to explain what might be happening.

# Summary

At this time I'm not sure what the cause is. But I believe to be the result of an error within the compiler. And as I learn more about this issue this repository will be updated.

## Update

I created a thread at *arduino.cc* - [compiler error that depends upon the location of a typedef...](http://forum.arduino.cc/index.php?topic=501909). And the responses were extremely helpful in providing information regarding the cause.

To summarize, the Arduino IDE's *preprocessor* is what I was up against. In my opinion there really isn't any fault there. I just needed a better understanding of how this particular preprocessor worked. 

Apparently the preprocessor is creating a `.cpp` file out of the sketch *before* passing it on to the compiler. And as part of the preprocess step function prototypes are being created. And since I had (*in the failing version*) the typedef further down in the file an error would occur because the preprocessor created a prototype above that line in the resulting `.cpp` file.

I learned from the replies in the thread that there is a way to see what's happening in the preprocessor. All that's needed is to change the preferences so that `Show verbose output during:` compilation is checked. To examine the `.cpp` that the IDE created look in the IDE's result window and find a file named `your-sketch-name.ino.cpp`. Open it in your editor a take a look at it.

In the case presented here you'll notice that the function prototypes are placed *before* the `typedef struct`. 

**`typedefstruct_fail.ino.cpp`** :

```
#include <Arduino.h>
#line 1 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"
#line 1 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"

#line 2 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"
void setup();
#line 5 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"
void loop();
#line 17 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"
bool testfunc(SensorData &_data);
#line 2 "E:\\Workspaces\\IoT\\ESP8266\\arducam_K0061_ESP8266_KIT\\repos\\arduino-odd-behavior-demo-01\\typedefstruct_fail\\typedefstruct_fail.ino"
void setup() {
}

void loop() {
}

typedef struct {
   const char* name;
   int time;
   float value;
} SensorData;

/* no error on the next line, why? */
SensorData sdata;

bool testfunc(SensorData &_data) {

}
```

Take note of line marked as #17. It was pointed out to me that the preprocessor *created* the prototype because I neglected to do so. Following that is the `typedef` that was giving me trouble. Although the error is actually on line 17 the IDE had to report something. And since the sketch file is what you see in the IDE (*not the preprocessed .cpp file*) the error is shown to be on the function definition of `bool testfunc(SensorData &_data)`.