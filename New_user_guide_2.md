# Sending feedback to the SinricPro Server
Let's say you want to control your device locally by turning it on and off with a push button.<br/>
When the device power state changes, we need to inform the SinricPro Server about this state change.<br/>
This will change the device state in the portal as well as in the Alexa and Google Home app and display it correctly.<br>

For the following example, we assume a push button is connected to the ESP via a pull-down resistor.<br/>
When the button is pressed `digitalRead` will return `true` and return `false` when the button is not pressed.<br/>

## Let's expand our class with a button
We extend our class with the pin for the pushbutton:
```C++
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
    
  protected:
    int switch_pin;
    int button_pin;
};
```

## Flipping the switch_pin and send an event to SinricPro server
Each SinricPro device have functions to send events to the Server.<br/>
SinricProSwitch offers the function `sendPowerStateEvent()` to send the actual power state of our device.<br/>
It takes a `bool` as parameter with the meaning `true`: the device has been turned on and `false`: the device has been turned off.<p>

To keep the code clean, we add a helper function `toggle()` to our class which will flip the `switch_pin` and send a powerState event<br/>
with the new state to the SinricPro server.

```C++
void toggle() {
  bool newSwitchState = !digitalRead(switch_pin);
  digitalWrite(switch_pin, newSwitchState);
  sendPowerStateEvent(newSwitchState);
}
```

<details>
<summary>Our class now looks like this (click to expand)</summary>
  
```C++
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
    
    void toggle() {
      bool newSwitchState = !digitalRead(switch_pin);
      digitalWrite(switch_pin, newSwitchState);
      sendPowerStateEvent(newSwitchState);
    }
    
  protected:
    int switch_pin;
    int button_pin;
};
```
  
</details>

## Handling the button
To handle a button press, we need to read the button state continuously.<br/>
For this we override the `loop()` function provided by each SinricPro device.<br/>
This function is automatically called by the SinricPro library on each pass of the main `loop()` function.<p>
  
### The need of debouncing
Since buttons have the annoying property to bounce, we have to debounce the button by software.<br/>
The easiest way to do this is to wait a few milliseconds after detecting a change of state on the button pin.<br/>
Waiting here does not mean to use the function `delay()`, because it blocks our sketch.<br/>
We use the concept "wait without delay" which is derived from the concept "[blink without delay](https://www.arduino.cc/en/Tutorial/BuiltInExamples/BlinkWithoutDelay)".<br/>
If you're not familiar with this concept, take a break and click the link above.<p>
  
### Extending our class
To handle the button correctly, we need to keep track of the last button state change<br/>
and we need to calculate the time that has been passed since the last state change happened.<br/>
For this we add two new variables to our class in the protected section:

```C++
bool lastButtonState;
unsigned long lastButtonChange;
```
  
### Overriding the device `loop()` function
The following code implements the wait without delay concept in our devices `loop()` function.<p>
  
Everytime the button state has changed and the debounce time (50 ms) has been passed,<br/>
it updates the `lastButtonState` and the `lastButtonChange` timestamp.<br/>
If the button is in pressed state the `toggle()` function is called which<br/>
will toggle the `switch_pin` and send an powerState event to the SinricPro server.<br/>
  
```C++
  void loop() override {
    unsigned long currentMillis = millis();
    bool currentButtonState     = digitalRead(button_pin);
    
    if (currentButtonState != lastButtonState && currentMillis - lastButtonChange >= 50) {
      lastButtonState  = currentButtonState;
      lastButtonChange = currentMillis();
      if (currentButtonState) toggle();
    }
  }
```

<details>
<summary>Our class now looks like this (click to expand)</summary>

```C++
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
  
## Using our extended class
Since we added another parameter to the constructor (for the button pin), we need to change the instantiation.<br/>
Now the first parameter is the deviceId, the second is our switch pin and the third parameter is the button pin.<br/>
A new instantiation will look like this:
```C++
MySwitch mySwitch("your-device-id-here", 13, 2);
```
**Note**: *Replace the placeholder id with your deviceId and change the pins to match your circuit.*<br>
  
# Putting things together: the final sketch #2
This is how our finals sketch looks now:
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
