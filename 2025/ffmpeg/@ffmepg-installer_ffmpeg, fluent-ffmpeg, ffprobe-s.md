<img src="https://r2cdn.perplexity.ai/pplx-full-logo-primary-dark%402x.png" class="logo" width="120"/>

# 

---

# @ffmepg-installer/ffmpeg, fluent-ffmpeg, ffprobe-static

다음에 대하여 설명

`@ffmpeg-installer/ffmpeg`, `fluent-ffmpeg`, 그리고 `ffprobe-static`은 Node.js 환경에서 FFmpeg와 FFprobe를 사용하기 위한 라이브러리입니다. 각각의 라이브러리는 FFmpeg와 FFprobe를 다루는 방식에 차이가 있으며, 사용 목적에 따라 적합한 라이브러리를 선택할 수 있습니다.

## 1. @ffmpeg-installer/ffmpeg

`@ffmpeg-installer/ffmpeg`는 Node.js에서 FFmpeg를 사용하기 위해 FFmpeg 바이너리를 설치하고 관리하는 라이브러리입니다. 이 라이브러리는 FFmpeg 바이너리를 자동으로 다운로드하고 설치하여, FFmpeg 명령어를 Node.js에서 실행할 수 있게 합니다.
- fmpeg를 직접 설치하지 않고 pnpm에서 설치하도록 해주는 툴

### 주요 기능

- **FFmpeg 바이너리 설치**: 시스템에 FFmpeg 바이너리가 설치되지 않은 경우 자동으로 다운로드 및 설치합니다.
- **FFmpeg 명령어 실행**: FFmpeg 명령어를 Node.js에서 실행할 수 있는 인터페이스를 제공합니다.


### 사용 예시

```javascript
const ffmpeg = require('@ffmpeg-installer/ffmpeg');

// FFmpeg 명령어 실행
ffmpeg.run('-version', (err) => {
  if (err) {
    console.error(err);
  }
});
```


## 2. fluent-ffmpeg

`fluent-ffmpeg`는 FFmpeg를 Node.js에서 사용하기 위한 더 높은 수준의 API를 제공하는 라이브러리입니다. 이 라이브러리는 FFmpeg 명령어를 생성하고 실행하는 것을 더 쉽게 만들어줍니다.

### 주요 기능

- **명령어 체이닝**: FFmpeg 명령어를 체이닝 방식으로 쉽게 생성할 수 있습니다.
- **스트림 처리**: 비디오 및 오디오 스트림을 처리하는 기능을 제공합니다.


### 사용 예시

```javascript
const ffmpeg = require('fluent-ffmpeg');

// 비디오 파일을 MP4로 변환
ffmpeg('input.mp4')
  .setFormat('mp4')
  .save('output.mp4');
```


## 3. ffprobe-static

`ffprobe-static`은 FFprobe 바이너리를 Node.js에서 사용하기 위한 라이브러리입니다. FFprobe는 FFmpeg와 함께 제공되는 도구로, 멀티미디어 파일의 메타데이터를 분석하는 데 사용됩니다.

### 주요 기능

- **FFprobe 바이너리 설치**: 시스템에 FFprobe 바이너리가 설치되지 않은 경우 자동으로 다운로드 및 설치합니다.
- **메타데이터 분석**: 멀티미디어 파일의 메타데이터를 분석할 수 있는 인터페이스를 제공합니다.


### 사용 예시

```javascript
const ffprobe = require('ffprobe-static');

// 비디오 파일의 메타데이터 분석
ffprobe('input.mp4', (err, metadata) => {
  if (err) {
    console.error(err);
  } else {
    console.log(metadata);
  }
});
```


## 요약

- **@ffmpeg-installer/ffmpeg**: FFmpeg 바이너리를 설치하고 명령어를 실행하는 데 사용됩니다.
- **fluent-ffmpeg**: FFmpeg 명령어를 체이닝 방식으로 생성하고 실행하는 데 사용됩니다.
- **ffprobe-static**: FFprobe 바이너리를 설치하고 멀티미디어 파일의 메타데이터를 분석하는 데 사용됩니다.

각 라이브러리는 FFmpeg와 FFprobe를 다루는 방식에 차이가 있으며, 사용 목적에 따라 적합한 라이브러리를 선택할 수 있습니다.

