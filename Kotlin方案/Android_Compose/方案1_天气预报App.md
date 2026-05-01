# Android + Compose 方案一：天气预报 App

## 项目介绍

### 项目是什么
这是一个现代化的天气预报应用，使用 Kotlin + Jetpack Compose 构建，支持城市搜索、天气预报、空气质量、生活指数等功能。

### 解决什么问题
- 获取实时天气信息
- 多城市天气管理
- 天气变化趋势展示
- 生活指数参考

### 核心技术亮点
| 技术 | 说明 |
|------|------|
| Jetpack Compose | 声明式 UI |
| MVVM + ViewModel | 架构模式 |
| Hilt | 依赖注入 |
| Retrofit + OkHttp | 网络请求 |
| Kotlin Coroutines + Flow | 异步编程 |
| Room | 本地数据库 |
| Navigation Compose | 导航 |

### 适合场景
- Android 开发入门项目
- 学习 Compose 现代 UI
- 展示项目（可上架应用商店）

---

## 完整可运行代码

### 项目结构
```
WeatherApp/
├── app/
│   ├── build.gradle.kts
│   └── src/main/
│       ├── java/com/example/weather/
│       │   ├── WeatherApplication.kt         # Application 类
│       │   ├── MainActivity.kt               # 入口 Activity
│       │   ├── data/
│       │   │   ├── model/                     # 数据模型
│       │   │   │   ├── Weather.kt
│       │   │   │   └── AirQuality.kt
│       │   │   ├── repository/                # 数据仓库
│       │   │   │   └── WeatherRepository.kt
│       │   │   ├── remote/                    # 远程数据源
│       │   │   │   ├── WeatherApi.kt
│       │   │   │   └── RetrofitClient.kt
│       │   │   └── local/                     # 本地数据源
│       │   │       └── WeatherDatabase.kt
│       │   ├── di/                            # 依赖注入
│       │   │   └── AppModule.kt
│       │   ├── ui/
│       │   │   ├── theme/                    # 主题
│       │   │   │   ├── Theme.kt
│       │   │   │   └── Color.kt
│       │   │   ├── navigation/                # 导航
│       │   │   │   └── NavGraph.kt
│       │   │   ├── components/               # 通用组件
│       │   │   │   └── WeatherCard.kt
│       │   │   └── screens/                  # 页面
│       │   │       ├── home/
│       │   │       │   ├── HomeScreen.kt
│       │   │       │   └── HomeViewModel.kt
│       │   │       └── search/
│       │   │           ├── SearchScreen.kt
│       │   │           └── SearchViewModel.kt
│       │   └── util/                         # 工具类
│       │       └── DateUtils.kt
│       └── res/
│           └── values/
│               └── strings.xml
├── build.gradle.kts                          # 根目录构建配置
├── settings.gradle.kts                       # 项目设置
└── gradle.properties                          # Gradle 配置
```

### 1. 根目录 build.gradle.kts

```kotlin
// build.gradle.kts - 根目录构建配置

// 插件版本管理
plugins {
    // Android Gradle 插件
    id("com.android.application") version "8.2.1" apply false
    
    // Kotlin Android 插件
    id("org.jetbrains.kotlin.android") version "1.9.22" apply false
    
    // Hilt 插件
    id("com.google.dagger.hilt.android") version "2.50" apply false
    
    // KSP 插件（用于注解处理器）
    id("com.google.devtools.ksp") version "1.9.22-1.0.17" apply false
}

// 所有项目
allprojects {
    // 配置仓库（在 settings.gradle.kts 中配置）
}

// 清理任务
tasks.register("clean", Delete::class) {
    delete(rootProject.layout.buildDirectory)
}
```

### 2. settings.gradle.kts

```kotlin
// settings.gradle.kts

pluginManagement {
    repositories {
        google()       // Google Maven 仓库
        mavenCentral() // Maven Central
        gradlePluginPortal()  // Gradle 插件门户
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}

rootProject.name = "WeatherApp"
include(":app")
```

### 3. app/build.gradle.kts

```kotlin
// app/build.gradle.kts - 应用模块配置

plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("com.google.dagger.hilt.android")
    id("com.google.devtools.ksp")
}

android {
    namespace = "com.example.weather"
    compileSdk = 34
    
    defaultConfig {
        applicationId = "com.example.weather"
        minSdk = 26           // 最低支持 Android 8.0
        targetSdk = 34        // 目标 SDK
        versionCode = 1
        versionName = "1.0.0"
        
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
        
        // 矢量图支持
        vectorDrawables {
            useSupportLibrary = true
        }
        
        // API Key（生产环境应使用 local.properties 或 CI 环境变量）
        buildConfigField("String", "WEATHER_API_KEY", "\"your_api_key_here\"")
        buildConfigField("String", "WEATHER_BASE_URL", "\"https://api.weatherapi.com/v1/\"")
    }
    
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
        debug {
            isMinifyEnabled = false
        }
    }
    
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    
    kotlinOptions {
        jvmTarget = "17"
    }
    
    buildFeatures {
        compose = true      // 启用 Compose
        buildConfig = true  // 生成 BuildConfig
    }
    
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"  // Compose 编译器版本
    }
    
    packaging {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // ===== AndroidX Core =====
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    
    // ===== Compose BOM =====
    // Compose BOM 统一管理所有 Compose 库版本
    implementation(platform("androidx.compose:compose-bom:2024.01.00"))
    
    // Compose UI
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    
    // Material 3 设计组件
    implementation("androidx.compose.material3:material3")
    
    // 导航组件
    implementation("androidx.navigation:navigation-compose:2.7.6")
    
    // ===== ViewModel =====
    // ViewModel 和 Compose 集成
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.7.0")
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.7.0")
    
    // ===== Hilt =====
    // Hilt 是 Dagger 的 Android 专用版本
    implementation("com.google.dagger:hilt-android:2.50")
    ksp("com.google.dagger:hilt-android-compiler:2.50")
    implementation("androidx.hilt:hilt-navigation-compose:1.1.0")
    
    // ===== Retrofit =====
    // Retrofit 是 Square 出品的 HTTP 客户端
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")
    
    // ===== Room =====
    // Room 是 Android 的本地数据库
    implementation("androidx.room:room-runtime:2.6.1")
    implementation("androidx.room:room-ktx:2.6.1")
    ksp("androidx.room:room-compiler:2.6.1")
    
    // ===== Coil =====
    // Coil 是 Kotlin 协程图片加载库
    implementation("io.coil-kt:coil-compose:2.5.0")
    
    // ===== 数据存储 =====
    // DataStore 是 SharedPreferences 的替代品
    implementation("androidx.datastore:datastore-preferences:1.0.0")
    
    // ===== Coroutines =====
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    
    // ===== 测试 =====
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
    androidTestImplementation(platform("androidx.compose:compose-bom:2024.01.00"))
    androidTestImplementation("androidx.compose.ui:ui-test-junit4")
    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
}
```

### 4. WeatherApplication.kt

```kotlin
package com.example.weather

import android.app.Application
import dagger.hilt.android.HiltAndroidApp

/**
 * Application 类
 * 
 * @HiltAndroidApp 标记应用使用 Hilt 进行依赖注入
 * 所有使用 @HiltAndroidApp 的应用都需要继承 Application
 * 
 * Hilt 会自动处理以下内容：
 * - Application 生命周期
 * - 生成 Hilt 组件
 * - 管理依赖图
 */
@HiltAndroidApp
class WeatherApplication : Application()
```

### 5. MainActivity.kt

```kotlin
package com.example.weather

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import com.example.weather.ui.navigation.WeatherNavHost
import com.example.weather.ui.theme.WeatherTheme
import dagger.hilt.android.AndroidEntryPoint

/**
 * 主 Activity
 * 
 * @AndroidEntryPoint 标记 Activity 使用 Hilt 注入
 * 相当于在 onCreate 中调用 Hilt 来注入字段
 */
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 启用边缘到边缘显示
        // 让内容延伸到状态栏和导航栏后面
        enableEdgeToEdge()
        
        setContent {
            // 应用主题
            WeatherTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    // 导航主机
                    WeatherNavHost()
                }
            }
        }
    }
}
```

### 6. 数据模型

```kotlin
// ===== Weather.kt =====

package com.example.weather.data.model

import com.google.gson.annotations.SerializedName

/**
 * 天气响应（API 返回的完整数据结构）
 * 
 * @SerializedName 用于 JSON 序列化/反序列化
 * 指定 JSON 字段名和 Kotlin 属性名的映射
 */
data class WeatherResponse(
    @SerializedName("location")
    val location: Location,
    
    @SerializedName("current")
    val current: CurrentWeather,
    
    @SerializedName("forecast")
    val forecast: Forecast? = null  // 预报数据可能为空
)

/**
 * 位置信息
 */
data class Location(
    @SerializedName("name")
    val name: String,           // 城市名称
    
    @SerializedName("region")
    val region: String,         // 地区/省份
    
    @SerializedName("country")
    val country: String,        // 国家
    
    @SerializedName("lat")
    val latitude: Double,       // 纬度
    
    @SerializedName("lon")
    val longitude: Double,      // 经度
    
    @SerializedName("tz_id")
    val timezoneId: String,     // 时区 ID
    
    @SerializedName("localtime")
    val localTime: String        // 本地时间
)

/**
 * 当前天气
 */
data class CurrentWeather(
    @SerializedName("temp_c")
    val tempCelsius: Double,    // 温度（摄氏度）
    
    @SerializedName("temp_f")
    val tempFahrenheit: Double, // 温度（华氏度）
    
    @SerializedName("is_day")
    val isDay: Int,             // 是否白天（1=白天，0=夜晚）
    
    @SerializedName("condition")
    val condition: WeatherCondition,
    
    @SerializedName("wind_kph")
    val windKph: Double,        // 风速（km/h）
    
    @SerializedName("wind_mph")
    val windMph: Double,        // 风速（mph）
    
    @SerializedName("wind_dir")
    val windDirection: String,  // 风向（如 N, NE, E...）
    
    @SerializedName("humidity")
    val humidity: Int,          // 湿度（%）
    
    @SerializedName("cloud")
    val cloud: Int,             // 云量（%）
    
    @SerializedName("feelslike_c")
    val feelsLikeCelsius: Double,  // 体感温度（摄氏度）
    
    @SerializedName("uv")
    val uvIndex: Double,        // 紫外线指数
    
    @SerializedName("vis_km")
    val visibilityKm: Double,   // 能见度（km）
    
    @SerializedName("precip_mm")
    val precipitationMm: Double // 降水量（mm）
)

/**
 * 天气状况（如晴、多云、雨天等）
 */
data class WeatherCondition(
    @SerializedName("text")
    val text: String,           // 文字描述
    
    @SerializedName("icon")
    val icon: String,           // 图标 URL
    
    @SerializedName("code")
    val code: Int               // 天气代码
)

/**
 * 天气预报
 */
data class Forecast(
    @SerializedName("forecastday")
    val forecastDays: List<ForecastDay>
)

/**
 * 每日预报
 */
data class ForecastDay(
    @SerializedName("date")
    val date: String,           // 日期
    
    @SerializedName("day")
    val day: DayForecast,
    
    @SerializedName("hour")
    val hourlyForecasts: List<HourForecast>  // 每小时预报
)

/**
 * 每日预报详情
 */
data class DayForecast(
    @SerializedName("maxtemp_c")
    val maxTempCelsius: Double,
    
    @SerializedName("mintemp_c")
    val minTempCelsius: Double,
    
    @SerializedName("avgtemp_c")
    val avgTempCelsius: Double,
    
    @SerializedName("condition")
    val condition: WeatherCondition,
    
    @SerializedName("daily_chance_of_rain")
    val chanceOfRain: Int,      // 下雨概率（%）
    
    @SerializedName("daily_chance_of_snow")
    val chanceOfSnow: Int,      // 下雪概率（%）
    
    @SerializedName("uv")
    val uvIndex: Double
)

/**
 * 每小时预报
 */
data class HourForecast(
    @SerializedName("time")
    val time: String,           // 时间
    
    @SerializedName("temp_c")
    val tempCelsius: Double,
    
    @SerializedName("condition")
    val condition: WeatherCondition,
    
    @SerializedName("chance_of_rain")
    val chanceOfRain: Int
)
```

```kotlin
// ===== AirQuality.kt =====

package com.example.weather.data.model

import com.google.gson.annotations.SerializedName

/**
 * 空气质量响应
 */
data class AirQualityResponse(
    @SerializedName("location")
    val location: Location,
    
    @SerializedName("current")
    val current: AirQuality
)

/**
 * 空气质量数据
 */
data class AirQuality(
    @SerializedName("pollution")
    val pollution: Pollution
)

/**
 * 污染数据
 */
data class Pollution(
    @SerializedName("aqius")
    val aqiUS: Int,             // AQI（美国标准）
    
    @SerializedName("aqicn")
    val aqiCN: Int,             // AQI（中国标准）
    
    @SerializedName("mainus")
    val mainPollutantUS: String, // 主要污染物（美国标准）
    
    @SerializedName("maincn")
    val mainPollutantCN: String  // 主要污染物（中国标准）
) {
    /**
     * 获取空气质量等级描述
     */
    fun getAqiLevel(): AqiLevel {
        return when (aqiUS) {
            in 0..50 -> AqiLevel.GOOD
            in 51..100 -> AqiLevel.MODERATE
            in 101..150 -> AqiLevel.UNHEALTHY_SENSITIVE
            in 151..200 -> AqiLevel.UNHEALTHY
            in 201..300 -> AqiLevel.VERY_UNHEALTHY
            else -> AqiLevel.HAZARDOUS
        }
    }
}

/**
 * 空气质量等级枚举
 */
enum class AqiLevel(val label: String, val description: String) {
    GOOD("优", "空气质量令人满意，基本无空气污染"),
    MODERATE("良", "空气质量可接受，某些污染物对极少数敏感人群健康有较弱影响"),
    UNHEALTHY_SENSITIVE("轻度污染", "易感人群症状有轻度加剧，健康人群出现刺激症状"),
    UNHEALTHY("中度污染", "进一步加剧易感人群症状，可能对心脏、呼吸系统有影响"),
    VERY_UNHEALTHY("重度污染", "心脏病和肺病患者症状显著加剧，运动耐受力降低，健康人群普遍出现症状"),
    HAZARDOUS("严重污染", "健康人群运动耐受力降低，有明显强烈症状，提前出现某些疾病")
}
```

### 7. WeatherApi.kt

```kotlin
package com.example.weather.data.remote

import com.example.weather.data.model.*
import retrofit2.http.GET
import retrofit2.http.Query

/**
 * 天气 API 接口
 * 
 * Retrofit 会根据这些方法自动生成实现类
 * 使用 suspend 关键字支持协程
 */
interface WeatherApi {
    
    /**
     * 获取当前天气和预报
     * 
     * @param apiKey API 密钥
     * @param query 查询城市（可以是名称、经纬度坐标）
     * @param days 预报天数（1-10）
     * @param aqi 是否包含空气质量（yes/no）
     * @param alerts 是否包含警报（yes/no）
     */
    @GET("forecast.json")
    suspend fun getForecast(
        @Query("key") apiKey: String,
        @Query("q") query: String,           // 如 "Beijing" 或 "39.92,116.38"
        @Query("days") days: Int = 7,
        @Query("aqi") aqi: String = "yes",
        @Query("alerts") alerts: String = "no"
    ): WeatherResponse
    
    /**
     * 获取当前天气（不包含预报）
     */
    @GET("current.json")
    suspend fun getCurrentWeather(
        @Query("key") apiKey: String,
        @Query("q") query: String
    ): WeatherResponse
    
    /**
     * 搜索城市
     * 用于自动补全功能
     */
    @GET("search.json")
    suspend fun searchCity(
        @Query("key") apiKey: String,
        @Query("q") query: String
    ): List<CitySearchResult>
}

/**
 * 城市搜索结果
 */
data class CitySearchResult(
    val id: Int,
    val name: String,
    val region: String,
    val country: String,
    val lat: Double,
    val lon: Double
)
```

### 8. RetrofitClient.kt

```kotlin
package com.example.weather.data.remote

import com.example.weather.BuildConfig
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

/**
 * Retrofit 客户端单例
 * 
 * 单例模式：确保全局只有一个 Retrofit 实例
 * 避免重复创建连接池，节省资源
 */
object RetrofitClient {
    
    // ===== 懒加载单例 =====
    // by lazy：第一次访问时初始化，之后直接返回
    val weatherApi: WeatherApi by lazy {
        createRetrofit().create(WeatherApi::class.java)
    }
    
    /**
     * 创建 Retrofit 实例
     */
    private fun createRetrofit(): Retrofit {
        return Retrofit.Builder()
            // 基础 URL（必须以 / 结尾）
            .baseUrl(BuildConfig.WEATHER_BASE_URL)
            
            // 添加 Gson 转换器
            // GsonConverterFactory 会自动将 JSON 转换为 Kotlin 对象
            .addConverterFactory(GsonConverterFactory.create())
            
            // 使用的 HTTP 客户端
            .client(createOkHttpClient())
            
            // 构建
            .build()
    }
    
    /**
     * 创建 OkHttp 客户端
     */
    private fun createOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            // 连接超时
            .connectTimeout(30, TimeUnit.SECONDS)
            
            // 读取超时
            .readTimeout(30, TimeUnit.SECONDS)
            
            // 写入超时
            .writeTimeout(30, TimeUnit.SECONDS)
            
            // 日志拦截器（开发时打印请求日志）
            .addInterceptor(
                HttpLoggingInterceptor().apply {
                    level = if (BuildConfig.DEBUG) {
                        HttpLoggingInterceptor.Level.BODY
                    } else {
                        HttpLoggingInterceptor.Level.NONE
                    }
                }
            )
            
            // 构建
            .build()
    }
}
```

### 9. WeatherRepository.kt

```kotlin
package com.example.weather.data.repository

import com.example.weather.BuildConfig
import com.example.weather.data.model.*
import com.example.weather.data.remote.CitySearchResult
import com.example.weather.data.remote.WeatherApi
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext
import javax.inject.Inject
import javax.inject.Singleton

/**
 * 天气数据仓库
 * 
 * Repository 模式：
 * - 封装数据源（远程 API、本地数据库）
 * - 提供统一的数据访问接口
 * - 处理数据缓存和错误
 */
class WeatherRepository @Inject constructor(
    private val weatherApi: WeatherApi
) {
    // ===== 协程作用域 =====
    // withContext(Dispatchers.IO)：在 IO 线程执行
    // IO 线程适合：网络请求、文件读写、数据库操作
    
    /**
     * 获取天气预报
     */
    suspend fun getWeather(city: String): Result<WeatherResponse> {
        return withContext(Dispatchers.IO) {
            try {
                val response = weatherApi.getForecast(
                    apiKey = BuildConfig.WEATHER_API_KEY,
                    query = city,
                    days = 7,
                    aqi = "yes"
                )
                Result.success(response)
            } catch (e: Exception) {
                // 捕获异常并转换为 Result
                Result.failure(e)
            }
        }
    }
    
    /**
     * 获取当前天气
     */
    suspend fun getCurrentWeather(city: String): Result<WeatherResponse> {
        return withContext(Dispatchers.IO) {
            try {
                val response = weatherApi.getCurrentWeather(
                    apiKey = BuildConfig.WEATHER_API_KEY,
                    query = city
                )
                Result.success(response)
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    /**
     * 搜索城市
     */
    suspend fun searchCity(query: String): Result<List<CitySearchResult>> {
        return withContext(Dispatchers.IO) {
            try {
                val results = weatherApi.searchCity(
                    apiKey = BuildConfig.WEATHER_API_KEY,
                    query = query
                )
                Result.success(results)
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
    
    /**
     * 获取空气质量
     */
    suspend fun getAirQuality(latitude: Double, longitude: Double): Result<AirQualityResponse> {
        return withContext(Dispatchers.IO) {
            try {
                // Weather API 的空气质量需要通过特定端点获取
                // 这里简化处理，使用默认实现
                Result.failure(Exception("Air quality API not implemented"))
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
}
```

### 10. AppModule.kt（Hilt 依赖注入）

```kotlin
package com.example.weather.di

import com.example.weather.data.remote.RetrofitClient
import com.example.weather.data.remote.WeatherApi
import com.example.weather.data.repository.WeatherRepository
import dagger.Module
import dagger.Provides
import dagger.hilt.InstallIn
import dagger.hilt.components.SingletonComponent
import javax.inject.Singleton

/**
 * Hilt 模块
 * 
 * @Module 标记这是一个提供依赖的模块
 * @InstallIn 指定模块安装到的组件
 * 
 * Hilt 组件：
 * - SingletonComponent：应用级别单例
 * - ActivityComponent：Activity 级别
 * - ViewModelComponent：ViewModel 级别
 */
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    
    /**
     * 提供 WeatherApi
     * 
     * @Provides 标记方法是依赖提供者
     * @Singleton 标记依赖是单例
     */
    @Provides
    @Singleton
    fun provideWeatherApi(): WeatherApi {
        return RetrofitClient.weatherApi
    }
    
    /**
     * 提供 WeatherRepository
     */
    @Provides
    @Singleton
    fun provideWeatherRepository(
        weatherApi: WeatherApi
    ): WeatherRepository {
        return WeatherRepository(weatherApi)
    }
}
```

### 11. HomeViewModel.kt

```kotlin
package com.example.weather.ui.screens.home

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.weather.data.model.*
import com.example.weather.data.repository.WeatherRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

/**
 * HomeScreen 的 ViewModel
 * 
 * @HiltViewModel：Hilt 会自动生成 ViewModel
 * 需要在构造函数中注入依赖
 * 
 * ViewModel 职责：
 * - 管理 UI 状态
 * - 处理业务逻辑
 * - 响应用户操作
 */
@HiltViewModel
class HomeViewModel @Inject constructor(
    private val repository: WeatherRepository
) : ViewModel() {
    
    // ===== UI 状态 =====
    // 使用 StateFlow 管理状态，支持 Compose 观察
    
    private val _uiState = MutableStateFlow<HomeUiState>(HomeUiState.Loading)
    val uiState: StateFlow<HomeUiState> = _uiState.asStateFlow()
    
    // 当前城市
    private val _currentCity = MutableStateFlow("Beijing")
    val currentCity: StateFlow<String> = _currentCity.asStateFlow()
    
    // 收藏的城市列表
    private val _savedCities = MutableStateFlow<List<String>>(emptyList())
    val savedCities: StateFlow<List<String>> = _savedCities.asStateFlow()
    
    init {
        // 初始化时加载天气
        loadWeather(_currentCity.value)
    }
    
    /**
     * 加载天气数据
     */
    fun loadWeather(city: String) {
        _currentCity.value = city
        _uiState.value = HomeUiState.Loading
        
        viewModelScope.launch {
            // viewModelScope：ViewModel 的协程作用域
            // 当 ViewModel 被销毁时，协程会自动取消
            
            val result = repository.getWeather(city)
            
            result.fold(
                onSuccess = { response ->
                    _uiState.value = HomeUiState.Success(
                        weather = response,
                        cityName = response.location.name
                    )
                },
                onFailure = { error ->
                    _uiState.value = HomeUiState.Error(
                        message = error.message ?: "加载失败"
                    )
                }
            )
        }
    }
    
    /**
     * 刷新天气
     */
    fun refresh() {
        loadWeather(_currentCity.value)
    }
    
    /**
     * 切换城市
     */
    fun switchCity(city: String) {
        loadWeather(city)
    }
}

/**
 * UI 状态密封类
 * 
 * sealed class：密封类
 * - 所有子类必须在同一个文件中定义
 * - 编译器可以检查 exhaustive（穷尽）条件
 */
sealed class HomeUiState {
    // 加载中状态
    object Loading : HomeUiState()
    
    // 成功状态，包含天气数据
    data class Success(
        val weather: WeatherResponse,
        val cityName: String
    ) : HomeUiState()
    
    // 错误状态
    data class Error(val message: String) : HomeUiState()
}
```

### 12. HomeScreen.kt

```kotlin
package com.example.weather.ui.screens.home

import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyRow
import androidx.compose.foundation.lazy.items
import androidx.compose.foundation.rememberScrollState
import androidx.compose.foundation.verticalScroll
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Brush
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.layout.ContentScale
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.hilt.navigation.compose.hiltViewModel
import coil.compose.AsyncImage
import com.example.weather.data.model.*
import kotlin.math.roundToInt

/**
 * 首页屏幕
 * 
 * @Composable 标记这是一个 Compose 函数
 * Compose 函数可以组合（Composable functions can be composed）
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun HomeScreen(
    onSearchClick: () -> Unit,  // 导航回调
    viewModel: HomeViewModel = hiltViewModel()
) {
    // 观察 ViewModel 的状态
    val uiState by viewModel.uiState.collectAsState()
    val currentCity by viewModel.currentCity.collectAsState()
    
    // Scaffold 提供 Material Design 布局结构
    Scaffold(
        topBar = {
            TopAppBar(
                title = {
                    Text(
                        text = currentCity,
                        style = MaterialTheme.typography.titleLarge
                    )
                },
                actions = {
                    // 搜索按钮
                    IconButton(onClick = onSearchClick) {
                        Icon(Icons.Default.Search, contentDescription = "搜索")
                    }
                },
                colors = TopAppBarDefaults.topAppBarColors(
                    containerColor = Color.Transparent
                )
            )
        }
    ) { paddingValues ->
        Box(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
        ) {
            when (val state = uiState) {
                is HomeUiState.Loading -> {
                    // 加载中
                    LoadingContent()
                }
                
                is HomeUiState.Success -> {
                    // 成功显示天气
                    WeatherContent(
                        weather = state.weather,
                        onRefresh = { viewModel.refresh() }
                    )
                }
                
                is HomeUiState.Error -> {
                    // 错误显示
                    ErrorContent(
                        message = state.message,
                        onRetry = { viewModel.refresh() }
                    )
                }
            }
        }
    }
}

/**
 * 加载中内容
 */
@Composable
private fun LoadingContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        CircularProgressIndicator()
    }
}

/**
 * 错误内容
 */
@Composable
private fun ErrorContent(
    message: String,
    onRetry: () -> Unit
) {
    Column(
        modifier = Modifier.fillMaxSize(),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Icon(
            imageVector = Icons.Default.ErrorOutline,
            contentDescription = null,
            modifier = Modifier.size(64.dp),
            tint = MaterialTheme.colorScheme.error
        )
        Spacer(modifier = Modifier.height(16.dp))
        Text(
            text = message,
            style = MaterialTheme.typography.bodyLarge
        )
        Spacer(modifier = Modifier.height(16.dp))
        Button(onClick = onRetry) {
            Text("重试")
        }
    }
}

/**
 * 天气内容
 */
@Composable
private fun WeatherContent(
    weather: WeatherResponse,
    onRefresh: () -> Unit
) {
    val scrollState = rememberScrollState()
    
    Column(
        modifier = Modifier
            .fillMaxSize()
            .verticalScroll(scrollState)
            .padding(16.dp)
    ) {
        // 当前天气卡片
        CurrentWeatherCard(
            current = weather.current,
            location = weather.location
        )
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 详细天气信息
        WeatherDetailsCard(current = weather.current)
        
        Spacer(modifier = Modifier.height(16.dp))
        
        // 天气预报
        weather.forecast?.let { forecast ->
            ForecastSection(forecastDays = forecast.forecastDays)
        }
    }
}

/**
 * 当前天气卡片
 */
@Composable
private fun CurrentWeatherCard(
    current: CurrentWeather,
    location: Location
) {
    Card(
        modifier = Modifier.fillMaxWidth(),
        colors = CardDefaults.cardColors(
            containerColor = MaterialTheme.colorScheme.primaryContainer
        )
    ) {
        Column(
            modifier = Modifier
                .fillMaxWidth()
                .padding(24.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            // 天气图标
            AsyncImage(
                model = "https:${current.condition.icon}",
                contentDescription = current.condition.text,
                modifier = Modifier.size(100.dp),
                contentScale = ContentScale.Fit
            )
            
            Spacer(modifier = Modifier.height(8.dp))
            
            // 温度
            Text(
                text = "${current.tempCelsius.roundToInt()}°",
                fontSize = 72.sp,
                fontWeight = FontWeight.Bold
            )
            
            // 天气描述
            Text(
                text = current.condition.text,
                style = MaterialTheme.typography.titleMedium
            )
            
            // 体感温度
            Text(
                text = "体感 ${current.feelsLikeCelsius.roundToInt()}°",
                style = MaterialTheme.typography.bodyMedium,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // 位置信息
            Text(
                text = "${location.name}, ${location.country}",
                style = MaterialTheme.typography.bodyMedium
            )
        }
    }
}

/**
 * 天气详情卡片
 */
@Composable
private fun WeatherDetailsCard(current: CurrentWeather) {
    Card(
        modifier = Modifier.fillMaxWidth()
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = "详细信息",
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.Bold
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // 详情网格
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceEvenly
            ) {
                WeatherDetailItem(
                    icon = Icons.Default.Air,
                    label = "风速",
                    value = "${current.windKph.roundToInt()} km/h"
                )
                WeatherDetailItem(
                    icon = Icons.Default.WaterDrop,
                    label = "湿度",
                    value = "${current.humidity}%"
                )
                WeatherDetailItem(
                    icon = Icons.Default.Visibility,
                    label = "能见度",
                    value = "${current.visibilityKm.roundToInt()} km"
                )
            }
            
            Spacer(modifier = Modifier.height(16.dp))
            
            Row(
                modifier = Modifier.fillMaxWidth(),
                horizontalArrangement = Arrangement.SpaceEvenly
            ) {
                WeatherDetailItem(
                    icon = Icons.Default.WbSunny,
                    label = "紫外线",
                    value = "${current.uvIndex.roundToInt()}"
                )
                WeatherDetailItem(
                    icon = Icons.Default.Cloud,
                    label = "云量",
                    value = "${current.cloud}%"
                )
                WeatherDetailItem(
                    icon = Icons.Default.Grain,
                    label = "降水",
                    value = "${current.precipitationMm} mm"
                )
            }
        }
    }
}

/**
 * 天气详情项
 */
@Composable
private fun WeatherDetailItem(
    icon: androidx.compose.ui.graphics.vector.ImageVector,
    label: String,
    value: String
) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Icon(
            imageVector = icon,
            contentDescription = label,
            tint = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.height(4.dp))
        Text(
            text = value,
            style = MaterialTheme.typography.bodyMedium,
            fontWeight = FontWeight.Medium
        )
        Text(
            text = label,
            style = MaterialTheme.typography.bodySmall,
            color = MaterialTheme.colorScheme.onSurfaceVariant
        )
    }
}

/**
 * 天气预报部分
 */
@Composable
private fun ForecastSection(forecastDays: List<ForecastDay>) {
    Card(
        modifier = Modifier.fillMaxWidth()
    ) {
        Column(
            modifier = Modifier.padding(16.dp)
        ) {
            Text(
                text = "天气预报",
                style = MaterialTheme.typography.titleMedium,
                fontWeight = FontWeight.Bold
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // 横向滚动列表
            LazyRow(
                horizontalArrangement = Arrangement.spacedBy(16.dp)
            ) {
                items(forecastDays) { day ->
                    ForecastDayItem(day = day)
                }
            }
        }
    }
}

/**
 * 每日预报项
 */
@Composable
private fun ForecastDayItem(day: ForecastDay) {
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        modifier = Modifier.width(80.dp)
    ) {
        Text(
            text = day.date.takeLast(5),  // 格式：MM-DD
            style = MaterialTheme.typography.bodyMedium
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        AsyncImage(
            model = "https:${day.condition.icon}",
            contentDescription = day.condition.text,
            modifier = Modifier.size(48.dp)
        )
        
        Spacer(modifier = Modifier.height(8.dp))
        
        // 温度范围
        Row {
            Text(
                text = "${day.maxTempCelsius.roundToInt()}°",
                fontWeight = FontWeight.Bold
            )
            Text(
                text = " / ${day.minTempCelsius.roundToInt()}°",
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
        
        Spacer(modifier = Modifier.height(4.dp))
        
        // 降雨概率
        Text(
            text = "💧 ${day.chanceOfRain}%",
            style = MaterialTheme.typography.bodySmall
        )
    }
}
```

### 13. SearchScreen.kt

```kotlin
package com.example.weather.ui.screens.search

import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.hilt.navigation.compose.hiltViewModel
import com.example.weather.data.remote.CitySearchResult

/**
 * 搜索屏幕
 */
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun SearchScreen(
    onCitySelected: (String) -> Unit,
    onBackClick: () -> Unit,
    viewModel: SearchViewModel = hiltViewModel()
) {
    val uiState by viewModel.uiState.collectAsState()
    var searchQuery by remember { mutableStateOf("") }
    
    Scaffold(
        topBar = {
            TopAppBar(
                title = { Text("搜索城市") },
                navigationIcon = {
                    IconButton(onClick = onBackClick) {
                        Icon(Icons.Default.ArrowBack, contentDescription = "返回")
                    }
                }
            )
        }
    ) { paddingValues ->
        Column(
            modifier = Modifier
                .fillMaxSize()
                .padding(paddingValues)
                .padding(horizontal = 16.dp)
        ) {
            // 搜索框
            OutlinedTextField(
                value = searchQuery,
                onValueChange = {
                    searchQuery = it
                    viewModel.search(it)
                },
                modifier = Modifier.fillMaxWidth(),
                placeholder = { Text("输入城市名称...") },
                leadingIcon = {
                    Icon(Icons.Default.Search, contentDescription = null)
                },
                trailingIcon = {
                    if (searchQuery.isNotEmpty()) {
                        IconButton(onClick = {
                            searchQuery = ""
                            viewModel.clearSearch()
                        }) {
                            Icon(Icons.Default.Clear, contentDescription = "清除")
                        }
                    }
                },
                singleLine = true
            )
            
            Spacer(modifier = Modifier.height(16.dp))
            
            // 搜索结果
            when (val state = uiState) {
                is SearchUiState.Idle -> {
                    // 空状态
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        Text(
                            text = "输入城市名称搜索",
                            style = MaterialTheme.typography.bodyLarge,
                            color = MaterialTheme.colorScheme.onSurfaceVariant
                        )
                    }
                }
                
                is SearchUiState.Loading -> {
                    // 加载中
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        CircularProgressIndicator()
                    }
                }
                
                is SearchUiState.Success -> {
                    // 显示结果
                    if (state.results.isEmpty()) {
                        Box(
                            modifier = Modifier.fillMaxSize(),
                            contentAlignment = Alignment.Center
                        ) {
                            Text(
                                text = "未找到城市",
                                style = MaterialTheme.typography.bodyLarge
                            )
                        }
                    } else {
                        LazyColumn {
                            items(state.results) { city ->
                                CityItem(
                                    city = city,
                                    onClick = { onCitySelected("${city.lat},${city.lon}") }
                                )
                            }
                        }
                    }
                }
                
                is SearchUiState.Error -> {
                    Box(
                        modifier = Modifier.fillMaxSize(),
                        contentAlignment = Alignment.Center
                    ) {
                        Text(
                            text = state.message,
                            color = MaterialTheme.colorScheme.error
                        )
                    }
                }
            }
        }
    }
}

/**
 * 城市列表项
 */
@Composable
private fun CityItem(
    city: CitySearchResult,
    onClick: () -> Unit
) {
    Row(
        modifier = Modifier
            .fillMaxWidth()
            .clickable(onClick = onClick)
            .padding(vertical = 12.dp),
        verticalAlignment = Alignment.CenterVertically
    ) {
        Icon(
            imageVector = Icons.Default.LocationOn,
            contentDescription = null,
            tint = MaterialTheme.colorScheme.primary
        )
        Spacer(modifier = Modifier.width(16.dp))
        Column {
            Text(
                text = city.name,
                style = MaterialTheme.typography.bodyLarge
            )
            Text(
                text = "${city.region}, ${city.country}",
                style = MaterialTheme.typography.bodySmall,
                color = MaterialTheme.colorScheme.onSurfaceVariant
            )
        }
    }
    HorizontalDivider()
}
```

### 14. SearchViewModel.kt

```kotlin
package com.example.weather.ui.screens.search

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.weather.data.remote.CitySearchResult
import com.example.weather.data.repository.WeatherRepository
import dagger.hilt.android.lifecycle.HiltViewModel
import kotlinx.coroutines.Job
import kotlinx.coroutines.delay
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.asStateFlow
import kotlinx.coroutines.launch
import javax.inject.Inject

@HiltViewModel
class SearchViewModel @Inject constructor(
    private val repository: WeatherRepository
) : ViewModel() {
    
    private val _uiState = MutableStateFlow<SearchUiState>(SearchUiState.Idle)
    val uiState: StateFlow<SearchUiState> = _uiState.asStateFlow()
    
    // 搜索防抖
    private var searchJob: Job? = null
    
    /**
     * 搜索城市
     * 
     * 使用 debounce 实现搜索防抖
     * 避免用户输入每个字符都发起请求
     */
    fun search(query: String) {
        // 取消之前的搜索
        searchJob?.cancel()
        
        if (query.isBlank()) {
            _uiState.value = SearchUiState.Idle
            return
        }
        
        // 防抖：延迟 300ms 后执行
        searchJob = viewModelScope.launch {
            delay(300)
            
            _uiState.value = SearchUiState.Loading
            
            val result = repository.searchCity(query)
            
            result.fold(
                onSuccess = { results ->
                    _uiState.value = SearchUiState.Success(results)
                },
                onFailure = { error ->
                    _uiState.value = SearchUiState.Error(
                        error.message ?: "搜索失败"
                    )
                }
            )
        }
    }
    
    /**
     * 清除搜索
     */
    fun clearSearch() {
        searchJob?.cancel()
        _uiState.value = SearchUiState.Idle
    }
}

/**
 * 搜索 UI 状态
 */
sealed class SearchUiState {
    object Idle : SearchUiState()
    object Loading : SearchUiState()
    data class Success(val results: List<CitySearchResult>) : SearchUiState()
    data class Error(val message: String) : SearchUiState()
}
```

### 15. NavGraph.kt

```kotlin
package com.example.weather.ui.navigation

import androidx.compose.runtime.Composable
import androidx.navigation.NavHostController
import androidx.navigation.compose.NavHost
import androidx.navigation.compose.composable
import androidx.navigation.compose.rememberNavController
import com.example.weather.ui.screens.home.HomeScreen
import com.example.weather.ui.screens.search.SearchScreen

/**
 * 导航图
 * 
 * 定义应用的所有页面和导航关系
 */
@Composable
fun WeatherNavHost(
    navController: NavHostController = rememberNavController()
) {
    NavHost(
        navController = navController,
        startDestination = Screen.Home.route  // 起始页面
    ) {
        // 首页
        composable(Screen.Home.route) {
            HomeScreen(
                onSearchClick = {
                    navController.navigate(Screen.Search.route)
                }
            )
        }
        
        // 搜索页面
        composable(Screen.Search.route) {
            SearchScreen(
                onCitySelected = { location ->
                    // 返回首页并传递选中的城市
                    navController.previousBackStackEntry?.savedStateHandle?.set(
                        "selectedCity",
                        location
                    )
                    navController.popBackStack()
                },
                onBackClick = {
                    navController.popBackStack()
                }
            )
        }
    }
}

/**
 * 屏幕定义
 */
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Search : Screen("search")
}
```

### 16. Theme.kt

```kotlin
package com.example.weather.ui.theme

import android.app.Activity
import android.os.Build
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.runtime.SideEffect
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.toArgb
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.platform.LocalView
import androidx.core.view.WindowCompat

// ===== 颜色定义 =====

// 晴天颜色
val SkyBlue = Color(0xFF87CEEB)
val SunnyYellow = Color(0xFFFFD700)
val SunnyOrange = Color(0xFFFF8C00)

// 夜晚颜色
val NightBlue = Color(0xFF1A237E)
val MoonYellow = Color(0xFFFFF9C4)

// 雨天气颜色
val RainBlue = Color(0xFF4FC3F7)
val CloudGray = Color(0xFF90A4AE)

// 主题色
val Purple80 = Color(0xFFD0BCFF)
val PurpleGrey80 = Color(0xFFCCC2DC)
val Pink80 = Color(0xFFEFB8C8)

val Purple40 = Color(0xFF6650a4)
val PurpleGrey40 = Color(0xFF625b71)
val Pink40 = Color(0xFF7D5260)

/**
 * 天气主题
 * 根据天气类型返回不同的颜色方案
 */
object WeatherTheme {
    fun getSkyGradient(isDay: Boolean): List<Color> {
        return if (isDay) {
            listOf(SkyBlue, Color.White)
        } else {
            listOf(NightBlue, Color(0xFF311B92))
        }
    }
}

/**
 * 应用主题
 */
private val DarkColorScheme = darkColorScheme(
    primary = Purple80,
    secondary = PurpleGrey80,
    tertiary = Pink80
)

private val LightColorScheme = lightColorScheme(
    primary = Purple40,
    secondary = PurpleGrey40,
    tertiary = Pink40
)

@Composable
fun WeatherTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    dynamicColor: Boolean = true,  // 动态颜色（Material You）
    content: @Composable () -> Unit
) {
    val colorScheme = when {
        dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
            // Material You 动态颜色
            val context = LocalContext.current
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        }
        darkTheme -> DarkColorScheme
        else -> LightColorScheme
    }
    
    val view = LocalView.current
    if (!view.isInEditMode) {
        SideEffect {
            val window = (view.context as Activity).window
            window.statusBarColor = colorScheme.primary.toArgb()
            WindowCompat.getInsetsController(window, view).isAppearanceLightStatusBars = !darkTheme
        }
    }

    MaterialTheme(
        colorScheme = colorScheme,
        typography = Typography,
        content = content
    )
}
```

---

## 代码关键点说明

### 1. Jetpack Compose 核心概念

**关键点**：声明式 UI 范式

```kotlin
// ❌ 命令式 UI（传统方式）
val textView = findViewById<TextView>(R.id.text)
textView.text = "Hello"
textView.setTextColor(Color.RED)

// ✅ 声明式 UI（Compose）
@Composable
fun Greeting() {
    var name by remember { mutableStateOf("") }
    
    Text(
        text = "Hello $name",
        color = Color.Red
    )
}
```

**核心概念**：
- `@Composable`：标记函数可以被 Compose 调用
- `State`：状态驱动 UI 更新
- `remember`：保存Composable 销毁前的状态
- `collectAsState`：将 Flow 转换为 State

### 2. Hilt 依赖注入

**关键点**：依赖注入让代码更可测试、更解耦

```kotlin
// ❌ 直接创建依赖（紧耦合）
class HomeViewModel {
    private val repository = WeatherRepository(
        RetrofitClient.weatherApi  // 直接创建
    )
}

// ✅ 依赖注入（解耦）
class HomeViewModel @Inject constructor(
    private val repository: WeatherRepository  // Hilt 自动注入
) : ViewModel()
```

### 3. Kotlin Coroutines + Flow

**关键点**：响应式编程处理异步数据

```kotlin
// StateFlow：状态流，适合 UI 状态
private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
val uiState: StateFlow<UiState> = _uiState.asStateFlow()

// 在 Compose 中观察
val uiState by viewModel.uiState.collectAsState()

// 协程：处理异步操作
viewModelScope.launch {
    val result = repository.getWeather(city)
    _uiState.value = when {
        result.isSuccess -> UiState.Success(result.getOrNull()!!)
        else -> UiState.Error(result.exceptionOrNull()!!.message)
    }
}
```

---

## 学习计划（7天）

### Day 1：环境搭建和项目创建

**目标**：搭建开发环境，创建 Compose 项目

**任务**：
- [ ] 安装 Android Studio
- [ ] 创建 Compose 项目
- [ ] 配置 Gradle
- [ ] 运行 Hello World

**产出**：
- ✅ 可运行的 Compose 项目

### Day 2：Compose 基础

**目标**：掌握 Compose 核心概念

**任务**：
- [ ] 理解 Composable 函数
- [ ] 掌握 Column、Row、Box 布局
- [ ] 掌握 Material 3 组件
- [ ] 处理用户输入

**产出**：
- ✅ Compose 基础练习项目

### Day 3：网络请求和数据模型

**目标**：实现天气 API 调用

**任务**：
- [ ] 定义数据模型
- [ ] 配置 Retrofit
- [ ] 创建 Repository
- [ ] 处理 API 响应

**产出**：
- ✅ 天气 API 集成

### Day 4：ViewModel 和状态管理

**目标**：掌握 MVVM 架构

**任务**：
- [ ] 创建 ViewModel
- [ ] 使用 StateFlow
- [ ] 状态收集
- [ ] UI 状态设计

**产出**：
- ✅ MVVM 架构实现

### Day 5：Hilt 依赖注入

**目标**：配置 Hilt

**任务**：
- [ ] 配置 Hilt
- [ ] 创建 AppModule
- [ ] @HiltViewModel
- [ ] @AndroidEntryPoint

**产出**：
- ✅ 依赖注入配置

### Day 6：UI 组件开发

**目标**：构建天气 UI

**任务**：
- [ ] 首页天气卡片
- [ ] 搜索页面
- [ ] 导航配置
- [ ] 主题定制

**产出**：
- ✅ 完整的天气 UI

### Day 7：完善和测试

**目标**：项目完善

**任务**：
- [ ] 错误处理
- [ ] 加载状态
- [ ] 单元测试
- [ ] Debug APK

**产出**：
- ✅ 可发布的应用

---

## 面试要点

### 题目1：Jetpack Compose 的工作原理？

**标准答案**：

Compose 使用**编译时注解处理器**和**运行时框架**来构建 UI。

**工作流程**：
1. **编译时**：Compose 编译器将 `@Composable` 函数转换为可组合的描述
2. **重组（Recomposition）**：当状态变化时，只重新执行受影响的 Composable
3. **渲染**：Compose 引擎根据描述生成实际的 UI 元素

**优化策略**：
- 使用 `remember` 保存不需要重建的状态
- 使用 `derivedStateOf` 避免不必要的重组
- 使用 `key` 帮助 Compose 识别列表项

---

### 题目2：Kotlin 协程和 RxJava 有什么区别？

**标准答案**：

| 特性 | Kotlin 协程 | RxJava |
|------|-------------|--------|
| 线程切换 | `withContext(dispatcher)` | `subscribeOn()` / `observeOn()` |
| 背压 | 使用 `suspend` 自然处理 | 内置背压支持 |
| 错误处理 | try-catch | `onError` 回调 |
| 生命周期 | `viewModelScope` 自动取消 | 需要手动管理 |
| 操作符 | 较少 | 非常丰富 |
| 学习曲线 | 低 | 高 |
| 性能 | 好 | 稍重 |

**协程优势**：
```kotlin
// 简洁的异步代码
viewModelScope.launch {
    val weather = repository.getWeather(city)  // 挂起函数
    _state.value = weather
}
```

---

### 题目3：Hilt 依赖注入的原理？

**标准答案**：

Hilt 是 Dagger 的 Android 专用版本，通过**注解处理器**自动生成代码。

**核心注解**：
- `@HiltAndroidApp`：标记 Application
- `@AndroidEntryPoint`：标记 Activity/Fragment
- `@HiltViewModel`：标记 ViewModel
- `@Inject`：标记要注入的字段/构造函数参数
- `@Module`：定义提供依赖的模块
- `@Provides`：标记提供方法

**生成代码**：
```kotlin
// 你写的代码
@HiltViewModel
class HomeViewModel @Inject constructor(
    val repo: Repository
) : ViewModel()

// Hilt 生成的代码（简化）
class HomeViewModel_Factory implements Factory<HomeViewModel> {
    @Override
    public HomeViewModel get() {
        return new HomeViewModel(
            AppModule_ProvideRepositoryFactory.get()
        );
    }
}
```

---

## 项目启动指南

### 前置条件
1. Android Studio Hedgehog (2023.1.1) 或更新版本
2. JDK 17
3. Android SDK 34

### 启动步骤

```bash
# 1. 克隆项目
git clone <project-url>

# 2. 用 Android Studio 打开项目

# 3. 同步 Gradle

# 4. 获取天气 API Key
# 访问 https://www.weatherapi.com/ 注册获取免费 API Key

# 5. 配置 API Key
# 替换 app/build.gradle.kts 中的 WEATHER_API_KEY

# 6. 运行应用
# 点击 Run 按钮，或使用命令行：
./gradlew assembleDebug
adb install app/build/outputs/apk/debug/app-debug.apk
```

### 获取天气 API Key

1. 访问 https://www.weatherapi.com/
2. 注册账号
3. 登录后获取 API Key
4. 免费版每天 100 万次调用，足够学习和测试
