# laceroom앱

도금 작업 지시서에서 출발해, 매출채권·거래처·원단 공급사·프리랜서 인건비·급여·고정비·사업경비·보조금·메모·부자재 바코드·마스터 상품·사용자/PC 관리·변경 로그까지 통합한 단일 페이지 React 18 앱.

라이브 코드는 루트의 `index.html` **하나뿐**이다. 빌드 도구·테스트 슈트·`package.json`은 없다. React, Tailwind, Babel-standalone, ExcelJS, SheetJS 모두 CDN 로딩.

## 실행

캐시 무효화를 위해 항상 `?v=숫자` 쿼리스트링을 붙여서 연다.

- **GitHub Pages (배포본)**: <https://yslike1399-maker.github.io/plating/?v=숫자>
- **로컬 파일**: `index.html?v=숫자`

> 로컬 파일에서 localStorage·이미지 접근이 필요하면 Chrome을 `--allow-file-access-from-files` 플래그로 실행한다.

## 작업 규칙

이 프로젝트의 모든 작업 규칙(한국어 응답, 호칭 "대장님", 작업 전 확인, 백업, 점진 변경, 탭 번호 소통, 색상명 이중 체계, 버전 형식, 접속 형식, 프로젝트 호칭 통일)은 [`CLAUDE.md`](./CLAUDE.md)에 정의되어 있다. **편집 전 반드시 읽을 것.**

## 디렉토리

```
plating/
├── index.html              # 라이브 코드 (단일 파일 앱)
├── images/                 # 상품 이미지
├── CLAUDE.md               # 작업 규칙 + 아키텍처 노트
├── README.md               # 이 파일
└── archive/snapshots/      # 옛 백업·스냅샷 (참고용, 편집 금지)
```

## 데이터 저장

전 도메인이 브라우저 `localStorage`에 snake_case 키로 영속화된다. 스키마·마이그레이션 패턴은 `CLAUDE.md`의 "Big-picture architecture" 참조.

## 버전

`v{yyyymmdd}.{숫자}` 형식. 같은 날 안에서는 숫자 증가, 날짜 바뀌면 `.01` 리셋. (예: `v20260502.03`)
