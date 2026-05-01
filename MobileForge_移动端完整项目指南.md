# MobileForge 移动端完整项目指南

> 基于 React Native + Node.js + Firebase 的任务追踪应用

---

## 📋 项目介绍

### 项目概述
MobileForge 是一个跨平台的任务追踪移动应用，支持 iOS 和 Android。项目展示了移动端开发的核心技能，包括原生模块调用、离线存储、推送通知等。

- **跨平台**：React Native 一套代码，支持 iOS/Android
- **实时同步**：Firebase 实时数据库
- **离线支持**：本地数据持久化
- **面试亮点**：移动端全栈能力

### 核心技术亮点
```
移动端：React Native 0.72 + TypeScript + Expo
后端：Node.js + Express + Firebase Admin
数据库：Firebase Firestore + Storage
推送：Firebase Cloud Messaging
认证：Firebase Auth
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                     移动端 (Mobile Client)                       │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                     React Native App                         │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │ │
│  │  │  屏幕    │  │  组件    │  │  状态    │  │  服务    │    │ │
│  │  │ Screens  │  │Components│  │ Zustand  │  │ Services │    │ │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    │ │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │ │
│  │  │  导航    │  │  存储    │  │  推送    │                  │ │
│  │  │ React    │  │ Async    │  │ Notif.   │                  │ │
│  │  │ Navigation│ │ Storage  │  │ Service  │                  │ │
│  │  └──────────┘  └──────────┘  └──────────┘                  │ │
│  └─────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                           │
                    Firebase SDK
                           │
┌──────────────────────────┼────────────────────────────────────┐
│                     Firebase 平台                               │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │
│  │  Firestore   │ │    Auth      │ │   Storage    │           │
│  │  (实时数据库) │ │   (认证)     │ │   (文件存储)  │           │
│  └──────────────┘ └──────────────┘ └──────────────┘           │
│  ┌──────────────┐ ┌──────────────┐                             │
│  │    FCM       │ │ Cloud Func.  │                             │
│  │ (推送通知)   │ │ (云函数)     │                             │
│  └──────────────┘ └──────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

### 数据流架构

```
┌─────────────┐     用户操作      ┌─────────────┐
│   Screen    │ ───────────────> │  Zustand    │
│  (UI 组件)  │                  │  (状态管理)  │
└─────────────┘                  └──────┬──────┘
       ^                                  │
       │                                  │
       │                                  ▼
       │                          ┌─────────────┐
       │                          │  Firebase  │
       │                          │   SDK      │
       │                          └──────┬──────┘
       │                                 │
       │              ┌─────────────────┼─────────────────┐
       │              │                 │                 │
       ▼              ▼                 ▼                 ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Firestore   │ │    Auth      │ │   Storage   │ │    FCM      │
│ (任务数据)   │ │ (用户认证)   │ │ (图片存储)   │ │ (推送通知)  │
└─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘
```

---

## 📅 学习计划（7天）

### Day 1：React Native 环境搭建 ⭐

**目标**：搭建完整的 React Native 开发环境

**学习内容**：
- React Native vs Expo 选择
- 开发环境配置
- 项目创建与运行
- 调试工具使用

**产出**：可运行在模拟器上的空白项目

**执行命令**：

```bash
# 1. 检查环境
node -v  # >= 18
npm -v   # >= 8
java -v  # JDK 17

# 2. 安装 Expo CLI
npm install -g expo-cli

# 3. 创建项目（推荐使用 Expo）
npx create-expo-app MobileForge --template blank-typescript

# 4. 进入项目目录
cd MobileForge

# 5. 安装核心依赖
npx expo install expo-router expo-secure-store expo-notifications
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install @tanstack/react-query zustand
npx expo install firebase

# 6. 安装 UI 库
npx expo install react-native-paper
npx expo install @expo/vector-icons
npx expo install react-native-gesture-handler react-native-reanimated
npx expo install react-native-safe-area-context

# 7. 启动开发服务器
npx expo start
```

---

### Day 2：Firebase 配置与认证 ⭐⭐

**目标**：完成 Firebase 配置和用户认证

**学习内容**：
- Firebase Console 配置
- Firebase SDK 初始化
- 邮箱/Google 登录
- 认证状态管理

**产出**：完整的用户认证系统

**核心代码**：

`MobileForge/src/config/firebase.ts` - Firebase 配置
```typescript
import { initializeApp, getApps, FirebaseApp } from 'firebase/app';
import {
  getAuth,
  Auth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  signOut as firebaseSignOut,
  onAuthStateChanged,
  GoogleAuthProvider,
  signInWithPopup,
  User
} from 'firebase/auth';
import {
  getFirestore,
  Firestore,
  doc,
  getDoc,
  setDoc,
  updateDoc,
  deleteDoc,
  collection,
  query,
  where,
  orderBy,
  onSnapshot,
  addDoc,
  serverTimestamp,
  Timestamp,
  DocumentData,
  QueryConstraint
} from 'firebase/firestore';
import {
  getStorage,
  Ref,
  uploadBytes,
  getDownloadURL,
  deleteObject
} from 'firebase/storage';

// Firebase 配置 - 请替换为您自己的配置
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

// 初始化 Firebase（防止重复初始化）
let app: FirebaseApp;
let auth: Auth;
let db: Firestore;
let storage: ReturnType<typeof getStorage>;

if (!getApps().length) {
  app = initializeApp(firebaseConfig);
  auth = getAuth(app);
  db = getFirestore(app);
  storage = getStorage(app);
} else {
  app = getApps()[0];
  auth = getAuth(app);
  db = getFirestore(app);
  storage = getStorage(app);
}

export { app, auth, db, storage };
```

`MobileForge/src/services/authService.ts` - 认证服务
```typescript
import {
  auth,
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
  firebaseSignOut,
  GoogleAuthProvider,
  signInWithPopup,
  onAuthStateChanged,
  User
} from '../config/firebase';
import { getFirestore, doc, setDoc, getDoc } from 'firebase/firestore';

export interface AuthState {
  user: User | null;
  loading: boolean;
  initialized: boolean;
}

// 邮箱登录
export async function signInWithEmail(
  email: string,
  password: string
): Promise<{ success: boolean; error?: string }> {
  try {
    const result = await signInWithEmailAndPassword(auth, email, password);
    console.log('登录成功:', result.user.email);
    return { success: true };
  } catch (error: any) {
    const errorMessage = getAuthErrorMessage(error.code);
    console.error('登录失败:', errorMessage);
    return { success: false, error: errorMessage };
  }
}

// 邮箱注册
export async function signUpWithEmail(
  email: string,
  password: string,
  username: string
): Promise<{ success: boolean; error?: string }> {
  try {
    const result = await createUserWithEmailAndPassword(auth, email, password);
    
    // 在 Firestore 创建用户文档
    await createUserDocument(result.user, { username });
    
    console.log('注册成功:', result.user.email);
    return { success: true };
  } catch (error: any) {
    const errorMessage = getAuthErrorMessage(error.code);
    console.error('注册失败:', errorMessage);
    return { success: false, error: errorMessage };
  }
}

// Google 登录
export async function signInWithGoogle(): Promise<{
  success: boolean;
  error?: string;
}> {
  try {
    const provider = new GoogleAuthProvider();
    const result = await signInWithPopup(auth, provider);
    
    // 如果是新用户，创建文档
    await createUserDocument(result.user, {
      username: result.user.displayName || 'User'
    });
    
    console.log('Google 登录成功:', result.user.email);
    return { success: true };
  } catch (error: any) {
    const errorMessage = getAuthErrorMessage(error.code);
    console.error('Google 登录失败:', errorMessage);
    return { success: false, error: errorMessage };
  }
}

// 登出
export async function signOut(): Promise<void> {
  try {
    await firebaseSignOut(auth);
    console.log('登出成功');
  } catch (error) {
    console.error('登出失败:', error);
  }
}

// 监听认证状态变化
export function onAuthChange(callback: (user: User | null) => void): () => void {
  return onAuthStateChanged(auth, callback);
}

// 创建用户文档
async function createUserDocument(
  user: User,
  additionalData: { username: string }
): Promise<void> {
  const userRef = doc(db, 'users', user.uid);
  const userSnap = await getDoc(userRef);
  
  if (!userSnap.exists()) {
    await setDoc(userRef, {
      uid: user.uid,
      email: user.email,
      username: additionalData.username,
      avatar: user.photoURL,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp()
    });
  }
}

// 获取用户文档
export async function getUserDocument(uid: string): Promise<DocumentData | null> {
  const userRef = doc(db, 'users', uid);
  const userSnap = await getDoc(userRef);
  
  if (userSnap.exists()) {
    return userSnap.data();
  }
  return null;
}

// 错误消息映射
function getAuthErrorMessage(code: string): string {
  const errorMessages: Record<string, string> = {
    'auth/email-already-in-use': '该邮箱已被注册',
    'auth/invalid-email': '邮箱格式不正确',
    'auth/operation-not-allowed': '该登录方式未启用',
    'auth/weak-password': '密码强度太弱',
    'auth/user-disabled': '用户已被禁用',
    'auth/user-not-found': '用户不存在',
    'auth/wrong-password': '密码错误',
    'auth/too-many-requests': '请求过于频繁，请稍后再试',
    'auth/popup-closed-by-user': '登录窗口已关闭',
    'auth/network-request-failed': '网络请求失败'
  };
  
  return errorMessages[code] || '认证失败，请重试';
}
```

`MobileForge/src/hooks/useAuth.ts` - 认证 Hook
```typescript
import { useEffect, useState, useCallback } from 'react';
import { User } from 'firebase/auth';
import {
  signInWithEmail,
  signUpWithEmail,
  signInWithGoogle,
  signOut,
  onAuthChange
} from '../services/authService';

export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [initialized, setInitialized] = useState(false);

  useEffect(() => {
    // 监听认证状态变化
    const unsubscribe = onAuthChange((firebaseUser) => {
      setUser(firebaseUser);
      setLoading(false);
      setInitialized(true);
    });

    return () => unsubscribe();
  }, []);

  const login = useCallback(async (email: string, password: string) => {
    setLoading(true);
    const result = await signInWithEmail(email, password);
    setLoading(false);
    return result;
  }, []);

  const register = useCallback(async (email: string, password: string, username: string) => {
    setLoading(true);
    const result = await signUpWithEmail(email, password, username);
    setLoading(false);
    return result;
  }, []);

  const loginWithGoogle = useCallback(async () => {
    setLoading(true);
    const result = await signInWithGoogle();
    setLoading(false);
    return result;
  }, []);

  const logout = useCallback(async () => {
    setLoading(true);
    await signOut();
    setLoading(false);
  }, []);

  return {
    user,
    loading,
    initialized,
    isAuthenticated: !!user,
    login,
    register,
    loginWithGoogle,
    logout
  };
}
```

---

### Day 3：Firestore 数据模型 ⭐⭐

**目标**：设计并实现 Firestore 数据模型

**学习内容**：
- Firestore 数据结构设计
- 实时数据同步
- 离线支持
- 性能优化

**产出**：完整的任务数据操作

**核心代码**：

`MobileForge/src/services/taskService.ts` - 任务服务
```typescript
import {
  db,
  collection,
  doc,
  query,
  where,
  orderBy,
  onSnapshot,
  addDoc,
  updateDoc,
  deleteDoc,
  getDocs,
  getDoc,
  serverTimestamp,
  Timestamp,
  DocumentData,
  QueryConstraint,
  writeBatch
} from '../config/firebase';
import { User } from 'firebase/auth';

// 任务优先级
export enum TaskPriority {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  URGENT = 'urgent'
}

// 任务状态
export enum TaskStatus {
  TODO = 'todo',
  IN_PROGRESS = 'in_progress',
  DONE = 'done',
  CANCELLED = 'cancelled'
}

// 任务类型
export interface Task {
  id: string;
  title: string;
  description?: string;
  status: TaskStatus;
  priority: TaskPriority;
  dueDate?: Timestamp;
  completedAt?: Timestamp;
  estimatedMinutes?: number;
  actualMinutes?: number;
  projectId?: string;
  tags: string[];
  attachments: string[];
  subtasks: SubTask[];
  createdAt: Timestamp;
  updatedAt: Timestamp;
  createdBy: string;
  assignedTo?: string;
}

// 子任务
export interface SubTask {
  id: string;
  title: string;
  completed: boolean;
}

// 任务输入
export interface TaskInput {
  title: string;
  description?: string;
  priority?: TaskPriority;
  dueDate?: Date;
  estimatedMinutes?: number;
  projectId?: string;
  tags?: string[];
}

// 项目类型
export interface Project {
  id: string;
  name: string;
  color: string;
  icon: string;
  description?: string;
  createdAt: Timestamp;
  createdBy: string;
  memberIds: string[];
}

// ============= 任务操作 =============

// 创建任务
export async function createTask(
  userId: string,
  taskInput: TaskInput
): Promise<{ success: boolean; taskId?: string; error?: string }> {
  try {
    const tasksRef = collection(db, 'tasks');
    const taskData = {
      ...taskInput,
      status: TaskStatus.TODO,
      tags: taskInput.tags || [],
      attachments: [],
      subtasks: [],
      dueDate: taskInput.dueDate
        ? Timestamp.fromDate(taskInput.dueDate)
        : null,
      createdBy: userId,
      createdAt: serverTimestamp(),
      updatedAt: serverTimestamp()
    };

    const docRef = await addDoc(tasksRef, taskData);
    console.log('任务创建成功:', docRef.id);
    return { success: true, taskId: docRef.id };
  } catch (error: any) {
    console.error('创建任务失败:', error);
    return { success: false, error: error.message };
  }
}

// 更新任务
export async function updateTask(
  taskId: string,
  updates: Partial<TaskInput & { status: TaskStatus; completedAt?: Date }>
): Promise<{ success: boolean; error?: string }> {
  try {
    const taskRef = doc(db, 'tasks', taskId);
    
    // 处理日期
    const formattedUpdates = {
      ...updates,
      updatedAt: serverTimestamp()
    };
    
    if (updates.dueDate) {
      formattedUpdates.dueDate = Timestamp.fromDate(updates.dueDate);
    }
    
    if (updates.completedAt) {
      formattedUpdates.completedAt = Timestamp.fromDate(updates.completedAt);
    }

    await updateDoc(taskRef, formattedUpdates);
    console.log('任务更新成功:', taskId);
    return { success: true };
  } catch (error: any) {
    console.error('更新任务失败:', error);
    return { success: false, error: error.message };
  }
}

// 删除任务
export async function deleteTask(
  taskId: string
): Promise<{ success: boolean; error?: string }> {
  try {
    const taskRef = doc(db, 'tasks', taskId);
    await deleteDoc(taskRef);
    console.log('任务删除成功:', taskId);
    return { success: true };
  } catch (error: any) {
    console.error('删除任务失败:', error);
    return { success: false, error: error.message };
  }
}

// 获取用户的所有任务（实时）
export function subscribeToUserTasks(
  userId: string,
  callback: (tasks: Task[]) => void,
  constraints?: QueryConstraint[]
): () => void {
  const tasksRef = collection(db, 'tasks');
  
  // 默认查询：按创建时间倒序
  const defaultConstraints: QueryConstraint[] = [
    where('createdBy', '==', userId),
    orderBy('createdAt', 'desc')
  ];
  
  const q = query(tasksRef, ...(constraints || defaultConstraints));

  const unsubscribe = onSnapshot(
    q,
    (snapshot) => {
      const tasks: Task[] = [];
      snapshot.forEach((doc) => {
        tasks.push({ id: doc.id, ...doc.data() } as Task);
      });
      callback(tasks);
    },
    (error) => {
      console.error('订阅任务失败:', error);
    }
  );

  return unsubscribe;
}

// 获取单个任务
export async function getTask(taskId: string): Promise<Task | null> {
  try {
    const taskRef = doc(db, 'tasks', taskId);
    const taskSnap = await getDoc(taskRef);
    
    if (taskSnap.exists()) {
      return { id: taskSnap.id, ...taskSnap.data() } as Task;
    }
    return null;
  } catch (error) {
    console.error('获取任务失败:', error);
    return null;
  }
}

// 完成任务
export async function completeTask(taskId: string): Promise<boolean> {
  const result = await updateTask(taskId, {
    status: TaskStatus.DONE,
    completedAt: new Date()
  });
  return result.success;
}

// 添加子任务
export async function addSubTask(
  taskId: string,
  subtask: Omit<SubTask, 'id'>
): Promise<boolean> {
  try {
    const taskRef = doc(db, 'tasks', taskId);
    const task = await getTask(taskId);
    
    if (!task) return false;
    
    const newSubtask: SubTask = {
      id: `subtask_${Date.now()}`,
      ...subtask
    };
    
    await updateDoc(taskRef, {
      subtasks: [...task.subtasks, newSubtask],
      updatedAt: serverTimestamp()
    });
    
    return true;
  } catch (error) {
    console.error('添加子任务失败:', error);
    return false;
  }
}

// 切换子任务完成状态
export async function toggleSubTask(
  taskId: string,
  subtaskId: string
): Promise<boolean> {
  try {
    const taskRef = doc(db, 'tasks', taskId);
    const task = await getTask(taskId);
    
    if (!task) return false;
    
    const updatedSubtasks = task.subtasks.map((st) =>
      st.id === subtaskId ? { ...st, completed: !st.completed } : st
    );
    
    await updateDoc(taskRef, {
      subtasks: updatedSubtasks,
      updatedAt: serverTimestamp()
    });
    
    return true;
  } catch (error) {
    console.error('切换子任务失败:', error);
    return false;
  }
}

// 批量删除已完成任务
export async function deleteCompletedTasks(userId: string): Promise<{
  success: boolean;
  deletedCount?: number;
}> {
  try {
    const tasksRef = collection(db, 'tasks');
    const q = query(
      tasksRef,
      where('createdBy', '==', userId),
      where('status', '==', TaskStatus.DONE)
    );
    
    const snapshot = await getDocs(q);
    
    if (snapshot.empty) {
      return { success: true, deletedCount: 0 };
    }
    
    const batch = writeBatch(db);
    let deletedCount = 0;
    
    snapshot.forEach((doc) => {
      batch.delete(doc.ref);
      deletedCount++;
    });
    
    await batch.commit();
    console.log(`批量删除 ${deletedCount} 个已完成任务`);
    
    return { success: true, deletedCount };
  } catch (error: any) {
    console.error('批量删除失败:', error);
    return { success: false };
  }
}
```

`MobileForge/src/stores/taskStore.ts` - 任务状态管理
```typescript
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import {
  Task,
  TaskInput,
  TaskStatus,
  TaskPriority,
  createTask,
  updateTask,
  deleteTask,
  completeTask,
  subscribeToUserTasks,
  deleteCompletedTasks
} from '../services/taskService';

interface TaskState {
  tasks: Task[];
  loading: boolean;
  error: string | null;
  filter: {
    status?: TaskStatus;
    priority?: TaskPriority;
    projectId?: string;
    search?: string;
  };
  
  // Actions
  setTasks: (tasks: Task[]) => void;
  addTask: (taskInput: TaskInput) => Promise<string | null>;
  editTask: (taskId: string, updates: Partial<TaskInput>) => Promise<boolean>;
  removeTask: (taskId: string) => Promise<boolean>;
  toggleComplete: (taskId: string) => Promise<boolean>;
  setFilter: (filter: Partial<TaskState['filter']>) => void;
  clearCompleted: () => Promise<number>;
  subscribe: (userId: string) => () => void;
  
  // Computed
  getFilteredTasks: () => Task[];
  getTasksByStatus: (status: TaskStatus) => Task[];
  getTodayTasks: () => Task[];
  getOverdueTasks: () => Task[];
}

export const useTaskStore = create<TaskState>()(
  persist(
    (set, get) => ({
      tasks: [],
      loading: false,
      error: null,
      filter: {},
      
      setTasks: (tasks) => set({ tasks }),
      
      addTask: async (taskInput) => {
        // 这个方法在 subscribe 模式下通过 userId 获取
        // 本地创建临时任务用于乐观更新
        const tempTask: Task = {
          id: `temp_${Date.now()}`,
          ...taskInput,
          status: TaskStatus.TODO,
          tags: taskInput.tags || [],
          attachments: [],
          subtasks: [],
          createdBy: '',
          createdAt: { seconds: Date.now() / 1000 } as any,
          updatedAt: { seconds: Date.now() / 1000 } as any
        };
        
        set((state) => ({
          tasks: [tempTask, ...state.tasks]
        }));
        
        return tempTask.id;
      },
      
      editTask: async (taskId, updates) => {
        // 乐观更新
        set((state) => ({
          tasks: state.tasks.map((task) =>
            task.id === taskId
              ? { ...task, ...updates, updatedAt: { seconds: Date.now() / 1000 } as any }
              : task
          )
        }));
        
        const result = await updateTask(taskId, updates);
        
        if (!result.success) {
          // 回滚
          set((state) => ({
            tasks: state.tasks.filter((t) => t.id !== taskId) // 简单回滚
          }));
        }
        
        return result.success;
      },
      
      removeTask: async (taskId) => {
        const previousTasks = get().tasks;
        
        // 乐观删除
        set((state) => ({
          tasks: state.tasks.filter((t) => t.id !== taskId)
        }));
        
        const result = await deleteTask(taskId);
        
        if (!result.success) {
          // 回滚
          set({ tasks: previousTasks });
        }
        
        return result.success;
      },
      
      toggleComplete: async (taskId) => {
        const task = get().tasks.find((t) => t.id === taskId);
        if (!task) return false;
        
        const newStatus =
          task.status === TaskStatus.DONE ? TaskStatus.TODO : TaskStatus.DONE;
        
        // 乐观更新
        set((state) => ({
          tasks: state.tasks.map((t) =>
            t.id === taskId
              ? {
                  ...t,
                  status: newStatus,
                  completedAt:
                    newStatus === TaskStatus.DONE
                      ? { seconds: Date.now() / 1000 } as any
                      : undefined
                }
              : t
          )
        }));
        
        const result = await updateTask(taskId, {
          status: newStatus,
          completedAt: newStatus === TaskStatus.DONE ? new Date() : undefined
        });
        
        return result.success;
      },
      
      setFilter: (filter) =>
        set((state) => ({
          filter: { ...state.filter, ...filter }
        })),
      
      clearCompleted: async () => {
        const completedTasks = get().tasks.filter(
          (t) => t.status === TaskStatus.DONE
        );
        
        // 乐观删除
        set((state) => ({
          tasks: state.tasks.filter((t) => t.status !== TaskStatus.DONE)
        }));
        
        const result = await deleteCompletedTasks(
          completedTasks[0]?.createdBy || ''
        );
        
        if (!result.success) {
          // 回滚（简化处理）
          set((state) => ({ tasks: [...state.tasks, ...completedTasks] }));
        }
        
        return result.deletedCount || 0;
      },
      
      subscribe: (userId) => {
        set({ loading: true });
        
        const unsubscribe = subscribeToUserTasks(userId, (tasks) => {
          set({ tasks, loading: false, error: null });
        });
        
        return unsubscribe;
      },
      
      getFilteredTasks: () => {
        const { tasks, filter } = get();
        
        return tasks.filter((task) => {
          if (filter.status && task.status !== filter.status) return false;
          if (filter.priority && task.priority !== filter.priority) return false;
          if (filter.projectId && task.projectId !== filter.projectId) return false;
          if (filter.search) {
            const searchLower = filter.search.toLowerCase();
            if (
              !task.title.toLowerCase().includes(searchLower) &&
              !task.description?.toLowerCase().includes(searchLower)
            ) {
              return false;
            }
          }
          return true;
        });
      },
      
      getTasksByStatus: (status) => {
        return get().tasks.filter((task) => task.status === status);
      },
      
      getTodayTasks: () => {
        const today = new Date();
        today.setHours(0, 0, 0, 0);
        const tomorrow = new Date(today);
        tomorrow.setDate(tomorrow.getDate() + 1);
        
        return get().tasks.filter((task) => {
          if (!task.dueDate) return false;
          const dueDate = task.dueDate.toDate();
          return dueDate >= today && dueDate < tomorrow;
        });
      },
      
      getOverdueTasks: () => {
        const now = new Date();
        
        return get().tasks.filter((task) => {
          if (!task.dueDate) return false;
          if (task.status === TaskStatus.DONE) return false;
          return task.dueDate.toDate() < now;
        });
      }
    }),
    {
      name: 'task-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({
        filter: state.filter
        // 不持久化 tasks（由 Firestore 管理）
      })
    }
  )
);
```

---

### Day 4：UI 组件开发 ⭐⭐

**目标**：实现核心 UI 组件

**学习内容**：
- React Native 组件编写
- 导航配置
- 列表组件
- 表单组件

**产出**：完整的页面组件

**核心代码**：

`MobileForge/src/components/TaskCard.tsx` - 任务卡片组件
```typescript
import React from 'react';
import {
  View,
  Text,
  StyleSheet,
  TouchableOpacity,
  Pressable
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { Task, TaskStatus, TaskPriority } from '../services/taskService';
import { formatDistanceToNow } from '../utils/date';

interface TaskCardProps {
  task: Task;
  onPress: () => void;
  onComplete: () => void;
  onDelete: () => void;
}

const priorityColors: Record<TaskPriority, string> = {
  [TaskPriority.LOW]: '#10B981',
  [TaskPriority.MEDIUM]: '#3B82F6',
  [TaskPriority.HIGH]: '#F59E0B',
  [TaskPriority.URGENT]: '#EF4444'
};

const statusIcons: Record<TaskStatus, keyof typeof Ionicons.glyphMap> = {
  [TaskStatus.TODO]: 'ellipse-outline',
  [TaskStatus.IN_PROGRESS]: 'ellipse',
  [TaskStatus.DONE]: 'checkmark-circle',
  [TaskStatus.CANCELLED]: 'close-circle'
};

export function TaskCard({
  task,
  onPress,
  onComplete,
  onDelete
}: TaskCardProps) {
  const isCompleted = task.status === TaskStatus.DONE;
  const isOverdue =
    task.dueDate &&
    !isCompleted &&
    task.dueDate.toDate() < new Date();

  return (
    <Pressable
      style={({ pressed }) => [
        styles.container,
        pressed && styles.pressed,
        isCompleted && styles.completed
      ]}
      onPress={onPress}
    >
      <TouchableOpacity
        style={styles.checkbox}
        onPress={onComplete}
        hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
      >
        <Ionicons
          name={statusIcons[task.status]}
          size={24}
          color={isCompleted ? '#10B981' : isOverdue ? '#EF4444' : '#9CA3AF'}
        />
      </TouchableOpacity>

      <View style={styles.content}>
        <Text
          style={[styles.title, isCompleted && styles.titleCompleted]}
          numberOfLines={1}
        >
          {task.title}
        </Text>

        {task.description && (
          <Text style={styles.description} numberOfLines={2}>
            {task.description}
          </Text>
        )}

        <View style={styles.meta}>
          <View style={styles.tags}>
            <View
              style={[
                styles.priorityBadge,
                { backgroundColor: priorityColors[task.priority] }
              ]}
            >
              <Text style={styles.priorityText}>
                {task.priority.toUpperCase()}
              </Text>
            </View>

            {task.tags.slice(0, 2).map((tag, index) => (
              <View key={index} style={styles.tag}>
                <Text style={styles.tagText}>{tag}</Text>
              </View>
            ))}
          </View>

          {task.dueDate && (
            <View style={[styles.dueDate, isOverdue && styles.dueDateOverdue]}>
              <Ionicons
                name="calendar-outline"
                size={12}
                color={isOverdue ? '#EF4444' : '#9CA3AF'}
              />
              <Text style={[styles.dueDateText, isOverdue && styles.dueDateTextOverdue]}>
                {formatDistanceToNow(task.dueDate.toDate())}
              </Text>
            </View>
          )}
        </View>

        {task.subtasks.length > 0 && (
          <View style={styles.subtasks}>
            <Ionicons name="list-outline" size={14} color="#9CA3AF" />
            <Text style={styles.subtasksText}>
              {task.subtasks.filter((st) => st.completed).length}/
              {task.subtasks.length}
            </Text>
          </View>
        )}
      </View>

      <TouchableOpacity
        style={styles.deleteButton}
        onPress={onDelete}
        hitSlop={{ top: 10, bottom: 10, left: 10, right: 10 }}
      >
        <Ionicons name="trash-outline" size={20} color="#EF4444" />
      </TouchableOpacity>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    backgroundColor: '#FFFFFF',
    borderRadius: 12,
    padding: 16,
    marginHorizontal: 16,
    marginVertical: 6,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 8,
    elevation: 2
  },
  pressed: {
    opacity: 0.9,
    transform: [{ scale: 0.98 }]
  },
  completed: {
    opacity: 0.7,
    backgroundColor: '#F9FAFB'
  },
  checkbox: {
    marginRight: 12,
    marginTop: 2
  },
  content: {
    flex: 1
  },
  title: {
    fontSize: 16,
    fontWeight: '600',
    color: '#111827',
    marginBottom: 4
  },
  titleCompleted: {
    textDecorationLine: 'line-through',
    color: '#9CA3AF'
  },
  description: {
    fontSize: 14,
    color: '#6B7280',
    marginBottom: 8,
    lineHeight: 20
  },
  meta: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'space-between'
  },
  tags: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 6
  },
  priorityBadge: {
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 4
  },
  priorityText: {
    fontSize: 10,
    fontWeight: '700',
    color: '#FFFFFF'
  },
  tag: {
    backgroundColor: '#F3F4F6',
    paddingHorizontal: 8,
    paddingVertical: 2,
    borderRadius: 4
  },
  tagText: {
    fontSize: 11,
    color: '#6B7280'
  },
  dueDate: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 4
  },
  dueDateOverdue: {
    backgroundColor: '#FEE2E2',
    paddingHorizontal: 6,
    paddingVertical: 2,
    borderRadius: 4
  },
  dueDateText: {
    fontSize: 12,
    color: '#9CA3AF'
  },
  dueDateTextOverdue: {
    color: '#EF4444'
  },
  subtasks: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: 4,
    marginTop: 8
  },
  subtasksText: {
    fontSize: 12,
    color: '#9CA3AF'
  },
  deleteButton: {
    padding: 4
  }
});
```

`MobileForge/src/screens/TaskListScreen.tsx` - 任务列表页
```typescript
import React, { useEffect, useCallback, useState } from 'react';
import {
  View,
  Text,
  StyleSheet,
  FlatList,
  RefreshControl,
  TouchableOpacity,
  Alert
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { useNavigation } from '@react-navigation/native';
import { NativeStackNavigationProp } from '@react-navigation/native-stack';
import { useTaskStore } from '../stores/taskStore';
import { useAuth } from '../hooks/useAuth';
import { TaskCard } from '../components/TaskCard';
import { TaskStatus, Task } from '../services/taskService';
import { RootStackParamList } from '../navigation/types';

type NavigationProp = NativeStackNavigationProp<RootStackParamList>;

const statusTabs: { key: TaskStatus | 'all'; label: string; icon: keyof typeof Ionicons.glyphMap }[] = [
  { key: 'all', label: '全部', icon: 'list' },
  { key: TaskStatus.TODO, label: '待办', icon: 'ellipse-outline' },
  { key: TaskStatus.IN_PROGRESS, label: '进行中', icon: 'ellipse' },
  { key: TaskStatus.DONE, label: '已完成', icon: 'checkmark-circle' }
];

export function TaskListScreen() {
  const navigation = useNavigation<NavigationProp>();
  const { user } = useAuth();
  const {
    tasks,
    loading,
    filter,
    subscribe,
    setFilter,
    removeTask,
    toggleComplete,
    getFilteredTasks,
    getOverdueTasks
  } = useTaskStore();
  
  const [activeTab, setActiveTab] = useState<TaskStatus | 'all'>('all');
  const [refreshing, setRefreshing] = useState(false);

  // 订阅任务
  useEffect(() => {
    if (user) {
      const unsubscribe = subscribe(user.uid);
      return () => unsubscribe();
    }
  }, [user]);

  // 过滤任务
  const filteredTasks = getFilteredTasks();
  const overdueTasks = getOverdueTasks();

  // 处理下拉刷新
  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    // Firebase 会自动更新，这里只是 UI 反馈
    setTimeout(() => setRefreshing(false), 1000);
  }, []);

  // Tab 切换
  const handleTabChange = (tab: TaskStatus | 'all') => {
    setActiveTab(tab);
    setFilter({ status: tab === 'all' ? undefined : tab });
  };

  // 完成/取消完成
  const handleToggleComplete = async (taskId: string) => {
    await toggleComplete(taskId);
  };

  // 删除任务
  const handleDelete = (task: Task) => {
    Alert.alert(
      '删除任务',
      `确定要删除「${task.title}」吗？`,
      [
        { text: '取消', style: 'cancel' },
        {
          text: '删除',
          style: 'destructive',
          onPress: () => removeTask(task.id)
        }
      ]
    );
  };

  // 添加任务
  const handleAddTask = () => {
    navigation.navigate('CreateTask');
  };

  // 渲染任务
  const renderTask = ({ item }: { item: Task }) => (
    <TaskCard
      task={item}
      onPress={() => navigation.navigate('TaskDetail', { taskId: item.id })}
      onComplete={() => handleToggleComplete(item.id)}
      onDelete={() => handleDelete(item)}
    />
  );

  // 空状态
  const renderEmpty = () => (
    <View style={styles.emptyContainer}>
      <Ionicons name="clipboard-outline" size={64} color="#D1D5DB" />
      <Text style={styles.emptyTitle}>暂无任务</Text>
      <Text style={styles.emptyText}>点击下方按钮创建新任务</Text>
    </View>
  );

  return (
    <SafeAreaView style={styles.container} edges={['top']}>
      {/* 头部 */}
      <View style={styles.header}>
        <View>
          <Text style={styles.greeting}>你好</Text>
          <Text style={styles.userName}>{user?.displayName || '用户'}</Text>
        </View>
        <TouchableOpacity
          style={styles.profileButton}
          onPress={() => navigation.navigate('Profile')}
        >
          {user?.photoURL ? (
            <View style={styles.avatar}>
              <Text style={styles.avatarText}>
                {(user.displayName || user.email || 'U')[0].toUpperCase()}
              </Text>
            </View>
          ) : (
            <Ionicons name="person-circle" size={40} color="#6B7280" />
          )}
        </TouchableOpacity>
      </View>

      {/* 统计卡片 */}
      <View style={styles.statsContainer}>
        <View style={[styles.statCard, styles.statCardPrimary]}>
          <Text style={styles.statNumber}>{tasks.length}</Text>
          <Text style={styles.statLabel}>总任务</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={[styles.statNumber, { color: '#F59E0B' }]}>
            {tasks.filter((t) => t.status !== TaskStatus.DONE).length}
          </Text>
          <Text style={styles.statLabel}>进行中</Text>
        </View>
        <View style={styles.statCard}>
          <Text style={[styles.statNumber, { color: '#EF4444' }]}>
            {overdueTasks.length}
          </Text>
          <Text style={styles.statLabel}>已逾期</Text>
        </View>
      </View>

      {/* Tab 切换 */}
      <View style={styles.tabContainer}>
        {statusTabs.map((tab) => (
          <TouchableOpacity
            key={tab.key}
            style={[styles.tab, activeTab === tab.key && styles.tabActive]}
            onPress={() => handleTabChange(tab.key)}
          >
            <Ionicons
              name={tab.icon}
              size={16}
              color={activeTab === tab.key ? '#3B82F6' : '#9CA3AF'}
            />
            <Text
              style={[
                styles.tabText,
                activeTab === tab.key && styles.tabTextActive
              ]}
            >
              {tab.label}
            </Text>
          </TouchableOpacity>
        ))}
      </View>

      {/* 任务列表 */}
      <FlatList
        data={filteredTasks}
        renderItem={renderTask}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.listContent}
        ListEmptyComponent={renderEmpty}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            colors={['#3B82F6']}
          />
        }
        showsVerticalScrollIndicator={false}
      />

      {/* 添加按钮 */}
      <TouchableOpacity style={styles.fab} onPress={handleAddTask}>
        <Ionicons name="add" size={28} color="#FFFFFF" />
      </TouchableOpacity>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F9FAFB'
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 20,
    paddingVertical: 16
  },
  greeting: {
    fontSize: 14,
    color: '#6B7280'
  },
  userName: {
    fontSize: 24,
    fontWeight: '700',
    color: '#111827'
  },
  profileButton: {
    padding: 4
  },
  avatar: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: '#3B82F6',
    justifyContent: 'center',
    alignItems: 'center'
  },
  avatarText: {
    color: '#FFFFFF',
    fontSize: 18,
    fontWeight: '600'
  },
  statsContainer: {
    flexDirection: 'row',
    paddingHorizontal: 16,
    gap: 12
  },
  statCard: {
    flex: 1,
    backgroundColor: '#FFFFFF',
    borderRadius: 12,
    padding: 16,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 1
  },
  statCardPrimary: {
    backgroundColor: '#3B82F6'
  },
  statNumber: {
    fontSize: 24,
    fontWeight: '700',
    color: '#FFFFFF'
  },
  statLabel: {
    fontSize: 12,
    color: '#6B7280',
    marginTop: 4
  },
  tabContainer: {
    flexDirection: 'row',
    paddingHorizontal: 16,
    marginTop: 20,
    marginBottom: 12,
    gap: 8
  },
  tab: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: 12,
    paddingVertical: 8,
    borderRadius: 20,
    backgroundColor: '#FFFFFF',
    gap: 4
  },
  tabActive: {
    backgroundColor: '#EFF6FF'
  },
  tabText: {
    fontSize: 13,
    color: '#9CA3AF'
  },
  tabTextActive: {
    color: '#3B82F6',
    fontWeight: '600'
  },
  listContent: {
    paddingVertical: 8,
    flexGrow: 1
  },
  emptyContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    paddingVertical: 60
  },
  emptyTitle: {
    fontSize: 18,
    fontWeight: '600',
    color: '#374151',
    marginTop: 16
  },
  emptyText: {
    fontSize: 14,
    color: '#9CA3AF',
    marginTop: 8
  },
  fab: {
    position: 'absolute',
    right: 20,
    bottom: 30,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: '#3B82F6',
    justifyContent: 'center',
    alignItems: 'center',
    shadowColor: '#3B82F6',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.3,
    shadowRadius: 8,
    elevation: 8
  }
});
```

---

### Day 5：导航与路由 ⭐⭐

**目标**：实现完整的导航系统

**核心代码**：

`MobileForge/src/navigation/types.ts` - 路由类型
```typescript
import { NavigatorScreenParams } from '@react-navigation/native';

export type RootStackParamList = {
  Auth: NavigatorScreenParams<AuthStackParamList>;
  Main: NavigatorScreenParams<MainTabParamList>;
  
  // 认证流程
  Login: undefined;
  Register: undefined;
  
  // 主流程
  TaskList: undefined;
  TaskDetail: { taskId: string };
  CreateTask: undefined;
  EditTask: { taskId: string };
  Profile: undefined;
  
  // 项目
  ProjectList: undefined;
  ProjectDetail: { projectId: string };
};

export type AuthStackParamList = {
  Login: undefined;
  Register: undefined;
};

export type MainTabParamList = {
  Home: undefined;
  Calendar: undefined;
  Statistics: undefined;
  Settings: undefined;
};

declare global {
  namespace ReactNavigation {
    interface RootParamList extends RootStackParamList {}
  }
}
```

`MobileForge/src/navigation/AppNavigator.tsx` - 导航器
```typescript
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { Ionicons } from '@expo/vector-icons';
import { useAuth } from '../hooks/useAuth';

import { LoginScreen } from '../screens/LoginScreen';
import { RegisterScreen } from '../screens/RegisterScreen';
import { TaskListScreen } from '../screens/TaskListScreen';
import { TaskDetailScreen } from '../screens/TaskDetailScreen';
import { CreateTaskScreen } from '../screens/CreateTaskScreen';
import { ProfileScreen } from '../screens/ProfileScreen';
import { CalendarScreen } from '../screens/CalendarScreen';
import { StatisticsScreen } from '../screens/StatisticsScreen';
import { SettingsScreen } from '../screens/SettingsScreen';

import type {
  RootStackParamList,
  AuthStackParamList,
  MainTabParamList
} from './types';

const RootStack = createNativeStackNavigator<RootStackParamList>();
const AuthStack = createNativeStackNavigator<AuthStackParamList>();
const MainTab = createBottomTabNavigator<MainTabParamList>();

// 认证堆栈
function AuthNavigator() {
  return (
    <AuthStack.Navigator screenOptions={{ headerShown: false }}>
      <AuthStack.Screen name="Login" component={LoginScreen} />
      <AuthStack.Screen name="Register" component={RegisterScreen} />
    </AuthStack.Navigator>
  );
}

// 主 Tab 导航
function MainNavigator() {
  return (
    <MainTab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          let iconName: keyof typeof Ionicons.glyphMap;

          switch (route.name) {
            case 'Home':
              iconName = focused ? 'list' : 'list-outline';
              break;
            case 'Calendar':
              iconName = focused ? 'calendar' : 'calendar-outline';
              break;
            case 'Statistics':
              iconName = focused ? 'stats-chart' : 'stats-chart-outline';
              break;
            case 'Settings':
              iconName = focused ? 'settings' : 'settings-outline';
              break;
            default:
              iconName = 'ellipse';
          }

          return <Ionicons name={iconName} size={size} color={color} />;
        },
        tabBarActiveTintColor: '#3B82F6',
        tabBarInactiveTintColor: '#9CA3AF',
        headerShown: false,
        tabBarStyle: {
          paddingBottom: 8,
          paddingTop: 8,
          height: 60
        }
      })}
    >
      <MainTab.Screen
        name="Home"
        component={TaskListScreen}
        options={{ title: '任务' }}
      />
      <MainTab.Screen
        name="Calendar"
        component={CalendarScreen}
        options={{ title: '日历' }}
      />
      <MainTab.Screen
        name="Statistics"
        component={StatisticsScreen}
        options={{ title: '统计' }}
      />
      <MainTab.Screen
        name="Settings"
        component={SettingsScreen}
        options={{ title: '设置' }}
      />
    </MainTab.Navigator>
  );
}

// 根导航
export function AppNavigator() {
  const { isAuthenticated, loading, initialized } = useAuth();

  if (!initialized || loading) {
    // 加载中状态
    return null;
  }

  return (
    <NavigationContainer>
      <RootStack.Navigator screenOptions={{ headerShown: false }}>
        {isAuthenticated ? (
          <>
            <RootStack.Screen name="Main" component={MainNavigator} />
            <RootStack.Screen
              name="TaskDetail"
              component={TaskDetailScreen}
              options={{
                headerShown: true,
                title: '任务详情',
                headerTintColor: '#3B82F6'
              }}
            />
            <RootStack.Screen
              name="CreateTask"
              component={CreateTaskScreen}
              options={{
                headerShown: true,
                title: '新建任务',
                headerTintColor: '#3B82F6',
                presentation: 'modal'
              }}
            />
            <RootStack.Screen
              name="EditTask"
              component={CreateTaskScreen}
              options={{
                headerShown: true,
                title: '编辑任务',
                headerTintColor: '#3B82F6'
              }}
            />
            <RootStack.Screen
              name="Profile"
              component={ProfileScreen}
              options={{
                headerShown: true,
                title: '个人中心',
                headerTintColor: '#3B82F6'
              }}
            />
          </>
        ) : (
          <RootStack.Screen
            name="Auth"
            component={AuthNavigator}
            options={{ headerShown: false }}
          />
        )}
      </RootStack.Navigator>
    </NavigationContainer>
  );
}
```

---

### Day 6：推送通知 ⭐⭐⭐

**目标**：实现 Firebase 推送通知

**核心代码**：

`MobileForge/src/services/notificationService.ts` - 通知服务
```typescript
import * as Notifications from 'expo-notifications';
import * as Permissions from 'expo-notifications';
import { Platform } from 'react-native';
import { getMessaging, getToken, onMessage } from 'firebase/messaging';
import { app } from '../config/firebase';

// 配置通知处理
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true
  })
});

export async function requestNotificationPermissions(): Promise<boolean> {
  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    console.log('通知权限未授予');
    return false;
  }

  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#3B82F6'
    });
  }

  return true;
}

export async function scheduleLocalNotification(
  title: string,
  body: string,
  data?: Record<string, string>,
  trigger?: Notifications.NotificationTriggerInput
): Promise<string | null> {
  try {
    const id = await Notifications.scheduleNotificationAsync({
      content: {
        title,
        body,
        data,
        sound: true
      },
      trigger
    });
    return id;
  } catch (error) {
    console.error('安排通知失败:', error);
    return null;
  }
}

// 任务到期提醒
export async function scheduleTaskReminder(
  taskId: string,
  taskTitle: string,
  dueDate: Date
): Promise<string | null> {
  // 提前 1 小时提醒
  const reminderDate = new Date(dueDate.getTime() - 60 * 60 * 1000);
  
  if (reminderDate <= new Date()) {
    return null;
  }

  return scheduleLocalNotification(
    '任务即将到期',
    `「${taskTitle}」将在 1 小时后到期`,
    { taskId, type: 'task_reminder' },
    {
      date: reminderDate
    }
  );
}

// 取消所有通知
export async function cancelAllNotifications(): Promise<void> {
  await Notifications.cancelAllScheduledNotificationsAsync();
}

// FCM 推送（需要配置 Firebase Cloud Messaging）
export async function registerForPushNotifications(): Promise<string | null> {
  try {
    // 检查权限
    const hasPermission = await requestNotificationPermissions();
    if (!hasPermission) return null;

    // 获取 Expo Push Token
    const tokenData = await Notifications.getExpoPushTokenAsync({
      projectId: 'your-project-id' // 替换为你的项目 ID
    });
    
    console.log('Push Token:', tokenData.data);
    return tokenData.data;
  } catch (error) {
    console.error('获取推送 Token 失败:', error);
    return null;
  }
}

// 监听通知点击
export function setupNotificationListeners(
  onNotificationReceived: (notification: Notifications.Notification) => void,
  onNotificationResponse: (response: Notifications.NotificationResponse) => void
): () => void {
  // 收到通知
  const receivedSubscription = Notifications.addNotificationReceivedListener(
    onNotificationReceived
  );

  // 通知点击
  const responseSubscription = Notifications.addNotificationResponseReceivedListener(
    onNotificationResponse
  );

  // 返回取消订阅函数
  return () => {
    receivedSubscription.remove();
    responseSubscription.remove();
  };
}
```

---

### Day 7：发布与部署 ⭐⭐⭐

**目标**：完成应用发布配置

**EAS 配置**：

`app.json` - Expo 配置
```json
{
  "expo": {
    "name": "MobileForge",
    "slug": "mobileforge",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#3B82F6"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.mobileforge.app",
      "infoPlist": {
        "NSCameraUsageDescription": "用于扫描文档",
        "NSPhotoLibraryUsageDescription": "用于选择头像",
        "UIBackgroundModes": ["remote-notification"]
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#3B82F6"
      },
      "package": "com.mobileforge.app",
      "permissions": [
        "CAMERA",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE",
        "RECEIVE_BOOT_COMPLETED",
        "VIBRATE",
        "INTERNET"
      ]
    },
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#3B82F6"
        }
      ]
    ],
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    }
  }
}
```

---

## 📝 面试要点

### Q1：React Native 与 Flutter 的区别？

**参考答案**：
```
1. 语言：React Native 用 JavaScript/TypeScript，Flutter 用 Dart
2. 渲染：React Native 调用原生组件，Flutter 自绘引擎
3. 性能：Flutter 更稳定一致，React Native 依赖桥接
4. 生态：React Native 社区更大，第三方库更多
5. 学习曲线：React Native 对 Web 开发者更友好
```

### Q2：React Native 的工作原理？

**参考答案**：
```
1. JavaScript 线程执行业务逻辑
2. 通过 Bridge 与原生模块通信
3. 原生线程渲染 UI 组件
4. 异步消息队列协调通信
5. 新架构（Fabric/TurboModules）提升性能
```

### Q3：Firebase 的优缺点？

**参考答案**：
```
优点：
- 快速集成，零服务器架构
- 实时数据库同步
- 免费额度充足
- 完整的认证解决方案

缺点：
- 厂商锁定
- 查询能力有限
- 免费版有限流
- 数据迁移困难
```

### Q4：离线优先如何实现？

**参考答案**：
```
1. 本地缓存：AsyncStorage/Realm
2. 离线检测：NetInfo
3. 乐观更新：先更新 UI，后同步
4. 冲突解决：Last-Write-Wins 或服务端为主
5. 后台同步：WorkManager/BackgroundFetch
```

---

## ✅ 项目清单

```
MobileForge/
├── src/
│   ├── config/
│   │   └── firebase.ts           # Firebase 配置
│   ├── components/
│   │   ├── TaskCard.tsx           # 任务卡片
│   │   ├── PriorityBadge.tsx      # 优先级徽章
│   │   └── EmptyState.tsx         # 空状态
│   ├── hooks/
│   │   └── useAuth.ts             # 认证 Hook
│   ├── navigation/
│   │   ├── types.ts               # 路由类型
│   │   └── AppNavigator.tsx       # 导航器
│   ├── screens/
│   │   ├── LoginScreen.tsx
│   │   ├── RegisterScreen.tsx
│   │   ├── TaskListScreen.tsx
│   │   ├── TaskDetailScreen.tsx
│   │   ├── CreateTaskScreen.tsx
│   │   ├── ProfileScreen.tsx
│   │   ├── CalendarScreen.tsx
│   │   ├── StatisticsScreen.tsx
│   │   └── SettingsScreen.tsx
│   ├── services/
│   │   ├── authService.ts         # 认证服务
│   │   ├── taskService.ts         # 任务服务
│   │   └── notificationService.ts  # 通知服务
│   ├── stores/
│   │   └── taskStore.ts           # 任务状态
│   ├── types/
│   │   └── index.ts
│   └── utils/
│       └── date.ts
├── App.tsx
├── app.json
├── eas.json
└── package.json
```

---

**文档版本**：v1.0  
**更新日期**：2024
