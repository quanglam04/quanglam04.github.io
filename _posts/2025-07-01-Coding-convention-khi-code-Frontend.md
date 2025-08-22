---
title: "Coding convention khi code Frontend"
date: 2025-07-01 21:38:00  +0700
categories: [Programming, Convention, Frontend]
tags: [coding style, backend, convention]
---

---

# Coding Guidelines

## Mục lục

1. Project Structure
2. Naming Conventions
3. Component Patterns
4. TypeScript Best Practices
5. State Managements
6. Hooks Guilelines
7. Error Handling
8. Performance
9. Code Organization
10. Screen Development Guide

## Project Structure

```
src/
├── app/
│   ├── layout/              # Layout components
│   ├── pages/               # Page components and routing
│   ├── routes/              # Router configuration
│   ├── store/               # State management (Redux Toolkit)
│   ├── styles/              # Global styles and theme
│   ├── App.tsx              # Root component
│   └── index.ts             # Entry point
│
├── assets/                  # Static assets (images, fonts, etc.)
├── environments/            # Environment configuration (api, etc.)
├── locales/                 # Internationalization (JA, EN, etc.)
│
└── shared/                  # Shared modules and utilities
    ├── components/          # Reusable UI Components
    ├── config/              # Configuration files
    ├── constants/           # Application constants
    ├── enums/               # TypeScript enums
    ├── hooks/               # Custom React hooks
    ├── services/            # API services and external integrations
    ├── types/               # TypeScript type definition
    ├── utils/               # Utility functions
    └── validations/         # Form and data validation rules
```

## Folders & Files Naming

```
//✅Good
user-profile/index.tsx       // Components: kebab-case
useUserData.ts               // Hooks: camelCase, with 'use' prefix
format-date.ts               // Utils: kebab-case
user-types.ts                // Types: kabab-case
api-endpoints.ts             // Constants: kebab-case
button/index.tsx             // Component folders:  kebab-case
useLoadingHook.ts            // Hook file: camelCase with 'Hook' suffix

//❌Bad
userProfile.tsx              // ❌ camelCase for component files
UserData.ts                  // ❌ Hook without 'use' prefix
format_date.ts               // ❌ snake_case
userTypes.ts                 // ❌ camelCase for type files
apiEndpoints.ts              // ❌ camelCase for constants
Button/index.tsx             // ❌ PascalCase for folders
useLoading.ts                // ❌ Missing 'Hook' suffix
```

## Naming Conventions

**Variables and Functions**

```
// ✅ Good
const userName = 'abc_d';
const isUserLoggedIn = true;
const getUserProfile = async (id: string) => { ... };
const handleSubmit = () => { ... };
const onUserSelect = (user: User) => { ... };

// ❌ Bad
const user_name = 'abc_d'; // snake_case
const userLoggedIn = true; // PascalCase for variable
const GetUserProfile = async () => { ... }; // PascalCase for function
const submit = () => { ... }; // Not descriptive
const userSelect = (user: User) => { ... }; // Missing 'on' prefix for event handlers
```

**Interfaces and Types**

```
// ✅ Good
interface User {
  id: string;
  name: string;
  email: string;
}

type UserStatus = 'active' | 'inactive' | 'pending';
type UserFormData = Omit<User, 'id' | 'createdAt'>;

// ❌ Bad
interface user { // lowercase
  ID: string; // ALL_CAPS for property
  user_name: string; // snake_case
}

type userStatus = string; // Too generic, should use union
type userData = any; // Using 'any' type
```

**Components**

```
// ✅ Good
interface UserProfileProps {
  user: User;
  onEdit?: (user: User) => void;
  className?: string;
}

const UserProfile = ({
  user,
  onEdit,
  className = ''
}: UserProfileProps) => {
  const handleEdit = () => onEdit?.(user);

  return (
    <div className={`user-card ${className}`}>
      <h3>{user.name}</h3>
      {onEdit && <button onClick={handleEdit}>Edit</button>}
    </div>
  );
};

// ❌ Bad
const userProfile = ({ user }) => { // lowercase, no typing
  return <div>{user.name}</div>
};

function UserProfile(user) { // No props typing
  return <div>{user.name}</div>
}
```

## Components Patterns

**Functional Components with TypeScript**

```
// ✅ Good
interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
  className?: string;
  loading?: boolean;
}

const UserCard = ({
  user,
  onEdit,
  className = '',
  loading = false
}: UserCardProps) => {
  const handleEdit = useCallback(() => {
    onEdit?.(user);
  }, [onEdit, user]);

  if (loading) {
    return <div className="loading-skeleton">Loading...</div>;
  }

  return (
    <div className="user-card ${className}">
      <h3>{user.name}</h3>
      {onEdit && <button onClick={handleEdit}>Edit</button>}
    </div>
  );
};

// ❌ Bad
const UserCard = ({ user, onEdit, className }) => { // No TypeScript
  return (
    <div className={`className || ''`}> // Manual fallback
      <h3>{user.name}</h3>
      <button onClick={() => onEdit(user)}>Edit</button> // No optional check
    </div>
  );
};
```

**Props with Default Values**

```
// ✅ Good
interface ButtonProps {
  children: React.ReactNode;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
  disabled?: boolean;
  loading?: boolean;
}

const Button = ({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false
}: ButtonProps) => (
  <button
    className={`btn btn-${variant} btn-${size}`}
    disabled={disabled || loading}
  >
    {loading ? 'Loading...' : children}
  </button>
);

// ❌ Bad
const Button = ({ children, variant, size }) => {
  const buttonVariant = variant || 'primary';
  const buttonSize = size || 'medium';
  // Manual fallback
  return (
    <button className={`btn btn-${buttonVariant} btn-${buttonSize}`}>
      {children}
    </button>
  );
};
```

# TypeScript Best Practices

**Type Definitions**

```
// ✅ Good
interface ApiResponse<T> {
  data: T;
  message: string;
  status: number;
  timestamp: string;
}

interface User {
  id: string;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}

type UserFormData = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
type UserStatus = 'active' | 'inactive' | 'pending';

// ❌ Bad
interface ApiResponse {
  data: any; // Using any
  message: string;
  status: number;
}

interface User {
  id: any; // Should be string
  name: string;
  email: string;
  createdAt: string; // Should be Date
  status: string; // Should be union type
}

type UserFormData = any; // Too generic
```

**Generic Types**

```
// ✅ Good
interface SelectProps<T> {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string | number;
  placeholder?: string;
  disabled?: boolean;
}

const Select = <T>({
  options,
  value,
  onChange,
  getLabel,
  getValue,
  placeholder = 'Select an option',
  disabled = false
}: SelectProps<T>) => {
  return (
    <select
      value={value ? getValue(value) : ''}
      onChange={(e) => {
        const selected = options.find(opt => getValue(opt) === e.target.value);
        if (selected) onChange(selected);
      }}
      disabled={disabled}
    >
      <option value="">{placeholder}</option>
      {options.map(option => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
};

// ❌ Bad
interface SelectProps {
  options: any[];
  value: any; // No generics
  onChange: (value: any) => void;
  getLabel: (option: any) => string;
}

const Select = ({ options, value, onChange, getLabel }) => { // No typing
  return (
    <select onChange={(e) => onChange(e.target.value)}> // Wrong Logic
      {options.map(option => (
        <option>{getLabel(option)}</option>      // Missing key
      ))}
    </select>

  );
};
```

# State Management

**useState with TypeScript**

```
// ✅ Good
const [user, setUser] = useState<User | null>(null);
const [loading, setLoading] = useState<boolean>(false);
const [errors, setErrors] = useState<Record<string, string>>({});

interface FormState {
  values: Record<string, any>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isValid: boolean;
}

const [formState, setFormState] = useState<FormState>({
  values: {},
  errors: {},
  touched: {},
  isValid: false,
});

// ❌ Bad
const [user, setUser] = useState(null); // No typing
const [loading, setLoading] = useState(); // Undefined initial value
const [errors, setErrors] = useState({}); // No typing
const [formValues, setFormValues] = useState(); // Multiple states instead of object
const [formErrors, setFormErrors] = useState();
const [formTouched, setFormTouched] = useState();
```

**Redux Toolkit Pattern**

```
// ✅ Good
interface AppState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: AppState = {
  user: null,
  loading: false,
  error: null,
};

const appSlice = createSlice({
  name: 'app',
  initialState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
      state.error = null;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string>) => {
      state.error = action.payload;
      state.loading = false;
    },
  },
});

// ❌ Bad
const app = createSlice({
  name: 'app',
  initialState: initialState,
  reducers: {
    setOperator: (state, action) => {
      state.operatorInfo = action.payload;
    },
  },
});
```

# Hooks Guidelines

**Custome Hooks**

```
// ✅ Good
interface UseApiHookReturn<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => Promise<void>;
}

const useApiHook = <T>(apiCall: () => Promise<T>): UseApiHookReturn<T> => {
  const { t } = useTranslation();
  const { start, finish } = useLoading();
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    try {
      setLoading(true);
      start();
      const result = await apiCall();
      setData(result);
      setError(null);
    } catch (err) {
      const errorMessage = errorHandler(t, err);
      setError(errorMessage);
    } finally {
      setLoading(false);
      finish();
    }
  }, [apiCall, t, start, finish]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return { data, loading, error, refetch: fetchData };
};

// ❌ Bad
const useApiHook = (apiCall) => {
  // No TypeScript
  const [data, setData] = useState();
  const [loading, setLoading] = useState();
  const [error, setError] = useState();

  const fetchData = async () => { // Not memoized
    setLoading(true);
    try {
      const result = await apiCall();
      setData(result);
      setLoading(false); // Not in finally
    } catch (err) {
      setError(err.message); // No error checking
      setLoading(false);
    }
  };

   useEffect(() => {
    fetchData();
  }, []); // Missing dependency

  return [data, loading, error, fetchData]; // Array instead of object
};
```

**useCallback and useMemo**

```
// ✅ Good
const UserList = ({ users }: { users: User[] }) => {
  const [searchTerm, setSearchTerm] = useState('');
  const [sortBy, setSortBy] = useState<'name' | 'email'>('name');

  const filteredAndSortedUsers = useMemo(() => {
    return users
      .filter(user =>
        user.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
        user.email.toLowerCase().includes(searchTerm.toLowerCase())
      )
      .sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
  }, [users, searchTerm, sortBy]);

  const handleUserSelect = useCallback((user: User) => {
    console.log('Selected user:', user);
  }, []);

  const handleSearchChange = useCallback((e: React.ChangeEvent<HTMLInputElement>) => {
    setSearchTerm(e.target.value);
  }, []);

  return (
    <div>
      <input
        value={searchTerm}
        onChange={handleSearchChange}
        placeholder="Search users..."
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value as 'name' | 'email')}>
        <option value="name">Sort by Name</option>
        <option value="email">Sort by Email</option>
      </select>
      {filteredAndSortedUsers.map(user => (
        <UserCard key={user.id} user={user} onSelect={handleUserSelect} />
      ))}
    </div>
  );
};

// ❌ Bad
const UserList = ({ users }) => {
  const [searchTerm, setSearchTerm] = useState('');
  // No memoization recalculates every render
  const filteredUsers = users.filter(user =>
    user.name.toLowerCase().includes(searchTerm.toLowerCase())
  );

  return (
    <div>
      <input
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
      />
      {filteredUsers.map(user => (
        <UserCard
          key={user.id}
                    user={user}
          onSelect={(user) => console.log('Selected:', user)} // New function every render
        />
      ))}
    </div>
  );
};
```

# Error Handling

**Try-catch in Async Functions**

```
// ✅ Good
const useUserOperations = () => {
  const { t } = useTranslation();
  const { start, finish } = useLoading();
  const { showError, showSuccess } = useNotificationHook();

  const createUser = async (userData: UserFormData): Promise<User | null> => {
    try {
      start();
      const newUser = await userService.create(userData);
      showSuccess(t('user.created.success'));
      return newUser;
    } catch (error) {
      errorHandler(t, error);
      return null;
    } finally {
      finish();
    }
  };

  const updateUser = async (
    id: string,
    userData: Partial<UserFormData>,
  ): Promise<boolean> => {
    try {
      start();
      await userService.update(id, userData);
      showSuccess(t('user.updated.success'));
      return true;
    } catch (error) {
      const errorMessage = errorHandler(t, error);
      showError([errorMessage]);
      return false;
    } finally {
      finish();
    }
  };

  return { createUser, updateUser };
};

// ❌ Bad
const useUserOperations = () => {
  const [loading, setLoading] = useState(false);

  const createUser = async (userData) => {
    // No typing
    setLoading(true);
    const newUser = await userService.create(userData); // No error handling
    setLoading(false);
    return newUser;
    // No error state, no finally block
  };

return { createUser, loading };
};
```

# Performance

**Lazy Loading**

```
// ✅ Good
const UserProfile = lazy(() => import('./components/UserProfile'));
const UserSettings = lazy(() => import('./components/UserSettings'));

const App = () => (
  <Router>
    <Routes>
      <Route
        path="/profile"
        element={
          <Suspense fallback={<div>Loading profile...</div>}>
            <UserProfile />
          </Suspense>
        }
      />
      <Route
        path="/settings"
        element={
          <Suspense fallback={<div>Loading settings...</div>}>
            <UserSettings />
          </Suspense>
        }
      />
    </Routes>
  </Router>
);

// ❌ Bad
import UserProfile from './components/UserProfile'; // Direct import

const App = () => (
  <Router>
    <Routes>
      <Route path="/profile" element={<UserProfile />} /> {/* No lazy loading */}
    </Routes>
  </Router>
);
```

# React.memo for Components

```
// ✅ Good
interface UserCardProps {
  user: User;
  onEdit: (user: User) => void;
  onDelete: (user: User) => void;
}

const UserCard = memo(( { user, onEdit, onDelete }: UserCardProps) => {
  const handleEdit = useCallback(() => onEdit(user), [onEdit, user]);
  const handleDelete = useCallback(() => onDelete(user), [onDelete, user]);

  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="actions">
        <button onClick={handleEdit}>Edit</button>
        <button onClick={handleDelete}>Delete</button>
      </div>
    </div>
  );
});

// ❌ Bad
const UserCard = ({ user, onEdit, onDelete }) => { // No memo, no typing
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      <div className="actions">
        <button onClick={() => onEdit(user)}>Edit</button> // New function every render
        <button onClick={() => onDelete(user)}>Delete</button> // New function every render
      </div>
    </div>
  );
};
// Component re-renders even when prop haven't changed
```

> useEffect - Gọi an toàn
>
> - Không gọi API trực tiếp trong useEffect nếu không xử lý Clean
> - Tránh để function fetch lặp lại nhiều lần không cần thiết, nên dùng useCallback nếu có dependency
> - Đảm baor không cập nhật state khi component đã unmount

# Code Organization

**Service Layer**

```
// ✅ Good
class UserService {
  async create(userData: UserFormData): Promise<Response> {
    return httpService.post<Response>(this.baseUrl, userData);
  }

  async getById(id: string) {
    // ...
  }

  async search(filters: UserSearchFilters) {
    // ...
  }

  async updateById(id: string, userData: Partial<UserFormData>) {
    // ...
  }

  async deleteById(id: string) {
    // ...
  }
}

const UserService = new UserService();
export default UserService;

// ❌ Bad
export const getUsers = async () => {
  // No class organization
  const response = await fetch('/api/users');
  return response.json(); // No error handling
};

export const createUser = async (userData) => {
  // No typing
  fetch('/api/users', {
    // No await
    method: 'POST',
    body: userData, // No JSON.stringify
  });
};
```

# Screen Development Guide

**Quy trình phát triển màn hình**

1. Setup Structure

```
src/pages/user-management/
├── index.tsx                 # Main page component
├── useUserManagementHook.ts  # Screen-specific hooks
├── components/               # Screen-specific components
├── user-management.type.ts   # Screen-specific types
└── user-management.const.ts  # Screen constants
```

> Trước khi làm xem màn đang làm có thuộc module nào không
> VD: màn C101-4 thuộc module của màn C101-1

2. Define Types

Các type dùng chung sẽ định nghĩa ở trong thư mục shared/types/,request,response từ API sẽ thêm ... Request, ...Response

```
// ✅ Good
interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  role: UserRole;
  status: UserStatus;
  createdAt: Date;
  updatedAt: Date;
}

type UserRole = 'admin' | 'manager' | 'user';
type UserStatus = 'active' | 'inactive' | 'pending' | 'suspended';
type CreateUserRequest = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
type UpdateUserRequest = Partial<CreateUserRequest>;

interface UserListFilters {
  search?: string;
  role?: UserRole;
  status?: UserStatus;
  page: number;
  limit: number;
}

interface UserListResponse {
  data: User[];
  total: number;
  page: number;
  limit: number;
}

// ❌ Bad
interface User {
  id: any; // No specific typing
  email: string;
  name: string; // Should be firstName/lastName
  role: string; // Should be union type
  status: string;
}

type UserData = any; // Too generic
```

3. Component Structure

```
/**
 * UserManagementPage
 *
 * C101-1 Ten man hinh
 */
const UserManagementPage = () => {
  const {
    users,
    loading,
    filters,
    pagination,
    handlePaginationChange
  } = useUserManagementHook();

  return (
    <BasePage>
      {() => (
        <>
          <PageHeader
            title="XXX"
            onCreate={handleCreate}
          />
          <UserFilters
            filters={filters}
            onSearch={handleSearch}
            onReset={handleReset}
          />
          <UserTable
            data={users}
            loading={loading}
            pagination={pagination}
            onEdit={handleEdit}
            onDelete={handleDelete}
            onPaginationChange={handlePaginationChange}
          />
        </>
      )}
    </BasePage>
  );
};

export default UserManagementPage;
```

4. API Service

```
class _UserService {
  search(payload: Request) {
    return httpService.post<Response, Request>(
      '/search',
      { body: ... },
    );
  }
}

const userService = new _UserService();
export default userService;
```

5. Business logic

- Các hàm chỉ dùng trong business logic thì thêm \_ trước mỗi hàm
- Comment rõ ràng với tài liệu VD

```
//1. 初期表示
  useEffect(() => {
    if (navigationType === 'POP') {
      const tempData = store.getState().appRoot.tempData;
      if (tempData) {
        const data = tempData as KeepSearchData;
        /**
        * パラメータreSearchFlagがfalseの場合、画面項目定義の初期値の通りに画面を表示する。
        * パラメータreSearchFlagがtureの場合、保存された検索条件を画面に再表示し、検索条件によりの再検索を行う。
        */
      if (data.key === CUSTOMER_MANAGEMENT_JP_KEY && data.shouldSetData) {
       form.setFieldsValue(data.data);
       const payload: KeepSearchData = {
        shouldSetData: false,
        ...data,
       };
      dispatch(setTempData(payload));
     }
   }
  }
}, []);

```

```
/**
 * useUserManagementHook
 *
 * API: /api/users
 */
const useUserManagementHook = () => {
  const { t } = useTranslation();
  const { start, finish } = useLoading();
  const [users, setUsers] = useState<User[]>([]);
  const [pagination, setPagination] = useState<PaginationInfo>(DEFAULT_PAGING);

  const handleSearch = async () => {
    try {
      start();
      const response = await userService.search(filters);
      if (response.data) {
        setUsers(response.data);
        setPagination(response.pagination);
      }
    } catch (error) {
      errorHandler(t, error);
    } finally {
      finish();
    }
  };

  return { pagination, users };
};
```
