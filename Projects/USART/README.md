## USART Driver Validation project

This is a validation project for testing API and functionality of the USART Driver.

Configure the tests in *DV_USART_Config.h*: 
 - Driver number
 - Specific peripheral parameters
 - Enable/disable test cases
 
For the loopback test, connect the pins:
 - USARTx_TX to USARTx_RX

See the documentation of the evaluation board to locate the port pins.

For driver tests with the server, please ensure that the connections required by
the corresponding server are established:
 - USARTx_TX
 - USARTx_RX

Note: For use with the server, please connect GND between test board and server together.

