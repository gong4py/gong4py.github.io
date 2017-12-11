# Introduction in Acoustic analysis w/ Python

Taehun Kim

---

## 갑자기 왜 때문에 소리 분석을 한다는 것인가?

<del>나도 모름...<del>

__교수님: 태훈아 정말로 데이터만 있으면 뭐든 다 할 수 있나?__

김: (또 무슨 얘기를 하시려고...)

__교수님: 멍멍이 짖는 소리만 듣고 얘가 어떤 기분인지를 맞춰야 한다__

김: 멍멍이요...?

### 우리의 대략적인 로드맵

![pet_emotion_analysis_overview.png](https://i.imgur.com/J6UQhwQ.png)


### 조사 결과

- Acoustic data analysis를 하기위해 Signal processing을 해야함

- Signal processing의 방법으로 푸리에 변환이라는게 있음!

- 이걸 좀 더 공부해보자, 관련 라이브러리는 파이선이 다 해결해주실거야 (/기도)

---

## 소리 분석의 기본

### 소리의 입력을 디지털로 변환 및 처리

#### Sampling: Analog를 digital로 변환

- 음악 파일에서 봤던 샘플링 rate가 이거랑 관련된 것임

- 음성을 특정 주기로 기록함

- 샘플링 rate가 높을수록 원래 소리에 가까움

- 예시: 44.1 kHz (위), 22.05 kHz (아래)

![sampling_sig.png](https://imgur.com/dHVGO2U.png)

#### Quantization: 샘플링된 소리를 몇개의 구간으로 양자화

![quantization_sig.png](https://imgur.com/r6hiROt.png)


### 변환된 소리에서 소리 구성 요소 분해

#### 소리의 구성 3요소
 
- 진폭 (소리의 크기)
 
- 주기 (소리의 높이)
 
- 파형 (소리의 형태)

![form_of_sig.png](https://i.imgur.com/Yh5jger.png)

__동일한 주기, 진폭이라도 파형이 다르다면 우리는 두 소리를 구분할 수 있다!__



#### 파형에 대한 추가 설명
 
- 파형은 주기나 위상이 다른 여러 파형의 합성으로 표현 가능
 
- 실제로 우리가 듣는 대부분의 소리는 합성음임
 
![sum_of_sig.png](https://imgur.com/00mGMor.png)



#### 소리의 요소를 분해하는 방법론

- ___Discrete Fourier Transform (DFT)___
 
- ___Short-time Fourier Transform (STFT)___
 
- Wavelet Transform (WT) (이 발표에서 생략)

- 대략 느낌적인 느낌으로 설명하면 이런 그림으로 표현할 수 있다.
 
![Fourier_series_sawtooth_wave_circles_animation.gif](https://i.imgur.com/9AY1iua.gif)
 
---

## 우선 소리를 받아오는 방법

이쯤에서 먼저 우리의 멍멍이 소리를 들어보자

### scipy, numpy, python 자체 등 .wav 파일을 받아올 수 있는 방법은 많음

- 그러나, 대다수의 method가 stereo 형식에는 안맞아서 mono로 변환을 해줘야함

- 이 외에도 까다로운게 좀 있어서, 언제는 scipy, numpy를 쓰는게 좋고 언제는 그냥 wave를 써서 받아오는게 좋다

- 결국 하려는게 뭐냐 따라서 어떻게 받아오느냐를 결정하는 것이 편함


### wave lib을 이용해 받아오는 방법

```python
import wave
import numpy as np

soundfileName = 'Dog-bark.wav'
wr = wave.open(soundfileName, 'r')
sz = wr.getframerate()
frames = wr.getnframes()
duration = int(np.floor(frames / float(sz)))
sf = 1.5  # signal scale factor
```

### scipy를 이용해 받아오는 방법

```python
from scipy import signal
from scipy.io import wavfile

soundfileName = 'Dog-bark.wav'

sample_rate, samples = wavfile.read(soundfileName)
samples = samples.sum(axis=1)/2 # make stereo to mono
```

### 받아온 소리를 플로팅해보자

```python
# continued the code
import matplotlib.pyplot as plt

snd = np.array([])
for j in range(duration):
    da = np.fromstring(wr.readframes(sz), dtype=np.int16)
    left, right = da[0::2]*1.5, da[1::2]*1.5
    snd = np.concatenate((snd, (left+right)/2))
    
x = np.arange(44100*duration)/44100
plt.ylim([-50000, 50000])
plt.xlabel('Time [s]')
plt.ylabel('|Amplitude|')
plt.plot(x, snd)
plt.show()
```

![sound_plot.png](https://imgur.com/p9rfQDA.png)

---

## 받아온 소리를 분석해보자

### Discrete Fourier Transform (DFT)

#### 입력 음성을 주기성을 갖는 여러 음성으로 분리해내는 방법

- Ex) 붉은 파형의 소리는 여러개의 파란 파형의 조합으로 나타낼 수 있음 

- 계속해서 반복되는 종류의 음성 분석에 유용함 (예: 엔진, 모터, SONAR 등)

![dft.png](https://imgur.com/BQWRd1e.png)


#### DFT의 수학적 의미:
 
- 하나의 소리는 주기가 서로 다른 sin, cos 파형의 소리의 조합으로 나타낼 수 있음
 
![dft_fomula.png](https://i.imgur.com/CA4uJjP.png)


#### DFT를 해보자

DFT는 제공되는 fft 알고리즘을 사용하면 할 수 있는데,

곧바로 찍는건 조금 어렵다.

아래의 코드를 이용해보자

```python
# continued the code
avgf = np.zeros(int(sz/2+1))
for j in range(duration):
    da = np.fromstring(wr.readframes(sz), dtype=np.int16)
    left, right = da[0::2]*1.5, da[1::2]*1.5
    lf, rf = abs(np.fft.rfft(left)), abs(np.fft.rfft(right))
    avgf += (lf+rf)/2
avgf /= duration
x = np.arange(44100*duration)/44100
plt.xlim([100,2000])  # our sound has 100~2000Hz elements only
plt.xlabel('Frequency [Hz]')
plt.ylabel('|Amplitude|')
plt.plot(abs(avgf))
plt.show()
```

![dft_plot.png](https://imgur.com/kF4qM0d.png)

---

### Short-time Fourier Transform (STFT)

#### 짧은 시간 동안의 DFT를 나타내고 이를 반복해서 진행

- 시간에 따라 핵심 주파수가 변하는 경우 분석에 용이

![stft.png](https://imgur.com/OFHOVtj.png)


#### STFT의 수학적 의미:

- 짧은 시간 동안의 DFT를 rolling 해가며 적용

- 다음번 time-window의 구성 요소는 이전의 구성 요소와 다를 수 있음 (cf. DFT에서는 시간에 상관없이 구성 요소가 동일)

![stft_fomula.png](https://i.imgur.com/nxKZ2oI.png)


#### STFT를 해보자

DFT랑 다르게 그냥 scipy에서 제공해주는 Spectogram method를 쓰는게 가장 편하다

```python
frequencies, times, spectogram = signal.spectrogram(samples, sample_rate)

plt.pcolormesh(times, frequencies, spectogram)
plt.ylim(ymax=2500)
plt.ylabel('Frequency [Hz]')
plt.xlabel('Time [s]')
plt.show()
```

![spectogram.png](https://imgur.com/LcmIZfI.png)

---

## Conclusion

![acoustic_analysis_full_plot.png](https://imgur.com/u3rHiEp.png)

- 분리해내는 방법은 공부했지만 결국 어떤 방법으로 분해하는지가 중요한지는 case by case 인듯

- 멍멍이 소리를 학습하는 작업은 우선 정지...

- <del>그래도 공4파이 발표자료는 남았잖아...<del>