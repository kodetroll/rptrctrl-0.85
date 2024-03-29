rptrctrl.c - A program to do a 'minimalist' repeater controller
using a Raspberry PI and simple interfacing circuitry on the
GPIO Port.

This code was converted from an Arduino sketch I wrote sometime
ago and is only intended as a test bed for testing the implementation
method. Standard Arduino sketch functions (digitalWrite, digitalRead, 
pinMode, etc.) were replaced with functions abstracting calls to the
Broadcom BCM2835 GPIO Library. This functionality is pretty close to 
a 1:1 map. (Yes, I realize there is a wiringPI lib).

This implementation does not afford a separate input for a
tone decoder output, hence the term, 'minimalist'. To provide
tone access control, the receiver must have tone decoding
built in and the COR output must AND with this.

Currently ID audio is intended to be generated by pulsing a
GPIO pin which should control an off board tone generator.
Onboard tone generation and WAV based VOICE ID in the next version!

The COR input and PTT output pins on the Rasberry PI GPIO port
are specified by defines. These should be changed to match your
hardware configuration/implementation.

The PTT output pin is GPIO17, the COR input is on GPIO18, the
CORLED (illuminates an LED when COR is active) is on GPIO22 and
the ID_PIN (gates or controls an offboard tone generator for
the ID audio) is on GPIO21.

For testing, the COR input is simulated by a switch to ground 
on the COR input pin. Pull up on this pin is enabled in the
pinMode() function. The PTT is simulated by an LED to ground,
which illuminates when PTT is active. The CORLED is also an LED
to ground to accomplish that function.

Default values for the ID timer (600 Seconds - 10 minutes) and
the squelch tail timer (1 second) are specified by defines.
The runtime values of these parameters are stored in variables
and could be changed programatically, if desired (e.g. via the
serial port). Of course, you'd have to write that code.

The ID Time out timer is implemented using the C time library and
is based on the elapsed time counter (unsigned long int seconds)
so timeout timer precision is restricted to integer values of
one second. The squelch tail timer is implemented the same way,
so it has the same restrictions.

 (C) 2013 KB4OID Labs - A division of Kodetroll Heavy Industries

 All rights reserved, but otherwise free to use for personal use.
 No warranty expressed or implied.
 This code is for educational or personal use only.

Build using: 'gcc -o rptrctrl rptrctrl.c  -l bcm2835 -lrt'
  - or -
  use the handy 'build' script
 
 NOTE: This application must be run as root to have permissions to
 modify the GPIO pins.

'sudo ./rptctrl'

There are three debug defines, DEBUG, DEBUG_BEEP, and DEBUG_TONE.
These should be set to zero. These are left over from my conversion 
from the Arduino sketch.

The default starting timer values are defined in DEFAULT_ID_TIMER 
(600 Seconds) and DEFAULT_SQ_TIMER (1 second)

Other misc timer values (specified in mS) ID_PTT_DELAY (200 mS),
ID_PTT_HANG (500 mS), CW_MIN_DELAY (30 mS) and COR_DEBOUNCE_DELAY
(50 mS).

COR and PTT logic can be specified as POSITIVE or NEGATIVE; there
are defines (COR_POSITIVE, etc) to explicitly set this. The 
defaults are COR_NEGATIVE (H to L transition, L active), and 
PTT_POSITIVE (L to H transition, H active).

This program utilizes a state machine to control the various stages
of the repeater action. These are enumerated as CtrlStates and are
(briefly):
  CS_START - Starting state
  CS_IDLE - IDLE state, waiting for COR activity
  CS_DEBOUNCE_COR_ON - COR active sensed, waiting debounce time.
  CS_PTT_ON - Setting PTT to active state
  CS_PTT - PTT in active hold state 
  CS_DEBOUNCE_COR_OFF - COR dropped, going to Squelch Tail
  CS_SQT_ON = Squelch tail activated
  CS_SQT_BEEP - Courtesy Beep generating
  CS_SQT - Squelch tail hold
  CS_SQT_OFF - Setting Squelch tail to inactive
  CS_PTT_OFF - Setting PTT to inactive
  CS_ID - Play ID

To program the ID callsign, you must map the dah/dit/spaces of your call
to 'elements' in an int array. A DAH is element '3', a dit is element
'1' and a space is element '0'. These values are chosen to indicate the
relative length of the element, with the length of the dit and space
being set the same. If an an element value is greater than zero, then
the specified GPIO ID pin is active (H) and if the element value is zero, 
then the specified GPIO ID pin is (L). This can be used to key an off
board tone generator gated into the repeater audio path. 

For example, the call 'N0S' would be DAH DIT, space, DAH DAH DAH DAH DAH, 
space, DAH DAH DAH. Represented as elements this would be:
3,1,0,3,3,3,3,3,0,3,3,3,0

So the Elements array would look like this:
int Elements[] = {
  3,1,0,3,3,3,3,3,0,3,3,3,0
};

To set the CW ID Speed, find and change the value of CW_TIMEBASE. This 
defaults to a value of '50' mS which is about 20 WPM, give or take.
The duration of the courtesy tone beep is set by BeepDuration, which 
defaults to '2' in CWID clock increments

