
## https://wiki.icbbuy.com/doku.php?id=developmentboard:nrf52840

## https://github.com/joric/nrfmicro/wiki/Alternatives


### 
* The SuperMini NRF52840 is Nice!Nano clone which in turn is based on the Pro Micro layout. This means that it can be used with almost any Pro Mcro keyboard. Unlike the Pro Micro however it sports wireless functionality in the form of Bluetooth and pins to use/charge a lithium-ion battery.

* The nRF52840 MCU uses an ARM Cortex-M4F CPU clocked at 64MHz and offers 1MB of flash memory and 256KB of RAM. Although CircuitPython will use up most of the flash memory.

* Bluetooth 5.0
* Microcontroller nRF52840
* ARM Cortex-M4F processor
* Clocked at 64MHz
* Flash Memory 1MB
* RAM 256KB
* Quiescent Current (power consumption at sleep mode 1mA)
* Battery charging chip: supports lithium battery charging and discharging
* dual-current Li-Po charging with a jumper that you solder with max of 300mA
software-controllable MOSFET on the VCC pin to cut standby current consumption of LEDs and other peripherals (in theory, i still need to test this)
* nice!nano bootloader pre-flashed
* low dropout current regulator (LDO): MICRONE(Nanjing Micro One Elec) ME6217C33M5G, with max output current 800 mA. Low current consumption during operation: 100uA and while turned off 1uA (spreadsheet data)
* have seen users with Zephyr firmware and LDO enabled reaching 150uA power consumption, but it will turn off current to the external peripherals like Oled Screens, RGB Leds etc. If LDO is disabled it goes to about 800uA and sends current to the external peripherals. I still have to test this also.



### 특별 참고 사항:
기본 공장 내장 프로그램은 Blink-All-IO입니다(모든 IO 포트가 1초 안에 자동으로 뒤집힙니다). Nice! Nano V2 부트로더를 입력해야 하는 경우 0.5초 이내에 RST를 GND에 두 번 단락시키세요. 키보드 보드에서 RST 버튼을 0.5초 이내에 두 번 누르기만 하면 됩니다.
소개
SuperMini NRF52840은 Nice! Nano와 호환되는 Pro Micro 대체 개발 보드입니다. 핀은 ProMicro와 동일하므로 거의 모든 ProMicro 키보드와 함께 사용할 수 있습니다. NRF5280 개발 보드에는 3.7V 리튬 배터리 인터페이스와 LED 전원을 차단할 수 있는 소프트웨어 스위치가 있습니다. 끄면 대기 전력 소비가 1mA에 도달할 수 있습니다.
nRF52840은 노르웨이 Nordic Semiconductor에서 출시한 고성능 저전력 무선 SoC 칩입니다. Bluetooth 5, Thread, Zigbee, ANT 및 2.4GHz를 포함한 여러 무선 프로토콜을 지원합니다. nRF52840 칩은 64MHz로 클록된 ARM Cortex-M4F 프로세서를 사용하며, 내장 플래시 메모리 1MB와 RAM 256KB가 있습니다. 또한 ADC, PWM, SPI, I2C, UART, USB 및 GPIO 등 다양한 주변 장치가 있습니다. 또한 nRF52840은 AES 암호화, SHA-256 해싱 및 TRNG(True Random Number Generator)와 같은 다양한 보안 기능도 지원합니다.

### 하드웨어 설명
s523f241efa674831ba4796424c4a8f3a6.jpg
제품 매개변수 강력한 무선 기능: Bluetooth 5.0, 온보드 안테나 강력한 CPU: nRF52840 칩은 ARM Cortex-M4F 프로세서를 사용, 64MHz로 클록, 내장 1MB 플래시 메모리 및 256KB RAM 배터리 충전 칩: 리튬 배터리 충전 및 방전 지원 전력 소비: 대기 전력 소비는 1mA에 도달할 수 있음

### 핀 다이어그램
<div align="center">
  <img src="nrf52840 suprrmini pinout.jpg" width="1000" alt="nrf52840 suprrmini pinout.jpg"/>
</div>

boards/nrf52840 supermini/nrf52840 suprrmini pinout.jpg


### 전원 공급 장치
SuperMini NRF52840은 3.7V 리튬 배터리의 충전 및 방전을 지원합니다. 충전 중 충전 속도는 100mA이므로 2000mAh 배터리는 완전히 충전하는 데 20시간이 걸립니다.
충전 전류 경고BOOST가 연결되면 충전 전류가 100mA에서 300mA로 증가하고 커패시터 용량이 500mAh보다 클 때만 연결할 수 있습니다(폭발을 방지하기 위해💥).

### 외부 배터리
외부 배터리가 필요한 경우 배터리의 +단을 B+에 연결하고 음극단을 B-에 연결하기만 하면 됩니다. 외부 리튬 배터리는 3.7V만 지원하고 충전 속도는 100mA입니다(2000mAh 배터리는 완전히 충전하는 데 20시간이 걸립니다).주의사항용접할 때는 양극과 음극을 단락시키지 않도록 주의하세요. 단락되면 배터리와 장비가 타버릴 수 있습니다.

### 외부 VCC 제어
힌트P0.13이 낮게 설정되면 3.3V, VCC 핀에 대한 전원 공급이 꺼집니다. 이것은 유휴 상태일 때 전력을 사용하는 구성 요소(예: RGB, LED)를 줄이는 데 유용합니다.

### 부트로더
Bootloder에 들어가려면 0.5초 이내에 RST를 GND에 두 번 쇼트하세요. Bootloder에 들어가 USB를 통해 컴퓨터에 연결하면 Nice! Nano라는 저장 장치가 표시됩니다. 이때 .uf2 파일을 끌어서 프로그램을 구울 수 있습니다.



