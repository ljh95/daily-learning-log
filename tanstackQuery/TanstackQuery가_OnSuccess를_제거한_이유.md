주요 이유:

- 동기화 문제: 컴포넌트가 unmount 후 remount 되면 onSuccess가 재싱행 안됨(캐시된 데이터라서)
- Side effect 위치 혼란: 데이터가 fetching 로직과 UI side effect가 섞임
- React 18의 Strict Mode와 충돌: 개발 모드에서 예측 불가능한 동작
