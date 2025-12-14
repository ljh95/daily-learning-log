## Context API 와 Zustand사이의 딜레마

Zustand와 같ㅇ느 전역 스토어 패턴이 가진 편리함 문제머을 파악하고
해결하기 위한 대안을 제시하는것

## 1. ContextAPI와 Zustand의 역할과 한계

Context API: 의존성 주입 도주로서의 가치
전역적ㅇ니 단일 상태르 만드느것이 아니라, 특정 컴포넌트에 의해 생성된 지역 상태와 그 상태를 변경하는 함수들에 접근할 수 잇다는 통롤 제공

가장 주용한 특징은
상태의 생명주기가 React컴포너트의 생명주기와 완벽하게 잂치한다는점
상태는 Provider가 마운트될 떄 생성되고, PRivder가 언마우튼될 떄 함꼐 소멸함
이는 상태가 필요한곳에서만 존재하고, 필요없어지면 자동으로 정리되느 ㄴ예측 가능한 동작으로 보장함

Zustand:선택적 구동을 통한 렌덜이 최적화

Context API는 불필요한 리렌더링 문제임
대시보드 예로 ㅗ벡ㅆ음

```tsx
interface DashboardState {
	// 모니터링 데이터 (실시간으로 자주 업데이트)
	cpuUsage: number;
	memoryUsage: number;
	networkTraffic: number[];
	errorLogs: string[];
	activeUsers: number;
	lastUpdated: Date;

	// 사용자 설정 (초기 로드 후 가끔 변경)
	userId: string;
	dashboardLayout: 'grid' | 'list';
	preferences: {
		theme: 'light' | 'dark';
		refreshInterval: number;
	};
}
```

이 상태를 Context API로 관리하면 Cpu 사용량만 표시하는 CpuWidget은 전혀 상관없는 memoryUsage나 preferences.theme이 변경될 떄도 불피룡하게 리렌더링됨
Contesxt는 구독하는 개체 내의 어떤 값이 바뀌드 모든 구독자에게 변경사실을 알리기 떄문

Zustand는 이 문제를 '선택적 구독' 이라는 면쾌한 방법으로 해결함
개발자는 스토어ㅏㅔ 자신이 필요한 다태 조각만 정확하게 지정하여 구독 가능

```tsx
function CpuWidget() {
	// cpuUsage만 구독 → memoryUsage, preferences 변경 시 리렌더링 안됨
	const cpuUsage = useDashboardStore((state) => state.cpuUsage);
	return <div>CPU: {cpuUsage}%</div>;
}
```

이처럼Zustand는 Context를 여러개로 잘게 쪼개는 수고 ㅇ벗이도 레덜이 성능을 극대ㅑ화할 수 있음
이러한 강력함에도 불구하고 Zustand사용법은 애플리케이션ㅇ늬 안정성을 해치는 심각한 문제를 야기할 수 있음

---

## 2. 전역상태의 3가지 문제점

### 1. 동적 초기화의 부재

Zustand는 React컴포넌트 트리와 무관하게 모듈이 처음 import되는 시점에 단 한번 초기화 됨
이는 userId처럼 런타임에 결정되는 ㄱ밧을 스토어 생성 시점에 직접 주입할 수 없다는 것을 의미

결국 개발자는 useEffect와 같ㅇ느 별도의 방법으로 런타임 초기화를 구축해야함

```tsx
function Dashboard({ userId }: { userId: string }) {
	const { initializeStore } = useDashboardStore();

	useEffect(() => {
		initializeStore(userId);
	}, [userId]);

	return <div>{/* 실제 대시보드 */}</div>;
}
```

이 방식을 초기값을 설정하기 위해 필연적으로 추가 렌덜이응ㄹ 유발하며
이는 비효율적이고, 더 큰 문제를 야기함

### 2. 의미없는 기본값으로 인한 초기 렌더링과 버그

동적 초기화가 불가능하기 때문에, 우리느 스토어 생성할 떄 userId:''와 같은 의밀없는 기본값을 사용할 수 밖에 없음

이것은 첫 렌더링이 불완전하다는 비ㅇ효율을 넘어
실제 버그의 원인이 될 슁ㅆ음
실제로, 초기 렌덜이 시 스토어르 구독하느 하위 컴포나ㅓ트가 빈문자열 상태인 userId를 그대로 API 요청에 사용해 400 Bad Requset에러를 유발했음

### 3. 수동으로 관리해야하는 '영구적' 상태와 정리의 함정

Zustand는 React와 완전히 별개로 존재함
한번 생성되면 애플리케이션이 종료될떄가지 메모리에 영구적으로 남아있음
이로인해 개발자는 상태 정리의 책임을 떠맡아야함

```tsx
function DashboardPage({ dashboardId }: { dashboardId: string }) {
	const { resetFilters } = useDashboardStore();

	useEffect(() => {
		resetFilters(); // 👈 진입 시 초기화
		return () => {
			resetFilters(); // 👈 이탈 시도 초기화 (깜빡하기 쉬움)
		};
	}, [dashboardId]);

	// ...
}
```

문제로 이 정리 로직을 DashboardPage에서만 하는게 아니라는점이다.
ProjectSwitcher컴포넌트에서 프로젝트를 변겨할 때나 LogoutButton으로 로그아웃 할 때도 이 초기화 로직을 일ㅇ리이 기억하고 호출해야함

```tsx
function ProjectSwitcher() {
	const handleProjectChange = (projectId: string) => {
		switchProject(projectId);
		// ⚠️ 개발자가 초기화를 깜빡했다면?
		// useDashboardStore.getState().resetFilters();
		// → 이전 프로젝트의 필터 설정이 그대로 적용됨
	};
}
function LogoutButton() {
	const handleLogout = () => {
		logout();
		// ⚠️ 여기서도 초기화를 빼먹으면?
		// useDashboardStore.getState().resetFilters();
		// → 다음 사용자가 로그인했을 때 이전 사용자의 설정이 보임
	};
}
```

이처럼 수동으로 상태를 정리하는 방식을 개발자의 실수를 유발하기 매우 쉬우며,
심각한 데이터 오염 및 유출 버그로 이어질 수 있음
이 세가지문제는 전역 스토어를 사용할 때 항상 경계해야할 핵심 위험임

---

## 3. 해결책: Context의 생명주기와 Zustand의 성능을 결합한 '스토어의 지역화'

아ㅠ서 제기된 전역 스토어의 세가지 문제 (동적 초기화, 의미없는 첫 렌덜이, 수동 상태 정리)를 모두 해결하기 위한 패턴이 바로 스토어 지역화임

### 1. 스토어 팩토리 함수 생성

전역에서 단일 스토어를 생성하는 대신, initialData를 인자로 받아 런타임에 동적으로 스토어를 생성하으 팩토리 함수를 만듬

```tsx
// 스토어를 생성하는 팩토리 함수
const createDashboardStore = (initialData: DashboardStoreInitial) => {
	return createStore<DashboardStore>((set) => ({
		// ... 다른 상태들
		// ✅ 더미 값 없이 런타임 데이터로 즉시 초기화
		userId: initialData.userId,
		preferences: initialData.preferences ?? { theme: 'light', refreshInterval: 5000 },
		// ... 액션 함수들
	}));
};
```

이 함수 덕에 userId: ''와 같은 의미없는 기본값을 사용할 필요가 엇ㅂ음

### 2. Provider를 통한 스토어 생성 및 주입

다음으로 , 이 팩토리 함수를 사용하여 React생명주기안에서 스토어를 생성하는 Provider컴포넌트를 만듬

```tsx
const DashboardStoreContext = createContext<StoreApi<DashboardStore> | null>(null);

export function DashboardStoreProvider({ children, userId, preferences }: Props) {
	// useRef를 사용해 Provider가 마운트되는 시점에 단 한 번만 스토어를 생성
	const storeRef = useRef<StoreApi<DashboardStore>>();
	if (!storeRef.current) {
		storeRef.current = createDashboardStore({ userId, preferences });
	}

	return <DashboardStoreContext.Provider value={storeRef.current}>{children}</DashboardStoreContext.Provider>;
}
```

useRef를 사용해 Provider가 처음 마운트 될 때만 스토어 생성하고 생성되 스토어 인스턴스를 Copntext를 통해 하위 컴포너트로 주입함
이제 스토어의 생명주기는 DashboardStoreProivider의 생명주기와 동ㅇ리해짐

### 3. 커스텀 훅으로 스토어 사용

```tsx
export const useDashboardStore = <T,>(selector: (state: DashboardStore) => T): T => {
	const store = useContext(DashboardStoreContext);
	if (!store) throw new Error('DashboardStoreProvider not found');

	// Context에서 받은 스토어 인스턴스와 Zustand의 useStore 훅을 결합
	return useStore(store, selector);
};
```

이 커스턱 훅은 Context로부터 스토어 인스턴스를 가져온 뒤, Zustand의 useStore훅과 셀렉터를 결합하여 선택적 구돌을완벽히 지원함

```tsx
function App() {
	const { userId, preferences } = useAuth(); // 로그인한 사용자 정보

	return (
		// Provider를 통해 런타임 데이터를 주입하며 스토어 생성
		<DashboardStoreProvider
			userId={userId}
			preferences={preferences}>
			<DashboardPage />
		</DashboardStoreProvider>
	);
}

function DashboardPage() {
	// 더미 값 없이 실제 userId로 첫 렌더링
	const userId = useDashboardStore((state) => state.userId);
	const cpuUsage = useDashboardStore((state) => state.cpuUsage);

	// useEffect로 초기화할 필요 없음
	// Provider가 언마운트될 때 스토어도 자동으로 정리됨
	return (
		<div>
			<h1>Dashboard for {userId}</h1>
			<p>CPU: {cpuUsage}%</p>
		</div>
	);
}
```

## 4. 보일러 플레이트 코드 줄이기

스토어 지역화 패턴을 많은 구조적 이점을 제공하지만
매벙 새로운 스토어를 만들 때마다 Context, Provider, 커스텀 훅을 반보ㅓㄱ적으로 작성해야하는 보일러 플레이트 코드가 많다는 단점이 있음
이러한 반복 작업을 줄여야함

간단한 Monitorting State를 위한 지역화된 스토어를 구현하느 전체 코드를 보면 양을 체감할 수 있음

```tsx
import { createContext, useContext, useRef } from 'react';
import { createStore, StoreApi, useStore } from 'zustand';

interface MonitoringState {
	cpuUsage: number;
	memoryUsage: number;
	updateCpuUsage: (value: number) => void;
	updateMemoryUsage: (value: number) => void;
}

const createMonitoringStore = (initial?: { cpuUsage?: number; memoryUsage?: number }) => {
	return createStore<MonitoringState>((set) => ({
		cpuUsage: initial?.cpuUsage ?? 0,
		memoryUsage: initial?.memoryUsage ?? 0,
		updateCpuUsage: (value) => set({ cpuUsage: value }),
		updateMemoryUsage: (value) => set({ memoryUsage: value }),
	}));
};

const MonitoringContext = createContext<StoreApi<MonitoringState> | null>(null);

interface MonitoringProviderProps {
	children: React.ReactNode;
	initialData?: { cpuUsage?: number; memoryUsage?: number };
}
export const MonitoringProvider = ({ children, initialData }: MonitoringProviderProps) => {
	const storeRef = useRef<StoreApi<MonitoringState>>();
	if (!storeRef.current) {
		storeRef.current = createMonitoringStore(initialData);
	}
	return <MonitoringContext.Provider value={storeRef.current}>{children}</MonitoringContext.Provider>;
};

export const useMonitoring = <T,>(selector: (state: MonitoringState) => T): T => {
	const store = useContext(MonitoringContext);
	if (!store) {
		throw new Error('Missing MonitoringContext.Provider in the tree');
	}
	return useStore(store, selector);
};
```

추상화를 통한 해결

이런 반복적인 구조를 createSimpleStore라는 유틸로 추강화하여 해결할 수 있음

```tsx
import { createSimpleStore } from '@/shared/react/storeFactory';

interface MonitoringState {
	cpuUsage: number;
	memoryUsage: number;
	updateCpuUsage: (value: number) => void;
	updateMemoryUsage: (value: number) => void;
}

const monitoringStore = createSimpleStore<MonitoringState>(
	'Monitoring', // 디버깅을 위한 스토어 이름
	(set, initialData) => ({
		// 스토어 로직
		cpuUsage: initialData?.cpuUsage ?? 0,
		memoryUsage: initialData?.memoryUsage ?? 0,
		updateCpuUsage: (value) => set({ cpuUsage: value }),
		updateMemoryUsage: (value) => set({ memoryUsage: value }),
	})
);

export const MonitoringProvider = monitoringStore.Provider;
export const useMonitoring = monitoringStore.useStore;
```

두 코드를 비교해보면, 우ㅠ틸 함수를 사용함으로 써ㅏ 코드가 70%까지 감소해음
이제 핵심 로직에만 집중할 수 있으며, 누구나 쉽게 지역화된 스토어 패턴을 사용할 수 잇음

---

## 5. 언제 스토어를 지역화 하는가?

1. 상태 업데이트가 매운 빈번하여 심각한 리렌덜이 문제가 예상될 때
2. 상태간 의존성이 복잡하여 Context API를 원자적으로 잘게 설계하기 부담스러울 때

실무에서 굳이 스토어를 쓸 필요가 없는 상황에서 무분별하게 전역 스토어를 사용하여 발생하는 버그를 너무나도 많이 목격했기 떄문
전역 스토어의 예측 불가능한 사이드 이펙트로인해 발생하는 무제 대부부은 스토어를 필요한 곳에만 지역화함으로 해결되었음
