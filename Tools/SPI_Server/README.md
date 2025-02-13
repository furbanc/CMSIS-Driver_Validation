# CMSIS-Driver Validation

## SPI Server test project

This application executes commands sent from the SPI Client over the SPI interface
and is used to physically test compliance of the SPI driver with the CMSIS SPI driver
specification from version 2.0.0 upwards.

Operation:  
The SPI Server (in Slave mode) waits to receive a command over the SPI interface.
After command is received it is executed, and SPI Server again waits to receive next command.  

Command execution finishes by finishing the requested operation or by timeout.
All commands except XFER use fixed communication configuration, and timeout as specified
in the SPI_Server_Config.h configuration file.

Only XFER command is executed using settings configured by SET COM command, 
after execution of the XFER command the SPI interface is configured back to 
fixed configuration, so next command can be received.

The XFER command is executed until it finishes or until timeout specified in 
the command expires. If timeout is not specified in the command the last specified timeout
from previous XFER command is used. If timeout was never specified in the XFER command,
then default timeout setting is used.

Fixed SPI interface configuration:
 - Mode:                 Slave mode with Slave Select Hardware monitored
 - Clock / Frame Format: Clock Polarity 0, Clock Phase 0
 - Data Bits:            8
 - Bit Order:            MSB to LSB

SPI Server commands (32 bytes long (zero padding), [] means parameter is optional):
 - GET VER                                      <-  followed by 16 bytes OUT data phase
 - GET CAP                                      <-  followed by 32 bytes OUT data phase
 - SET BUF RX/TX,len[,pattern]                  ->  followed by optional 'len' bytes IN data phase
 - GET BUF RX/TX,len                            <-  followed by 'len' bytes OUT data phase
 - SET COM mode,format,bit_num,bit_order,ss_mode,bus_speed
 - XFER    num[,delay_c][,delay_t][,timeout]    <-> followed by 'num' items data IN/OUT transfer
 - GET CNT                                      <-  followed by 16 bytes OUT data phase

SPI Server command parameters:
 - RX/TX:      RX = SPI Server's receive buffer, TX = SPI Server's transmit buffer
 - len:        length of content in the following IN/OUT data phase
 - pattern:    value (in hex notation) to pre-fill the buffer with
 - mode:       0 = Master, 1 = Slave
 - format:     0 = Clock Polarity 0, Clock Phase 0
               1 = Clock Polarity 0, Clock Phase 1
               2 = Clock Polarity 1, Clock Phase 0
               3 = Clock Polarity 1, Clock Phase 1
               4 = Texas Instruments Frame Format
               5 = National Semiconductor Microwire Frame Format
 - bit_num:    number of bits (1 .. 32)
 - bit_order:  0 = MSB to LSB, 1 = LSB to MSB
 - ss_mode:    0 = unused, 1 = Master driven / Slave monitored
 - bus_speed:  bus speed (in bps)
 - num:        number of items (according CMSIS SPI driver specification)
 - delay_c:    delay before Control function is called, in milliseconds
 - delay_t:    delay after Control function is called but before Transfer function is called, in milliseconds
 - timeout:    total transfer timeout including delay_c and delay_t delays, in milliseconds

SPI Server responses to commands:
 - GET VER:  16 bytes containing string representation in form:
             "major.minor.patch"
 - GET CAP:  32 bytes containing values representing masks (bit value 1 means supported) or values as follows:
             "mode_mask,format_mask,data_bit_mask,bit_order_mask,min_bus_speed_in_kbps,max_bus_speed_in_kbps"
   - mode_mask (2 digits in hex): specifies mask of supported modes
                 - bit 0.:  Master
                 - bit 1.:  Slave
   - format_mask (2 digits in hex): specifies mask of supported clock/frame formats
                 - bit 0.:  Clock Polarity 0, Clock Phase 0
                 - bit 1.:  Clock Polarity 0, Clock Phase 1
                 - bit 2.:  Clock Polarity 1, Clock Phase 0
                 - bit 3.:  Clock Polarity 1, Clock Phase 1
                 - bit 4.:  Texas Instruments Frame Format
                 - bit 5.:  National Semiconductor Microwire Frame Format
   - data_bit_mask (8 digits in hex): specifies mask of supported data bits
                 - bit 0.:  Data Bits 1
                    ...
                 - bit 31.: Data Bits 32
   - bit_order_mask (1 byte in hex): specifies mask of supported bit orders
                 - bit 0.:  MSB to LSB
                 - bit 1.:  LSB to MSB
   - min_bus_speed_in_kbps (dec): minimum supported bus speed (in kbps)
   - max_bus_speed_in_kbps (dec): maximum supported bus speed (in kbps)
 - GET BUF:  'len' bytes from respective buffer, in binary format
 - GET CNT:  16 bytes containing value in decimal notation

Command examples:  
 - Get version: -> GET VER <- 16 bytes (for example "1.1.0")  
 - Get capabilities: -> GET CAP <- 32 bytes (for example "03,1F,00008080,03,1000,10000")  
 - Set Tx buffer to 'S': -> SET BUF TX,0,53  
 - Set Rx buffer to '?': -> SET BUF RX,0,3F  
 - Get 16 bytes of Rx buffer content: -> GET BUF RX,16 <- 16 bytes (for example "????????????????")  
 - Set communication mode to slave, clock phase 0, clock polarity 0, 8 data bits, MSB first, slave select
   hardware monitored, 2MBps: -> SET COM 1,0,8,0,1,2000000  
 - Transfer 16 bytes in both directions: -> XFER 16,10,0,100 <-> 16 bytes  
 - Get count: -> GET CNT <- 16 bytes (for example "16")  
