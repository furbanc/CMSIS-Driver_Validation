# CMSIS-Driver Validation

## USART Server test project

This application executes commands sent from the USART Client over the 
USART interface and is used to physically test compliance of the USART driver 
with the CMSIS USART driver specification from version 2.0.0 upwards.

Operation:  
The USART Server waits to receive a command over the USART interface.
After command is received it is executed, and USART Server again waits to receive 
next command.

Command execution finishes by finishing the requested operation or by timeout.
All commands except XFER use (mostly fixed) communication configuration and timeout as 
specified in the USART_Server_Config.h configuration file.

Only XFER command is executed using settings configured by SET COM command, 
after execution of the XFER command the USART interface is configured back to 
default settings from configuration file, so next command can be received.

The XFER command is executed until it finishes or until timeout specified in 
the command expires.
If timeout is not specified in the command the last specified timeout from 
previous XFER command is used.
If timeout was never specified in the XFER command then default timeout setting 
is used.

Default USART interface configuration:
Fixed settings:
 - Baudrate:      115200
 - Data Bits:     8
 - Parity:        None
 - Stop Bits:     1
 - Flow Control:  None
Settings configurable in the USART_Server_Config.h file:
 - Mode:          Asynchronous, Single-wire or IrDA

Note: In Single-wire mode Tx pin is used for both transmit and receive working 
      in half-duplex mode

USART Server commands (32 bytes long (zero padding), [] means parameter is optional):
 - GET VER                                      <-  followed by 16 bytes Tx data
 - GET CAP                                      <-  followed by 32 bytes Tx data
 - SET BUF RX/TX,len[,pattern]                  ->  followed by optional 'len' bytes Rx data
 - GET BUF RX/TX,len                            <-  followed by 'len' bytes Tx data
 - SET COM mode,data_bits,parity,stop_bits,flow_ctrl,cpol,cpha,baudrate
 - XFER    dir,num[,delay][,timeout][,num_rts]  <-> followed by 'num' items data Tx, Rx or Tx and Rx transfer
 - GET CNT                                      <-  followed by 16 bytes Tx data
 - SET BRK delay,duration
 - GET BRK                                      <-  followed by 1 byte Rx data phase
 - SET MDM mdm_ctrl,delay,duration
 - GET MDM                                      <-  followed by 1 byte Tx data

USART Server command parameters:
 - RX/TX:      RX = USART Server's receive buffer, TX = USART Server's transmit buffer
 - len:        length of content in following Tx or Rx data phase
 - pattern:    value (in hex notation) to pre-fill the buffer with
 - mode:       1 = Asynchronous
               2 = Synchronous Master
               3 = Synchronous Slave
               4 = Single Wire
               5 = IrDA
               6 = Smart Card
 - data_bits:  data bits (5 .. 9)
 - parity:     0 = None
               1 = Even
               2 = Odd
 - stop_bits:  0 = 1 Stop Bit
               1 = 2 Stop Bits
               2 = 1.5 Stop Bits
               3 = 0.5 Stop Bits
 - flow_ctrl:  0 = None
               1 = CTS
               2 = RTS
               3 = RTS/CTS
 - cpol:       This setting is only relevant for Synchronous mode
               0 = Data are captured on rising edge
               1 = Data are captured on falling edge
 - cpha:       This setting is only relevant for Synchronous mode
               0 = Sample on first (leading) edge
               1 = Sample on second (trailing) edge
 - baudrate:   baudrate in bauds
 - dir:        direction of transfer
               0 = Send (Tx)
               1 = Receive (Rx)
               2 = Transfer (simultaneous Tx and Rx (in synchronous mode only))
 - num:        number of items (according CMSIS USART driver specification)
 - delay:      initial delay, in milliseconds, before starting requested operation or line control 
 - timeout:    timeout in milliseconds, after delay, if delay is specified
 - num_rts:    number of items after which RTS line should be de-activated
              (used to test client's CTS line functionality)
 - mdm_ctrl:   modem lines requested state (in hex notation):
               - bit 0.: RTS pin state
               - bit 1.: DTS pin state
               - bit 2.: state of GPIO line connected to USART Client's DCD pin
               - bit 3.: state of GPIO line connected to USART Client's RI pin
 - duration:   duration, in milliseconds, of controlling modem lines

USART Server responses to commands:
 - GET VER:  16 bytes containing string representation in form:
             "major.minor.patch"
 - GET CAP:  32 bytes containing values representing masks (bit value 1 means supported) or values as follows:
             "mode_mask,data_bits_mask,parity_mask,stop_bits_mask,flow_control_mask,min_baudrate,max_baudrate"
             - mode_mask (2 digits in hex): specifies mask of supported modes
                 - bit 0.:  Asynchronous
                 - bit 1.:  Synchronous Master
                 - bit 2.:  Synchronous Slave
                 - bit 3.:  Single Wire
                 - bit 4.:  IrDA
                 - bit 5.:  Smart Card
             - data_bits_mask (2 digits in hex): specifies mask of supported data bits
                 - bit 0.:  Data Bits 5
                 - bit 1.:  Data Bits 6
                 - bit 2.:  Data Bits 7
                 - bit 3.:  Data Bits 8
                 - bit 4.:  Data Bits 9
             - parity_mask: (1 digits in hex): specifies mask of supported parity options
                 - bit 0.:  No Parity
                 - bit 1.:  Even Parity
                 - bit 2.:  Odd Parity
             - stop_bits_mask (1 digits in hex): specifies mask of supported stop bits
                 - bit 0.:  1 Stop Bit
                 - bit 1.:  2 Stop Bits
                 - bit 2.:  1.5 Stop Bits
                 - bit 3.:  0.5 Stop Bits
             - flow_control_mask (1 digits in hex): specifies mask of supported flow control options
                 - bit 0.:  No Flow Control
                 - bit 1.:  CTS
                 - bit 2.:  RTS
                 - bit 3.:  RTS/CTS
             - modem_line_mask: (1 digits in hex): specifies mask of supported modem lines (input/output)
                 - bit 0.:  RTS line available
                 - bit 1.:  CTS line available
                 - bit 2.:  DTR line available
                 - bit 3.:  DSR line available
                 - bit 4.:  GPIO for DCD line available
                 - bit 5.:  GPIO for RI line available
             - min_baudrate (dec): minimum supported baudrate (in bauds)
             - max_baudrate (dec): maximum supported baudrate (in bauds)
 - GET BUF:  'len' bytes from respective buffer, in binary format
 - GET CNT:  16 bytes containing value in decimal notation
 - GET BRK:  1 byte (in hex) containing break signal status
 - GET MDM:  1 byte (in hex) containing values representing modem lines state:
                 - bit 0.:  CTS line current state
                 - bit 1.:  DSR line current state

Command examples:
 - Get version: -> GET VER <- 16 bytes (for example "1.0.0")
 - Get capabilities: -> GET CAP <- 32 bytes (for example "3B,18,7,F,F,03,9600,5000000")
 - Set Tx buffer to 'S': -> SET BUF TX,0,53
 - Set Rx buffer to '?': -> SET BUF RX,0,3F
 - Get 16 bytes of Rx buffer content: -> GET BUF RX,16 <- 16 bytes (for example "????????????????")
 - Set communication mode to asynchronous, 8 data bits, no parity, 1 stop bit, no flow control, 115200 bauds:
   -> SET COM 0,8,0,0,0,0,0,115200
 - Receive 16 bytes: -> XFER 0,16,0,100 <- 16 bytes
 - Send 16 bytes: -> XFER 1,16,0,100 -> 16 bytes
 - Transfer 16 bytes in both directions (in synchronous mode only): -> XFER 2,16,0,100 <-> 16 bytes
 - Get count: -> GET CNT <- 16 bytes (for example "16")
 - Get break status: -> GET BRK <- 1 byte (for example "1")
 - Set activate RTS modem line after 10 ms for 50 ms and then deactivate it: -> SET MDM 01,10,50
 - Get modem line state: -> GET MDM <- 1 byte (for example "3")
