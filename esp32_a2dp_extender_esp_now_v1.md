## 최종 요구사항 정리 (주석 설명 포함) 및 전체 소스코드

**지금까지 구현된 요구사항:**

1.  **Arduino Core 환경:** ESP32 Arduino Core 환경에서 개발되었습니다.
2.  **ESP32 #1 (A2DP 수신기):** 블루투스를 통해 A2DP로 오디오 데이터를 수신합니다.
3.  **ESP-NOW 무선 전송:** Wi-Fi 대신 ESP-NOW 프로토콜을 사용하여 오디오 데이터를 무선으로 ESP32 #2로 전송합니다.
4.  **ESP32 #2 (A2DP 송신기):** ESP-NOW로 받은 오디오 데이터를 블루투스를 통해 A2DP로 스피커에 송신합니다.
5.  **블루투스 자동 페어링 (ESP32 #2):** ESP32 #2는 마지막으로 페어링된 블루투스 스피커에 자동으로 연결을 시도합니다.
6.  **양방향 버퍼링:** ESP32 #1과 ESP32 #2 모두 FreeRTOS 큐를 사용하여 오디오 데이터를 버퍼링합니다.
7.  **지능형 채널 자동 선택 (ESP32 #1):** ESP32 #1은 주변 Wi-Fi 채널을 스캔하여 AP 개수가 적고 신호 강도가 높은 최적의 채널을 선택하여 ESP-NOW 통신에 사용합니다.
8.  **채널 정보 공유:** ESP32 #1은 선택한 채널 정보를 ESP-NOW를 통해 ESP32 #2로 전송합니다.
9.  **채널 정보 확인 응답:** ESP32 #2는 채널 정보를 받으면 ESP32 #1에게 확인 응답을 보냅니다.
10. **블루투스 재연결 (ESP32 #2):** ESP32 #2는 블루투스 연결이 끊어지면 주기적으로 재연결을 시도합니다.
11. **메모리 관리:** `malloc()`으로 할당된 메모리는 사용 후 `free()`를 통해 해제하여 메모리 누수를 방지합니다.
12. **WS2812 상태 표시:** WS2812 LED를 사용하여 블루투스 연결 및 페어링 상태를 표시합니다.
13. **개선된 채널 선택 알고리즘:** 채널 선택 시 AP 개수와 함께 신호 강도(RSSI)를 고려합니다.
14. **블루투스 페어링 버튼 (ESP32 #2):** ESP32 #2에 페어링 버튼을 추가하여 수동으로 블루투스 재연결 및 페어링 준비 상태로 진입할 수 있습니다.
15. **디바운싱 라이브러리:** `Bounce2` 라이브러리를 적용하여 페어링 버튼의 디바운싱을 처리합니다.

---

**전체 소스코드:**

**ESP32 #1 (A2DP 수신기 & ESP-NOW 송신기):**

```arduino
#include <esp_now.h>
#include <WiFi.h>
#include <BluetoothA2DPSink.h>
#include <freertos/FreeRTOS.h>
#include <freertos/queue.h>
#include <esp_wifi.h>
#include <Adafruit_NeoPixel.h> // WS2812 라이브러리

// --- 설정 ---
#define WIFI_CHANNEL 1         // 초기 Wi-Fi 채널 (스캔용)
#define ESPNOW_CHANNEL 1       // 기본 ESP-NOW 채널
#define WIFI_BUFFER_SLOTS 10
#define LED_PIN 2          // WS2812 데이터 핀 번호
#define NUM_LEDS 1         // 사용할 LED 개수

// --- 구조체 정의 ---
typedef struct struct_channel_info {
  uint8_t channel;
} struct_channel_info;

typedef struct struct_ack {
  bool channel_confirmed;
} struct_ack;

// --- 전역 변수 ---
QueueHandle_t espNowSendQueue; // ESP-NOW 전송을 위한 큐
BluetoothA2DPSink a2dpSink;    // A2DP 수신 객체
uint8_t peerAddress[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF}; // ESP32 #2의 MAC 주소 (실제 주소로 변경 필요)
struct_channel_info channelInfo; // 채널 정보를 담는 구조체
bool channelAckReceived = false; // ESP32 #2로부터 채널 정보 확인 응답을 받았는지 여부
Adafruit_NeoPixel pixels(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800); // WS2812 LED 객체

// --- 함수 선언 ---
void a2dp_data_callback(uint8_t *data, uint32_t len);
void onEspNowSent(const uint8_t *mac_addr, esp_now_send_status_t status);
void onEspNowRecv(const uint8_t *mac_addr, const uint8_t *incomingData, int len);
void setupEspNow(uint8_t channel);
void scanAndSetChannel();
void updateA2DPSinkLED(bool receiving);

// --- A2DP 데이터 수신 콜백 함수 ---
void a2dp_data_callback(uint8_t *data, uint32_t len) {
  uint8_t *buffer = (uint8_t *)malloc(len); // 수신된 오디오 데이터를 저장할 버퍼 할당
  if (buffer) {
    memcpy(buffer, data, len); // 데이터 복사
    if (xQueueSend(espNowSendQueue, &buffer, 0) != pdTRUE) { // ESP-NOW 전송 큐에 데이터 추가
      Serial.println("ESP-NOW 전송 큐에 데이터 추가 실패 (버퍼 가득?)");
      free(buffer); // 큐에 추가 실패 시 버퍼 해제
    }
    updateA2DPSinkLED(true); // WS2812 LED 업데이트 (수신 중)
    updateA2DPSinkLED(false); // WS2812 LED 업데이트 (수신 완료)
  } else {
    Serial.println("A2DP 데이터 버퍼 메모리 할당 실패");
  }
}

// --- ESP-NOW 전송 완료 콜백 함수 ---
void onEspNowSent(const uint8_t *mac_addr, esp_now_send_status_t status) {
  if (status != ESP_NOW_SEND_SUCCESS) {
    Serial.println("ESP-NOW 패킷 전송 실패");
  }
}

// --- ESP-NOW 수신 콜백 함수 (확인 응답 처리) ---
void onEspNowRecv(const uint8_t *mac_addr, const uint8_t *incomingData, int len) {
  if (len == sizeof(struct_ack)) { // 수신된 데이터가 확인 응답 구조체 크기와 같은지 확인
    struct_ack ack;
    memcpy(&ack, incomingData, sizeof(ack)); // 수신된 데이터를 구조체에 복사
    if (ack.channel_confirmed) { // 채널 확인 응답인지 확인
      Serial.println("ESP32 #2로부터 채널 정보 확인 응답 받음.");
      channelAckReceived = true; // 채널 확인 응답 수신 플래그 설정
    }
  }
}

// --- ESP-NOW 초기화 함수 ---
void setupEspNow(uint8_t channel) {
  WiFi.mode(WIFI_STA);
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW 초기화 실패");
    delay(3000);
    ESP.restart();
    return;
  }
  esp_now_set_self_role(ESP_NOW_ROLE_CONTROLLER); // ESP32 #1은 컨트롤러 역할
  esp_now_register_send_cb(onEspNowSent);       // 전송 완료 콜백 등록
  esp_now_register_recv_cb(onEspNowRecv);       // 수신 콜백 등록
  esp_now_peer_info_t peerInfo;                 // 피어 정보 구조체
  memcpy(peerInfo.peer_addr, peerAddress, 6); // 피어 MAC 주소 복사
  peerInfo.channel = channel;                  // 사용할 ESP-NOW 채널 설정
  peerInfo.encrypt = false;                    // 암호화 비활성화
  if (esp_now_add_peer(&peerInfo) != ESP_OK) { // 피어 추가
    Serial.println("ESP-NOW 피어 추가 실패");
    return;
  }
  Serial.printf("ESP-NOW 시작 (채널: %d), 피어 MAC: %02x:%02x:%02x:%02x:%02x:%02x\n", channel, channel, peerAddress[0], peerAddress[1], peerAddress[2], peerAddress[3], peerAddress[4], peerAddress[5]);
}

// --- 최적 채널 스캔 및 설정 함수 ---
void scanAndSetChannel() {
  Serial.println("Wi-Fi 채널 스캔 시작...");
  WiFi.mode(WIFI_STA);
  WiFi.scanNetworks(true); // 비동기 스캔 시작

  int bestChannel = ESPNOW_CHANNEL;
  int minAPs = 255;
  int bestRSSI = -100; // 초기 RSSI 값 설정

  delay(5000); // 스캔 완료 대기

  int n = WiFi.scanComplete(); // 스캔 결과 개수 얻기
  if (n > 0) {
    Serial.printf("%d개의 Wi-Fi 네트워크 발견\n", n);
    std::map<int, int> channelCounts;   // 채널별 AP 개수를 저장할 맵
    std::map<int, int> channelMaxRSSI;  // 채널별 최대 RSSI를 저장할 맵

    for (int i = 0; i < n; ++i) {
      int channel = WiFi.channel(i);
      int rssi = WiFi.RSSI(i);
      channelCounts[channel]++;
      if (channelMaxRSSI.find(channel) == channelMaxRSSI.end() || rssi > channelMaxRSSI[channel]) {
        channelMaxRSSI[channel] = rssi;
      }
    }

    for (auto const& [channel, count] : channelCounts) {
      Serial.printf("채널 %d: %d개의 AP 발견, 최대 RSSI: %d dBm\n", channel, count, channelMaxRSSI[channel]);
      // AP 개수가 적거나, AP 개수가 비슷하면 RSSI가 높은 채널을 선택
      if (count < minAPs || (count == minAPs && channelMaxRSSI[channel] > bestRSSI)) {
        minAPs = count;
        bestChannel = channel;
        bestRSSI = channelMaxRSSI[channel];
      }
    }
    Serial.printf("최적 채널: %d (발견된 AP 수: %d, 최대 RSSI: %d dBm)\n", bestChannel, minAPs, bestRSSI);
    ESPNOW_CHANNEL = bestChannel;
  } else {
    Serial.println("Wi-Fi 스캔 결과 없음 또는 실패. 기본 채널 사용.");
  }
  WiFi.mode(WIFI_OFF);
}

// --- WS2812 LED 업데이트 함수 (A2DP 수신 상태 표시) ---
void updateA2DPSinkLED(bool receiving) {
  if (receiving) {
    pixels.setPixelColor(0, 0x00FF00); // 녹색: A2DP 수신 중
  } else {
    pixels.setPixelColor(0, 0x001000); // 희미한 녹색: 대기
  }
  pixels.show();
}

// --- setup 함수 ---
void setup() {
  Serial.begin(115200);
  Serial.println("ESP32 #1 - A2DP 수신기 & ESP-NOW 송신기");

  espNowSendQueue = xQueueCreate(WIFI_BUFFER_SLOTS, sizeof(uint8_t *)); // ESP-NOW 전송 큐 생성
  if (espNowSendQueue == NULL) {
    Serial.println("ESP-NOW 전송 큐 생성 실패");
  }

  pixels.begin();
  pixels.setBrightness(50);
  pixels.show(); // 초기 LED 끄기

  scanAndSetChannel(); // 최적 채널 스캔 및 설정
  setupEspNow(ESPNOW_CHANNEL); // ESP-NOW 초기화

  // 선택된 채널 정보를 ESP32 #2로 전송
  channelInfo.channel = ESPNOW_CHANNEL;
  esp_now_send(peerAddress, (uint8_t *)&channelInfo, sizeof(channelInfo));
  Serial.printf("선택된 채널 정보 (%d)를 ESP32 #2로 전송했습니다. 응답 대기 중...\n", ESPNOW_CHANNEL);

  // A2DP 싱크 초기화
  a2dpSink.set_on_data_received(a2dp_data_callback);
  a2dpSink.start("ESP32-A2DP-Sink");

  Serial.println("A2DP 싱크 시작. 연결 대기 중...");
}

// --- loop 함수 ---
void loop() {
  if (channelAckReceived) { // 채널 정보 확인 응답을 받았으면 오디오 데이터 전송 시작
    uint8_t *audioData;
    if (xQueueReceive(espNowSendQueue, &audioData, portMAX_DELAY)) { // 큐에서 오디오 데이터 수신 대기
      size_t dataLen = _msize(audioData);
      if (dataLen > 0) {
        esp_now_send(peerAddress, (uint8_t *)&dataLen, sizeof(size_t)); // 데이터 길이 전송
        esp_now_send(peerAddress, audioData, dataLen);                 // 오디오 데이터 전송
      }
      free(audioData); // 사용 완료된 버퍼 해제
    }
  } else {
    delay(100); // 응답 대기
  }
  delay(1);
}
```

**ESP32 #2 (ESP-NOW 수신기 & A2DP 송신기):**

```arduino
#include <esp_now.h>
#include <WiFi.h>
#include <BluetoothA2DPSource.h>
#include <freertos/FreeRTOS.h>
#include <freertos/queue.h>
#include <esp_wifi.h>
#include <Adafruit_NeoPixel.h> // WS2812 라이브러리
#include <Bounce2.h>        // Bounce2 라이브러리 추가

// --- 설정 ---
#define WIFI_CHANNEL 1
#define ESPNOW_CHANNEL 1 // 기본 채널
#define WIFI_BUFFER_SLOTS 20
#define LED_PIN 2          // WS2812 데이터 핀 번호
#define NUM_LEDS 1         // 사용할 LED 개수
#define PAIRING_BUTTON_PIN 0 // 블루투스 페어링 버튼 핀 번호
#define DEBOUNCE_INTERVAL 50 // 디바운스 간격 (ms)

// --- 구조체 정의 ---
typedef struct struct_channel_info {
  uint8_t channel;
} struct_channel_info;

typedef struct struct_ack {
  bool channel_confirmed;
} struct_ack;

// --- 전역 변수 ---
QueueHandle_t espNowReceiveQueue; // ESP-NOW 수신을 위한 큐
BluetoothA2DPSource a2dpSource;    // A2DP 송신 객체
bool a2dpConnected = false;        // 블루투스 연결 상태
uint8_t peerAddress[] = {0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF}; // ESP32 #1의 MAC 주소 (실제 주소로 변경 필요)
bool channelReceived = false;        // ESP32 #1로부터 채널 정보를 받았는지 여부
uint8_t currentEspNowChannel = ESPNOW_CHANNEL; // 현재 ESP-NOW 채널
Adafruit_NeoPixel pixels(NUM_LEDS, LED_PIN, NEO_GRB + NEO_KHZ800); // WS2812 LED 객체
bool pairingMode = false;        // 페어링 모드 상태
Bounce2::Button button = Bounce2::Button(); // Bounce2 객체 생성

// --- 클래스 정의 ---
class MyA2DPSourceCallbacks : public BluetoothA2DPSourceCallbacks {
public:
  void a2dp_connected() {
    Serial.println("블루투스 스피커에 A2DP 연결됨");
    a2dpConnected = true;
    pairingMode = false; // 연결 성공 시 페어링 모드 종료
    updateBluetoothLED(); // WS2812 LED 업데이트
  }
  void a2dp_disconnected() {
    Serial.println("블루투스 스피커에서 A2DP 연결 끊김");
    a2dpConnected = false;
    updateBluetoothLED(); // WS2812 LED 업데이트
  }
};

// --- 전역 객체 ---
MyA2DPSourceCallbacks a2dpCallbacks;

// --- 함수 선언 ---
void onEspNowRecv(const uint8_t *mac_addr, const uint8_t *incomingData, int len);
void setupEspNow(uint8_t channel);
void setupBluetooth();
void updateBluetoothLED();
void handlePairingButton();

// --- ESP-NOW 수신 콜백 함수 (오디오 데이터 및 채널 정보 처리) ---
void onEspNowRecv(const uint8_t *mac_addr, const uint8_t *incomingData, int len) {
  if (len == sizeof(struct_channel_info) && !channelReceived) { // 채널 정보 수신
    struct_channel_info receivedChannelInfo;
    memcpy(&receivedChannelInfo, incomingData, sizeof(receivedChannelInfo));
    if (receivedChannelInfo.channel != currentEspNowChannel) { // 채널이 변경되었으면
      Serial.printf("ESP32 #1로부터 채널 정보 수신: %d. ESP-NOW 재시작.\n", receivedChannelInfo.channel);
      setupEspNow(receivedChannelInfo.channel); // ESP-NOW를 새로운 채널로 재시작
      currentEspNowChannel = receivedChannelInfo.channel;
      // 확인 응답 전송
      struct_ack ack;
      ack.channel_confirmed = true;
      esp_now_send(peerAddress, (uint8_t *)&ack, sizeof(ack));
      Serial.println("채널 정보 확인 응답을 ESP32 #1로 보냈습니다.");
    }
    channelReceived = true; // 채널 정보 수신 완료 플래그 설정
    Serial.println("채널 정보 수신 완료 및 ESP-NOW 초기화 완료.");
  } else if (channelReceived) { // 오디오 데이터 수신
    static size_t expectedLen = 0;
    static uint8_t *receiveBuffer = nullptr;
    static size_t receivedLen = 0;

    if (len == sizeof(size_t) && expectedLen == 0) { // 데이터 길이 수신
      memcpy(&expectedLen, incomingData, sizeof(size_t));
      if (expectedLen > 0 && expectedLen < 2048) {
        receiveBuffer = (uint8_t *)malloc(expectedLen); // 수신 버퍼 할당
        if (receiveBuffer == nullptr) {
          Serial.println("ESP-NOW 수신 버퍼 메모리 할당 실패 (길이)");
          expectedLen = 0;
        }
        receivedLen = 0;
      } else {
        Serial.printf("수신된 데이터 길이 이상: %d\n", expectedLen);
        expectedLen = 0;
      }
    } else if (expectedLen > 0 && receivedLen < expectedLen) { // 오디오 데이터 수신
      size_t remaining = expectedLen - receivedLen;
      size_t bytesToCopy = min((size_t)len, remaining);
      memcpy(receiveBuffer + receivedLen, incomingData, bytesToCopy);
      receivedLen += bytesToCopy;

      if (receivedLen == expectedLen) { // 데이터 수신 완료
        if (xQueueSend(espNowReceiveQueue, &receiveBuffer, 0) != pdTRUE) { // 수신 큐에 데이터 추가
          Serial.println("ESP-NOW 수신 큐에 데이터 추가 실패 (버퍼 가득?)");
          free(receiveBuffer); // 큐에 추가 실패 시 버퍼 해제
        }
        receiveBuffer = nullptr;
        expectedLen = 0;
        receivedLen = 0;
      }
    } else {
      Serial.println("잘못된 ESP-NOW 패킷 수신");
    }
  } else {
    Serial.println("채널 정보를 받기 전에 ESP-NOW 데이터 수신");
  }
}

// --- ESP-NOW 초기화 함수 ---
void setupEspNow(uint8_t channel) {
  WiFi.mode(WIFI_STA);
  if (esp_now_init() != ESP_OK) {
    Serial.println("ESP-NOW 초기화 실패");
    delay(3000);
    ESP.restart();
    return;
  }
  esp_now_set_self_role(ESP_NOW_ROLE_SLAVE); // ESP32 #2는 슬레이브 역할
  esp_now_register_recv_cb(onEspNowRecv);       // 수신 콜백 등록
  esp_now_peer_info_t peerInfo;                 // 피어 정보 구조체
  memcpy(peerInfo.peer_addr, peerAddress, 6); // 피어 MAC 주소 복사
  peerInfo.channel = channel;                  // 사용할 ESP-NOW 채널 설정
  peerInfo.encrypt = false;                    // 암호화 비활성화
  if (esp_now_add_peer(&peerInfo) != ESP_OK) { // 피어 추가
    Serial.println("ESP-NOW 피어 추가 실패");
    return;
  }
  Serial.printf("ESP-NOW 시작 (채널: %d), 피어 MAC: %02x:%02x:%02x:%02x:%02x:%02x\n", channel, channel, peerAddress[0], peerAddress[1], peerAddress[2], peerAddress[3], peerAddress[4], peerAddress[5]);
}

// --- 블루투스 초기화 함수 ---
void setupBluetooth() {
  a2dpSource.register_callback(&a2dpCallbacks);
  a2dpSource.start("ESP32-A2DP-Source");
  Serial.println("A2DP 소스 시작.");

  if (a2dpSource.connect(true)) {
    Serial.println("마지막 페어링된 블루투스 장치에 연결 시도...");
  } else {
    Serial.println("페어링된 블루투스 장치를 찾을 수 없거나 연결 실패.");
    Serial.println("블루투스 스피커가 페어링 모드인지 확인하고 필요하면 페어링 버튼을 누르세요.");
    pairingMode = true; // 초기 연결 실패 시 페어링 모드 활성화
    updateBluetoothLED();
  }
}

// --- WS2812 LED 업데이트 함수 (블루투스 연결 및 페어링 상태 표시) ---
void updateBluetoothLED() {
  if (pairingMode) {
    // 페어링 모드: 보라색 깜빡임
    uint32_t purple = pixels.Color(128, 0, 128);
    pixels.setPixelColor(0, (millis() / 500) % 2 == 0 ? purple : 0x000000);
  } else if (a2dpConnected) {
    pixels.setPixelColor(0, 0x0000FF); // 파란색: 블루투스 연결됨
  } else {
    pixels.setPixelColor(0, 0x000010); // 희미한 파란색: 연결 시도 중 또는 끊김
  }
  pixels.show();
}

// --- 페어링 버튼 처리 함수 ---
void handlePairingButton() {
  button.update(); // Bounce2 라이브러리로 버튼 상태 업데이트
  if (button.fell()) { // 버튼이 눌렸을 때
    Serial.println("페어링 버튼 눌림 (Bounce2)");
    pairingMode = true;
    updateBluetoothLED();
    a2dpSource.connect(true); // 재연결 시도
  }
}

// --- setup 함수 ---
void setup() {
  Serial.begin(115200);
  Serial.println("ESP32 #2 - ESP-NOW 수신기 & A2DP 송신기");

  espNowReceiveQueue = xQueueCreate(WIFI_BUFFER_SLOTS * 2, sizeof(uint8_t *)); // ESP-NOW 수신 큐 생성
  if (espNowReceiveQueue == NULL) {
    Serial.println("ESP-NOW 수신 큐 생성 실패");
  }

  pixels.begin();
  pixels.setBrightness(50);
  pixels.show(); // 초기 LED 끄기

  pinMode(PAIRING_BUTTON_PIN, INPUT_PULLUP); // 페어링 버튼 핀 설정 (풀업)
  button.attach(PAIRING_BUTTON_PIN, INPUT_PULLUP); // Bounce2에 핀 연결 및 풀업 설정
  button.interval(DEBOUNCE_INTERVAL); // 디바운스 간격 설정

  // ESP-NOW 초기화 (기본 채널로 시작)
  setupEspNow(ESPNOW_CHANNEL);
  setupBluetooth();
}

// --- loop 함수 ---
void loop() {
  // 블루투스 연결 상태 확인 및 재연결 시도 (버튼 없을 때)
  if (!a2dpConnected && !pairingMode) {
    static unsigned long lastReconnectAttempt = 0;
    unsigned long now = millis();
    if (now - lastReconnectAttempt > 5000) { // 5초마다 재연결 시도
      Serial.println("블루투스 연결 끊김 감지. 자동 재연결 시도...");
      if (a2dpSource.connect(true)) {
        Serial.println("재연결 시도 중...");
      }
      lastReconnectAttempt = now;
    }
  }

  handlePairingButton(); // 페어링 버튼 상태 확인 및 처리
  updateBluetoothLED();   // LED 상태 업데이트

  uint8_t *audioData;
  if (a2dpConnected && xQueueReceive(espNowReceiveQueue, &audioData, 0)) {
    size_t dataLen = _msize(audioData);
    if (dataLen > 0) {
      a2dpSource.stream_data(audioData, dataLen);
    }
    free(audioData);
  }
  delay(1);
}
```

위 코드는 최종적으로 구현된 모든 요구사항을 포함하고 있으며, 각 기능에 대한 설명을 주석으로 자세하게 추가했습니다. ESP32 #1과 ESP32 #2 각각의 코드를 해당 보드에 업로드하여 테스트해 볼 수 있습니다. ESP32 #2의 `peerAddress`에는 ESP32 #1의 MAC 주소를, ESP32 #1의 `peerAddress`에는 ESP32 #2의 MAC 주소를 정확하게 입력해야 ESP-NOW 통신이 정상적으로 이루어집니다. 또한 WS2812 LED와 페어링 버튼의 핀 번호가 실제 사용하는 핀 번호와 일치하는지 확인해 주세요.
