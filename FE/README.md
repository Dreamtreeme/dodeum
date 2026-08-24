# 돋음 프론트엔드

돋음의 학습·녹음·결과 확인 화면을 제공하는 React 애플리케이션입니다. React 19, TypeScript, Vite 7을 사용하며 기능 단위 코드는 `src/features`, 공통 코드는 `src/shared`에 둡니다.

## 실행

Vite 7이 지원하는 Node.js 20.x(20.19 이상) 또는 22.12 이상과 npm이 필요합니다.

```bash
npm ci
cp .env.example .env
npm run dev
```

Windows PowerShell에서는 두 번째 명령 대신 `Copy-Item .env.example .env`를 사용합니다. 기본 개발 서버는 `http://localhost:5173`입니다.

`.env`에서 백엔드 API 주소를 설정합니다.

```dotenv
VITE_BASE_URL=http://localhost:8000/api/v1
```

## 검증

```bash
npm run lint
npm run build
```

`npm run build`는 TypeScript 프로젝트 빌드와 Vite 프로덕션 번들 생성을 차례로 실행합니다.

## 주요 디렉터리

- `src/features`: 인증, 발음 연습, 음성 훈련, 결과 조회 등 기능별 코드
- `src/shared`: API 클라이언트, 공통 컴포넌트, 훅과 유틸리티
- `src/App.tsx`, `src/features/*/pages`: 라우트와 화면 구성
- `public/wasm`, `public/models`: 브라우저 얼굴 감지에 사용하는 MediaPipe 런타임과 모델

전체 시스템 구성과 백엔드 실행 방법은 저장소 루트의 `README.md`를 참고하세요.
