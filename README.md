# hello-world
// 3 phase inverter for ac motor or BLDC motor
/* Hi All, im new to programing and working with both Arduino Uno and Mega 2560. 
im trying to wtite a VFD to control etiher a BLDC or 3-phase induction motor. i have found the code below and it runs well, giving 3 very
clear sine waves at about 25Hz with delay(2) and 50Hz with delay (1).
I would like to ask what adaptations would be required to control the frequency for speed and apmlitue for torque?
the code alreay has amplitude terms but im not sure how to vary these. i was thinking of using an analogue POT on the Adruino, 
reading its value with and mapping that to the amplitude term with the float function. Not sure how to though... 
was thinking of doing the same for frequency control but also not sure how?

// For the MEGA!

#define HALF 400
#define PERIOD 800

void setup ()
{
 setup_cosines () ;
 TCCR4A = 0xFE ;  // phase correct (mode 1010)
 TCCR4B = 0x11 ;  // prescale by 1
 TIMSK4 = 0x01 ;  // overflow interrupt
 ICR4  = PERIOD+1 ;    // 100us cycle time, so effectively 50us.
 OCR4A = HALF ;
 OCR4B = HALF ;
 OCR4C = HALF ;
 pinMode (6, OUTPUT) ;  // the OCR4A pin, our U phase
 pinMode (7, OUTPUT) ;  // the OCR4B pin, our V phase
 pinMode (8, OUTPUT) ;  // the OCR4C pin, our W phase
}

// control time values, in units of 62.5ns
volatile int u = HALF ;
volatile int v = HALF ;
volatile int w = HALF ;


// every complete cycle we update the registers.
ISR (TIMER4_OVF_vect)
{
 OCR4A = u ;
 OCR4B = v ;
 OCR4C = w ;
}

// a cosine table, 1024 entries in range +/-127
char cosine [0x400] ;

void setup_cosines ()
{
 for (int i = 0 ; i < 0x400 ; i++)
 {
   float a = PI * i / 0x200 ;
   cosine [i] = round (127.0 * cos (a)) ;
 }
}

int phase = 0 ; // taken modulo 1024
int amplitude = 200 ;  // 201 is maximum value without overflow.

void loop ()
{
 phase ++ ;  // phase increment, normally this would be done by DDS loop
 int newu = (cosine [phase & 0x3FF] * amplitude + 0x1F) >> 6 ;
 int newv = (cosine [(phase + 0x155) & 0x3FF] * amplitude + 0x1F) >> 6 ;
 int neww = - newu - newv ;
 newu += HALF ;
 newv += HALF ;
 neww += HALF ;
 noInterrupts () ;  // interrupt-safe updating of u,v,w
 u = newu ;
 v = newv ;
 w = neww ;
 interrupts () ;
 delay (2) ;
}
