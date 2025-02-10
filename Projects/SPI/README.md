## SPI Driver Validation project

This is a validation project to test the API and functionality of the SPI driver.

Configure the tests in *DV_SPI_Config.h*:
 - Driver number
 - Specific peripheral parameters
 - Enable/disable test cases

For the loopback test, connect the pins:
 - SPIx_MISO to SPIx_MOSI

See the documentation of the evaluation board to locate the port pins.

For driver tests with the server, please ensure that the connections required by
the corresponding server are established:
 - SPIx_MISO
 - SPIx_MOSI
 - SPIx_SCLK
 - SPIx_SS

Note: For use with the server, please connect GND between test board and server together.

