# gForce SDK for Embedded

## Overview

gForce SDK for Embedded is for using [gForce Embedded Suite] [gForceEmbeddedSuite]  
in Embedded devices. It offers an easy way for AVR/ARM to connect to [gForce Armband][gForceArmband] and read data such as gestures, quaternions and emgrawdata from it.

A gForce Embedded Suite includes a gForce Armband and a gForceJoint. The gForceJoint is a hardware module that bridges the gForce armband over BLE to the AVR/ARM board over UART.

## Usage

1. Importing the gForce SDK for Embedded

Download and unzip the library file into your project.
In the main.c function you need to add `#include "gForceAdapterC.h"`.

2. The API we provide in library

We provide some API interfaces in the library to help you analyze the data of gForce armband.
`gForceAdapter.h` and `gForceAdapter.cpp` are files of C++ version, while `gForceAdapterC.h` and `gForceAdapterC.cpp` for wrapped version in C.
You must add those files into your project, and call the interfaces:

```C
  //Parse the data received by the serial port;  
  //*gForceData : Points to the structure that stores the parsed data;  
  //timeout : Set data acceptance judgment time;

  enum GF_Ret GFC_GetgForceData(struct GF_Data *gForceData, unsigned long timeout);
```

```C
  //Converts the received quaternions into euler angles;  
  //*quat :  Points to a structure that stores quaternions;  
  //*euler : Points to the structure that stores the euler Angle;  

  enum GF_Ret GFC_QuaternionToEuler(const struct GF_Quaternion *quat, struct GF_Euler *euler);
```

```C
  //Determine if the hand gesture is obtained;  
  //gesture : The variable that stores the gesture;
  //timeout : Set data acceptance judgment time;  

  BOOL GFC_GetGesture(enum GF_Gesture gesture, unsigned long timeout);
```

The following two functions require users to populate themselves based on the platform they use:

```C
  // Gets the data received by the serial port
  // *data : Point to the received data
  // return : Returns 1 if data got, otherwise 0 
  int SYS_GetChar(unsigned char *data)  
  {  
    int ret = Your_USART_RxData;  

    if(ret == -1)  
      return 0;  

    *data = (unsigned char)ret;  

    return 1;
  }
```

```C
  // returns System time
  unsigned long SYS_GetTick(void)
  {  
    return System_RunTime();  
  }  
```

Then in main() or proper funtion：

```C
  while (true)
  {
    struct GF_Data gForceData;
    struct GF_Euler Euler;
    GF_Ret ret = GFC_GetgForceData((&gForceData), Timeout);
    if (GF_RET_OK == ret)
    {
      GF_Gesture gesture;
      switch (gForceData.type)
      {
      case GF_Data::QUATERNION:
        GFC_QuaternionToEuler(&(gForceData.value.quaternion), &Euler);
        // Serial.print("pitch: "); Serial.print(Euler.pitch);
        // Serial.print(", yaw: "); Serial.print(Euler.yaw);
        // Serial.print(", roll: "); Serial.print(Euler.roll);
        // Serial.println();
        break;
      case GF_Data::GESTURE:
        gesture = gForceData.value.gesture;
        printf("gesture: %d\n", gesture);
        // Do something?
        break;
      default:
        break;
      }
    }
    else
    {
      printf("GFC_GetgForceData() returned: %d\n", ret);
    }
  }
```

**Note:**
    > To make sure gForce armband can recognize your gestures, please refer to
    > [Guide to Performing Gestures][GuideToPerformingGestures]
    > and spend several minutes learning and training yourself. The recognition
    > rate can reach 95% and even higher after you get familiar with performing
    > the gestures properly.

## Trouble shooting

1. Some Arduino boards may use GPIO to simulate UART, alleged software UART. This may consume most of the CPU cycles and cause problems with processing quaternion data. Quaternion data packages may be dropped in this situation. If you meet such a problem,  you may have to use a hardware UART instead.  

[gForceEmbeddedSuite]: https://oymotion.github.io/doc/gForce100EmbeddedSuiteUserGuide/
[gForceArmband]: https://oymotion.github.io/doc/gForce100UserGuide/
[GuideToPerformingGestures]: https://www.youtube.com/watch?v=wBsYJf0wrkk  
