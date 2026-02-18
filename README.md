<h1>Coworkers</h1>

> 여러 명이 하나의 그룹을 만들어 할 일(Task)을 함께 공유하고 관리할 수 있는 협업용 To-do 플랫폼

<h2>역할 및 성과</h2>

<h3>전역 상태 관리 라이브러리를 사용하여 Toast 제작</h3>

<h4>개요</h4>

- 알림이 필요한 위치마다 `useState`와 `setTimeout` 로직이 중복 작성되고 있었고, 깊은 계층 구조에서는 prop drilling이 발생하고 있었습니다.

- 컴포넌트 트리에 종속되지 않는 중앙 집중식 알림이 필요하다고 판단했고, Jotai의 Atom 기반 상태와 React Portal을 결합해 전역 Toast 시스템을 구축했습니다.

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

<h3>계정 설정 페이지 데이터 손실 개선</h3>

<mark>문제 상황</mark>

- 계정 설정 페이지에서 닉네임 변경 혹은 프로필 사진 변경을 할 때 사용자가 뒤로가기, 새로고침, 브라우저 탭 닫기 등 페이지 이탈 행동을 할 시 작성했던 변경사항들이 초기화되어 불편함을 겪는 문제가 발생했다.

<mark>문제 원인</mark>

- 문제의 원인으로는 단순히 변경사항 저장에 대한 로직만 작성하고 사용자 경험 측면에서 고려하지않아 이러한 불편한 점이 생겼다고 볼 수 있다.

<mark>문제 해결방법</mark>

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

- 사용자 이탈 시나리오를 세 가지로 구분해 처리했습니다.
  
  - 브라우저 탭 닫기 및 새로고침은 `beforeunload` 이벤트를 활용해 기본 경고창을 노출했습니다.
    
  - 라우트 이동 및 뒤로가기 동작은 직접 이벤트를 제어하여 커스텀 모달을 통해 이탈 여부를 확인하도록 구현했습니다.

- 이탈 유형에 따라 브라우저 기본 동작과 커스텀 로직을 분리하여 사용자 경험을 개선했습니다.

<mark>아쉬운 점</mark>

- 브라우저 탭 종료의 경우 보안 정책상 커스텀 UI를 노출할 수 없어 기본 경고창을 사용할 수밖에 없었습니다.
  
- 이로 인해 모든 이탈 시나리오에서 동일한 UI 경험을 제공하지 못한 점이 아쉬웠습니다.

---

<h3>비속어 필터링 기능으로 커뮤니티 신뢰도 향상</h3>

<mark>문제 상황</mark>

- 다수가 사용하는 협업 플랫폼 특성상, 부적절한 언어 사용에 대한 최소한의 가이드라인이 필요했습니다.
  
- QA 과정에서 닉네임 및 게시글에 비속어가 그대로 노출되는 것을 확인했고, 서비스 신뢰도 측면에서 개선이 필요하다고 판단했습니다.

<mark>문제 원인</mark>

- 입력값에 대한 별도의 검증 로직이 존재하지 않아, 사용자가 입력한 텍스트가 그대로 렌더링되고 있었습니다.
  
- 악의적 사용까지 완전히 방어하려면 서버 단 검증이 필요하지만, 리소스 제약을 고려해 우선 프론트 단에서 1차 필터링을 적용하기로 결정했습니다.

<mark>문제 해결방법</mark>

```ts
export const filterProfanity = (text: string): string => {
  return filter.clean(text);
};

export const hasProfanity = (text: string): boolean => {
  return filter.isProfane(text);
};

export const addBadWords = (words: string[]): void => {
  filter.addWords(...words);
};

export const removeBadWords = (words: string[]): void => {
  filter.removeWords(...words);
};
```

- badwords-ko 라이브러리를 활용해 텍스트 입력 시 비속어를 감지하고 특수문자로 자동 치환하도록 적용했습니다.

- 사용자 정의 단어를 추가/제거할 수 있도록 확장성을 고려해 유틸 함수로 분리했습니다.

- 직접적인 입력 차단보다는 자연스럽게 콘텐츠를 정제하는 방향으로 UX를 설계했습니다.


<mark>아쉬운 점</mark>

- 한국어 기반 필터링 라이브러리의 선택지가 제한적이었고, 기본 제공 단어 목록의 범위가 충분하지 않았습니다.

- 일부 우회 표현이나 변형 단어에 대한 대응에는 한계가 있었습니다.

- 향후에는 외부 라이브러리 의존도를 줄이고, 서비스 특성에 맞는 자체 필터링 로직을 설계해보고자 합니다.
