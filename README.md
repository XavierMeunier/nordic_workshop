# Nordic Workshop


## HW Requirements
- nRF52 Development Kit
- Computer
- Micro USB Cable

## SW Requirements
All participants must download the following software:
- **nRF5 SDK v14.2.0 [download page](http://developer.nordicsemi.com/nRF5_SDK/nRF5_SDK_v14.x.x/)**
   <pre>
    1.	Download the zip file
    2.	Create a new folder on your C:\ drive and name it Nordic_Semiconductor
    3.	Create another new folder inside Nordic_Semiconductor called nRF5_SDK_14.2.0_17b948a
    and extract the content of nRF5_SDK_14.2.0_17b948a.zip to this folder.
   </pre>

- **nRF Command Line Tools [download page](http://infocenter.nordicsemi.com/topic/com.nordic.infocenter.tools/dita/tools/nrf5x_command_line_tools/nrf5x_installation.html?cp=5_1_1)**

- **Latest version of Segger Embedded Studio [download page](https://www.segger.com/downloads/embedded-studio/)**
    <pre>
    1.	Download and install Segger Embedded Studio for your operating system.
          Embedded Studio is available for Windows, macOS, and Linux.
    2.	Run the installer and follow the installer instructions.
    </pre>
    For commercial use of Embedded Studio, follow the steps in the SEGGER Embedded studio – Getting started video to
         install the free SES license. Link to video [Here](https://www.youtube.com/watch?v=YZouRE_Ol8g&t=334s)

- **nRF Connect for Mobile, [download page](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Connect-for-mobile-previously-called-nRF-Master-Control-Panel)**
Available on the App Store for iOS devices, and Google Play for Android devices


# Hands-on Tasks
The hands-on tasks in this workshop will consist of modifying the ble_app_uart example in the nRF5x SDK v14.2.0 so that the LEDs on the nRF52 DK and a servo can be controlled using commands sent from the nRF Toolbox app.


## TASK 1: Control LEDs using the nRF Toolbox App
Scope: Modify the ble_app_uart example to recognise specific commands sent from the nRF Toolbox app and turn on a LED when one of these commands are received.

### Step 1: Change the device name

Open the ble_app_uart example found in the nRF5_SDK_14.2.0\examples\ble_peripheral\ble_app_uart\pca10040\s132\ses folder. Find the `DEVICE_NAME` define and change the device to a unique name that is easily recognisable, for example.

#define DEVICE_NAME                     "Sigurd_UART"       


### Step 2

We need a variable to keep track of the current command that the nRF52 should handle. We can do this by creating an enumeration, which is basically a list of commands that are assigned a number from 0 and upwards. We create an enumeration  like this

```C    
    typedef enum {
        COMMAND_1,
        COMMAND_2,
        COMMAND_3,
        NO_COMMAND
    } uart_command_t;
```
Every variable of the uart_command_t type can be set to one of the commands in the list.  We've  added a `NO_COMMAND` command which is going to be the default state when no command has been received or the last command has been completed. After declaring the enumeration type `uart_command_t` we need to create a variable `m_command` of the `uart_command_t` type and initialize it to `NO_COMMAND`, i.e.

```C  
    uart_command_t m_command = NO_COMMAND;
```

In Segger Embedded Studio this should look something like this:

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task1_Step2.PNG" width="800">


### Step 3

Find the function `nus_data_handler`. This function is called when data is sent to the Nordic UART Service(NUS) from the nRF Toolbox app and this is where we have to look for the specific commands. Each time we receive data from the phone over NUS we will get the event BLE_NUS_EVT_RX_DATA. The data that has been received is stored in a array pointed to by the `p_data` pointer and we want to store it in a local char array for later use. This can be done by using the [memcpy](https://www.tutorialspoint.com/c_standard_library/c_function_memcpy.htm) function. It will copy the content from cell 0 to `length` in the array pointed to by p_data into the uart_string.

```C  
    char uart_string[BLE_NUS_MAX_DATA_LEN];
    memset(uart_string,0,BLE_NUS_MAX_DATA_LEN);
    memcpy(uart_string, p_evt->params.rx_data.p_data, p_evt->params.rx_data.length);
```
<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task1_Step3v2.PNG" width="550">

### Step 4

Now that we have copied the received data into the `uart_string` array we want to compare the content of `uart_string` with a known command. This can be done by using the [strcmp](https://www.tutorialspoint.com/c_standard_library/c_function_strcmp.htm) function, which will return 0 if `uart_string` is equal to the command.

```C  
    if(strcmp(uart_string,"COMMAND_1") == 0 )
    {
        m_command = COMMAND_1;    
    }
    else if(strcmp(uart_string,"COMMAND_2") == 0 )
    {
        m_command = COMMAND_2;
    }
    else if(strcmp(uart_string,"COMMAND_3") == 0)
    {
        m_command = COMMAND_3;
    }
    else
    {
        m_command = NO_COMMAND;
    }
```
If the uart_string that we received is equal to the known "COMMAND_1" string, then we set the `m_command` variable to the corresponding command in enumeration we created in step 1. Add this functionality to the nus_data_handler() function.  

### Step 5

Now that the `m_command` variable is set to the correct command if the correct string is received, the last thing we need is a function that checks these commands at a regular interval and runs the code we have assosiated with that command. We'll call this function `uart_command_handler` and it takes the pointer to the m_command variable as an input. Inside the function we have a [switch](https://www.tutorialspoint.com/cprogramming/switch_statement_in_c.htm) statement, which is very useful when comparing a variable against a list of values, like our `uart_command_t` enumeration. The `uart_command_handler` should look something like this

```C  
    void uart_command_handler(uart_command_t * m_command)
    {
        uint32_t err_code = NRF_SUCCESS;

        switch(*m_command)
        {
            case COMMAND_1:
                // Put Action to COMMAND_1 here.
                break;

            case COMMAND_2:
                // Put Action to COMMAND_2 here.
                break;

            case COMMAND_3:
                // Put Action to COMMAND_3 here.
                break;

            case NO_COMMAND:
                // No command has been received -> Do nothing
                break;

            default:
                // Invalid command -> Do nothing.
                break;
        }
        /* Reset the command variable to NO_COMMAND after a command has been handled */
        *m_command = NO_COMMAND;

        // Check for errors
        APP_ERROR_CHECK(err_code);
    }
```
After declaring the `uart_command_handler` we add the uart_command_handler to the infinite for-loop in main as shown below.
```C
    for (;;)
        {
        UNUSED_RETURN_VALUE(NRF_LOG_PROCESS());
        uart_command_handler(&m_command);
        power_manage();
        }
```

### Step 6

Now we want to toggle a led when we receive "COMMAND_1". Since the ble_app_uart example uses LED_1 on the nRF52 DK to indicate if the device is advertising or connected to central, so we have to toggle LED_4 instead.
```C
    case COMMAND_1:
        nrf_gpio_pin_toggle(LED_4);
        break;
```
We also have to configure the pin connected to LED_4 as an output so make sure that you add to main()
```C
    nrf_gpio_cfg_output(LED_4);
```

### Step 7

Compile the project by pressing the F7 key, and flash it to the nRF52 DK. You can flash the project by clicking on the Target tab, and click on "Download ble_app_uart_pca10040_s132". The project will also be flashed to the nRF52-DK if we start a debug seesion. A debug session can be started by pressing the F5 key twice.
The Segger Embedded Studio project have already been configured to flash the SoftDevice for us. LED 1 on the nRF52 DK should start blinking, indicating that its advertising.  We've now completed the configuration on the nRF52 side  

### Step 8

Install the nRF Toolbox app on you Android/iOS phone. You can find the app [here](https://www.nordicsemi.com/eng/Products/Nordic-mobile-Apps/nRF-Toolbox-App) on Google Play Store and [here](https://itunes.apple.com/us/app/nrf-toolbox/id820906058?mt=8) on Apple App Store. Open the nRF Toolbox app and click the UART symbol, which should display the picture under "UART Menu". Press EDIT in the top right corner, the menu should now turn orange and then press the top-left square in the 3x3 matrix. The app should now display the same image as shown under "Edit Button Menu". Enter the command shown in the "Configure Command 1" and select 1 as the icon. After pressing OK you should see return to the orange edit screen shown under "Edit Mode". Press "DONE" in the top-right corner and you should return to the blue UART menu as shown under "Edit Complete".  



nRF Toolbox Menu  | UART Menu     | Edit Button Menu| Configure Command 1 | Edit Mode | Edit Completed |
------------ | ------------- | ------------  | ------------  | ------------  | ------------  |
<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/nrf_toolbox.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/default.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/edit.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/command_1.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/edit_done.png" width="200"> | <img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/done.png" width="200">


### Step 9

Press the Connect button, this should bring up a list of nearby BLE devices. Select the device with the name you assigned in step 1 of this task. It should be the one of the devices with the strongest signal. LED 1 on your nRF52 DK should now stop blinking and stay lit, indicating that its in a connected state.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/device_list.png" width="200">

You can now press the button we configured to send "COMMAND_1" to the nRF52 DK. This should turn on LED 4 on the nRF52 DK. Pressing it again should turn it off. Congratulations, you've just controlled one of the GPIO pins of the nRF52 using Bluetooth Low Energy.

## Task 2: Control the Servo using the nRF Toolbox App

**Scope:** The goal of this task is to include the PWM library in the ble_app_uart example so that we can control the servo from our smartphone. The app_pwm library is documented on [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/lib_pwm.html?resultof=%22%61%70%70%5f%70%77%6d%5f%69%6e%69%74%22%20) Infocenter page

Connecting the Servo to your nRF52 DK:

The three wires coming from the SG92R Servo are:

Brown: 	Ground 				- Should be connected to one of the pins marked GND on your nRF52 DK. 

Red: 	5V 					- Should be connected to the pin marked 5V on your nRF52 DK.

Orange: PWM Control Signal 	- Should be connected to one of the unused GPIO pins of the nRF52 DK (for example P0.4, pin number 4).

### Step 1

In order to use the app_pwm library in the ble_app_uart project you have to add the necessary .c and .h files to the project. The app_pwm library uses the following files:

* `app_pwm.h`
* `app_pwm.c`
* `nrf_drv_ppi.c`
* `nrf_drv_timer.c`


#### Adding .c files
 

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/adding_libraries.png" width="500">

Right-clik the folder that you want to add the .c file to and select "Add Existing File...". Navigate to the components folder and then find the missing .c file in either nrf_drivers or libraries. 

* The `app_pwm.c` file can be found in the folder "SDK_folder\components\libraries\pwm"
* The `nrf_drv_ppi.c` file can be found in the folder "SDK_folder\components\drivers_nrf\ppi"
* The `nrf_drv_timer.c` file can be found in the folder "SDK_folder\components\drivers_nrf\timer"




#### Modifying sdk_config.h  

We also have to make sure that the correct nRF_Drivers and nRF_Libraries are enabled in the sdk_config.h file.


Open the sdk_config.h file, and set the following defines to 1:

* APP_PWM_ENABLED  (line 2042)
* PPI_ENABLED      (line 968)
* TIMER_ENABLED    (line 1582)
* TIMER1_ENABLED   (line 1649)

E.g. for APP_PWM_ENABLED it should look like this:

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/app_pwm_sdk_config.png" width="1000">

Note that it's also possible to use a GUI java tool called CMSIS Configuration Wizard to modify the sdk_config.h file.
For more information about that, you can later have a look at [this](https://www.youtube.com/watch?v=b0MxWaAjMco) and [this link](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v14.2.0/sdk_config.html?cp=4_0_0_1_6_4_1#sdk_config_ide_ses). But for now, we will move forward to step 2.



### Step 2

The first thing we have to do in main.c is to include the header to the PWM library, `app_pwm.h` and create a PWM instance using the TIMER1 peripheral. This is done as shown below 

```C
    #include "app_pwm.h"
    
    APP_PWM_INSTANCE(PWM1,1);                       // Create the instance "PWM1" using TIMER1.
```
### Step 3

The next thing we have to do is creating the function `pwm_init()` where we configure, initialize and enable the PWM peripheral. You configure the pwm library by creating a app_pwm_config_t struct like shown below

```C
    app_pwm_config_t pwm_config = {
        .pins               = {4, APP_PWM_NOPIN},
        .pin_polarity       = {APP_PWM_POLARITY_ACTIVE_HIGH, APP_PWM_POLARITY_ACTIVE_LOW}, 
        .num_of_channels    = 1,                                                          
        .period_us          = 20000L                                                
    };
```
This struct contains the information about which pins that are used as PWM pins, which polarity the pisn should have, how many channels(the number of PWM outputs, this is limited to 2 per PWM instance) and the period of the PWM signal. The struct is given as an input to the [app_pwm_init](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#gae3b3e1d5404fd776bbf7bf22224b4b0d) function which initializes the PWM library. 

```C
    uint32_t err_code;
    err_code = app_pwm_init(&PWM1,&pwm_config,NULL);
    APP_ERROR_CHECK(err_code);
```

You can initialize the PWM library with a callback function that is called when duty cycle change process is finsihed, but this is not necessary for this example so we will just pass NULL as an argument. After initializng the PWM library you have to enable the PWM instance by calling [app_pwm_enable](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga94f5d824afec86aff163f7cccedaa436).

```C
    app_pwm_enable(&PWM1);
```
The pwm_init() function is now finished and can add it to the `main()` function before the infinite for-loop.

### Step 4

Now that we have initialized the PWM library its time to set the duty cycle of the PWM signal to the servo using the  [app_pwm_channel_duty_set](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.sdk5.v12.2.0/group__app__pwm.html#ga071ee86851d8c0845f297df5d23a240d) function. This will set the duty cycle of the PWM signal, i.e. the percentage of the total time the signal is high or low depending on the polarity that has been chosen. If we want to set the PWM signal to be high 50% of the time, then we call `app_pwm_channel_duty_set` with the following parameters.

```C
    while (app_pwm_channel_duty_set(&PWM1, 0, 50) == NRF_ERROR_BUSY);
```
The `app_pwm_channel_duty_set` function should be called until it does not return `NRF_ERROR_BUSY` in order to make sure that the duty cycle is correctly set. 

### Step 5

The goal of this task was to make the servo sweep from its maximum angle to its minimum angle. This can be done by calling app_pwm_channel_duty_set twice with a delay between the two calls in the main while-loop.

```C
    while (true)
    {
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        nrf_delay_ms(1000);
    }
    
```
The code snippet above sets the duty cycle to 0, you have to figure out the correct duty cycle values for the min and max angle. 

Tips:
* Period should be 20ms (20000us) and duty cycle  for the min and max angle corresponds to 1ms and 2ms respectivly.

### Step 6

Add `SERVO_POS_1` and  `SERVO_POS_2` to the `uart_command_t` enumeration created in Task 1, step 2. Add these commands to the `nus_data_handler` and the `uart_command_handler`. Call `app_pwm_channel_duty_set` when the commands are processed by the `uart_command_handler`, i.e.
```C 
    case SERVO_POS_1:
        while (app_pwm_channel_duty_set(&PWM1, 0, duty_cycle) == NRF_ERROR_BUSY);
        break;
```

### Step 7

Configure two buttons in the nRF Toolbox app to send the `SERVO_POS_1` and  `SERVO_POS_2` commands to your nRF52 DK.


### Step 8

Compile the project, flash it to the nRF52 DK and control the servo using the nRF Toolbox App.

## TASK 3: Measure the die temperature of the nRF52 and send it to the nRF Toolbox app.
**Scope:**  Measure the temperature of the nRF52 die and use an application timer to send the measurement to the nRF Toolbox App.

### Step 1

In order to measure the temperature of the nRF52 die you have to read the registers of the TEMP peripheral of the nRF52, see [this](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.nrf52832.ps.v1.1/temp.html?cp=2_2_0_26#concept_fcz_vw4_sr) page on the Nordic Infocenter. However, the SoftDevice uses this peripheral to calibrate the clock of the nRF52 so that its accurate enough to be used for BLE. We can therefore not access the TEMP registers directly, we have to go through the SoftDevice and ask it to check what the temperature is. This is done by calling the [sd_temp_get](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.s132.api.v3.0.0/group___n_r_f___s_o_c___f_u_n_c_t_i_o_n_s.html#gade0ea69f513ff1feab2c4f6e1c393313
) function.


We can now create a function called `read_temperature()` that in turn calls `sd_temp_get` and returns the temperature in degrees celsius.

```C
    static double read_temperature()
    {
        int32_t temp;
        sd_temp_get(&temp);

        return ((double)temp*0.25);   
    }
```

### Step 2
Next, we're going to create an application timer that calls `read_temperature()` periodically. First, create the timer ID as shown below

```C
    APP_TIMER_DEF(m_temp_timer_id);                 // Create the timer ID "m_temp_timer_id" .
```

### Step 3
Next, we're going to create the application timer timeout handler that is going to call `read_temperature()`. We need to use the `ble_nus_string_send()` function to send the measured temperature to the nRF Toolbox app. This function takes the pointer to an `uint8_t` array so we need to format a string and place it in the array. The  [sprintf](https://www.tutorialspoint.com/c_standard_library/c_function_sprintf.htm) function does exactly this, i.e. copies the content of a string into an array.

```C
    void temp_timer_timeout_handler(void * p_context)
    {
        double temp = read_temperature();

        // Place the temperature measurement into the data array
        uint32_t err_code;
        uint8_t data[20];

        sprintf((char *)data, "Temperature: %0.2f", temp);
        uint16_t length = sizeof(data);

        //Send temperature measurement to nRF Toolbox app
        err_code = ble_nus_string_send(&m_nus, data, &length);
        APP_ERROR_CHECK(err_code);
    }
```

### Step 4
Next, we need to create the timer and specify that it should use `temp_timer_timeout_handler` as its timeout handler.

```C
    void create_timers()
    {
        uint32_t err_code;
        // Create  temperature timer
        err_code = app_timer_create(&m_temp_timer_id,
                                    APP_TIMER_MODE_REPEATED,
                                    temp_timer_timeout_handler);
        APP_ERROR_CHECK(err_code);
    }
```
### Step 5
Now, the only thing that remains is to start the application timer. However, it is important that the `ble_nus_string_send` function is not called when we're not connected to the nRF Toolbox app. If we try to send the string containing the temperature without being in a connection then the ble_nus_string_send function will return NRF_ERROR_INVALID_STATE, and this will be passed into the APP_ERROR_CHECK() error-handler. It should therefore only be possible to start the timer when we issue a command from nRF Toolbox.  

Add `TEMP_TIMER_START` and `TEMP_TIMER_STOP` to the uart_command_t enumeration and modify the `nus_data_handler` so that these commands are recognized.


Lastly, add the following cases to the `uart_command_handler`

```C
        case TEMP_TIMER_START:
            err_code = app_timer_start(m_temp_timer_id, APP_TIMER_TICKS(1000),NULL);
            APP_ERROR_CHECK(err_code);
            break;

        case TEMP_TIMER_STOP:
            err_code = app_timer_stop(m_temp_timer_id);
            APP_ERROR_CHECK(err_code);
            break;
```

### Step 6
Configure two buttons in the nRF Toolbox app to send the `TEMP_TIMER_START`and `TEMP_TIMER_STOP` commands

### Step 7
Before compiling the project, we need to configure Embedded Studio to allow the data type `double` to be used by the sprintf function.
In the Project Explorer on the left side, right click on **Project 'ble_app_uart_pca10040_s132**, and click `Edit Options...`

Select the Common configuration as shown in the image below.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task2_Step7.png" width="200">

Click on Printf/Scanf, and set the option `Printf Floating Point Supported` to `Double`. Press OK to apply the change.

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/Task2_Step7_2.png" width="400">

### Step 8
Compile the project and flash it to you nRF52 DK.  After pressing the button you configured to send the `TEMP_TIMER_START` command you should be able to see the temperature in the nRF Toolbox app log ( you open this by holding your finger above the UART text to the left of the screen and swiping from left to right)  

<img src="https://github.com/sigurdnev/nordic_workshop/blob/master/images/temperature.png" width="500">


## BONUS TASK: Creating a Custom Service
This is an bonus task if you have managed to finish the previous task within the time period of the workshop. Note that this is a larger task than the previous tasks, and it takes more time to finish this task.

Scope: The aim of this task is simply to create one service with one characteristic without too much theory in between the steps. In this task we will be starting from scratch in the ble_app_template project.



## Tutorial Steps
<!---
```c    
    uint8_t data
    // put C code here
```

[link to webpage](www.google.com)
--->
### Step 1 - Getting started  
1. Navigate to the nRF5_SDK_14.2.0_17b948a/examples/ble_peripheral folder and find the ble_app_template project folder.

2. Create a copy of the folder and name it `custom_ble_service_example`.

3. Navigate to custom_ble_service_example\pca10040\s132\ses and open the ble_app_template_pca10040_s132.emProject project

### Step 2 - Creating a Custom Base UUID 

The first thing we need to do is to create a new .c file, lets call it ble_cus.c (**Cu**stom **S**ervice), and its accompaning .h file ble_cus.h. Create the two files in the same folder as the main.c file.   At the top of the header file ble_cus.h we'll need to include the following .h files

```c
/* This code belongs in ble_cus.h*/
#include <stdint.h>
#include <stdbool.h>
#include "ble.h"
#include "ble_srv_common.h"
```



Next, we're going to need a 128-bit UUID for our custom service since we're not going to implement our service with one of the  16-bit Bluetooth SIG UUIDs that are reserved for standardized profiles. There are several ways to generate a 128-bit UUID, but we'll use [this](https://www.uuidgenerator.net/version4) Online UUID generator. The webpage will generate a random 128-bit UUID, which in my case was

```
f364adc9-b000-4042-ba50-05ca45bf8abc
```
The UUID is given as the sixteen octets of a UUID are represented as 32 hexadecimal (base 16) digits, displayed in five groups separated by hyphens, in the form 8-4-4-4-12. The 16 octets are given in big-endian, while we use the small-endian representation in our SDK. Thus, we must reverse the byte-ordering when we define our UUID base in the ble_cus.h, as shown below.

```c
/* This code belongs in ble_cus.h*/
#define CUSTOM_SERVICE_UUID_BASE         {0xBC, 0x8A, 0xBF, 0x45, 0xCA, 0x05, 0x50, 0xBA, \
                                          0x40, 0x42, 0xB0, 0x00, 0xC9, 0xAD, 0x64, 0xF3}
```
Now that we have defined our Base UUID, we need to define a 16-bit UUID for the Custom Service and a 16-bit UUID for a Custom Value Characteristic.  

```c
/* This code belongs in ble_cus.h*/
#define CUSTOM_SERVICE_UUID               0x1400
#define CUSTOM_VALUE_CHAR_UUID            0x1401
```
The values for the 16-bit UUIDs that will be inserted into the base UUID can be choosen by random. 

### Step 2 - Implementing the Custom Service 

First things first, we need to include the ble_cus.h header file we just created as well as some common SDK header files in ble_cus.c.

```c
/* This code belongs in ble_cus.c*/
#include "sdk_common.h"
#include "ble_srv_common.h"
#include "ble_cus.h"
#include <string.h>
#include "nrf_gpio.h"
#include "boards.h"
#include "nrf_log.h"
```

The next step is to add a macro for defining a Custom Service(ble_cus) instance by adding the following snippet below the includes in ble_cus.h

```c
/* This code belongs in ble_cus.h*/

/**@brief   Macro for defining a ble_cus instance.
 *
 * @param   _name   Name of the instance.
 * @hideinitializer
 */
#define BLE_CUS_DEF(_name)                                                                          \
static ble_cus_t _name;                                                                             \

```
we will use this macro to define a custom service instance in main.c later in the tutorial. 

Ok, so far so good. Now we need to create two structures in ble_cus.h, one Custom Service init structure, ble_cus_init_t struct to hold all the options and data needed to initialize our custom service.

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service init structure. This contains all options and data needed for
 *        initialization of the service.*/
typedef struct
{
    uint8_t                       initial_custom_value;           /**< Initial custom value */
    ble_srv_cccd_security_mode_t  custom_value_char_attr_md;     /**< Initial security level for Custom characteristics attribute */
} ble_cus_init_t;
```

The second struct that we need to create is the Custom Service structure, ble_cus_s, which holds the status information of the service. 

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service structure. This contains various status information for the service. */
struct ble_cus_s
{
    uint16_t                      service_handle;                 /**< Handle of Custom Service (as provided by the BLE stack). */
    ble_gatts_char_handles_t      custom_value_handles;           /**< Handles related to the Custom Value characteristic. */
    uint16_t                      conn_handle;                    /**< Handle of the current connection (as provided by the BLE stack, is BLE_CONN_HANDLE_INVALID if not in a connection). */
    uint8_t                       uuid_type; 
};
```

The next step is to add a forward declaration of the ble_cus_t type
```c
/* This code belongs in ble_cus.h*/

// Forward declaration of the ble_cus_t type.
typedef struct ble_cus_s ble_cus_t;
```


The first function we're going to implement is ble_cus_init function, which we're going to initialize our service with. First, we need to do is to add its function decleration in the ble_cus.h file. 

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for initializing the Custom Service.
 *
 * @param[out]  p_cus       Custom Service structure. This structure will have to be supplied by
 *                          the application. It will be initialized by this function, and will later
 *                          be used to identify this particular service instance.
 * @param[in]   p_cus_init  Information needed to initialize the service.
 *
 * @return      NRF_SUCCESS on successful initialization of service, otherwise an error code.
 */
uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init);
```

The first thing we should do upon entering ble_cus_init is to check that none of the pointers that we passed as arguments are NULL and declare the two variables err_code and ble_uuid.

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;
}
```
After verifying that the pointers are valid we can initialize the Custom Service structure

```c
/* This code belongs in ble_cus_init() in ble_cus.c*/

// Initialize service structure
p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;
```

This consists of setting the connection handle to invalid( should only be valid when we're in a connection). Next, we're going to add our custom (also referred to as vendor specific) base UUID to the BLE stack's table.

```c
/* This code belongs in ble_cus_init() ble_cus.c*/

// Add Custom Service UUID
ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
VERIFY_SUCCESS(err_code);

ble_uuid.type = p_cus->uuid_type;
ble_uuid.uuid = CUSTOM_SERVICE_UUID;
```

We're almost done, the last thing we have to do is to add the Custom Service decleration to the BLE Stack's GATT table. 

```c
/* This code belongs in ble_cus_init() in ble_cus.c*/

// Add the Custom Service
err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
if (err_code != NRF_SUCCESS)
{
    return err_code;
}
```

If you have followed the steps correctly, then ble_cus_init should look like this.

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;

    // Initialize service structure
    p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;

    // Add Custom Service UUID
    ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
    err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
    VERIFY_SUCCESS(err_code);
    
    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_SERVICE_UUID;

    // Add the Custom Service
    err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }
    return err_code;
}
```

### Step 3 - Initializing the Service and advertising our 128-bit UUID.

Now it's time to initialize the service in main.c and put our 128-bit UUID in the advertisement packet so that other BLE devices can see that our device has the Custom Service. 

First, add the ble_cus.h file to the include list in main.c

```c
#include "ble_cus.h"
```

and then use the BLE_CUS_DEF macro to add a custom service instance called m_cus, i.e. add the following line below the defines in main.c

```c
BLE_CUS_DEF(m_cus);
``` 

The next step is to find the empty services_init function in main.c, which should look like this

```c
/**@brief Function for initializing services that will be used by the application.
 */
static void services_init(void)
{
    /* YOUR_JOB: Add code to initialize the services used by the application.
       ret_code_t                         err_code;
       ble_xxs_init_t                     xxs_init;
       ble_yys_init_t                     yys_init;

       // Initialize XXX Service.
       memset(&xxs_init, 0, sizeof(xxs_init));

       xxs_init.evt_handler                = NULL;
       xxs_init.is_xxx_notify_supported    = true;
       xxs_init.ble_xx_initial_value.level = 100;

       err_code = ble_bas_init(&m_xxs, &xxs_init);
       APP_ERROR_CHECK(err_code);

       // Initialize YYY Service.
       memset(&yys_init, 0, sizeof(yys_init));
       yys_init.evt_handler                  = on_yys_evt;
       yys_init.ble_yy_initial_value.counter = 0;

       err_code = ble_yy_service_init(&yys_init, &yy_init);
       APP_ERROR_CHECK(err_code);
     */
}
```

Ok, we're going to do as we're told, i.e. create a ble_cus_init_t struct and populate it with the necessary data and then pass it as an argument to our service init function ble_cus_init();

```c
/**@brief Function for initializing services that will be used by the application.
 */
static void services_init(void)
{
    /* YOUR_JOB: Add code to initialize the services used by the application.*/
    ret_code_t                         err_code;
    ble_cus_init_t                     cus_init;

     // Initialize CUS Service init structure to zero.
    memset(&cus_init, 0, sizeof(cus_init));
	
    err_code = ble_cus_init(&m_cus, &cus_init);
    APP_ERROR_CHECK(err_code);	
}
```
Now that we have initialized the service we have to add the 128bit UUID to the advertisement packet. If you navigate to the top of main.c you should find the m_adv_uuids array.

```c
// YOUR_JOB: Use UUIDs for service(s) used in your application.
static ble_uuid_t m_adv_uuids[] = {{BLE_UUID_DEVICE_INFORMATION_SERVICE, BLE_UUID_TYPE_BLE}}; /**< Universally unique service identifiers. */
```

We need to replace the BLE_UUID_DEVICE_INFORMATION_SERVICE with the CUSTOM_SERVICE_UUID we defined in ble_cus.h as well as replace  BLE_UUID_TYPE_BLE with BLE_UUID_TYPE_VENDOR_BEGIN since this is a 128-bit vendor specific UUID and not a 16-bit Bluetooth SIG UUDID. m_adv_uuids should now look like this

```c
// YOUR_JOB: Use UUIDs for service(s) used in your application.
 ble_uuid_t m_adv_uuids[] = {{CUSTOM_SERVICE_UUID, BLE_UUID_TYPE_VENDOR_BEGIN }}; /**< Universally unique service identifiers. */
```
After this step we need to tell the BLE stack that we're using a vendor-specific 128-bit UUID and not a 16-bit bit UUID. This is done by changing the following define in sdk_config.h from  
```c
#define NRF_SDH_BLE_VS_UUID_COUNT 0
```
to 

```c
#define NRF_SDH_BLE_VS_UUID_COUNT 1
```

Now, adding a vendor-specific UUID to the BLE stack results in the RAM requirement of the SoftDevice increasing, which we need to take into account. 
<!---
- [ ] Optional: Add section where the function of app_ram_base since its useful for debugging.
--->
Segger Embedded Studio(SES): Click "Project -> Edit Options", select the Common Configuration, then select Linker and then open the Section Placement Macros Section abd modify RAM_START IRAM1 to 0x200020F0 and RAM_SIZE to 0xDF10, as shown in the screenshot below

Memory Settings Segger Embedded Studio | 
------------ |
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/memory_settings_SDK_v14_SES.JPG" width="1000"> |

Keil: Click "Options for Target" in Keil and modify the Read/Write Memory Areas so that IRAM1 has the start address 0x200020F0 and size 0xDF10, as shown in the screenshot below

Memory Settings Keil | 
------------ |
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/memory_settings_SDK_v14.JPG" width="1000"> |


The final step we have to do is to change the calling order in  main() so that services_init() is called before advertising_init(). This is because we need to add the CUSTOM_SERVICE_UUID_BASE to the BLE stack's table using sd_ble_uuid_vs_add() in ble_cus_init() before we call advertising_init(). Doing it the otherway around will cause advertising_init() to return an error code. 

That should be it, compile the ble_app_template project, flash the S132 v5.0.0 SoftDevice and then flash the ble_app_template application. LED1 on your nRF52 DK should now start blinking, indicating that its advertising.  Use nRF Connect for Android/iOS to scan for the device and view the content of the advertisment package. If you connect to the device you should see the service listed as an "Unknow Service" since we're using a vendor-specific UUID.

Advertising Device  | Content of Advertisment Packet    | Service listed in the GATT table    |
------------ | ------------- | ------------- | 
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/advertising_device.png" width="250">  | <img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/advertisement_packet.png" width="250 "> | <img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/service.png" width="250"> |


### Step 4 - Adding a Custom Value Characteristic to the Custom Service.

A service is nothing with out a characteristic, so lets add one of those by creating the custom_value_char_add function. As this is a function that we want to be public, i.e. available to the application, we need to declare it in the ble_cus.h 


```c
/* This code belongs in ble_cus.h*/

/**@brief Function for adding the Custom Value characteristic.
 *
 * @param[in]   p_cus        Custom Service structure.
 * @param[in]   p_cus_init   Information needed to initialize the service.
 *
 * @return      NRF_SUCCESS on success, otherwise an error code.
 */
static uint32_t custom_value_char_add(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init);
```
Now, back in ble_cus.c, the first thing we have to do is to add several metadata variables that we will later populate. 


```c
/* This code belongs in ble_cus.c*/

/**@brief Function for adding the Custom Value characteristic.
 *
 * @param[in]   p_cus        Custom Service structure.
 * @param[in]   p_cus_init   Information needed to initialize the service.
 *
 * @return      NRF_SUCCESS on success, otherwise an error code.
 */
static uint32_t custom_value_char_add(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    uint32_t            err_code;
    ble_gatts_char_md_t char_md;
    ble_gatts_attr_md_t cccd_md;
    ble_gatts_attr_t    attr_char_value;
    ble_uuid_t          ble_uuid;
    ble_gatts_attr_md_t attr_md;
}
```
Now starts the rather tedious part of populating all these variables, we'll start with char_md, which sets the properties that will be displayed to the central during service discovery.

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&char_md, 0, sizeof(char_md));

    char_md.char_props.read   = 1;
    char_md.char_props.write  = 1;
    char_md.char_props.notify = 0; 
    char_md.p_char_user_desc  = NULL;
    char_md.p_char_pf         = NULL;
    char_md.p_user_desc_md    = NULL;
    char_md.p_cccd_md         = NULL; 
    char_md.p_sccd_md         = NULL;
}
```
So we want to be able to both write and read to our Custom Value characteristic, but we do not want to enable the notify property until later. Next we're going to populate the attr_md, which actually sets the properties( i.e. accessability of the attribute).

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&attr_md, 0, sizeof(attr_md));

    attr_md.read_perm  = p_cus_init->custom_value_char_attr_md.read_perm;
    attr_md.write_perm = p_cus_init->custom_value_char_attr_md.write_perm;
    attr_md.vloc       = BLE_GATTS_VLOC_STACK;
    attr_md.rd_auth    = 0;
    attr_md.wr_auth    = 0;
    attr_md.vlen       = 0;
}
```  

The permissions set in the attr_md struct should correspond with the properties set in the characteristic metadata struct char_md. We're going to provide the permissions in the Custom Service init structure that we pass to ble_cus_init in services_init(). The .vloc option is set to BLE_GATTS_VLOC_STACK as we want the characteristic to be stored in the SoftDevice RAM section and not in the Application RAM section. 

The next variable that we have to populate is ble_uuid, which is going to hold our CUSTOM_VALUE_CHAR_UUID and is of the same type as the CUSTOM_SERVICE_UUID_BASE, i.e. vendor specific, which we specified in the .uuid_type field of Custom Service strucure when we added the CUSTOM_SERVICE_UUID_BASE to the BLE stack's table.

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_VALUE_CHAR_UUID;
```

Next, we're going to populate the attr_char_value struct, which sets the UUID, points to the attribute metadata and sets the size of the characteristic, in our case a single byte(uint8_t). 

```c
    /* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&attr_char_value, 0, sizeof(attr_char_value));

    attr_char_value.p_uuid    = &ble_uuid;
    attr_char_value.p_attr_md = &attr_md;
    attr_char_value.init_len  = sizeof(uint8_t);
    attr_char_value.init_offs = 0;
    attr_char_value.max_len   = sizeof(uint8_t);
```

Finally, we're done populating structs and we can add our characteristic by calling sd_ble_gatts_characteristic_add() with the structs as arguments.

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

err_code = sd_ble_gatts_characteristic_add(p_cus->service_handle, &char_md,
                                               &attr_char_value,
                                               &p_cus->custom_value_handles);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    return NRF_SUCCESS;
```

After all that hard work (copy-pasting) your custom_value_char_add() function should look like this. 

```c
/* This code belongs in ble_cus.c*/

static uint32_t custom_value_char_add(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    uint32_t            err_code;
    ble_gatts_char_md_t char_md;
    ble_gatts_attr_md_t cccd_md;
    ble_gatts_attr_t    attr_char_value;
    ble_uuid_t          ble_uuid;
    ble_gatts_attr_md_t attr_md;

    memset(&char_md, 0, sizeof(char_md));

    char_md.char_props.read   = 1;
    char_md.char_props.write  = 1;
    char_md.char_props.notify = 0; 
    char_md.p_char_user_desc  = NULL;
    char_md.p_char_pf         = NULL;
    char_md.p_user_desc_md    = NULL;
    char_md.p_cccd_md         = NULL; 
    char_md.p_sccd_md         = NULL;
		
    memset(&attr_md, 0, sizeof(attr_md));

    attr_md.read_perm  = p_cus_init->custom_value_char_attr_md.read_perm;
    attr_md.write_perm = p_cus_init->custom_value_char_attr_md.write_perm;
    attr_md.vloc       = BLE_GATTS_VLOC_STACK;
    attr_md.rd_auth    = 0;
    attr_md.wr_auth    = 0;
    attr_md.vlen       = 0;

    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_VALUE_CHAR_UUID;

    memset(&attr_char_value, 0, sizeof(attr_char_value));

    attr_char_value.p_uuid    = &ble_uuid;
    attr_char_value.p_attr_md = &attr_md;
    attr_char_value.init_len  = sizeof(uint8_t);
    attr_char_value.init_offs = 0;
    attr_char_value.max_len   = sizeof(uint8_t);

    err_code = sd_ble_gatts_characteristic_add(p_cus->service_handle, &char_md,
                                               &attr_char_value,
                                               &p_cus->custom_value_handles);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    return NRF_SUCCESS;
}
```
 The final step is to call custom_value_char_add() at the end of ble_cus_init the service has been added, i.e. 

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_init(ble_cus_t * p_cus, const ble_cus_init_t * p_cus_init)
{
    if (p_cus == NULL || p_cus_init == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t   err_code;
    ble_uuid_t ble_uuid;

    // Initialize service structure
    p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;

    // Add Custom Service UUID
    ble_uuid128_t base_uuid = {CUSTOM_SERVICE_UUID_BASE};
    err_code =  sd_ble_uuid_vs_add(&base_uuid, &p_cus->uuid_type);
    VERIFY_SUCCESS(err_code);
    
    ble_uuid.type = p_cus->uuid_type;
    ble_uuid.uuid = CUSTOM_SERVICE_UUID;

    // Add the Custom Service
    err_code = sd_ble_gatts_service_add(BLE_GATTS_SRVC_TYPE_PRIMARY, &ble_uuid, &p_cus->service_handle);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    // Add Custom Value characteristic
    return custom_value_char_add(p_cus, p_cus_init);
}
```
Compile the project and flash it to you nRF5x DK. If you open the nRF Connect app on your smartphone, scan and connect to the device, you should see that the characteristic has been added by clicking on the service, as shown in the screenshot below.

Service and Characteristic  | 
------------ |
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/service_and_char.png" width="250"> |


### Step 5 - Handling events from the SoftDevice.
Great, we now have a Custom Service and a Custom Value Characteristic, but we want to be able to write to the characteristic and perform a specific task based on the value that was written to the characteristic, e.g. turn on a LED. However, before we can do that we need to do some event handling in ble_cus.h and ble_cus.c. 


Lastly, we're going to add the ble_cus_on_ble_evt function decleration to  ble_cus.h, which will handle the events of the ble_cus_evt_type_t from our service.

```c
/* This code belongs in ble_cus.h*/

/**@brief Function for handling the Application's BLE Stack events.
 *
 * @details Handles all events from the BLE stack of interest to the Battery Service.
 *
 * @note 
 *
 * @param[in]   p_ble_evt  Event received from the BLE stack.
 * @param[in]   p_context  Custom Service structure.
 */
void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context);
```

We're now going to implement the ble_cus_on_ble_evt event handler in ble_cus.c.Upon entry its considered good practice to check that none of the pointers that we provided as arguments are NULL. 

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;
    
    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
}
```

After the NULL check we're going to add a switch-case to check which event that has been propagated to the application by the SoftDevice. 

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;
    
    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            break;

        default:
            // No implementation needed.
            break;
    }
}
```

For now we only need to care about the BLE_GAP_EVT_CONNECTED and BLE_GAP_EVT_DISCONNECTED events. So let's create the two functions on_connect() and on_disconnect(), starting with on_connect(). The only thing we need to do when we get the Connect event is to assign the connection handle in the Custom Service structure to the connection handle that is passed with the event. 

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Connect event.
 *
 * @param[in]   p_cus       Custom Service structure.
 * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_connect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    p_cus->conn_handle = p_ble_evt->evt.gap_evt.conn_handle;
}
```

Similarly, when we get the Disconnect event, the only thing we need to do is invalidate the connection handle in the Custom Service structure since the connection is now dead.

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Disconnect event.
 *
 * @param[in]   p_cus       Custom Service structure.
 * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_disconnect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    UNUSED_PARAMETER(p_ble_evt);
    p_cus->conn_handle = BLE_CONN_HANDLE_INVALID;
}
```

Now that we have one function for each event we only need to call the function in ble_cus_on_ble_evt, i.e.

```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;

    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            on_connect(p_cus, p_ble_evt);
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            on_disconnect(p_cus, p_ble_evt);
            break;

        default:
            // No implementation needed.
            break;
    }
}
```

The last thing we have to do is to make sure that our ble_cus_on_ble_evt event handler function receives SoftDevice events. This is done registering the ble_cus_on_ble_evt event handler as event observer using the NRF_SDH_BLE_OBSERVER() macro. It is convenient to do this within the BLE_CUS_DEF macro that we defined in ble_cus.h, which should be modfied as shown below 

```c
/* This code belongs in ble_cus.h*/

#define BLE_CUS_DEF(_name)                                                                          \
static ble_cus_t _name;                                                                             \
NRF_SDH_BLE_OBSERVER(_name ## _obs,                                                                 \
                     BLE_HRS_BLE_OBSERVER_PRIO,                                                     \
                     ble_cus_on_ble_evt, &_name)

```

Compile your project and verify that there are no errors before you proceed to the next step. 

### Step 6 - Handling the Write event from the SoftDevice.

 Ok, now we really want to be able to write to the characteristic and perform a specific task based on the value that was written to the characteristic, e.g. turn on a LED. How are we going to that? You guessed it! More event handling!

 Whenever a characteristic is written to, a BLE_GATTS_EVT_WRITE event will be propagated to the application and dispatched to the functions in ble_evt_dispatch(). So this means that we need to add another case to our ble_cus_on_ble_evt switch-case statement, namely BLE_GATTS_EVT_WRITE


 ```c
/* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;

    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            on_connect(p_cus, p_ble_evt);
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            on_disconnect(p_cus, p_ble_evt);
            break;
        case BLE_GATTS_EVT_WRITE:
            break;
        default:
            // No implementation needed.
            break;
    }
}
```

Just like we did for the BLE_GAP_EVT_CONNECTED and BLE_GAP_EVT_DISCONNECTED we're going to create a on_write function that should be called when we get the Write event. 

```c
/* This code belongs in ble_cus.c*/

/**@brief Function for handling the Write event.
 *
 * @param[in]   p_cus       Custom Service structure.
 * @param[in]   p_ble_evt   Event received from the BLE stack.
 */
static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{

}
```

Now, once we get the Write event we have to get hold of the Write event parameters that are passed with the event and we have to verify that the the handle that is written to matches the Custom Value Characteristic handle, i.e.

```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Check if the handle passed with the event matches the Custom Value Characteristic handle.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        // Put specific task here. 
    }
}
```

So lets say that our specifc task is to toggle a LED on the nRF5x DK every time the Custom Value Characteristic is written to. We can do this by calling nrf_gpio_pin_toggle on one of the pins connected to the nRF5x DK LEDs, e.g. LED4. In order to do this we'll have to include boards.h and nrf_gpio.h in ble_cus.h as well as call nrf_gpio_pin_toggle in the on_write function


```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Check if the handle passed with the event matches the Custom Value Characteristic handle.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        nrf_gpio_pin_toggle(LED_4); 
    }
}
```

We'we now implemented the necessary event handling so on_write() should be added to the ble_cus_on_ble_evt() function under the BLE_GATTS_EVT_WRITE case, i.e.


 ```c
 /* This code belongs in ble_cus.c*/

void ble_cus_on_ble_evt( ble_evt_t const * p_ble_evt, void * p_context)
{
    ble_cus_t * p_cus = (ble_cus_t *) p_context;

    if (p_cus == NULL || p_ble_evt == NULL)
    {
        return;
    }
    
    switch (p_ble_evt->header.evt_id)
    {
        case BLE_GAP_EVT_CONNECTED:
            on_connect(p_cus, p_ble_evt);
            break;

        case BLE_GAP_EVT_DISCONNECTED:
            on_disconnect(p_cus, p_ble_evt);
            break;
        case BLE_GATTS_EVT_WRITE:
            on_write(p_cus, p_ble_evt);
            break;
        default:
            // No implementation needed.
            break;
    }
}
```

However, all this will be for nothing if we do not to allow the peer to actually write and/or read from the characteristic value. This is done by adding two lines to services_init() in main.c before we call ble_cus_init(). 

```c
/* This code belongs in services_init() in main.c*/

BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cus_init.custom_value_char_attr_md.read_perm);
BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cus_init.custom_value_char_attr_md.write_perm);
```

These two lines sets the write and read permissions to the characteristic value attribute to open, i.e. the peer is allowed to write/read the value without encrypting the link first. Now, try writting to the characteristic using nRF Connect for Desktop or Android/iOS. Every time a value is written to the characteristic, LED4 on the nRF5x DK should toogle. 

Write button  | Write value   | 
------------ | ------------- | 
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/write_to_char_1.png" width="250">  | <img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/write_to_char_2.png" width="250"> | 


**Challenge 1:** p_evt_write also has a data field. Use the data to decide if the LED is to be turned on or off. 

### Step 7 - Propagating Custom Service events to the application

Until now we've only handled the events that are propagated by the SoftDevice, but in some cases it makes sense to propagate events to the application. 

In order to do this we need to add an event handler to our Custom Service Init structure and Custom Service structure

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service init structure. This contains all options and data needed for
 *        initialization of the service.*/
typedef struct
{
    ble_cus_evt_handler_t         evt_handler;                    /**< Event handler to be called for handling events in the Custom Service. */
    uint8_t                       initial_custom_value;           /**< Initial custom value */
    ble_srv_cccd_security_mode_t  custom_value_char_attr_md;     /**< Initial security level for Custom characteristics attribute */
} ble_cus_init_t;

/**@brief Custom Service structure. This contains various status information for the service. */
struct ble_cus_s
{
    ble_cus_evt_handler_t         evt_handler;                    /**< Event handler to be called for handling events in the Custom Service. */
    uint16_t                      service_handle;                 /**< Handle of Custom Service (as provided by the BLE stack). */
    ble_gatts_char_handles_t      custom_value_handles;           /**< Handles related to the Custom Value characteristic. */
    uint16_t                      conn_handle;                    /**< Handle of the current connection (as provided by the BLE stack, is BLE_CONN_HANDLE_INVALID if not in a connection). */
    uint8_t                       uuid_type; 
};
```

After adding the event handler to the structures we must make sure that we initialize our service correctly in ble_cus_init()

```c
 /* This code belongs in ble_cus_init() in ble_cus.c*/

// Initialize service structure
p_cus->evt_handler               = p_cus_init->evt_handler;
p_cus->conn_handle               = BLE_CONN_HANDLE_INVALID;
```

Next, we need to declare an event type specific to our service

```c
 /* This code belongs in ble_cus.h*/

typedef enum
{
    BLE_CUS_EVT_DISCONNECTED,
    BLE_CUS_EVT_CONNECTED
} ble_cus_evt_type_t;
```
For now we're only going to add the BLE_CUS_EVT_CONNECTED and BLE_CUS_EVT_DISCONNECTED events, but we'll add some additional events later in the tutorial. 

After declaring the event type we need to declare an event structure that holds a ble_cus_evt_type_t event, i.e. 

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service event. */
typedef struct
{
    ble_cus_evt_type_t evt_type;                                  /**< Type of event. */
} ble_cus_evt_t;
```

 Next, we need declare the Custom Service event handler type 

```c
 /* This code belongs in ble_cus.h*/

/**@brief Custom Service event handler type. */
typedef void (*ble_cus_evt_handler_t) (ble_cus_t * p_cus, ble_cus_evt_t * p_evt);
```

Now, back in main we're going to create the event handler function on_cus_evt, which takes the same parameters as the ble_cus_evt_handler_t type.

```c
 /* This code belongs in main.c*/

/**@brief Function for handling the Custom Service Service events.
 *
 * @details This function will be called for all Custom Service events which are passed to
 *          the application.
 *
 * @param[in]   p_cus_service  Custom Service structure.
 * @param[in]   p_evt          Event received from the Custom Service.
 *
 */
static void on_cus_evt(ble_cus_t     * p_cus_service,
                       ble_cus_evt_t * p_evt)
{
    switch(p_evt->evt_type)
    {
        case BLE_CUS_EVT_CONNECTED:
            break;

        case BLE_CUS_EVT_DISCONNECTED:
              break;

        default:
              // No implementation needed.
              break;
    }
}
```

Now, in order to propagate events from our service we need to assign the on_cus_evt function as the event handler function of our service when we initialize the Custom Service. This is done by setting the .evt_handler field of the cus_init struct equal to on_cus_evt in services_init() in main.c, i.e.

```c
 /* This code belongs in services_init in main.c*/

    // Set the cus event handler
    cus_init.evt_handler                = on_cus_evt;
```

We can now invoke this event handler from ble_cus.c by calling p_cus->evt_handler(p_cus, &evt) and as an example we'll invoke the event handler when we get the BLE_GAP_EVT_CONNECTED event, i.e. in the on_connect() function. It's fairly straight forward,  we simply declare a ble_cus_evt_t variable and set its .evt_type field to the BLE_CUS_EVT_CONNECTED and then invoke the event handler with said event. 

```c
 /* This code belongs in ble_cus.c*/

static void on_connect(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    p_cus->conn_handle = p_ble_evt->evt.gap_evt.conn_handle;

    ble_cus_evt_t evt;

    evt.evt_type = BLE_CUS_EVT_CONNECTED;

    p_cus->evt_handler(p_cus, &evt);
}
```

### Step 8 - Notifying the Custom Value Characteristic

Next, we're going to add the ble_cus_custom_value_update function decleration to the ble_cus.h file, which we're going to use to update our Custom Value Characteristic.

```c
 /* This code belongs in ble_cus.h*/

/**@brief Function for updating the custom value.
 *
 * @details The application calls this function when the cutom value should be updated. If
 *          notification has been enabled, the custom value characteristic is sent to the client.
 *
 * @note 
 *       
 * @param[in]   p_cus          Custom Service structure.
 * @param[in]   Custom value 
 *
 * @return      NRF_SUCCESS on success, otherwise an error code.
 */

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value);
```
Back in ble_cus.c, we're going to continue the good practice of checking that the pointer we passed as an argument is'nt NULL

```c
 /* This code belongs in ble_cus.c*/

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value){
    if (p_cus == NULL)
    {
        return NRF_ERROR_NULL;
    }
}
```

Next, we're going to update the value in the GATT table with our custom_value that we passed as an argument to the ble_cus_custom_value_update() function. 

```c
/* This code belongs in ble_cus_custom_value_update() in ble_cus.c*/

uint32_t err_code = NRF_SUCCESS;
ble_gatts_value_t gatts_value;

// Initialize value struct.
memset(&gatts_value, 0, sizeof(gatts_value));

gatts_value.len     = sizeof(uint8_t);
gatts_value.offset  = 0;
gatts_value.p_value = &custom_value;

// Update database.
err_code = sd_ble_gatts_value_set(p_cus->conn_handle,
                                    p_cus->custom_value_handles.value_handle,
                                    &gatts_value);
if (err_code != NRF_SUCCESS)
{
    return err_code;
}
```

After updating the value in the GATT table we're ready to notify our peer that the value of our Custom Value Characteristic has changed. 

```c
/* This code belongs in ble_cus_custom_value_update() in ble_cus.c*/

// Send value if connected and notifying.
if ((p_cus->conn_handle != BLE_CONN_HANDLE_INVALID)) 
{
    ble_gatts_hvx_params_t hvx_params;

    memset(&hvx_params, 0, sizeof(hvx_params));

    hvx_params.handle = p_cus->custom_value_handles.value_handle;
    hvx_params.type   = BLE_GATT_HVX_NOTIFICATION;
    hvx_params.offset = gatts_value.offset;
    hvx_params.p_len  = &gatts_value.len;
    hvx_params.p_data = gatts_value.p_value;

    err_code = sd_ble_gatts_hvx(p_cus->conn_handle, &hvx_params);
}
else
{
    err_code = NRF_ERROR_INVALID_STATE;
}

return err_code;

```
The first thing we should do is to verify that we have a valid connection handle, i.e. that we're actually connected to a peer, if not we should return an error indicating that we're in an invalid state. Next, we set the ble_gatts_hvx_params_t, which contains the handle of custom value attribute, the type(i.e. if its a Notification or Indication), the value offsett( only used if its larger than 20bytes), the length and the data. After setting the hvx_params, we notify the peer by calling sd_ble_gatts_hvx(). Lastly, we should return the error code. 

After adding the code snippets above ble_cus_custom_value_update() should look like below

```c
/* This code belongs in ble_cus.c*/

uint32_t ble_cus_custom_value_update(ble_cus_t * p_cus, uint8_t custom_value)
{
    NRF_LOG_INFO("In ble_cus_custom_value_update. \r\n"); 
    if (p_cus == NULL)
    {
        return NRF_ERROR_NULL;
    }

    uint32_t err_code = NRF_SUCCESS;
    ble_gatts_value_t gatts_value;

    // Initialize value struct.
    memset(&gatts_value, 0, sizeof(gatts_value));

    gatts_value.len     = sizeof(uint8_t);
    gatts_value.offset  = 0;
    gatts_value.p_value = &custom_value;

    // Update database.
    err_code = sd_ble_gatts_value_set(p_cus->conn_handle,
                                      p_cus->custom_value_handles.value_handle,
                                      &gatts_value);
    if (err_code != NRF_SUCCESS)
    {
        return err_code;
    }

    // Send value if connected and notifying.
    if ((p_cus->conn_handle != BLE_CONN_HANDLE_INVALID)) 
    {
        ble_gatts_hvx_params_t hvx_params;

        memset(&hvx_params, 0, sizeof(hvx_params));

        hvx_params.handle = p_cus->custom_value_handles.value_handle;
        hvx_params.type   = BLE_GATT_HVX_NOTIFICATION;
        hvx_params.offset = gatts_value.offset;
        hvx_params.p_len  = &gatts_value.len;
        hvx_params.p_data = gatts_value.p_value;

        err_code = sd_ble_gatts_hvx(p_cus->conn_handle, &hvx_params);
    }
    else
    {
        err_code = NRF_ERROR_INVALID_STATE;
    }


    return err_code;
}
```

Like the characteristic metadata we need to set the metadata of the CCCD in custom_value_char_add(), which is done by adding the following snippet before the characteristic metadata is set. 

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/

    memset(&cccd_md, 0, sizeof(cccd_md));

    //  Read  operation on Cccd should be possible without authentication.
    BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.read_perm);
    BLE_GAP_CONN_SEC_MODE_SET_OPEN(&cccd_md.write_perm);
    
    cccd_md.vloc       = BLE_GATTS_VLOC_STACK;
```
We're setting the read and write permissions to the CCCD to open, i.e. no encryption is needed to write or read from the CCCD. The .vloc option is set to BLE_GATTS_VLOC_STACK, which means that the value of the CCCD will be stored in SoftDevice memory section and not in the application memory section. 

The next step is to add the Notify property to the Custom Value Characteristic and add a pointer to  the CCCD metadata in the characteristic metadata. This is done by modifying  the characteristic metadata properties in custom_value_char_add() function , i.e. setting .notify to 1 and .p_cccd_md to point to the CCCD metadata struct. 

```c
/* This code belongs in custom_value_char_add() in ble_cus.c*/
    char_md.char_props.notify = 1;  
    char_md.p_cccd_md         = &cccd_md; 
```

This will add a Client Characteristic Configuration Descriptor or CCCD to the Custom Value Characteristic which allows us to enable or disable notifications by writing to the CCCD. Notification is by default disabled and in order to enable it we have to write 0x0001 to the CCCD. Remember that everytime we write to a characteristic or one of its descriptors, we will get a Write event, thus we need to handle the case where a peer writes to the CCCD in the on_write() function. 

However, before modifying the on_write() function we need do is to add two additional event, BLE_CUS_EVT_NOTIFICATION_ENABLED and BLE_CUS_EVT_NOTIFICATION_DISABLED, to the ble_cus_evt_type_t enumeration in ble_cus.h

```c
/* This code belongs in ble_cus.h*/

/**@brief Custom Service event type. */
typedef enum
{
    BLE_CUS_EVT_NOTIFICATION_ENABLED,                             /**< Custom value notification enabled event. */
    BLE_CUS_EVT_NOTIFICATION_DISABLED,                            /**< Custom value notification disabled event. */
    BLE_CUS_EVT_DISCONNECTED,
    BLE_CUS_EVT_CONNECTED
} ble_cus_evt_type_t;
```

Back to the on_write() function, the first thing we should do is to add a if-statement that checks if the handle that is written to matches the handle of the CCCD and that the value has the correct length. If we pass this check we need to check wether notifications has been enabled or not. 

```c
/* This code belongs in on_write() in ble_cus.c*/

    // Check if the Custom value CCCD is written to and that the value is the appropriate length, i.e 2 bytes.
    if ((p_evt_write->handle == p_cus->custom_value_handles.cccd_handle)
        && (p_evt_write->len == 2)
       )
    {

        // CCCD written, call application event handler
        if (p_cus->evt_handler != NULL)
        {
            ble_cus_evt_t evt;

            if (ble_srv_is_notification_enabled(p_evt_write->data))
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_ENABLED;
            }
            else
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_DISABLED;
            }
            // Call the application event handler.
            p_cus->evt_handler(p_cus, &evt);
        }

    }
```

We do not actually need to enable notifications as this is done automatically by the SoftDevice. However, the SoftDevice will propagate the write event and based on this event we can decide if its ok to start notifying characteristic values or not. 

Adding the code-snippet above to the on_write() function should result in the following function. 

```c
/* This code belongs in ble_cus.c*/

static void on_write(ble_cus_t * p_cus, ble_evt_t const * p_ble_evt)
{
    ble_gatts_evt_write_t * p_evt_write = &p_ble_evt->evt.gatts_evt.params.write;
    
    // Custom Value Characteristic Written to.
    if (p_evt_write->handle == p_cus->custom_value_handles.value_handle)
    {
        nrf_gpio_pin_toggle(LED_4);
    }

    // Check if the Custom value CCCD is written to and that the value is the appropriate length, i.e 2 bytes.
    if ((p_evt_write->handle == p_cus->custom_value_handles.cccd_handle)
        && (p_evt_write->len == 2)
       )
    {
        // CCCD written, call application event handler
        if (p_cus->evt_handler != NULL)
        {
            ble_cus_evt_t evt;

            if (ble_srv_is_notification_enabled(p_evt_write->data))
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_ENABLED;
            }
            else
            {
                evt.evt_type = BLE_CUS_EVT_NOTIFICATION_DISABLED;
            }
            // Call the application event handler.
            p_cus->evt_handler(p_cus, &evt);
        }
    }
}
```

Now the last thing we have to do is to add the BLE_CUS_EVT_NOTIFICATION_ENABLED and BLE_CUS_EVT_NOTIFICATION_DISABLED to the on_cus_evt() event handler in main.c, i.e. 

```c
/* This code belongs in main.c*/

static void on_cus_evt(ble_cus_t     * p_cus_service,
                       ble_cus_evt_t * p_evt)
{

    switch(p_evt->evt_type)
    {
        case BLE_CUS_EVT_NOTIFICATION_ENABLED:
            break;

        case BLE_CUS_EVT_NOTIFICATION_DISABLED:
            break;

        case BLE_CUS_EVT_CONNECTED :
            break;

        case BLE_CUS_EVT_DISCONNECTED:
            break;

        default:
              // No implementation needed.
              break;
    }
}
```
Compile the project and flash it to you nRF5x DK. If you open the nRF Connect app and connect to the device you should see that a Client Characteristic Configuration Descriptor has been added under the characteristic as shown in the screen shot below.

Service, Characteristic and Descriptor  | 
------------ |
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/service_char_desc.png" width="500"> |

We're now ready to notify some values from the nRF52 DK to the nRF Connect app. In order to do that we're going to create an application timer that calls ble_cus_custom_value_update() at a regular interval and then start it when we get the BLE_CUS_EVT_NOTIFICATION_ENABLED event. So first, add the following define to the top of main.c

```c
/* This code belongs in main.c*/
#define NOTIFICATION_INTERVAL           APP_TIMER_TICKS(1000)
```

which is going to the timeout interval of our application timer. Next you will need to create an application timer id with the name m_notification_timer_id by calling the APP_TIMER_DEF  below the defines in main.c, i.e. 
```c
/* This code belongs in main.c*/
APP_TIMER_DEF(m_notification_timer_id);
```

Below this macro call you can declare a unsigned 8-bit variable called  m_custom_value, i.e. 

```c
/* This code belongs in main.c*/
static uint8_t m_custom_value = 0;
```

which will be used to hold the custom value we're going to notify to the nRF Connect app. Next, find the timers_init() functino in main.c and add the folling snippet to create the application timer

```c
/* This code belongs in timers_init() in main.c*/
    // Create timers.
    err_code = app_timer_create(&m_notification_timer_id, APP_TIMER_MODE_REPEATED, notification_timeout_handler);
    APP_ERROR_CHECK(err_code);
```

Great, we now have a application timer and the next step is to start the application timer when we receive the BLE_CUS_EVT_NOTIFICATION_ENABLED, i.e. add the following snippet under the BLE_CUS_EVT_NOTIFICATION_ENABLED case in the on_cus_evt event handler in main.c

```c
/* This code belongs in on_cus_evt() in main.c*/         
           err_code = app_timer_start(m_notification_timer_id, NOTIFICATION_INTERVAL, NULL);
           APP_ERROR_CHECK(err_code);
```

Similarly, we want the notification timer to stop when we get the BLE_CUS_EVT_NOTIFICATION_DISABLED event. Thus, we add the following snippet to the BLE_CUS_EVT_NOTIFICATION_DISABLED case in the on_cus_evt event handler in main.c

```c
/* This code belongs in on_cus_evt() in main.c*/           
           err_code = app_timer_stop(m_notification_timer_id);
           APP_ERROR_CHECK(err_code);
```

Lastly, we need to create the notification_timeout_handler which will increment the m_custom_value variable and then call ble_cus_custom_value_update by declaring the following function above timers_init() in main.c

```c
/* This code belongs in main.c*/  

/**@brief Function for handling the Battery measurement timer timeout.
 *
 * @details This function will be called each time the battery level measurement timer expires.
 *
 * @param[in] p_context  Pointer used for passing some arbitrary information (context) from the
 *                       app_start_timer() call to the timeout handler.
 */
static void notification_timeout_handler(void * p_context)
{
    UNUSED_PARAMETER(p_context);
    ret_code_t err_code;
    
    // Increment the value of m_custom_value before nortifing it.
    m_custom_value++;
    
    err_code = ble_cus_custom_value_update(&m_cus, m_custom_value);
    APP_ERROR_CHECK(err_code);
}
```

Compile the project and flash it to your nRF52 DK. Open the nRF Connect app, connect to the nRF52 DK and enable notification by clicking the button highlighted in the screenshot below

Enable Notifications  | 
------------ |
<img src="https://github.com/bjornspockeli/custom_ble_service_example/blob/master/images/enable_notif.png" width="500"> |

You should now see a value field appear below the Unknown Characteristics properties and the value should be incrementing every second. Congratulations you have now created a custom service and notified custom values!
