# NeonStack Node 全栈完整项目指南

> 基于 Node.js + Express + Vue + MongoDB 的社交笔记应用

---

## 📋 项目介绍

### 项目概述
NeonStack 是一个社交化的笔记管理平台，支持Markdown编辑、笔记分享、标签分类等功能。这个项目采用经典的 Node.js + Express 技术栈，是学习全栈开发的绝佳入门项目。

- **前后分离**：Vue 3 前端 + Express API
- **数据库灵活**：MongoDB 无schema设计，适合快速迭代
- **实时功能**：支持笔记协作预览
- **面试友好**：涵盖Node.js全栈核心知识点

### 核心技术亮点
```
后端：Node.js + Express + Mongoose + MongoDB
前端：Vue 3 + Composition API + Pinia + Element Plus
通信：RESTful API + JWT认证
特色：Markdown渲染、标签系统、笔记分享
```

---

## 🏗️ 技术架构图

### 系统架构

```
┌─────────────────────────────────────────────────────────────────┐
│                         客户端层                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐             │
│  │   Vue 3     │  │   移动端    │  │    浏览器   │             │
│  │   SPA       │  │   H5        │  │   Postman   │             │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘             │
└─────────┼────────────────┼────────────────┼─────────────────────┘
          │                │                │
          └────────────────┼────────────────┘
                           │ HTTP/REST
┌──────────────────────────┼────────────────────────────────────┐
│                     Express.js Server                           │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                      Express Middleware                      ││
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    ││
│  │  │  CORS    │  │  Helmet   │  │ Morgan   │  │ BodyParser│    ││
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘    ││
│  │  ┌──────────────────────────────────────────────────────┐   ││
│  │  │                    Routes                             │   ││
│  │  │  /api/auth   /api/notes   /api/users   /api/tags      │   ││
│  │  └──────────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────────┘│
└──────────────────────────┼─────────────────────────────────────┘
                           │
          ┌────────────────┴────────────────┐
          │                                 │
┌─────────┴───────┐               ┌─────────┴───────┐
│    MongoDB       │               │   存储层        │
│    (主数据库)     │               │                 │
│                  │               │  - 笔记内容      │
│  Collections:    │               │  - 用户头像      │
│  - users         │               │  - 附件文件      │
│  - notes         │               │                 │
│  - tags           │               │  (本地/OSS)     │
│  - comments       │               │                 │
└──────────────────┘               └─────────────────┘
```

### 技术栈层级

```
┌────────────────────────────────────────────────┐
│                 视图层 (View Layer)              │
│         Vue 3 + Element Plus + Markdown        │
├────────────────────────────────────────────────┤
│                 状态管理 (State)                │
│              Pinia Store + LocalStorage        │
├────────────────────────────────────────────────┤
│                 API 层 (API Layer)              │
│              Axios + Interceptors              │
├────────────────────────────────────────────────┤
│                 Express.js                      │
│     Router + Middleware + Controller           │
├────────────────────────────────────────────────┤
│                 数据访问层                       │
│            Mongoose ODM + MongoDB              │
├────────────────────────────────────────────────┤
│                 数据库层                         │
│              MongoDB Atlas / 本地                │
└────────────────────────────────────────────────┘
```

---

## 📅 学习计划（7天）

### Day 1：项目初始化与环境搭建 ⭐

**目标**：搭建 Node.js 后端和 Vue 3 前端项目

**学习内容**：
- Node.js 项目初始化
- Express 框架核心概念
- Vue 3 + Vite 项目创建
- npm/yarn/pnpm 包管理

**产出**：完整的前后端项目骨架

**执行命令**：

```bash
# 1. 创建项目目录
mkdir neonstack && cd neonstack

# 2. 创建后端项目
mkdir backend && cd backend
npm init -y

# 3. 安装后端依赖
npm install express mongoose bcryptjs jsonwebtoken cors helmet morgan 
npm install dotenv express-validator
npm install -D nodemon

# 4. 创建前端项目（返回上级目录）
cd ..
npm create vite@latest frontend -- --template vue-ts
cd frontend
npm install

# 5. 安装前端依赖
npm install vue-router@4 pinia axios element-plus @element-plus/icons-vue
npm install markdown-it marked highlight.js
npm install -D @types/node sass

# 6. 后端项目结构
cd ../backend
mkdir -p src/{config,controllers,middleware,models,routes,utils}
touch src/index.js src/config/database.js
mkdir src/{users,notes,tags,auth}
```

---

### Day 2：MongoDB 数据模型设计 ⭐⭐

**目标**：设计并实现 MongoDB 数据模型

**学习内容**：
- MongoDB 核心概念（Collection、Document）
- Mongoose ODM 的使用
- 数据验证和中间件
- 索引设计

**产出**：完整的 Mongoose 模型

**核心代码**：

`backend/src/config/database.js` - 数据库配置
```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    const conn = await mongoose.connect(process.env.MONGODB_URI, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    
    console.log(`MongoDB Connected: ${conn.connection.host}`);
    
    // 监听连接事件
    mongoose.connection.on('error', (err) => {
      console.error('MongoDB connection error:', err);
    });
    
    mongoose.connection.on('disconnected', () => {
      console.warn('MongoDB disconnected');
    });
    
  } catch (error) {
    console.error('MongoDB connection failed:', error.message);
    process.exit(1);
  }
};

module.exports = connectDB;
```

`backend/src/models/User.js` - 用户模型
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

const UserSchema = new mongoose.Schema({
  username: {
    type: String,
    required: [true, '用户名不能为空'],
    unique: true,
    trim: true,
    minlength: [3, '用户名至少3个字符'],
    maxlength: [20, '用户名最多20个字符'],
    match: [/^[a-zA-Z0-9_]+$/, '用户名只能包含字母、数字和下划线']
  },
  email: {
    type: String,
    required: [true, '邮箱不能为空'],
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, '请输入有效的邮箱']
  },
  password: {
    type: String,
    required: [true, '密码不能为空'],
    minlength: [6, '密码至少6个字符'],
    select: false  // 查询时默认不返回
  },
  avatar: {
    type: String,
    default: null
  },
  bio: {
    type: String,
    maxlength: [200, '简介最多200个字符'],
    default: ''
  },
  // 社交功能
  followers: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  following: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  // 笔记统计
  notesCount: {
    type: Number,
    default: 0
  },
  likesReceived: {
    type: Number,
    default: 0
  }
}, {
  timestamps: true
});

// 索引
UserSchema.index({ email: 1 });
UserSchema.index({ username: 1 });
UserSchema.index({ createdAt: -1 });

// 密码加密中间件
UserSchema.pre('save', async function(next) {
  // 如果密码未修改，跳过
  if (!this.isModified('password')) {
    return next();
  }
  
  try {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
    next();
  } catch (error) {
    next(error);
  }
});

// 验证密码方法
UserSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

// 生成 JWT Token
UserSchema.methods.generateToken = function() {
  return jwt.sign(
    { id: this._id, username: this.username },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRE || '30d' }
  );
};

// 转为 JSON 时移除敏感字段
UserSchema.methods.toJSON = function() {
  const user = this.toObject();
  delete user.password;
  return user;
};

module.exports = mongoose.model('User', UserSchema);
```

`backend/src/models/Note.js` - 笔记模型
```javascript
const mongoose = require('mongoose');

const NoteSchema = new mongoose.Schema({
  title: {
    type: String,
    required: [true, '标题不能为空'],
    trim: true,
    maxlength: [200, '标题最多200个字符']
  },
  content: {
    type: String,
    default: ''
  },
  // Markdown 渲染后的 HTML
  contentHtml: {
    type: String,
    default: ''
  },
  // 作者
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  // 标签
  tags: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Tag'
  }],
  // 状态
  isPublic: {
    type: Boolean,
    default: true
  },
  isDeleted: {
    type: Boolean,
    default: false
  },
  // 统计数据
  likes: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  likesCount: {
    type: Number,
    default: 0
  },
  commentsCount: {
    type: Number,
    default: 0
  },
  views: {
    type: Number,
    default: 0
  },
  // 置顶和收藏
  isPinned: {
    type: Boolean,
    default: false
  },
  isFavorite: {
    type: Boolean,
    default: false
  },
  // 封面图
  coverImage: {
    type: String,
    default: null
  },
  // 摘要（自动生成或手动设置）
  excerpt: {
    type: String,
    maxlength: 300,
    default: ''
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// 索引
NoteSchema.index({ title: 'text', content: 'text' });  // 全文搜索
NoteSchema.index({ author: 1, createdAt: -1 });
NoteSchema.index({ tags: 1 });
NoteSchema.index({ isPublic: 1, isDeleted: 1 });
NoteSchema.index({ likesCount: -1 });  // 热门笔记

// Virtual field: 是否点赞
NoteSchema.virtual('isLiked').get(function() {
  return false; // 需要通过上下文判断
});

// 保存前自动生成摘要
NoteSchema.pre('save', function(next) {
  if (this.isModified('content') && !this.excerpt) {
    // 从内容中提取前200个字符作为摘要
    const text = this.content.replace(/[#*`>\-\[\]]/g, '').trim();
    this.excerpt = text.substring(0, 200) + (text.length > 200 ? '...' : '');
  }
  next();
});

// 软删除
NoteSchema.methods.softDelete = async function() {
  this.isDeleted = true;
  await this.save();
};

module.exports = mongoose.model('Note', NoteSchema);
```

`backend/src/models/Tag.js` - 标签模型
```javascript
const mongoose = require('mongoose');

const TagSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, '标签名不能为空'],
    unique: true,
    trim: true,
    maxlength: [20, '标签名最多20个字符']
  },
  color: {
    type: String,
    default: '#409EFF',
    match: [/^#[0-9A-Fa-f]{6}$/, '颜色格式不正确']
  },
  icon: {
    type: String,
    default: 'tag'
  },
  description: {
    type: String,
    maxlength: 100,
    default: ''
  },
  // 使用次数
  usageCount: {
    type: Number,
    default: 0
  },
  // 创建者（系统标签无创建者）
  createdBy: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    default: null
  }
}, {
  timestamps: true
});

// 索引
TagSchema.index({ name: 1 });
TagSchema.index({ usageCount: -1 });

// 使用标签（增加计数）
TagSchema.methods.incrementUsage = async function() {
  this.usageCount += 1;
  await this.save();
};

module.exports = mongoose.model('Tag', TagSchema);
```

`backend/src/models/Comment.js` - 评论模型
```javascript
const mongoose = require('mongoose');

const CommentSchema = new mongoose.Schema({
  content: {
    type: String,
    required: [true, '评论内容不能为空'],
    maxlength: [1000, '评论最多1000个字符']
  },
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  note: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Note',
    required: true
  },
  // 回复功能
  parent: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Comment',
    default: null
  },
  // 回复对象
  replyTo: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    default: null
  },
  // 点赞
  likes: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User'
  }],
  likesCount: {
    type: Number,
    default: 0
  },
  isDeleted: {
    type: Boolean,
    default: false
  }
}, {
  timestamps: true
});

// 索引
CommentSchema.index({ note: 1, createdAt: -1 });
CommentSchema.index({ author: 1 });
CommentSchema.index({ parent: 1 });

module.exports = mongoose.model('Comment', CommentSchema);
```

---

### Day 3：Express 路由与中间件 ⭐⭐

**目标**：实现完整的 Express API

**学习内容**：
- Express 路由设计
- 中间件编写
- JWT 认证中间件
- 错误处理

**产出**：完整的 API 路由和控制器

**核心代码**：

`backend/src/middleware/auth.js` - 认证中间件
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// 保护路由：需要登录
const protect = async (req, res, next) => {
  try {
    let token;
    
    // 从 Header 获取 Token
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    }
    
    if (!token) {
      return res.status(401).json({
        success: false,
        message: '请先登录'
      });
    }
    
    // 验证 Token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 获取用户
    const user = await User.findById(decoded.id);
    
    if (!user) {
      return res.status(401).json({
        success: false,
        message: '用户不存在'
      });
    }
    
    // 挂载到 req
    req.user = user;
    next();
    
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({
        success: false,
        message: 'Token 无效'
      });
    }
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({
        success: false,
        message: 'Token 已过期'
      });
    }
    next(error);
  }
};

// 可选认证：获取用户但不强制
const optionalAuth = async (req, res, next) => {
  try {
    let token;
    
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    }
    
    if (token) {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      const user = await User.findById(decoded.id);
      if (user) {
        req.user = user;
      }
    }
    
    next();
  } catch (error) {
    // 忽略错误，继续执行
    next();
  }
};

// 管理员权限
const admin = (req, res, next) => {
  if (req.user && req.user.isAdmin) {
    next();
  } else {
    res.status(403).json({
      success: false,
      message: '需要管理员权限'
    });
  }
};

// 限流中间件（简单版）
const rateLimitMap = new Map();

const rateLimit = (options = {}) => {
  const { windowMs = 60000, max = 100 } = options;
  
  return (req, res, next) => {
    const key = req.ip;
    const now = Date.now();
    
    // 获取或初始化记录
    let record = rateLimitMap.get(key);
    if (!record) {
      record = { count: 0, resetTime: now + windowMs };
      rateLimitMap.set(key, record);
    }
    
    // 检查是否需要重置
    if (now > record.resetTime) {
      record.count = 0;
      record.resetTime = now + windowMs;
    }
    
    // 检查限制
    if (record.count >= max) {
      return res.status(429).json({
        success: false,
        message: '请求过于频繁，请稍后再试',
        retryAfter: Math.ceil((record.resetTime - now) / 1000)
      });
    }
    
    record.count++;
    next();
  };
};

// 清理过期记录（定期）
setInterval(() => {
  const now = Date.now();
  for (const [key, record] of rateLimitMap.entries()) {
    if (now > record.resetTime) {
      rateLimitMap.delete(key);
    }
  }
}, 60000);

module.exports = { protect, optionalAuth, admin, rateLimit };
```

`backend/src/middleware/errorHandler.js` - 错误处理
```javascript
// 全局错误处理中间件
const errorHandler = (err, req, res, next) => {
  console.error('Error:', err);
  
  let error = { ...err };
  error.message = err.message;
  
  // Mongoose 错误处理
  if (err.name === 'ValidationError') {
    const messages = Object.values(err.errors).map(val => val.message);
    error = {
      success: false,
      message: messages.join(', '),
      statusCode: 400
    };
  }
  
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    error = {
      success: false,
      message: `${field} 已存在`,
      statusCode: 400
    };
  }
  
  if (err.name === 'CastError') {
    error = {
      success: false,
      message: '资源不存在',
      statusCode: 404
    };
  }
  
  res.status(error.statusCode || 500).json({
    success: false,
    message: error.message || '服务器错误',
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};

// 404 处理
const notFound = (req, res, next) => {
  res.status(404).json({
    success: false,
    message: `找不到 ${req.originalUrl}`
  });
};

// Async 包装器
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

module.exports = { errorHandler, notFound, asyncHandler };
```

`backend/src/controllers/authController.js` - 认证控制器
```javascript
const User = require('../models/User');
const { asyncHandler } = require('../middleware/errorHandler');

// @desc    用户注册
// @route   POST /api/auth/register
// @access  Public
exports.register = asyncHandler(async (req, res) => {
  const { username, email, password } = req.body;
  
  // 检查用户是否存在
  const existingUser = await User.findOne({
    $or: [{ email }, { username }]
  });
  
  if (existingUser) {
    return res.status(400).json({
      success: false,
      message: existingUser.email === email ? '邮箱已被注册' : '用户名已被使用'
    });
  }
  
  // 创建用户
  const user = await User.create({
    username,
    email,
    password
  });
  
  // 生成 Token
  const token = user.generateToken();
  
  res.status(201).json({
    success: true,
    data: {
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        avatar: user.avatar,
        bio: user.bio
      },
      token
    }
  });
});

// @desc    用户登录
// @route   POST /api/auth/login
// @access  Public
exports.login = asyncHandler(async (req, res) => {
  const { username, password } = req.body;
  
  // 查找用户
  const user = await User.findOne({ username }).select('+password');
  
  if (!user) {
    return res.status(401).json({
      success: false,
      message: '用户名或密码错误'
    });
  }
  
  // 验证密码
  const isMatch = await user.comparePassword(password);
  
  if (!isMatch) {
    return res.status(401).json({
      success: false,
      message: '用户名或密码错误'
    });
  }
  
  // 生成 Token
  const token = user.generateToken();
  
  res.json({
    success: true,
    data: {
      user: {
        id: user._id,
        username: user.username,
        email: user.email,
        avatar: user.avatar,
        bio: user.bio
      },
      token
    }
  });
});

// @desc    获取当前用户
// @route   GET /api/auth/me
// @access  Private
exports.getMe = asyncHandler(async (req, res) => {
  const user = await User.findById(req.user._id)
    .populate('followers', 'username avatar')
    .populate('following', 'username avatar');
  
  res.json({
    success: true,
    data: user
  });
});

// @desc    更新用户信息
// @route   PUT /api/auth/profile
// @access  Private
exports.updateProfile = asyncHandler(async (req, res) => {
  const { username, bio, avatar } = req.body;
  
  const updateFields = {};
  if (username) updateFields.username = username;
  if (bio !== undefined) updateFields.bio = bio;
  if (avatar !== undefined) updateFields.avatar = avatar;
  
  const user = await User.findByIdAndUpdate(
    req.user._id,
    updateFields,
    { new: true, runValidators: true }
  );
  
  res.json({
    success: true,
    data: user
  });
});

// @desc    更新密码
// @route   PUT /api/auth/password
// @access  Private
exports.updatePassword = asyncHandler(async (req, res) => {
  const { currentPassword, newPassword } = req.body;
  
  const user = await User.findById(req.user._id).select('+password');
  
  // 验证当前密码
  const isMatch = await user.comparePassword(currentPassword);
  if (!isMatch) {
    return res.status(400).json({
      success: false,
      message: '当前密码不正确'
    });
  }
  
  user.password = newPassword;
  await user.save();
  
  const token = user.generateToken();
  
  res.json({
    success: true,
    data: { token }
  });
});
```

`backend/src/controllers/noteController.js` - 笔记控制器
```javascript
const Note = require('../models/Note');
const Tag = require('../models/Tag');
const User = require('../models/User');
const { asyncHandler } = require('../middleware/errorHandler');

// @desc    获取笔记列表
// @route   GET /api/notes
// @access  Public
exports.getNotes = asyncHandler(async (req, res) => {
  const { 
    page = 1, 
    limit = 20, 
    search, 
    tag, 
    author,
    sort = '-createdAt'
  } = req.query;
  
  const query = { isDeleted: false };
  
  // 搜索
  if (search) {
    query.$text = { $search: search };
  }
  
  // 标签筛选
  if (tag) {
    query.tags = tag;
  }
  
  // 作者筛选
  if (author) {
    query.author = author;
  }
  
  // 只显示公开笔记（除非是自己的）
  if (!req.user || req.user._id.toString() !== author) {
    query.isPublic = true;
  }
  
  const total = await Note.countDocuments(query);
  const notes = await Note.find(query)
    .populate('author', 'username avatar bio')
    .populate('tags', 'name color')
    .sort(sort)
    .skip((page - 1) * limit)
    .limit(parseInt(limit));
  
  res.json({
    success: true,
    data: {
      notes,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit)
      }
    }
  });
});

// @desc    获取单个笔记
// @route   GET /api/notes/:id
// @access  Public
exports.getNote = asyncHandler(async (req, res) => {
  const note = await Note.findById(req.params.id)
    .populate('author', 'username avatar bio')
    .populate('tags', 'name color')
    .populate('likes', 'username avatar');
  
  if (!note || note.isDeleted) {
    return res.status(404).json({
      success: false,
      message: '笔记不存在'
    });
  }
  
  // 检查访问权限
  if (!note.isPublic) {
    if (!req.user || note.author._id.toString() !== req.user._id.toString()) {
      return res.status(403).json({
        success: false,
        message: '无权访问该笔记'
      });
    }
  }
  
  // 增加浏览数
  note.views += 1;
  await note.save();
  
  // 标记是否点赞
  if (req.user) {
    note.isLiked = note.likes.some(
      like => like._id.toString() === req.user._id
    );
  }
  
  res.json({
    success: true,
    data: note
  });
});

// @desc    创建笔记
// @route   POST /api/notes
// @access  Private
exports.createNote = asyncHandler(async (req, res) => {
  const { title, content, contentHtml, tags, isPublic, coverImage } = req.body;
  
  // 处理标签
  let tagIds = [];
  if (tags && tags.length > 0) {
    // 创建不存在的标签
    const tagDocs = await Promise.all(
      tags.map(async (tagName) => {
        let tag = await Tag.findOne({ name: tagName });
        if (!tag) {
          tag = await Tag.create({ name: tagName });
        }
        return tag;
      })
    );
    tagIds = tagDocs.map(t => t._id);
  }
  
  const note = await Note.create({
    title,
    content,
    contentHtml,
    author: req.user._id,
    tags: tagIds,
    isPublic,
    coverImage
  });
  
  // 更新用户统计
  await User.findByIdAndUpdate(req.user._id, {
    $inc: { notesCount: 1 }
  });
  
  // 更新标签使用次数
  if (tagIds.length > 0) {
    await Tag.updateMany(
      { _id: { $in: tagIds } },
      { $inc: { usageCount: 1 } }
    );
  }
  
  const populatedNote = await Note.findById(note._id)
    .populate('author', 'username avatar')
    .populate('tags', 'name color');
  
  res.status(201).json({
    success: true,
    data: populatedNote
  });
});

// @desc    更新笔记
// @route   PUT /api/notes/:id
// @access  Private
exports.updateNote = asyncHandler(async (req, res) => {
  let note = await Note.findById(req.params.id);
  
  if (!note || note.isDeleted) {
    return res.status(404).json({
      success: false,
      message: '笔记不存在'
    });
  }
  
  // 检查所有权
  if (note.author.toString() !== req.user._id.toString()) {
    return res.status(403).json({
      success: false,
      message: '无权修改该笔记'
    });
  }
  
  const { title, content, contentHtml, tags, isPublic, coverImage, excerpt } = req.body;
  
  // 处理标签
  let tagIds = note.tags;
  if (tags) {
    const tagDocs = await Promise.all(
      tags.map(async (tagName) => {
        let tag = await Tag.findOne({ name: tagName });
        if (!tag) {
          tag = await Tag.create({ name: tagName });
        }
        return tag;
      })
    );
    tagIds = tagDocs.map(t => t._id);
  }
  
  note = await Note.findByIdAndUpdate(
    req.params.id,
    {
      title,
      content,
      contentHtml,
      tags: tagIds,
      isPublic,
      coverImage,
      excerpt
    },
    { new: true, runValidators: true }
  )
    .populate('author', 'username avatar')
    .populate('tags', 'name color');
  
  res.json({
    success: true,
    data: note
  });
});

// @desc    删除笔记
// @route   DELETE /api/notes/:id
// @access  Private
exports.deleteNote = asyncHandler(async (req, res) => {
  const note = await Note.findById(req.params.id);
  
  if (!note || note.isDeleted) {
    return res.status(404).json({
      success: false,
      message: '笔记不存在'
    });
  }
  
  // 检查所有权
  if (note.author.toString() !== req.user._id.toString()) {
    return res.status(403).json({
      success: false,
      message: '无权删除该笔记'
    });
  }
  
  // 软删除
  note.isDeleted = true;
  await note.save();
  
  // 更新统计
  await User.findByIdAndUpdate(req.user._id, {
    $inc: { notesCount: -1 }
  });
  
  res.json({
    success: true,
    message: '笔记已删除'
  });
});

// @desc    点赞笔记
// @route   POST /api/notes/:id/like
// @access  Private
exports.likeNote = asyncHandler(async (req, res) => {
  const note = await Note.findById(req.params.id);
  
  if (!note || note.isDeleted) {
    return res.status(404).json({
      success: false,
      message: '笔记不存在'
    });
  }
  
  const userId = req.user._id;
  const isLiked = note.likes.includes(userId);
  
  if (isLiked) {
    // 取消点赞
    note.likes = note.likes.filter(id => id.toString() !== userId.toString());
    note.likesCount -= 1;
  } else {
    // 点赞
    note.likes.push(userId);
    note.likesCount += 1;
  }
  
  await note.save();
  
  // 更新作者统计
  if (note.author.toString() !== userId.toString()) {
    await User.findByIdAndUpdate(note.author, {
      $inc: { likesReceived: isLiked ? -1 : 1 }
    });
  }
  
  res.json({
    success: true,
    data: {
      isLiked: !isLiked,
      likesCount: note.likesCount
    }
  });
});
```

`backend/src/routes/auth.js` - 认证路由
```javascript
const express = require('express');
const router = express.Router();
const { protect } = require('../middleware/auth');
const { register, login, getMe, updateProfile, updatePassword } = require('../controllers/authController');

// 公开路由
router.post('/register', register);
router.post('/login', login);

// 保护路由
router.get('/me', protect, getMe);
router.put('/profile', protect, updateProfile);
router.put('/password', protect, updatePassword);

module.exports = router;
```

`backend/src/routes/notes.js` - 笔记路由
```javascript
const express = require('express');
const router = express.Router();
const { protect, optionalAuth } = require('../middleware/auth');
const {
  getNotes,
  getNote,
  createNote,
  updateNote,
  deleteNote,
  likeNote
} = require('../controllers/noteController');

// 公开路由
router.get('/', optionalAuth, getNotes);
router.get('/:id', optionalAuth, getNote);

// 保护路由
router.post('/', protect, createNote);
router.put('/:id', protect, updateNote);
router.delete('/:id', protect, deleteNote);
router.post('/:id/like', protect, likeNote);

module.exports = router;
```

`backend/src/index.js` - 应用入口
```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
const dotenv = require('dotenv');
const connectDB = require('./config/database');
const { errorHandler, notFound } = require('./middleware/errorHandler');
const { rateLimit } = require('./middleware/auth');

// 加载环境变量
dotenv.config();

// 连接数据库
connectDB();

const app = express();

// 安全中间件
app.use(helmet());

// CORS 配置
app.use(cors({
  origin: process.env.CLIENT_URL || 'http://localhost:5173',
  credentials: true
}));

// 请求日志
if (process.env.NODE_ENV === 'development') {
  app.use(morgan('dev'));
}

// 解析 JSON
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// 限流
app.use('/api', rateLimit({ windowMs: 60 * 1000, max: 100 }));

// 路由
app.use('/api/auth', require('./routes/auth'));
app.use('/api/notes', require('./routes/notes'));
app.use('/api/tags', require('./routes/tags'));
app.use('/api/users', require('./routes/users'));

// 健康检查
app.get('/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

// 错误处理
app.use(notFound);
app.use(errorHandler);

const PORT = process.env.PORT || 5000;

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

---

### Day 4：Vue 3 前端开发 ⭐⭐

**目标**：完成 Vue 3 前端核心功能

**学习内容**：
- Vue 3 Composition API
- Pinia 状态管理
- Vue Router 路由
- Element Plus 组件

**产出**：完整的前端应用

**核心代码**：

`frontend/src/stores/auth.ts` - 认证 Store
```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { login as apiLogin, register as apiRegister, getMe } from '@/api/auth';
import type { User, LoginForm, RegisterForm } from '@/types';

export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null);
  const token = ref<string | null>(localStorage.getItem('token'));
  const loading = ref(false);
  const error = ref<string | null>(null);

  const isLoggedIn = computed(() => !!token.value && !!user.value);

  // 登录
  async function login(form: LoginForm) {
    loading.value = true;
    error.value = null;
    
    try {
      const response = await apiLogin(form);
      token.value = response.data.token;
      user.value = response.data.user;
      localStorage.setItem('token', response.data.token);
      return true;
    } catch (err: any) {
      error.value = err.response?.data?.message || '登录失败';
      return false;
    } finally {
      loading.value = false;
    }
  }

  // 注册
  async function register(form: RegisterForm) {
    loading.value = true;
    error.value = null;
    
    try {
      await apiRegister(form);
      return true;
    } catch (err: any) {
      error.value = err.response?.data?.message || '注册失败';
      return false;
    } finally {
      loading.value = false;
    }
  }

  // 获取当前用户
  async function fetchUser() {
    if (!token.value) return;
    
    try {
      const response = await getMe();
      user.value = response.data;
    } catch {
      logout();
    }
  }

  // 登出
  function logout() {
    user.value = null;
    token.value = null;
    localStorage.removeItem('token');
  }

  // 初始化时获取用户
  if (token.value) {
    fetchUser();
  }

  return {
    user,
    token,
    loading,
    error,
    isLoggedIn,
    login,
    register,
    logout,
    fetchUser
  };
});
```

`frontend/src/stores/notes.ts` - 笔记 Store
```typescript
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { 
  getNotes, 
  getNote as apiGetNote, 
  createNote, 
  updateNote, 
  deleteNote,
  likeNote as apiLikeNote 
} from '@/api/notes';
import type { Note, NoteForm, NoteListResponse, Pagination } from '@/types';

export const useNotesStore = defineStore('notes', () => {
  const notes = ref<Note[]>([]);
  const currentNote = ref<Note | null>(null);
  const loading = ref(false);
  const pagination = ref<Pagination>({
    page: 1,
    limit: 20,
    total: 0,
    pages: 0
  });
  const filters = ref({
    search: '',
    tag: '',
    author: ''
  });

  // 获取笔记列表
  async function fetchNotes(params?: {
    page?: number;
    limit?: number;
    search?: string;
    tag?: string;
  }) {
    loading.value = true;
    
    try {
      const response = await getNotes({
        page: params?.page || pagination.value.page,
        limit: params?.limit || pagination.value.limit,
        search: params?.search || filters.value.search,
        tag: params?.tag || filters.value.tag
      });
      
      notes.value = response.data.notes;
      pagination.value = response.data.pagination;
    } catch (err) {
      console.error('获取笔记失败:', err);
    } finally {
      loading.value = false;
    }
  }

  // 获取单个笔记
  async function fetchNote(id: string) {
    loading.value = true;
    
    try {
      const response = await apiGetNote(id);
      currentNote.value = response.data;
      return response.data;
    } catch (err) {
      console.error('获取笔记详情失败:', err);
      return null;
    } finally {
      loading.value = false;
    }
  }

  // 创建笔记
  async function addNote(form: NoteForm): Promise<Note | null> {
    loading.value = true;
    
    try {
      const response = await createNote(form);
      notes.value.unshift(response.data);
      return response.data;
    } catch (err) {
      console.error('创建笔记失败:', err);
      return null;
    } finally {
      loading.value = false;
    }
  }

  // 更新笔记
  async function editNote(id: string, form: Partial<NoteForm>): Promise<Note | null> {
    loading.value = true;
    
    try {
      const response = await updateNote(id, form);
      const index = notes.value.findIndex(n => n.id === id);
      if (index !== -1) {
        notes.value[index] = response.data;
      }
      if (currentNote.value?.id === id) {
        currentNote.value = response.data;
      }
      return response.data;
    } catch (err) {
      console.error('更新笔记失败:', err);
      return null;
    } finally {
      loading.value = false;
    }
  }

  // 删除笔记
  async function removeNote(id: string): Promise<boolean> {
    loading.value = true;
    
    try {
      await deleteNote(id);
      notes.value = notes.value.filter(n => n.id !== id);
      if (currentNote.value?.id === id) {
        currentNote.value = null;
      }
      return true;
    } catch (err) {
      console.error('删除笔记失败:', err);
      return false;
    } finally {
      loading.value = false;
    }
  }

  // 点赞笔记
  async function toggleLike(id: string): Promise<boolean> {
    try {
      const response = await apiLikeNote(id);
      const note = notes.value.find(n => n.id === id);
      if (note) {
        note.isLiked = response.data.isLiked;
        note.likesCount = response.data.likesCount;
      }
      if (currentNote.value?.id === id) {
        currentNote.value.isLiked = response.data.isLiked;
        currentNote.value.likesCount = response.data.likesCount;
      }
      return true;
    } catch (err) {
      console.error('点赞失败:', err);
      return false;
    }
  }

  return {
    notes,
    currentNote,
    loading,
    pagination,
    filters,
    fetchNotes,
    fetchNote,
    addNote,
    editNote,
    removeNote,
    toggleLike
  };
});
```

`frontend/src/pages/Login.vue` - 登录页面
```vue
<template>
  <div class="login-page">
    <div class="login-card">
      <h1 class="title">NeonStack</h1>
      <p class="subtitle">社交化笔记管理平台</p>
      
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        class="login-form"
        @submit.prevent="handleLogin"
      >
        <el-form-item prop="username">
          <el-input
            v-model="form.username"
            placeholder="用户名"
            prefix-icon="User"
            size="large"
          />
        </el-form-item>
        
        <el-form-item prop="password">
          <el-input
            v-model="form.password"
            type="password"
            placeholder="密码"
            prefix-icon="Lock"
            size="large"
            show-password
            @keyup.enter="handleLogin"
          />
        </el-form-item>
        
        <el-button
          type="primary"
          size="large"
          :loading="authStore.loading"
          class="login-btn"
          @click="handleLogin"
        >
          登录
        </el-button>
      </el-form>
      
      <div class="footer">
        还没有账号？
        <router-link to="/register">立即注册</router-link>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { useAuthStore } from '@/stores/auth';
import { ElMessage } from 'element-plus';
import type { FormInstance, FormRules } from 'element-plus';

const router = useRouter();
const authStore = useAuthStore();
const formRef = ref<FormInstance>();

const form = reactive({
  username: '',
  password: ''
});

const rules: FormRules = {
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '用户名长度在 3 到 20 个字符', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, message: '密码至少 6 个字符', trigger: 'blur' }
  ]
};

async function handleLogin() {
  if (!formRef.value) return;
  
  await formRef.value.validate(async (valid) => {
    if (!valid) return;
    
    const success = await authStore.login(form);
    if (success) {
      ElMessage.success('登录成功');
      router.push('/');
    } else {
      ElMessage.error(authStore.error || '登录失败');
    }
  });
}
</script>

<style scoped lang="scss">
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.login-card {
  width: 400px;
  padding: 40px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

.title {
  font-size: 28px;
  font-weight: 700;
  text-align: center;
  color: #333;
  margin-bottom: 8px;
}

.subtitle {
  text-align: center;
  color: #666;
  margin-bottom: 32px;
}

.login-form {
  margin-bottom: 24px;
}

.login-btn {
  width: 100%;
  height: 48px;
  font-size: 16px;
}

.footer {
  text-align: center;
  color: #666;
  
  a {
    color: #409eff;
    text-decoration: none;
    
    &:hover {
      text-decoration: underline;
    }
  }
}
</style>
```

`frontend/src/pages/NoteDetail.vue` - 笔记详情页
```vue
<template>
  <div class="note-detail" v-loading="loading">
    <div v-if="note">
      <!-- 头部 -->
      <header class="note-header">
        <div class="meta">
          <el-tag
            v-for="tag in note.tags"
            :key="tag.id"
            :color="tag.color"
            effect="dark"
            size="small"
          >
            {{ tag.name }}
          </el-tag>
          <span class="date">{{ formatDate(note.createdAt) }}</span>
        </div>
        <h1 class="title">{{ note.title }}</h1>
        <div class="author">
          <el-avatar :src="note.author.avatar" :size="32">
            {{ note.author.username }}
          </el-avatar>
          <span class="username">{{ note.author.username }}</span>
        </div>
      </header>
      
      <!-- 内容 -->
      <article class="note-content" v-html="note.contentHtml"></article>
      
      <!-- 底部操作 -->
      <footer class="note-footer">
        <div class="actions">
          <el-button
            :type="note.isLiked ? 'primary' : 'default'"
            @click="handleLike"
          >
            <el-icon><Star /></el-icon>
            {{ note.likesCount }}
          </el-button>
          <el-button @click="handleShare">
            <el-icon><Share /></el-icon>
            分享
          </el-button>
          <el-button 
            v-if="isOwner" 
            type="warning"
            @click="handleEdit"
          >
            <el-icon><Edit /></el-icon>
            编辑
          </el-button>
          <el-button 
            v-if="isOwner" 
            type="danger"
            @click="handleDelete"
          >
            <el-icon><Delete /></el-icon>
            删除
          </el-button>
        </div>
        <div class="stats">
          <span><el-icon><View /></el-icon> {{ note.views }}</span>
          <span><el-icon><ChatLineSquare /></el-icon> {{ note.commentsCount }}</span>
        </div>
      </footer>
    </div>
    
    <el-empty v-else description="笔记不存在" />
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import { useNotesStore } from '@/stores/notes';
import { useAuthStore } from '@/stores/auth';
import { ElMessage, ElMessageBox } from 'element-plus';
import { formatDate } from '@/utils/date';

const route = useRoute();
const router = useRouter();
const notesStore = useNotesStore();
const authStore = useAuthStore();

const loading = computed(() => notesStore.loading);
const note = computed(() => notesStore.currentNote);
const isOwner = computed(() => 
  authStore.user && note.value && authStore.user.id === note.value.author.id
);

onMounted(async () => {
  const id = route.params.id as string;
  await notesStore.fetchNote(id);
});

async function handleLike() {
  if (!authStore.isLoggedIn) {
    ElMessage.warning('请先登录');
    router.push('/login');
    return;
  }
  await notesStore.toggleLike(note.value!.id);
}

function handleShare() {
  const url = window.location.href;
  navigator.clipboard.writeText(url);
  ElMessage.success('链接已复制到剪贴板');
}

function handleEdit() {
  router.push(`/editor/${note.value!.id}`);
}

async function handleDelete() {
  try {
    await ElMessageBox.confirm('确定要删除这篇笔记吗？', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    });
    
    await notesStore.removeNote(note.value!.id);
    ElMessage.success('删除成功');
    router.push('/');
  } catch {
    // 用户取消
  }
}
</script>

<style scoped lang="scss">
.note-detail {
  max-width: 800px;
  margin: 0 auto;
  padding: 40px 20px;
}

.note-header {
  margin-bottom: 32px;
  
  .meta {
    display: flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 16px;
    
    .date {
      color: #999;
      font-size: 14px;
    }
  }
  
  .title {
    font-size: 32px;
    font-weight: 700;
    color: #333;
    margin-bottom: 16px;
    line-height: 1.4;
  }
  
  .author {
    display: flex;
    align-items: center;
    gap: 12px;
    
    .username {
      font-weight: 500;
      color: #333;
    }
  }
}

.note-content {
  font-size: 16px;
  line-height: 1.8;
  color: #333;
  
  :deep(h1), :deep(h2), :deep(h3) {
    margin-top: 24px;
    margin-bottom: 16px;
    font-weight: 600;
  }
  
  :deep(code) {
    background: #f5f5f5;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: monospace;
  }
  
  :deep(pre) {
    background: #f5f5f5;
    padding: 16px;
    border-radius: 8px;
    overflow-x: auto;
  }
  
  :deep(img) {
    max-width: 100%;
    border-radius: 8px;
  }
  
  :deep(blockquote) {
    border-left: 4px solid #409eff;
    padding-left: 16px;
    margin: 16px 0;
    color: #666;
  }
}

.note-footer {
  margin-top: 40px;
  padding-top: 24px;
  border-top: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  
  .actions {
    display: flex;
    gap: 12px;
  }
  
  .stats {
    display: flex;
    gap: 20px;
    color: #999;
    
    span {
      display: flex;
      align-items: center;
      gap: 4px;
    }
  }
}
</style>
```

---

### Day 5：Markdown 编辑器实现 ⭐⭐⭐

**目标**：实现完整的 Markdown 编辑器

**学习内容**：
- Markdown 语法和渲染
- 富文本编辑器集成
- 实时预览
- 代码高亮

**产出**：完整的 Markdown 编辑器组件

**核心代码**：

`frontend/src/components/MarkdownEditor.vue` - Markdown 编辑器
```vue
<template>
  <div class="markdown-editor">
    <div class="toolbar">
      <div class="toolbar-left">
        <el-button-group>
          <el-tooltip content="标题">
            <el-button @click="insertAtCursor('# ')"><strong>H</strong></el-button>
          </el-tooltip>
          <el-tooltip content="粗体">
            <el-button @click="insertWrap('**')"><strong>B</strong></el-button>
          </el-tooltip>
          <el-tooltip content="斜体">
            <el-button @click="insertWrap('*')"><em>I</em></el-button>
          </el-tooltip>
          <el-tooltip content="删除线">
            <el-button @click="insertWrap('~~')"><s>S</s></el-button>
          </el-tooltip>
        </el-button-group>
        
        <el-divider direction="vertical" />
        
        <el-button-group>
          <el-tooltip content="链接">
            <el-button @click="insertLink">
              <el-icon><Link /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="图片">
            <el-button @click="insertImage">
              <el-icon><Picture /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="代码">
            <el-button @click="insertCode">
              <el-icon><Document /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="代码块">
            <el-button @click="insertCodeBlock">
              <el-icon><Box /></el-icon>
            </el-button>
          </el-tooltip>
        </el-button-group>
        
        <el-divider direction="vertical" />
        
        <el-button-group>
          <el-tooltip content="无序列表">
            <el-button @click="insertAtCursor('- ')">
              <el-icon><List /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="有序列表">
            <el-button @click="insertAtCursor('1. ')">
              <el-icon><List /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="引用">
            <el-button @click="insertAtCursor('> ')">
              <el-icon><ChatLineSquare /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="分割线">
            <el-button @click="insertAtCursor('\n---\n')">
              <el-icon><Minus /></el-icon>
            </el-button>
          </el-tooltip>
        </el-button-group>
        
        <el-divider direction="vertical" />
        
        <el-button-group>
          <el-tooltip content="表格">
            <el-button @click="insertTable">
              <el-icon><Grid /></el-icon>
            </el-button>
          </el-tooltip>
          <el-tooltip content="待办事项">
            <el-button @click="insertTodo">
              <el-icon><Finished /></el-icon>
            </el-button>
          </el-tooltip>
        </el-button-group>
      </div>
      
      <div class="toolbar-right">
        <el-radio-group v-model="viewMode" size="small">
          <el-radio-button value="edit">编辑</el-radio-button>
          <el-radio-button value="preview">预览</el-radio-button>
          <el-radio-button value="both">分屏</el-radio-button>
        </el-radio-group>
      </div>
    </div>
    
    <div class="editor-container" :class="viewMode">
      <div class="editor-pane" v-show="viewMode !== 'preview'">
        <textarea
          ref="textareaRef"
          v-model="content"
          class="editor-textarea"
          :placeholder="placeholder"
          @input="handleInput"
          @keydown="handleKeydown"
        />
      </div>
      
      <div class="preview-pane" v-show="viewMode !== 'edit'" v-html="renderedContent"></div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue';
import MarkdownIt from 'markdown-it';
import hljs from 'highlight.js';
import 'highlight.js/styles/github.css';

interface Props {
  modelValue: string;
  placeholder?: string;
}

const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  placeholder: '开始编写你的笔记...'
});

const emit = defineEmits<{
  (e: 'update:modelValue', value: string): void;
}>();

const textareaRef = ref<HTMLTextAreaElement>();
const content = ref(props.modelValue);
const viewMode = ref<'edit' | 'preview' | 'both'>('both');

// Markdown 配置
const md = new MarkdownIt({
  html: true,
  linkify: true,
  typographer: true,
  highlight: (str: string, lang: string) => {
    if (lang && hljs.getLanguage(lang)) {
      try {
        return hljs.highlight(str, { language: lang, ignoreIllegals: true }).value;
      } catch {}
    }
    return '';
  }
});

// 渲染内容
const renderedContent = computed(() => {
  return md.render(content.value || '');
});

// 监听外部变化
watch(() => props.modelValue, (newVal) => {
  if (newVal !== content.value) {
    content.value = newVal;
  }
});

// 输入事件
function handleInput() {
  emit('update:modelValue', content.value);
}

// 快捷键处理
function handleKeydown(e: KeyboardEvent) {
  // Tab 插入空格
  if (e.key === 'Tab') {
    e.preventDefault();
    insertAtCursor('  ');
  }
  
  // Ctrl+B 粗体
  if (e.ctrlKey && e.key === 'b') {
    e.preventDefault();
    insertWrap('**');
  }
  
  // Ctrl+I 斜体
  if (e.ctrlKey && e.key === 'i') {
    e.preventDefault();
    insertWrap('*');
  }
}

// 插入文本
function insertAtCursor(prefix: string, suffix: string = '') {
  const textarea = textareaRef.value;
  if (!textarea) return;
  
  const start = textarea.selectionStart;
  const end = textarea.selectionEnd;
  const selected = content.value.substring(start, end);
  
  const newText = content.value.substring(0, start) + 
                  prefix + selected + suffix + 
                  content.value.substring(end);
  
  content.value = newText;
  emit('update:modelValue', content.value);
  
  // 恢复光标位置
  setTimeout(() => {
    textarea.focus();
    textarea.setSelectionRange(
      start + prefix.length,
      start + prefix.length + selected.length
    );
  }, 0);
}

// 包裹文本
function insertWrap(wrapper: string) {
  insertAtCursor(wrapper, wrapper);
}

// 插入链接
function insertLink() {
  const url = prompt('请输入链接地址:');
  if (!url) return;
  insertAtCursor(`[链接文本](${url})`);
}

// 插入图片
function insertImage() {
  const url = prompt('请输入图片地址:');
  if (!url) return;
  insertAtCursor(`![图片描述](${url})`);
}

// 插入代码
function insertCode() {
  insertAtCursor('`', '`');
}

// 插入代码块
function insertCodeBlock() {
  insertAtCursor('\n```javascript\n', '\n```\n');
}

// 插入表格
function insertTable() {
  const table = `
| 列1 | 列2 | 列3 |
| --- | --- | --- |
| 内容 | 内容 | 内容 |
`;
  insertAtCursor(table);
}

// 插入待办
function insertTodo() {
  insertAtCursor('- [ ] ');
}

defineExpose({
  focus: () => textareaRef.value?.focus(),
  getContent: () => content.value
});
</script>

<style scoped lang="scss">
.markdown-editor {
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  overflow: hidden;
}

.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  background: #f5f7fa;
  border-bottom: 1px solid #dcdfe6;
}

.editor-container {
  display: flex;
  min-height: 500px;
  
  &.edit {
    .editor-pane {
      width: 100%;
    }
  }
  
  &.preview {
    .preview-pane {
      width: 100%;
    }
  }
  
  &.both {
    .editor-pane {
      width: 50%;
      border-right: 1px solid #dcdfe6;
    }
    .preview-pane {
      width: 50%;
    }
  }
}

.editor-pane {
  display: flex;
}

.editor-textarea {
  width: 100%;
  padding: 16px;
  border: none;
  resize: none;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 14px;
  line-height: 1.6;
  outline: none;
  
  &::placeholder {
    color: #c0c4cc;
  }
}

.preview-pane {
  padding: 16px;
  overflow-y: auto;
  background: #fff;
  
  :deep(h1), :deep(h2), :deep(h3), :deep(h4) {
    margin-top: 24px;
    margin-bottom: 16px;
    font-weight: 600;
  }
  
  :deep(pre) {
    background: #f5f5f5;
    padding: 16px;
    border-radius: 4px;
    overflow-x: auto;
  }
  
  :deep(code) {
    background: #f5f5f5;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: monospace;
  }
  
  :deep(img) {
    max-width: 100%;
    border-radius: 4px;
  }
  
  :deep(blockquote) {
    border-left: 4px solid #409eff;
    padding-left: 16px;
    margin: 16px 0;
    color: #666;
  }
  
  :deep(table) {
    width: 100%;
    border-collapse: collapse;
    margin: 16px 0;
    
    th, td {
      border: 1px solid #dcdfe6;
      padding: 8px 12px;
      text-align: left;
    }
    
    th {
      background: #f5f7fa;
    }
  }
}
</style>
```

---

### Day 6：Docker 部署 ⭐⭐⭐

**目标**：完成 Docker 部署配置

**Docker 配置文件**：

`backend/Dockerfile`
```dockerfile
FROM node:18-alpine

WORKDIR /app

# 安装依赖
COPY package*.json ./
RUN npm ci --only=production

# 复制代码
COPY . .

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

EXPOSE 5000

CMD ["node", "src/index.js"]
```

`docker-compose.yml`
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:6
    container_name: neonstack-mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: mongopass123
    networks:
      - neonstack-network

  backend:
    build: ./backend
    container_name: neonstack-backend
    ports:
      - "5000:5000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://admin:mongopass123@mongodb:27017/neonstack?authSource=admin
      - JWT_SECRET=your-production-secret
      - JWT_EXPIRE=30d
      - CLIENT_URL=http://localhost:3000
    depends_on:
      - mongodb
    networks:
      - neonstack-network

  frontend:
    build: ./frontend
    container_name: neonstack-frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - neonstack-network

volumes:
  mongo_data:

networks:
  neonstack-network:
    driver: bridge
```

---

### Day 7：项目总结与面试准备 ⭐⭐⭐

**目标**：总结项目亮点，准备面试

---

## 📝 面试要点

### Q1：Express 和 Koa 的区别？

**参考答案**：
```
1. 异步处理：Express 使用回调，Koa 使用 async/await
2. 中间件：Express 线性，Koa 洋葱模型
3. 体积：Express 更重，Koa 更轻量
4. 错误处理：Express 同步捕获，Koa try/catch 支持
```

### Q2：MongoDB 与 PostgreSQL 的区别？

**参考答案**：
```
1. 数据模型：MongoDB 文档型，PostgreSQL 关系型
2. 灵活性：MongoDB schema-less，PostgreSQL 严格 schema
3. 事务：PostgreSQL 原生支持，MongoDB 需要配置
4. 适用场景：
   - MongoDB：快速迭代、内容管理、日志
   - PostgreSQL：金融、复杂查询、数据一致性
```

### Q3：JWT Token 的安全问题？

**参考答案**：
```
1. Token 泄露：使用 HTTPS 传输
2. Token 伪造：使用强密钥，定期轮换
3. Token 过期：短期 Access + 长期 Refresh
4. Token 撤销：黑名单或短过期时间
5. CSRF：结合 SameSite Cookie
```

### Q4：Vue 3 相比 Vue 2 的改进？

**参考答案**：
```
1. 性能：Proxy 代替 Object.defineProperty
2. Composition API：逻辑复用更灵活
3. TS 支持：更好的类型推断
4. 体积：更小的包体积，更快的渲染
5. 更好的 Tree-shaking
```

---

## ✅ 项目清单

```
NeonStack/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   └── database.js
│   │   ├── controllers/
│   │   │   ├── authController.js
│   │   │   ├── noteController.js
│   │   │   └── userController.js
│   │   ├── middleware/
│   │   │   ├── auth.js
│   │   │   └── errorHandler.js
│   │   ├── models/
│   │   │   ├── User.js
│   │   │   ├── Note.js
│   │   │   ├── Tag.js
│   │   │   └── Comment.js
│   │   ├── routes/
│   │   │   ├── auth.js
│   │   │   ├── notes.js
│   │   │   └── users.js
│   │   └── index.js
│   ├── package.json
│   ├── .env.example
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── stores/
│   │   ├── types/
│   │   ├── router/
│   │   └── App.vue
│   ├── package.json
│   └── Dockerfile
│
└── docker-compose.yml
```

---

**文档版本**：v1.0  
**更新日期**：2024
