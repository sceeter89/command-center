# Home Command Center
It's a plugin based software that run on your Raspberry Pi, constantly monitoring home state basing on multiple sensors
such as:
* 1-wire thermometer
* Alarm stand-by and alert mode

After states of all sensors are retrieved then all motors are triggered.

## Running Command Center
There are few requirements that must be fulfilled for core application to run. Every motor and sensor may introduce
plugin specific dependencies. Application is know to run on:
* Python 3.4+
* Kubuntu 14.10
* Python packages:
  * _yapsy_

To run tests you will additionally need `pyhamcrest` package. Navigate in command line to `command_center` directory 
and type following command to run all tests:
```bash
python -m unittest discover tests/
```

# Architecture
Service consists of:
* Core application
* Sensors
* Motors

*Sensor* - is basically anything that collects some information from environment. For instance it might be code that
reads temperature from 1-wire thermometer or humidity from hygrometer. There are two types of sensors: _custom_ and
_built-in_. Custom sensors are pluggable via [Yapsy](http://yapsy.sourceforge.net) library, and should be put in
`plugins/sensors` directory. For details on developing your own sensor take a look at _Development_ section.

Below you will find list of all _built-in_ sensors:
* `errors` - value is list of all exceptions that occured during current run. It's cleared every iteration. It contains
  tuples (plugin key, exception object) for every exception thrown.
* `now` - contains datetime of current loop start. It's identical for every motor.
* `disabled_plugins` - list containing keys of all plugins that were disabled because of exceeded number of failures.
* `termination` - if value is not `None`, means that system is about to shut down, and contains tuple in following format: ```python
  (terminator's key, terminator's type name, reason string)
  ```
* `runtime` - is a dictionary containing technical information about application. Following keys are available:
  * `start_time` - datetime object when application was launched
  * `loop_counter` - counter incremented with every loop of service
  * `last_loop_time` - duration of last iteration in seconds
  * `average_loop_time` - average loop duration in seconds
  * `errors` - dictionary where keys are plugin keys, and values are lists of thrown exceptions by those plugins

*Motor* - motor is a plugin, that basing on current sensors' state and own internal state, should perform some actions.
For example when temperature remains too high, air conditioner might be turned on to drop it. Motors are loaded 
similarly to sensors - using [Yapsy](http://yapsy.sourceforge.net), and reside in `plugins/motors` directory. There are
no built-in motors.

## Application flow
Sensors, then motors, if too much fails then disabled and logged information about all exceptions. Conditions when
disabled.

# Development
To develop custom sensor or motor you need to create a yapsy-plugin, which basically means:
1. Copy template from `plugin/templates` directory, and give it appropriate name, .yapsy-plugin file must have name
 matching directory name.
1. Update information in yapsy-plugin file
  1. Mandatory fields are in `Core` section:
    1. _Name_ - plugin's name, that will be logged if something noticeable happens to plugin
    1. _Module_ - name of plugin's directory, or file name if it's single file plugin
    1. _Key_ - short-hand, unique name, that will be used to clearly identify plugin. Only one plugin with given name will
       be loaded. This is also string that motors will use to access some concrete sensor from system state. It also means,
       that you cannot enter key that matches _built-in_ sensor name. Keys are case-insensitive.
    1. _Last chance_ - this parameter is valid only for motor plugins. If you set this parameter to `True` then this motor
       will be executed during last loop when application is terminated. This might be useful for motors to shutdown
       corresponding devices or send notification that system is shutting down.

Generally `command_center` will "swallow" all exceptions thrown in sensor/motor, log them and eventually disable plugin 
if it does not work properly. There are two exceptions:
1. If you press Ctrl+C, then `command_center` will shutdown gracefully, so you may kill it from console.
1. If, for some reason, you really want application to stop (probably due to some hardware failure, or alert) you can
   throw `TerminateApplication` error which will force application to stop.

Shutdown procedure is as follows:
1. Finish current loop
1. Set `termination` _built-in_ sensor
1. Execute final loop only for motors declared as _last chance_. When you declare such a motor please notice, that in
   last loop state will only contain _built-in_ sensors, so be prepared for missing custom sensors state. 
1. Terminate application

## Sensors
Available sensors out of the box.
### Writing custom sensor
## Motors
Available motors out of the box.
### Writing custom motor
