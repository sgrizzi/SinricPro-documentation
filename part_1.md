# Part 1 - A simple switch device

## Include Files

At the top of your application add a include line for the device you want to use.<br/>
For example SinricProSwitch:<br/>
```C++
#include <SinricProSwitch.h>
```
## Create a new Class
Create a new class `MySwitch` that inherits from `SinricProSwitch`
```C++
class MySwitch : public SinricProSwitch {
};
```

### Define a new constructor
Our constructor needs at least the `deviceId` and must initialize the inherited `SinricProSwitch` constructor with that id.<br/>
In this example, we will give our new class a variable `switch_pin` and initialize this by the constructor's `switch_pin` parameter.<br/>
Inside the constructer we use the pin variable to set the `pinMode` to `OUTPUT`:
```C++
class MySwitch : public SinricProSwitch {
  public:
    MySwitch(const String& deviceId, int switch_pin) 
    : SinricProSwitch(deviceId)
    , switch_pin(switch_pin) {
      pinMode(switch_pin, OUTPUT);
    }
    
  protected:
    int switch_pin;
};
```

### Adding some functionality to turn our device on and off
The SinricProSwitch class offers the function `onPowerState` which gets called when our device is requested to turn on or off.<br/>
All those functions have to return a `bool` to indicate that the request was handled properly `return true;` or not `return false;`<br/>
In our new class we override this function to turn on or off the pin we have defined before. And since there should nothing go wrong<br/>
turning a pin on or off, we will return the value `true`.

```C++
class MySwitch : public SinricProSwitch {
  public:
    MySwitch(const String& deviceId, int switch_pin) 
    : SinricProSwitch(deviceId)
    , switch_pin(switch_pin) {
      pinMode(switch_pin, OUTPUT);
    }
    
    bool onPowerState(bool& state) override {
      digitalWrite(switch_pin, state);
      return true;
    }
    
  protected:
    int switch_pin;
};
```

### Create an instance from our class
Create a new instance `mySwitch` from our class and initialize the `deviceId` and the `switch_pin`.<br/>
For this example we will use pin (gpio) number 13.<br/>
```C++
MySwitch mySwitch("your-device-id-here", 13);

```
**Note**: *You have to create a new Switch device in SinricPro portal.*<br/>
*After you created the device you will get the deviceId and additionally the App Key and App Secret which is needed later.*<br/>
*Replace the placeholder `your-device-id-here` with the deivce ID you received from the portal.*

## Setting up the WiFi connection
Of course we need the WiFi functionality. For this we use the "good old" standard wifi initialization code.<br/>
To keep our code clean, we will seperate this into a seperate function `setupWiFi()` which is then called<br/>
by the `setup()` function later:
```C++
void setupWiFi() {
  Serial.println("Connecting WiFi Network");
  WiFi.begin("your-wifi-ssid", "your-wifi-password");
  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(250);
  }
  Serial.println("connected!");
}
```
**Note**: *replace the placeholders with your wifi credentials (ssid and password)*

## Initialize the Library
We need to initialize the SinricPro library with our App Key and App Secret which we got from the SinricPro portal.<br/>
For this we have to call the function `SinricPro.begin()` which needs the App Key as first parameter and the App secret as second parameter.<br/>
To keep the code clean, we will also do this in a seperate function `setupSinricPro` which is called from `setup()` later.
```C++
void setupSinricPro() {
  SinricPro.begin("your-app-key-here", "your-app-secret-here");
}
```
**Note**: *replace the placeholders with the App Key and App Secret you got from the step before*

## Writing the setup() function
First we initialize the Serial output with a baudrate of 115200.<br/>
Then we initialize the WiFi by calling our seperate `setupWiFi()` function.<br/>
Last but not least we initialize the SinricPro library by calling our `setupSinricPro()`function.<br/>
```C++
void setup() {
  Serial.begin(115200);
  setupWiFi();
  setupSinricPro();
}
```

## Writing the main-`loop()` function
All wee need to run our code is to call `SinricPro.handle()` in our `loop()` function:
```C++
void loop() {
  SinricPro.handle();
}
```

# Putting things together: the final sketch
This is how our finals sketch looks now:
```C++
#include <SinricProSwitch.h>

class MySwitch : public SinricProSwitch {
  public:
    MySwitch(const String& deviceId, int switch_pin) 
    : SinricProSwitch(deviceId)
    , switch_pin(switch_pin) {
      pinMode(switch_pin, OUTPUT);
    }
    
    bool onPowerState(bool& state) override {
      digitalWrite(switch_pin, state);
      return true;
    }
    
  protected:
    int switch_pin;
};

MySwitch mySwitch("your-device-id-here", 13);

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

# How to run multiple Switch devices?
Since we can reuse our class, this is a very simple task<br/>.
First, we need additional deviceId's. Visit the SinricPro portal and create a few more switch devices and note their deviceId's<br/>.
Then create new instances of our class like so:
```C++
MySwitch mySwitch1("device_id_1", 13);
MySwitch mySwitch2("device_id_2", 14);
MySwitch mySwitch3("device_id_3", 15);
```
**Note**: *Replace the placeholders `device_id_1`, `device_id_2` and `device_id_3` with the device IDs you received from the portal.*<br/>
