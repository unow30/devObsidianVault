설치파일인 @ffmepg-installer/ffmpeg와 메타데이터 분석 툴인 ffprobe-static을 fluent-ffmpeg가 사용할 수 있도록 설정한다.

```node
//main.ts

import * as ffmpeg from '@ffmpeg-installer/ffmpeg';  
import * as ffmpegFluent from 'fluent-ffmpeg';  
import * as ffprobe from 'ffprobe-static';  
  
ffmpegFluent.setFfmpegPath(ffmpeg.path);  
ffmpegFluent.setFfprobePath(ffmpeg.path);

async function bootstrap() {...}
```