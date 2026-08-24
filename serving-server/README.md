# 돋음 AI 추론 서버

FastAPI로 Wav2Lip 립싱크 생성과 Omnilingual ASR 음성 인식을 제공하는 서버입니다. `PORT` 값에 따라 한 프로세스가 하나의 기능만 노출합니다.

| `PORT` | 모드 | API |
| --- | --- | --- |
| `8000` | Wav2Lip | `POST /api/v1/lip-video` |
| `8080` | STT | `POST /api/v1/stt/transcribe` |

두 모드 모두 `GET /`와 `GET /health`를 제공합니다. `PORT`를 지정하지 않으면 Wav2Lip 모드로 실행됩니다.

## 준비 사항

- Python 3.11과 Linux 또는 WSL2 환경
- FFmpeg와 FFprobe
- 입력·출력 파일을 저장할 Google Cloud Storage 버킷과 서비스 계정
- Wav2Lip 모드: `models/Wav2Lip/checkpoints/wav2lip_gan.pth`
- GPU 실행: 호환되는 NVIDIA 드라이버와 CUDA 런타임

모델 체크포인트와 GCP 인증 파일은 저장소에 포함하지 않습니다.

## 환경 설정

```bash
cp .env.example .env
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

현재 의존성에는 Linux용 NVIDIA 패키지가 포함되어 있어 Windows 네이티브 실행은 검증하지 않았습니다. `.env`에는 다음 값이 필요합니다.

```dotenv
PORT=8000
DEBUG=false
GCP_PROJECT_ID=your-project-id
GCS_BUCKET=your-bucket-name
GCS_CREDENTIAL_PATH=credentials/key.json
```

## 로컬 실행

Wav2Lip 서버:

```bash
PORT=8000 uvicorn api.main:app --host 0.0.0.0 --port 8000
```

STT 서버:

```bash
PORT=8080 uvicorn api.main:app --host 0.0.0.0 --port 8080
```

PowerShell에서는 먼저 `$env:PORT = '8000'` 또는 `$env:PORT = '8080'`을 설정한 뒤 `uvicorn`을 실행합니다.

## Docker 실행

두 Compose 파일은 Wav2Lip 모드를 컨테이너의 8000번 포트에서 실행하고 호스트의 `http://localhost:8001`로 노출합니다. GPU 설정이 포함돼 있으므로 NVIDIA Container Toolkit이 준비된 환경을 전제로 합니다.

```bash
docker compose -f docker-compose.dev.yml up --build
docker compose up --build -d
```

## 요청 예시

립싱크 생성:

```bash
curl -X POST http://localhost:8000/api/v1/lip-video \
  -H "Content-Type: application/json" \
  -d '{
    "user_video_gs": "gs://your-bucket/input.mp4",
    "gen_audio_gs": "gs://your-bucket/audio.wav",
    "output_video_gs": "gs://your-bucket/output.mp4",
    "word": "안녕하세요"
  }'
```

`target_fps`를 생략하면 `word` 길이에 따라 15fps 또는 18fps를 사용합니다.

음성 인식:

```bash
curl -X POST http://localhost:8080/api/v1/stt/transcribe \
  -H "Content-Type: application/json" \
  -d '{
    "audio_gs": "gs://your-bucket/audio.wav",
    "lang": "kor_Hang",
    "model_size": "300M"
  }'
```

`audio_gs` 대신 서버가 접근할 수 있는 `audio_url`을 보낼 수 있습니다.

## 검증

현재 추론 모델을 포함한 자동화 테스트는 제공하지 않습니다. 모델 없이 실행 가능한 최소 정적 검사는 다음과 같습니다.

```bash
python -m compileall api
```

실제 동작 검증에는 GCS 인증 정보와 모델 체크포인트가 필요합니다. Wav2Lip 모드의 `/health` 응답이 `warning`이면 체크포인트 경로를 확인하세요.

## 외부 코드와 사용 조건

`models/Wav2Lip`은 Wav2Lip 기반 코드입니다. 원본 프로젝트의 안내에 비상업적·연구·개인 목적 사용 제한이 명시돼 있으므로 배포 또는 상업적 이용 전에 `models/Wav2Lip/README.md`의 조건과 원 저작권을 확인해야 합니다.
