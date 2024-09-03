소리의 방향 탐지를 위한 신호 처리에서 **FFT(Fast Fourier Transform)**를 사용하는 것은 신호를 주파수 도메인으로 변환하여 보다 정밀하게 분석하는 데 유용합니다. 이를 통해 크로스 코릴레이션을 주파수 도메인에서 수행할 수 있으며, 시간 도메인보다 더 정확하게 신호의 시간 차이를 계산할 수 있습니다.

### 1. FFT의 기본 개념
FFT는 시간 도메인의 신호를 주파수 도메인으로 변환하여 각 주파수 성분을 분석합니다. 주파수 도메인에서 두 신호의 상관성을 분석하면 신호 간의 시간 차이(TDOA)를 더욱 정밀하게 계산할 수 있습니다.

### 2. FFT 적용 흐름
1. **샘플링**: I2S 마이크로폰을 통해 소리 신호를 수집합니다.
2. **FFT 변환**: 수집한 신호에 대해 FFT를 수행하여 주파수 도메인으로 변환합니다.
3. **크로스 스펙트럼 계산**: 두 마이크로폰 신호의 크로스 스펙트럼(Cross Spectrum)을 계산하여 위상 차이를 구합니다.
4. **IFFT(Inverse FFT)**: 크로스 스펙트럼에 대한 IFFT를 수행하여 시간 도메인에서의 시간 차이를 계산합니다.

### 3. 코드 예제

#### 필요 라이브러리
- **Arduino FFT 라이브러리**: FFT 계산에 사용됩니다.
- **Arduino I2S 라이브러리**: I2S 마이크로폰으로 데이터를 수집합니다.

먼저, 필요한 라이브러리를 설치하고 아래 예제를 실행하세요.

```cpp
#include <ArduinoFFT.h>
#include <I2S.h>

#define SAMPLE_RATE 16000    // 샘플링 레이트 (Hz)
#define BUFFER_SIZE 1024     // FFT를 위한 버퍼 크기
#define MIC_PIN_WS  25       // Word Select (WS) 핀
#define MIC_PIN_SD  32       // Serial Data (SD) 핀
#define MIC_PIN_SCK 26       // Clock (SCK) 핀

ArduinoFFT FFT = ArduinoFFT();

double mic1Real[BUFFER_SIZE];
double mic1Imag[BUFFER_SIZE];
double mic2Real[BUFFER_SIZE];
double mic2Imag[BUFFER_SIZE];

void setup() {
  Serial.begin(115200);

  // I2S 설정
  I2S.begin(I2S_PHILIPS_MODE, SAMPLE_RATE, 32);  // 32-bit 데이터
  I2S.setPins(MIC_PIN_SD, MIC_PIN_SCK, MIC_PIN_WS);
}

void loop() {
  // 버퍼에 데이터 읽기
  for (int i = 0; i < BUFFER_SIZE; i++) {
    mic1Real[i] = (double)I2S.read();
    mic2Real[i] = (double)I2S.read();
    mic1Imag[i] = 0;
    mic2Imag[i] = 0;
  }

  // FFT 변환
  FFT.Windowing(mic1Real, BUFFER_SIZE, FFT_WIN_TYP_HAMMING, FFT_FORWARD);
  FFT.Compute(mic1Real, mic1Imag, BUFFER_SIZE, FFT_FORWARD);
  FFT.Windowing(mic2Real, BUFFER_SIZE, FFT_WIN_TYP_HAMMING, FFT_FORWARD);
  FFT.Compute(mic2Real, mic2Imag, BUFFER_SIZE, FFT_FORWARD);

  // 크로스 스펙트럼 계산 (복소수 곱셈)
  double crossReal[BUFFER_SIZE];
  double crossImag[BUFFER_SIZE];
  for (int i = 0; i < BUFFER_SIZE; i++) {
    crossReal[i] = mic1Real[i] * mic2Real[i] + mic1Imag[i] * mic2Imag[i];
    crossImag[i] = mic1Imag[i] * mic2Real[i] - mic1Real[i] * mic2Imag[i];
  }

  // IFFT를 통해 크로스 코릴레이션 계산
  FFT.Compute(crossReal, crossImag, BUFFER_SIZE, FFT_BACKWARD);

  // 최대 크로스 코릴레이션을 찾고 시간 차이 계산
  int maxIndex = 0;
  double maxVal = 0;
  for (int i = 0; i < BUFFER_SIZE; i++) {
    double magnitude = sqrt(crossReal[i] * crossReal[i] + crossImag[i] * crossImag[i]);
    if (magnitude > maxVal) {
      maxVal = magnitude;
      maxIndex = i;
    }
  }

  int timeDelayIndex = maxIndex < BUFFER_SIZE / 2 ? maxIndex : maxIndex - BUFFER_SIZE;

  // 소리의 방향 계산
  float speedOfSound = 343.0; // m/s
  float distanceBetweenMics = 0.1; // meters

  float timeDelay = (float)timeDelayIndex / SAMPLE_RATE;
  float angle = asin(timeDelay * speedOfSound / distanceBetweenMics) * 180.0 / PI;

  Serial.print("Time Delay: ");
  Serial.println(timeDelay);
  Serial.print("Angle: ");
  Serial.println(angle);

  delay(1000);
}
```

### 4. 코드 설명
- **I2S 초기화 및 데이터 수집**: I2S 인터페이스를 통해 두 마이크로폰의 데이터를 샘플링합니다.
- **FFT 변환**: 수집된 데이터를 주파수 도메인으로 변환합니다. `Windowing` 함수를 통해 신호에 해밍 윈도우를 적용하여 FFT의 정확도를 높입니다.
- **크로스 스펙트럼 계산**: 두 신호의 FFT 결과를 곱하여 크로스 스펙트럼을 계산합니다. 이 과정에서 신호 간의 위상 차이를 구할 수 있습니다.
- **IFFT 적용**: 크로스 스펙트럼에 IFFT를 적용하여 시간 도메인으로 변환합니다. 이때, 최대 상관 값을 가진 인덱스를 찾고, 이를 이용해 소리의 시간 차이와 방향을 계산합니다.
  
### 5. 개선 및 최적화
- **신호 필터링**: 입력 신호에 저역통과 필터(Low-pass Filter)를 적용하여 고주파 노이즈를 제거하고 분석할 주파수 대역을 설정할 수 있습니다.
- **창 함수(Window Function)**: 해밍 윈도우 외에도 다른 창 함수를 시도하여 FFT 성능을 개선할 수 있습니다.
- **정확도 향상**: 샘플링 레이트를 높이거나 버퍼 크기를 증가시켜 시간 차이 계산의 정밀도를 높일 수 있습니다.

이 예제는 소리의 방향 탐지를 위해 FFT를 활용하는 방법을 보여줍니다. 이 방식은 시간 도메인에서의 단순한 신호 처리보다 더 높은 정확도를 제공하며, 특히 복잡한 환경에서 유용합니다.

