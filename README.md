<h1>Coworkers</h1>

> 여러 명이 하나의 그룹을 만들어 할 일(Task)을 함께 공유하고 관리할 수 있는 협업용 To-do 플랫폼

<h2>소개</h2>

- 코드잇 스프린트 FE 18기 과정에서 진행한 심화 프로젝트
- 주제 선정 이유
  - 단순한 협업툴이 아닌 익명 게시판을 통한 팀원 모집이 가능한 매력적인 주제

  - 세세하고 다양한 구현사항이 있는 높은 난이도 프로젝트 도전
  <h2>사용기술</h2>

- React 19
- Next.js ( App router )
- TypeScript
- Tailwind
- TanStack Query
- Axios
- Storybook
- Vercel
- prettier + husky

<h2>역할 및 성과</h2>

<h3>전역 상태 관리 라이브러리를 사용하여 Toast 제작</h3>

<h4>개요</h4>

- 애플리케이션 전역에서 사용할 수 있는 Toast 알림 시스템을 구축하기 위해 jotai 상태관리 라이브러리를 도입하였다.
- 컴포넌트 계층 구조에 상관없이 어디서든 Toast 메시지를 호출하고 표시할 수 있도록 설계하였다.

<h4>구현 방법</h4>

**1. Jotai Atom을 활용한 상태 관리 ([toast-atom.ts:1-21](src/atoms/toast-atom.ts#L1-L21))**

```javascript
export const ToastDataAtom = atom<ToastProps | null>(null);

export const ToastAtom = atom(null, (_, set, { type, message }: ToastProps) => {
  const newToast = { type, message };
  set(ToastDataAtom, newToast);

  setTimeout(() => {
    set(ToastDataAtom, null);
  }, 1200);
});

export const RemoveToastAtom = atom(null, (_, set) => {
  set(ToastDataAtom, null);
});
```

- `ToastDataAtom`: Toast 데이터를 담는 읽기 전용 Atom
- `ToastAtom`: Toast를 추가하고 1.2초 후 자동으로 제거하는 쓰기 전용 Atom
- `RemoveToastAtom`: 사용자가 Toast를 클릭하여 수동으로 제거할 수 있는 Atom

**2. 커스텀 Hook으로 간편한 사용성 제공 ([use-toast.ts:4-14](src/hooks/use-toast.ts#L4-L14))**

```javascript
const useToast = () => {
  const addToast = useSetAtom(ToastAtom);

  return {
    success: (message: string) => addToast({ type: "success", message }),
    error: (message: string) => addToast({ type: "error", message }),
    warning: (message: string) => addToast({ type: "warning", message }),
  };
};
```

- 각 Toast 타입별로(success, error, warning) 메서드를 제공하여 직관적인 API 제공

**3. Portal을 활용한 Toast Container ([toast-container.tsx:34-42](src/components/toast/toast-container.tsx#L34-L42))**

```javascript
return ReactDOM.createPortal(
  <dialog
    ref={dialogRef}
    className="pointer-events-none fixed inset-0 flex h-full w-full items-start justify-center border-0 bg-transparent p-0 pt-8 outline-none backdrop:bg-transparent"
  >
    {toast && <Toast {...toast} />}
  </dialog>,
  document.body
);
```

- `ReactDOM.createPortal`을 사용하여 컴포넌트 계층 구조와 관계없이 DOM 트리의 최상위에 Toast를 렌더링
- HTML `<dialog>` 요소를 활용하여 접근성 향상

**4. 타입 안정성 확보 ([toast.ts:1-6](src/types/toast.ts#L1-L6))**

```typescript
export type ToastType = "success" | "error" | "warning";

export interface ToastProps {
  type: ToastType;
  message: string;
}
```

- TypeScript를 활용한 타입 정의로 개발 단계에서 오류 방지

<h4>주요 특징</h4>

- **전역 상태 관리**: jotai를 사용하여 props drilling 없이 전역에서 Toast 호출 가능
- **자동 제거**: 1.2초 후 자동으로 Toast가 사라지며, 클릭 시 즉시 제거 가능
- **타입 구분**: success, error, warning 세 가지 타입으로 시각적 피드백 제공
- **애니메이션**: fade-in/fade-out 애니메이션으로 부드러운 사용자 경험 제공
- **Portal 패턴**: DOM 구조와 독립적으로 최상위 레벨에서 렌더링하여 z-index 이슈 방지
- **접근성**: HTML `<dialog>` 요소 사용으로 스크린 리더 등 보조 기술 지원

<h4>사용 예시</h4>

```typescript
const toast = useToast();

// 성공 메시지
toast.success("작업이 완료되었습니다.");

// 에러 메시지
toast.error("오류가 발생했습니다.");

// 경고 메시지
toast.warning("입력 내용을 확인해주세요.");
```

<h4>성과</h4>

- 경량 상태관리 라이브러리인 jotai를 통해 최소한의 번들 사이즈로 효율적인 전역 상태 관리 구현
- 사용자에게 명확한 피드백을 제공하여 UX 향상
- 재사용 가능한 컴포넌트 설계로 프로젝트 전반에서 일관된 알림 시스템 구축

---

<h2>문제해결 과정 및 성과</h2>

<h3>사이드바 너비로 인한 여백 문제 개선</h3>
<h4>문제 상황</h4>

- PC기준 오른쪽 사이드바의 너비로 인해 브라우저의 UI들이 전체적으로 가운데 정렬이 되어있는 것 처럼 보이지않아 최상단 **layout.tsx**에 기본적으로 **margin** 값을 주게 되었는데, 이로인해 랜딩페이지에서 사이드바를 펼치고 접는 동작에서 불필요한 여백이 생기는 문제가 발생하였다.

<h4>문제 원인</h4>

```javascript
...
<main className="ml-0 tablet:ml-[72px] pc:ml-[270px]">
  {children}
</main>
...
```

- 기본적으로 사이드바 너비를 기준으로 왼쪽 외부여백 값을 지정했는데, 이로인해 사이드바를 접을 때 너비를 고려하지않아 여백이 생기는 문제가 발생하였다.

<h4>문제 해결방법</h4>

```javascript
...
<div className="min-h-screen bg-white pc:ml-[-198px]">
  ...
</div>
...
```

- 최상단 **layout.tsx** 여백을 유지한 후 랜딩페이지에서만 해당 여백을 음수값으로 상쇄하여 다른 페이지들과는 다른 위치 정렬을 할 수 있도록 하였다.

<h4>아쉬운 점</h4>

- 문제는 간단했지만, 생각보다 해결방법이 떠오르지않아 여백을 상쇄하는 방법을 사용했지만 각 페이지마다 **layout.tsx**를 만들어 지정해주는게 더 좋지않았을까라는 생각이 든다.

---

<h3>계정 설정 페이지 데이터 손실 개선</h3>
<h4>문제 상황</h4>

- 계정 설정 페이지에서 닉네임 변경 혹은 프로필 사진 변경을 할 때 사용자가 뒤로가기, 새로고침, 브라우저 탭 닫기 등 페이지 이탈 행동을 할 시 작성했던 변경사항들이 초기화되어 불편함을 겪는 문제가 발생했다.

<h4>문제 원인</h4>

- 문제의 원인으로는 단순히 변경사항 저장에 대한 로직만 작성하고 사용자 경험 측면에서 고려하지않아 이러한 불편한 점이 생겼다고 볼 수 있다.

<h4>문제 해결방법</h4>

```javascript
  {
    /* 탭 닫기/새로고침 사용자 이탈 감지*/
  }
  useEffect(() => {
    if (typeof window === "undefined") return;

    const handleBeforeUnload = (e: BeforeUnloadEvent) => {
      if (isDirty) {
        e.preventDefault();
      }
    };

    window.addEventListener("beforeunload", handleBeforeUnload);
    return () => window.removeEventListener("beforeunload", handleBeforeUnload);
  }, [isDirty]);

  {
    /* 페이지 이동시 사용자 이탈 감지*/
  }
  useEffect(() => {
    const handleClick = (e: MouseEvent) => {
      if (!isDirty) return;

      const target = e.target as HTMLElement;
      const link = target.closest("a");

      if (link && link.href && !link.href.startsWith("#")) {
        const url = new URL(link.href);
        const currentUrl = new URL(window.location.href);

        if (url.pathname !== currentUrl.pathname) {
          e.preventDefault();
          e.stopPropagation();
          setPendingNavigation(url.href);
          openUnsavedModal();
        }
      }
    };

    document.addEventListener("click", handleClick, true);
    return () => document.removeEventListener("click", handleClick, true);
  }, [isDirty, openUnsavedModal]);

  {
    /* 뒤로가기 버튼 클릭시 사용자 이탈 감지*/
  }
  useEffect(() => {
    if (!isDirty) return;

    const handlePopState = () => {
      openUnsavedModal();
      window.history.pushState(null, "", window.location.href);
    };

    window.history.pushState(null, "", window.location.href);
    window.addEventListener("popstate", handlePopState);

    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, [isDirty, openUnsavedModal]);
```

- 이를 해결하기 위해 총 3가지의 사용자 이탈 시나리오를 감지하는 커스텀 함수를 만들어 사용자 이탈 이벤트가 감지되게 되면 만들어둔 커스텀 모달을 띄어주는 형식으로 사용자 경험을 개선하였다.

<h4>아쉬운 점</h4>

- 브라우저 탭을 닫는 경우에도 커스텀 모달을 표출하는 방식으로 적용하고 싶었지만, 이는 브라우저 경고창을 사용해야하는 제한적인 요소가 있어 통일감을 주지 못한 아쉬움이 남았다.

---

<h3>비속어 필터링 기능으로 커뮤니티 신뢰도 향상</h3>
<h4>문제 상황</h4>

- 프로젝트 마지막 QA를 진행하면서, 자유게시판, 계정 설정 페이지 등에서 비속어가 들어간 닉네임 혹은 댓글을 작성했을 때 그대로 표출되는 문제가 있었다.

<h4>문제 원인</h4>

- 문제 원인으로는 단순히 사용자가 입력한 단어들에 대한 필터링 기능이 없었기에 당연히 입력한 그대로 표출이 되고있었고, 이러한 점들을 넘어가는 것 보다는 외부 라이브러리를 통해 필터링하여 커뮤니티 신뢰도를 향상시키는 방향이 더 좋다고 생각이 들게 되었다.

<h4>문제 해결방법</h4>

```javascript
/**
 * 텍스트에서 욕설을 *로 치환하는 함수
 * @param text 필터링할 텍스트
 * @returns 욕설이 *로 치환된 텍스트
 */
export const filterProfanity = (text: string): string => {
  return filter.clean(text);
};

/**
 * 텍스트에 욕설이 포함되어 있는지 확인하는 함수
 * @param text 확인할 텍스트
 * @returns 욕설 포함 여부
 */
export const hasProfanity = (text: string): boolean => {
  return filter.isProfane(text);
};

/**
 * 사용자 정의 욕설을 추가하는 함수
 * @param words 추가할 욕설 단어 배열
 */
export const addBadWords = (words: string[]): void => {
  filter.addWords(...words);
};

/**
 * 욕설 목록에서 단어를 제거하는 함수
 * @param words 제거할 단어 배열
 */
export const removeBadWords = (words: string[]): void => {
  filter.removeWords(...words);
};
```

- **badwords-ko** 라이브러리를 통한 외부 함수 정의와 커스텀 비속어 목록을 작성하여 함수를 통해 사용자가 입력한 텍스트에 대한 필터링을 한 후 표출 시키는 형태로 적용하여 커뮤니티 신뢰도를 향상 시킬 수 있도록 하였다.

<h4>아쉬운 점</h4>

- 한국어 기준 비속어 필터링에 대한 외부 라이브러리가 많이 없어 향후 확장성에 대한 아쉬움과 기본적으로 필터링해주는 단어 목록의 범위가 제한적이라 일일히 테스트 후 목록을 구축해줘야하는 제한사항들이 아쉬웠다. 다음에는 외부라이브러리가 아닌 직접 구현 방식으로 해보는 것도 좋을 것 같다는 생각이다.
