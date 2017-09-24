# arduino-odd-behavior-demo-01

This repository contains a demonstration of a compile error that occurs depending on the position of a `typedef struct`.

# Overview

This error is specific to the Arduino IDE (*version 1.83*). I fist noticed it while working on an **ESP8266** project where I was adding in some code to an `ino` file. At first I had looked for any syntax errors or errors where the intent wasn't matched to the code. Eventually I distilled the code down to its simplest form and could still reproduce the error.

# Problem Details

There are two folders within this repo, `typedefstruct_pass` and `typedefstruct_fail`. Each contans a single `ino` file.

When the failing example is verified the error is a on a `typedef` symbol and the result is -  "*was not declared in this scope*".

The unusual aspect of the failure is that it appears to be related to the position of the `typedef` within the `ino` file. 

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

# Summary

At this time I'm not sure what the cause is. But I believe to be the result of an error within the compiler. And as I learn more about this issue this repository will be updated.



