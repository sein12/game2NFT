# game2NFT

게임 아이템 코드를 NFT로 발행하고, 전용 토큰으로 거래하는 Web3 거래소 프로토타입입니다. React 웹 화면, 이미지·메타데이터를 생성하는 Express 서버, ERC-20·ERC-721·마켓플레이스 스마트 컨트랙트로 구성되어 있습니다.

> 이 저장소는 [Minkun00/game2NFT](https://github.com/Minkun00/game2NFT)를 포크한 저장소입니다. 아래 내용은 이 포크의 `main` 브랜치에 포함된 코드를 기준으로 정리했습니다. `Game/`에는 Unity 프로젝트 소스가 없으며, 현재 확인할 수 있는 구현은 `Exchange/`에 있습니다.

## 주요 기능

| 기능 | 구현 내용 |
| --- | --- |
| 지갑 연결 | Kaikas의 `window.klaytn`을 이용한 계정 연결 |
| 아이템 NFT 발행 | 17자리 아이템 코드 해석 → 이미지 합성 → Pinata 업로드 → NFT 민팅 |
| 보유 NFT 조회 | 지갑이 보유한 NFT의 이미지와 메타데이터 표시 |
| 판매 등록 | NFT 전송 권한 승인 후 가격을 지정해 거래소에 등록 |
| NFT 구매 | ERC-20 토큰 사용 승인 후 등록된 NFT 구매 |
| 토큰 구매 | 네이티브 코인으로 `MyToken` 발행; 현재 UI는 5 KLAY 단위 구매 |
| 아이템 코드 반환 | 메타데이터의 코드 복호화와 NFT 소각 후 코드 표시 |

화면 이름은 `Create Auction`이지만 컨트랙트는 **고정 가격 판매**를 구현합니다. 게임으로 아이템을 실제 지급하거나 회수하는 연동 코드는 포함되어 있지 않습니다.

## 구성과 기술

| 영역 | 기술·주요 파일 |
| --- | --- |
| 웹 | React 18, React Router 6, caver-js, eject된 Create React App / webpack 구성 |
| API 서버 | Node.js, Express, Axios, Sharp, CryptoJS |
| 파일 저장 | Pinata를 통한 IPFS 이미지·메타데이터 저장 |
| 컨트랙트 | Solidity `^0.8.18`, OpenZeppelin; 루트 lockfile의 OpenZeppelin 버전은 `4.0.0` |
| 지갑·네트워크 | 코드 작성 당시의 Kaikas / Klaytn Baobab testnet 구성 |

```text
game2NFT/
├── Exchange/
│   ├── contracts/                 # ERC-20, ERC-721, 마켓플레이스 Solidity
│   ├── src/
│   │   ├── App.jsx                # 라우팅과 컨트랙트 주소 설정
│   │   └── contracts/             # 지갑 연결, 민팅, 판매, 구매 UI와 ABI
│   ├── server/
│   │   ├── server.js              # POST /api/pinata
│   │   └── Pinata/                # 이미지 합성, 코드 암복호화, IPFS 업로드
│   ├── config/                   # webpack / Jest 설정
│   └── package.json
├── Game/                          # 자리표시자 디렉터리
└── package.json                   # Solidity 관련 의존성 등
```

## 로컬 실행 준비

Node.js와 npm, Kaikas 호환 브라우저 지갑, Pinata API 키가 필요합니다. 지갑에서 사용할 네트워크에 배포된 세 컨트랙트의 주소와 ABI도 준비해야 합니다. 저장소에는 Node.js 버전 고정 파일이나 컨트랙트 배포 스크립트가 없습니다.

네트워크 이름과 기존 배포 주소는 과거 개발 환경의 기록입니다. 현재 지갑·네트워크와의 호환성 및 컨트랙트 사용 가능 여부를 확인한 뒤 설정하세요.

### 1. 소스와 의존성

```bash
git clone https://github.com/sein12/game2NFT.git
cd game2NFT
npm ci
cd Exchange
npm ci
```

루트의 설치는 OpenZeppelin 등의 의존성을 위한 것이며, 웹과 서버 실행에 사용하는 패키지는 `Exchange/package.json`에 있습니다.

### 2. 프런트엔드 환경 변수

`Exchange/.env`를 작성합니다. 세 주소는 동일한 네트워크에 배포된 컨트랙트를 가리켜야 하며, `Exchange/src/contracts/Hardhat_abis/`의 ABI와 일치해야 합니다.

```dotenv
REACT_APP_NFT_CONTRACT_ADDRESS=your_nft_contract_address
REACT_APP_TOKEN_CONTRACT_ADDRESS=your_token_contract_address
REACT_APP_MARKET_CONTRACT_ADDRESS=your_marketplace_contract_address
```

`REACT_APP_*` 변수는 브라우저 번들에 포함됩니다. Pinata 비밀 키는 다음 서버 설정에만 넣습니다.

### 3. 서버 환경 변수

`Exchange/server/.env`를 작성합니다. 기존 `.env.example`에는 빠져 있는 `PARSE_KEY`도 필요합니다.

```dotenv
ACCESS_KEY=your_pinata_api_key
SECRET_ACCESS_KEY=your_pinata_secret_api_key
PARSE_KEY=your_private_item_code_encryption_key
PORT=3001
```

`Pinata.js`와 `hash.js`는 실행 디렉터리의 `.env`를 읽고 이미지 경로도 상대 경로로 사용합니다. 따라서 **서버는 반드시 `Exchange/server`에서 실행**합니다. `server.js`에 별도로 적힌 `../src/Pinata/.env`는 현재 파일 구조와 맞지 않습니다.

### 4. 서버와 웹 실행

터미널 1 — 저장소 루트에서:

```bash
cd Exchange/server
node server.js
```

터미널 2 — 저장소 루트에서:

```bash
cd Exchange
npm start
```

기본 웹 주소는 `http://localhost:3000`, API는 `http://localhost:3001/api/pinata`입니다. 프런트엔드의 이미지 생성·복호화 요청에는 이 API 주소가 직접 지정되어 있으므로 서버 포트를 바꾸면 해당 요청 코드도 함께 수정해야 합니다.

웹 빌드는 `Exchange`에서 `npm run build`로 수행하며 결과는 `Exchange/build/`에 생성됩니다. 위 절차는 소스와 스크립트 기준 안내이며, 지갑 연결·배포·실제 거래까지의 실행 검증 결과를 의미하지 않습니다.

## 사용 흐름

1. 지갑을 연결하고 설정한 컨트랙트와 동일한 네트워크를 선택합니다.
2. **Item to Image**에서 아이템 코드를 입력합니다. 예: `10130100FF0000123`.
3. 서버가 이미지를 합성하고 메타데이터를 업로드합니다. 이미지가 로드되면 웹에서 민팅 트랜잭션을 요청합니다.
4. **Create Auction**에서 NFT를 선택하고 권한 승인과 판매 등록을 진행합니다.
5. **Marketplace**에서 토큰 사용 승인과 NFT 구매를 진행합니다.
6. 보유 NFT를 **Burn**하면 소각 후 복호화한 아이템 코드가 표시됩니다. 게임 내 지급은 별도 구현 대상입니다.

아이템 코드는 `수식어 3자리 + 장비군 2자리 + 부위 3자리 + 배경색 6자리 + 등급 3자리`로 구성됩니다. 코드 값은 [itemCode.json](Exchange/server/Pinata/itemCode.json), 예시 설명은 [서버 문서](Exchange/server/README.md)를 참고하세요.

## 구현 범위와 확인할 점

- 마켓플레이스는 판매 대금의 10%를 소유자에게 수수료로 전달하도록 구현되어 있습니다.
- 구매 UI는 ERC-20 결제와 함께 `value: price`도 전송하지만, 컨트랙트의 결제 처리는 ERC-20 기준입니다. 거래를 재현하기 전에 이 불일치를 확인해야 합니다.
- `Game/`에는 `.gitignore`와 `.gitkeep`만 있습니다. Unity 실행, 게임 로그인, 아이템 진위 확인 및 게임 연동은 이 저장소의 완성된 기능으로 볼 수 없습니다.
- 컨트랙트를 다시 컴파일할 때는 루트 lockfile의 OpenZeppelin `4.0.0`과 소스의 상속·override 형식을 함께 확인하세요. 저장소에 자동 배포 환경은 포함되어 있지 않습니다.
- `npm test` 스크립트는 있지만 Jest 설정은 `src/setupTests.js`를 가리키고 실제 파일은 `src/tests/setupTests.js`에 있어 경로 정리가 필요합니다.

## 원본과 개발 기록

- 원본 프로젝트: [Minkun00/game2NFT](https://github.com/Minkun00/game2NFT)
- 기존 README에 기록된 데모: [GitHub Pages](https://minkun00.github.io/game2NFT/) — 원본 프로젝트의 과거 데모 주소이며 현재 동작은 별도 확인이 필요합니다.
- 2023-11-05: 기존 문서에 아이템 코드 기반 NFT 생성과 거래소 데모 구현 기록.
- 2024-01-09: 기존 문서에 GitHub Pages 데모 게시와 주석 정리 기록.
