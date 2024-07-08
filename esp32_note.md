

## ESP32 UART 핀 변경 
 
UART1 과 UART2 의 경우, GPIO중에서 아무 핀이나 할당하여 사용 가능합니다. 즉, Pin이 고정된게 아니고, IO Mux+GPIO Matrix를 통하여 유져가 원하는 Pin으로 할당 가능합니다. 

https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/peripherals/uart.html#_CPPv412uart_set_pin11uart_port_tiiii

// Set UART pins(TX: IO4, RX: IO5, RTS: IO18, CTS: IO19)
ESP_ERROR_CHECK(uart_set_pin(UART_NUM_2, 4, 5, 18, 19));


## ESP32-C3 UART 
- UART 2개지원
- UART0 : 기본 핀 U0TXD(GPIO21), U0RXD(GPIO 20)
- UART1 : 핀할당 필요 RX(GPIO6), TX(GPIO7)
