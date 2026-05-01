# 方案二：Next.js + Prisma SaaS 项目管理工具

## 项目介绍

### 项目是什么
这是一个多租户 SaaS 项目管理平台，类似 Asana/Jira，支持团队管理、甘特图、任务看板、时间追踪等功能。

### 解决什么问题
- 团队项目协作效率低
- 展示 SaaS 多租户架构
- 学习复杂业务系统设计

### 核心技术亮点
- **Next.js 14 App Router + RSC**
- **Prisma + PostgreSQL** (更适合复杂关系)
- **多租户架构** (Row-Level Security)
- **实时协作** (Server-Sent Events)
- **RBAC 权限控制**

---

## 完整可运行代码

### 1. Prisma Schema

**prisma/schema.prisma**
```prisma
/**
 * SaaS 项目管理系统数据模型
 * 支持多租户、团队协作、权限管理
 */

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

// ============ 账户与认证 ============

/**
 * 账户（顶级组织）
 * 每个账户是一个独立的企业/团队
 */
model Account {
  id        String   @id @default(cuid())
  name      String                    // 组织名称
  slug      String   @unique         // URL 标识
  plan      Plan     @default(FREE)  // 订阅计划
  logo      String?
  
  // 计费
  stripeCustomerId  String?
  subscriptionId    String?
  subscriptionEnd  DateTime?
  
  // 设置
  settings Json     @default("{}")
  
  // 关联
  members  Member[]
  projects Project[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([slug])
}

/**
 * 订阅计划
 */
enum Plan {
  FREE
  PRO
  ENTERPRISE
}

/**
 * 成员（用户与账户的关系）
 */
model Member {
  id        String   @id @default(cuid())
  accountId String
  account   Account  @relation(fields: [accountId], references: [id], onDelete: Cascade)
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // 角色
  role      Role     @default(MEMBER)
  
  // 设置
  preferences Json   @default("{}")
  
  // 加入时间
  joinedAt DateTime @default(now())
  
  // 唯一约束：每个用户在每个账户只能有一个角色
  @@unique([accountId, userId])
  @@index([userId])
  @@index([accountId])
}

/**
 * 角色枚举
 */
enum Role {
  OWNER      // 所有者
  ADMIN      // 管理员
  MEMBER     // 普通成员
  GUEST      // 访客（只读）
}

/**
 * 用户（全局账户）
 */
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  avatar    String?
  password  String   // 加密存储
  
  // OAuth
  accounts  OAuthAccount[]
  
  // 关联
  memberships Member[]
  
  // 会话
  sessions   Session[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

/**
 * OAuth 关联
 */
model OAuthAccount {
  id           String  @id @default(cuid())
  userId       String
  user         User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  provider     String  // 'google', 'github', 'microsoft'
  providerId   String  // OAuth 提供商的用户 ID
  accessToken  String
  refreshToken String?
  expiresAt    DateTime?
  
  @@unique([provider, providerId])
  @@index([userId])
}

/**
 * 会话
 */
model Session {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  token     String   @unique
  ipAddress String?
  userAgent String?
  
  expiresAt DateTime
  createdAt DateTime @default(now())
  
  @@index([userId])
  @@index([token])
}

// ============ 项目与任务 ============

/**
 * 项目
 */
model Project {
  id        String   @id @default(cuid())
  accountId String
  account   Account  @relation(fields: [accountId], references: [id], onDelete: Cascade)
  
  name      String
  key       String   // 项目标识，如 "PROJ"，任务会以 PROJ-1 形式编号
  slug      String   // URL 友好标识
  
  // 描述
  description String?
  
  // 图标和颜色
  icon      String   @default("📁")
  color     String   @default("#6366f1")
  
  // 可见性
  isPublic  Boolean  @default(false)
  
  // 设置
  settings  Json     @default("{}")
  
  // 关联
  members   ProjectMember[]
  tasks     Task[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@unique([accountId, slug])
  @@unique([accountId, key])
  @@index([accountId])
}

/**
 * 项目成员
 */
model ProjectMember {
  id        String  @id @default(cuid())
  projectId String
  project   Project @relation(fields: [projectId], references: [id], onDelete: Cascade)
  memberId  String
  member    Member  @relation(fields: [memberId], references: [id], onDelete: Cascade)
  
  role      ProjectRole @default(MEMBER)
  
  // 通知设置
  notifications Json   @default("{}")
  
  createdAt DateTime @default(now())
  
  @@unique([projectId, memberId])
  @@index([memberId])
}

/**
 * 项目角色
 */
enum ProjectRole {
  ADMIN     // 项目管理员
  EDITOR    // 可编辑
  MEMBER    // 成员
  VIEWER    // 只读
  COMMENTER // 仅评论
}

/**
 * 任务
 */
model Task {
  id        String   @id @default(cuid())
  projectId String
  project   Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  
  // 编号
  taskNumber Int     // 任务编号，如 1
  
  // 基本信息
  title     String
  description String?
  status    TaskStatus @default(BACKLOG)
  priority  Priority  @default(MEDIUM)
  
  // 分类
  type      TaskType  @default(TASK)
  
  // 负责人
  assigneeId String?
  assignee   Member?  @relation(fields: [assigneeId], references: [id])
  
  // 父任务（子任务支持）
  parentId  String?
  parent    Task?    @relation("Subtasks", fields: [parentId], references: [id])
  subtasks  Task[]   @relation("Subtasks")
  
  // 标签
  labels    Label[]
  
  // 里程碑
  milestoneId String?
  milestone   Milestone? @relation(fields: [milestoneId], references: [id])
  
  // 日期
  startDate DateTime?
  dueDate   DateTime?
  
  // 估算
  estimate  Int?     // 预估小时数
  timeSpent Int?     @default(0) // 已用小时数
  
  // 排序
  order     Int      @default(0)
  
  // 归档
  archived  Boolean  @default(false)
  
  // 关联
  comments  Comment[]
  attachments Attachment[]
  activities Activity[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([projectId])
  @@index([assigneeId])
  @@index([status])
  @@index([priority])
  @@index([milestoneId])
}

enum TaskStatus {
  BACKLOG      // 待办
  TODO         // 计划中
  IN_PROGRESS  // 进行中
  IN_REVIEW    // 审核中
  DONE         // 完成
  CANCELLED    // 已取消
}

enum Priority {
  LOWEST
  LOW
  MEDIUM
  HIGH
  HIGHEST
}

enum TaskType {
  TASK       // 任务
  BUG        // Bug
  STORY      // 故事
  EPIC       // 史诗
}

/**
 * 标签
 */
model Label {
  id        String  @id @default(cuid())
  projectId String
  name      String
  color     String
  
  tasks     Task[]
  
  @@unique([projectId, name])
  @@index([projectId])
}

/**
 * 里程碑
 */
model Milestone {
  id        String   @id @default(cuid())
  projectId String
  project   Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  
  title     String
  description String?
  dueDate   DateTime?
  
  tasks     Task[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([projectId])
}

// ============ 协作功能 ============

/**
 * 评论
 */
model Comment {
  id        String   @id @default(cuid())
  taskId    String
  task      Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  
  content   String
  authorId  String
  author    Member   @relation(fields: [authorId], references: [id])
  
  // Markdown 内容
  contentHtml String?
  
  // 编辑
  editedAt DateTime?
  
  // 回复
  parentId  String?
  parent    Comment?  @relation("Replies", fields: [parentId], references: [id])
  replies   Comment[] @relation("Replies")
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([taskId])
  @@index([authorId])
}

/**
 * 附件
 */
model Attachment {
  id        String   @id @default(cuid())
  taskId    String
  task      Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  
  name      String
  url       String
  mimeType  String?
  size      Int      // bytes
  
  uploadedBy String
  uploadedAt DateTime @default(now())
  
  @@index([taskId])
}

/**
 * 活动日志
 */
model Activity {
  id        String   @id @default(cuid())
  taskId    String
  task      Task     @relation(fields: [taskId], references: [id], onDelete: Cascade)
  
  type      String   // 'created', 'updated', 'commented', 'assigned', etc.
  actorId   String
  actor     Member   @relation(fields: [actorId], references: [id])
  
  // 变更详情
  changes   Json?    // { field: [oldValue, newValue] }
  metadata  Json?    // 额外信息
  
  createdAt DateTime @default(now())
  
  @@index([taskId])
  @@index([actorId])
}
```

### 2. 多租户中间件

**src/lib/tenant.ts**
```typescript
/**
 * 多租户支持
 * 
 * 实现 Row-Level Security (行级安全)
 * 确保每个用户只能访问自己账户的数据
 */

import { prisma } from './prisma';
import { getServerSession } from 'next-auth';
import { authOptions } from './auth-options';

/**
 * 获取当前请求的租户上下文
 * 从 session 和请求头获取账户信息
 */
export async function getTenantContext(request?: Request) {
  // 1. 获取用户 session
  const session = await getServerSession(authOptions);
  if (!session?.user?.id) {
    return null;
  }
  
  // 2. 从请求头获取账户 ID（如果是子域名或路径）
  const accountSlug = request?.headers.get('x-account-slug') || 
                      extractSubdomain(request?.url);
  
  // 3. 获取用户在当前账户的成员信息
  const member = await prisma.member.findFirst({
    where: {
      userId: session.user.id,
      account: accountSlug ? {
        slug: accountSlug,
      } : undefined,
    },
    include: {
      account: true,
      user: {
        select: {
          id: true,
          email: true,
          name: true,
          image: true,
        },
      },
    },
  });
  
  if (!member) {
    return null;
  }
  
  return {
    user: member.user,
    account: member.account,
    member,
    role: member.role,
  };
}

/**
 * 提取子域名
 */
function extractSubdomain(url?: string): string | null {
  if (!url) return null;
  
  try {
    const hostname = new URL(url).hostname;
    const parts = hostname.split('.');
    
    // 如果有子域名，返回第一部分
    // 例如: acme.projectapp.com -> acme
    if (parts.length > 2) {
      return parts[0];
    }
  } catch {
    // URL 解析失败
  }
  
  return null;
}

/**
 * 权限检查函数
 */
export async function requireAccountAccess() {
  const context = await getTenantContext();
  
  if (!context) {
    throw new Error('需要登录并选择账户');
  }
  
  return context;
}

/**
 * 检查是否有账户管理员权限
 */
export async function requireAdminAccess() {
  const context = await requireAccountAccess();
  
  if (!['OWNER', 'ADMIN'].includes(context.role)) {
    throw new Error('需要管理员权限');
  }
  
  return context;
}

/**
 * 创建带租户过滤的 Prisma 查询
 * 自动添加账户 ID 过滤条件
 */
export function createTenantQuery<T>(
  model: keyof typeof prisma,
  where: any = {}
) {
  return async (accountId: string) => {
    const method = prisma[model] as any;
    
    return method.findMany({
      where: {
        ...where,
        // 自动添加账户过滤
        ...(await getTenantModelFilter(model, accountId)),
      },
    });
  };
}

/**
 * 根据模型获取租户过滤条件
 */
function getTenantModelFilter(model: string, accountId: string): any {
  const modelFilters: Record<string, any> = {
    Project: { accountId },
    ProjectMember: { project: { accountId } },
    Task: { project: { accountId } },
    Comment: { task: { project: { accountId } } },
    // ... 其他模型
  };
  
  return modelFilters[model] || {};
}

/**
 * 项目成员权限检查
 */
export async function checkProjectAccess(
  projectId: string,
  userId: string,
  requiredRole: ProjectRole[] = []
): Promise<boolean> {
  const member = await prisma.projectMember.findFirst({
    where: {
      projectId,
      member: {
        userId,
      },
    },
  });
  
  if (!member) return false;
  
  // 如果没有指定必需角色，只要能访问项目就行
  if (requiredRole.length === 0) return true;
  
  // 检查是否有足够的权限
  return requiredRole.includes(member.role);
}
```

### 3. 项目 CRUD 操作

**src/actions/project.ts**
```typescript
/**
 * 项目 Server Actions
 */

'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { requireAccountAccess, requireAdminAccess } from '@/lib/tenant';
import { generateSlug } from '@/lib/utils';
import { ProjectRole } from '@prisma/client';

// ============ 验证 Schema ============

const createProjectSchema = z.object({
  name: z.string().min(1, '项目名称不能为空').max(100),
  key: z.string()
    .min(2, '项目标识至少2个字符')
    .max(10, '项目标识最多10个字符')
    .regex(/^[A-Z]+$/, '项目标识只能使用大写字母'),
  description: z.string().max(1000).optional(),
  icon: z.string().max(10).default('📁'),
  color: z.string().regex(/^#[0-9A-Fa-f]{6}$/).default('#6366f1'),
  isPublic: z.boolean().default(false),
});

const updateProjectSchema = createProjectSchema.partial().extend({
  id: z.string().min(1),
});

// ============ 项目操作 ============

/**
 * 获取用户可访问的项目列表
 */
export async function getProjects() {
  const context = await requireAccountAccess();
  
  const projects = await prisma.project.findMany({
    where: {
      accountId: context.account.id,
      OR: [
        { isPublic: true },  // 公开项目
        { members: { some: { memberId: context.member.id } } },  // 或是成员
      ],
    },
    include: {
      members: {
        include: {
          member: {
            include: {
              user: {
                select: { id: true, name: true, avatar: true },
              },
            },
          },
        },
      },
      _count: {
        select: { tasks: true },
      },
    },
    orderBy: { createdAt: 'desc' },
  });
  
  return projects;
}

/**
 * 获取单个项目详情
 */
export async function getProject(idOrSlug: string) {
  const context = await requireAccountAccess();
  
  const project = await prisma.project.findFirst({
    where: {
      OR: [
        { id: idOrSlug },
        { slug: idOrSlug },
      ],
      accountId: context.account.id,
      OR: [
        { isPublic: true },
        { members: { some: { memberId: context.member.id } } },
      ],
    },
    include: {
      account: true,
      members: {
        include: {
          member: {
            include: {
              user: {
                select: { id: true, name: true, email: true, avatar: true },
              },
            },
          },
        },
      },
      labels: true,
      milestones: {
        orderBy: { dueDate: 'asc' },
      },
    },
  });
  
  return project;
}

/**
 * 创建项目
 */
export async function createProject(data: z.infer<typeof createProjectSchema>) {
  try {
    const context = await requireAccountAccess();
    
    // 验证
    const validation = createProjectSchema.safeParse(data);
    if (!validation.success) {
      return { error: validation.error.errors[0].message };
    }
    
    // 检查权限
    if (!['OWNER', 'ADMIN'].includes(context.role)) {
      return { error: '需要管理员权限创建项目' };
    }
    
    // 生成唯一 slug
    let slug = generateSlug(data.name);
    while (await prisma.project.findUnique({
      where: { accountId_slug: { accountId: context.account.id, slug } },
    })) {
      slug = `${slug}-${Date.now()}`;
    }
    
    // 创建项目
    const project = await prisma.project.create({
      data: {
        name: data.name,
        slug,
        key: data.key.toUpperCase(),
        description: data.description,
        icon: data.icon,
        color: data.color,
        isPublic: data.isPublic,
        accountId: context.account.id,
      },
    });
    
    // 添加创建者为项目管理员
    await prisma.projectMember.create({
      data: {
        projectId: project.id,
        memberId: context.member.id,
        role: ProjectRole.ADMIN,
      },
    });
    
    revalidatePath('/dashboard');
    revalidatePath(`/dashboard/projects/${project.slug}`);
    
    return { success: true, project };
    
  } catch (error) {
    console.error('创建项目失败:', error);
    return { error: '创建失败，请重试' };
  }
}

/**
 * 更新项目
 */
export async function updateProject(data: z.infer<typeof updateProjectSchema>) {
  try {
    const context = await requireAccountAccess();
    
    // 验证
    const validation = updateProjectSchema.safeParse(data);
    if (!validation.success) {
      return { error: validation.error.errors[0].message };
    }
    
    // 检查项目和权限
    const project = await prisma.project.findFirst({
      where: {
        id: data.id,
        accountId: context.account.id,
        members: {
          some: {
            memberId: context.member.id,
            role: { in: ['ADMIN' as ProjectRole, 'MEMBER' as ProjectRole] },
          },
        },
      },
    });
    
    if (!project) {
      return { error: '项目不存在或无权限' };
    }
    
    // 更新
    const updated = await prisma.project.update({
      where: { id: data.id },
      data: {
        name: data.name,
        description: data.description,
        icon: data.icon,
        color: data.color,
        isPublic: data.isPublic,
      },
    });
    
    revalidatePath(`/dashboard/projects/${project.slug}`);
    
    return { success: true, project: updated };
    
  } catch (error) {
    console.error('更新项目失败:', error);
    return { error: '更新失败，请重试' };
  }
}

/**
 * 删除项目
 */
export async function deleteProject(projectId: string) {
  try {
    const context = await requireAdminAccess();
    
    const project = await prisma.project.findFirst({
      where: {
        id: projectId,
        accountId: context.account.id,
      },
    });
    
    if (!project) {
      return { error: '项目不存在' };
    }
    
    await prisma.project.delete({
      where: { id: projectId },
    });
    
    revalidatePath('/dashboard');
    
    return { success: true };
    
  } catch (error) {
    console.error('删除项目失败:', error);
    return { error: '删除失败，请重试' };
  }
}

// ============ 成员管理 ============

/**
 * 添加项目成员
 */
export async function addProjectMember(
  projectId: string,
  memberId: string,
  role: ProjectRole = ProjectRole.MEMBER
) {
  try {
    const context = await requireAccountAccess();
    
    // 检查权限
    const existingMember = await prisma.projectMember.findFirst({
      where: {
        projectId,
        memberId: context.member.id,
        role: { in: ['ADMIN' as ProjectRole] },
      },
    });
    
    if (!existingMember) {
      return { error: '需要管理员权限' };
    }
    
    // 检查成员是否属于同一账户
    const targetMember = await prisma.member.findFirst({
      where: {
        id: memberId,
        accountId: context.account.id,
      },
    });
    
    if (!targetMember) {
      return { error: '成员不存在于当前组织' };
    }
    
    // 添加或更新成员
    await prisma.projectMember.upsert({
      where: {
        projectId_memberId: { projectId, memberId },
      },
      update: { role },
      create: { projectId, memberId, role },
    });
    
    revalidatePath(`/dashboard/projects/${projectId}/settings/members`);
    
    return { success: true };
    
  } catch (error) {
    console.error('添加成员失败:', error);
    return { error: '添加失败' };
  }
}
```

### 4. 任务操作

**src/actions/task.ts**
```typescript
/**
 * 任务 Server Actions
 */

'use server';

import { revalidatePath } from 'next/cache';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { requireAccountAccess } from '@/lib/tenant';
import { checkProjectAccess } from '@/lib/tenant';
import { Priority, TaskStatus, TaskType, ProjectRole } from '@prisma/client';

// ============ 验证 Schema ============

const createTaskSchema = z.object({
  projectId: z.string().min(1),
  title: z.string().min(1, '标题不能为空').max(500),
  description: z.string().max(10000).optional(),
  status: z.nativeEnum(TaskStatus).default(TaskStatus.BACKLOG),
  priority: z.nativeEnum(Priority).default(Priority.MEDIUM),
  type: z.nativeEnum(TaskType).default(TaskType.TASK),
  assigneeId: z.string().optional(),
  milestoneId: z.string().optional(),
  startDate: z.string().datetime().optional(),
  dueDate: z.string().datetime().optional(),
  estimate: z.number().int().positive().optional(),
});

const updateTaskSchema = createTaskSchema.partial().extend({
  id: z.string().min(1),
});

const moveTaskSchema = z.object({
  id: z.string().min(1),
  status: z.nativeEnum(TaskStatus).optional(),
  order: z.number().int().min(0).optional(),
  assigneeId: z.string().nullable().optional(),
});

// ============ 任务操作 ============

/**
 * 获取项目任务列表
 */
export async function getProjectTasks(projectId: string) {
  const context = await requireAccountAccess();
  
  // 验证访问权限
  const hasAccess = await checkProjectAccess(projectId, context.user.id);
  if (!hasAccess) {
    return [];
  }
  
  const tasks = await prisma.task.findMany({
    where: {
      projectId,
      archived: false,
      parentId: null,  // 只获取顶级任务
    },
    include: {
      assignee: {
        include: {
          user: {
            select: { id: true, name: true, avatar: true },
          },
        },
      },
      labels: true,
      milestone: {
        select: { id: true, title: true, dueDate: true },
      },
      subtasks: {
        select: { id: true },
      },
      _count: {
        select: { comments: true, attachments: true },
      },
    },
    orderBy: [
      { status: 'asc' },
      { order: 'asc' },
    ],
  });
  
  return tasks;
}

/**
 * 获取单个任务详情
 */
export async function getTask(taskId: string) {
  const context = await requireAccountAccess();
  
  const task = await prisma.task.findUnique({
    where: { id: taskId },
    include: {
      project: true,
      assignee: {
        include: {
          user: {
            select: { id: true, name: true, email: true, avatar: true },
          },
        },
      },
      labels: true,
      milestone: true,
      parent: {
        select: { id: true, title: true, taskNumber: true },
      },
      subtasks: {
        include: {
          assignee: {
            include: {
              user: {
                select: { id: true, name: true, avatar: true },
              },
            },
          },
        },
        orderBy: { order: 'asc' },
      },
      comments: {
        include: {
          author: {
            include: {
              user: {
                select: { id: true, name: true, avatar: true },
              },
            },
          },
          replies: {
            include: {
              author: {
                include: {
                  user: {
                    select: { id: true, name: true, avatar: true },
                  },
                },
              },
            },
          },
        },
        where: { parentId: null },  // 只获取顶级评论
        orderBy: { createdAt: 'desc' },
      },
      attachments: true,
      activities: {
        include: {
          actor: {
            include: {
              user: {
                select: { id: true, name: true, avatar: true },
              },
            },
          },
        },
        orderBy: { createdAt: 'desc' },
        take: 20,
      },
    },
  });
  
  if (!task) {
    return null;
  }
  
  // 验证访问权限
  const hasAccess = await checkProjectAccess(task.projectId, context.user.id);
  if (!hasAccess) {
    return null;
  }
  
  return task;
}

/**
 * 创建任务
 */
export async function createTask(data: z.infer<typeof createTaskSchema>) {
  try {
    const context = await requireAccountAccess();
    
    // 验证
    const validation = createTaskSchema.safeParse(data);
    if (!validation.success) {
      return { error: validation.error.errors[0].message };
    }
    
    // 检查项目访问权限
    const hasAccess = await checkProjectAccess(data.projectId, context.user.id, [
      ProjectRole.ADMIN,
      ProjectRole.EDITOR,
      ProjectRole.MEMBER,
    ]);
    if (!hasAccess) {
      return { error: '无权限在此项目中创建任务' };
    }
    
    // 获取下一个任务编号
    const lastTask = await prisma.task.findFirst({
      where: { projectId: data.projectId },
      orderBy: { taskNumber: 'desc' },
    });
    const taskNumber = (lastTask?.taskNumber || 0) + 1;
    
    // 获取当前状态最后任务的 order
    const lastInStatus = await prisma.task.findFirst({
      where: {
        projectId: data.projectId,
        status: data.status,
        parentId: null,
      },
      orderBy: { order: 'desc' },
    });
    const order = (lastInStatus?.order || 0) + 1;
    
    // 创建任务
    const task = await prisma.task.create({
      data: {
        projectId: data.projectId,
        taskNumber,
        title: data.title,
        description: data.description,
        status: data.status,
        priority: data.priority,
        type: data.type,
        assigneeId: data.assigneeId,
        milestoneId: data.milestoneId,
        startDate: data.startDate ? new Date(data.startDate) : undefined,
        dueDate: data.dueDate ? new Date(data.dueDate) : undefined,
        estimate: data.estimate,
        order,
      },
      include: {
        assignee: {
          include: {
            user: {
              select: { id: true, name: true, avatar: true },
            },
          },
        },
        labels: true,
        milestone: true,
      },
    });
    
    // 记录活动
    await prisma.activity.create({
      data: {
        taskId: task.id,
        actorId: context.member.id,
        type: 'created',
        metadata: { title: task.title },
      },
    });
    
    revalidatePath(`/dashboard/projects/${task.projectId}`);
    
    return { success: true, task };
    
  } catch (error) {
    console.error('创建任务失败:', error);
    return { error: '创建失败，请重试' };
  }
}

/**
 * 更新任务
 */
export async function updateTask(data: z.infer<typeof updateTaskSchema>) {
  try {
    const context = await requireAccountAccess();
    
    // 获取任务
    const task = await prisma.task.findUnique({
      where: { id: data.id },
      include: { project: true },
    });
    
    if (!task) {
      return { error: '任务不存在' };
    }
    
    // 检查权限
    const hasAccess = await checkProjectAccess(task.projectId, context.user.id, [
      ProjectRole.ADMIN,
      ProjectRole.EDITOR,
      ProjectRole.MEMBER,
    ]);
    if (!hasAccess) {
      return { error: '无权限修改此任务' };
    }
    
    // 记录变更
    const changes: any = {};
    const fields = ['title', 'description', 'status', 'priority', 'assigneeId', 'dueDate'] as const;
    
    fields.forEach((field) => {
      if (data[field] !== undefined && data[field] !== task[field]) {
        changes[field] = [task[field], data[field]];
      }
    });
    
    // 更新任务
    const updated = await prisma.task.update({
      where: { id: data.id },
      data: {
        title: data.title,
        description: data.description,
        status: data.status,
        priority: data.priority,
        type: data.type,
        assigneeId: data.assigneeId,
        milestoneId: data.milestoneId,
        startDate: data.startDate ? new Date(data.startDate) : undefined,
        dueDate: data.dueDate ? new Date(data.dueDate) : undefined,
        estimate: data.estimate,
      },
      include: {
        assignee: {
          include: {
            user: {
              select: { id: true, name: true, avatar: true },
            },
          },
        },
        labels: true,
        milestone: true,
      },
    });
    
    // 记录活动
    if (Object.keys(changes).length > 0) {
      await prisma.activity.create({
        data: {
          taskId: task.id,
          actorId: context.member.id,
          type: 'updated',
          changes,
        },
      });
    }
    
    revalidatePath(`/dashboard/projects/${task.projectId}/tasks/${task.id}`);
    
    return { success: true, task: updated };
    
  } catch (error) {
    console.error('更新任务失败:', error);
    return { error: '更新失败，请重试' };
  }
}

/**
 * 移动任务（拖拽、状态变更）
 */
export async function moveTask(data: z.infer<typeof moveTaskSchema>) {
  try {
    const context = await requireAccountAccess();
    
    const task = await prisma.task.findUnique({
      where: { id: data.id },
      include: { project: true },
    });
    
    if (!task) {
      return { error: '任务不存在' };
    }
    
    // 权限检查
    const hasAccess = await checkProjectAccess(task.projectId, context.user.id, [
      ProjectRole.ADMIN,
      ProjectRole.EDITOR,
      ProjectRole.MEMBER,
    ]);
    if (!hasAccess) {
      return { error: '无权限移动此任务' };
    }
    
    // 如果状态变更，需要重新排序
    if (data.status && data.status !== task.status) {
      // 将该状态的其他任务往后移
      await prisma.task.updateMany({
        where: {
          projectId: task.projectId,
          status: data.status,
          order: { gte: data.order || 0 },
          parentId: null,
        },
        data: {
          order: { increment: 1 },
        },
      });
      
      // 将原状态的任务往前移
      await prisma.task.updateMany({
        where: {
          projectId: task.projectId,
          status: task.status,
          order: { gt: task.order },
          parentId: null,
        },
        data: {
          order: { decrement: 1 },
        },
      });
    }
    
    // 更新任务
    const updated = await prisma.task.update({
      where: { id: data.id },
      data: {
        status: data.status,
        order: data.order,
        assigneeId: data.assigneeId,
      },
    });
    
    // 记录活动
    await prisma.activity.create({
      data: {
        taskId: task.id,
        actorId: context.member.id,
        type: 'moved',
        metadata: {
          from: { status: task.status },
          to: { status: data.status },
        },
      },
    });
    
    revalidatePath(`/dashboard/projects/${task.projectId}`);
    
    return { success: true };
    
  } catch (error) {
    console.error('移动任务失败:', error);
    return { error: '移动失败，请重试' };
  }
}

/**
 * 删除任务
 */
export async function deleteTask(taskId: string) {
  try {
    const context = await requireAccountAccess();
    
    const task = await prisma.task.findUnique({
      where: { id: taskId },
    });
    
    if (!task) {
      return { error: '任务不存在' };
    }
    
    // 权限检查
    const hasAccess = await checkProjectAccess(task.projectId, context.user.id, [
      ProjectRole.ADMIN,
    ]);
    if (!hasAccess) {
      return { error: '无权限删除此任务' };
    }
    
    await prisma.task.delete({
      where: { id: taskId },
    });
    
    revalidatePath(`/dashboard/projects/${task.projectId}`);
    
    return { success: true };
    
  } catch (error) {
    console.error('删除任务失败:', error);
    return { error: '删除失败，请重试' };
  }
}

/**
 * 添加评论
 */
export async function addComment(taskId: string, content: string, parentId?: string) {
  try {
    const context = await requireAccountAccess();
    
    const task = await prisma.task.findUnique({
      where: { id: taskId },
    });
    
    if (!task) {
      return { error: '任务不存在' };
    }
    
    // 权限检查
    const hasAccess = await checkProjectAccess(task.projectId, context.user.id);
    if (!hasAccess) {
      return { error: '无权限评论此任务' };
    }
    
    const comment = await prisma.comment.create({
      data: {
        taskId,
        content,
        authorId: context.member.id,
        parentId,
      },
      include: {
        author: {
          include: {
            user: {
              select: { id: true, name: true, avatar: true },
            },
          },
        },
      },
    });
    
    // 记录活动
    await prisma.activity.create({
      data: {
        taskId,
        actorId: context.member.id,
        type: 'commented',
        metadata: { commentId: comment.id },
      },
    });
    
    return { success: true, comment };
    
  } catch (error) {
    console.error('添加评论失败:', error);
    return { error: '评论失败' };
  }
}
```

---

## 学习计划（8天）

### Day 1-2: 项目架构与数据库设计
- 多租户架构设计
- Prisma Schema 设计
- 认证与授权

### Day 3-4: 核心 CRUD 实现
- 项目管理功能
- 任务管理功能
- Server Actions

### Day 5-6: 权限与协作
- RBAC 权限系统
- 评论功能
- 活动日志

### Day 7-8: 前端与优化
- 看板视图
- 甘特图
- 性能优化

---

## 面试要点

### 问题 1: 多租户架构有哪些实现方式？

**参考答案**:
```
1. 独立数据库
   - 每个租户独立数据库
   - 隔离性最好，成本高
   
2. 共享数据库，独立 Schema
   - 使用 PostgreSQL Schema 隔离
   - 较好的隔离性
   
3. 共享数据库，共享 Schema + 行级安全
   - 所有租户共享表
   - 使用 tenant_id 字段区分
   - 通过 RLS 或中间件过滤
   - 成本最低，推荐大多数场景

Row-Level Security 实现：
- 中间件自动注入 tenant_id
- Prisma 扩展自动添加过滤
- 数据库层 RLS 策略
```

---

### 问题 2: 如何实现细粒度的权限控制？

**参考答案**:
```typescript
// RBAC (Role-Based Access Control)
enum Role {
  OWNER = 10,    // 权重越高权限越大
  ADMIN = 8,
  MANAGER = 5,
  MEMBER = 1,
  GUEST = 0,
}

// 权限检查
function canAccess(userRole: Role, requiredRole: Role): boolean {
  return userRole >= requiredRole;
}

// 在 Server Action 中检查
export async function deleteProject(id: string) {
  const { role } = await getUserContext();
  if (!canAccess(role, Role.ADMIN)) {
    throw new Error('需要管理员权限');
  }
  // ...
}
```

---

### 问题 3: Prisma 关联查询的性能优化？

**参考答案**:
```
1. 选择性字段
   - 只 select 需要的字段
   - 不要 select *

2. 分页
   - 使用 take/skip
   - 游标分页更高效

3. include vs join
   - include 用于一对多
   - select + join 用于多对一

4. $transaction
   - 批量操作使用事务
   - 减少数据库往返

5. lean()
   - 跳过类型转换，更快
```

---

## 项目产出

- ✅ 多租户 SaaS 架构
- ✅ Prisma 高级用法
- ✅ RBAC 权限系统
- ✅ 复杂业务逻辑
