<!doctype html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <title>노다지 - 노가다 지도 찾아바!</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- 🔑 키/상수들 -->
  <script>
    // VWorld 지도/지오코딩 키
    const VWORLD_KEY = "C51AD54A-04FE-3EB0-A92A-35F719698213";

    // 건설일드림넷 openApi 일반 인증키
    const CID_KEY  = "EFGNWDDDjbLmTpvAR%2BwDrLu9654q8W5AfhHKPp3DlOu3yn%2BtkuPPOlsMI15Yof%2FHpTnI6pCohMBRS%2FFE523TrQ%3D%3D";
    const CID_BASE = "https://www.cid.or.kr/job/openApi/service";

    // 경기도 공동주택 시공현황 API
    const GG_BASE    = "https://openapi.gg.go.kr";
    const GG_SERVICE = "Constwkcotnhousng";
    const GG_KEY     = "598471d431a54597bf2bfcb2e5773d03";

    // 🔥 CORS 우회 프록시 (테스트용)
    const PROXY = "https://cors-anywhere.herokuapp.com/";
  </script>

  <!-- Leaflet -->
  <link rel="stylesheet"
        href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <!-- jQuery (VWorld 검색용) -->
  <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>

  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      display: flex;
      flex-direction: column;
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: #FFF7BF; /* 전체 연노랑 */
      overflow: auto;      /* 🔹스크롤 다시 허용 */
    }

    header {
      padding: 8px 12px;
      background: #FFEB00;
      color: #111827;
      font-size: 13px;
      display: flex;
      flex-direction: column;
      gap: 6px;
      position: relative;
      z-index: 10;
      border-bottom: 2px solid #111827;
    }
    header .top-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 8px;
    }
    .title {
      font-size: 20px;
      font-weight: 800;
      display: flex;
      align-items: center;
      gap: 4px;
    }
    .title-emoji {
      font-size: 22px;
    }
    /* 노다지 쓰러진 서체 느낌 */
    .title-main-slant {
      font-style: italic;
      transform: skew(-8deg);
      display: inline-block;
    }
    .sub {
      font-size: 11px;
      opacity: 0.95;
      font-weight: 600;
    }
    .sub span {
      opacity: 0.8;
    }
    .ad-banner-top {
      margin-top: 4px;
      background: #FFFBEA;
      border-radius: 8px;
      padding: 6px;
      font-size: 11px;
      color: #111827;
      text-align: center;
      border: 1px dashed #111827;
    }

    .user-status {
      text-align: right;
      font-size: 11px;
      opacity: 0.9;
    }

    .hamburger-btn {
      width: 32px;
      height: 32px;
      border-radius: 999px;
      border: 1px solid #111827;
      background: #111827;
      color: #FFEB00;
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      font-size: 18px;
    }
    .hamburger-btn:active {
      transform: scale(0.96);
    }

    .side-menu {
      position: fixed;
      top: 0;
      left: 0;
      width: 260px;
      height: 100%;
      background: #111827;
      color: #FACC15;
      transform: translateX(-100%);
      transition: transform 0.2s ease-out;
      z-index: 9998;
      display: flex;
      flex-direction: column;
    }
    .side-menu.show {
      transform: translateX(0);
    }
    .side-menu-header {
      padding: 12px 14px;
      border-bottom: 1px solid rgba(250,204,21,0.4);
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 13px;
    }
    .side-menu-header-title {
      font-weight: 600;
      font-size: 13px;
    }
    .side-menu-body {
      padding: 10px 14px;
      font-size: 12px;
      flex: 1;
      overflow-y: auto;
    }
    .side-menu-footer {
      padding: 8px 14px;
      border-top: 1px solid rgba(250,204,21,0.4);
      font-size: 11px;
      color: #fef9c3;
    }
    .side-menu-title {
      font-size: 12px;
      font-weight: 600;
      margin-top: 6px;
      margin-bottom: 4px;
      color: #FEF9C3;
    }
    .side-menu-item {
      margin: 4px 0;
      padding: 6px 8px;
      border-radius: 6px;
      background: #020617;
      border: 1px solid rgba(250,204,21,0.6);
      cursor: pointer;
    }
    .side-menu-item.small {
      font-size: 11px;
    }
    .side-menu-item.disabled {
      opacity: 0.4;
      cursor: default;
    }

    .side-menu-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(15,23,42,0.35);
      display: none;
      z-index: 9997;
    }
    .side-menu-backdrop.show {
      display: block;
    }

    main {
      flex: 1;
      display: flex;
      flex-direction: column;
      min-height: 0;
    }

    #map {
      height: 33vh;
      min-height: 220px;
      width: 100%;
      /* 🔹touch-action 제거 → 다시 드래그 가능 */
    }

    #toolbar {
      background: #FFE96A;
      border-bottom: 1px solid #111827;
    }

    #list-wrap {
      flex: 1;
      padding: 8px 10px 60px;
      overflow-y: auto;
      background: #FFF7BF;
    }

    section.block {
      padding: 8px 10px;
      background: #FFE96A;
      border-bottom: 1px solid #111827;
    }
    h2 {
      margin: 0 0 4px;
      font-size: 13px;
      font-weight: 700;
      color: #111827;
    }
    label {
      display: block;
      font-size: 11px;
      margin-top: 4px;
      color: #111827;
      font-weight: 600;
    }
    select,
    input[type="text"],
    textarea {
      width: 100%;
      padding: 7px 8px;
      margin-top: 3px;
      font-size: 12px;
      border-radius: 6px;
      border: 1px solid #111827;
      background: #FFFDF0;
      color: #111827;
    }
    select:focus,
    input:focus,
    textarea:focus {
      outline: none;
      border-color: #111827;
      box-shadow: 0 0 0 1px #111827;
      background: #ffffff;
    }
    textarea {
      resize: vertical;
      min-height: 40px;
      max-height: 80px;
    }
    button {
      width: 100%;
      margin-top: 8px;
      padding: 9px;
      font-size: 13px;
      border-radius: 999px;
      border: 2px solid #111827;
      cursor: pointer;
      font-weight: 700;
      background: #111827;
      color: #FFEB00;
      box-shadow: 0 2px 0 #000;
    }
    button.secondary {
      background: #FFEB00;
      color: #111827;
    }
    button:hover {
      opacity: 0.96;
    }
    button:disabled {
      opacity: 0.4;
      cursor: default;
    }

    .small {
      font-size: 11px;
      color: #3F3F46;
    }
    .row {
      display: flex;
      gap: 6px;
      margin-top: 4px;
    }
    .row > div {
      flex: 1;
    }

    /* 목록 타이틀 바: 검정 배경 + 노랑 글씨 */
    #list-wrap h3 {
      margin: 10px 0 6px;
      font-size: 13px;
      font-weight: 800;
      color: #FFEB00;
      background: #111827;
      padding: 6px 10px;
      border-radius: 999px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .item-card {
      border-radius: 10px;
      border: 2px solid #111827;
      padding: 7px 9px;
      margin-bottom: 8px;
      background: #FFEB00;
      font-size: 12px;
      display: flex;
      flex-direction: column;
      gap: 3px;
      box-shadow: 0 2px 0 #111827;
    }
    .item-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .item-title {
      font-weight: 700;
      font-size: 12px;
      color: #111827;
    }
    .badge {
      font-size: 10px;
      border-radius: 999px;
      padding: 2px 8px;
      background: #111827;
      color: #FFEB00;
      border: 1px solid #FFEB00;
    }
    .item-meta {
      font-size: 11px;
      color: #1F2933;
    }
    .item-actions {
      margin-top: 5px;
      display: flex;
      gap: 6px;
    }
    .item-actions button {
      flex: 1;
      padding: 6px;
      font-size: 11px;
      border-radius: 999px;
    }

    .ad-banner-inline {
      margin: 8px 0;
      border-radius: 10px;
      border: 2px dashed #111827;
      padding: 8px;
      font-size: 11px;
      text-align: center;
      background: #FFFBBA;
      color: #111827;
      box-shadow: 0 2px 0 #111827;
    }

    /* 스플래시 */
    .splash-overlay {
      position: fixed;
      inset: 0;
      background: #FFEB00;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 24px;
      z-index: 10000;
    }
    .splash-title {
      font-size: 26px;
      font-weight: 900;
      margin-bottom: 4px;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
    }
    .splash-emoji {
      font-size: 32px;
    }
    .splash-sub {
      font-size: 13px;
      margin-bottom: 14px;
    }
    .splash-powered {
      font-size: 11px;
      opacity: 0.9;
      margin-top: 6px;
    }
    .splash-button {
      display: inline-flex;
      align-items: center;
      justify-content: center;
      padding: 10px 20px;
      border-radius: 999px;
      border: 2px solid #111827;
      font-size: 14px;
      font-weight: 800;
      background: #111827;
      color: #FFEB00;
      cursor: pointer;
      box-shadow: 0 3px 0 #000;
      margin-top: 8px;
    }
    .splash-button span {
      margin-left: 6px;
      font-size: 16px;
    }

    /* 모달 공통: 노란 카드 스타일 */
    .modal-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.45);
      display: none;
      align-items: center;
      justify-content: center;
      z-index: 9999;
    }
    .modal-backdrop.show {
      display: flex;
    }

    /* 알림 모달이 맨 위로 */
    #notifyModalBackdrop {
      z-index: 10001;
    }

    .modal {
      background: #FFEB00;
      border-radius: 14px;
      max-width: 420px;
      width: calc(100% - 32px);
      padding: 12px;
      box-shadow: 0 10px 0 #111827;
      border: 2px solid #111827;
      color: #111827;
    }
    .modal h3 {
      margin: 0 0 6px;
      font-size: 14px;
      font-weight: 800;
    }
    .modal-body {
      max-height: 380px;
      overflow-y: auto;
      font-size: 12px;
    }
    .modal-footer {
      margin-top: 8px;
      display: flex;
      gap: 6px;
    }
    .modal-footer button {
      flex: 1;
      padding: 7px;
      font-size: 12px;
    }

    .match-item {
      border-radius: 8px;
      border: 2px solid #111827;
      padding: 6px;
      margin-bottom: 6px;
      font-size: 12px;
      background: #FFFDE7;
    }
    .match-item-title {
      font-weight: 700;
      font-size: 12px;
      color: #111827;
    }
    .match-item-meta {
      font-size: 11px;
      color: #374151;
    }

    .map-icon {
      font-size: 16px;
      line-height: 1;
      text-align: center;
      transform: translateY(-4px);
    }
    .map-icon.job {
      color: #ea580c;
      text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }
    .map-icon.seeker {
      color: #2563eb;
      text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }
    .map-icon.site {
      color: #16a34a;
      text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }
    .map-icon.cid {
      color: #7c3aed;
      text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }
    .map-icon.apt {
      color: #0d9488;
      text-shadow: 0 0 4px rgba(255,255,255,0.9);
    }

    @media (max-width: 768px) {
      header { padding: 8px 10px; }
      #map { height: 33vh; }
    }
  </style>
</head>
<body>

<!-- 스플래시 -->
<div id="splash" class="splash-overlay">
  <div class="splash-emoji">👷‍♂️😏</div>
  <div class="splash-title">
    <span class="title-main-slant">노다지</span>
  </div>
  <div class="splash-sub">
    노가다 지도 찾아바!
  </div>
  <button id="splashBtn" class="splash-button">
    현장 어디있노? <span>📍</span>
  </button>
  <div class="splash-powered">
    powered by <b>NODAJI</b>
  </div>
</div>

<header>
  <div class="top-row">
    <div>
      <div class="title">
        <span class="title-emoji">👷‍♂️</span>
        <span class="title-main-slant">노다지</span>
      </div>
      <div class="sub">
        노가다 지도 찾아바! · <span>powered by NODAJI</span>
      </div>
    </div>
    <div style="display:flex; flex-direction:column; align-items:flex-end; gap:4px;">
      <div class="user-status" id="userStatus">👤 게스트 (읽기 전용)</div>
      <button id="hamburgerBtn" class="hamburger-btn">☰</button>
    </div>
  </div>
  <div class="ad-banner-top">
    👷‍♂️ 여기에는 <b>작업복·안전화·장갑</b> 같은 <b>쿠팡파트너스 상단 배너</b>를 넣으시면 좋습니다.
  </div>
</header>

<!-- 햄버거 메뉴 -->
<div id="sideMenu" class="side-menu">
  <div class="side-menu-header">
    <span class="side-menu-header-title">노다지 · 메뉴</span>
    <span id="sideMenuClose" style="cursor:pointer;">✕</span>
  </div>
  <div class="side-menu-body">
    <div class="side-menu-title">내 계정</div>
    <div id="menuUserInfo" class="small" style="margin-bottom:4px;">로그인되지 않음</div>

    <div id="menuLoginBtn" class="side-menu-item">
      🔐 회원가입 / 로그인
      <div class="small">구인·구직 등록은 가입자만 가능합니다.</div>
    </div>

    <div id="menuLogoutBtn" class="side-menu-item small disabled">
      🚪 로그아웃
    </div>

    <div class="side-menu-title">바로가기</div>
    <div id="menuGoJob" class="side-menu-item small">
      🛠 구인 등록 창 열기
    </div>
    <div id="menuGoSeek" class="side-menu-item small">
      🙋‍♂️ 구직 등록 창 열기
    </div>
    <div id="menuFav" class="side-menu-item small">
      ⭐ 즐겨찾기 공고 (준비 중)
    </div>
    <div id="menuEditDelete" class="side-menu-item small">
      ✏️ 공고 수정·삭제 (준비 중)
    </div>

    <div class="side-menu-title">기능</div>
    <div class="side-menu-item small">
      📍 지도 기준 현장 보기<br />
      <span class="small">지도를 드래그하면 화면에 보이는 영역 안의 현장/구인/구직/공동주택이 목록에 표시됩니다.</span>
    </div>
    <div class="side-menu-item small">
      🟣 건설일드림넷 공고<br/>
      <span class="small">보라색 마커는 건설일드림넷 공식 구인공고입니다.</span>
    </div>
    <div class="side-menu-item small">
      🏢 경기도 공동주택 시공현황<br/>
      <span class="small">청록색(🏢) 마커는 경기도 공동주택 시공현장입니다.</span>
    </div>
    <div class="side-menu-item small">
      💬 1:1 메시지 (준비 중)
    </div>

    <div class="side-menu-title">안내 / 정책</div>
    <div id="menuHowTo" class="side-menu-item small">
      ℹ️ 이용 방법
    </div>
    <div id="menuPrivacy" class="side-menu-item small">
      🔒 개인정보 처리방침<br />
      <span class="small">관리자: 김태현 · 이메일: th3788@gmail.com</span>
    </div>
  </div>
  <div class="side-menu-footer">
    © 노다지 (현장구인지도) · 관리자 김태현 · th3788@gmail.com<br />
    v0.5 · LocalStorage 임시 로그인 · 서버/DB 없음
  </div>
</div>
<div id="sideMenuBackdrop" class="side-menu-backdrop"></div>

<main>
  <div id="map"></div>

  <section id="toolbar" class="block">
    <h2>지도 기준 현장 보기</h2>

    <label>전국 현장 / 주소 / 동·리 / 아파트명 검색
      <div class="row">
        <div><input id="searchQuery" type="text" placeholder="예) 파주시 서패동, 운정동, ○○아파트" /></div>
        <div style="flex:0.7;"><button id="btnSearch">검색</button></div>
      </div>
    </label>
    <div class="small" id="searchInfo">VWorld 검색 API로 전국 이동</div>

    <div class="row" style="margin-top:6px;">
      <div style="flex:1.4;">
        <label>내 위치
          <div class="small" id="locationStatus">📍 내 위치 확인 중...</div>
        </label>
      </div>
      <div style="flex:0.6; align-self:flex-end;">
        <button id="btnGoMyLocation" class="secondary" style="margin-top:0; padding:6px 4px; font-size:11px;">
          현위치
        </button>
      </div>
    </div>

    <label style="margin-top:6px;">업종 필터</label>
    <div class="row">
      <div style="flex:1.2;">
        <select id="typeFilter">
          <option value="ALL">전체</option>
          <option>전기</option><option>설비</option><option>통신</option>
          <option>소방</option><option>에어컨</option><option>철근</option>
          <option>목수</option><option>아시바</option><option>낙방</option>
          <option>조적</option><option>견출</option><option>미장</option>
          <option>샷시</option><option>내장</option><option>직영</option><option>잡부</option>
          <option>신호수</option><option>도배</option><option>마루</option>
          <option>가구</option><option>석공</option><option>타일</option><option>환기</option>
        </select>
      </div>
      <div style="flex:1;">
        <div class="small">선택 업종만 구인·구직 표시</div>
      </div>
    </div>

    <label style="margin-top:6px;">등록</label>
    <div class="row">
      <div>
        <button id="openSiteModalBtn" class="secondary" style="margin-top:4px; font-size:12px; padding:7px;">
          🏗 현장 등록
        </button>
      </div>
      <div>
        <button id="openJobModalBtn" style="margin-top:4px; font-size:12px; padding:7px;">
          🛠 구인 등록
        </button>
      </div>
      <div>
        <button id="openSeekModalBtn" style="margin-top:4px; font-size:12px; padding:7px;">
          🙋‍♂️ 구직 등록
        </button>
      </div>
    </div>

    <div class="ad-banner-inline">
      💰 여기는 <b>쿠팡파트너스 광고</b> 배너로 쓰면 좋습니다.<br />
      (작업복, 안전화, 장갑, 공구 링크 추천)
    </div>
  </section>

  <div id="list-wrap">
    <h3>
      현장 (지도 화면 기준)
      <span id="siteCnt" class="small"></span>
    </h3>
    <div id="siteList" class="small">등록된 현장이 없습니다.</div>

    <h3>
      현장 구인 (지도 화면 기준)
      <span id="jobCnt" class="small"></span>
    </h3>
    <div id="jobList" class="small">등록된 현장이 없습니다.</div>

    <h3 style="margin-top:10px;">
      구직자 (지도 화면 기준)
      <span id="seekCnt" class="small"></span>
    </h3>
    <div id="seekList" class="small">등록된 구직자가 없습니다.</div>

    <!-- 경기도 공동주택 시공현황 -->
    <h3 style="margin-top:10px;">
      🏢 경기도 공동주택 시공현황 (지도 기준)
      <span id="aptCnt" class="small"></span>
    </h3>
    <div id="aptList" class="small">
      경기도 공동주택 시공현황 데이터를 불러오는 중입니다...
    </div>

    <div class="ad-banner-inline">
      📦 여기에도 <b>중간 쿠팡파트너스 배너</b>를 넣을 수 있습니다.<br />
      (예: "미장 장갑 베스트 3", "전기공 안전화")
    </div>

    <div class="item-card" style="margin-top:10px;">
      <div class="item-header">
        <div class="item-title">📰 건설 뉴스 / 정보 보기</div>
        <span class="badge">링크 모음</span>
      </div>
      <div class="item-meta">
        자주 보는 건설 관련 사이트를 모아두면 좋습니다. (링크는 예시, 나중에 원하는 걸로 교체)
      </div>
      <ul style="margin:6px 0 0; padding-left:18px; font-size:11px; color:#111827;">
        <li><a href="https://www.molit.go.kr" target="_blank" rel="noopener noreferrer">국토교통부 정책·뉴스</a></li>
        <li><a href="https://www.cnews.co.kr" target="_blank" rel="noopener noreferrer">건설경제신문</a></li>
        <li><a href="https://www.koscaj.com" target="_blank" rel="noopener noreferrer">대한전문건설협회</a></li>
        <li><a href="https://www.cid.or.kr" target="_blank" rel="noopener noreferrer">건설일드림넷 공식사이트</a></li>
      </ul>
    </div>
  </div>
</main>

<input type="hidden" id="lat" />
<input type="hidden" id="lng" />

<!-- 알림 모달 (확인창) -->
<div id="notifyModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3 id="notifyTitle">알림</h3>
    <div class="modal-body">
      <p id="notifyMessage" class="small" style="white-space:pre-line;"></p>
    </div>
    <div class="modal-footer">
      <button id="notifyCancelBtn" class="secondary" style="display:none;">취소</button>
      <button id="notifyOkBtn">확인</button>
    </div>
  </div>
</div>

<!-- 로그인/닉네임 모달 -->
<div id="authModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>로그인 / 닉네임 설정</h3>
    <div class="modal-body">
      <p id="authNotice" class="small" style="white-space:pre-line;">
구인·구직 등록을 위해 닉네임을 입력해 주세요.
(브라우저에만 저장되며, 서버에는 저장되지 않습니다.)
      </p>
      <label>닉네임
        <input type="text" id="authNickname" placeholder="예) 노다지전기, 서패동미장팀" />
      </label>
    </div>
    <div class="modal-footer">
      <button id="authCancelBtn" class="secondary">취소</button>
      <button id="authSubmitBtn">확인</button>
    </div>
  </div>
</div>

<!-- 현장 등록 모달 -->
<div id="siteModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>🏗 현장 등록</h3>
    <div class="modal-body">
      <p class="small">
1) 지도에서 현장 위치를 먼저 클릭해 주세요.<br>
2) 아래에 현장 이름과 메모를 작성합니다.
      </p>

      <div class="ad-banner-inline">
        🏗 <b>현장 자재/공구</b> 쿠팡 링크 배치 추천 구역<br/>
        (예: 해머드릴, 줄자, 레이저레벨 등)
      </div>

      <div id="clickInfo" class="small" style="margin-bottom:4px; color:#16a34a;">
지도에서 위치를 클릭하면 좌표가 표시됩니다.
      </div>
      <label>현장 이름
        <input type="text" id="siteName" placeholder="예) 파주 서패동 ○○아파트 신축" />
      </label>
      <label>메모
        <textarea id="siteMemo" placeholder="예) 전기 마감 중, 미장 투입 예정 등"></textarea>
      </label>
    </div>
    <div class="modal-footer">
      <button id="siteCancelBtn" class="secondary">닫기</button>
      <button id="siteSubmitBtn">등록</button>
    </div>
  </div>
</div>

<!-- 구인 등록 모달 -->
<div id="jobModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>🛠 현장 구인 등록</h3>
    <div class="modal-body">
      <p class="small">
1) 지도에서 현장 위치를 클릭해 주세요.<br>
2) 업종, 숙련도, 팀/개인, 한 줄 메모를 적어주세요.
      </p>

      <div class="ad-banner-inline">
        👷‍♂️ <b>현장 작업복·안전화·장갑</b> 쿠팡 배너<br/>
        (구인 등록할 때 자연스럽게 눈에 들어오게)
      </div>

      <label>업종
        <select id="jobType">
          <option value="">선택</option>
          <option>전기</option><option>설비</option><option>통신</option>
          <option>소방</option><option>에어컨</option><option>철근</option>
          <option>목수</option><option>아시바</option><option>낙방</option>
          <option>조적</option><option>견출</option><option>미장</option>
          <option>샷시</option><option>내장</option><option>직영</option><option>잡부</option>
          <option>신호수</option><option>도배</option><option>마루</option>
          <option>가구</option><option>석공</option><option>타일</option><option>환기</option>
        </select>
      </label>
      <label>숙련도
        <select id="jobLevel">
          <option value="">선택</option>
          <option>기능공</option>
          <option>조공</option>
          <option>초보</option>
        </select>
      </label>
      <label>형태 (팀 / 개인)
        <select id="jobTeamType">
          <option value="">선택</option>
          <option>팀</option>
          <option>개인</option>
        </select>
      </label>
      <label>한 줄 메모
        <textarea id="jobMemo" placeholder="예) 전기 기능공 2명, 하루 인건비, 기간 등"></textarea>
      </label>
    </div>
    <div class="modal-footer">
      <button id="jobCancelBtn" class="secondary">닫기</button>
      <button id="jobSubmitBtn">등록</button>
    </div>
  </div>
</div>

<!-- 구직 등록 모달 -->
<div id="seekModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>🙋‍♂️ 구직 등록</h3>
    <div class="modal-body">
      <p class="small">
1) 지도에서 본인이 일하고 싶은 위치를 클릭해 주세요.<br>
2) 업종, 숙련도, 팀/개인, 한 줄 소개를 적어주세요.
      </p>

      <div class="ad-banner-inline">
        🙋‍♂️ <b>개인 작업복·무릎보호대·장갑</b> 쿠팡 광고<br/>
        (구직자용 장비/의류 노출 구역)
      </div>

      <label>희망 업종
        <select id="seekType">
          <option value="">선택</option>
          <option>전기</option><option>설비</option><option>통신</option>
          <option>소방</option><option>에어컨</option><option>철근</option>
          <option>목수</option><option>아시바</option><option>낙방</option>
          <option>조적</option><option>견출</option><option>미장</option>
          <option>샷시</option><option>내장</option><option>직영</option><option>잡부</option>
          <option>신호수</option><option>도배</option><option>마루</option>
          <option>가구</option><option>석공</option><option>타일</option><option>환기</option>
        </select>
      </label>
      <label>숙련도
        <select id="seekLevel">
          <option value="">선택</option>
          <option>기능공</option>
          <option>조공</option>
          <option>초보</option>
        </select>
      </label>
      <label>형태 (팀 / 개인)
        <select id="seekTeamType">
          <option value="">선택</option>
          <option>팀</option>
          <option>개인</option>
        </select>
      </label>
      <label>한 줄 소개
        <textarea id="seekMemo" placeholder="예) 파주·일산 인근 전기 조공, 일 배우고 싶습니다 등"></textarea>
      </label>
    </div>
    <div class="modal-footer">
      <button id="seekCancelBtn" class="secondary">닫기</button>
      <button id="seekSubmitBtn">등록</button>
    </div>
  </div>
</div>

<!-- 매칭 모달 -->
<div id="matchModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>근처 구직자 보기</h3>
    <div class="modal-body" id="matchBody">
      <p class="small">주변 구직자를 검색 중입니다...</p>
    </div>
    <div class="modal-footer">
      <button id="matchCloseBtn">닫기</button>
    </div>
  </div>
</div>

<!-- 채팅 안내 모달 -->
<div id="chatModalBackdrop" class="modal-backdrop">
  <div class="modal">
    <h3>1:1 메시지 (준비 중)</h3>
    <div class="modal-body">
      <p class="small">
현재 버전은 실제 채팅 기능이 붙어있지 않습니다.<br>
추후 정식 서비스에서는 번호·카톡 공개 없이<br>
앱 내부에서만 대화 가능한 1:1 메시지 기능을 붙일 예정입니다.
      </p>
    </div>
    <div class="modal-footer">
      <button id="chatCloseBtn">닫기</button>
    </div>
  </div>
</div>

<script>
  let map;
  let baseLayer;
  let tempMarker = null;
  let searchMarker = null;
  let userLoc = null;
  let userMarker = null;

  let sites = [];
  let jobs = [];
  let seekers = [];
  let aptSites = [];

  const STORAGE_SITES   = "vw_sites_v1";
  const STORAGE_JOBS    = "vw_jobs_v3";
  const STORAGE_SEEKER  = "vw_seekers_v3";
  const STORAGE_USER    = "jobmap_user_v1";
  const STORAGE_SPLASH  = "nodaji_splash_hidden_v1";

  let currentUser = null;
  let markerOffsetRegistry = {};

  let notifyMode = "alert";
  let notifyConfirmCallback = null;

  function showAlert(message, title = "알림") {
    const backdrop = document.getElementById("notifyModalBackdrop");
    document.getElementById("notifyTitle").textContent = title;
    document.getElementById("notifyMessage").textContent = message;
    document.getElementById("notifyCancelBtn").style.display = "none";
    notifyMode = "alert";
    notifyConfirmCallback = null;
    backdrop.classList.add("show");
  }
  function showConfirm(title, message, onConfirm) {
    const backdrop = document.getElementById("notifyModalBackdrop");
    document.getElementById("notifyTitle").textContent = title || "확인";
    document.getElementById("notifyMessage").textContent = message || "";
    document.getElementById("notifyCancelBtn").style.display = "inline-block";
    notifyMode = "confirm";
    notifyConfirmCallback = function(confirmed) {
      if (confirmed && typeof onConfirm === "function") onConfirm();
    };
    backdrop.classList.add("show");
  }
  function closeNotifyModal() {
    document.getElementById("notifyModalBackdrop").classList.remove("show");
  }

  function distM(lat1, lon1, lat2, lon2) {
    const R = 6371000;
    const toRad = d => d * Math.PI / 180;
    const dLat = toRad(lat2 - lat1);
    const dLon = toRad(lon2 - lon1);
    const a =
      Math.sin(dLat/2) * Math.sin(dLat/2) +
      Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) *
      Math.sin(dLon/2) * Math.sin(dLon/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    return R * c;
  }

  function getTypeEmoji(type) {
    switch (type) {
      case "전기": return "⚡";
      case "설비": return "🔧";
      case "통신": return "📡";
      case "소방": return "🔥";
      case "에어컨": return "❄️";
      case "철근": return "🏗️";
      case "목수": return "🪚";
      case "아시바": return "🧗";
      case "낙방": return "🧱";
      case "조적": return "🧱";
      case "견출": return "🧱";
      case "미장": return "🎨";
      case "샷시": return "🚪";
      case "내장": return "🏠";
      case "직영": return "📋";
      case "잡부": return "🙋";
      case "신호수": return "🚦";
      case "도배": return "🖌️";
      case "마루": return "🪵";
      case "가구": return "🪑";
      case "석공": return "🪨";
      case "타일": return "🧱";
      case "환기": return "🌬️";
      default: return "🛠️";
    }
  }

  function createDivIcon(type, role) {
    let emoji;
    let cls = role;
    if (role === "site") emoji = "🏗️";
    else if (role === "cid") emoji = "🟣";
    else if (role === "apt") emoji = "🏢";
    else emoji = getTypeEmoji(type);
    return L.divIcon({
      className: "map-icon " + cls,
      html: emoji,
      iconSize: [24, 24],
      iconAnchor: [12, 24],
    });
  }

  function getOffsetLatLng(lat, lng) {
    const key = lat.toFixed(6) + ":" + lng.toFixed(6);
    const currentCount = markerOffsetRegistry[key] || 0;
    markerOffsetRegistry[key] = currentCount + 1;
    if (currentCount === 0) return { lat, lng };

    const radiusM = 10;
    const angleDeg = (currentCount - 1) * 45;
    const angle = angleDeg * Math.PI / 180;

    const dLat = (radiusM * Math.cos(angle)) / 111320;
    const dLng = (radiusM * Math.sin(angle)) / (111320 * Math.cos(lat * Math.PI / 180));

    return { lat: lat + dLat, lng: lng + dLng };
  }

  function getOrigin() {
    if (userLoc) return { lat: userLoc.lat, lng: userLoc.lng };
    const c = map.getCenter();
    return { lat: c.lat, lng: c.lng };
  }

  function isInView(lat, lng) {
    if (!map) return true;
    const bounds = map.getBounds();
    return bounds.contains([lat, lng]);
  }

  function loadUser() {
    const raw = localStorage.getItem(STORAGE_USER);
    if (!raw) currentUser = null;
    else {
      try { currentUser = JSON.parse(raw); }
      catch { currentUser = null; }
    }
    updateUserUI();
  }
  function saveUser() {
    if (currentUser) localStorage.setItem(STORAGE_USER, JSON.stringify(currentUser));
    else localStorage.removeItem(STORAGE_USER);
    updateUserUI();
  }
  function updateUserUI() {
    const userStatus = document.getElementById("userStatus");
    const menuUserInfo = document.getElementById("menuUserInfo");
    const menuLogoutBtn = document.getElementById("menuLogoutBtn");
    if (currentUser && currentUser.nickname) {
      userStatus.textContent = "👤 " + currentUser.nickname + "님 (등록 가능)";
      menuUserInfo.textContent = "현재 로그인: " + currentUser.nickname;
      menuLogoutBtn.classList.remove("disabled");
    } else {
      userStatus.textContent = "👤 게스트 (읽기 전용)";
      menuUserInfo.textContent = "로그인되지 않았습니다.";
      menuLogoutBtn.classList.add("disabled");
    }
  }
  function requireLogin() {
    if (currentUser && currentUser.nickname) return true;
    openAuthModal("구인·구직 등록을 위해 로그인이 필요합니다.");
    return false;
  }

  function initMap() {
    baseLayer = L.tileLayer(
      "https://api.vworld.kr/req/wmts/1.0.0/" + VWORLD_KEY + "/Base/{z}/{y}/{x}.png",
      { maxZoom: 19 }
    );
    map = L.map("map", {
      center: [37.5665, 126.9780],
      zoom: 13,
      layers: [baseLayer],
    });

    map.on("click", function(e) {
      const lat = e.latlng.lat;
      const lng = e.latlng.lng;
      document.getElementById("lat").value = lat;
      document.getElementById("lng").value = lng;
      const info = document.getElementById("clickInfo");
      if (info) {
        info.textContent = "📍 선택된 위치: " + lat.toFixed(6) + ", " + lng.toFixed(6);
      }
      if (tempMarker) map.removeLayer(tempMarker);
      tempMarker = L.marker([lat, lng]).addTo(map);
    });

    map.on("moveend", function() {
      renderSites();
      renderJobs();
      renderSeekers();
      renderAptList();
    });

    initGeolocation();
  }

  function initGeolocation() {
    const statusEl = document.getElementById("locationStatus");
    if (!navigator.geolocation) {
      statusEl.textContent = "위치 정보 미지원";
      return;
    }
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        const lat = pos.coords.latitude;
        const lng = pos.coords.longitude;
        userLoc = { lat, lng };
        statusEl.textContent = "📍 내 위치: " + lat.toFixed(4) + ", " + lng.toFixed(4);
        if (userMarker) map.removeLayer(userMarker);
        userMarker = L.marker([lat, lng], { title: "내 위치" }).addTo(map);
        map.setView([lat, lng], 13);
      },
      () => {
        statusEl.textContent = "내 위치 가져오기 실패 (IP 기준일 수 있음)";
      }
    );
  }
  function goMyLocation() {
    if (userLoc) {
      map.setView([userLoc.lat, userLoc.lng], 13);
      if (userMarker && userMarker.openPopup) userMarker.openPopup();
    } else {
      initGeolocation();
      showAlert("내 위치 정보를 다시 가져옵니다.\n브라우저 위치 권한을 확인해주세요.", "현위치");
    }
  }

  function closeSplash() {
    const s = document.getElementById("splash");
    if (s) {
      s.style.display = "none";
      localStorage.setItem(STORAGE_SPLASH, "1");
    }
    goMyLocation();
  }
  function maybeHideSplashImmediately() {
    const saved = localStorage.getItem(STORAGE_SPLASH);
    if (saved === "1") {
      const s = document.getElementById("splash");
      if (s) s.style.display = "none";
    }
  }

  function openSiteModal() { document.getElementById("siteModalBackdrop").classList.add("show"); }
  function closeSiteModal() { document.getElementById("siteModalBackdrop").classList.remove("show"); }
  function openJobModal() { document.getElementById("jobModalBackdrop").classList.add("show"); }
  function closeJobModal() { document.getElementById("jobModalBackdrop").classList.remove("show"); }
  function openSeekModal() { document.getElementById("seekModalBackdrop").classList.add("show"); }
  function closeSeekModal() { document.getElementById("seekModalBackdrop").classList.remove("show"); }

  function addSite() {
    const name = document.getElementById("siteName").value.trim();
    const memo = document.getElementById("siteMemo").value.trim();
    const lat = parseFloat(document.getElementById("lat").value);
    const lng = parseFloat(document.getElementById("lng").value);
    if (!name) { showAlert("현장 이름을 입력해주세요.", "현장 등록"); return; }
    if (isNaN(lat) || isNaN(lng)) {
      showAlert("먼저 지도에서 현장 위치를 클릭해주세요.", "현장 등록");
      return;
    }
    const summary =
      "현장 이름: " + name +
      "\n위치: " + lat.toFixed(6) + ", " + lng.toFixed(6) +
      (memo ? "\n메모: " + memo : "");
    showConfirm("현장 등록 확인", summary, () => {
      const site = { id: Date.now(), name, memo, lat, lng, marker: null };
      const pos = getOffsetLatLng(lat, lng);
      site.marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(null, "site") }).addTo(map);
      site.marker.bindPopup("🏗️ <b>현장</b><br>" + name + (memo ? "<br>" + memo : ""));
      sites.push(site);
      saveSites();
      renderSites();
      closeSiteModal();
      showAlert("현장 정보가 등록되었습니다.\n아래 '현장' 목록에서 확인할 수 있습니다.", "현장 등록 완료");
      const list = document.getElementById("siteList");
      if (list) setTimeout(() => list.scrollIntoView({ behavior: "smooth" }), 150);
    });
  }

  function addJob() {
    if (!requireLogin()) return;
    const type = document.getElementById("jobType").value;
    const level = document.getElementById("jobLevel").value;
    const teamType = document.getElementById("jobTeamType").value;
    const memo = document.getElementById("jobMemo").value.trim();
    const lat = parseFloat(document.getElementById("lat").value);
    const lng = parseFloat(document.getElementById("lng").value);
    if (!type) { showAlert("업종을 선택해주세요.", "구인 등록"); return; }
    if (!level) { showAlert("숙련도를 선택해주세요.", "구인 등록"); return; }
    if (!teamType) { showAlert("형태를 선택해주세요.", "구인 등록"); return; }
    if (isNaN(lat) || isNaN(lng)) {
      showAlert("먼저 지도에서 현장 위치를 클릭해주세요.", "구인 등록");
      return;
    }
    const summary =
      "업종: " + type +
      "\n숙련도: " + level +
      "\n형태: " + teamType +
      (memo ? "\n메모: " + memo : "") +
      "\n위치: " + lat.toFixed(6) + ", " + lng.toFixed(6);
    showConfirm("구인 등록 확인", summary, () => {
      const job = {
        id: Date.now(),
        type, level, teamType, memo, lat, lng,
        marker: null,
        owner: currentUser ? currentUser.nickname : null
      };
      const pos = getOffsetLatLng(lat, lng);
      job.marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(type, "job") }).addTo(map);
      job.marker.bindPopup(
        "🛠 <b>현장 구인</b><br>[" + type + " · " + level + " · " + teamType + "]" +
        (memo ? "<br>" + memo : "") +
        (job.owner ? "<br><span style='font-size:11px;'>등록자: " + job.owner + "</span>" : "")
      );
      jobs.push(job);
      saveJobs();
      renderJobs();
      closeJobModal();
      showAlert("현장 구인이 등록되었습니다.\n아래 '현장 구인' 목록에서 확인할 수 있습니다.", "구인 등록 완료");
      const list = document.getElementById("jobList");
      if (list) setTimeout(() => list.scrollIntoView({ behavior: "smooth" }), 150);
    });
  }

  function addSeeker() {
    if (!requireLogin()) return;
    const type = document.getElementById("seekType").value;
    const level = document.getElementById("seekLevel").value;
    const teamType = document.getElementById("seekTeamType").value;
    const memo = document.getElementById("seekMemo").value.trim();
    const lat = parseFloat(document.getElementById("lat").value);
    const lng = parseFloat(document.getElementById("lng").value);
    if (!type) { showAlert("희망 업종을 선택해주세요.", "구직 등록"); return; }
    if (!level) { showAlert("숙련도를 선택해주세요.", "구직 등록"); return; }
    if (!teamType) { showAlert("형태를 선택해주세요.", "구직 등록"); return; }
    if (isNaN(lat) || isNaN(lng)) {
      showAlert("먼저 지도에서 내 활동 위치를 클릭해주세요.", "구직 등록");
      return;
    }
    const summary =
      "희망 업종: " + type +
      "\n숙련도: " + level +
      "\n형태: " + teamType +
      (memo ? "\n한 줄 소개: " + memo : "") +
      "\n위치: " + lat.toFixed(6) + ", " + lng.toFixed(6);
    showConfirm("구직 등록 확인", summary, () => {
      const seeker = {
        id: Date.now(),
        type, level, teamType, memo, lat, lng,
        marker: null,
        owner: currentUser ? currentUser.nickname : null
      };
      const pos = getOffsetLatLng(lat, lng);
      seeker.marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(type, "seeker") }).addTo(map);
      seeker.marker.bindPopup(
        "🙋‍♂️ <b>구직자</b><br>[" + type + " · " + level + " · " + teamType + "]" +
        (memo ? "<br>" + memo : "") +
        (seeker.owner ? "<br><span style='font-size:11px;'>등록자: " + seeker.owner + "</span>" : "")
      );
      seekers.push(seeker);
      saveSeekers();
      renderSeekers();
      closeSeekModal();
      showAlert("구직자 정보가 등록되었습니다.\n아래 '구직자' 목록에서 확인할 수 있습니다.", "구직 등록 완료");
      const list = document.getElementById("seekList");
      if (list) setTimeout(() => list.scrollIntoView({ behavior: "smooth" }), 150);
    });
  }

  function saveSites() {
    const plain = sites.map(s => ({
      id: s.id, name: s.name, memo: s.memo, lat: s.lat, lng: s.lng
    }));
    localStorage.setItem(STORAGE_SITES, JSON.stringify(plain));
  }
  function loadSites() {
    const raw = localStorage.getItem(STORAGE_SITES);
    if (!raw) return;
    try {
      JSON.parse(raw).forEach(data => {
        const pos = getOffsetLatLng(data.lat, data.lng);
        const marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(null, "site") }).addTo(map);
        marker.bindPopup("🏗️ <b>현장</b><br>" + data.name + (data.memo ? "<br>" + data.memo : ""));
        sites.push({ ...data, marker });
      });
    } catch (e) { console.error("현장 데이터 오류", e); }
  }

  function saveJobs() {
    const plain = jobs.map(j => ({
      id: j.id, type: j.type, level: j.level,
      teamType: j.teamType, memo: j.memo,
      lat: j.lat, lng: j.lng,
      owner: j.owner || null
    }));
    localStorage.setItem(STORAGE_JOBS, JSON.stringify(plain));
  }
  function loadJobs() {
    const raw = localStorage.getItem(STORAGE_JOBS);
    if (!raw) return;
    try {
      JSON.parse(raw).forEach(data => {
        const pos = getOffsetLatLng(data.lat, data.lng);
        const marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(data.type, "job") }).addTo(map);
        marker.bindPopup(
          "🛠 <b>현장 구인</b><br>[" + data.type + " · " + data.level + " · " + data.teamType + "]" +
          (data.memo ? "<br>" + data.memo : "") +
          (data.owner ? "<br><span style='font-size:11px;'>등록자: " + data.owner + "</span>" : "")
        );
        jobs.push({ ...data, marker });
      });
    } catch (e) { console.error("구인 데이터 오류", e); }
  }

  function saveSeekers() {
    const plain = seekers.map(s => ({
      id: s.id, type: s.type, level: s.level,
      teamType: s.teamType, memo: s.memo,
      lat: s.lat, lng: s.lng,
      owner: s.owner || null
    }));
    localStorage.setItem(STORAGE_SEEKER, JSON.stringify(plain));
  }
  function loadSeekers() {
    const raw = localStorage.getItem(STORAGE_SEEKER);
    if (!raw) return;
    try {
      JSON.parse(raw).forEach(data => {
        const pos = getOffsetLatLng(data.lat, data.lng);
        const marker = L.marker([pos.lat, pos.lng], { icon: createDivIcon(data.type, "seeker") }).addTo(map);
        marker.bindPopup(
          "🙋‍♂️ <b>구직자</b><br>[" + data.type + " · " + data.level + " · " + data.teamType + "]" +
          (data.memo ? "<br>" + data.memo : "") +
          (data.owner ? "<br><span style='font-size:11px;'>등록자: " + data.owner + "</span>" : "")
        );
        seekers.push({ ...data, marker });
      });
    } catch (e) { console.error("구직 데이터 오류", e); }
  }

  function renderSites() {
    const listEl = document.getElementById("siteList");
    const cntEl = document.getElementById("siteCnt");
    sites.forEach(s => {
      if (!s.marker) return;
      const inV = isInView(s.lat, s.lng);
      s.marker.setOpacity(inV ? 1.0 : 0.2);
    });
    if (sites.length === 0) {
      listEl.textContent = "등록된 현장이 없습니다.";
      cntEl.textContent = "";
      return;
    }
    listEl.innerHTML = "";
    let visibleCount = 0;
    sites.slice().sort((a,b) => b.id - a.id).forEach(site => {
      if (!isInView(site.lat, site.lng)) return;
      visibleCount++;
      const card = document.createElement("div");
      card.className = "item-card";
      const header = document.createElement("div");
      header.className = "item-header";
      header.innerHTML =
        '<div class="item-title">🏗️ ' + site.name + "</div>" +
        '<span class="badge">현장</span>';
      const meta = document.createElement("div");
      meta.className = "item-meta";
      const origin = getOrigin();
      const dKm = distM(origin.lat, origin.lng, site.lat, site.lng) / 1000;
      const dText = dKm < 1 ? (dKm * 1000).toFixed(0) + " m" : dKm.toFixed(1) + " km";
      meta.textContent =
        "위치: " + site.lat.toFixed(5) + ", " + site.lng.toFixed(5) +
        " · 기준점에서 " + dText;
      const memo = document.createElement("div");
      memo.className = "item-meta";
      memo.textContent = site.memo || "메모 없음";
      const actions = document.createElement("div");
      actions.className = "item-actions";
      const btnMap = document.createElement("button");
      btnMap.className = "secondary";
      btnMap.textContent = "지도에서 보기";
      btnMap.addEventListener("click", () => {
        map.setView([site.lat, site.lng], 16);
        if (site.marker) site.marker.openPopup();
      });
      actions.appendChild(btnMap);
      card.appendChild(header);
      card.appendChild(meta);
      card.appendChild(memo);
      card.appendChild(actions);
      listEl.appendChild(card);
    });
    cntEl.textContent = "(" + sites.length + "개 / 화면 내 " + visibleCount + "개)";
  }

  function getTypeFilterValue() {
    const sel = document.getElementById("typeFilter");
    if (!sel) return "ALL";
    return sel.value || "ALL";
  }

  function renderJobs() {
    const listEl = document.getElementById("jobList");
    const cntEl = document.getElementById("jobCnt");
    const typeFilter = getTypeFilterValue();
    jobs.forEach(j => {
      if (!j.marker) return;
      const inV = isInView(j.lat, j.lng);
      const typeOk = (typeFilter === "ALL" || j.type === typeFilter);
      j.marker.setOpacity(inV && typeOk ? 1.0 : 0.0);
    });
    if (jobs.length === 0) {
      listEl.textContent = "등록된 현장이 없습니다.";
      cntEl.textContent = "";
      return;
    }
    listEl.innerHTML = "";
    let visibleCount = 0;
    jobs.slice().sort((a,b) => b.id - a.id).forEach(job => {
      if (!isInView(job.lat, job.lng)) return;
      if (typeFilter !== "ALL" && job.type !== typeFilter) return;
      visibleCount++;
      const card = document.createElement("div");
      card.className = "item-card";
      const header = document.createElement("div");
      header.className = "item-header";
      header.innerHTML =
        '<div class="item-title">' + getTypeEmoji(job.type) + " [" + job.type + "]</div>" +
        '<span class="badge">구인</span>';
      const meta = document.createElement("div");
      meta.className = "item-meta";
      const origin = getOrigin();
      const dKm = distM(origin.lat, origin.lng, job.lat, job.lng) / 1000;
      const dText = dKm < 1 ? (dKm * 1000).toFixed(0) + " m" : dKm.toFixed(1) + " km";
      meta.textContent =
        job.level + " · " + job.teamType +
        " · " + job.lat.toFixed(5) + ", " + job.lng.toFixed(5) +
        " · 기준점에서 " + dText +
        (job.owner ? " · 등록자: " + job.owner : "");
      const memo = document.createElement("div");
      memo.className = "item-meta";
      memo.textContent = job.memo || "메모 없음";
      const actions = document.createElement("div");
      actions.className = "item-actions";
      const btnMap = document.createElement("button");
      btnMap.className = "secondary";
      btnMap.textContent = "지도에서 보기";
      btnMap.addEventListener("click", () => {
        map.setView([job.lat, job.lng], 16);
        if (job.marker) job.marker.openPopup();
      });
      const btnMatch = document.createElement("button");
      btnMatch.textContent = "근처 구직자";
      btnMatch.addEventListener("click", () => openMatchModal(job));
      const btnChat = document.createElement("button");
      btnChat.className = "secondary";
      btnChat.textContent = "1:1 메시지 (준비중)";
      btnChat.addEventListener("click", () => {
        if (!requireLogin()) return;
        openChatStub(job, "job");
      });
      actions.appendChild(btnMap);
      actions.appendChild(btnMatch);
      actions.appendChild(btnChat);
      card.appendChild(header);
      card.appendChild(meta);
      card.appendChild(memo);
      card.appendChild(actions);
      listEl.appendChild(card);
    });
    cntEl.textContent = "(" + jobs.length + "개 / 화면 내 " + visibleCount + "개)";
  }

  function renderSeekers() {
    const listEl = document.getElementById("seekList");
    const cntEl = document.getElementById("seekCnt");
    const typeFilter = getTypeFilterValue();
    seekers.forEach(s => {
      if (!s.marker) return;
      const inV = isInView(s.lat, s.lng);
      const typeOk = (typeFilter === "ALL" || s.type === typeFilter);
      s.marker.setOpacity(inV && typeOk ? 1.0 : 0.0);
    });
    if (seekers.length === 0) {
      listEl.textContent = "등록된 구직자가 없습니다.";
      cntEl.textContent = "";
      return;
    }
    listEl.innerHTML = "";
    let visibleCount = 0;
    seekers.slice().sort((a,b) => b.id - a.id).forEach(s => {
      if (!isInView(s.lat, s.lng)) return;
      if (typeFilter !== "ALL" && s.type !== typeFilter) return;
      visibleCount++;
      const card = document.createElement("div");
      card.className = "item-card";
      const header = document.createElement("div");
      header.className = "item-header";
      header.innerHTML =
        '<div class="item-title">' + getTypeEmoji(s.type) + " [" + s.type + "]</div>" +
        '<span class="badge">구직</span>';
      const meta = document.createElement("div");
      meta.className = "item-meta";
      const origin = getOrigin();
      const dKm = distM(origin.lat, origin.lng, s.lat, s.lng) / 1000;
      const dText = dKm < 1 ? (dKm * 1000).toFixed(0) + " m" : dKm.toFixed(1) + " km";
      meta.textContent =
        s.level + " · " + s.teamType +
        " · " + s.lat.toFixed(5) + ", " + s.lng.toFixed(5) +
        " · 기준점에서 " + dText +
        (s.owner ? " · 등록자: " + s.owner : "");
      const memo = document.createElement("div");
      memo.className = "item-meta";
      memo.textContent = s.memo || "소개 없음";
      const actions = document.createElement("div");
      actions.className = "item-actions";
      const btnMap = document.createElement("button");
      btnMap.className = "secondary";
      btnMap.textContent = "지도에서 보기";
      btnMap.addEventListener("click", () => {
        map.setView([s.lat, s.lng], 16);
        if (s.marker) s.marker.openPopup();
      });
      const btnChat = document.createElement("button");
      btnChat.textContent = "1:1 메시지 (준비중)";
      btnChat.addEventListener("click", () => {
        if (!requireLogin()) return;
        openChatStub(s, "seeker");
      });
      actions.appendChild(btnMap);
      actions.appendChild(btnChat);
      card.appendChild(header);
      card.appendChild(meta);
      card.appendChild(memo);
      card.appendChild(actions);
      listEl.appendChild(card);
    });
    cntEl.textContent = "(" + seekers.length + "명 / 화면 내 " + visibleCount + "명)";
  }

  function openMatchModal(job) {
    const backdrop = document.getElementById("matchModalBackdrop");
    const body = document.getElementById("matchBody");
    body.innerHTML = "";
    if (seekers.length === 0) {
      body.innerHTML = "<p class='small'>등록된 구직자가 없습니다.</p>";
    } else {
      const list = seekers.map(s => {
        const d = distM(job.lat, job.lng, s.lat, s.lng);
        const sameType = (s.type === job.type);
        return { seeker: s, distance: d, sameType };
      });
      list.sort((a,b) => {
        if (a.sameType && !b.sameType) return -1;
        if (!a.sameType && b.sameType) return 1;
        return a.distance - b.distance;
      });
      const topN = list.slice(0, 5);
      topN.forEach(item => {
        const s = item.seeker;
        const km = item.distance / 1000;
        const div = document.createElement("div");
        div.className = "match-item";
        div.innerHTML =
          "<div class='match-item-title'>" +
          getTypeEmoji(s.type) + " [" + s.type + " · " + s.level + " · " + s.teamType + "]" +
          "</div>" +
          "<div class='match-item-meta'>" +
          "거리: " +
          (km < 1 ? item.distance.toFixed(0) + " m" : km.toFixed(1) + " km") +
          (s.type === job.type ? " · 같은 업종" : "") +
          "</div>" +
          "<div class='match-item-meta'>" +
          (s.memo || "소개 없음") +
          "</div>";
        body.appendChild(div);
      });
      if (topN.length === 0) {
        body.innerHTML = "<p class='small'>근처에 구직 등록자가 없습니다.</p>";
      } else {
        const note = document.createElement("p");
        note.className = "small";
        note.textContent = "※ 실제 연락은 나중에 1:1 채팅 기능이 붙고 나서 가능합니다.";
        body.appendChild(note);
      }
    }
    backdrop.classList.add("show");
  }
  function closeMatchModal() {
    document.getElementById("matchModalBackdrop").classList.remove("show");
  }

  function openChatStub(entity, type) {
    document.getElementById("chatModalBackdrop").classList.add("show");
  }
  function closeChatModal() {
    document.getElementById("chatModalBackdrop").classList.remove("show");
  }

  function searchPlace() {
    const q = document.getElementById("searchQuery").value.trim();
    const infoEl = document.getElementById("searchInfo");
    if (!q) {
      showAlert("검색어를 입력해주세요.", "검색");
      return;
    }
    infoEl.textContent = "검색 중...";
    $.ajax({
      url: "https://api.vworld.kr/req/search",
      dataType: "jsonp",
      data: {
        service: "search",
        request: "search",
        version: "2.0",
        crs: "EPSG:4326",
        size: 10,
        page: 1,
        query: q,
        type: "place",
        format: "json",
        key: VWORLD_KEY,
      },
      success: function(data) {
        try {
          if (!data.response ||
              data.response.status !== "OK" ||
              !data.response.result ||
              !data.response.result.items ||
              data.response.result.items.length === 0) {
            infoEl.textContent = "결과 없음";
            showAlert("검색 결과가 없습니다.", "검색");
            return;
          }
          const item = data.response.result.items[0];
          const x = parseFloat(item.point.x);
          const y = parseFloat(item.point.y);
          if (searchMarker) map.removeLayer(searchMarker);
          searchMarker = L.marker([y, x]).addTo(map)
            .bindPopup("<b>" + (item.title || "검색결과") + "</b><br>" + (item.address || ""))
            .openPopup();
          map.setView([y, x], 15);
          infoEl.textContent = "첫 번째 결과로 이동: " + (item.title || "");
        } catch (e) {
          console.error(e);
          infoEl.textContent = "검색 처리 중 오류";
          showAlert("검색 처리 중 오류가 발생했습니다.", "검색 오류");
        }
      },
      error: function() {
        infoEl.textContent = "검색 요청 실패";
        showAlert("검색 요청에 실패했습니다.\n잠시 후 다시 시도해주세요.", "검색 오류");
      }
    });
  }

  async function geocodeAddress(addr) {
    const url =
      `https://api.vworld.kr/req/address?` +
      `service=address&request=getcoord&version=2.0&` +
      `crs=EPSG:4326&format=json&type=road&` +
      `address=${encodeURIComponent(addr)}&` +
      `key=${VWORLD_KEY}`;
    try {
      const res = await fetch(url);
      const data = await res.json();
      if (data.response.status !== "OK" || !data.response.result?.point) return null;
      const x = parseFloat(data.response.result.point.x);
      const y = parseFloat(data.response.result.point.y);
      if (isNaN(x) || isNaN(y)) return null;
      return { lat: y, lng: x };
    } catch (e) {
      console.error("지오코딩 오류:", e);
      return null;
    }
  }

  async function fetchCidJobs() {
    const url =
      `${CID_BASE}/getJobOpenInfoList.do` +
      `?serviceKey=${CID_KEY}` +
      `&pageNo=1` +
      `&numOfRows=20` +
      `&resultType=json`;

    try {
      console.log("CID URL:", PROXY + url);
      const res = await fetch(PROXY + url);

      const buf = await res.arrayBuffer();
      const decoder = new TextDecoder("euc-kr");
      const text = decoder.decode(buf);

      const data = JSON.parse(text);
      console.log("건설일드림넷 채용 데이터(raw):", data);

      const body = data.response && data.response.body ? data.response.body : null;
      if (!body) return [];

      let items = body.items || body.item || [];
      if (!Array.isArray(items)) items = [items];

      if (items.length > 0) {
        console.log(
          "건설일드림넷 첫 공고:",
          items[0].jobTitle,
          items[0].workPlcAddr
        );
      }

      return items;
    } catch (e) {
      console.error("CID API 오류:", e);
      return [];
    }
  }

  async function loadCidJobsToMap() {
    const items = await fetchCidJobs();
    if (!items || items.length === 0) {
      console.log("건설일드림넷 공고 없음 혹은 파싱 실패");
      return;
    }
    let count = 0;
    for (const job of items) {
      const title = job.jobTitle || job.recruitTitle || "건설일드림넷 공고";
      const addr = job.workPlcAddr || job.workPlcAddress || job.workPlace || "";
      if (!addr) continue;

      const coord = await geocodeAddress(addr);
      console.log("CID 공고:", title, addr, coord);
      if (!coord) continue;

      const pos = getOffsetLatLng(coord.lat, coord.lng);
      const marker = L.marker([pos.lat, pos.lng], {
        icon: createDivIcon(null, "cid"),
      }).addTo(map);
      marker.bindPopup(
        "🟣 <b>공식 구인공고</b><br>" +
        title +
        "<br>" + addr +
        "<br><span style='font-size:11px;'>출처: 건설일드림넷</span>"
      );
      count++;
    }
    console.log("지도에 표시된 건설일드림넷 공고 수:", count);
  }

  async function fetchGgAptConstruction(page = 1, size = 200) {
    const url =
      `${GG_BASE}/${GG_SERVICE}` +
      `?KEY=${encodeURIComponent(GG_KEY)}` +
      `&Type=json` +
      `&pIndex=${page}` +
      `&pSize=${size}`;

    try {
      console.log("GG APT URL:", PROXY + url);
      const res = await fetch(PROXY + url);
      const data = await res.json();
      console.log("공동주택 시공현황 raw:", data);

      const root = data[GG_SERVICE];
      if (!root || root.length < 2) return [];

      const body = root[1];
      const rows = body.row || [];
      console.log("공동주택 시공현황 건수:", rows.length);
      if (rows[0]) console.log("첫 공사 예시 row:", rows[0]);

      return rows;
    } catch (e) {
      console.error("경기도 공동주택 시공현황 API 오류:", e);
      return [];
    }
  }

  async function loadGgAptToMap() {
    const rows = await fetchGgAptConstruction(1, 200);
    if (!rows || rows.length === 0) {
      document.getElementById("aptList").textContent =
        "경기도 공동주택 시공현황 데이터가 없습니다.";
      return;
    }
    let count = 0;
    for (const row of rows) {
      const title =
        row.CONST_NM || row.PJTNM || row.HOUSE_NM || "경기도 공동주택 시공현장";

      const addr =
        row.ADRES || row.ADDR || row.ADDR1 || row.ROAD_ADDR || "";

      if (!addr) continue;

      const coord = await geocodeAddress(addr);
      if (!coord) continue;

      const pos = getOffsetLatLng(coord.lat, coord.lng);
      const marker = L.marker([pos.lat, pos.lng], {
        icon: createDivIcon(null, "apt"),
      }).addTo(map);

      marker.bindPopup(
        "🏢 <b>경기도 공동주택 시공현장</b><br>" +
        title +
        "<br>" + addr +
        "<br><span style='font-size:11px;'>출처: 경기도 공공데이터</span>"
      );

      aptSites.push({
        title,
        addr,
        lat: coord.lat,
        lng: coord.lng,
        marker
      });
      count++;
    }
    console.log("지도에 표시된 공동주택 시공현장 수:", count);
    renderAptList();
  }

  function renderAptList() {
    const listEl = document.getElementById("aptList");
    const cntEl = document.getElementById("aptCnt");
    if (!listEl) return;

    if (aptSites.length === 0) {
      listEl.textContent = "경기도 공동주택 시공현황 데이터가 아직 로드되지 않았습니다.";
      if (cntEl) cntEl.textContent = "";
      return;
    }

    listEl.innerHTML = "";
    let visibleCount = 0;
    aptSites.forEach(site => {
      if (!isInView(site.lat, site.lng)) return;
      visibleCount++;

      const card = document.createElement("div");
      card.className = "item-card";

      const header = document.createElement("div");
      header.className = "item-header";
      header.innerHTML =
        '<div class="item-title">🏢 ' + site.title + "</div>" +
        '<span class="badge">경기도 공동주택</span>';

      const meta = document.createElement("div");
      meta.className = "item-meta";
      const origin = getOrigin();
      const dKm = distM(origin.lat, origin.lng, site.lat, site.lng) / 1000;
      const dText = dKm < 1 ? (dKm * 1000).toFixed(0) + " m" : dKm.toFixed(1) + " km";
      meta.textContent =
        (site.addr || "주소 정보 없음") +
        " · 기준점에서 " + dText;

      const actions = document.createElement("div");
      actions.className = "item-actions";
      const btnMap = document.createElement("button");
      btnMap.className = "secondary";
      btnMap.textContent = "지도에서 보기";
      btnMap.addEventListener("click", () => {
        map.setView([site.lat, site.lng], 15);
        if (site.marker) site.marker.openPopup();
      });
      actions.appendChild(btnMap);

      card.appendChild(header);
      card.appendChild(meta);
      card.appendChild(actions);
      listEl.appendChild(card);
    });

    if (cntEl)
      cntEl.textContent =
        "(" + aptSites.length + "개 / 화면 내 " + visibleCount + "개)";
  }

  function openAuthModal(message) {
    const backdrop = document.getElementById("authModalBackdrop");
    const nicknameInput = document.getElementById("authNickname");
    const notice = document.getElementById("authNotice");
    if (message) {
      notice.textContent = message;
      notice.style.display = "block";
    } else {
      notice.textContent = "";
      notice.style.display = "none";
    }
    nicknameInput.value = currentUser && currentUser.nickname ? currentUser.nickname : "";
    backdrop.classList.add("show");
    setTimeout(() => nicknameInput.focus(), 50);
  }
  function closeAuthModal() {
    document.getElementById("authModalBackdrop").classList.remove("show");
  }
  function submitAuth() {
    const nickname = document.getElementById("authNickname").value.trim();
    if (!nickname) {
      showAlert("닉네임을 입력해 주세요.", "로그인");
      return;
    }
    currentUser = { nickname };
    saveUser();
    showAlert(nickname + "님으로 로그인되었습니다.", "로그인 완료");
    closeAuthModal();
  }

  function openSideMenu() {
    document.getElementById("sideMenu").classList.add("show");
    document.getElementById("sideMenuBackdrop").classList.add("show");
  }
  function closeSideMenu() {
    document.getElementById("sideMenu").classList.remove("show");
    document.getElementById("sideMenuBackdrop").classList.remove("show");
  }

  window.onload = function() {
    markerOffsetRegistry = {};
    maybeHideSplashImmediately();
    initMap();
    loadUser();

    loadSites();
    loadJobs();
    loadSeekers();

    renderSites();
    renderJobs();
    renderSeekers();

    loadCidJobsToMap();
    loadGgAptToMap();

    document.getElementById("splashBtn").addEventListener("click", closeSplash);

    document.getElementById("btnSearch").addEventListener("click", searchPlace);
    document.getElementById("searchQuery").addEventListener("keydown", function(e) {
      if (e.key === "Enter") {
        e.preventDefault();
        searchPlace();
      }
    });

    document.getElementById("typeFilter").addEventListener("change", () => {
      renderJobs();
      renderSeekers();
    });

    document.getElementById("btnGoMyLocation").addEventListener("click", goMyLocation);

    document.getElementById("matchCloseBtn").addEventListener("click", closeMatchModal);
    document.getElementById("matchModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeMatchModal();
    });
    document.getElementById("chatCloseBtn").addEventListener("click", closeChatModal);
    document.getElementById("chatModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeChatModal();
    });

    document.getElementById("hamburgerBtn").addEventListener("click", openSideMenu);
    document.getElementById("sideMenuBackdrop").addEventListener("click", closeSideMenu);
    document.getElementById("sideMenuClose").addEventListener("click", closeSideMenu);

    document.getElementById("menuLoginBtn").addEventListener("click", () => {
      closeSideMenu();
      openAuthModal();
    });
    document.getElementById("menuLogoutBtn").addEventListener("click", () => {
      if (!currentUser) return;
      if (document.getElementById("menuLogoutBtn").classList.contains("disabled")) return;
      showConfirm("로그아웃", "정말 로그아웃 하시겠습니까?", () => {
        currentUser = null;
        saveUser();
        showAlert("로그아웃되었습니다.", "로그아웃 완료");
        closeSideMenu();
      });
    });

    document.getElementById("menuGoJob").addEventListener("click", () => {
      closeSideMenu();
      openJobModal();
    });
    document.getElementById("menuGoSeek").addEventListener("click", () => {
      closeSideMenu();
      openSeekModal();
    });

    document.getElementById("menuFav").addEventListener("click", () => {
      closeSideMenu();
      showAlert(
        "즐겨찾기 기능은 아직 준비 중입니다.\n\n" +
        "추후 로그인 계정별로 자주 보는 공고를 별표(⭐)로 모아볼 수 있게 할 예정입니다.",
        "즐겨찾기 공고 (준비 중)"
      );
    });

    document.getElementById("menuEditDelete").addEventListener("click", () => {
      closeSideMenu();
      showAlert(
        "현재 버전은 테스트용으로, 브라우저 LocalStorage에만\n" +
        "데이터가 저장되어 있습니다.\n\n" +
        "추후 서버/DB 연동 후에는 내가 올린 공고를\n" +
        "목록에서 수정·삭제할 수 있게 할 예정입니다.",
        "공고 수정·삭제 (준비 중)"
      );
    });

    document.getElementById("menuHowTo").addEventListener("click", () => {
      closeSideMenu();
      showAlert(
        "① 지도를 드래그해서 보고 싶은 지역을 화면에 맞춥니다.\n" +
        "② 화면 안에 있는 현장/구인/구직/공동주택이 아래 목록에 표시됩니다.\n" +
        "③ 현장/구인/구직을 등록할 위치를 지도에서 클릭합니다.\n" +
        "④ 위쪽 '현장/구인/구직 등록' 버튼을 눌러 새 창에서 내용을 입력합니다.\n\n" +
        "보라색(🟣) 마커: 건설일드림넷 공고\n" +
        "청록색(🏢) 마커: 경기도 공동주택 시공현장",
        "이용 방법"
      );
    });

    document.getElementById("menuPrivacy").addEventListener("click", () => {
      closeSideMenu();
      showAlert(
        "이 서비스는 테스트용 MVP로, 서버/DB에 별도 회원정보를 저장하지 않고\n" +
        "브라우저(LocalStorage)에만 최소한의 정보(닉네임)만 보관합니다.\n\n" +
        "- 서비스명: 노다지 (현장구인지도)\n" +
        "- 관리자: 김태현\n" +
        "- 이메일: th3788@gmail.com\n\n" +
        "추후 정식 서비스 전환 시에는 정식 개인정보 처리방침을 별도로 고지할 예정입니다.",
        "개인정보 처리방침 (요약)"
      );
    });

    document.getElementById("authCancelBtn").addEventListener("click", closeAuthModal);
    document.getElementById("authSubmitBtn").addEventListener("click", submitAuth);
    document.getElementById("authModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeAuthModal();
    });

    document.getElementById("notifyOkBtn").addEventListener("click", () => {
      if (notifyMode === "confirm" && notifyConfirmCallback) notifyConfirmCallback(true);
      notifyConfirmCallback = null;
      closeNotifyModal();
    });
    document.getElementById("notifyCancelBtn").addEventListener("click", () => {
      if (notifyMode === "confirm" && notifyConfirmCallback) notifyConfirmCallback(false);
      notifyConfirmCallback = null;
      closeNotifyModal();
    });
    document.getElementById("notifyModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) {
        if (notifyMode === "confirm" && notifyConfirmCallback) notifyConfirmCallback(false);
        notifyConfirmCallback = null;
        closeNotifyModal();
      }
    });

    document.getElementById("openSiteModalBtn").addEventListener("click", openSiteModal);
    document.getElementById("openJobModalBtn").addEventListener("click", openJobModal);
    document.getElementById("openSeekModalBtn").addEventListener("click", openSeekModal);

    document.getElementById("siteCancelBtn").addEventListener("click", closeSiteModal);
    document.getElementById("jobCancelBtn").addEventListener("click", closeJobModal);
    document.getElementById("seekCancelBtn").addEventListener("click", closeSeekModal);

    document.getElementById("siteSubmitBtn").addEventListener("click", addSite);
    document.getElementById("jobSubmitBtn").addEventListener("click", addJob);
    document.getElementById("seekSubmitBtn").addEventListener("click", addSeeker);

    document.getElementById("siteModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeSiteModal();
    });
    document.getElementById("jobModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeJobModal();
    });
    document.getElementById("seekModalBackdrop").addEventListener("click", function(e) {
      if (e.target === this) closeSeekModal();
    });
  };
</script>

</body>
</html>
