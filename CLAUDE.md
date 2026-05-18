# CLAUDE.md — equip-sync-l-module (라벨/접수증 프린터)

이 레포의 설계·운영 문서는 **`dps-store`** 프로젝트에서 통합 관리한다. Claude 세션을 이 레포에서 실행하더라도 아래 문서를 우선 참조하라.

## 문서 진입점

- **dps-store 로컬 경로**: `~/Workspace/dps-store`
- **프린터 문서 인덱스**: [`dps-store/docs/print/README.md`](../dps-store/docs/print/README.md) — 모듈별 추천 읽기 순서
- **외부 레포 통합 정책**: `dps-store/CLAUDE.md`의 "관련 외부 레포" 섹션

### 라벨 모듈 담당이 읽어야 할 문서 (순서대로)
1. `dps-store/docs/print/20260511-equipment-gui-unification.md` — 3개 모듈 공통 아키텍처
2. `dps-store/docs/print/20260511-equipment-gui-spec.md` — GUI 공통 규칙·코드 샘플
3. `dps-store/docs/print/20260310-label-printer-watcher-spec.md` — Watcher 명세
4. `dps-store/docs/print/20260311-label-printer-agent-spec.md` — Agent 명세
5. `dps-store/docs/print/20260310-printer-client-api.md` — 서버 API 명세
6. `dps-store/docs/print/20260310-label-printer-troubleshooting.md` — 운영 트러블슈팅

## 모듈 개요

- SLK TS200 감열 라벨 프린터로 DPS Store 접수증 자동 출력 (Windows)
- Watcher + Agent 통합 단일 EXE (v3.0 이후, PyInstaller)
- 빌드 산출물: `equip-sync-l-vX.Y.Z.exe` (태그 push 시 GitHub Actions 자동 빌드)
- `config.ini` / 런타임 폴더(incoming/processing/done/error) / 로그는 **exe 옆 경로**에 자동 생성
- poppler Windows 바이너리는 PyInstaller 번들에 포함

## 디렉토리 구조 (2026-05-18 평탄화)

```
equip-sync-l-module/
├── .github/workflows/build.yml    # tag push → 자동 빌드 & Release
├── assets/fonts/                  # Pretendard 번들
├── gui/                           # 슬라이드 패널, 헤더, 카드, 로그 박스
│   ├── app.py
│   ├── settings_panel.py          # 프린터명 드롭다운 + 새로고침
│   └── ...
├── main.py                        # 진입점 (fonts.register → WatcherApp.mainloop)
├── config.py                      # config.ini 로드·자동 생성
├── printer.py                     # win32 출력 + list_printers()
├── agent_worker.py                # API 풀링 → 접수증 출력
├── api_client.py
├── auth.py                        # Device Auth (브라우저 승인)
├── watcher.py                     # 폴더 감시 (Watcher 모드)
├── processor.py                   # 파일 처리 흐름
├── receipt_builder.py             # 영수증 이미지 생성
├── build.bat
├── label-printer-watcher.spec     # PyInstaller spec (참고용, 빌드는 build.bat/Actions)
├── requirements.txt
├── GUIDE.txt                      # 매장 배포 가이드
└── CLAUDE.md
```

## 개발·릴리즈 흐름

1. 로컬에서 코드 수정 (`python3 -m py_compile` 으로 macOS에서 문법 검증 가능)
2. `dps-store/CLAUDE.md`의 커밋 규칙 동일 적용 (관련 파일 3개씩, `feat:`/`fix:`/`refactor:`)
3. `git tag vX.Y.Z && git push origin vX.Y.Z` → GitHub Actions가 단일 EXE 빌드 + Release 자동 생성
4. 과거 태그/릴리즈는 최신 2개만 유지 (`gh release delete <tag> --cleanup-tag --yes`)
