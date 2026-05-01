# Android + Compose 方案三：社交动态 App

## 项目介绍

### 项目是什么
这是一个现代化的社交动态应用，类似小红书/Instagram，支持动态发布、图片浏览、点赞评论、用户资料等功能。

### 解决什么问题
- 浏览和发布动态
- 图片社交
- 点赞和评论互动
- 用户社交

### 核心技术亮点
| 技术 | 说明 |
|------|------|
| Jetpack Compose | 声明式 UI |
| Coil | 图片加载 |
| Hilt | 依赖注入 |
| Room | 本地数据库 |
| Retrofit | 网络请求 |
| Navigation Compose | 导航 |
| PullRefresh | 下拉刷新 |

### 适合场景
- 学习图片社交应用开发
- 展示项目
- 完整的社交应用架构

---

## 完整可运行代码

### 项目结构
```
SocialApp/
├── app/src/main/java/com/example/social/
│   ├── data/
│   │   ├── model/
│   │   │   ├── Post.kt
│   │   │   ├── User.kt
│   │   │   └── Comment.kt
│   │   ├── remote/
│   │   │   └── ApiService.kt
│   │   ├── repository/
│   │   │   └── PostRepository.kt
│   │   └── local/
│   │       └── LocalDatabase.kt
│   ├── di/
│   │   └── AppModule.kt
│   ├── ui/
│   │   ├── theme/
│   │   ├── components/
│   │   │   ├── PostCard.kt
│   │   │   ├── UserAvatar.kt
│   │   │   └── CommentSheet.kt
│   │   └── screens/
│   │       ├── home/
│   │       │   ├── HomeScreen.kt
│   │       │   └── HomeViewModel.kt
│   │       ├── post/
│   │       │   ├── PostDetailScreen.kt
│   │       │   └── PostDetailViewModel.kt
│   │       ├── profile/
│   │       │   ├── ProfileScreen.kt
│   │       │   └── ProfileViewModel.kt
│   │       └── create/
│   │           ├── CreatePostScreen.kt
│   │           └── CreatePostViewModel.kt
│   └── util/
│       └── ImagePicker.kt
```

### 1. 数据模型

```kotlin
// ===== Post.kt =====

package com.example.social.data.model

import androidx.room.*
import java.time.LocalDateTime

/**
 * 动态/帖子实体
 */
@Entity(
    tableName = "posts",
    indices = [
        Index(value = ["userId"]),
        Index(value = ["createdAt"])
    ]
)
data class Post(
    @PrimaryKey
    val id: String,           // 使用 UUID 作为 ID
    val userId: String,
    val userName: String,
    val userAvatar: String?,
    
    // 内容
    val content: String,
    val images: List<String>,  // 图片 URL 列表
    
    // 互动
    val likeCount: Int = 0,
    val commentCount: Int = 0,
    val shareCount: Int = 0,
    
    // 用户是否点赞（本地缓存）
    val isLiked: Boolean = false,
    
    // 位置（可选）
    val location: String? = null,
    
    // 时间
    val createdAt: LocalDateTime = LocalDateTime.now(),
    val updatedAt: LocalDateTime = LocalDateTime.now()
) {
    /**
     * 判断是否是新帖子（1小时内）
     */
    fun isNew(): Boolean {
        val oneHourAgo = LocalDateTime.now().minusHours(1)
        return createdAt.isAfter(oneHourAgo)
    }
    
    /**
     * 获取格式化的时间
     */
    fun getFormattedTime(): String {
        val now = LocalDateTime.now()
        val diff = java.time.Duration.between(createdAt, now)
        
        return when {
            diff.toMinutes() < 1 -> "刚刚"
            diff.toMinutes() < 60 -> "${diff.toMinutes()}分钟前"
            diff.toHours() < 24 -> "${diff.toHours()}小时前"
            diff.toDays() < 7 -> "${diff.toDays()}天前"
            else -> createdAt.format(java.time.format.DateTimeFormatter.ofPattern("MM-dd"))
        }
    }
}

/**
 * 用户实体
 */
@Entity(tableName = "users")
data class User(
    @PrimaryKey
    val id: String,
    val username: String,
    val displayName: String,
    val avatar: String?,
    val bio: String?,
    val postCount: Int = 0,
    val followerCount: Int = 0,
    val followingCount: Int = 0,
    val isFollowing: Boolean = false,
    val createdAt: LocalDateTime = LocalDateTime.now()
)

/**
 * 评论实体
 */
@Entity(
    tableName = "comments",
    indices = [Index(value = ["postId"])]
)
data class Comment(
    @PrimaryKey
    val id: String,
    val postId: String,
    val userId: String,
    val userName: String,
    val userAvatar: String?,
    val content: String,
    val likeCount: Int = 0,
    val isLiked: Boolean = false,
    val parentId: String? = null,  // 回复的评论 ID
    val createdAt: LocalDateTime = LocalDateTime.now()
)

/**
 * 点赞关系（用于本地缓存）
 */
@Entity(
    tableName = "likes",
    primaryKeys = ["userId", "postId"]
)
data class Like(
    val userId: String,
    val postId: String,
    val createdAt: LocalDateTime = LocalDateTime.now()
)
```

### 2. PostRepository.kt

```kotlin
// ===== PostRepository.kt =====

package com.example.social.data.repository

import com.example.social.data.model.*
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.map
import java.time.LocalDateTime
import java.util.UUID
import javax.inject.Inject
import javax.inject.Singleton

/**
 * 帖子仓库
 * 
 * 这个应用使用本地模拟数据
 * 实际项目中应该调用远程 API
 */
@Singleton
class PostRepository @Inject constructor() {
    
    // ===== 模拟数据存储 =====
    private val _posts = MutableStateFlow(generateMockPosts())
    private val _comments = MutableStateFlow(generateMockComments())
    private val _currentUser = MutableStateFlow(generateCurrentUser())
    
    // ===== 公开的 Flow =====
    
    /**
     * 获取所有帖子
     */
    fun getPosts(): Flow<List<Post>> = _posts
    
    /**
     * 获取单个帖子
     */
    fun getPost(postId: String): Flow<Post?> = _posts.map { posts ->
        posts.find { it.id == postId }
    }
    
    /**
     * 获取用户帖子
     */
    fun getUserPosts(userId: String): Flow<List<Post>> = _posts.map { posts ->
        posts.filter { it.userId == userId }
    }
    
    /**
     * 获取评论
     */
    fun getComments(postId: String): Flow<List<Comment>> = _comments.map { comments ->
        comments.filter { it.postId == postId }
    }
    
    /**
     * 获取当前用户
     */
    fun getCurrentUser(): Flow<User> = _currentUser
    
    // ===== 操作方法 =====
    
    /**
     * 创建帖子
     */
    suspend fun createPost(
        content: String,
        images: List<String>,
        location: String? = null
    ): Post {
        val user = _currentUser.value
        val post = Post(
            id = UUID.randomUUID().toString(),
            userId = user.id,
            userName = user.displayName,
            userAvatar = user.avatar,
            content = content,
            images = images,
            location = location,
            createdAt = LocalDateTime.now(),
            updatedAt = LocalDateTime.now()
        )
        
        // 添加到列表开头
        _posts.value = listOf(post) + _posts.value
        
        return post
    }
    
    /**
     * 点赞/取消点赞
     */
    suspend fun toggleLike(postId: String): Boolean {
        var isNowLiked = false
        
        _posts.value = _posts.value.map { post ->
            if (post.id == postId) {
                isNowLiked = !post.isLiked
                post.copy(
                    isLiked = isNowLiked,
                    likeCount = if (isNowLiked) post.likeCount + 1 else post.likeCount - 1
                )
            } else post
        }
        
        return isNowLiked
    }
    
    /**
     * 添加评论
     */
    suspend fun addComment(
        postId: String,
        content: String,
        parentId: String? = null
    ): Comment {
        val user = _currentUser.value
        val comment = Comment(
            id = UUID.randomUUID().toString(),
            postId = postId,
            userId = user.id,
            userName = user.displayName,
            userAvatar = user.avatar,
            content = content,
            parentId = parentId,
            createdAt = LocalDateTime.now()
        )
        
        _comments.value = _comments.value + comment
        
        // 更新帖子评论数
        _posts.value = _posts.value.map { post ->
            if (post.id == postId) {
                post.copy(commentCount = post.commentCount + 1)
            } else post
        }
        
        return comment
    }
    
    /**
     * 删除帖子
     */
    suspend fun deletePost(postId: String) {
        _posts.value = _posts.value.filter { it.id != postId }
        _comments.value = _comments.value.filter { it.postId != postId }
    }
    
    // ===== 模拟数据生成 =====
    
    private fun generateMockPosts(): List<Post> {
        val users = listOf(
            Triple("user1", "小红书达人", "https://i.pravatar.cc/150?u=user1"),
            Triple("user2", "美食爱好者", "https://i.pravatar.cc/150?u=user2"),
            Triple("user3", "旅行家", "https://i.pravatar.cc/150?u=user3"),
            Triple("user4", "时尚博主", "https://i.pravatar.cc/150?u=user4")
        )
        
        val imageSets = listOf(
            listOf("https://picsum.photos/400/500?random=1"),
            listOf(
                "https://picsum.photos/400/500?random=2",
                "https://picsum.photos/400/500?random=3"
            ),
            listOf(
                "https://picsum.photos/400/500?random=4",
                "https://picsum.photos/400/500?random=5",
                "https://picsum.photos/400/500?random=6"
            )
        )
        
        val contents = listOf(
            "今天天气真好！出门散散步，拍了几张照片分享给大家~ 🌞",
            "周末打卡了这家网红餐厅，味道真的很棒！强烈推荐他们家的招牌菜 🥘",
            "新入手的相机终于到了！迫不及待要出去拍大片了 📷",
            "最近在学习做饭，今天尝试了一道新菜，感觉还不错~ 🍳",
            "旅行日记 | 第三天来到了这个小镇，风景如画，美得让人窒息 🏞️"
        )
        
        return users.flatMapIndexed { userIndex, (userId, userName, avatar) ->
            (0..2).map { postIndex ->
                val index = userIndex * 3 + postIndex
                Post(
                    id = "post_$index",
                    userId = userId,
                    userName = userName,
                    userAvatar = avatar,
                    content = contents[index % contents.size],
                    images = imageSets[index % imageSets.size],
                    likeCount = (10..500).random(),
                    commentCount = (0..50).random(),
                    shareCount = (0..20).random(),
                    isLiked = index % 3 == 0,
                    location = if (index % 2 == 0) "北京·朝阳区" else null,
                    createdAt = LocalDateTime.now().minusHours(index.toLong())
                )
            }
        }.sortedByDescending { it.createdAt }
    }
    
    private fun generateMockComments(): List<Comment> {
        val contents = listOf(
            "太棒了！",
            "这个真的很好看！",
            "求地址！",
            "周末我也去",
            "👍👍👍",
            "羡慕",
            "好美啊"
        )
        
        return (0..20).map { index ->
            Comment(
                id = "comment_$index",
                postId = "post_${index % 12}",
                userId = "user_${index % 4}",
                userName = listOf("小明", "小红", "小李", "小王")[index % 4],
                userAvatar = "https://i.pravatar.cc/150?u=comment$index",
                content = contents[index % contents.size],
                likeCount = (0..100).random(),
                isLiked = index % 4 == 0,
                createdAt = LocalDateTime.now().minusMinutes(index.toLong() * 10)
            )
        }
    }
    
    private fun generateCurrentUser(): User {
        return User(
            id = "current_user",
            username = "currentuser",
            displayName = "当前用户",
            avatar = "https://i.pravatar.cc/150?u=currentuser",
            bio = "这是我的个人简介",
            postCount = 10,
            followerCount = 100,
            followingCount = 50
        )
    }
}
```

### 3. HomeViewModel.kt

```kotlin
// ===== HomeViewModel.kt =====

package com.example.social.ui.screens.home

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.social.data.model.Post
import com.example.social.data.model.User
import com.example.social.data.repository.PostRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class HomeViewModel @Inject constructor(
    private val repository: PostRepository
) : ViewModel() {
    
    // ===== 状态 =====
    
    private val _isRefreshing = MutableStateFlow(false)
    val isRefreshing: StateFlow<Boolean> = _isRefreshing.asStateFlow()
    
    val posts: StateFlow<List<Post>> = repository.getPosts()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    val currentUser: StateFlow<User> = repository.getCurrentUser()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = User("", "", "", null, null)
        )
    
    // ===== 操作 =====
    
    /**
     * 下拉刷新
     */
    fun refresh() {
        viewModelScope.launch {
            _isRefreshing.value = true
            // 模拟网络请求延迟
            kotlinx.coroutines.delay(1000)
            _isRefreshing.value = false
        }
    }
    
    /**
     * 点赞
     */
    fun toggleLike(postId: String) {
        viewModelScope.launch {
            repository.toggleLike(postId)
        }
    }
    
    /**
     * 删除帖子
     */
    fun deletePost(postId: String) {
        viewModelScope.launch {
            repository.deletePost(postId)
        }
    }
}
```

### 4. HomeScreen.kt

```kotlin
// ===== HomeScreen.kt =====

package com.example.social.ui.screens.home

import androidx.compose.animation.animateContentSize
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material.icons.outlined.*
import androidx.compose.material3.*
import androidx.compose.material3.pulltorefresh.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import coil.compose.AsyncImage
import com.example.social.data.model.Post
import com.example.social.ui.components.PostCard

/**
 * 首页
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeScreen(
    onPostClick: (String) -> Unit,
    onCreateClick: () -> Unit,
    onProfileClick: (String) -> Unit,
    viewModel: HomeViewModel = hiltViewModel()
) {
    val posts by viewModel.posts.collectAsState()
    val currentUser by viewModel.currentUser.collectAsState()
    val isRefreshing by viewModel.isRefreshing.collectAsState()
    
    // 下拉刷新状态
    val pullRefreshState = rememberPullToRefreshState()
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(
                        text = "社交动态",
                        fontWeight = FontWeight.Bold
                    )
                },
                actions = {
                    // 消息
                    IconButton(onClick = { /* TODO */ }) {
                        Icon(Icons.Default.Notifications, contentDescription = "消息")
                    }
                    // 用户头像
                    AsyncImage(
                        model = currentUser.avatar,
                        contentDescription = "我的头像",
                        modifier = Modifier
                            .size(36.dp)
                            .clip(CircleShape)
                            .clickable { onProfileClick(currentUser.id) }
                            .padding(end = 8.dp)
                    )
                }
            )
        },
        floatingActionButton = {
            FloatingActionButton(
                onClick = onCreateClick,
                containerColor = MaterialTheme.colorScheme.primary
            ) {
                Icon(Icons.Default.Add, contentDescription = "发布动态")
            }
        }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            if (posts.isEmpty()) {
                // 空状态
                Box(
                    modifier = Modifier.fillMaxSize(),
                    contentAlignment = Alignment.Center
                ) {
                    Column(
                        horizontalAlignment = Alignment.CenterHorizontally
                    ) {
                        Icon(
                            imageVector = Icons.Default.Article,
                            contentDescription = null,
                            modifier = Modifier.size(64.dp),
                            tint = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                        Spacer(modifier = Modifier.height(16.dp))
                        Text(
                            text = "还没有动态",
                            style = MaterialTheme.typography.bodyLarge,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                        Text(
                            text = "点击右下角按钮发布第一条动态",
                            style = MaterialTheme.typography.bodyMedium,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
            } else {
                // 动态列表
                PullToRefreshBox(
                    state = pullRefreshState,
                    isRefreshing = isRefreshing,
                    onRefresh = { viewModel.refresh() },
                    modifier = Modifier.fillMaxSize()
                ) {
                    LazyColumn(
                        modifier = Modifier.fillMaxSize(),
                        contentPadding = PaddingValues(vertical = 8.dp)
                    ) {
                        items(
                            items = posts,
                            key = { it.id }
                        ) { post ->
                            PostCard(
                                post = post,
                                onClick = { onPostClick(post.id) },
                                onLikeClick = { viewModel.toggleLike(post.id) },
                                onCommentClick = { onPostClick(post.id) },
                                onShareClick = { /* TODO */ },
                                onUserClick = { onProfileClick(post.userId) }
                            )
                        }
                    }
                }
            }
        }
    }
}
```

### 5. PostCard.kt（帖子卡片组件）

```kotlin
// ===== PostCard.kt =====

package com.example.social.ui.components

import androidx.compose.animation.animateColorAsState
import androidx.compose.animation.core.animateFloatAsState
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material.icons.outlined.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.graphicsLayer
import androidx.compose.ui.graphics.vector.ImageVector
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage
import com.example.social.data.model.Post

/**
 * 帖子卡片
 * 
 * 包含：用户信息、图片、内容、互动按钮
 */
@Composable
fun PostCard(
    post: Post,
    onClick: () -> Unit,
    onLikeClick: () -> Unit,
    onCommentClick: () -> Unit,
    onShareClick: () -> Unit,
    onUserClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Card(
        modifier = modifier
            .fillMaxWidth()
            .clickable(onClick = onClick),
        shape = RoundedCornerShape(0.dp),  // 全宽卡片
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.surface
        )
    ) {
        Column {
            // ===== 用户信息头部 =====
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(12.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                // 用户头像
                AsyncImage(
                    model = post.userAvatar,
                    contentDescription = post.userName,
                    modifier = Modifier
                        .size(40.dp)
                        .clip(CircleShape)
                        .clickable(onClick = onUserClick),
                    contentScale = ContentScale.Crop
                )
                
                Spacer(modifier = Modifier.width(12.dp))
                
                // 用户名和位置
                Column(
                    modifier = Modifier.weight(1f)
                ) {
                    Text(
                        text = post.userName,
                        style = MaterialTheme.typography.bodyMedium,
                        fontWeight = FontWeight.Bold
                    )
                    
                    if (post.location != null) {
                        Row(
                            verticalAlignment = Alignment.CenterVertically
                        ) {
                            Icon(
                                imageVector = Icons.Default.LocationOn,
                                contentDescription = null,
                                modifier = Modifier.size(12.dp),
                                tint = MaterialTheme.colorScheme.onSurfaceVariant
                            )
                            Spacer(modifier = Modifier.width(2.dp))
                            Text(
                                text = post.location,
                                style = MaterialTheme.typography.bodySmall,
                                color = MaterialTheme.colorScheme.onSurfaceVariant,
                                maxLines = 1,
                                overflow = TextOverflow.Ellipsis
                            )
                        }
                    }
                }
                
                // 发布时间
                Text(
                    text = post.getFormattedTime(),
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            
            // ===== 内容 =====
            Column(
                modifier = Modifier.padding(horizontal = 12.dp)
            ) {
                // 文字内容
                if (post.content.isNotBlank()) {
                    Text(
                        text = post.content,
                        style = MaterialTheme.typography.bodyMedium,
                        maxLines = 10,
                        overflow = TextOverflow.Ellipsis
                    )
                    Spacer(modifier = Modifier.height(8.dp))
                }
            }
            
            // ===== 图片 =====
            if (post.images.isNotEmpty()) {
                ImageGallery(
                    images = post.images,
                    modifier = Modifier.padding(top = 8.dp)
                )
            }
            
            // ===== 互动按钮 =====
            Row(
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(horizontal = 8.dp, vertical = 4.dp),
                horizontalArrangement = Arrangement.SpaceBetween,
                verticalAlignment = Alignment.CenterVertically
            ) {
                // 点赞、评论、分享
                Row {
                    ActionButton(
                        icon = if (post.isLiked) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
                        count = post.likeCount,
                        isActive = post.isLiked,
                        activeColor = Color.Red,
                        onClick = onLikeClick
                    )
                    
                    Spacer(modifier = Modifier.width(8.dp))
                    
                    ActionButton(
                        icon = Icons.Outlined.ChatBubbleOutline,
                        count = post.commentCount,
                        onClick = onCommentClick
                    )
                    
                    Spacer(modifier = Modifier.width(8.dp))
                    
                    ActionButton(
                        icon = Icons.Outlined.Share,
                        count = post.shareCount,
                        onClick = onShareClick
                    )
                }
                
                // 收藏按钮
                IconButton(onClick = { /* TODO */ }) {
                    Icon(
                        imageVector = Icons.Outlined.BookmarkBorder,
                        contentDescription = "收藏"
                    )
                }
            }
            
            HorizontalDivider(modifier = Modifier.padding(top = 8.dp))
        }
    }
}

/**
 * 图片画廊
 */
@Composable
private fun ImageGallery(
    images: List<String>,
    modifier: Modifier = Modifier
) {
    when (images.size) {
        1 -> {
            // 单图
            AsyncImage(
                model = images[0],
                contentDescription = null,
                modifier = modifier
                    .fillMaxWidth()
                    .aspectRatio(1f)
                    .clickable { /* TODO: 查看大图 */ },
                contentScale = ContentScale.Crop
            )
        }
        2 -> {
            // 两图
            Row(
                modifier = modifier.fillMaxWidth()
            ) {
                images.forEach { url ->
                    AsyncImage(
                        model = url,
                        contentDescription = null,
                        modifier = Modifier
                            .weight(1f)
                            .aspectRatio(1f)
                            .clickable { /* TODO */ },
                        contentScale = ContentScale.Crop
                    )
                }
            }
        }
        else -> {
            // 多图网格
            LazyRow(
                contentPadding = PaddingValues(horizontal = 12.dp),
                horizontalArrangement = Arrangement.spacedBy(4.dp)
            ) {
                items(images) { url ->
                    AsyncImage(
                        model = url,
                        contentDescription = null,
                        modifier = Modifier
                            .size(200.dp)
                            .clip(RoundedCornerShape(8.dp))
                            .clickable { /* TODO */ },
                        contentScale = ContentScale.Crop
                    )
                }
            }
        }
    }
}

/**
 * 互动按钮
 */
@Composable
private fun ActionButton(
    icon: ImageVector,
    count: Int,
    isActive: Boolean = false,
    activeColor: Color = MaterialTheme.colorScheme.primary,
    onClick: () -> Unit
) {
    val iconColor by animateColorAsState(
        targetValue = if (isActive) activeColor else MaterialTheme.colorScheme.onSurfaceVariant,
        label = "iconColor"
    )
    
    val scale by animateFloatAsState(
        targetValue = if (isActive) 1.2f else 1f,
        label = "scale"
    )
    
    Row(
        modifier = Modifier
            .clickable(onClick = onClick)
            .padding(8.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Icon(
            imageVector = icon,
            contentDescription = null,
            tint = iconColor,
            modifier = Modifier
                .size(24.dp)
                .graphicsLayer {
                    scaleX = scale
                    scaleY = scale
                }
        )
        if (count > 0) {
            Spacer(modifier = Modifier.width(4.dp))
            Text(
                text = formatCount(count),
                style = MaterialTheme.typography.bodySmall,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
}

/**
 * 格式化数字（如 1000 -> 1k）
 */
private fun formatCount(count: Int): String {
    return when {
        count >= 10000 -> String.format("%.1fw", count / 10000.0)
        count >= 1000 -> String.format("%.1fk", count / 1000.0)
        else -> count.toString()
    }
}

/**
 * 用户头像组件
 */
@Composable
fun UserAvatar(
    avatarUrl: String?,
    size: Int = 40,
    modifier: Modifier = Modifier
) {
    AsyncImage(
        model = avatarUrl ?: "https://i.pravatar.cc/150",
        contentDescription = "用户头像",
        modifier = modifier
            .size(size.dp)
            .clip(CircleShape),
        contentScale = ContentScale.Crop
    )
}
```

### 6. PostDetailScreen.kt

```kotlin
// ===== PostDetailScreen.kt =====

package com.example.social.ui.screens.post

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.CircleShape
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material.icons.outlined.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import coil.compose.AsyncImage
import com.example.social.data.model.Comment
import com.example.social.data.model.Post

/**
 * 帖子详情屏幕
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun PostDetailScreen(
    postId: String,
    onBack: () -> Unit,
    viewModel: PostDetailViewModel = hiltViewModel()
) {
    val post by viewModel.post.collectAsState()
    val comments by viewModel.comments.collectAsState()
    var commentText by remember { mutableStateOf("") }
    
    LaunchedEffect(postId) {
        viewModel.loadPost(postId)
    }
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("动态") },
                navigationIcon = {
                    IconButton(onClick = onBack) {
                        Icon(Icons.Default.ArrowBack, contentDescription = "返回")
                    }
                },
                actions = {
                    IconButton(onClick = { /* TODO */ }) {
                        Icon(Icons.Outlined.MoreHoriz, contentDescription = "更多")
                    }
                }
            )
        },
        bottomBar = {
            // 评论输入栏
            Surface(
                tonalElevation = 3.dp
            ) {
                Row(
                    modifier = Modifier
                        .fillMaxWidth()
                        .padding(horizontal = 16.dp, vertical = 8.dp),
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    OutlinedTextField(
                        value = commentText,
                        onValueChange = { commentText = it },
                        modifier = Modifier.weight(1f),
                        placeholder = { Text("写评论...") },
                        shape = RoundedCornerShape(24.dp),
                        colors = OutlinedTextFieldDefaults.colors(
                            unfocusedBorderColor = Color.Transparent,
                            focusedBorderColor = MaterialTheme.colorScheme.primary
                        )
                    )
                    
                    Spacer(modifier = Modifier.width(8.dp))
                    
                    IconButton(
                        onClick = {
                            if (commentText.isNotBlank()) {
                                viewModel.addComment(postId, commentText)
                                commentText = ""
                            }
                        },
                        enabled = commentText.isNotBlank()
                    ) {
                        Icon(
                            imageVector = Icons.Default.Send,
                            contentDescription = "发送",
                            tint = if (commentText.isNotBlank()) 
                                MaterialTheme.colorScheme.primary 
                            else 
                                MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
            }
        }
    ) { paddingValues ->
        post?.let { currentPost ->
            LazyColumn(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(paddingValues),
                contentPadding = PaddingValues(bottom = 16.dp)
            ) {
                // ===== 帖子内容 =====
                item {
                    PostContent(post = currentPost)
                }
                
                // ===== 评论区标题 =====
                item {
                    Text(
                        text = "评论 ${currentPost.commentCount}",
                        style = MaterialTheme.typography.titleMedium,
                        fontWeight = FontWeight.Bold,
                        modifier = Modifier.padding(16.dp)
                    )
                }
                
                // ===== 评论列表 =====
                items(comments, key = { it.id }) { comment ->
                    CommentItem(comment = comment)
                }
                
                if (comments.isEmpty()) {
                    item {
                        Box(
                            modifier = Modifier
                                .fillMaxWidth()
                                .padding(32.dp),
                            contentAlignment = Alignment.Center
                        ) {
                            Text(
                                text = "还没有评论，快来抢沙发~",
                                color = MaterialTheme.colorScheme.onSurfaceVariant
                            )
                        }
                    }
                }
            }
        } ?: run {
            // 加载中
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(paddingValues),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        }
    }
}

/**
 * 帖子内容
 */
@Composable
private fun PostContent(post: Post) {
    Column {
        // 用户信息
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            verticalAlignment = Alignment.CenterVertically
        ) {
            AsyncImage(
                model = post.userAvatar,
                contentDescription = null,
                modifier = Modifier
                    .size(48.dp)
                    .clip(CircleShape)
            )
            Spacer(modifier = Modifier.width(12.dp))
            Column {
                Text(
                    text = post.userName,
                    style = MaterialTheme.typography.bodyLarge,
                    fontWeight = FontWeight.Bold
                )
                Text(
                    text = post.getFormattedTime(),
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
        }
        
        // 内容
        if (post.content.isNotBlank()) {
            Text(
                text = post.content,
                style = MaterialTheme.typography.bodyLarge,
                modifier = Modifier.padding(horizontal = 16.dp)
            )
        }
        
        // 图片
        post.images.forEach { imageUrl ->
            AsyncImage(
                model = imageUrl,
                contentDescription = null,
                modifier = Modifier
                    .fillMaxWidth()
                    .padding(top = 12.dp),
                contentScale = ContentScale.FillWidth
            )
        }
        
        // 点赞等按钮
        Row(
            modifier = Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalArrangement = Arrangement.SpaceBetween,
            verticalAlignment = Alignment.CenterVertically
        ) {
            Row {
                Icon(
                    imageVector = if (post.isLiked) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
                    contentDescription = "点赞",
                    tint = if (post.isLiked) Color.Red else MaterialTheme.colorScheme.onSurface
                )
                Spacer(modifier = Modifier.width(4.dp))
                Text(text = "${post.likeCount}")
            }
            Row {
                Icon(Icons.Outlined.Share, contentDescription = "分享")
                Spacer(modifier = Modifier.width(16.dp))
                Icon(Icons.Outlined.BookmarkBorder, contentDescription = "收藏")
            }
        }
        
        HorizontalDivider()
    }
}

/**
 * 评论项
 */
@Composable
private fun CommentItem(comment: Comment) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .padding(horizontal = 16.dp, vertical = 8.dp)
    ) {
        AsyncImage(
            model = comment.userAvatar,
            contentDescription = null,
            modifier = Modifier
                .size(36.dp)
                .clip(CircleShape)
        )
        
        Spacer(modifier = Modifier.width(12.dp))
        
        Column(modifier = Modifier.weight(1f)) {
            Row(
                verticalAlignment = Alignment.CenterVertically
            ) {
                Text(
                    text = comment.userName,
                    style = MaterialTheme.typography.bodyMedium,
                    fontWeight = FontWeight.Bold
                )
                Spacer(modifier = Modifier.width(8.dp))
                Text(
                    text = comment.createdAt.toString().take(16),
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
            }
            
            Text(
                text = comment.content,
                style = MaterialTheme.typography.bodyMedium,
                modifier = Modifier.padding(top = 4.dp)
            )
            
            Row(
                modifier = Modifier.padding(top = 4.dp)
            ) {
                Text(
                    text = "回复",
                    style = MaterialTheme.typography.bodySmall,
                    color = MaterialTheme.colorScheme.onSurfaceVariant
                )
                Spacer(modifier = Modifier.width(16.dp))
                Icon(
                    imageVector = if (comment.isLiked) Icons.Filled.Favorite else Icons.Outlined.FavoriteBorder,
                    contentDescription = "点赞",
                    modifier = Modifier.size(16.dp),
                    tint = if (comment.isLiked) Color.Red else MaterialTheme.colorScheme.onSurfaceVariant
                )
                if (comment.likeCount > 0) {
                    Text(
                        text = "${comment.likeCount}",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.onSurfaceVariant
                    )
                }
            }
        }
    }
}
```

### 7. PostDetailViewModel.kt

```kotlin
// ===== PostDetailViewModel.kt =====

package com.example.social.ui.screens.post

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.social.data.model.Comment
import com.example.social.data.model.Post
import com.example.social.data.repository.PostRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class PostDetailViewModel @Inject constructor(
    private val repository: PostRepository
) : ViewModel() {
    
    private val _postId = MutableStateFlow<String?>(null)
    
    val post: StateFlow<Post?> = _postId
        .filterNotNull()
        .flatMapLatest { id -> repository.getPost(id) }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = null
        )
    
    val comments: StateFlow<List<Comment>> = _postId
        .filterNotNull()
        .flatMapLatest { id -> repository.getComments(id) }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = emptyList()
        )
    
    fun loadPost(postId: String) {
        _postId.value = postId
    }
    
    fun toggleLike() {
        viewModelScope.launch {
            _postId.value?.let { repository.toggleLike(it) }
        }
    }
    
    fun addComment(postId: String, content: String) {
        viewModelScope.launch {
            repository.addComment(postId, content)
        }
    }
}
```

### 8. CreatePostScreen.kt

```kotlin
// ===== CreatePostScreen.kt =====

package com.example.social.ui.screens.create

import android.net.Uri
import androidx.activity.compose.rememberLauncherForActivityResult
import androidx.activity.result.contract.ActivityResultContracts
import androidx.compose.foundation.background
import androidx.compose.foundation.border
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import coil.compose.AsyncImage

/**
 * 创建帖子屏幕
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun CreatePostScreen(
    onBack: () -> Unit,
    onSuccess: () -> Unit,
    viewModel: CreatePostViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    var showLocationDialog by remember { mutableStateOf(false) }
    
    LaunchedEffect(uiState.isPosted) {
        if (uiState.isPosted) {
            onSuccess()
        }
    }
    
    // 图片选择器
    val imagePicker = rememberLauncherForActivityResult(
        contract = ActivityResultContracts.GetMultipleContents()
    ) { uris: List<Uri> ->
        viewModel.addImages(uris.map { it.toString() })
    }
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("发布动态") },
                navigationIcon = {
                    IconButton(onClick = onBack) {
                        Icon(Icons.Default.Close, contentDescription = "关闭")
                    }
                },
                actions = {
                    TextButton(
                        onClick = { viewModel.createPost() },
                        enabled = uiState.canPost
                    ) {
                        Text("发布")
                    }
                }
            )
        }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .padding(16.dp)
        ) {
            // 内容输入
            OutlinedTextField(
                value = uiState.content,
                onValueChange = { viewModel.updateContent(it) },
                modifier = Modifier
                    .fillMaxWidth()
                    .weight(1f),
                placeholder = { Text("分享你的想法...") },
                colors = OutlinedTextFieldDefaults.colors(
                    unfocusedBorderColor = androidx.compose.ui.graphics.Color.Transparent,
                    focusedBorderColor = MaterialTheme.colorScheme.primary
                )
            )
            
            // 图片预览
            if (uiState.images.isNotEmpty()) {
                LazyRow(
                    horizontalArrangement = Arrangement.spacedBy(8.dp),
                    modifier = Modifier.padding(vertical = 8.dp)
                ) {
                    items(uiState.images) { imageUri ->
                        Box {
                            AsyncImage(
                                model = imageUri,
                                contentDescription = null,
                                modifier = Modifier
                                    .size(100.dp)
                                    .clip(RoundedCornerShape(8.dp)),
                                contentScale = ContentScale.Crop
                            )
                            
                            // 删除按钮
                            IconButton(
                                onClick = { viewModel.removeImage(imageUri) },
                                modifier = Modifier.align(Alignment.TopEnd)
                            ) {
                                Icon(
                                    imageVector = Icons.Default.Close,
                                    contentDescription = "删除",
                                    tint = MaterialTheme.colorScheme.onSurface
                                )
                            }
                        }
                    }
                    
                    // 添加图片按钮
                    if (uiState.images.size < 9) {
                        item {
                            Box(
                                modifier = Modifier
                                    .size(100.dp)
                                    .clip(RoundedCornerShape(8.dp))
                                    .border(
                                        width = 1.dp,
                                        color = MaterialTheme.colorScheme.outline,
                                        shape = RoundedCornerShape(8.dp)
                                    )
                                    .clickable { imagePicker.launch("image/*") },
                                contentAlignment = Alignment.Center
                            ) {
                                Icon(
                                    imageVector = Icons.Default.AddPhotoAlternate,
                                    contentDescription = "添加图片",
                                    tint = MaterialTheme.colorScheme.onSurfaceVariant
                                )
                            }
                        }
                    }
                }
            }
            
            // 工具栏
            HorizontalDivider(modifier = Modifier.padding(vertical = 8.dp))
            
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceBetween
            ) {
                Row {
                    // 添加图片
                    IconButton(onClick = { imagePicker.launch("image/*") }) {
                        Icon(Icons.Default.Image, contentDescription = "图片")
                    }
                    
                    // @好友
                    IconButton(onClick = { /* TODO */ }) {
                        Icon(Icons.Default.PersonAdd, contentDescription = "@好友")
                    }
                    
                    // 话题
                    IconButton(onClick = { /* TODO */ }) {
                        Icon(Icons.Default.Tag, contentDescription = "话题")
                    }
                }
                
                // 位置
                TextButton(onClick = { showLocationDialog = true }) {
                    Icon(Icons.Default.LocationOn, contentDescription = null)
                    Spacer(modifier = Modifier.width(4.dp))
                    Text(uiState.location ?: "添加位置")
                }
            }
        }
        
        // 位置对话框
        if (showLocationDialog) {
            LocationDialog(
                currentLocation = uiState.location,
                onDismiss = { showLocationDialog = false },
                onConfirm = {
                    viewModel.updateLocation(it)
                    showLocationDialog = false
                }
            )
        }
    }
}

/**
 * 位置对话框
 */
@Composable
private fun LocationDialog(
    currentLocation: String?,
    onDismiss: () -> Unit,
    onConfirm: (String?) -> Unit
) {
    var locationText by remember { mutableStateOf(currentLocation ?: "") }
    
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("添加位置") },
        text = {
            OutlinedTextField(
                value = locationText,
                onValueChange = { locationText = it },
                placeholder = { Text("输入位置...") },
                leadingIcon = {
                    Icon(Icons.Default.LocationOn, contentDescription = null)
                }
            )
        },
        confirmButton = {
            TextButton(onClick = { onConfirm(locationText.ifBlank { null }) }) {
                Text("确定")
            }
        },
        dismissButton = {
            TextButton(onClick = onDismiss) {
                Text("取消")
            }
        }
    )
}
```

### 9. CreatePostViewModel.kt

```kotlin
// ===== CreatePostViewModel.kt =====

package com.example.social.ui.screens.create

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.social.data.repository.PostRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.*
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class CreatePostViewModel @Inject constructor(
    private val repository: PostRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow(CreatePostUiState())
    val uiState: StateFlow<CreatePostUiState> = _uiState.asStateFlow()
    
    fun updateContent(content: String) {
        _uiState.update { it.copy(content = content) }
    }
    
    fun updateLocation(location: String?) {
        _uiState.update { it.copy(location = location) }
    }
    
    fun addImages(images: List<String>) {
        _uiState.update { state ->
            val newImages = (state.images + images).take(9)  // 最多9张
            state.copy(images = newImages)
        }
    }
    
    fun removeImage(image: String) {
        _uiState.update { state ->
            state.copy(images = state.images - image)
        }
    }
    
    fun createPost() {
        val state = _uiState.value
        if (!state.canPost) return
        
        viewModelScope.launch {
            _uiState.update { it.copy(isPosting = true) }
            
            try {
                repository.createPost(
                    content = state.content,
                    images = state.images,
                    location = state.location
                )
                _uiState.update { it.copy(isPosted = true) }
            } catch (e: Exception) {
                _uiState.update { it.copy(isPosting = false) }
            }
        }
    }
}

data class CreatePostUiState(
    val content: String = "",
    val images: List<String> = emptyList(),
    val location: String? = null,
    val isPosting: Boolean = false,
    val isPosted: Boolean = false
) {
    val canPost: Boolean
        get() = (content.isNotBlank() || images.isNotEmpty()) && !isPosting
}
```

---

## 代码关键点说明

### 1. Coil 图片加载

**关键点**：Coil 是 Kotlin 优先的图片加载库。

```kotlin
// 基础用法
AsyncImage(
    model = "https://example.com/image.jpg",
    contentDescription = "图片描述",
    modifier = Modifier.size(100.dp)
)

// 占位图
AsyncImage(
    model = imageUrl,
    contentDescription = null,
    placeholder = Icons.Default.Image,
    error = Icons.Default.BrokenImage
)

// 裁剪
AsyncImage(
    model = imageUrl,
    contentDescription = null,
    modifier = Modifier.clip(CircleShape),
    contentScale = ContentScale.Crop
)
```

### 2. PullRefresh 下拉刷新

```kotlin
// Compose Material 3 下拉刷新
val state = rememberPullToRefreshState()

PullToRefreshBox(
    state = state,
    isRefreshing = isLoading,
    onRefresh = { viewModel.refresh() }
) {
    // 内容
    LazyColumn { ... }
}
```

### 3. rememberLauncherForActivityResult

```kotlin
// 图片选择器
val imagePicker = rememberLauncherForActivityResult(
    contract = ActivityResultContracts.GetMultipleContents()
) { uris: List<Uri> ->
    // 处理选择的图片
    uris.forEach { uri ->
        // uri.toString() 可以作为图片 URL
    }
}

// 使用
Button(onClick = { imagePicker.launch("image/*") }) {
    Text("选择图片")
}
```

---

## 学习计划（5天）

### Day 1：项目架构和数据层

**目标**：搭建项目结构，实现数据层

**任务**：
- [ ] 创建 Compose 项目
- [ ] 定义数据模型
- [ ] 创建 Repository
- [ ] 配置 Hilt

### Day 2：帖子列表

**目标**：实现首页帖子列表

**任务**：
- [ ] PostCard 组件
- [ ] HomeScreen
- [ ] HomeViewModel
- [ ] 下拉刷新

### Day 3：帖子详情

**目标**：实现帖子详情和评论

**任务**：
- [ ] PostDetailScreen
- [ ] 评论列表
- [ ] 评论输入
- [ ] 点赞功能

### Day 4：发布功能

**目标**：实现发布动态

**任务**：
- [ ] 图片选择
- [ ] 图片预览
- [ ] 发布逻辑
- [ ] 位置功能

### Day 5：用户资料

**目标**：实现用户资料页

**任务**：
- [ ] ProfileScreen
- [ ] 用户帖子列表
- [ ] 关注功能
- [ ] UI 优化

---

## 面试要点

### 题目1：Jetpack Compose 中的副作用（Side Effect）？

**标准答案**：

副作用是组件外部的操作，如网络请求、导航等。

```kotlin
// LaunchedEffect：在 Composition 时执行
LaunchedEffect(key) {
    // key 变化时重新执行
    repository.getData()
}

// rememberCoroutineScope：在 Composable 中获取作用域
val scope = rememberCoroutineScope()
scope.launch {
    // 协程代码
}

// rememberUpdatedState：始终获取最新值
val latestData by rememberUpdatedState(newData)
```

---

### 题目2：Kotlin Coroutines Flow 的背压处理？

**标准答案**：

背压处理数据产生速度大于消费速度的情况。

```kotlin
// buffer：缓冲
flow.buffer(64).collect { ... }

// conflate：丢弃中间值
flow.conflate().collect { ... }

// collectLatest：只处理最新值
flow.collectLatest { value ->
    // 处理最新值，丢弃旧的
}

// debounce：防抖
flow.debounce(300).collect { ... }
```

---

### 题目3：Compose 的重组（Recomposition）优化？

**标准答案**：

```kotlin
// 避免不必要的重组
val data by remember(expensiveComputation) { 
    derivedStateOf { /* 派生状态 */ } 
}

// 使用 Stable 注解优化性能
@Stable
class MyUiState {
    // Stable 类型不会触发重组
}

// remember 计算
val value = remember(key) { expensiveOperation() }

// key 标识列表项
items(items, key = { it.id }) { item ->
    // 只有 id 变化时重组
}
```

---

## 项目启动指南

```bash
# 1. 创建 Compose 项目
# Android Studio -> New Project -> Empty Activity (Compose)

# 2. 添加依赖
# 添加 Coil、Hilt、Room 依赖

# 3. 运行
./gradlew assembleDebug
```
