# CA-Test-AGG-lib

To download the most recent project without installing GIT, please press the green "Clone or Download" button and select "Download ZIP".



The Steps 
-------------------

Here is our action plan for creating a simple access control application. To follow the steps, download **test_agg_lib.zip**  

The archive contains all the implementation steps as listed below:


1. *Prepare* (install, configure) the AggreGate server and client. 

2. *Embryonic project* — the device connects to the server, does nothing else (**test_agg_lib_1**). 

3. *Add setting A-variables* for storing the device configuration (**test_agg_lib_2**). 

4. *Add table A-variable* for storing user codes (**test_agg_lib_3**). 

5. *Add an A-function* for remote door unlocking (**test_agg_lib_4**). 

6. *Generate instant A-event* on device boot (**test_agg_lib_5**). 

7. *Generate stored A-event* for access control activity reporting (**test_agg_lib_6**). 

8. *Add the glue code* that ties it all together (**test_agg_lib_6**). 

   ​


Preparing the AggreGate Server
-------------------

The AggreGate Server and AggreGate client software can be downloaded here:

http://aggregate.tibbo.com/downloads.html

Be careful to select correct files!

You can install both the Server and Client components on the same PC.

The demo application you are about to test expects the AggreGate server to have an account named **admin**.



Step 1: The Embryo
------------------

This step corresponds to **test_agg_lib_1**.

In this step we are going to create an embryonic application. It will be able to connect to the AggreGate server and transmit the minimally necessary identifying information.

The success of this step depends on many factors. You have to correctly install and prepare the **AggreGate Server** and **AggreGate Client** software. You have to use matching account names on the server and on the device. Passwords must match. The IP address of your device must be correctly chosen. The IP address of the server must be specified correctly, and so on. In other words, pay attention to all the details!

### The steps 

We assume you are not going to type everything in from scratch and just open the **test_agg_lib_1** project. Notice that...

1. The project contains the SOCK and TIME libraries [not yet documented]. 

2. **Aggregate.tbs** and **aggregate.tbh** are added to the project as well (from current_library_set\aggregate\trunk\). There is also a necessary line in **global.tbh: include "aggregate\trunk\aggregate.tbh".** 

3. There is an **aggregate.xtxt** configuration file with type= **configuration file**, and format= **AggreGate (AGG) library**. There is a line in global.tbh that is required for the configuration file to work correctly: **includepp "aggregate.xtxt".** 

4. In the configuration file, **Debug Printing** is enabled. This allows you to "see what's going on". Don't forget to disable this later, after you've made sure that the library operates as expected. 

5. Also, the **Description** field is set to *AGG_TEST_DEVICE*. The **Context Type** is *Apoint*. 

6. AGG library event procedures are added to event handlers: 

   - ***agg_proc_timer()*** is in the ***on_sys_timer()*** event handler (this library assumes that this event is generated twice per second);
   - ***agg_proc_data()*** is in the ***on_sock_data_arrival()*** event handler;
   - ***agg_proc_sock_event()*** is in the ***on_sock_event()*** event handler (the exact line is **agg_proc_sock_event(newstate,newstatesimple).**

7. There are empty callback procedures in device.tbs: 

   - **callback_agg_get_firmware_version()**;
   - **callback_agg_pre_buffrq()**;
   - **callback_agg_buff_released()**;
   - **callback_agg_error()**;
   - **callback_agg_device_function()**;
   - **callback_agg_synchronized()**.

8. ***Callback_agg_get_firmware_version()*** contains the code that returns the firmware version string. There is a **#define FIRMWARE_VERSION** in **global.tbh**.  

9. ***on_sys_init()*** calls ***agg_start()*** — this is required to make the library operational. Notice how we check the code returned by ***agg_start()***!  

10. Ours is a simple demo and we are sure we won't have memory shortage. In large projects where a lot of things compete for memory you may get the ***callback_agg_pre_buffrq()*** call. This will indicate that the AGG library doesn't have enough memory for its buffers and that you need to clear up this memory right inside ***callback_agg_pre_buffrq()***. Failure to do so will result in the EN_AGG_STATUS_INSUFFICIENT_BUFFER_SPACE code returned by ***agg_start()***. 


### The result

With debug printing enabled, you will see the following output in TIDE:

    AGG()> —-START---
    AGG()> connection established
    AGG()> Device synchronized with the server

At the same time your device will appear in AggreGate. Nothing fancy yet, just the basics



![AGG1](http://docs.tibbo.com/taiko/test_agg_lib_1.png)



Step 2: Adding Setting A-variables
-------

This step corresponds to **test_agg_lib_2**.

In this step we are going to add necessary setting A-variables. These will be used to store the few configuration and status parameters our application requires.

### A-variables we are going to have are:

- The **BR** setting to store the baudrate of the serial port. This will allow us to select a standard baudrate from a list of baudrates (like 9600, 19200, etc.) 

- The **EG** setting to enable/disable access control event generation. 

- The **UT** setting to store the unlock time for the door. This is the number of half-second intervals that the door lock will stay unlocked for (once it is unlocked). 

- The **DS** setting for reading the current door state (closed or opened).  


### The steps

Looking now at the **test_agg_lib_2** project, notice that... 

1. The **STG** library is now in the project (and all related steps were taken, including calling ***stg_start()*** in ***on_sys_init()***, etc.). 

2. A way to initialize settings is provided. After each boot, the device will start blinking its red and green status LEDs. This will continue for 5 seconds. If you press the ***MD button*** within this time, ***stg_restore_multiple()*** will be called. Reboot the device after the green status LED is turned on. If you do not press the button within the 5-second time window, the application will continue running. All related code is in the ***on_sys_init()*** event handler. 

3. **Settings.xtxt** defines required settings. Notice that the **Timestamp** option is enabled in the setting **configurator**. This is because the AGG library requires settings to carry timestamps. 

4. **AggreGate.xtxt** lists required A-variables. 

5. There is an additional callback procedure in **device.tbs** — for ***callback_agg_convert_setting()***. Examine the code — it "converts" between the UT setting (measuring time in half-second intervals) and the UT A-variable (measuring time in seconds). You can now see this procedure's usefulness — it allows you to store values differently compared to how they are perceived in AggreGate. 


### The result

You will now be able to see newly created A-variables in the AggreGate Client (as shown on the screenshot below). Notice how all four A-variables appear on the **Access Control tab**? This is defined in the AggreGate configurator.

Notice also that each A-variable is displayed/edited differently:

- The Baudrate (**BR**) setting has a drop-down which shows all available values; 
- The Generate Event (**GE**) setting as a checkbox because it is of the boolean type; 
- The Unlock Time (**UT**) setting is just a plain value that is expressed in seconds (although the UT setting is in half-seconds). 
- The Door State (**DS**) setting displays custom values "closed" and "opened" and is read-only. 

Setting A-variables as we have them now are not doing anything useful on the device just yet. They simply exist as values. We will glue it all together later.



![AGG2](http://docs.tibbo.com/taiko/test_agg_lib_2.png)



### Define Required Settings

Add **BR**, **EG**, **UT**, and **DS** variables using STG library's configurator. You should end up with a setting table as shown below. Some things to notice:

- The **Timestamp** is enabled. This is required by the AggreGate library. 

- The **BR** setting has the maximum value of 13. This is because we will support 14 different baudrates (0~13). 

- The **EG** setting has the maximum value of 1. On the AggreGate side it will be boolean, hence there are only two possible values for it: 0 and 1. 

- The **UT** setting is limited to 255 max. This is measure in half-second intervals (this is how often the on_sys_timer event is generated). 

- The **DS** setting resides in the volatile memory (RAM). This is a read-only parameter that simply reflects the door state, so it makes no sense to put it into the EEPROM. 

  ​

![AGG3](http://docs.tibbo.com/taiko/test_agg_lib_2_stg.png)



Add **BR**, **EG**, **UT**, and **DS** setting A-variables using AGG library's configurator. You should end up with a setting table as shown below:



![AGG4](http://docs.tibbo.com/taiko/test_agg_lib_2_agg_general.png)



Note that Reference Library (Settings) must be enabled and correct setting configurator file must be specified (settings.xtxt). We are sure you will quickly find your way around the AggreGate configurator. Let's just briefly review some things worth noting. Double-click on the BR A-variable line — the Edit (A-variables) dialog will open.

- On the General tab, notice that the A-variable is Linked to the Settings Library. The Variable Name drop-down contents reflect available settings. Your A-variable is basically a "propagated" setting! You can also link to tables — this will be our next step's task.

- Notice also that the A-variable is Readable and Writable. All our A-variables are, except DS, which is only Readable (read-only).

- Description is set to Baudrate, and this is how this A-variable is visible in AggreGate.

- The Group is set to Access Control. This is where the Access Control tab you see in the AggreGate client comes from. The client will create separate tabs for all groups encountered. We only use one custom group in this project.

- Now switch to the Advanced tab and press Edit. Don't we have a lot here!

- Notice how the Field Type is Integer. The AggreGate server supports a different type set, and byte isn't on the list. The nearest suitable type is selected, which is integer.

- Selection Value is enabled and if you click ... you will see the whole list. Now you know where the drop-down selector on the Baudrate A-variable comes from.

- Maximum Limit will not allow you to set anything above 13.

- There are numerous other fields we haven't touched. Explore and you shall discover!

  ​

![AGG4](http://docs.tibbo.com/taiko/test_agg_lib_2_agg_dialog.png)



Moving now to the remaining A-variables, and only touching on the notable differences:

- The **EG** A-variable has its Field Type = Boolean.

- The **UT** A-variable is limited to 127.

- The **DS** A-variable has two custom Selection Values (opened and closed). it is only Readable, and not Writable.

  ​



Step 3: Adding Table A-variables
--------------------------------

This step corresponds to **test_agg_lib_3**.

This is an access control project, hence it requires a user table which will keep user names and ID codes. In this step we create this **user** table.

### The steps

Notice that...
1. The **TBL library** [not yet documented] is now in the project. Inspect the on_sys_init() event handler — it contains several things related to the library's operation: the flash disk is mounted and formatted if necessary. The flash disk is also formatted during the initialization. ***Tbl_start() [***not yet documented] is called.
2. The **FILENUM** library is now in the project — it is required by the TBL library.
3. **Tables.xtxt** defines the USER table.
4. **Aggregate.xtxt** contains new table A-variable.
5. Notice how the **AggreGate Hash** option is enabled in the table **configurator** [not yet documented]. This is because the AGG library requires this.

### The result

Now you have an editable **user** table! It isn't yet "connected" to anything in any useful way — we will take care of this later.

# ![AGG5](http://docs.tibbo.com/taiko/test_agg_lib_3.png)



### Define the User Table

This table **configurator [**not yet documented] screenshot shows the added **user** table. Note that...

- **AggreGate Hash** is enabled. This is required by the AggreGate library.

- The **user** table has one **key field.** It is the first one in the list — the user_id field.

  ​

![AGG6](http://docs.tibbo.com/taiko/test_agg_lib_3_tbl.png)



### Add the Table A-variable

These AggreGate configurator screenshots show the added user table A-variable.

![AGG7](http://docs.tibbo.com/taiko/test_agg_lib_3_agg_general.png)



![AGG8](http://docs.tibbo.com/taiko/test_agg_lib_3_agg_dialog.png)





## Step 4: Adding A-functions

This step corresponds to **test_agg_lib_4.**

A-functions are methods that the AggreGate server can execute on the device. In our access control project, we will add an "open door" A-function which will be used for remote unlocking of the door. This A-function will have an argument — for how long the door will have to remain unlocked.

The A-function's argument will be of an overriding nature. When set to 0, it will have no effect and the door will be unlocked for the period of time specified by the UT A-variable. When not zero, the argument will override the **UT** A-variable once.

### **The steps**

Notice that...

1. **Aggregate.xtxt** now contains the UL A-function.
2. Callback_agg_device_function() implements necessary code for executing the **UL** A-function.
3. Notice how Agg_record_decode() is used to extract the argument of the A-function.
4. Agg_record_encode() is used to return A-function value to the AggreGate server. The value is always "success" in our case.

### The result

Single-click on the device in the AggreGate tree, and locate Unlock the Door in the Related Actions pane below. Click Unlock the Door, and you will get a dialog requesting Door Unlock Duration. Input a value, press OK. The function will be executed and return Success.

Of course, there is no code (yet) that actually unlocks the door. We will take care of this later.

> **Note:**
> Can't see Unlock the Door in the list? Navigate away from your test device, i.e. click on something else in the tree. Come back — and you will see the newly added A-function

![AGG9](http://docs.tibbo.com/taiko/test_agg_lib_4.png)



### Adding A-function

These AggreGate configurator screenshots show the added UL A-function.

![AGG10](http://docs.tibbo.com/taiko/test_agg_lib_4_agg_general.png)

![AGG11](http://docs.tibbo.com/taiko/test_agg_lib_4_agg_dialog.png)



## Step 5: Firing Instant A-events

This step corresponds to **test_agg_lib_5**.

Instant A-events are sent to the AggreGate server "on the spot". They are "lightweight" — there is no dealing with storing them on the flash disk, etc. They are fast(er) — just send! They are more reliable — no flash disk complexity involved. They are also easier to be lost. Should your device lose power at a wrong moment, the instant A-event will be forgotten. Another limitation: instant A-events can only be fired successfully when there is an active connection to the AggreGate server. Bottom line: use instant A-events for events that are not-so-important and/or numerous.

In this step we add an instant A-event for reporting the booting (powering up) of the device.

### The steps

Notice that...
1. **Aggregate.xtxt** now contains the DB instant A-event. This event has only one field — the date/time of the event. It is possible to have additional fields, of course.
2. There is a new ***generate_boot_event()*** sub is in **device.tbs.** It calls ***agg_fire_instant_event()*** to send the instant A-event to the AggreGate server. We call ***generate_boot_event()*** from ***callback_agg_synchronized()***, not ***on_sys_init()***. As was explained above, you need to have an established connection to the AggreGate server in order to be able to fire instant A-events, and the connection is ready when ***callback_agg_synchronized()*** is called.
3. Notice how this procedure uses ***agg_record_encode()***.
4. Notice also the use of the TIME library [not yet documented] for producing the date/time field of correct format.



Notice how the ***add_fire_instant_event()*** call has the ***event_level*** argument. The level defined by this argument will override the default level set in the configurator, unless you set the **event_level**= *EN_AGG_EVENT_LEVEL_USE_DEFAULT*, in which case the default event level will be used.

### The result

To be able to see *instances* of your new event, right-click on the device in the tree and choose **Monitor Related Events**.

![AGG12](http://docs.tibbo.com/taiko/test_agg_lib_5.png)



### Adding Instant A-event

These AggreGate configurator screenshots show the added instant A-event.

Notice the **Default Level** field. This is the event level that will be reported if ***agg_fire_instant_event()*** is called with **event_level**=*EN_AGG_EVENT_LEVEL_USE_DEFAULT*.



![AGG13](http://docs.tibbo.com/taiko/test_agg_lib_5_agg_general.png)

![AGG14](http://docs.tibbo.com/taiko/test_agg_lib_5_agg_dialog.png)



## Step 6: Handling Stored A-events

This step corresponds to **test_agg_lib_6**.

Stored A-events, as their name implies, are first stored on the device. Stored A-events are kept in the log table, and log tables are handled by the **TBL library** [not yet documented]. Each stored A-event is kept in the log until the AGG library has a chance to send it to the AggreGate server. Once recorded, these A-events won't be lost. Having an AggreGate server connection is not a precondition for the generation of stored A-events. The disadvantage is somewhat heavier implementation and slower event handling speed.

There will be a separate log table [not yet documented] (and a file on the flash disk) for each type of stored A-event in your application. This is because different A-event types can potentially have a different set of fields and, hence, the different storage format.

Another important point to discuss: if you come from a field like access control or IT, then you may be accustomed to a certain way of using the term "event". You probably dealt with events like "access granted", "access violation", "access denied", and so on. Each one of those is considered to be a separate "event".
With AggreGate, if it comes from the same log table, then it is the same A-event. The ACE A-event below is generated on "access granted", "access violation", etc., yet it is all the same single event called the "Access Control Event (ACE)". It is the event description that differentiates each ACE A-event instance. Keep this in mind when reviewing the code added in step 6.

### The steps

Notice that...

1. The **TBL library** [not yet documented] is already in the project. We added it when we were creating the USER table.
2. **Tables.xtxt** now contains the **ACE** table. Note how there are three fields:
   - The **DT** field for storing the date and time of the event.
   - The **DS** field carrying the event description (like "access granted", "access violation", etc.). Notice how this field's size is only four characters — how can we possible fit a meaningful description in four characters?.. Read on and you will know!
   - The **AEL** field for event the level. Stored A-events, like instant A-events, have the default event level which can be overridden. To be able to override, have a special field named **AEL** (A-event level) in your log table. This field must be of the byte type, with possible values from 0 to 5. Once you have the **AEL** field in the table, the event level for each particular event instance will come from this field.
3. **Aggregate.xtxt** now contains the **ACE** stored A-event. It "links" to the ACE table.
4. Call to ***agg_proc_data_sent()*** is in ***on_sock_data_sent()*** event handler.
5. New ***generate_door_event()*** sub is in device.tbs. We call this from ***on_button_pressed()***. You don't have to be connected to the AggreGate server in order to be able to generate this event — that's the beauty of stored A-events. Whenever you press the MD button, one ACE event is generated. The description it carries for now is always "ACE". This is temporary — we will add real descriptions in the next step.
6. Notice how this procedure uses ***agg_stored_event_added()***. You have to use it every time you add a stored A-event. It "nudges" the AGG library, i.e. tells it that there are stored events that haven't been sent to the server yet.
7. Notice that ***agg_proc_stored_events()*** is also called(). **This is completely optional** and serves to speed up the sending of the stored A-event to the AggreGate server. The AGG library already uses this procedure internally. Calling this "manually" just after doing agg_stored_event_added() may speed things up a bit, especially on big projects with heavy and complicated code.
8. Our sample code also makes use of ***callback_agg_convert_event_field()***. If you put a breakpoint there you will notice that this procedure is invoked every time you generate a stored A-event. The procedure is called separately for each even't field, except the ACE field. The idea is the same as for the ***callback_agg_convert_setting*** call, which was discussed in Adding Setting A-variables: you get a chance to convert or change a field's value before sending it to the AggreGate Server. In our code we use the call to convert short event descriptions (abbreviations, really) into comprehensible lines of text!



### The result

To be able to see instances of your new event, right-click on the device in the tree and choose Monitor Related Events.

![AGG15](http://docs.tibbo.com/taiko/test_agg_lib_6.png)



### Define the ACE Table

This table **configurator** [not yet documented] screenshot shows the added ACE log table.

- Notice the **AEL** (A-Event level) field. Since the **ACE** log table has this field, the level of each ACE event instance will be taken from this field.

- Notice also that the **DS** (description) field accommodates up to 4 characters — read about our processing technique for this field here.

  ![AGG16](http://docs.tibbo.com/taiko/test_agg_lib_6_tbl.png)



### Define the ACE Stored Event

These AggreGate configurator screenshots show the added ACE stored A-event.

Notice the note in red under the **Default Level**. It means that the **AEL** (A-event level) field is found in the ACE table and the event level will be defined separately for each event *instance* in accordance with the AEL field's value.



![AGG16](http://docs.tibbo.com/taiko/test_agg_lib_6_agg_general.png)

![AGG17](http://docs.tibbo.com/taiko/test_agg_lib_6_agg_dialog.png)



## Step 7: Gluing it All Together

This step corresponds to **test_agg_lib_7.**

And now, ladies and gentlemen, let's crown this application with some glue code. Observe the code and notice the following...

1. Handling of the BR setting A-variable:
   - Every time we select new baudrate in AggreGate, callback_stg_post_set() is invoked. Our code there calls baudrate_set(). Don't understand how this works? Read Using Pre-gets and Post-sets.
   - We also call baudrate_get() from callback_stg_pre_get().
   - Baudrate_get() is also invoked from setup_serial_port(), which is called from the on_sys_init().
2. Handling of the UT setting A-variable: again, we just return the actual door status in callback_stg_pre_get(). If you may recall, the MD button is used as a make-believe door sensor.
3. The GE setting A-variable now properly allows or prevents the events from generating.
4. Handling of user access codes is in the on_ser_data_arrival event handler.




## Step 8: Adding Bells and Whistles



This step does not exist yet — you can just go ahead and create it yourself.
You can enhance the access control demo in a myriad ways. We will just point out additional facilities offered by the AGG library.

Note that...
1. You can always stop the library and free up buffer memory by calling ***agg_stop()***. When your application does this, it will get ***callback_agg_buff_released()*** invoked.
2. You can use ***agg_get_connection_state()*** to poll the AGG library for it connection state. You can then display the state on the LCD screen (if your device has it), using LEDs, or in some other way.
3. ***Callback_agg_error()*** provides a unified place to respond to errors generated by the AGG library.
4. Some hardware projects require precise timekeeping (don't we know this!). If you are not satisfied with the internal RTC of your device (such as the EM1000), or you are using a device that has no RTC (EM500) then you can always connect your own RTC chip and write your own RTC routines:

   - Enable (check) **Using Custom RTC** in the AggreGate configurator.
   - Add the ***Callback_agg_rtc_sg()*** procedure to your code.




For more detail about this project, please visit [Project Description Page](http://tibbo.com/programmable/applications/examples/agg_lib_test.html).