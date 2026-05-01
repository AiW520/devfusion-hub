# Spring + Android - 新闻资讯 App

## 一、项目介绍

### 1.1 项目概述
新闻资讯 App 是一个基于 Spring Boot 后端 + Android 前端的移动应用，支持新闻浏览、分类筛选、收藏点赞、搜索评论等功能。

### 1.2 解决的问题
- **新闻获取**：随时随地获取最新新闻
- **个性化推荐**：根据兴趣推送新闻
- **离线阅读**：支持离线缓存
- **互动评论**：用户可以评论互动

### 1.3 核心技术亮点
- **Spring Boot 后端**：RESTful API
- **Android 前端**：Jetpack Compose + MVVM
- **Retrofit 网络请求**：HTTP 客户端
- **Room 数据库**：本地数据持久化
- **ViewModel + LiveData**：生命周期管理

---

## 二、后端代码

### 2.1 项目结构

```
news-app/
├── backend/
│   ├── pom.xml
│   └── src/main/java/com/news/
│       ├── entity/
│       │   ├── News.java
│       │   ├── Category.java
│       │   └── Comment.java
│       ├── repository/
│       ├── service/
│       ├── controller/
│       └── config/
└── android/
    └── app/src/main/java/com/news/
```

### 2.2 后端 pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.news</groupId>
    <artifactId>news-backend</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 2.3 实体类

#### 2.3.1 新闻实体

```java
package com.news.entity;

import jakarta.persistence.*;
import lombok.Data;
import java.time.LocalDateTime;

/**
 * 新闻实体
 */
@Data
@Entity
@Table(name = "news")
public class News {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 200)
    private String title;

    @Column(columnDefinition = "TEXT")
    private String content;

    @Column(length = 500)
    private String summary;

    @Column(name = "cover_image")
    private String coverImage;

    @Column(name = "source_name")
    private String sourceName;

    @Column(name = "source_url")
    private String sourceUrl;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "category_id")
    private Category category;

    @Column(name = "view_count")
    private Integer viewCount = 0;

    @Column(name = "like_count")
    private Integer likeCount = 0;

    @Column(name = "comment_count")
    private Integer commentCount = 0;

    @Column(name = "is_top")
    private Boolean isTop = false;

    @Column(name = "published_at")
    private LocalDateTime publishedAt;

    @Column(name = "created_at")
    private LocalDateTime createdAt;
}
```

#### 2.3.2 分类实体

```java
package com.news.entity;

import jakarta.persistence.*;
import lombok.Data;

@Data
@Entity
@Table(name = "category")
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 50)
    private String name;

    @Column(length = 50)
    private String slug;

    @Column(name = "icon_url")
    private String iconUrl;
}
```

### 2.4 Service 层

```java
package com.news.service;

import com.news.entity.News;
import com.news.repository.NewsRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class NewsService {

    private final NewsRepository newsRepository;

    @Transactional(readOnly = true)
    public Page<News> getNewsList(Long categoryId, Pageable pageable) {
        if (categoryId != null) {
            return newsRepository.findByCategoryId(categoryId, pageable);
        }
        return newsRepository.findAllByOrderByPublishedAtDesc(pageable);
    }

    @Transactional(readOnly = true)
    public News getNewsById(Long id) {
        News news = newsRepository.findById(id).orElseThrow();
        // 增加浏览量
        news.setViewCount(news.getViewCount() + 1);
        newsRepository.save(news);
        return news;
    }

    @Transactional
    public void likeNews(Long id) {
        News news = newsRepository.findById(id).orElseThrow();
        news.setLikeCount(news.getLikeCount() + 1);
        newsRepository.save(news);
    }
}
```

### 2.5 Controller 层

```java
package com.news.controller;

import com.news.entity.News;
import com.news.service.NewsService;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/news")
@RequiredArgsConstructor
public class NewsController {

    private final NewsService newsService;

    @GetMapping
    public ResponseEntity<Page<News>> getNewsList(
            @RequestParam(required = false) Long categoryId,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        return ResponseEntity.ok(newsService.getNewsList(categoryId, PageRequest.of(page, size)));
    }

    @GetMapping("/{id}")
    public ResponseEntity<News> getNewsById(@PathVariable Long id) {
        return ResponseEntity.ok(newsService.getNewsById(id));
    }

    @PostMapping("/{id}/like")
    public ResponseEntity<Void> likeNews(@PathVariable Long id) {
        newsService.likeNews(id);
        return ResponseEntity.ok().build();
    }
}
```

---

## 三、Android 前端代码

### 3.1 Android 项目配置

#### 3.1.1 build.gradle (app)

```groovy
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
}

android {
    namespace 'com.news.app'
    compileSdk 34

    defaultConfig {
        applicationId "com.news.app"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
        vectorDrawables {
            useSupportLibrary true
        }
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = '17'
    }
    buildFeatures {
        compose true
    }
    composeOptions {
        kotlinCompilerExtensionVersion '1.5.1'
    }
    packaging {
        resources {
            excludes += '/META-INF/{AL2.0,LGPL2.1}'
        }
    }
}

dependencies {
    // Core Android
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.6.2'
    implementation 'androidx.activity:activity-compose:1.8.1'
    
    // Compose
    implementation platform('androidx.compose:compose-bom:2023.10.01')
    implementation 'androidx.compose.ui:ui'
    implementation 'androidx.compose.ui:ui-graphics'
    implementation 'androidx.compose.ui:ui-tooling-preview'
    implementation 'androidx.compose.material3:material3'
    implementation 'androidx.compose.material:material-icons-extended'
    
    // Navigation
    implementation 'androidx.navigation:navigation-compose:2.7.5'
    
    // ViewModel
    implementation 'androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2'
    
    // Retrofit
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.12.0'
    
    // Coil for images
    implementation 'io.coil-kt:coil-compose:2.5.0'
    
    // Room
    implementation 'androidx.room:room-runtime:2.6.1'
    implementation 'androidx.room:room-ktx:2.6.1'
    kapt 'androidx.room:room-compiler:2.6.1'
    
    // Paging
    implementation 'androidx.paging:paging-runtime-ktx:3.2.1'
    implementation 'androidx.paging:paging-compose:3.2.1'
    
    // Testing
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
    androidTestImplementation platform('androidx.compose:compose-bom:2023.10.01')
    androidTestImplementation 'androidx.compose.ui:ui-test-junit4'
    debugImplementation 'androidx.compose.ui:ui-tooling'
    debugImplementation 'androidx.compose.ui:ui-test-manifest'
}
```

### 3.2 API 服务

```kotlin
package com.news.app.data.api

import com.news.app.data.model.News
import retrofit2.http.GET
import retrofit2.http.Path
import retrofit2.http.Query

/**
 * 新闻 API 服务接口
 */
interface NewsApiService {

    /**
     * 获取新闻列表
     */
    @GET("news")
    suspend fun getNewsList(
        @Query("categoryId") categoryId: Long? = null,
        @Query("page") page: Int = 0,
        @Query("size") size: Int = 20
    ): NewsListResponse

    /**
     * 获取新闻详情
     */
    @GET("news/{id}")
    suspend fun getNewsDetail(@Path("id") id: Long): News

    /**
     * 点赞新闻
     */
    @POST("news/{id}/like")
    suspend fun likeNews(@Path("id") id: Long)

    /**
     * 获取分类列表
     */
    @GET("categories")
    suspend fun getCategories(): List<Category>
}
```

### 3.3 数据模型

```kotlin
package com.news.app.data.model

import com.google.gson.annotations.SerializedName

/**
 * 新闻数据模型
 */
data class News(
    @SerializedName("id")
    val id: Long,
    
    @SerializedName("title")
    val title: String,
    
    @SerializedName("content")
    val content: String,
    
    @SerializedName("summary")
    val summary: String?,
    
    @SerializedName("coverImage")
    val coverImage: String?,
    
    @SerializedName("sourceName")
    val sourceName: String?,
    
    @SerializedName("viewCount")
    val viewCount: Int = 0,
    
    @SerializedName("likeCount")
    val likeCount: Int = 0,
    
    @SerializedName("commentCount")
    val commentCount: Int = 0,
    
    @SerializedName("isTop")
    val isTop: Boolean = false,
    
    @SerializedName("publishedAt")
    val publishedAt: String?,
    
    @SerializedName("category")
    val category: Category?
)

/**
 * 分类数据模型
 */
data class Category(
    @SerializedName("id")
    val id: Long,
    
    @SerializedName("name")
    val name: String,
    
    @SerializedName("slug")
    val slug: String?,
    
    @SerializedName("iconUrl")
    val iconUrl: String?
)

/**
 * 新闻列表响应
 */
data class NewsListResponse(
    @SerializedName("content")
    val content: List<News>,
    
    @SerializedName("totalElements")
    val totalElements: Long,
    
    @SerializedName("totalPages")
    val totalPages: Int,
    
    @SerializedName("number")
    val number: Int
)
```

### 3.4 Repository

```kotlin
package com.news.app.data.repository

import androidx.paging.Pager
import androidx.paging.PagingConfig
import androidx.paging.PagingData
import com.news.app.data.api.NewsApiService
import com.news.app.data.model.News
import kotlinx.coroutines.flow.Flow

/**
 * 新闻仓库
 * 负责数据请求和缓存
 */
class NewsRepository(
    private val apiService: NewsApiService
) {
    
    /**
     * 获取新闻列表（分页）
     */
    fun getNewsList(categoryId: Long? = null): Flow<PagingData<News>> {
        return Pager(
            config = PagingConfig(
                pageSize = 20,
                enablePlaceholders = false,
                prefetchDistance = 5
            ),
            pagingSourceFactory = { NewsPagingSource(apiService, categoryId) }
        ).flow
    }

    /**
     * 获取新闻详情
     */
    suspend fun getNewsDetail(id: Long): Result<News> {
        return try {
            val news = apiService.getNewsDetail(id)
            Result.success(news)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    /**
     * 点赞新闻
     */
    suspend fun likeNews(id: Long): Result<Unit> {
        return try {
            apiService.likeNews(id)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

### 3.5 Paging Source

```kotlin
package com.news.app.data.repository

import androidx.paging.PagingSource
import androidx.paging.PagingState
import com.news.app.data.api.NewsApiService
import com.news.app.data.model.News

/**
 * 新闻分页数据源
 */
class NewsPagingSource(
    private val apiService: NewsApiService,
    private val categoryId: Long?
) : PagingSource<Int, News>() {

    override fun getRefreshKey(state: PagingState<Int, News>): Int? {
        // 返回用于刷新数据的键
        return state.anchorPosition?.let { anchorPosition ->
            state.closestPageToPosition(anchorPosition)?.prevKey?.plus(1)
                ?: state.closestPageToPosition(anchorPosition)?.nextKey?.minus(1)
        }
    }

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, News> {
        return try {
            val page = params.key ?: 0
            
            val response = apiService.getNewsList(
                categoryId = categoryId,
                page = page,
                size = params.loadSize
            )

            LoadResult.Page(
                data = response.content,
                prevKey = if (page == 0) null else page - 1,
                nextKey = if (response.content.isEmpty() || page >= response.totalPages - 1) null else page + 1
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }
}
```

### 3.6 ViewModel

```kotlin
package com.news.app.ui.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import androidx.paging.PagingData
import androidx.paging.cachedIn
import com.news.app.data.model.News
import com.news.app.data.repository.NewsRepository
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.launch

/**
 * 新闻列表 ViewModel
 */
class NewsListViewModel(
    private val repository: NewsRepository
) : ViewModel() {

    /**
     * 当前选中的分类 ID
     */
    private var currentCategoryId: Long? = null

    /**
     * 新闻列表 Flow
     */
    val newsFlow: Flow<PagingData<News>> = repository.getNewsList()
        .cachedIn(viewModelScope)

    /**
     * 选择分类
     */
    fun selectCategory(categoryId: Long?) {
        if (currentCategoryId != categoryId) {
            currentCategoryId = categoryId
            // 重新加载数据
            // 实际应用中需要重新触发 PagingSource
        }
    }

    /**
     * 点赞新闻
     */
    fun likeNews(newsId: Long) {
        viewModelScope.launch {
            repository.likeNews(newsId)
        }
    }
}
```

### 3.7 UI 组件

#### 3.7.1 新闻列表页面

```kotlin
package com.news.app.ui.screen

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.Favorite
import androidx.compose.material.icons.outlined.FavoriteBorder
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import androidx.paging.LoadState
import androidx.paging.compose.collectAsLazyPagingItems
import coil.compose.AsyncImage
import com.news.app.data.model.News

/**
 * 新闻列表页面
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun NewsListScreen(
    viewModel: NewsListViewModel,
    onNewsClick: (Long) -> Unit
) {
    val newsList = viewModel.newsFlow.collectAsLazyPagingItems()
    var selectedCategoryId by remember { mutableStateOf<Long?>(null) }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("新闻资讯") },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = MaterialTheme.colorScheme.primary,
                    titleContentColor = MaterialTheme.colorScheme.onPrimary
                )
            )
        }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            // 分类筛选
            CategoryFilter(
                selectedId = selectedCategoryId,
                onSelect = { id ->
                    selectedCategoryId = id
                    viewModel.selectCategory(id)
                }
            )

            // 新闻列表
            when (newsList.loadState.refresh) {
                is LoadState.Loading -> {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        CircularProgressIndicator()
                    }
                }
                is LoadState.Error -> {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        Text("加载失败，请重试")
                    }
                }
                else -> {
                    LazyColumn(
                        modifier = Modifier.fillMaxSize(),
                        contentPadding = PaddingValues(16.dp),
                        verticalArrangement = Arrangement.spacedBy(16.dp)
                    ) {
                        items(newsList.itemCount) { index ->
                            val news = newsList[index]
                            if (news != null) {
                                NewsCard(
                                    news = news,
                                    onClick = { onNewsClick(news.id) },
                                    onLike = { viewModel.likeNews(news.id) }
                                )
                            }
                        }

                        // 加载更多
                        if (newsList.loadState.append is LoadState.Loading) {
                            item {
                                Box(
                                    modifier = Modifier
                                        .fillMaxWidth()
                                        .padding(16.dp),
                                    contentAlignment = Alignment.Center
                                ) {
                                    CircularProgressIndicator(modifier = Modifier.size(24.dp))
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}

/**
 * 新闻卡片组件
 */
@Composable
fun NewsCard(
    news: News,
    onClick: () -> Unit,
    onLike: () -> Unit
) {
    Card(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(onClick = onClick),
        elevation = CardDefaults.cardElevation(defaultElevation = 2.dp),
        shape = RoundedCornerShape(12.dp)
    ) {
        Column {
            // 封面图
            if (!news.coverImage.isNullOrEmpty()) {
                AsyncImage(
                    model = news.coverImage,
                    contentDescription = news.title,
                    modifier = Modifier
                        .fillMaxWidth()
                        .height(180.dp)
                        .clip(RoundedCornerShape(topStart = 12.dp, topEnd = 12.dp)),
                    contentScale = ContentScale.Crop
                )
            }

            Column(
                modifier = Modifier.padding(16.dp)
            ) {
                // 标题
                Text(
                    text = news.title,
                    style = MaterialTheme.typography.titleMedium,
                    fontWeight = FontWeight.Bold,
                    maxLines = 2,
                    overflow = TextOverflow.Ellipsis
                )

                Spacer(modifier = Modifier.height(8.dp))

                // 摘要
                if (!news.summary.isNullOrEmpty()) {
                    Text(
                        text = news.summary,
                        style = MaterialTheme.typography.bodyMedium,
                        color = MaterialTheme.colorScheme.onSurfaceVariant,
                        maxLines = 2,
                        overflow = TextOverflow.Ellipsis
                    )
                    Spacer(modifier = Modifier.height(8.dp))
                }

                // 底部信息
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceBetween,
                    verticalAlignment = Alignment.CenterVertically
                ) {
                    // 来源
                    Text(
                        text = news.sourceName ?: "未知来源",
                        style = MaterialTheme.typography.bodySmall,
                        color = MaterialTheme.colorScheme.outline
                    )

                    // 统计数据
                    Row(
                        horizontalArrangement = Arrangement.spacedBy(16.dp)
                    ) {
                        Text(
                            text = "👁 ${news.viewCount}",
                            style = MaterialTheme.typography.bodySmall
                        )
                        IconButton(
                            onClick = onLike,
                            modifier = Modifier.size(24.dp)
                        ) {
                            Icon(
                                imageVector = if (news.likeCount > 0) 
                                    Icons.Filled.Favorite 
                                else 
                                    Icons.Outlined.FavoriteBorder,
                                contentDescription = "点赞",
                                tint = if (news.likeCount > 0) 
                                    MaterialTheme.colorScheme.error 
                                else 
                                    MaterialTheme.colorScheme.outline
                            )
                        }
                        Text(
                            text = "${news.likeCount}",
                            style = MaterialTheme.typography.bodySmall
                        )
                    }
                }
            }
        }
    }
}

/**
 * 分类筛选组件
 */
@Composable
fun CategoryFilter(
    selectedId: Long?,
    onSelect: (Long?) -> Unit
) {
    val categories = remember {
        listOf(
            Category(0, "全部", null, null),
            Category(1, "科技", "tech", null),
            Category(2, "体育", "sports", null),
            Category(3, "娱乐", "entertainment", null),
            Category(4, "财经", "finance", null),
            Category(5, "国际", "world", null)
        )
    }

    LazyRow(
        modifier = Modifier
            .fillMaxWidth()
            .padding(vertical = 8.dp),
        horizontalArrangement = Arrangement.spacedBy(8.dp),
        contentPadding = PaddingValues(horizontal = 16.dp)
    ) {
        items(categories) { category ->
            FilterChip(
                selected = selectedId == (if (category.id == 0L) null else category.id),
                onClick = { onSelect(if (category.id == 0L) null else category.id) },
                label = { Text(category.name) }
            )
        }
    }
}
```

#### 3.7.2 新闻详情页面

```kotlin
package com.news.app.ui.screen

import androidx.compose.foundation.layout.*
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.ArrowBack
import androidx.compose.material.icons.filled.Share
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import coil.compose.AsyncImage

/**
 * 新闻详情页面
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun NewsDetailScreen(
    newsId: Long,
    viewModel: NewsListViewModel,
    onBack: () -> Unit
) {
    var news by remember { mutableStateOf<com.news.app.data.model.News?>(null) }
    var isLoading by remember { mutableStateOf(true) }

    LaunchedEffect(newsId) {
        // 加载新闻详情
        // 这里简化处理，实际应该调用 API
        isLoading = false
    }

    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("新闻详情") },
                navigationIcon = {
                    IconButton(onClick = onBack) {
                        Icon(Icons.Default.ArrowBack, contentDescription = "返回")
                    }
                },
                actions = {
                    IconButton(onClick = { /* 分享 */ }) {
                        Icon(Icons.Default.Share, contentDescription = "分享")
                    }
                }
            )
        }
    ) { paddingValues ->
        if (isLoading) {
            Box(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(paddingValues),
                contentAlignment = Alignment.Center
            ) {
                CircularProgressIndicator()
            }
        } else {
            Column(
                modifier = Modifier
                    .fillMaxSize()
                    .padding(paddingValues)
                    .verticalScroll(rememberScrollState())
            ) {
                news?.let { article ->
                    // 封面图
                    if (!article.coverImage.isNullOrEmpty()) {
                        AsyncImage(
                            model = article.coverImage,
                            contentDescription = article.title,
                            modifier = Modifier
                                .fillMaxWidth()
                                .height(250.dp),
                            contentScale = ContentScale.Crop
                        )
                    }

                    Column(
                        modifier = Modifier.padding(16.dp)
                    ) {
                        // 标题
                        Text(
                            text = article.title,
                            style = MaterialTheme.typography.headlineSmall,
                            fontWeight = FontWeight.Bold
                        )

                        Spacer(modifier = Modifier.height(12.dp))

                        // 元信息
                        Row(
                            modifier = Modifier.fillMaxWidth(),
                            horizontalArrangement = Arrangement.SpaceBetween
                        ) {
                            Text(
                                text = article.sourceName ?: "未知来源",
                                style = MaterialTheme.typography.bodyMedium,
                                color = MaterialTheme.colorScheme.primary
                            )
                            Text(
                                text = article.publishedAt ?: "",
                                style = MaterialTheme.typography.bodyMedium,
                                color = MaterialTheme.colorScheme.outline
                            )
                        }

                        Spacer(modifier = Modifier.height(24.dp))

                        // 内容
                        Text(
                            text = article.content,
                            style = MaterialTheme.typography.bodyLarge,
                            lineHeight = 28.dp
                        )

                        Spacer(modifier = Modifier.height(32.dp))

                        // 点赞按钮
                        Button(
                            onClick = { viewModel.likeNews(article.id) },
                            modifier = Modifier.fillMaxWidth(),
                            colors = ButtonDefaults.buttonColors(
                                containerColor = MaterialTheme.colorScheme.primary
                            )
                        ) {
                            Text("👍 点赞 (${article.likeCount})")
                        }
                    }
                }
            }
        }
    }
}
```

---

## 四、学习计划（5天）

### 第1天：后端基础
- [ ] 创建 Spring Boot 项目
- [ ] 实现新闻实体和 CRUD
- [ ] 实现分类接口
- [ ] 测试 API

### 第2天：Android 基础
- [ ] 创建 Android 项目
- [ ] 配置依赖
- [ ] 实现数据模型
- [ ] 配置 Retrofit

### 第3天：列表功能
- [ ] 实现 Paging 分页
- [ ] 实现新闻列表 UI
- [ ] 实现分类筛选
- [ ] 实现下拉刷新

### 第4天：详情功能
- [ ] 实现新闻详情 API
- [ ] 实现详情页面
- [ ] 实现点赞功能
- [ ] 实现图片加载

### 第5天：完善优化
- [ ] 添加加载状态
- [ ] 添加错误处理
- [ ] 优化 UI
- [ ] 测试打包

---

## 五、面试要点

### 5.1 Android 架构

**面试题：Android 中的 MVVM 架构是如何实现的？**

**参考答案：**
- **Model**：数据层，负责数据获取和处理
- **View**：UI 层，负责界面展示
- **ViewModel**：视图模型，连接 View 和 Model，管理 UI 状态

```kotlin
// ViewModel
class NewsListViewModel : ViewModel() {
    private val _newsList = MutableLiveData<List<News>>()
    val newsList: LiveData<List<News>> = _newsList
}
```

### 5.2 Jetpack Compose

**面试题：Jetpack Compose 有什么优势？**

**参考答案：**
- **声明式 UI**：描述 UI 而不是操作 UI
- **简洁代码**：减少样板代码
- **实时预览**：提高开发效率
- **响应式**：数据驱动 UI 更新

### 5.3 Retrofit

**面试题：Retrofit 是如何工作的？**

**参考答案：**
- 基于 OkHttp 的网络库
- 使用注解定义 API 接口
- 自动解析 JSON 到数据类
- 支持协程和 LiveData

---

## 六、总结

这个新闻资讯 App 项目涵盖了：

1. **后端**：Spring Boot + JPA + RESTful API
2. **前端**：Android + Jetpack Compose + MVVM
3. **网络**：Retrofit + OkHttp
4. **分页**：Paging 3 库
5. **图片**：Coil 库

通过这个项目，你可以掌握 Android 开发的基本技能。
