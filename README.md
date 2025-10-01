# 📖 GitHub Pages 설정 가이드

본 문서는 **ms_team_convention** 저장소의 Convention 문서를 GitHub Pages로 배포하는 방법을 정리한 가이드입니다.  
저장소의 `/docs/index.md` 파일을 메인 페이지로 하여, Convention 문서를 웹에서 쉽게 열람할 수 있도록 구성합니다.

---

## 📂 저장소 구조

```
ms_team_convention/
 ├─ README.md
 └─ docs/
     ├─ index.md
     └─ convention/
         ├─ java.md
         └─ git.md
```

- `README.md` : 저장소 가이드 문서
- `docs/index.md` : 메인 페이지 (Convention 문서 링크 포함)
- `docs/convention/java.md` : Java / Spring Boot Convention
- `docs/convention/git.md` : Git Branch & Commit Convention

---

## ⚙️ GitHub Pages 설정 방법

1. GitHub 저장소 접속 → **Settings** 클릭
2. 좌측 메뉴에서 **Pages** 선택
3. **Source** 설정
   - Branch: `main`
   - Folder: `/docs`
4. **Save** 버튼 클릭
5. 잠시 후, Pages 배포 URL이 활성화됨  
   - 예: `https://samuel-obigo.github.io/ms_team_convention/`

---

## 🌐 접근 경로

- 메인 페이지 : [index.md](https://orgname.github.io/ms_team_convention/)  
- Java Convention : [convention/java.md](https://orgname.github.io/ms_team_convention/convention/java.html)  
- Git Convention : [convention/git.md](https://orgname.github.io/ms_team_convention/convention/git.html)  

---

## ✅ 참고 사항

- `.md` 파일은 GitHub Pages에서 자동으로 `.html`로 변환됩니다.  
- `/docs/index.md` 파일이 있으면 자동으로 메인 페이지로 인식됩니다.  
- Public 저장소는 무료로 Pages 사용 가능, Private 저장소는 **Team/Enterprise 플랜**에서만 Pages 지원됩니다.  

---