//Osvaldo Tirado-Cortes
//I2C transmits light information from onboard sensor

#include <msp430fr6989.h>
#include <stdlib.h>
#include <stdint.h>

void Initialize_UART(void);
void uart_write_char(unsigned char);
unsigned char uart_read_char(void);
void uart_write_uint16(unsigned int);
void Initialize_I2C(void);
int i2c_read_word(unsigned char, unsigned char, unsigned int *);
int i2c_write_word(unsigned char, unsigned char, unsigned int);
void uart_write_string(char * str);

#define FLAGS       UCA1IFG       // Contains the transmit & receive flags
#define RXFLAG      UCRXIFG      // Receive flag
#define TXFLAG      UCTXIFG      // Transmit flag
#define TXBUFFER    UCA1TXBUF  // Transmit buffer
#define RXBUFFER    UCA1RXBUF  // Receive buffer
#define redLED      BIT0

int main(void)
{
    WDTCTL = WDTPW | WDTHOLD;   // stop watchdog timer
    PM5CTL0 &= ~LOCKLPM5;       //enable GPIO pins

    P1DIR |= redLED;
    P1OUT |= redLED;

    Initialize_UART();
    Initialize_I2C();
    unsigned int data, i2cAddr=0x44;
    unsigned int i;

    while(1) {
        i2c_write_word(i2cAddr, 0x01, 0x7604);
        i2c_read_word(i2cAddr, 0x00, &data);
        uart_write_uint16(data*1.28);
        uart_write_char('\n');
        uart_write_char('\r');
        for(i=0;i<60000;i++) {}
    }
    return 0;
}


void Initialize_UART(void) {
    P3SEL1 &= ~(BIT4|BIT5);             // Divert pins to UART functionality
    P3SEL0 |= (BIT4|BIT5);
    UCA1CTLW0 |= UCSSEL_2;              // Use SMCLK clock; leave other settings default
    UCA1BRW = 6;
    UCA1MCTLW = UCBRS5|UCBRS1|UCBRF3|UCBRF2|UCBRF0|UCOS16;
    UCA1CTLW0 &= ~UCSWRST;              // Exit the reset state (so transmission/reception can begin)
}

void uart_write_char(unsigned char ch) {
    while ( (FLAGS & TXFLAG)==0 ) {}        // Wait for any ongoing transmission to complete
    TXBUFFER = ch;                          // Write the byte to the transmit buffer
}

unsigned char uart_read_char(void) {
    unsigned char temp;
    if( (FLAGS & RXFLAG) == 0)
        return NULL;
    temp = RXBUFFER;
    return temp;
}

void uart_write_uint16(unsigned int n) {
    int temp,i;
    if(n>= 10000){
        temp = (n/10000) % 10;
        uart_write_char(temp + '0');
    }

    if(n>= 1000){
            temp = (n/1000) % 10;
            uart_write_char(temp + '0');
        }

    if(n>= 100){
            temp = (n/100) % 10;
            uart_write_char(temp + '0');
        }

    if(n>= 10){
            temp = (n/10) % 10;
            uart_write_char(temp + '0');
        }
    temp = n%10;
    uart_write_char(temp +'0');
    for(i=0; i < 1000 ;i++);
    return;
}

////////////////////////////////////////////////////////////////////
// Read a word (2 bytes) from I2C (address, register)
int i2c_read_word(unsigned char i2c_address, unsigned char i2c_reg,
unsigned int * data) {
unsigned char byte1, byte2;
// Initialize the bytes to make sure data is received every time
byte1 = 111;
byte2 = 111;
//********** Write Frame #1 ***************************
UCB1I2CSA = i2c_address; // Set I2C address
UCB1IFG &= ~UCTXIFG0;
UCB1CTLW0 |= UCTR; // Master writes (R/W bit = Write)
UCB1CTLW0 |= UCTXSTT; // Initiate the Start Signal
while ((UCB1IFG & UCTXIFG0) ==0) {}
UCB1TXBUF = i2c_reg; // Byte = register address
while((UCB1CTLW0 & UCTXSTT)!=0) {}
if(( UCB1IFG & UCNACKIFG )!=0) return -1;
UCB1CTLW0 &= ~UCTR; // Master reads (R/W bit = Read)
UCB1CTLW0 |= UCTXSTT; // Initiate a repeated Start Signal
//****************************************************
//********** Read Frame #1 ***************************
while ( (UCB1IFG & UCRXIFG0) == 0) {}
byte1 = UCB1RXBUF;
//****************************************************
//********** Read Frame #2 ***************************
while((UCB1CTLW0 & UCTXSTT)!=0) {}
UCB1CTLW0 |= UCTXSTP; // Setup the Stop Signal
while ( (UCB1IFG & UCRXIFG0) == 0) {}
byte2 = UCB1RXBUF;
while ( (UCB1CTLW0 & UCTXSTP) != 0) {}
//****************************************************
// Merge the two received bytes
*data = ( (byte1 << 8) | (byte2 & 0xFF) );
return 0;
}

////////////////////////////////////////////////////////////////////
// Write a word (2 bytes) to I2C (address, register)
int i2c_write_word(unsigned char i2c_address, unsigned char i2c_reg,
unsigned int data) {
unsigned char byte1, byte2;
byte1 = (data >> 8) & 0xFF; // MSByte
byte2 = data & 0xFF; // LSByte
UCB1I2CSA = i2c_address; // Set I2C address
UCB1CTLW0 |= UCTR; // Master writes (R/W bit = Write)
UCB1CTLW0 |= UCTXSTT; // Initiate the Start Signal
while ((UCB1IFG & UCTXIFG0) ==0) {}
UCB1TXBUF = i2c_reg; // Byte = register address
while((UCB1CTLW0 & UCTXSTT)!=0) {}
//********** Write Byte #1 ***************************
UCB1TXBUF = byte1;
while ( (UCB1IFG & UCTXIFG0) == 0) {}
//********** Write Byte #2 ***************************
UCB1TXBUF = byte2;
while ( (UCB1IFG & UCTXIFG0) == 0) {}
UCB1CTLW0 |= UCTXSTP;
while ( (UCB1CTLW0 & UCTXSTP) != 0) {}
return 0;
}

// Configure eUSCI in I2C master mode
void Initialize_I2C(void) {
// Enter reset state before the configuration starts...
UCB1CTLW0 |= UCSWRST;
// Divert pins to I2C functionality
P4SEL1 |= (BIT1|BIT0);
P4SEL0 &= ~(BIT1|BIT0);
// Keep all the default values except the fields below...
// (UCMode 3:I2C) (Master Mode) (UCSSEL 1:ACLK, 2,3:SMCLK)
UCB1CTLW0 |= UCMODE_3 | UCMST | UCSSEL_3;
// Clock divider = 8 (SMCLK @ 1.048 MHz / 8 = 131 KHz)
UCB1BRW = 8;
// Exit the reset mode
UCB1CTLW0 &= ~UCSWRST;
}

void uart_write_string(char * str) {
        int i=0;
        while (str[i] != '\0'){
            uart_write_char(str[i]);
            i++;
        }
}
