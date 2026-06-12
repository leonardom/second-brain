# State Management in Mobile Apps

> Tags: #tradeoffs #dx

Patterns and tools for managing application state across iOS, Android, and cross-platform mobile apps.

---

## Types of State

Understanding what kind of state you're managing drives the right tool choice:

| Type | Examples | Lifetime |
|------|----------|----------|
| **UI State** | Loading flags, selected tab, form input | View lifecycle |
| **Navigation State** | Current screen, back stack | App session |
| **Server Cache** | API responses, remote data | Session / persistent |
| **Local App State** | User preferences, drafts | Persistent |
| **Ephemeral/Shared State** | Auth token, current user | Session |

---

## Principles

1. **Single Source of Truth** — avoid duplicating state across multiple places
2. **Unidirectional Data Flow** — state flows down; events flow up
3. **Minimal State** — derive what you can from existing state (computed properties)
4. **Lift State Up** — share state at the lowest common ancestor

---

## Cross-Platform (React Native)

### Redux Toolkit (RTK)

Best for: complex apps with many shared slices, strong DevTools support.

```typescript
// Slice
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface CartState {
    items: CartItem[];
    total: number;
}

const cartSlice = createSlice({
    name: "cart",
    initialState: { items: [], total: 0 } as CartState,
    reducers: {
        addItem(state, action: PayloadAction<CartItem>) {
            state.items.push(action.payload);
            state.total += action.payload.price;
        },
        removeItem(state, action: PayloadAction<string>) {
            state.items = state.items.filter(i => i.id !== action.payload);
        },
    },
});

export const { addItem, removeItem } = cartSlice.actions;
```

### RTK Query — Server Cache

```typescript
const api = createApi({
    reducerPath: "api",
    baseQuery: fetchBaseQuery({ baseUrl: "/api" }),
    endpoints: (builder) => ({
        getUser: builder.query<User, string>({
            query: (id) => `/users/${id}`,
        }),
        updateUser: builder.mutation<User, Partial<User>>({
            query: (body) => ({ url: `/users/${body.id}`, method: "PATCH", body }),
            invalidatesTags: ["User"],
        }),
    }),
});

export const { useGetUserQuery, useUpdateUserMutation } = api;
```

### Zustand — Lightweight Alternative

Best for: simpler apps, fewer boilerplate requirements.

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AuthState {
    user: User | null;
    setUser: (user: User | null) => void;
    logout: () => void;
}

const useAuthStore = create<AuthState>()(
    persist(
        (set) => ({
            user: null,
            setUser: (user) => set({ user }),
            logout:  () => set({ user: null }),
        }),
        { name: "auth-storage" } // persisted to AsyncStorage
    )
);
```

### React Query (TanStack Query)

Best for: server state management with minimal boilerplate.

```typescript
// Automatic caching, background refresh, loading/error states
const { data: user, isLoading, error } = useQuery({
    queryKey: ["user", userId],
    queryFn: () => fetchUser(userId),
    staleTime: 1000 * 60 * 5, // 5 minutes
});

const mutation = useMutation({
    mutationFn: updateUser,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ["user"] }),
});
```

### Tool Comparison

| | Redux Toolkit | Zustand | TanStack Query | Jotai |
|-|:---:|:---:|:---:|:---:|
| Server state | RTK Query | ❌ | ✅ (primary) | ❌ |
| Client/UI state | ✅ | ✅ | ❌ | ✅ |
| DevTools | Excellent | Good | Good | Good |
| Bundle size | Large | Tiny | Medium | Tiny |
| Boilerplate | Medium | Low | Low | Low |
| Learning curve | High | Low | Medium | Low |

---

## iOS (Swift / SwiftUI)

### SwiftUI + `@StateObject` / `@ObservableObject`

```swift
// ViewModel
class CartViewModel: ObservableObject {
    @Published var items: [CartItem] = []
    @Published var isLoading = false

    var total: Double { items.reduce(0) { $0 + $1.price } }

    func addItem(_ item: CartItem) {
        items.append(item)
    }
}

// View
struct CartView: View {
    @StateObject private var viewModel = CartViewModel()

    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        Text("Total: \(viewModel.total, format: .currency(code: "USD"))")
    }
}
```

### `@Observable` (iOS 17+ / Swift 5.9)

Simpler than `ObservableObject` — no need for `@Published`:

```swift
@Observable
class UserStore {
    var currentUser: User?
    var isAuthenticated: Bool { currentUser != nil }

    func login(with credentials: Credentials) async throws {
        currentUser = try await authService.login(credentials)
    }
}
```

### The Composable Architecture (TCA)

Elm-inspired, unidirectional. Best for: large teams, complex flows, testability.

```swift
@Reducer
struct CounterFeature {
    struct State: Equatable { var count = 0 }
    enum Action { case incrementTapped, decrementTapped }

    var body: some ReducerOf<Self> {
        Reduce { state, action in
            switch action {
            case .incrementTapped: state.count += 1; return .none
            case .decrementTapped: state.count -= 1; return .none
            }
        }
    }
}
```

---

## Android (Kotlin / Jetpack Compose)

### ViewModel + StateFlow

```kotlin
class CartViewModel(private val cartRepo: CartRepository) : ViewModel() {

    private val _uiState = MutableStateFlow(CartUiState())
    val uiState: StateFlow<CartUiState> = _uiState.asStateFlow()

    fun addItem(item: CartItem) {
        viewModelScope.launch {
            _uiState.update { it.copy(items = it.items + item) }
            cartRepo.saveItem(item)
        }
    }
}

// Composable
@Composable
fun CartScreen(viewModel: CartViewModel = hiltViewModel()) {
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    // ...
}
```

### UiState Pattern

```kotlin
sealed class UiState<out T> {
    data object Loading : UiState<Nothing>()
    data class Success<T>(val data: T) : UiState<T>()
    data class Error(val message: String) : UiState<Nothing>()
}
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| God ViewModel | Unmanageable state in one class | Split by feature/screen |
| Passing state as props 6 levels deep | Prop drilling | Use context/store |
| Calling API in component body | Re-fetch on every render | Use query hook/ViewModel |
| Storing derived state | State gets out of sync | Compute from source state |
| Not handling loading/error states | Poor UX | Always model all async states |

---

## References

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [Zustand](https://zustand-demo.pmnd.rs/)
- [TanStack Query](https://tanstack.com/query)
- [The Composable Architecture (TCA)](https://github.com/pointfreeco/swift-composable-architecture)
- [Android ViewModel Guide](https://developer.android.com/topic/libraries/architecture/viewmodel)
- [Jotai](https://jotai.org/)
