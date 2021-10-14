# Part 3 - Organizing the project

## Keep the code clean and organized

It's a good habit to keep your code clean and organized.<br/>
This makes it much easier to read and later changes can be made specifically in the necessary places.<br/>

## Outsourcing our class into a seperate .h file for reuse in other projects

The name of our class is "MySwitch". It is common to create a new .h file with the same name.<br/>
So create a new file "MySwitch.h" and move the class together with the `#include <SinricProSwitch.h>`<br/>
statement into this new file.<br/>

## Include guard

To avoid duplicate includes, we need to protect our MySwitch.h file from duplicate includes.<br/>
This is done by adding `#pragma once` in the first line of our MySwitch.h file.<br/>

## Including our "MySwitch.h" into our main sketch

Since we moved our class out of the main sketch, we need to include the new "MySwitch.h" by adding a simple `#include "MySwitch.h"`

<details>
<summary>MySwitch.h (click to expand)</summary>

```C++
#include <SinricProSwitch.h>

class MySwitch : public SinricProSwitch {
  public:
    MySwitch(const String& deviceId, int switch_pin, int button_pin) 
    : SinricProSwitch(deviceId)
    , switch_pin(switch_pin)
    , button_pin(button_pin) {
      pinMode(switch_pin, OUTPUT);
      pinMode(button_pin, INPUT);
    }
    
    bool onPowerState(bool& state) override {
      digitalWrite(switch_pin, state);
      return true;
    }
  
    void loop() override {
      unsigned long currentMillis = millis();
      bool currentButtonState     = digitalRead(button_pin);
    
      if (currentButtonState != lastButtonState && currentMillis - lastButtonChange >= 50) {
        lastButtonState  = currentButtonState;
        lastButtonChange = currentMillis();
        if (currentButtonState) toggle();
      }
    }  
    
    void toggle() {
      bool newSwitchState = !digitalRead(switch_pin);
      digitalWrite(switch_pin, newSwitchState);
      sendPowerStateEvent(newSwitchState);
    }
    
  protected:
    int switch_pin;
    int button_pin;
    bool lastButtonState;
    unsigned long lastButtonChange;
};
```
</details>
<details>
<summary>Main.ino (click to expand)</summary>

```C++
#include "MySwitch.h"

MySwitch mySwitch("your-device-id-here", 13, 2);
  
void setupWiFi() {
  Serial.println("Connecting WiFi Network");
  WiFi.begin("your-wifi-ssid", "your-wifi-password");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(250);
  }
  Serial.println("connected!");
}

void setupSinricPro() {
  SinricPro.begin("your-app-key-here", "your-app-secret-here");
}

void setup() {
  Serial.begin(115200);
  setupWiFi();
  setupSinricPro();
}

void loop() {
  SinricPro.handle();
}
```
</details>
  
## Moving the implementation outside of the class declaration

To make it even clearer, we now separate the implementation outside of the class definition.<br/>
This may seem a bit unusual at first, but for larger classes, the readability is significantly increased.<br/>
The big advantage is that we can see the class definition at first glance in MySwitch.h without having<br/>
to scroll through all the lines of code of the implementation.

Let's start by seperating the constructors implementation:<p>
  
We change the constructor from this
```C++
  MySwitch(const String& deviceId, int switch_pin, int button_pin)
    : SinricProSwitch(deviceId)
    , switch_pin(switch_pin)
    , button_pin(button_pin) {
    pinMode(switch_pin, OUTPUT);
    pinMode(button_pin, INPUT);
  }
```
  
to a single line like this
```C++
  MySwitch(const String& deviceId, int switch_pin, int button_pin);
```
  
and move the implementation to a point outside the class definition
  
```C++
MySwitch::MySwitch(const String& deviceId, int switch_pin, int button_pin)
  : SinricProSwitch(deviceId)
  , switch_pin(switch_pin)
  , button_pin(button_pin) {
  pinMode(switch_pin, OUTPUT);
  pinMode(button_pin, INPUT);
}  
```

## Repeating the step above for the remaining functions
  
We repeat the steps we did with the contructor to all the remaining functions and getting the final result.<br/>
  
## The final MySwitch.h
This is how our `MySwitch.h` looks now:
  
```C++
#include <SinricProSwitch.h>

class MySwitch : public SinricProSwitch {
  public:
    MySwitch(const String& deviceId, int switch_pin, int button_pin);
    bool onPowerState(bool& state);
    void loop() override;
    
    void toggle();
    
  protected:
    int switch_pin;
    int button_pin;
    bool lastButtonState;
    unsigned long lastButtonChange;
};
  
MySwitch::MySwitch(const String& deviceId, int switch_pin, int button_pin)
  : SinricProSwitch(deviceId)
  , switch_pin(switch_pin)
  , button_pin(button_pin) {
  pinMode(switch_pin, OUTPUT);
  pinMode(button_pin, INPUT);
}
    
bool MySwitch::onPowerState(bool& state) override {
  digitalWrite(switch_pin, state);
  return true;
}
  
void MySwitch::loop() {
  unsigned long currentMillis = millis();
  bool currentButtonState     = digitalRead(button_pin);

  if (currentButtonState != lastButtonState && currentMillis - lastButtonChange >= 50) {
    lastButtonState  = currentButtonState;
    lastButtonChange = currentMillis();
    if (currentButtonState) toggle();
  }
}  
    
void MySwitch::toggle() {
  bool newSwitchState = !digitalRead(switch_pin);
  digitalWrite(switch_pin, newSwitchState);
  sendPowerStateEvent(newSwitchState);
}

```

<hr>

[Part 1 - A simple switch device](part_1.md)<br/>
[Part 2 - Adding a button for local control](part_2.md)<br/>
