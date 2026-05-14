## 專案交接：Noti Android App

你接手一個進行中的 Android 專案。以下是完整背景，請閱讀後立刻執行任務，不需要問問題。

---

### 專案概述

**Noti** — AI 驅動的 Android 通知管理系統

動機：人每天被 Gmail、LINE、Discord、IG、Facebook、Classroom 等 app 分散注意力。
Noti 把所有通知整合成一條由 AI 主導的清醒資訊流，讓用戶掌控何時被打擾。

核心功能：
- 用 NotificationListenerService 攔截手機所有 app 通知
- 整合成一個終端機風格的 feed（黑底、等寬字體、綠字）
- Claude AI 自動分類每則通知為 URGENT / REPLY / INFO / SKIP
- 可設定哪個時段哪類通知才震動（安靜時段）
- 每天早晨推送今明兩日行程簡報
- 根據明天行事曆推論打包清單（有體育課→帶運動服）
- 可輸入長期目標，AI 分解成每日子任務寫入 Calendar

設計原則：極簡、本地優先（通知內容不上傳雲端）、終端機美學

---

### 技術規格

語言：Kotlin
UI：Jetpack Compose
資料庫：Room (SQLite)
AI：Claude API（Haiku 快速分類、Sonnet 複雜推理）
行程：Google Calendar API
排程：WorkManager
Min SDK：29 (Android 10)
Target SDK：35
Package name：dev.noti.app
AGP：8.7.3
Kotlin：2.0.21
Compose BOM：2024.12.01
Room：2.6.1

---

### 開發進度（4 個 Phase）

P1 ← 你現在要完成這個（建立所有基礎架構並 push）
- NotificationListenerService 攔截所有通知
- Room Database（Entity + DAO + Database singleton + Repository）
- Jetpack Compose 終端機風格 UI（FeedScreen + FeedViewModel）
- MainActivity（啟動時引導用戶授權 Notification Access）
- 所有 Gradle build 設定

P2（下一步）
- Claude API 整合（Haiku 模型）
- 通知自動分類（URGENT / REPLY / INFO / SKIP）
- 安靜時段設定畫面（SettingsScreen）
- 每 app / 每分類的通知行為控制

P3
- Google Calendar API OAuth 設定
- 從通知提取行程事件，一鍵寫入 Calendar
- WorkManager 每日簡報排程
- 打包清單生成邏輯

P4
- 長期目標輸入介面
- Sonnet 模型任務分解推理
- 自動寫入 Calendar 子任務

---

### 通知分類系統

URGENT — 時間敏感，需立即處理
REPLY — 有人等待回覆
INFO — 有用資訊，不需動作
SKIP — 演算法推播，可忽略
UNCLASSIFIED — 尚未經 AI 處理（預設值）

---

### UI 設計規格（終端機風格）

背景色：#000000（純黑）
主色：#00FF41（矩陣綠）
次色：#FFB300（琥珀，用於 REPLY tag）
警示：#FF3B3B（紅，用於 URGENT tag）
暗綠：#003D10（用於 INFO tag 背景、分隔線）
卡片背景：#0A0A0A
Surface：#111111
灰色：#888888（次要文字）
白色：#E8E8E8（主要文字）
字體：FontFamily.Monospace（全部元件）
邊框：無圓角，1dp 線條
互動：點擊展開 → 顯示完整內容 + tag 切換按鈕 + ARCHIVE 按鈕

---

### 完整檔案結構

Noti/
├── .claude/
│   └── settings.json
├── CLAUDE.md
├── HANDOFF.md
├── .gitignore
├── local.properties.example
├── settings.gradle.kts
├── build.gradle.kts
├── gradle/
│   └── wrapper/
│       └── gradle-wrapper.properties
└── app/
    ├── build.gradle.kts
    ├── proguard-rules.pro
    └── src/main/
        ├── AndroidManifest.xml
        └── java/dev/noti/app/
        │   ├── NotiApplication.kt
        │   ├── MainActivity.kt
        │   ├── data/
        │   │   ├── NotificationRepository.kt
        │   │   └── db/
        │   │       ├── NotificationEntity.kt
        │   │       ├── NotificationDao.kt
        │   │       └── NotificationDatabase.kt
        │   ├── service/
        │   │   └── NotiListenerService.kt
        │   └── ui/
        │       ├── theme/
        │       │   ├── Color.kt
        │       │   ├── Theme.kt
        │       │   └── Type.kt
        │       └── screens/
        │           ├── FeedScreen.kt
        │           └── FeedViewModel.kt
        └── res/
            ├── values/
            │   ├── strings.xml
            │   └── themes.xml
            └── xml/
                └── notification_service_config.xml

---

### 各檔案規格

#### .claude/settings.json
內容如下，這個檔案要最先建立：

{
  "permissions": {
    "allow": [
      "Bash(./gradlew *)",
      "Bash(gradle *)",
      "Bash(adb *)",
      "Bash(java *)",
      "Bash(javac *)",
      "Bash(git status)",
      "Bash(git diff*)",
      "Bash(git log*)",
      "Bash(git add *)",
      "Bash(git commit*)",
      "Bash(git push*)",
      "Bash(git checkout*)",
      "Bash(git branch*)",
      "Bash(git fetch*)",
      "Bash(git pull*)",
      "Bash(find . *)",
      "Bash(grep *)",
      "Bash(ls *)",
      "Bash(mkdir *)",
      "Bash(cp *)",
      "Bash(mv *)",
      "Bash(cat *)",
      "Bash(curl *)",
      "Bash(which *)"
    ],
    "deny": []
  }
}

#### settings.gradle.kts

pluginManagement {
    repositories {
        google {
            content {
                includeGroupByRegex("com\\.android.*")
                includeGroupByRegex("com\\.google.*")
                includeGroupByRegex("androidx.*")
            }
        }
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "Noti"
include(":app")

#### build.gradle.kts（root）

plugins {
    id("com.android.application") version "8.7.3" apply false
    id("org.jetbrains.kotlin.android") version "2.0.21" apply false
    id("org.jetbrains.kotlin.plugin.compose") version "2.0.21" apply false
    id("com.google.devtools.ksp") version "2.0.21-1.0.28" apply false
}

#### app/build.gradle.kts

import java.util.Properties

plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
    id("com.google.devtools.ksp")
}

val localProps = Properties().apply {
    rootProject.file("local.properties").takeIf { it.exists() }?.inputStream()?.use { load(it) }
}

android {
    namespace = "dev.noti.app"
    compileSdk = 35

    defaultConfig {
        applicationId = "dev.noti.app"
        minSdk = 29
        targetSdk = 35
        versionCode = 1
        versionName = "0.1.0"
        buildConfigField(
            "String",
            "ANTHROPIC_API_KEY",
            "\"${localProps.getProperty("ANTHROPIC_API_KEY", "")}\""
        )
    }

    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions { jvmTarget = "17" }

    buildFeatures {
        compose = true
        buildConfig = true
    }
}

dependencies {
    val composeBom = platform("androidx.compose:compose-bom:2024.12.01")
    implementation(composeBom)
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.compose.material:material-icons-extended")
    implementation("androidx.activity:activity-compose:1.9.3")
    implementation("androidx.navigation:navigation-compose:2.8.5")
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.8.7")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.7")

    val roomVersion = "2.6.1"
    implementation("androidx.room:room-runtime:$roomVersion")
    implementation("androidx.room:room-ktx:$roomVersion")
    ksp("androidx.room:room-compiler:$roomVersion")

    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")
    implementation("androidx.work:work-runtime-ktx:2.10.0")
    implementation("com.squareup.okhttp3:okhttp:4.12.0")

    debugImplementation("androidx.compose.ui:ui-tooling")
    debugImplementation("androidx.compose.ui:ui-test-manifest")
}

#### gradle/wrapper/gradle-wrapper.properties

distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.9-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists

#### app/src/main/AndroidManifest.xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <uses-permission android:name="android.permission.READ_CALENDAR" />
    <uses-permission android:name="android.permission.WRITE_CALENDAR" />
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".NotiApplication"
        android:allowBackup="false"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.Noti">

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:windowSoftInputMode="adjustResize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".service.NotiListenerService"
            android:exported="false"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE">
            <intent-filter>
                <action android:name="android.service.notification.NotificationListenerService" />
            </intent-filter>
            <meta-data
                android:name="android.service.notification.default_filter_types"
                android:value="conversations|alerting" />
        </service>

    </application>

</manifest>

#### NotiApplication.kt

package dev.noti.app

import android.app.Application
import dev.noti.app.data.db.NotificationDatabase

class NotiApplication : Application() {
    val database by lazy { NotificationDatabase.get(this) }
}

#### data/db/NotificationEntity.kt

package dev.noti.app.data.db

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "notifications")
data class NotificationEntity(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val packageName: String,
    val appLabel: String,
    val title: String,
    val text: String,
    val postedAt: Long,
    val tag: String = "UNCLASSIFIED",
    val isRead: Boolean = false,
    val isArchived: Boolean = false,
)

#### data/db/NotificationDao.kt

package dev.noti.app.data.db

import androidx.room.*
import kotlinx.coroutines.flow.Flow

@Dao
interface NotificationDao {

    @Query("SELECT * FROM notifications WHERE isArchived = 0 ORDER BY postedAt DESC")
    fun observeActive(): Flow<List<NotificationEntity>>

    @Query("SELECT * FROM notifications WHERE tag IN ('URGENT','REPLY') AND isArchived = 0 ORDER BY postedAt DESC")
    fun observeActionRequired(): Flow<List<NotificationEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(entity: NotificationEntity)

    @Query("UPDATE notifications SET tag = :tag WHERE id = :id")
    suspend fun updateTag(id: Long, tag: String)

    @Query("UPDATE notifications SET isRead = 1 WHERE id = :id")
    suspend fun markRead(id: Long)

    @Query("UPDATE notifications SET isArchived = 1 WHERE id = :id")
    suspend fun archive(id: Long)

    @Query("DELETE FROM notifications WHERE isArchived = 1 AND postedAt < :cutoff")
    suspend fun pruneOld(cutoff: Long)
}

#### data/db/NotificationDatabase.kt

package dev.noti.app.data.db

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase

@Database(entities = [NotificationEntity::class], version = 1, exportSchema = false)
abstract class NotificationDatabase : RoomDatabase() {

    abstract fun notificationDao(): NotificationDao

    companion object {
        @Volatile
        private var INSTANCE: NotificationDatabase? = null

        fun get(context: Context): NotificationDatabase =
            INSTANCE ?: synchronized(this) {
                Room.databaseBuilder(
                    context.applicationContext,
                    NotificationDatabase::class.java,
                    "noti.db"
                ).build().also { INSTANCE = it }
            }
    }
}

#### data/NotificationRepository.kt

package dev.noti.app.data

import dev.noti.app.data.db.NotificationDao
import dev.noti.app.data.db.NotificationEntity
import kotlinx.coroutines.flow.Flow

class NotificationRepository(private val dao: NotificationDao) {

    val activeNotifications: Flow<List<NotificationEntity>> = dao.observeActive()
    val actionRequired: Flow<List<NotificationEntity>> = dao.observeActionRequired()

    suspend fun insert(entity: NotificationEntity) = dao.insert(entity)
    suspend fun updateTag(id: Long, tag: String) = dao.updateTag(id, tag)
    suspend fun archive(id: Long) = dao.archive(id)
    suspend fun markRead(id: Long) = dao.markRead(id)

    suspend fun pruneOld(olderThanMs: Long = System.currentTimeMillis() - 7 * 24 * 60 * 60 * 1000L) =
        dao.pruneOld(olderThanMs)
}

#### service/NotiListenerService.kt

package dev.noti.app.service

import android.service.notification.NotificationListenerService
import android.service.notification.StatusBarNotification
import dev.noti.app.NotiApplication
import dev.noti.app.data.NotificationRepository
import dev.noti.app.data.db.NotificationEntity
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.launch

class NotiListenerService : NotificationListenerService() {

    private val job = SupervisorJob()
    private val scope = CoroutineScope(Dispatchers.IO + job)
    private lateinit var repository: NotificationRepository

    override fun onCreate() {
        super.onCreate()
        val db = (applicationContext as NotiApplication).database
        repository = NotificationRepository(db.notificationDao())
    }

    override fun onNotificationPosted(sbn: StatusBarNotification) {
        if (sbn.packageName == packageName) return

        val extras = sbn.notification.extras
        val title = extras.getString("android.title")?.takeIf { it.isNotBlank() } ?: return
        val text = extras.getCharSequence("android.text")?.toString() ?: ""

        scope.launch {
            repository.insert(
                NotificationEntity(
                    packageName = sbn.packageName,
                    appLabel = resolveAppLabel(sbn.packageName),
                    title = title,
                    text = text,
                    postedAt = sbn.postTime,
                )
            )
        }
    }

    override fun onNotificationRemoved(sbn: StatusBarNotification) = Unit

    override fun onDestroy() {
        super.onDestroy()
        job.cancel()
    }

    private fun resolveAppLabel(pkg: String): String = runCatching {
        val info = packageManager.getApplicationInfo(pkg, 0)
        packageManager.getApplicationLabel(info).toString()
    }.getOrDefault(pkg.substringAfterLast('.'))
}

#### ui/theme/Color.kt

package dev.noti.app.ui.theme

import androidx.compose.ui.graphics.Color

val NotiGreen = Color(0xFF00FF41)
val NotiGreenDim = Color(0xFF00C832)
val NotiGreenDark = Color(0xFF003D10)
val NotiAmber = Color(0xFFFFB300)
val NotiRed = Color(0xFFFF3B3B)
val NotiBackground = Color(0xFF000000)
val NotiCard = Color(0xFF0A0A0A)
val NotiSurface = Color(0xFF111111)
val NotiGrey = Color(0xFF888888)
val NotiWhite = Color(0xFFE8E8E8)

#### ui/theme/Type.kt

package dev.noti.app.ui.theme

import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp

val NotiTypography = Typography(
    bodyLarge = TextStyle(fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Normal, fontSize = 14.sp, lineHeight = 22.sp),
    bodyMedium = TextStyle(fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Normal, fontSize = 12.sp, lineHeight = 18.sp),
    bodySmall = TextStyle(fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Normal, fontSize = 10.sp, lineHeight = 15.sp, color = NotiGrey),
    labelSmall = TextStyle(fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Bold, fontSize = 9.sp, letterSpacing = 1.sp),
    titleMedium = TextStyle(fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Bold, fontSize = 14.sp, letterSpacing = 2.sp),
)

#### ui/theme/Theme.kt

package dev.noti.app.ui.theme

import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.darkColorScheme
import androidx.compose.runtime.Composable

private val NotiColorScheme = darkColorScheme(
    primary = NotiGreen,
    onPrimary = NotiBackground,
    primaryContainer = NotiGreenDark,
    onPrimaryContainer = NotiGreen,
    secondary = NotiAmber,
    onSecondary = NotiBackground,
    background = NotiBackground,
    onBackground = NotiWhite,
    surface = NotiCard,
    onSurface = NotiWhite,
    surfaceVariant = NotiSurface,
    onSurfaceVariant = NotiGrey,
    error = NotiRed,
    onError = NotiBackground,
    outline = NotiGreenDark,
)

@Composable
fun NotiTheme(content: @Composable () -> Unit) {
    MaterialTheme(
        colorScheme = NotiColorScheme,
        typography = NotiTypography,
        content = content,
    )
}

#### ui/screens/FeedViewModel.kt

package dev.noti.app.ui.screens

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import androidx.lifecycle.viewModelScope
import dev.noti.app.data.NotificationRepository
import dev.noti.app.data.db.NotificationEntity
import kotlinx.coroutines.flow.SharingStarted
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.flow.stateIn
import kotlinx.coroutines.launch

class FeedViewModel(private val repository: NotificationRepository) : ViewModel() {

    val notifications: StateFlow<List<NotificationEntity>> = repository
        .activeNotifications
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), emptyList())

    fun archive(id: Long) = viewModelScope.launch { repository.archive(id) }
    fun markRead(id: Long) = viewModelScope.launch { repository.markRead(id) }
    fun setTag(id: Long, tag: String) = viewModelScope.launch { repository.updateTag(id, tag) }

    class Factory(private val repository: NotificationRepository) : ViewModelProvider.Factory {
        @Suppress("UNCHECKED_CAST")
        override fun <T : ViewModel> create(modelClass: Class<T>): T =
            FeedViewModel(repository) as T
    }
}

#### ui/screens/FeedScreen.kt

package dev.noti.app.ui.screens

import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.lazy.items
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextOverflow
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import dev.noti.app.data.db.NotificationEntity
import dev.noti.app.ui.theme.*
import java.text.SimpleDateFormat
import java.util.*

@Composable
fun FeedScreen(viewModel: FeedViewModel, modifier: Modifier = Modifier) {
    val notifications by viewModel.notifications.collectAsState()
    Column(modifier = modifier.fillMaxSize().background(NotiBackground)) {
        FeedHeader(count = notifications.size)
        HorizontalDivider(color = NotiGreenDark, thickness = 1.dp)
        if (notifications.isEmpty()) EmptyState()
        else LazyColumn(modifier = Modifier.fillMaxSize()) {
            items(notifications, key = { it.id }) { item ->
                NotificationRow(item, onArchive = { viewModel.archive(item.id) }, onRead = { viewModel.markRead(item.id) }, onTagChange = { viewModel.setTag(item.id, it) })
                HorizontalDivider(color = NotiGreenDark.copy(alpha = 0.4f), thickness = 1.dp)
            }
        }
    }
}

@Composable
private fun FeedHeader(count: Int) {
    Row(modifier = Modifier.fillMaxWidth().padding(horizontal = 16.dp, vertical = 12.dp), horizontalArrangement = Arrangement.SpaceBetween, verticalAlignment = Alignment.CenterVertically) {
        Text("NOTI", color = NotiGreen, fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Bold, fontSize = 16.sp, letterSpacing = 3.sp)
        Text("$count active", color = NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 11.sp)
    }
}

@Composable
private fun NotificationRow(item: NotificationEntity, onArchive: () -> Unit, onRead: () -> Unit, onTagChange: (String) -> Unit) {
    var expanded by remember { mutableStateOf(false) }
    Column(modifier = Modifier.fillMaxWidth().background(if (item.isRead) NotiBackground else NotiCard).clickable { expanded = !expanded; if (!item.isRead) onRead() }.padding(horizontal = 16.dp, vertical = 10.dp)) {
        Row(verticalAlignment = Alignment.CenterVertically, horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            TagChip(item.tag)
            Text(item.appLabel, color = NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 10.sp, modifier = Modifier.weight(1f), maxLines = 1, overflow = TextOverflow.Ellipsis)
            Text(formatTime(item.postedAt), color = NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 10.sp)
        }
        Spacer(Modifier.height(4.dp))
        Text(item.title, color = NotiWhite, fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Bold, fontSize = 12.sp, maxLines = if (expanded) Int.MAX_VALUE else 1, overflow = TextOverflow.Ellipsis)
        if (item.text.isNotBlank()) Text(item.text, color = NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 11.sp, maxLines = if (expanded) Int.MAX_VALUE else 1, overflow = TextOverflow.Ellipsis, modifier = Modifier.padding(top = 2.dp))
        if (expanded) { Spacer(Modifier.height(8.dp)); TagRow(item.tag, onTagChange, onArchive) }
    }
}

@Composable
private fun TagChip(tag: String) {
    val (bg, fg) = when (tag) {
        "URGENT" -> NotiRed to NotiBackground
        "REPLY" -> NotiAmber to NotiBackground
        "INFO" -> NotiGreenDark to NotiGreen
        "SKIP" -> NotiSurface to NotiGrey
        else -> NotiGreenDark.copy(alpha = 0.3f) to NotiGrey
    }
    Box(modifier = Modifier.background(bg).padding(horizontal = 5.dp, vertical = 2.dp)) {
        Text(tag, color = fg, fontFamily = FontFamily.Monospace, fontWeight = FontWeight.Bold, fontSize = 8.sp, letterSpacing = 1.sp)
    }
}

@Composable
private fun TagRow(currentTag: String, onTagChange: (String) -> Unit, onArchive: () -> Unit) {
    Row(horizontalArrangement = Arrangement.spacedBy(6.dp)) {
        listOf("URGENT","REPLY","INFO","SKIP").forEach { tag ->
            val sel = tag == currentTag
            Box(modifier = Modifier.background(if (sel) NotiGreenDark else NotiSurface).clickable { onTagChange(tag) }.padding(horizontal = 8.dp, vertical = 4.dp)) {
                Text(tag, color = if (sel) NotiGreen else NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 9.sp, fontWeight = if (sel) FontWeight.Bold else FontWeight.Normal)
            }
        }
        Spacer(Modifier.weight(1f))
        Box(modifier = Modifier.background(NotiSurface).clickable(onClick = onArchive).padding(horizontal = 8.dp, vertical = 4.dp)) {
            Text("ARCHIVE", color = NotiGrey, fontFamily = FontFamily.Monospace, fontSize = 9.sp)
        }
    }
}

@Composable
private fun EmptyState() {
    Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        Column(horizontalAlignment = Alignment.CenterHorizontally) {
            Text("> no notifications", color = NotiGreenDark, fontFamily = FontFamily.Monospace, fontSize = 13.sp)
            Text("waiting for input_", color = NotiGreenDark, fontFamily = FontFamily.Monospace, fontSize = 11.sp, modifier = Modifier.padding(top = 4.dp))
        }
    }
}

private fun formatTime(epochMs: Long): String {
    val diff = System.currentTimeMillis() - epochMs
    return when {
        diff < 60_000 -> "just now"
        diff < 3_600_000 -> "${diff / 60_000}m ago"
        diff < 86_400_000 -> "${diff / 3_600_000}h ago"
        else -> SimpleDateFormat("MM/dd", Locale.getDefault()).format(Date(epochMs))
    }
}

#### MainActivity.kt

package dev.noti.app

import android.content.Intent
import android.os.Bundle
import android.provider.Settings
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.*
import androidx.compose.material3.AlertDialog
import androidx.compose.material3.Text
import androidx.compose.material3.TextButton
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.font.FontFamily
import androidx.lifecycle.viewmodel.compose.viewModel
import dev.noti.app.data.NotificationRepository
import dev.noti.app.ui.screens.FeedScreen
import dev.noti.app.ui.screens.FeedViewModel
import dev.noti.app.ui.theme.NotiBackground
import dev.noti.app.ui.theme.NotiGreen
import dev.noti.app.ui.theme.NotiTheme

class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        val repository = NotificationRepository((application as NotiApplication).database.notificationDao())
        setContent {
            NotiTheme {
                var showDialog by remember { mutableStateOf(!hasNotificationAccess()) }
                Box(modifier = Modifier.fillMaxSize().background(NotiBackground).windowInsetsPadding(WindowInsets.systemBars)) {
                    FeedScreen(viewModel = viewModel(factory = FeedViewModel.Factory(repository)))
                }
                if (showDialog) {
                    NotificationAccessDialog(onConfirm = { showDialog = false; startActivity(Intent(Settings.ACTION_NOTIFICATION_LISTENER_SETTINGS)) }, onDismiss = { showDialog = false })
                }
            }
        }
    }

    private fun hasNotificationAccess(): Boolean {
        val flat = Settings.Secure.getString(contentResolver, "enabled_notification_listeners")
        return flat?.contains(packageName) == true
    }
}

@Composable
private fun NotificationAccessDialog(onConfirm: () -> Unit, onDismiss: () -> Unit) {
    AlertDialog(
        onDismissRequest = onDismiss,
        title = { Text("NOTIFICATION ACCESS", fontFamily = FontFamily.Monospace, color = NotiGreen) },
        text = { Text("Noti needs notification access.\n\nSettings → Notifications → Notification Access → enable NOTI.", fontFamily = FontFamily.Monospace) },
        confirmButton = { TextButton(onClick = onConfirm) { Text("OPEN SETTINGS", fontFamily = FontFamily.Monospace, color = NotiGreen) } },
        dismissButton = { TextButton(onClick = onDismiss) { Text("LATER", fontFamily = FontFamily.Monospace) } },
    )
}

#### res/values/strings.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">NOTI</string>
</resources>

#### res/values/themes.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="Theme.Noti" parent="android:Theme.Material.NoTitleBar">
        <item name="android:windowBackground">@android:color/black</item>
        <item name="android:statusBarColor">@android:color/black</item>
        <item name="android:navigationBarColor">@android:color/black</item>
    </style>
</resources>

#### res/xml/notification_service_config.xml

<?xml version="1.0" encoding="utf-8"?>
<notification-listener
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:filterTypes="conversations|alerting" />

#### app/proguard-rules.pro

-keep class * extends androidx.room.RoomDatabase
-keep @androidx.room.Entity class *
-keepclassmembers @androidx.room.Entity class * { *; }

#### .gitignore

.gradle/
build/
local.properties
*.apk
*.aab
*.keystore
*.jks
.idea/
*.iml
.DS_Store
.claude/settings.local.json

#### local.properties.example

# 複製成 local.properties 並填入（不要 commit local.properties）
sdk.dir=/path/to/Android/Sdk
ANTHROPIC_API_KEY=sk-ant-api03-...

---

### 你現在的任務

立刻建立以上所有檔案並 push 到 main branch。

執行順序：
1. 先建立 .claude/settings.json
2. 建立其餘所有檔案和目錄
3. git add -A
4. git commit -m "feat(p1): initial scaffold — notification intercept + Room DB + terminal UI"
5. git push

完成後報告「P1 已完成」，然後等待下一個指令。
下一個指令會是：「開始 P2：Claude API 分類 + 安靜時段控制」
